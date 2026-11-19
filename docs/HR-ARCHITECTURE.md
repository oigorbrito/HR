# HR runtime architecture baseline

**Status:** evidence-based architecture baseline; no runtime redesign is proposed here.

## 1. System role

HR is a **router and orchestrator of multiple harnesses**. It is not itself one agent harness. A harness owns its own agent loop, tools and session semantics; HR exposes a stable server interface, selects a configured harness, drives its runtime through an adapter, and normalizes progress and results.

This boundary is explicit in the Unified Harness Protocol (UHP): a client speaks to a server, the server drives one or more harnesses, and harness execution details stay behind the server boundary.

## 2. Minimal runtime topology

```text
Client / product / CLI
        |
        | UHP over HTTP; Responses-compatible task surface
        v
Gateway
  - authentication and tenant/workspace scope
  - configured-harness lookup and selection
  - model/provider availability and request policy
  - task/session/response lifecycle and event normalization
        |
        | pool / runner endpoint
        v
Runner + base-specific drivers
        |
        v
Harness runtime (Codex, Claude Code, Pi, Hermes, ...)

Gateway persistence:
  GraphStore | BlobStore | SecretStore | WorkspaceFiles
  Separate control store: idempotency, leases, response state/cursors
```

The gateway is the stable integration boundary. The runner is the execution boundary. The drivers adapt individual harnesses; they do not replace the harnesses or define their quality.

The client selects a configured harness through `metadata.harness_id`. A configured harness is a base plus its model, prompt, tools, skills, plugins, environment and budgets. The model is a separate selection dimension: HR must either run the requested model or make any substitution observable.

## 3. Keep three evaluation planes separate

### A. Harness quality

Question: *How well does a particular harness perform a task with a particular model and configuration?*

Evidence already present in this repository:
- `scripts/support-matrix/README.md`: five-scenario harness/model protocol (first turn, follow-up, model switch, artifact, recycle), with per-pair records and explicit failure handling.
- `docs/support-matrix.md` and `docs/support-matrix-notes.md`: recorded support-matrix outcomes and run conditions.
- `docs/benchmark.md` and `scripts/benchmark/README.md`: task-pack benchmark records, graders, token/latency/tool-call metrics and exclusion rules.

These results are scoped to their recorded provider, model, harness version, instance and task pack. They are not a universal ranking of harnesses.

### B. Router and orchestration

Question: *Does HR select, configure, invoke and observe the requested harness correctly?*

Evidence:
- `protocol/versions/2026-10-19/architecture.md`: Client → Server → Harness boundary and server responsibilities.
- `protocol/versions/2026-10-19/tasks.md`: configured-harness selection, model selection/substitution reporting, response semantics and streaming/task fields.
- `gateway/app.py`: gateway implementation and API/task orchestration.
- `runner/server.py` and `runner/*_driver.py`: execution endpoint and base-specific adapters.
- `protocol/conformance/`: executable UHP conformance suite and historical reports.

A router pass does not prove that a harness is good at the task. A harness benchmark pass does not prove correct routing, isolation, streaming or response semantics.

### C. Durability, fencing and recovery

Question: *Does state survive retries, concurrent turns, process loss and workspace restoration?*

These are related but distinct mechanisms:

- **Durable control state:** `gateway/control_store.py` stores idempotency reservations, per-session leases, response terminal state and harvest cursors separately from graph metadata.
- **Compare-and-swap:** `gateway/control_sqlite.py` implements ETag-style replacement inside an IMMEDIATE SQLite transaction; the hosted control-store path uses conditional replacement. A lost CAS is a conflict, not an overwrite.
- **Idempotency:** reservation is keyed by organization and request key. A duplicate reads and replays the winning response. A reservation-store failure is designed to fail closed.
- **Fencing:** session leases carry a monotonic fence and owner; renew/release/checkpoint operations compare the fence. This path is best-effort in places and explicitly has fail-open behavior when the control store is unavailable. Do not describe fencing as an unconditional guarantee without checking the exact call path and failure mode.
- **Checkpoint/recovery:** `gateway/backing.py` separates graph, blob, secret and workspace-file interfaces. Checkpoints preserve session workspace state; `gateway/tests/test_step_checkpoint.py` documents mid-turn versus end-of-turn checkpoint behavior.
- **Branch restore:** `runner/branching.py` restores a selected committed checkpoint into a fresh session repository. It verifies Git objects and excludes CLI credentials/configuration, dependency directories and other non-user state. It restores committed user files, not a harness's native process or conversation state.

A durable checkpoint is not the same thing as a live sandbox. A fencing token is not the same thing as a successful recovery. Idempotent request replay is not a substitute for checkpoint correctness.

## 4. Storage boundary

`gateway/backing.py` defines semantic interfaces rather than backend-specific query APIs:

| Interface | Responsibility |
|---|---|
| GraphStore | harness, session, response, API-key records and ownership edges |
| BlobStore | traces, response records, checkpoints and produced files |
| SecretStore | provider credentials and policies |
| WorkspaceFiles | session-scoped file reads and writes |

The local mode uses SQLite/filesystem/environment-backed storage. The hosted mode delegates to remote backing services. The control store is intentionally separate from the graph/blob interfaces because reservation, lease and terminal-state races require atomic control operations.

## 5. Existing evidence; no benchmark restart

This migration did not rerun Pi, Marshall, Orca, fencing, idempotency or CAS/OCC experiments. Existing measurements and artifacts remain the evidence base. The migration validation workflow runs the repository's ordinary regression/conformance suites only; it does not launch the support-matrix or task-pack benchmark runners.

The historical UHP report `protocol/conformance/reports/harnessrouter/2026-09-27T2228Z-full.json` records 76/76 checks for the hosted target under protocol version 2026-09-12. It is historical evidence, not a claim that the current 2026-10-19 implementation has been freshly measured against that report.

## 6. Architecture rule

Reuse the existing gateway, UHP contract, runner adapters and storage/control boundaries. Do not add a second router, a duplicate control store, or a new orchestration layer unless a reproducible gate demonstrates a concrete missing capability.

Compare after the relevant gate. Correct only what fails or is demonstrably absent.

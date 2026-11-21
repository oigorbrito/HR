# MH snapshot imported into HR

- Source repository: https://github.com/oigorbrito/mh
- Source branch: main
- Exact source commit: 0f627064d17f24d3b4732d02837131b0f06e99d4
- Source tree: 2475a1748f2e7a3389e6425ec26a1bd9980d1820
- Snapshot method: Git tree snapshot; no MH commit ancestry imported
- Files copied: 1264
- Source bytes represented by copied blobs: 55817126
- New history: one independent root commit
- Excluded own change: `.github/workflows/upstream-tracker.yml`
- Excluded blob: ae26a3fbbdb839ffbdc0d7cc233abf7985873939
- Exclusion reason: workflow invokes `.github/scripts/sync-upstream.py`, absent from the source tree
- Other own work: MH PRs #1, #2, and #3 were open at migration time and were not integrated, as instructed
- Backup: `backup/pre-HR-migration-2026-11-19` in MH
- HR template backup: `backup/pre-MH-migration-2026-11-19` in HR

This is a source snapshot, not an upstream merge. Upstream files and modes are preserved except for the explicitly identified broken workflow. Application validation is separate and is not implied by this manifest.

## Validation evidence

- Final candidate CI run: https://github.com/oigorbrito/HR/actions/runs/44666200244
- Tested commit: `dbd085405f1123fbbb185c61c5ceb943a9ef032f` on `migration/mh-clean-history`.
- Conclusion: success; all 10 jobs passed, including gateway, runner, root-only runner tests, integration, console build/type-check, protocol conformance, site build, install pins, canary and deploy checks.
- The first candidate run exposed one stale OpenRouter model route: `grok-build-0.1 -> x-ai/grok-build-0.1` was absent from the live `GET /api/v1/models` response. The legacy detail page still resolved, but the model was not in the provider's live catalogue. The OpenRouter route is now excluded fail-closed; the model remains in the broader catalog for other providers that still expose it.
- The capability test's source parser previously failed to recognize the typed `_OPENROUTER_NO_CHANNEL: set[str]` declaration. Its regex now accepts the annotation, so it tests the actual exclusion table instead of silently ignoring it.
- Run 44666200244 is the qualification evidence for the application tree and these corrections. The manifest-only update that records this evidence does not change application code.
- The migration validation did not rerun Pi, Marshall, Orca, fencing, idempotency or CAS/OCC benchmarks.

## Promotion gate

- Default branch promoted to `migration/mh-clean-history`.
- Repository ruleset `27663587` updated to require the 10 active workflow check contexts (`gateway`, `console`, `runner`, `conformance`, `integration`, `install-pins`, `canary-gate`, `site`, `memories-deploy`, `readme-badge`), removing obsolete checks such as `cargo test`.
- Backups remain intact (`backup/pre-MH-migration-2026-11-19` in HR and `backup/pre-HR-migration-2026-11-19` in MH); previous `main` ref is preserved.
- Post-promotion validation: controlled PR to verify ruleset status check enforcement on the promoted default branch.

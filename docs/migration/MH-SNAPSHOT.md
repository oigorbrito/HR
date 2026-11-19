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

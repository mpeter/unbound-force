---
description: >
  Run gaze-py contract coverage analysis on this Python project.
  Detects side effects and maps them to test assertions.
---
<!-- scaffolded by uf v0.15.0 -->

# Command: /gaze-report

## Description

Run `gaze-py report` on this Python project to compute GazeCRAP scores
and contract coverage for all analyzed functions.

## Instructions

1. Check that `gaze-py` is available:

   ```bash
   which gaze-py
   ```

   If not found, emit: "gaze-py not found. Run `uf init` to install it."
   and stop.

2. Determine source and test paths:
   - Check for `.gaze.yaml` and read `src_path` / `tests_path` if present.
   - Default: `src/` for source, `tests/` for tests.

3. Run analysis:

   ```bash
   gaze-py report src/ tests/ --format=json
   ```

4. Parse and summarize the JSON output for the agent context.
   Highlight functions with GazeCRAP > 15 as needing attention.

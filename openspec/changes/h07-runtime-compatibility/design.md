## Context

The supported-runtime H-07 run failed reviewer runtime inspection,
helper coverage (78.79% versus 80%), and scaffold temporary-directory
cleanup. The proposal assesses all six constitution principles.

## Goals / Non-Goals

### Goals
- Preserve reviewer policy isolation under supported runtime inspection.
- Verify real Homebrew failure outcomes and existing output preservation.
- Resolve teardown only from reproduced or concrete lifecycle evidence.
- Deliver revision-bound local and hosted PR verification.

### Non-Goals
- Adopting or merging this branch, updating the fork's default branch,
  migrating plugin APIs, changing thresholds or dependency pins.
- Hiding installed executables or replacing integration checks with mocks.

## Decisions

Keep runtime inspection in the test boundary, using the installed
runtime's supported inspection interface after verifying its contract.
Retain isolated HOME/XDG state and explicit assertions on agent name,
provider options and absence of policy metadata. Bound subprocess
execution and close any test-owned runtime resources before cleanup.
Do not add a production compatibility framework for this single test.

Extend Homebrew tests using t.TempDir and real missing inputs or invalid
destination layouts. Use existing injected I/O only when a portable
real failure cannot exercise the contract. No production refactoring
solely to increase coverage is planned.

Investigate scaffold lifecycle independently. A diagnostic rerun or
instrumentation must establish the responsible writer before changing
cleanup behavior. Do not add sleeps, retries or ignored cleanup errors
to disguise a live process or unjoined task.

## Coverage Strategy

Unit tests cover validation, read errors, and destination failure
contracts with explicit returned-error and on-disk state assertions.
Integration tests exercise the real installed OpenCode in isolated
state. Run all Go tests with -race -count=1 and a coverage profile;
enforce unchanged global and scope floors. Run lint, build, plugin
coverage and pipeline checks from the current CI workflow. Verify
hosted checks on the exact PR head; pending checks are not passes.

## Risks / Trade-offs

OpenCode inspection output can change across major versions. Select
the smallest verified interface and fail visibly for unsupported
execution rather than silently weakening assertions. Process ownership
and credentials must remain isolated from the user's service.

Filesystem permission failures differ for privileged users; prefer
structural path errors over permission-bit assumptions. Teardown may
remain non-reproducible; retain failed evidence and state that limit.

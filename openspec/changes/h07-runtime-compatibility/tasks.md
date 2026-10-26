## 1. Establish contracts

- [ ] 1.1 Validate and publish planning artifacts before source edits.
- [ ] 1.2 Confirm installed runtime inspection and isolation semantics.
- [ ] 1.3 Reproduce the reviewer failure on the unchanged test.

## 2. Implement bounded repairs

- [ ] 2.1 [P] Adapt reviewer inspection and add regression assertions
  retaining provider isolation, error propagation and bounded execution.
- [ ] 2.2 [P] Add Homebrew missing-input, temporary-file creation and
  destination-replacement failure tests with state assertions.
- [ ] 2.3 Diagnose scaffold teardown with concrete lifecycle evidence;
  repair only if established, otherwise retain the unresolved limit.

## 3. Verify and deliver

- [ ] 3.1 Run focused race tests and unchanged coverage floors.
- [ ] 3.2 Run required local CI checks with supported runtimes.
- [ ] 3.3 Verify alignment with all six constitution principles and
  assess CHANGELOG, README and AGENTS documentation impact.
- [ ] 3.4 Run review council and resolve blocking findings.
- [ ] 3.5 Create PR, inspect exact-head hosted checks and report
  verified readiness or precise remaining blockers without merging.
- [ ] 3.6 Publish scoped journey evidence separately in agent-config;
  keep adoption blocked until its required checks pass.

## Why

H-07 upstream adoption is blocked by an obsolete OpenCode agent-load
test, incomplete Homebrew failure-path coverage, and an intermittent
scaffold teardown failure. Supported Node resolves the earlier version
rejection but does not resolve these remaining checks.

## What Changes

- Verify reviewer provider-option isolation using supported OpenCode
  inspection without leaking user configuration or leaving services.
- Cover observable Homebrew input and filesystem failure outcomes.
- Diagnose the scaffold teardown failure and repair its lifecycle only
  if the cause is reproduced or established by concrete evidence.
- Preserve all existing quality thresholds and test assertions.

## Capabilities

### New Capabilities

- `runtime-qualification`: isolated runtime checks and failure-path
  evidence required for the H-07 repair candidate.

### Modified Capabilities

None. Public CLI behavior and artifact schemas remain unchanged.

### Removed Capabilities

None.

## Impact

Targets internal/schemas reviewer-load tests, internal/homebrew tests,
and any proven scaffold lifecycle defect. No dependency, governance,
global runtime, remote-tracking or primary-checkout change is planned.
Delivery is a reviewed PR, not a merge or adoption promotion.

## Constitution Alignment

### I. Autonomous Collaboration

**Assessment**: PASS. Existing artifact boundaries remain unchanged.

### II. Composability First

**Assessment**: PASS. No mandatory runtime dependency is added.

### III. Observable Quality

**Assessment**: PASS. Tests assert observable output and filesystem
state; retained results identify the checked revision and runtime.

### IV. Testability

**Assessment**: PASS. Unit cases use isolated temporary files and
existing injection points; installed-CLI integration stays isolated.

### V. Security by Default

**Assessment**: PASS. Runtime checks exclude real credentials and user
configuration. No dependency pins or access policies change.

### VI. Deterministic Execution

**Assessment**: PASS. Verification remains executable tests, not new
procedural agent instructions.

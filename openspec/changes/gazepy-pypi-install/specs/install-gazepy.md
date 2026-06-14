## ADDED Requirements

### Requirement: FR-001 — uv fallback install method

`installGazePy()` MUST support a `method: uv` explicit config option
that invokes `uv tool install gaze-py` when uv is available on PATH.

#### Scenario: Explicit uv method, uv present

- **GIVEN** `setup.tools.gazepy.method` is `"uv"` and `uv` is on PATH
- **WHEN** `uf setup` runs the GazePy install step
- **THEN** `uv tool install gaze-py` is executed and, on success, the
  step result has `action: "installed"` and `detail: "via uv"`

#### Scenario: Explicit uv method, uv absent

- **GIVEN** `setup.tools.gazepy.method` is `"uv"` and `uv` is NOT on PATH
- **WHEN** `uf setup` runs the GazePy install step
- **THEN** the step result has `action: "failed"` and a non-nil `err`
  indicating uv was not found

### Requirement: FR-002 — Homebrew-to-uv auto cascade

When `method: auto` (the default), `installGazePy()` MUST attempt
Homebrew first and, if Homebrew fails, MUST fall through to uv if uv
is available on PATH.

#### Scenario: Auto mode, Homebrew fails, uv present

- **GIVEN** `method` is `"auto"`, Homebrew is installed, `brew install`
  exits non-zero, and `uv` is on PATH
- **WHEN** `uf setup` runs the GazePy install step
- **THEN** `uv tool install gaze-py` is attempted; on success the step
  result has `action: "installed"` and `detail: "via uv"`

#### Scenario: Auto mode, neither Homebrew nor uv

- **GIVEN** `method` is `"auto"`, Homebrew is NOT installed, and `uv`
  is NOT on PATH
- **WHEN** `uf setup` runs the GazePy install step
- **THEN** the step result has `action: "skipped"`

### Requirement: FR-003 — Cascade diagnostic output

When `installGazePy()` falls through from Homebrew to uv in auto mode,
it MUST write a human-readable diagnostic line to `opts.Stdout` (when
non-nil) so operators understand why uv was selected.

#### Scenario: Diagnostic line emitted on Homebrew fallthrough

- **GIVEN** Homebrew install fails and uv is available
- **WHEN** the cascade falls through to uv
- **THEN** a line containing `"brew install failed"` and `"trying uv"`
  is written to `opts.Stdout` before the uv invocation

### Requirement: FR-004 — Error propagation from uv failure

When uv install fails in any code path (auto or explicit), the step
result's `err` field MUST be non-nil and MUST carry the original error
from `opts.ExecCmd`.

#### Scenario: uv install fails, err propagated

- **GIVEN** `uv tool install gaze-py` exits non-zero
- **WHEN** `installGazePy()` returns
- **THEN** `stepResult.err` is non-nil and wraps the original error

## MODIFIED Requirements

### Requirement: FR-005 — Config template method values

Previously: `setup.tools.gazepy.method` accepted `auto | homebrew | skip`.

The accepted values MUST now be `auto | homebrew | uv | skip`. The
`template.go` user-facing comment MUST reflect this expanded set.

## REMOVED Requirements

None.
<!-- scaffolded by uf vdev -->

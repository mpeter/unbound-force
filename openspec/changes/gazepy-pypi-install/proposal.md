# Change: gazepy-pypi-install

## Why

`gaze-py` published v0.2.0 to PyPI. The original `python-gaze-integration`
change implemented `installGazePy()` as Homebrew-only (matching `installGaze`)
with a skip-plus-hint fallback, because no PyPI artifact existed at the time.

With gaze-py on PyPI, `uv tool install gaze-py` now works. Since `uv` is
already installed by `uf setup` before `GazePy`, it is always available as
a fallback when the Homebrew tap formula has not yet been published.

## What Changes

### Modified Capabilities

- `installGazePy()`: Extends from Homebrew-only to Homebrew-first with uv
  fallback. In auto mode: try Homebrew → if Homebrew fails (e.g. tap formula
  not yet available), fall through to `uv tool install gaze-py`. Emits a
  diagnostic line to stdout when falling back so operators understand why uv
  was chosen over Homebrew.
- `method: uv` config option: New explicit install method `uv` added to the
  `setup.tools.gazepy` per-tool config override. Respects `opts.LookPath`
  for uv availability.
- `config/template.go`: `gazepy` method comment updated from
  `auto | homebrew | skip` to `auto | homebrew | uv | skip`.

### New Capabilities

None beyond the modified install behavior above.

### Removed Capabilities

None. Homebrew remains the preferred path in auto mode.

## Impact

- `internal/setup/setup.go`: `installGazePy()` rewritten.
- `internal/setup/setup_test.go`: Tests updated and extended.
- `internal/config/template.go`: Method comment updated.

## Constitution Alignment

Assessed against `.specify/memory/constitution.md` (v1.2.0).

### I. Autonomous Collaboration

**Assessment**: PASS. No inter-hero coupling changes.

### II. Composability First

**Assessment**: PASS. `installGazePy` degrades gracefully when neither
Homebrew nor uv is available. The uv prerequisite is always satisfied before
this step in `buildSteps()`.

### III. Observable Quality

**Assessment**: PASS. The fallback path now emits a diagnostic line.
The `err` field is correctly propagated in all failure paths.

### IV. Testability

**Assessment**: PASS. All new code paths covered by injected `LookPath`
and `ExecCmd`. Regression test added for the previously silent `err`-drop bug.

### V. Security by Default

**Assessment**: PASS. No new secrets surfaces. PyPI install is consistent
with existing `uv tool install` patterns (`specify-cli`).

### Cross-Repo Documentation Exemption

The constitution requires a `unbound-force/website` issue for user-facing
changes. This change updates the generated `opencode.yaml` config comment
from `auto | homebrew | skip` to `auto | homebrew | uv | skip`. This is
a generated file comment — it is not surfaced in CLI help text, command
output, or public documentation. The change is considered internal
tooling config; no website doc update is required.

**Tracking**: `mpeter/website#1` — filed to track doc update when the
Homebrew tap formula for gaze-py is published (at which point the full
`auto | homebrew | uv | skip` surface is documented end-to-end).

# Change: python-gaze-integration

## Why

`uf init` currently installs Gaze integration unconditionally for any
project that has the `gaze` binary in PATH. Gaze is a Go-specific test
quality tool — it analyzes Go ASTs, runs `go test`, and reports CRAP
and GazeCRAP scores. Running `gaze init` on a Python project produces
a Go-oriented agent file that references Go-specific commands and is
not useful for Python developers.

`gaze-py` (`gazepy`) is a Python-native port of Gaze. It performs
equivalent analysis on Python ASTs using `ast` (no code execution),
computes CRAP and GazeCRAP scores, and produces schema-compatible
output. It is distributed as a separate binary (`gazepy`) via the
Unbound Force Homebrew tap.

This change teaches `uf init` and `uf setup` to dispatch on project
language: Go projects get `gaze`, Python projects get `gazepy`.

## What Changes

### New Capabilities

- `gazepy init` dispatch: `uf init` runs `gazepy init` for Python
  projects when `gazepy` is in PATH, writing a Python-oriented
  `gaze-reporter.md` agent file.
- `installGazePy`: `uf setup` installs `gazepy` via
  `brew install unbound-force/tap/gazepy` on machines with Homebrew,
  mirroring the existing `installGaze` pattern.
- `gazepy` doctor check: `uf doctor` reports `gazepy` as a
  recommended tool, with severity configurable via `.uf/config.yaml`.
- `gazepy` config documentation: `.uf/config.yaml` template documents
  the `setup.tools.gazepy` override key.

### Modified Capabilities

- `simpleTool{}` struct: gains a `forLang` field (`string`). Empty
  string means language-agnostic (existing behavior preserved). A
  non-empty value means the tool only initializes when the detected
  project language matches.
- `simpleTools` loop: filters entries where `forLang != ""` and
  `forLang != lang`, skipping silently.

### Removed Capabilities

None.

## Impact

- `internal/scaffold/scaffold.go`: `simpleTool{}` struct + loop +
  new `gazepy` entry.
- `internal/setup/setup.go`: `installGazePy()` + `buildSteps` entry.
- `internal/doctor/checks.go`: `gazepy` recommended check entry.
- `internal/config/template.go`: `setup.tools.gazepy` documentation.

Existing `gaze` behavior is unchanged. The `gaze` entry in
`simpleTools` receives no `forLang` value and continues to run for
any project that has `gaze` in PATH. Backwards compatibility is
fully preserved.

## Documentation Obligation

This change affects user-facing CLI behavior (`uf setup`, `uf doctor`,
`uf init`). Per the constitution Development Workflow section, a
GitHub issue MUST be created in `unbound-force/website` to track
documentation updates before the implementing PR is merged. The
issue should cover: new `gazepy` entry in the tools reference,
updated `uf setup` command documentation, and updated `uf doctor`
output description.

Companion change in `gaze-py` repo (implements `gazepy init` and
the agent template): track issue reference here before PR merge.

## Constraints

- Single-language repos only. Go and Python are mutually exclusive
  primary languages within a single repository. This constraint is
  already implicit in `detectLang()` (returns one language string)
  and the convention pack system. This change makes it explicit.
- `uf setup` install of `gazepy` is gated on Homebrew availability,
  identical to the `gaze` pattern. PyPI publication is out of scope.
- `gazepy init` implementation (the `init` subcommand and agent
  template content) is owned by the `gaze-py` repo and handled in a
  companion change there. This change only calls `gazepy init` — it
  does not define what it produces.
- The shared sentinel (`.opencode/agents/gaze-reporter.md`) means
  first-writer-wins on projects where both `gaze` and `gazepy` are
  in PATH. This is an accepted edge case under the single-language
  constraint.

## Constitution Alignment

Assessed against `.specify/memory/constitution.md` (v1.2.0).

### I. Autonomous Collaboration

**Assessment**: PASS

Each tool (`gaze`, `gazepy`) writes its own integration artifact
(`gaze-reporter.md`) independently. `uf init` orchestrates via
`LookPath` and `forLang` filtering — no runtime coupling between
tools. If `gazepy` is absent, init skips silently and the project
remains fully functional with whatever tools are present.

### II. Composability First

**Assessment**: PASS

`gazepy` is independently installable and usable without any other
hero present. The `forLang` dispatch is additive — it does not
remove or gate any existing functionality. The single-language
constraint is already embodied in `detectLang()` and does not
introduce new mandatory dependencies between heroes.

### III. Observable Quality

**Assessment**: PASS

`uf setup` and `uf init` both produce structured output (via
`subToolResult` and `stepResult`) that is machine-parseable. The
`gazepy` entries follow the same output contract as all existing
tool entries. No silent failures are introduced.

### IV. Testability

**Assessment**: PASS

`installGazePy` follows the `installGaze` pattern exactly: all
external calls go through injected `LookPath` and `ExecCmd`
fields on `Options`, making the function fully testable in
isolation. The `forLang` filter in the `simpleTools` loop is a
pure string comparison with no external dependencies.

### V. Security by Default

**Assessment**: PASS

Supply chain integrity is provided by Homebrew's native mechanism.
Homebrew Formula `virtualenv_install_with_resources` requires a
`sha256` content hash for every `resource` block — `brew audit`
refuses a Formula without them. SHA256 verification happens at
install time, before any code is executed. This applies to both
`click` and `pyyaml` dependencies.

The `unbound-force/homebrew-tap` has no tap-level CI workflow,
which is consistent with the existing Cask entries. Homebrew's
built-in content hash verification is the supply chain control.

No new runtime input surfaces are introduced: `forLang` is a
compile-time struct field, and the `brew install` invocation uses
a hardcoded package name with no user-controlled interpolation.

# Design: python-gaze-integration

## Context

`uf init` runs a set of "simple tools" concurrently after the main
scaffold step. Each tool is looked up in PATH; if found, `<tool> init`
is called. The `simpleTools` list in `scaffold.go` is currently
language-agnostic — every tool runs for every project.

`uf setup` installs tools via an ordered `stepDef` slice in
`setup.go`. Gaze is installed via Homebrew (`unbound-force/tap/gaze`).
The Homebrew tap (`unbound-force/homebrew-tap`) has infrastructure to
support both Casks (compiled binaries) and Formulae (Python virtualenv
tools). `gazepy` will be distributed as a Formula.

Language detection is already implemented in `detectLang()`. It
returns a single string (`"go"`, `"python"`, `"typescript"`, etc.) or
`"default"`. This string is available as `lang` throughout the `uf
init` execution path.

## Goals / Non-Goals

### Goals

- Python projects get `gazepy init` instead of (or in addition to)
  `gaze init` when `gazepy` is in PATH.
- `uf setup` installs `gazepy` via Homebrew on developer machines.
- `uf doctor` surfaces `gazepy` as a recommended tool.
- Backwards compatibility: existing `gaze` behavior is unchanged.

### Non-Goals

- Polyglot (Go + Python) repo support. Single-language per repo only.
- `gazepy` PyPI publication. Homebrew is the distribution channel.
- `gazepy init` implementation. Owned by the `gaze-py` repo.
- macOS notarization or code signing for `gazepy`. Python wheels
  distributed via Homebrew Formula do not require signing.

## Decisions

### D1 — `forLang` field on `simpleTool{}`

Add a `forLang string` field to the `simpleTool` struct in
`scaffold.go`. The simpleTools loop gains one guard:

```go
if tool.forLang != "" && tool.forLang != lang {
    continue
}
```

Empty string preserves existing language-agnostic behavior. The
existing `gaze` entry gets no `forLang` value — it continues to run
for any project with `gaze` in PATH (backwards compatibility). The
new `gazepy` entry gets `forLang: "python"`.

**Rationale**: Minimal struct change, zero new mechanism. The `lang`
string is already computed before the simpleTools loop runs.

### D2 — Shared sentinel (`.opencode/agents/gaze-reporter.md`)

Both `gaze init` and `gazepy init` write to the same sentinel path.
First-writer-wins. On a project where both binaries are in PATH
(possible on a developer's machine), the tool that wins the race
writes the agent file and the other skips.

This is an accepted edge case under the single-language constraint.
A Python developer who also has `gaze` installed may get a Go agent
file if `gaze` wins. The fix is to ensure `gazepy` is in PATH and
`gaze` is not, or to run `gazepy init --force` manually.

**Rationale**: No changes to sentinel logic. Complexity of per-tool
sentinel files or content-based detection is not justified for this
edge case.

### D3 — `installGazePy` mirrors `installGaze` exactly

`installGazePy` follows the same structure as `installGaze`:

1. `LookPath("gazepy")` — skip if already installed.
2. `toolMethod("gazepy")` — respect config override.
3. Homebrew primary: `brew install unbound-force/tap/gazepy`.
4. Skip with download hint if Homebrew unavailable.

No RPM/DNF path for this iteration. The Homebrew Formula handles
macOS and Linux. RPM support can follow the same pattern as `gaze`
in a future change once demand is established.

**Rationale**: Pattern consistency. The `installGaze` function is
explicitly documented as the pattern to follow for new tool installs
in `setup.go` comments.

### D4 — `gazepy` entry in `buildSteps` — no gate

The `gazepy` step in `buildSteps` has no `gate`. `uv` availability
is not required to install `gazepy` via Homebrew — Homebrew manages
its own Python interpreter. Adding a `uv` gate would incorrectly
block installation on machines that use Homebrew as their primary
Python tool manager.

**Rationale**: Homebrew Formula install is self-contained. The `uv`
gate was considered but rejected because it reflects the wrong
dependency.

### D5 — Single-language constraint is documented, not enforced

`uf init` does not error or warn when both `go.mod` and
`pyproject.toml` exist. `detectLang()` returns the first match —
whichever indicator appears first in its check list. The constraint
is captured in documentation and the proposal, not enforced at
runtime.

**Rationale**: Enforcement adds complexity and may surprise users in
monorepos with auxiliary scripts. The convention is sufficient for
the UF org context.

## Risks / Trade-offs

### R1 — Race condition on polyglot machines (D2)

If a developer has both `gaze` and `gazepy` in PATH on a Python
project, `gaze` may win the concurrent init race and write a Go
agent file. **Accepted**. The single-language constraint makes
this rare. The developer can resolve by running `gazepy init
--force` manually.

### R2 — No RPM path for `gazepy` (D3)

Linux users on Fedora/RHEL without Homebrew get a skip with a
download link. **Accepted** for this iteration. The RPM pattern
is established and can be added when demand is confirmed.

### R3 — Homebrew Formula for Python (new pattern in tap)

The existing tap uses Casks for compiled Go binaries. Adding a
Formula for a Python tool is a different Homebrew mechanism
(`virtualenv_install_with_resources`). **Mitigated**: Homebrew
Formulae for Python tools are well-established and documented.
The `click` and `pyyaml` dependencies are minimal.

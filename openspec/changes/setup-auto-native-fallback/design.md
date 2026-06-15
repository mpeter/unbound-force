## Context

The `linux-cross-platform-install` change (merged) added
`installViaRpm()` at setup.go:852, `ManagerDnf` detection
in environ.go:103, and `toolMethod()` dispatch in
`installGaze()` at setup.go:525. It explicitly deferred
wiring sibling tool install functions to use this
infrastructure in auto mode.

Issue #214 reports the user-visible consequence: on
Fedora/RHEL without Homebrew, `uf setup` skips Gaze,
Replicator, Dewey, and GitHub CLI with download-link
hints.

The `go install` pattern is already established in the
codebase: `installGolangciLint()` (setup.go:770) and
`installGovulncheck()` (setup.go:802) both use
`opts.ExecCmd("go", "install", ...)`.

## Goals / Non-Goals

### Goals
- Auto mode installs Gaze, Replicator, and Dewey on
  Fedora/RHEL via dnf or `go install`
- Auto mode installs GitHub CLI on Fedora/RHEL via dnf
- `UF_PACKAGE_MANAGER=dnf` forces the dnf install path
- All four install functions have `toolMethod()` dispatch
  for per-tool config overrides
- All new code is testable via injectable dependencies
  (Constitution Principle IV — Testability)

### Non-Goals
- DEB/apt package support (future increment)
- Windows package manager support (winget, choco)
- Ollama/Podman/DevPod fallback (different install
  contracts, no GoReleaser RPMs)
- Adding RPM generation to sibling repos (separate
  concern per repo)

## Decisions

### D1: resolveMethod() helper

Add a `resolveMethod()` method on `*Options` that takes
`toolName` and `DetectedEnvironment`, returning the
resolved install method string. Logic:

1. If `toolMethod(toolName)` returns a non-`"auto"` value,
   use that (per-tool override takes precedence).
2. If `opts.PackageManager` is `"homebrew"`, `"dnf"`, or
   `"apt"`, return that (global preference).
3. If `opts.PackageManager` is `"auto"` (default):
   return `"auto"` (let the install function apply its
   own fallback chain using environment detection).

This keeps the resolution logic centralized while
preserving the per-function fallback chains that differ
by tool (e.g., Dewey has no RPM path).

**Rationale**: A single resolution point avoids
duplicating the precedence logic across 4+ install
functions. The `"auto"` passthrough preserves the
existing fallback-chain pattern, which varies by tool.

**Constitution**: Composability First — each tool's
install function retains its own fallback chain, so tools
with different distribution channels (e.g., Dewey without
RPMs) compose correctly without special-casing in the
resolver.

### D2: Auto-mode fallback order

Each install function applies fallbacks in order:

| Priority | Method | When used |
|----------|--------|-----------|
| 1 | Homebrew | `HasManager(env, ManagerHomebrew)` |
| 2 | dnf (RPM) | `HasManager(env, ManagerDnf)` and tool has RPMs |
| 3 | `go install` | `LookPath("go")` succeeds and tool is a Go binary |
| 4 | skip | Last resort, with download link |

Per-tool variations:

- **Gaze**: Homebrew -> dnf -> `go install` -> skip
- **Replicator**: Homebrew -> dnf -> `go install` -> skip
- **Dewey**: Homebrew -> `go install` -> skip (no RPMs)
- **GH CLI**: Homebrew -> dnf -> skip (not a Go binary
  in the same sense; official dnf repo is the native
  path)

**Rationale**: Homebrew first because it provides the
most consistent experience across platforms. dnf second
because it uses pre-built release binaries. `go install`
third because it builds from source (slower, different
binary than release artifacts). Skip as last resort.

### D3: installViaGo() helper

Add a helper function:

```go
func installViaGo(
    opts *Options,
    toolName, goModule string,
) stepResult
```

Follows the `installGolangciLint()` pattern: calls
`opts.ExecCmd("go", "install", goModule+"@latest")`.
Returns a `stepResult` with `detail: "via go install"`.

**Go module paths**:
- Gaze: `github.com/unbound-force/gaze/cmd/gaze`
- Dewey: `github.com/unbound-force/dewey`
- Replicator: `github.com/unbound-force/replicator/cmd/replicator`

**Rationale**: Extracts the repeated `go install` pattern
into a reusable helper, same as `installViaRpm()`.

**Constitution**: Testability — uses `opts.ExecCmd` and
`opts.LookPath` injection, testable without network
access.

### D4: GH CLI dnf installation

GitHub CLI publishes its own dnf repository. The
install path uses `dnf install gh` (no URL construction
needed). This differs from the GoReleaser RPM pattern
used by `installViaRpm()`.

The dnf repo may not be pre-configured on all systems.
If `dnf install gh` fails, fall through to skip with the
existing download link.

**Rationale**: `gh` is not built with GoReleaser and does
not follow the same RPM URL pattern. Using `dnf install`
directly is simpler and matches GitHub's official install
instructions for Fedora.

### D5: PackageManager semantics

| Value | Behavior |
|-------|----------|
| `"auto"` | Detect and try fallback chain (default) |
| `"homebrew"` | Homebrew only, skip if absent |
| `"dnf"` | dnf/RPM only, `go install` fallback for tools without RPMs |
| `"apt"` | Reserved for future use, behaves as `"auto"` |
| `"manual"` | Skip all tools (existing behavior) |

When `PackageManager` is explicitly set to `"dnf"`, the
install functions skip the Homebrew attempt and go
directly to the dnf path. For Dewey (no RPMs), they
fall through to `go install`.

**Rationale**: Users who set `UF_PACKAGE_MANAGER=dnf`
have explicitly chosen their preference. Homebrew should
not be attempted when the user has opted out.

## Risks / Trade-offs

### R1: `go install` builds from source

`go install` produces a binary built from source at
`@latest`, which may differ from the tested release
binary. Mitigation: this is tier 3 — it only activates
when both Homebrew and dnf are unavailable. The binary
is functionally equivalent; only the build provenance
differs.

### R2: GH CLI dnf repo not pre-configured

`dnf install gh` will fail if the GitHub CLI repository
is not configured. Mitigation: fall through to skip with
a download link. The user can configure the repo
manually and re-run `uf setup`.

### R3: `go install` requires Go toolchain

`go install` requires `go` in PATH. Mitigation: Go is
already a prerequisite for `unbound-force` development.
If `go` is not available, the fallback gracefully
continues to skip. The `LookPath("go")` check prevents
confusing error messages.

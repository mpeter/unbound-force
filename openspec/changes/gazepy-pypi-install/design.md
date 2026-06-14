# Design: gazepy-pypi-install

## Context

`installGazePy()` in `internal/setup/setup.go` was implemented as
Homebrew-only with a skip-plus-hint fallback. This matched `installGaze`
(the Go analysis tool) because at the time, `gaze-py` had no PyPI
artifact. With v0.2.0 now published to PyPI, `uv tool install gaze-py`
is a viable install path. `uv` is already installed earlier in the
`buildSteps()` sequence, making it a reliable fallback.

## Goals / Non-Goals

### Goals

- Extend `installGazePy()` to cascade: Homebrew first → uv fallback
- Emit a diagnostic stdout line when the cascade falls through to uv
- Add `method: uv` as an explicit per-tool config override
- Propagate `error` correctly from all failure paths (fixes silent
  err-drop bug in auto-path uv failure)
- Update `config/template.go` method comment to reflect new option
- Cover all new code paths with injected-stub unit tests

### Non-Goals

- Publishing a Homebrew formula for `gaze-py` (separate tap work)
- Version pinning for `uv tool install gaze-py` (consistent with
  existing `@latest` pattern used for `specify-cli`)
- Adding `gaze-py` to the doctor required-tools list (already done
  in `python-gaze-integration`)

## Decisions

### D-1: Cascade order — Homebrew first, uv second

Homebrew is the canonical distribution channel for the unbound-force
toolchain. If a tap formula exists, it is preferred (reproducible,
versioned, offline-capable after first install). uv is the fallback
for the window between PyPI publication and tap formula availability.
This matches the precedent set by `installGaze` (Homebrew-first) and
avoids privileging PyPI over the tap.

### D-2: Diagnostic stdout line on cascade

When Homebrew fails and the cascade falls through to uv, a line is
written to `opts.Stdout` (nil-guarded for test compatibility):

```
  GazePy: brew install failed (tap formula not yet available), trying uv...
```

This satisfies the Observable Quality principle: operators must be
able to understand why a tool was installed via uv rather than
Homebrew without reading source code.

### D-3: err propagation fix — restructure if-block

The original auto-path uv block used:

```go
if _, err := opts.ExecCmd(...); err == nil { ... }
return stepResult{..., err: nil}  // bug: err scoped inside if
```

The fix captures `installErr` as a named variable before the branch,
so the failure return can set `err: installErr`. This is a correctness
fix, not a behavioral change for the success path.

### D-4: nil-guard on opts.Stdout

`opts.Stdout` is nil in unit tests that do not set it. The diagnostic
`fmt.Fprintf` is wrapped in `if opts.Stdout != nil { ... }` to avoid
a nil pointer dereference in tests. This matches the pattern used
elsewhere in `setup.go`.

### D-5: Dry-run wording for no-toolchain case

When both Homebrew and uv are unavailable and `DryRun` is true, the
detail string is corrected from the contradictory
`"Would install: uv tool install gaze-py (uv not found)"` to
`"Would skip: uv and Homebrew not available. Install uv: https://docs.astral.sh/uv/"`.

## Risks / Trade-offs

| Risk | Likelihood | Mitigation |
|------|-----------|------------|
| PyPI package name collision (`gaze-py`) | Low | Package is published by the unbound-force org; verified at v0.2.0 |
| uv not on PATH even though installed earlier | Very low | GazePy is placed after the uv step in `buildSteps()` (position 9 vs uv at position 7); LookPath check provides a runtime gate so the graceful skip path handles any remaining edge cases |
| Homebrew tap formula published later, users stay on uv | Low | Users on `method: auto` will pick up Homebrew on next `uf setup` run once formula lands |
| Test isolation: `opts.Stdout` nil panic | Resolved | nil-guard applied (D-4) |
| uv fallback path outlives its usefulness | Low | Future simplification: once the Homebrew tap formula for `gaze-py` is published and stable, the auto-cascade to uv can be removed in a follow-up change |
<!-- scaffolded by uf vdev -->

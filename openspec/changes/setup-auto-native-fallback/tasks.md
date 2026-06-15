<!--
  [P] marks tasks eligible for parallel execution.
  Add [P] when a task: (a) touches different files from
  other [P] tasks in the group, (b) has no dependency
  on prior tasks in the group, (c) can safely execute
  without ordering constraints.
  Do NOT add [P] when tasks modify the same file —
  parallel workers will cause merge conflicts.
  Tasks without [P] run sequentially first, then [P]
  tasks run in parallel.

  All tasks modify internal/setup/setup.go and
  internal/setup/setup_test.go. No [P] markers —
  all tasks are sequential to avoid merge conflicts
  in the same two files.
-->

## 1. Add resolveMethod() and installViaGo() Helpers

- [ ] 1.1 Add `resolveMethod(toolName string,
  env doctor.DetectedEnvironment) string` method on
  `*Options` in `internal/setup/setup.go`. Logic:
  (1) if `toolMethod(toolName)` returns non-`"auto"`,
  return that value (per-tool override wins);
  (2) if `opts.PackageManager` is `"homebrew"`, `"dnf"`,
  or `"apt"`, return that (global preference);
  (3) otherwise return `"auto"` for fallback chain.
- [ ] 1.2 Add `installViaGo(opts *Options, toolName,
  goModule string) stepResult` function in
  `internal/setup/setup.go`. Check `LookPath("go")`
  first — if Go is not available, return a skip result
  with install hint. Otherwise call
  `ExecCmd("go", "install", goModule+"@latest")` and
  return `stepResult` with `detail: "via go install"`.
  Handle dry-run mode.
- [ ] 1.3 Add tests for `resolveMethod()` in
  `internal/setup/setup_test.go`: per-tool override
  takes precedence over global, global `PackageManager`
  takes precedence over `"auto"`, `"auto"` passes
  through, `"manual"` is handled by `shouldSkipTool`
  (not `resolveMethod`).
- [ ] 1.4 Add tests for `installViaGo()` in
  `internal/setup/setup_test.go`: Go available and
  install succeeds, Go available but install fails,
  Go not available returns skip result, dry-run mode
  returns correct detail string.

## 2. Update installGaze() Auto-Mode Fallback

- [ ] 2.1 Update `installGaze()` (setup.go:519) to call
  `resolveMethod("gaze", env)` instead of
  `opts.toolMethod("gaze")`. Add dispatch cases for
  `"go"` (calls `installViaGo` with module
  `github.com/unbound-force/gaze/cmd/gaze`).
  Update auto-mode block: after Homebrew-absent check,
  try dnf via `installViaRpm()` when
  `HasManager(env, ManagerDnf)`, then try `installViaGo`
  as final fallback before skip. Update dry-run to
  reflect new fallback chain.
- [ ] 2.2 Add tests for `installGaze()` fallback chain:
  dnf available + no Homebrew -> installs via RPM;
  no Homebrew + no dnf + Go available -> installs via
  `go install`; no Homebrew + no dnf + no Go -> skips;
  `PackageManager: "dnf"` -> skips Homebrew, uses dnf.

## 3. Update installReplicator() with Dispatch + Fallback

- [ ] 3.1 Add `resolveMethod("replicator", env)` dispatch
  to `installReplicator()` (setup.go:721) with explicit
  `"rpm"`/`"dnf"` case (calls `installViaRpm` with repo
  `unbound-force/replicator`), `"homebrew"` case, and
  `"go"` case (calls `installViaGo` with module
  `github.com/unbound-force/replicator/cmd/replicator`).
  Follow the `installGaze()` dispatch pattern at
  setup.go:525-538.
- [ ] 3.2 Add auto-mode fallback chain: Homebrew -> dnf
  (via `installViaRpm`) -> `go install` (via
  `installViaGo`) -> skip. Update dry-run to reflect
  new fallback chain.
- [ ] 3.3 Add tests: same matrix as 2.2 for Replicator.

## 4. Update installDewey() with Dispatch + Fallback

- [ ] 4.1 Add `resolveMethod("dewey", env)` dispatch to
  `installDewey()` (setup.go:1128) with explicit
  `"homebrew"` and `"go"` cases. No `"rpm"`/`"dnf"`
  case — Dewey does not publish RPMs. If
  `resolveMethod` returns `"dnf"`, fall through to
  `"go"` since dnf is not available for Dewey.
- [ ] 4.2 Add auto-mode fallback chain: Homebrew ->
  `go install` (via `installViaGo` with module
  `github.com/unbound-force/dewey`) -> skip. Preserve
  the `pullEmbeddingModel()` call after any successful
  install path.
- [ ] 4.3 Add tests: no Homebrew + Go available ->
  installs via `go install` + pulls embedding model;
  no Homebrew + no Go -> skips; `PackageManager: "dnf"`
  with no dnf method available -> falls through to
  `go install`.

## 5. Update installGH() with Dispatch + Fallback

- [ ] 5.1 Add `resolveMethod("gh", env)` dispatch to
  `installGH()` (setup.go:470) with explicit `"dnf"`
  case (calls `ExecCmd("dnf", "install", "-y", "gh")`
  directly — GH CLI has its own dnf repo, not
  GoReleaser RPMs) and `"homebrew"` case.
- [ ] 5.2 Add auto-mode fallback chain: Homebrew -> dnf
  (`dnf install -y gh`) -> skip. If `dnf install gh`
  fails, fall through to skip with download link.
  Update dry-run to reflect new fallback chain.
- [ ] 5.3 Add tests: dnf available + no Homebrew ->
  attempts `dnf install gh`; dnf install fails ->
  skips with download link; `PackageManager: "dnf"` ->
  skips Homebrew, uses dnf directly.

## 6. Dry-Run Path Updates

- [ ] 6.1 Verify dry-run output in all four updated
  install functions reflects the new fallback chain.
  When Homebrew is absent, dry-run should show the
  next available method (dnf or `go install`) instead
  of "Would install: download from...".
- [ ] 6.2 Add dry-run tests: verify each fallback tier
  produces the correct dry-run detail string for all
  four tools.

## 7. Verification

- [ ] 7.1 Run `make check` — all tests pass, lint clean,
  build succeeds.
- [ ] 7.2 Run `go test -race -count=1 ./internal/setup/`
  — verify new tests pass in isolation.
- [ ] 7.3 Manual smoke test: `uf setup --dry-run` on a
  system without Homebrew — verify tools show dnf or
  `go install` intent instead of "skipped".

## 8. Constitution Alignment Verification

- [ ] 8.1 Verify Composability First: each tool's
  fallback chain degrades gracefully — no tool requires
  a specific package manager. Standalone installation
  is preserved via `go install` as the universal
  fallback for Go-based tools.
- [ ] 8.2 Verify Testability: all new code uses
  injectable dependencies (`LookPath`, `ExecCmd` on
  the Options struct). No tests require network access,
  external services, or shared mutable state. All tests
  verify observable side effects (returned `stepResult`
  values).

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
-->

## 1. Fix installGazePy() in setup.go

- [x] 1.1 Restructure auto-path uv block to capture `installErr`
      as a named variable before the branch, so the failure return
      sets `err: installErr` (fixes silent err-drop bug, D-3)
- [x] 1.2 Add Homebrew-to-uv fallthrough diagnostic:
      `fmt.Fprintf(opts.Stdout, ...)` wrapped in nil-guard (D-2, D-4)
- [x] 1.3 Add `method: uv` explicit path that checks `LookPath("uv")`
      and runs `uv tool install gaze-py` (FR-001)
- [x] 1.4 Fix dry-run wording for no-toolchain case (D-5)

## 2. Update config template

- [x] 2.1 [P] Update `internal/config/template.go` method comment from
      `auto | homebrew | skip` to `auto | homebrew | uv | skip` (FR-005)

## 3. Add and update tests in setup_test.go

- [x] 3.1 Add `TestInstallGazePy_AutoPath_UVFails` — auto mode, uv
      present, uv install fails; asserts `err != nil` (regression
      for the err-drop bug)
- [x] 3.2 [P] Add `TestInstallGazePy_ExplicitUVMethodDryRun` — explicit
      `method: uv`, `DryRun: true`; asserts `action: "dry-run"` and
      detail contains `"uv tool install gaze-py"`
- [x] 3.3 [P] Verify `TestInstallGazePy_HomebrewFailsFallsBackToUV`
      passes with nil-guarded stdout (no panic)
- [x] 3.4 [P] Verify `TestInstallGazePy_NoHomebrewSkip` assertion still
      holds with updated code paths
- [x] 3.5 [P] Add `TestInstallGazePy_ExplicitUV_UVPresent` — explicit
      `method: uv`, uv on PATH, `DryRun: false`; asserts
      `action: "installed"` and `detail: "via uv"` (FR-001 scenario 1)
- [x] 3.6 [P] Add `TestInstallGazePy_ExplicitUV_UVAbsent` — explicit
      `method: uv`, uv NOT on PATH (LookPath returns not-found);
      asserts `action: "failed"` and `err` non-nil (FR-001 scenario 2)

## 4. Verification

- [x] 4.1 Run `go build ./...` — must pass with no errors
- [x] 4.2 Run `go test -race -count=1 ./internal/setup/...
      ./internal/config/...` — all tests must pass
- [x] 4.3 Run `go vet ./...` — no findings
- [x] 4.4 Verify constitution alignment: all five principles assessed
      in proposal.md; no new inter-hero coupling introduced; all
      new code paths covered by injected-stub tests (no external
      service required)

## 5. Branch and PR

- [x] 5.0 Post-merge: file `docs` issue in `unbound-force/website` when
      Homebrew tap formula for gaze-py is published, documenting the
      full `method: auto | homebrew | uv | skip` config surface.
      (Constitution cross-repo doc tracking; exempted pre-merge per
      proposal.md Constitution Alignment section.)
- [ ] 5.1 Stage only: `internal/setup/setup.go`,
      `internal/setup/setup_test.go`, `internal/config/template.go`,
      `openspec/changes/gazepy-pypi-install/`
- [ ] 5.2 Commit with message:
      `feat(setup): add uv fallback for gaze-py install`
- [ ] 5.3 Push branch `opsx/gazepy-pypi-install` and open PR to fork main
<!-- scaffolded by uf vdev -->

<!-- spec-review: passed -->

<!-- code-review: passed -->

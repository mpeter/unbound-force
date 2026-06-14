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

## 1. Scaffold: language-conditional dispatch

- [x] 1.1 Add `forLang string` field to `simpleTool{}` struct in
  `internal/scaffold/scaffold.go`. Field comment: "forLang, if
  non-empty, restricts this tool to projects whose detected language
  matches. Empty means language-agnostic."
- [x] 1.2 Add `forLang` guard to the `simpleTools` loop:
  `if tool.forLang != "" && tool.forLang != lang { continue }`.
  Place immediately after the `shouldSkipTool` check.
- [x] 1.3 Add `gazepy` entry to `simpleTools` slice immediately after
  the existing `gaze` entry. The `simpleTool` struct fields in order
  are: cmd (binary name), sentinel (agent file path), label (display
  name for step output), initSubTools (nil or sub-tool slice), and
  forLang (language filter). Use named struct initialization to avoid
  positional ambiguity:
  ```go
  {cmd: "gazepy",
   sentinel: filepath.Join(".opencode", "agents", "gaze-reporter.md"),
   label:    "Gaze integration (Python)",
   forLang:  "python"}
  ```
  Verify the field names against the current `simpleTool` struct
  definition in `scaffold.go` before committing.
- [x] 1.4 Update the existing `gaze` entry in `simpleTools` to use
  named struct initialization (if it currently uses positional syntax)
  so that the new `forLang` field compiles cleanly with its zero value.
  Update any test fixtures in `scaffold_test.go` that construct
  `simpleTool{}` literals positionally.

## 2. Setup: install gazepy

- [x] 2.1 Write `installGazePy` function in
  `internal/setup/setup.go` following the `installGaze` pattern
  exactly. Binary name: `"gazepy"`. Homebrew package:
  `"unbound-force/tap/gazepy"`. Config key: `"gazepy"`. Display
  name: `"GazePy"`. No RPM path for this iteration. Download
  fallback hint: `https://github.com/unbound-force/gaze-py/releases`.
- [x] 2.2 Add `{name: "GazePy", tool: "gazepy", install: installGazePy}`
  entry to `buildSteps()` in `setup.go`. Place immediately after
  the existing `{name: "Gaze", ...}` entry. No gate required
  (see design D4).

## 3. Doctor and config (parallel group)

- [x] 3.1 [P] Add `gazepy` check entry to the checks slice in
  `internal/doctor/checks.go`: `{name: "gazepy", recommended: true}`.
  Place after the existing `gaze` entry. Add `"gazepy"` to the
  `displayNames` map in `internal/doctor/environ.go` as
  `"GazePy (Tester)"`. (If the map lives in a different file,
  `grep -r "displayNames" internal/doctor/` will locate it.)
- [x] 3.2 [P] Add `gazepy` to the `setup.tools` section of the
  config template in `internal/config/template.go`. Add a commented
  entry: `#     gazepy:\n#       method: auto             # auto | homebrew | skip`.
  Place after the existing `gaze` entry.

## 4. Tests

- [x] 4.1 [P] Write test for `forLang` filtering in
  `internal/scaffold/scaffold_test.go`. Verify: (a) a tool with
  `forLang: "python"` is skipped when `lang == "go"`, (b) a tool
  with `forLang: "python"` runs when `lang == "python"`, (c) a
  tool with empty `forLang` runs regardless of lang.
- [x] 4.2 [P] Write test for `installGazePy` in
  `internal/setup/setup_test.go` following the `installGaze` test
  pattern. Cover: already installed, Homebrew install success,
  Homebrew install failure, Homebrew unavailable (skip + hint),
  dry-run, config method `skip`.
- [x] 4.3 [P] Write test for the `gazepy` doctor check in
  `internal/doctor/checks_test.go` (or the equivalent test file for
  `checks.go`). Verify: (a) `gazepy` missing with default severity
  reports as `recommended`, (b) severity override to `required` via
  `.uf/config.yaml` is reflected in the check result. Follow the
  existing `gaze` check test as the reference pattern.
- [x] 4.4 [P] Verify config template drift detection: confirm that
  the existing drift detection test in `internal/config/` covers
  the `setup.tools.gazepy` entry added in task 3.2, or add an
  explicit assertion that the template contains the expected
  commented `gazepy` block.

## 5. Verification

- [x] 5.1 Run `go test -race -count=1 ./...` — all tests pass.
- [x] 5.2 Run `go vet ./...` and `golangci-lint run` — no issues.
- [x] 5.3 Run `go build ./...` — clean build.
- [x] 5.4 Manual smoke test: create a temp Python project directory
  with `pyproject.toml`, run `uf init --dry-run`, confirm `gazepy`
  init step appears and `gaze` init step does not appear (assuming
  `gazepy` is in PATH and `gaze` is not).
- [x] 5.5 Coverage target: `internal/scaffold`, `internal/setup`, and
  `internal/doctor` packages MUST maintain ≥85% line coverage on
  modified files. Verify via `go test -race -count=1 -coverprofile=coverage.out ./internal/scaffold/... ./internal/setup/... ./internal/doctor/...`
  followed by `go tool cover -func=coverage.out`.

<!-- spec-review: passed -->

<!-- code-review: passed -->

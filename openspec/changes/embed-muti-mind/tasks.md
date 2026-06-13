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

## 1. Fix `go run` references in source files

- [ ] 1.1 [P] Fix `.opencode/agents/muti-mind-po.md`: replace all
  `go run cmd/mutimind/main.go` occurrences with `mutimind`; remove
  `(or go run cmd/mutimind/main.go ...)` parentheticals
- [ ] 1.2 [P] Fix `.opencode/commands/muti-mind.init.md`: replace
  `go run cmd/mutimind/main.go init` with `mutimind init`; remove the
  conditional "if the project has a `cmd/mutimind/main.go` file" guard
- [ ] 1.3 [P] Fix `.opencode/commands/muti-mind.backlog-add.md`
- [ ] 1.4 [P] Fix `.opencode/commands/muti-mind.backlog-list.md`
- [ ] 1.5 [P] Fix `.opencode/commands/muti-mind.backlog-show.md`
- [ ] 1.6 [P] Fix `.opencode/commands/muti-mind.backlog-update.md`
- [ ] 1.7 [P] Fix `.opencode/commands/muti-mind.generate-stories.md`:
  replace `go run cmd/mutimind/main.go add` with `mutimind add`; update
  surrounding prose
- [ ] 1.8 [P] Fix `.opencode/commands/muti-mind.prioritize.md`: remove
  `(or go run cmd/mutimind/main.go update ...)` parenthetical; keep
  `mutimind update` as the sole form
- [ ] 1.9 [P] Fix `.opencode/commands/muti-mind.sync.md`
- [ ] 1.10 [P] Fix `.opencode/commands/muti-mind.sync-project.md`
- [ ] 1.11 [P] Fix `.opencode/commands/muti-mind.sync-pull.md`
- [ ] 1.12 [P] Fix `.opencode/commands/muti-mind.sync-push.md`
- [ ] 1.13 [P] Fix `.opencode/commands/muti-mind.sync-status.md`

## 2. Embed files in scaffold assets

- [ ] 2.1 Copy cleaned `.opencode/agents/muti-mind-po.md` to
  `internal/scaffold/assets/opencode/agents/muti-mind-po.md`
- [ ] 2.2 Copy all 13 cleaned command files to
  `internal/scaffold/assets/opencode/commands/muti-mind.*.md`

## 3. Update test manifest

- [ ] 3.1 In `internal/scaffold/scaffold_test.go`, remove the 13
  `muti-mind.*.md` command entries and the 1 `muti-mind-po.md` agent
  entry from `knownNonEmbeddedFiles`
- [ ] 3.2 In `internal/scaffold/scaffold_test.go`, add all 14 paths
  (1 agent + 13 commands) to `expectedAssetPaths` with an explanatory
  comment

## 4. Verify

- [ ] 4.1 Run `go test -race -count=1 ./internal/scaffold/...` — all
  tests must pass
- [ ] 4.2 Confirm constitution alignment: Composability First (II) —
  downstream `uf init` now deploys Muti-Mind independently

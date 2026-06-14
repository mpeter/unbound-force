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

## 1. Write python.md v3.0.0

- [ ] 1.1 Rewrite `.opencode/uf/packs/python.md` from
  v1.0.0 to v3.0.0. Use gaze-py v2.0.0 as the base,
  tightened to Go-pack style (no code examples, single
  declarative bullets). Apply all wording decisions from
  design.md: TYPE_CHECKING exemption in CS-002, CS-008
  reworded to defer mutable-default prohibition to CS-023,
  CS-005 upgraded to MUST, CS-014 as SHOULD with EAFP
  carveout, TC-001 accepting pytest-mock, CS-023 scoped to
  mutable defaults only. Add all fourteen new rules
  (CS-017–025, AP-009–010, TA-005–006, TC-014) per
  specs/upgrade-python-pack.md FR-UPP-001. Bump frontmatter
  to `version: 3.0.0`.

- [ ] 1.2 Copy the completed `.opencode/uf/packs/python.md`
  verbatim to `internal/scaffold/assets/opencode/uf/packs/python.md`.
  The two files MUST be byte-identical.

## 2. Update drift detection test

- [ ] 2.1 In `internal/scaffold/scaffold_test.go`, add a
  **new** assertion (sub-test or table entry) that reads the
  embedded `opencode/uf/packs/python.md` content and
  verifies it contains `"version: 3.0.0"`, `"CS-017"`, and
  `"TC-014"` per FR-UPP-002. Do NOT modify the existing
  `hasRuleContent` stub fixtures at lines ~5207 and ~5245 —
  those contain unrelated `version: 1.0.0` strings for
  `go-custom.md` and `default-custom.md`. The existing
  `TestEmbeddedAssets_MatchSource` and
  `TestCanonicalSources_AreEmbedded` already enumerate
  `python.md` and handle byte-identity — only the
  version/rule-ID content assertion is new.

## 3. Verify

- [ ] 3.1 Run `go build ./...` — clean build.
- [ ] 3.2 Run `go test -race -count=1 ./internal/scaffold/...`
  — all tests pass, including drift detection and new
  v3.0.0 content assertion.
- [ ] 3.3 Run `go vet ./...` — no issues.
- [ ] 3.4 Confirm the two `python.md` files are identical:
  `diff .opencode/uf/packs/python.md
  internal/scaffold/assets/opencode/uf/packs/python.md`
  — zero output.
- [ ] 3.5 Confirm `python-custom.md` is unchanged in both
  locations (content identical to pre-change state).
- [ ] 3.6 Add a `CHANGELOG.md` entry for the
  v1.0.0 → v3.0.0 upgrade. Include the version-skip
  rationale (v2.0.0 reserved for gaze-py style
  compatibility), the count of new rules (14 total: 12
  from Dagster/gaze-py + 2 promoted), and the key
  tightening (CS-005 type annotations now MUST).
- [ ] 3.7 Constitution alignment verification: confirm the
  two `python.md` files have YAML frontmatter with
  `version` and `pack_id` (Observable Quality), no
  mandatory runtime dependencies introduced (Composability
  First), and drift detection test is independently
  runnable (Testability).

## 4. Pre-merge gate (MUST complete before opening PR)

- [ ] 4.1 File a `docs`-labelled issue in
  `unbound-force/website` titled
  "docs: update Python convention pack page for v3.0.0
  (14 new rules)". Body MUST list the 14 new rule IDs,
  the 5 modified rules (CS-002, CS-005, CS-008, CS-014,
  TC-001), and the version-skip rationale. Record the
  issue number here: # 157 (https://github.com/unbound-force/website/issues/157)
  Do NOT open the implementing PR until this task is
  complete.

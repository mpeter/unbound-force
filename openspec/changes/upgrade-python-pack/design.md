# Design: upgrade-python-pack

## Context

`python.md` v1.0.0 was scaffolded conservatively — rules
hedged with "or equivalent" and type annotations at SHOULD
level. Since that change, gaze-py has run v2.0.0 through a
full development cycle and Dagster's Dignified Python rules
have been cross-referenced against five major AI/LLM
projects and eight authoritative style sources.

The pack ships as two identical files:

1. `.opencode/uf/packs/python.md` — the live canonical
   source, loaded by agents in this repo.
2. `internal/scaffold/assets/opencode/uf/packs/python.md`
   — the embedded asset deployed by `uf init` to new
   Python projects.

A drift detection test in `scaffold_test.go` verifies both
are identical at every CI run. The implementation task is
to update both files atomically and update the test.

## Goals / Non-Goals

### Goals

- Upgrade `python.md` to v3.0.0 with fourteen new rules
  (twelve from Dagster/gaze-py + two promoted from
  gaze-py custom) and tightened wording on existing rules.
- Match Go-pack style: single declarative bullets, no code
  examples, 1-3 sentences per rule.
- Resolve three known weaknesses in v1.0.0: hedged tooling
  names, SHOULD-level type annotations, no Dagster rules.
- Promote two gaze-py custom rules to canonical.

### Non-Goals

- No changes to `python-custom.md` — stays empty template.
- No Go source changes (no new doctor checks, no new
  scaffold logic).
- No changes to Go, TypeScript, or default packs.
- No enforcement changes — rules are enforced by Divisor
  agents at review time, not by new CI gates.

## Decisions

### D1 — Go-pack style: no code examples

v1.0.0 followed the verbose TypeScript/Python-v1 style.
gaze-py v2.0.0 added code examples for CS-014, CS-015,
CS-016. This upgrade adopts Go-pack style throughout: one
declarative bullet per rule, no `\`\`\`python` blocks.

**Rationale**: Code examples bloat the pack significantly
(277 lines in gaze-py v2.0.0 vs 66 in go.md). Divisor
agents don't need examples — they need declarative
constraints. Examples belong in reference docs, not review
packs.

### D2 — CS-014 LBYL as SHOULD with EAFP carveout

Cross-source research shows EAFP is the Pythonic default
(Brett Cannon / CPython core dev, Google Style Guide
examples, all five surveyed AI/LLM projects). Mandating
LBYL as MUST would conflict with idiomatic Python and
generate PR friction.

Wording: "Prefer LBYL for local attribute and type checks
where the precondition is cheap and non-racy. Use EAFP at
I/O and third-party API call boundaries."

### D3 — CS-023 mutable defaults only (not all defaults)

`None` immutable defaults are idiomatic Python and
universal across all surveyed projects. The real bug
category (mutable `[]`, `{}` defaults) is already caught
by ruff B006. CS-023 is scoped to mutable defaults only.

### D4 — Explicit re-exports rule (not a blanket ban)

A blanket "no re-exports from `__init__.py`" conflicts with
how every major Python library exposes its public API.
The canonical rule from mypy `--no-implicit-reexport`: if
you re-export, make it explicit with `from .module import
Thing as Thing`. This aligns with tooling without breaking
library patterns.

### D5 — TC-001 relaxed to accept pytest-mock

Strict monkeypatch-only conflicts with common practice.
Research confirms `pytest-mock` is the right tool for
complex mock behavior (call tracking, side_effect,
return_value). Rule becomes: "Use `monkeypatch` for simple
value substitution. Use `pytest-mock` or `unittest.mock`
for complex mock behavior."

### D6 — Version bump to v3.0.0

MAJOR bump: significant rule count increase (12 new) and
substantive rewrites of existing rules (CS-002, CS-005,
CS-014, TC-001). Skipping v2.0.0 avoids confusion with
gaze-py's v2.0.0 which uses a different (more verbose)
style.

## Risks / Trade-offs

### R1 — Rules without ruff enforcement

CS-017, CS-018, CS-019, CS-020, CS-022, TC-014 have no
ruff rule backing — they are enforced only by Divisor
agents at review time. They will erode on PRs from
contributors unfamiliar with them. **Accepted** — this is
the same situation as many existing rules in all packs.

### R2 — CS-014 LBYL cultural friction

Even as a SHOULD, LBYL runs counter to Python's EAFP
culture. External contributors may push back on it.
**Accepted** — the EAFP carveout in D2 handles the
legitimate cases and reduces friction significantly.

### R3 — Drift detection test coupling

The drift test checks specific string content in
`python.md`. A version bump from v1.0.0 to v3.0.0 will
cause the test to fail until updated. **Mitigated** —
the test update is an explicit task in the implementation.

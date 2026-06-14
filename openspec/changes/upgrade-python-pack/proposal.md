# Change: upgrade-python-pack

## Why

The `python-convention-pack` change (merged) established a
v1.0.0 `python.md` pack as a foundation. Since then, two
additional sources have been validated against mainstream
Python AI/LLM projects and major agent-targeted rule sets:

1. **gaze-py v2.0.0** — a battle-tested evolution of the
   same pack, incorporating stronger opinions on ruff,
   type annotations, exception chaining, LBYL, keyword-only
   args, and testing conventions.

2. **Dagster Dignified Python** — ten rules extracted from
   Dagster's production Claude prompt, validated against
   five major AI/LLM Python projects (LangChain, LlamaIndex,
   PydanticAI, DSPy, Instructor) and cross-referenced with
   Google's Python Style Guide, Hypermodern Python, and
   five agent-targeted rule packs (sanjeed5, cursorrules.org,
   murataslan1, MuhammadUsmanGM, kdeldycke).

v1.0.0 is too hedged ("or equivalent") and misses rules
that appear in 4+ authoritative sources. This change upgrades
to v3.0.0, tightened to Go-pack style, with rules rewritten
as single declarative bullets.

## What Changes

### Modified Capabilities

- `python.md`: Upgraded from v1.0.0 to v3.0.0 (v2.0.0 is
  skipped to avoid confusion with gaze-py v2.0.0, which
  uses a different verbose style — see design.md D6).
  Existing
  rules tightened (ruff specifically named, type annotations
  upgraded from SHOULD to MUST, LBYL softened to SHOULD with
  EAFP carveout). Twelve new rules added from Dagster and
  cross-source research. Two rules promoted from gaze-py
  custom pack to canonical. No code examples — Go-pack style
  throughout.

### New Capabilities

Twelve rules added to `python.md`:

- **CS-017**: O(1) magic methods and properties.
- **CS-018**: Variables declared at use site.
- **CS-019**: No destructuring objects into single-use locals.
- **CS-020**: Maximum 4 indentation levels.
- **CS-021**: Always specify `encoding=` on file I/O
  (enforced by ruff PLW1514).
- **CS-022**: Context managers kept inline in `with`
  statements.
- **CS-023**: No mutable default parameter values.
- **AP-009**: No import-time computation — defer with
  `@cache`.
- **AP-010**: `pyproject.toml` as the project config file.
- **TA-005**: Runtime `isinstance()` assertion before
  `typing.cast()`.
- **TA-006**: `Literal` types for programmatic string/int
  value sets.
- **TC-014**: No speculative test infrastructure.

Two rules promoted from gaze-py `python-custom.md` to
canonical `python.md`:

- **CS-024** Explicit re-exports only (`from .module import X as X`).
- **CS-025** No placeholder strings in output — use `None`/`null`.

### Removed Capabilities

None. All v1.0.0 rules are preserved (some reworded).

## Impact

- `.opencode/uf/packs/python.md` — rewritten.
- `internal/scaffold/assets/opencode/uf/packs/python.md`
  — mirrors `.opencode/` copy (both must be identical).
- `internal/scaffold/scaffold_test.go` — drift detection
  tests updated to cover v3.0.0 content.
- `python-custom.md` — unchanged. Stays as empty template.

No Go source changes. No doctor changes. No new file paths.

## Documentation Obligation

Per constitution Development Workflow, a documentation
issue MUST be filed against `unbound-force/website` before
PR merge to reflect the upgraded Python pack in user-facing
docs.

## Constitution Alignment

Assessed against `.specify/memory/constitution.md` (v1.2.0).

### I. Autonomous Collaboration

**Assessment**: PASS

Convention packs are self-describing Markdown artifacts.
Divisor persona agents consume them dynamically at review
time without runtime coupling. The upgraded pack follows
the same YAML frontmatter + numbered rule format as all
existing packs — no agent communication changes required.

### II. Composability First

**Assessment**: PASS

The Python pack is independently deployable. No new
mandatory dependencies are introduced — the pack documents
tools (ruff, mypy, pytest) but does not require them for
the pack itself to function. New rules such as PLW1514 and
`@cache` reference standard library or existing toolchain
components.

### III. Observable Quality

**Assessment**: PASS

Pack files use RFC 2119 severity indicators and numbered
rule identifiers (CS-NNN, AP-NNN, etc.) consistent with all
existing packs. Machine-parseable YAML frontmatter preserved.
Version bump to v3.0.0 follows semantic versioning (MAJOR:
significant rule additions and rewrites).

### IV. Testability

**Assessment**: PASS

The only testable artifact is the drift detection test in
`scaffold_test.go`, which verifies that the embedded asset
matches the canonical source. This test is updated as part
of the change. No new functions, no new injectable
dependencies.

### V. Security by Default

**Assessment**: PASS

No new external inputs, no new distribution artifacts, no
supply chain surface. Pure Markdown content change.

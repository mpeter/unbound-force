# Specification: upgrade-python-pack

## Numbering Authority

gaze-py v2.0.0 rule numbering is the authoritative base
for v3.0.0. The statement "all v1.0.0 rules are preserved"
refers to rule *content and intent*, not rule IDs. Where
IDs differ between v1.0.0 and gaze-py v2.0.0, the gaze-py
v2.0.0 ID prevails. v1.0.0 rules that do not exist in
gaze-py v2.0.0 are assigned the next available ID in their
section.

## ADDED Requirements

### FR-UPP-001: Fourteen new rules in python.md v3.0.0

`python.md` MUST include the following rules, written as
single declarative bullets in Go-pack style with no code
examples:

- **CS-017** O(1) magic methods and properties.
- **CS-018** Variables declared at use site.
- **CS-019** No destructuring objects into single-use locals.
- **CS-020** Maximum 4 indentation levels.
- **CS-021** Always specify `encoding=` on file I/O
  (ruff PLW1514).
- **CS-022** Context managers kept inline in `with`
  statements.
- **CS-023** No mutable default parameter values (B006).
- **AP-009** No import-time computation — defer with
  `@cache`.
- **AP-010** `pyproject.toml` as the project config file.
- **TA-005** Runtime `isinstance()` before `typing.cast()`.
- **TA-006** `Literal` for programmatic string/int sets.
- **TC-014** No speculative test infrastructure.
- **CS-024** Explicit re-exports only: `from .module import
  Thing as Thing`. Implicit re-exports are forbidden. mypy
  `--no-implicit-reexport` enforces this.
- **CS-025** Never emit placeholder strings in output
  consumed by users or tooling. Fields that cannot be
  populated MUST be `None`/`null` — not `"unknown"`,
  `"n/a"`, or `0`.

#### Scenario: Pack deployed to new Python project

- **GIVEN** a new project directory with `pyproject.toml`
- **WHEN** `uf init` runs and deploys the Python pack
- **THEN** the deployed `python.md` contains rule IDs
  `CS-017`, `TC-014`, `CS-024`, and `CS-025`

*Note*: Verified indirectly — `TestEmbeddedAssets_MatchSource`
confirms the embedded asset equals the canonical source;
existing scaffold integration tests confirm `uf init`
deploys from embedded assets.

---

### FR-UPP-002: Drift detection test covers v3.0.0

The drift detection test in `scaffold_test.go` MUST verify
that `.opencode/uf/packs/python.md` and
`internal/scaffold/assets/opencode/uf/packs/python.md` are
byte-identical AND that the embedded pack contains the
v3.0.0 version string and two rule IDs spanning distinct
categories.

#### Scenario: Drift detection catches divergence

- **GIVEN** the two `python.md` files exist
- **WHEN** one file is modified without updating the other
- **THEN** the drift detection test fails with a clear
  message identifying which file is out of sync

#### Scenario: v3.0.0 content assertion

- **GIVEN** the drift detection test runs
- **WHEN** the embedded `python.md` is read
- **THEN** the content contains `"version: 3.0.0"`
- **AND** the content contains `"CS-017"` (new coding style
  rule)
- **AND** the content contains `"TC-014"` (new testing
  convention rule)

---

## Documentation Dependencies

Upon implementation, the following documentation MUST be
updated before PR merge:

- **unbound-force/website**: Update the Python convention
  pack reference page for v3.0.0 (new rule IDs, wording
  changes, version bump). File a `docs`-labelled issue and
  record the issue number in the PR description.
  See `proposal.md` §Documentation Obligation.
- **CHANGELOG.md**: Add a change entry for the
  v1.0.0 → v3.0.0 upgrade, including the version-skip
  rationale (v2.0.0 reserved for gaze-py style
  compatibility).

---

## MODIFIED Requirements

### python.md version bumped to v3.0.0

The `pack_id: python` frontmatter MUST read `version: 3.0.0`.
Previously: `version: 1.0.0`.

### CS-002: TYPE_CHECKING exemption explicit

CS-002 MUST explicitly state that `if TYPE_CHECKING:`
guards are a legitimate exception to the module-level
import rule. Previously: rule did not mention this
exemption.

### CS-005: Type annotations upgraded to MUST

CS-005 MUST require type annotations on all function
signatures (parameters and return types), public and
private. Previously: SHOULD-level recommendation.

### CS-008: Reworded to defer mutable-default prohibition to CS-023

CS-008 MUST be reworded to cover the `None`-sentinel
initialization idiom only: "Use `None` as default for
optional arguments and initialize inside the function body."
The mutable-default *prohibition* is exclusively carried by
CS-023 with the B006 ruff reference. Previously: CS-008
combined both the prohibition and the idiom in one rule.

### CS-014: LBYL as SHOULD with EAFP carveout

CS-014 MUST be a SHOULD-level rule. It MUST include an
explicit EAFP carveout: EAFP is appropriate at I/O and
third-party API call boundaries. Previously: MUST-level
with no carveout in v1.0.0.

### TC-001: pytest-mock accepted alongside monkeypatch

TC-001 MUST accept both `monkeypatch` (for simple value
substitution) and `pytest-mock`/`unittest.mock` (for
complex mock behavior with call tracking, side_effect, or
return_value). Note: in gaze-py v2.0.0 (the v3.0.0 base),
TC-001 is the mocking/monkeypatch rule; in v1.0.0 TC-001
was the test runner rule — numbering authority note above
applies. Previously: monkeypatch-only.

### CS-023: Mutable defaults only (not all defaults)

CS-023 MUST scope the prohibition to mutable default
argument values (`list`, `dict`, `set`, and other mutable
containers). Immutable `None` defaults are explicitly
permitted. Previously: N/A (new rule; supersedes the
mutable-default scope of CS-008 — see CS-008 MODIFIED).

---

## REMOVED Requirements

None. All v1.0.0 rules are preserved (content and intent
intact; some IDs renumbered per gaze-py v2.0.0 authority;
CS-008 reworded to delegate to CS-023).

# H-07 repair verification

## Checked scope

Repair baseline: upstream `ebed757` on branch
`opsx/h07-runtime-compatibility`. Planning artifacts passed
`openspec validate h07-runtime-compatibility --strict` and were
committed/pushed as `09d2ac0` before implementation. The staged adoption
candidate and primary checkout have not been promoted or modified.

## Homebrew contracts

Tests now exercise required input/output paths, missing input with an
existing destination, missing destination parent, and rename into a
nonempty directory. They assert contextual errors, destination state
and temporary-file removal. No production code or threshold changed.

Command: `GOTOOLCHAIN=go1.25.10 go test -race -count=1 ./internal/homebrew`
with a coverage profile retained during validation. Result:

```text
ok github.com/unbound-force/unbound-force/internal/homebrew
coverage: 93.9% of statements
```

## Reviewer runtime integration

The unchanged test failed with installed OpenCode v2.1.4:

```text
Unknown subcommand "agent" for "opencode debug"
Unrecognized flag: --pure in command opencode debug
```

An isolated private-server API probe discovered the fixture config
directory but returned an empty agent registry and AgentNotFoundError.
Documentation alone did not establish a working replacement. A bounded
follow-up tried `opencode debug agents --standalone`; installed v2.1.4
rejected that flag on the debug subcommand. No passing replacement or
completed compatibility repair is claimed. The original schema test
remains unchanged and its required check remains failed.

API contract references:
- https://opencode.ai/v2/openapi.json
- https://opencode.ai/v2/docs/cli

## Scaffold teardown

The isolated pipeline-operability recheck passed once on supported
Node 22.23.1 and Go 1.25.10. Historical full-suite cleanup failure is
not erased. Source inspection found that newStealthOpts uses real
LookPath and defaults ExecCmdInDir to real execution; activation runs
npm and OpenCode probes inside the temporary target. This establishes
real collaborators, not the identity of a late writer or a proven race.
No sleeps, cleanup retries, ignored errors or lifecycle edits added.
Full-suite verification remains necessary.

## Other checks

- All-active parity: Claude, OpenCode, Codex selected and verified;
  none skipped, no findings, exit 0.
- Plugin suite: passed, statements 95.34% against unchanged 94% floor.
- Council pipeline test script: passed, exit 0.

These passes do not certify the entire Go suite or PR readiness.
The Homebrew-only diff received a focused independent review with no
blockers. That review does not replace the required council.

## Delivery boundary

No PR or adoption promotion yet. Mandatory council dispatch tools
including capture_review_diff and plan_review_dispatch were not
exposed by this harness's tool discovery. An informal review does not
replace the required council protocol. Full local checks, council,
exact-head hosted checks and verified PR delivery remain outstanding.

The council command requires its policy tools as the source of truth
(`.opencode/commands/uf.review-council.md`, lines 228-260), and the
dispatch skill requires dispatch_review_runs rather than individual
agent dispatches. The tools are registered by the review-dispatch
plugin, not a documented standalone runner. Execution needs a harness
that exposes those project tools; no exemption or adapter was added.

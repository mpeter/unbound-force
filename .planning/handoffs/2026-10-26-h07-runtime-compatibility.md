# H-07 bounded repair continuation

## Objective

The operator authorized bounded OpenCode compatibility and Homebrew
coverage repairs through a verified PR. Keep adoption blocked until
required checks pass; do not merge or weaken gates.

## Current state

Worktree: unbound-force.worktrees/opsx-h07-runtime-compatibility.
Branch: opsx/h07-runtime-compatibility, based on upstream ebed757.
Spec validated and pushed in 09d2ac0. Homebrew test-only repair and
CHANGELOG pushed in 78c39a5, focused race tests pass at 93.9% coverage.
Original primary and staged adoption candidate remain unchanged.

Authoritative tasks and evidence are in
openspec/changes/h07-runtime-compatibility/{tasks,verification}.md.
Do not repeat the old runtime reruns without a new hypothesis.

## Blockers and next action

The original schema test remains unchanged and red. Isolated V2 API
calls discover config but not the fixture agent; debug agents rejects
the standalone flag. Verify a supported isolated initialization/read
path before adapting the assertions. Do not skip the check or migrate
production plugins as an incidental workaround.

Mandatory council tools are unavailable in this harness and no
supported standalone council runner was found. Continue council in a
harness exposing the project's policy tools; do not substitute informal
review. No PR has been created. Full integrated local checks, council,
exact-head hosted verification and journey closeout remain outstanding.

Workers finished: schema investigators made no edits; Homebrew worker
and focused reviewer completed. No worker is still editing this branch.

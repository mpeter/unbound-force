## Why

Muti-Mind is a cross-repo hero — it should be deployable to any
project via `uf init`, just like Cobalt-Crush, the Divisor council,
and Mx F. However, `uf init` does not deploy Muti-Mind to downstream
projects. The 13 command files and the agent persona live only in the
`unbound-force/unbound-force` repo's `.opencode/` directory and are
never embedded in the scaffold binary.

The root cause is that all 14 files contain `go run cmd/mutimind/main.go
<subcommand>` — a path that only resolves inside this repo. This made
the files unsafe to embed because they would break immediately in any
downstream project. As a result, they were classified as "local-only
tooling" and explicitly excluded from `internal/scaffold/assets/`.

The asymmetry with `mx-f-coach.md` (which IS embedded and deployed) has
no documented rationale. The `mx-f-coach` agent was written from the
start to invoke the `mxf` binary, so it was safe to embed. Muti-Mind
commands were not.

## What Changes

1. Replace every `go run cmd/mutimind/main.go <subcommand>` with
   `mutimind <subcommand>` across all 14 files (1 agent + 13 commands).
2. Embed the 14 cleaned files into `internal/scaffold/assets/opencode/`
   so `uf init` deploys them to downstream projects.
3. Update `scaffold_test.go` to remove the 13 muti-mind entries from
   `knownNonEmbeddedFiles` and add all 14 paths to `expectedAssetPaths`.

## Capabilities

### New Capabilities
- `uf init` deploys Muti-Mind: downstream projects initialized with
  `uf init` receive the `muti-mind-po` agent and all 13 backlog
  management commands.

### Modified Capabilities
- `muti-mind-po.md` (agent): `go run cmd/mutimind/main.go` references
  replaced with `mutimind` binary invocations — now safe to deploy to
  any project.
- All 13 `muti-mind.*.md` commands: same `go run` → `mutimind` fix,
  making them project-agnostic.

### Removed Capabilities
- None. No functionality is removed; the `go run` path was always broken
  in downstream projects so removing it is a pure fix.

## Impact

- `internal/scaffold/assets/opencode/agents/muti-mind-po.md` — new file
- `internal/scaffold/assets/opencode/commands/muti-mind.*.md` — 13 new files
- `.opencode/agents/muti-mind-po.md` — `go run` → `mutimind` fixes
- `.opencode/commands/muti-mind.*.md` — `go run` → `mutimind` fixes (13 files)
- `internal/scaffold/scaffold_test.go` — test manifest updated
- All downstream projects: receive Muti-Mind on next `uf init` run

## Constitution Alignment

Assessed against the Unbound Force org constitution.

### I. Autonomous Collaboration

**Assessment**: PASS

Muti-Mind communicates with the swarm via well-defined artifacts
(backlog items, acceptance decisions, spec files). Deploying the agent
and commands via `uf init` does not change how it collaborates — it
only ensures the collaboration capability is available in every hero
repo. The `mutimind` binary invocations produce the same structured
backlog item and acceptance-decision artifacts as before.

### II. Composability First

**Assessment**: PASS

This change makes Muti-Mind independently installable in any project
via `uf init`, directly satisfying the Composability First principle.
The `mutimind` binary (available in PATH as part of the unbound-force
toolchain, same as `mxf`) is the only runtime dependency — no new
mandatory coupling is introduced. Projects can skip muti-mind via
`.uf/config.yaml` `setup.tools.mutimind.method: skip` if needed.

### III. Observable Quality

**Assessment**: PASS

The `mutimind` CLI produces the same machine-parseable backlog item
and acceptance-decision JSON artifacts as before. Replacing
`go run cmd/mutimind/main.go` with `mutimind` does not change the
output format or provenance metadata.

### IV. Testability

**Assessment**: PASS

The scaffold test suite (`TestAssetPaths_MatchExpected` and
`TestCanonicalSources_AreEmbedded`) verifies the embedded asset
manifest bidirectionally. Updating `expectedAssetPaths` and removing
entries from `knownNonEmbeddedFiles` restores full drift detection
coverage for muti-mind files. No new test infrastructure is required.

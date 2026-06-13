## Context

`uf init` scaffolds a standard set of OpenCode agents and commands into
downstream hero repos via `internal/scaffold/assets/`. The `mx-f-coach`
agent and its associated `mxf` binary follow the pattern: agent file is
embedded, binary is expected in PATH. Muti-Mind should follow the same
pattern but does not — its 14 files were excluded because they contained
`go run cmd/mutimind/main.go`, a repo-internal path.

## Goals / Non-Goals

### Goals
- Deploy `muti-mind-po.md` and all 13 `muti-mind.*.md` commands via
  `uf init` to every downstream project
- Match the deployment model already established by `mx-f-coach.md`
- Restore full scaffold drift-detection test coverage for muti-mind files

### Non-Goals
- Adding `mutimind` to GoReleaser (not done for `mxf` either)
- Adding a `mutimind` entry to `initSubTools` (not done for `mxf`)
- Changing `cmd/mutimind/main.go` logic
- Updating spec 004 text

## Decisions

### D1 — Mirror the `mx-f-coach` pattern exactly

`mx-f-coach.md` is user-owned (deployed once, never overwritten by
`uf init` on subsequent runs). `muti-mind-po.md` gets the same
treatment: `isToolOwned` returns false for all `opencode/agents/`
paths, so no ownership code changes are needed.

The 13 command files are tool-owned (all `opencode/commands/` paths
are tool-owned), so they will be updated on subsequent `uf init` runs
when the embedded content differs — consistent with every other command.

### D2 — Replace `go run cmd/mutimind/main.go` with `mutimind` in source files

The source files in `.opencode/` are the canonical sources for the
embedded assets (validated by `TestCanonicalSources_AreEmbedded`). The
fix is made in the source files; the asset copies are then identical to
the sources. This keeps the single-source-of-truth invariant intact.

The pattern `mutimind <subcommand>` (binary invocation) is exactly how
`mx-f-coach.md` invokes `mxf collect` / `mxf metrics` — no special
handling needed.

### D3 — No `initSubTools` entry

`mxf` has no `initSubTools` entry. `uf init` does not run `mxf init`.
Muti-Mind follows the same pattern: `mutimind init` is a user-invoked
step via `/muti-mind.init`, not an automatic `uf init` side effect.
Adding an `initSubTools` entry would be a scope expansion beyond what
this change addresses.

### D4 — Test manifest is the enforcement mechanism

`TestAssetPaths_MatchExpected` verifies every embedded asset is in
`expectedAssetPaths`. `TestCanonicalSources_AreEmbedded` verifies every
canonical source file is either embedded or explicitly excluded. Moving
the 14 muti-mind paths from `knownNonEmbeddedFiles` to
`expectedAssetPaths` re-enables bidirectional drift detection — future
edits to the source files that are not reflected in the assets will
cause test failures.

## Risks / Trade-offs

**Risk**: Downstream projects that already have customized
`muti-mind-po.md` will not be overwritten (user-owned), but the 13
command files (tool-owned) will be updated on the next `uf init` run.
This is the established behaviour for all tool-owned commands and is
expected.

**Trade-off**: The `mutimind` binary must be in PATH for the commands to
work. Projects that have not installed the unbound-force toolchain will
have the files but a broken backend. This is identical to the `mxf`
situation and is accepted — the files still provide documentation value
and the agent persona is useful even without the binary for read-only
backlog queries via Dewey.

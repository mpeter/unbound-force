## ADDED Requirements

### Requirement: FR-001 Isolated reviewer runtime inspection

The reviewer integration test MUST inspect the fixture agent through
a supported interface of the installed OpenCode runtime. It MUST
verify empty provider options and absence of reviewer policy metadata.
It MUST NOT consume user credentials or configuration. It MUST report
runtime execution errors rather than convert them into passing skips.

#### Scenario: Supported runtime loads the fixture
- **GIVEN** an installed supported OpenCode and isolated fixture
- **WHEN** the integration test loads the reviewer agent
- **THEN** it SHALL assert the expected name and empty provider options
- **AND** it SHALL reject leaked policy metadata

#### Scenario: Runtime invocation fails
- **GIVEN** an installed runtime whose inspection fails
- **WHEN** the integration test invokes inspection
- **THEN** the test MUST fail with the execution diagnostic

### Requirement: FR-002 Homebrew failure contracts

Tests MUST verify input validation, read failure, and output filesystem
failure outcomes of Homebrew Run. Failed operations MUST return an
error and preserve existing destination content without partial output.
Tests MUST assert error context and cleanup where applicable.

#### Scenario: Missing input cannot replace a destination
- **GIVEN** a missing input and existing output file
- **WHEN** Run attempts the transformation
- **THEN** it MUST return a read error and leave output unchanged

#### Scenario: Destination cannot be replaced
- **GIVEN** a valid input and an invalid destination layout
- **WHEN** Run attempts to publish the transformed content
- **THEN** it MUST return a contextual error and remove temporary output

### Requirement: FR-003 Qualification preserves gates and lifecycle

All required checks MUST retain their thresholds and race flags.
Any test-owned processes MUST terminate before temporary directory
cleanup. Intermittent failures MUST remain reported until diagnosed;
one isolated pass MUST NOT certify a failed full suite.

#### Scenario: Adoption checks fail
- **GIVEN** any failed required check on the candidate
- **WHEN** readiness is reported
- **THEN** adoption MUST remain blocked and the failure MUST be named

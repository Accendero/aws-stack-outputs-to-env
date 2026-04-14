# Changelog

All notable changes to this project will be documented in this file.

## [v2.3] — 2026-04-14

### Added
- `Resources` as a valid `scope` value. When set, the action calls `describe-stack-resources` and exports each resource's `LogicalResourceId` as the variable name with its `PhysicalResourceId` as the value. The `matrix` output follows the same structure as `Outputs` and `Parameters`.
- Test coverage for `Resources` scope: a mock-based job verifying env var and matrix output, and integration test steps in `test-cfn-fetch`.

## [v2.2] — 2026-04-10

### Added
- `matrix` output: a declared JSON object of all collected stack outputs/parameters, keyed by name (with prefix applied if set). Because it is statically declared in `action.yml`, it is accessible to importing workflows and can be passed between jobs via `fromJSON`.
- Structural test verifying that `action.yml` declares an `outputs:` section, so the test suite correctly fails when output declaration is missing.

## [v2.1.1] — 2026-04-09

### Added
- Debug steps for environment variables and step outputs, printed when the runner debug flag is set (`ACTIONS_STEP_DEBUG=true`).

### Fixed
- Underscores are now allowed in `prefix` values (e.g. `MY_PREFIX` is valid).
- Empty string output values are no longer passed to `::add-mask::`, which caused spurious masking of whitespace in logs.

## [v2.1] — 2026-04-09

### Added
- `scope` input: set to `Parameters` to collect stack parameters instead of outputs. Defaults to `Outputs`.
- Stack outputs and parameters are now written to `$GITHUB_OUTPUT` as step outputs in addition to `$GITHUB_ENV`, making them accessible as `${{ steps.<id>.outputs.<key> }}` within the same job.

## [v2] — 2026-04-09

### Added
- `prefix` input: optional prefix applied to all exported variable names (e.g. `prefix: MYAPP` exports `MYAPP_BucketName`). Must be uppercase letters and underscores only.
- OIDC and session token support: `access_key_id` and `secret_access_key` are no longer required when an active AWS session is already configured (e.g. via `aws-actions/configure-aws-credentials`).
- `session_token` input for temporary credentials from STS or OIDC.
- Validation that `access_key_id` and `secret_access_key` are always provided together.
- Test coverage for credential validation and prefix behaviour.

## [v1] — 2023-06-21

### Added
- Initial release. Fetches CloudFormation stack outputs and exports each key as an environment variable via `$GITHUB_ENV`.

# Changelog

All notable changes to the Kodexa Sync GitHub Action will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [2.1.0] - 2025-12-03

### Added
- **Parallel Execution**: New `workers` input to control parallel resource deployment (default: 8 workers)
- **Resource Filtering**: New `filter` input for selective resource deployment by pattern (e.g., `invoice-*`)
- **Branch Override**: New `branch` input to override automatic branch detection
- **Tag Override**: New `tag` input to enable tag-based deployment mappings
- **JSON Report Output**: New `json-report` output provides structured deployment data for subsequent workflow steps
- **JSON Report Path**: New `json-report-path` output provides path to the JSON report file

### Changed
- Default worker count is now 8 for faster deployments (previously single-threaded)
- Enhanced command building to support all new kdx-cli v0.5.0 features
- Updated documentation with comprehensive examples for new features

### Documentation
- Added examples for parallel execution and filtering
- Added examples for tag-based deployment workflows
- Added examples for using JSON output in subsequent steps
- Added tag_mappings configuration documentation

## [2.0.4] - 2025-11-19

### Fixed
- Stream kdx command output in real-time instead of capturing silently
- Show helpful diagnostic message when kdx produces no output
- Use `tee` to display output immediately while still capturing for parsing
- Prevents silent failures where errors are hidden until after command completes

### Changed
- Output now appears in real-time during deployment instead of all at once at the end
- Better error messages when deployment fails without output

## [2.0.3] - 2025-11-19

### Fixed
- Fixed output parsing to correctly handle kdx v0.2.1 output format
- Replaced Perl-compatible regex (grep -P) with portable sed patterns
- Fixed GitHub Actions error "Invalid format '75'" when parsing deployment statistics
- Improved regex to specifically match "Resources: X updated" pattern

## [2.0.2] - 2025-11-19

### Changed
- Updated to use kdx-cli v0.2.1 which fixes critical panic in non-TTY environments
- CI/CD deployments now work reliably in GitHub Actions and other automated pipelines
- No action code changes required - automatic kdx-cli version detection handles the update

### Fixed
- Resolved issue where deployments would fail with exit code 2 and no error message
- Fixed nil pointer dereference that prevented `kdx sync deploy` from running in CI/CD

## [2.0.1] - 2025-11-19

### Changed
- Updated to kdx-cli v0.1.20 with enhanced environment configuration and module path resolution
- Improved manifest handling with better path resolution
- Enhanced resource type alias consistency

### Added
- Support for profile-based environment configuration in kdx-cli
- Better handling of module paths in manifest files
- Enhanced validation features

## [2.0.0] - 2025-11-19

### Changed
- Simplified action to be purely branch-based
- Refactored architecture for better maintainability

## [1.0.0] - 2025-11-09

### Added
- Initial release of Kodexa Sync GitHub Action
- Support for syncing metadata using kdx-cli
- Multi-platform support (Linux, macOS, Windows)
- Automatic binary download and caching
- Dry-run mode for validation
- Organization and project filtering
- Comprehensive examples and documentation
- Action outputs for sync statistics
- Environment-based deployment support

### Security
- Secure handling of authentication tokens via GitHub secrets
- Binary verification from official release repository

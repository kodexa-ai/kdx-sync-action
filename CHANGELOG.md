# Changelog

All notable changes to the Kodexa Sync GitHub Action will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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

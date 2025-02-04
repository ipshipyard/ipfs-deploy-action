# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## Unreleased

## [0.2.1] - 2025-02-04

### Fixed

- Make sure that storacha inputs are not required by action to allow for IPFS Cluster only deployments (inputs will be validated at the beginning of the action ensuring that either Storacha or IPFS Cluster inputs are provided).

## [0.2.0] - 2025-02-04

### Added

- Add support for IPFS Cluster CAR uploads.
- Add `cluster-upload-timeout` input to set the timeout for IPFS Cluster CAR uploads.

### Changed

- Storacha is now optional. You can now choose to upload the build CAR to IPFS Cluster instead.

### Fixed

- Fix action step summary output from Merkleizing into CAR step.

## [0.1.0] - 2025-01-31

### Added

- Initial release of of the ipfs-deploy-action

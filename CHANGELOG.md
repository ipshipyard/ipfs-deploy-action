# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## Unreleased

## [1.4.0] - 2025-03-05

### Added

- Add support for time-bound pins in IPFS Cluster via the `cluster-pin-expire-in` input parameter.
- Add support for custom pin names via the `pin-name` input parameter.

## [1.3.0] - 2025-03-05

### Added

- Add `ipfs-add-options` input to allow for customizing the `ipfs add` command used to merkleize the build into a CAR file.

### Changed

- Remove dependency on `ipfs-car` npm package, and use kubo instead to create the CAR file, since we need Kubo for pinning anyways.

## [1.2.1] - 2025-03-03

### Fixed

- Fix bash bug where the debug logging was not being set correctly.

## [1.2.0] - 2025-03-03

### Added

- Add `upload-car-artifact` input which will upload the CAR file as an artifact visible on GitHub Action Summary pages.

## [1.1.2] - 2025-02-26

### Fixed

- Improve error handling and logging when the action is not configured correctly, like when the folder to deploy is not found or empty.

## [1.1.1] - 2025-02-26

### Fixed

- Improve formatting of the action summary output.

## [1.1.0] - 2025-02-26

### Fixed

- Improve formatting of the action summary output.
- Add `dweb.link` and `w3s.link` (if `storacha-key` is provided) to the list of preview links.

## [1.0.0] - 2025-02-19

### Added

- Add timeout and retry logic to IPFS Cluster uploads.
  - Uploads to IPFS Cluster have a default timeout of 5 minutes.
  - If the upload fails, the action will retry by default 3 times with a 5 second delay between attempts.
  - The number of retry attempts and timeout can be customized using the `cluster-retry-attempts` and `cluster-timeout-minutes` inputs.

### Fixed

- Remove duplicate preview link from PR comment.

## [0.3.1] - 2025-02-10

### Fixed

- Log info to stdout instead of GitHub workflow summary

## [0.3.0] - 2025-02-10

### Added

- Add support for CAR uploads to Kubo via the Kubo RPC API.

### Removed

- Removed `cluster-upload-timeout` input as GitHub Actions does not support setting [timeout-minutes](https://github.com/actions/runner/blob/main/docs/adrs/0549-composite-run-steps.md#composite-run-steps-features) for steps in composite actions.

## [0.2.2] - 2025-02-10

### Fixed

- Default for `cluster-upload-timeout` input is a number instead of a string.

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

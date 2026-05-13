# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## Unreleased (v2.0.0)

**Own the merkleization. Pin anywhere.**

Merkleization (chunking your site, computing hashes, and assembling the merkle DAG into a Content Archive) is what decides your root CID. v2 narrows this action to that step so the CID is generated under your code review with versions and chunker settings you control. Once the CAR exists, it's a portable artifact: pinning services consume the bytes you hand them, they do not re-derive the CID. That makes it safe to compose any pinning service against the same CAR.

### What you can do now

- **Run with no native pinning configured.** Pass `path-to-deploy` and `github-token` alone; the action produces a CAR and exits. Recipes for Filecoin, Pinata, and Filebase are now under your control as separate workflow steps; see [`docs/recipes/`](https://github.com/ipshipyard/ipfs-deploy-action/tree/main/docs/recipes/).
- **Hand the CAR off downstream.** New outputs `car-path` (workspace-relative path) and `car-artifact-name` (workflow-artifact name) let follow-up steps and jobs consume the CAR without hard-coding filenames.
- **Pick the CAR filename.** New `car-file-name` input (default `'build.car'`); useful when the action runs more than once per job or when downstream tooling expects a specific name.
- **Get IPIP-0499 CIDs by default.** New `cid-profile` input applies a Kubo CID profile before merkleizing. Default `unixfs-v1-2025` follows [IPIP-0499](https://specs.ipfs.tech/ipips/ipip-0499/) for cross-implementation CID determinism. Set `unixfs-v0-2015` for legacy CIDv0 behavior.
- **Get a hard error instead of a silent skip on partial config.** If you set `cluster-url` you must also set `cluster-user` and `cluster-password`; same rule for Kubo's `kubo-api-url` and `kubo-api-auth`.
- **Auto-quiet reporting.** PR comments and commit status post when Kubo or Cluster is pinning, and stay silent in CAR-only mode unless you set `set-pr-comment: 'true'` or `set-github-status: 'true'` explicitly.

### Breaking changes

Native support for third-party pinning services was removed. The action is no longer the right place for vendor SDKs and auth wrappers; moving them out keeps it small, fast to release, and free of vendor lock-in. If you used any of them on `@v1`, copy the matching recipe into your workflow before bumping to `@v2`:

| Removed input | Migration |
| ------------- | --------- |
| `pinata-jwt-token`, `pinata-pinning-url` | [`docs/recipes/pinata.md`](https://github.com/ipshipyard/ipfs-deploy-action/blob/main/docs/recipes/pinata.md) (V3 Files API) |
| `filebase-access-key`, `filebase-secret-key`, `filebase-bucket` | [`docs/recipes/filebase.md`](https://github.com/ipshipyard/ipfs-deploy-action/blob/main/docs/recipes/filebase.md) |
| `storacha-key`, `storacha-proof` | Service sunset on 2026-04-15; pick another service from [`docs/recipes/`](https://github.com/ipshipyard/ipfs-deploy-action/tree/main/docs/recipes/) |

Bumping to `@v2` without migrating fails fast with a step-summary table that names each removed input and links to its recipe; nothing is silently dropped.

Other default changes:

- `kubo-version` default bumped to `v0.41.0` so the [IPIP-0499](https://specs.ipfs.tech/ipips/ipip-0499/) profile is available out of the box.
- `ipfs-add-options` default is now empty so the chosen `cid-profile` governs CID generation.
- `set-pr-comment` and `set-github-status` defaults changed from `'true'` to empty; they now auto-enable when Kubo or Cluster is configured. Existing Kubo and IPFS Cluster users see the same behavior as on v1.

Users on `@v1` are unaffected until they bump.

## [1.10.0] - 2026-05-06

### Changed

- Updated actions (requires Actions Runner v2.327.1+ on self-hosted runners due to Node 24 runtime):
  - `actions/upload-artifact@v4` -> `actions/upload-artifact@v7`
  - `actions/github-script@v7` -> `actions/github-script@v8`
  - `peter-evans/find-comment@v3` -> `peter-evans/find-comment@v4`
  - `peter-evans/create-or-update-comment@v4` -> `peter-evans/create-or-update-comment@v5`

## [1.9.2] - 2026-04-07

### Fixed

- README: dual workflow example now documents `event == 'push'` check for production deployment gates to prevent fork PRs from bypassing branch name conditions.

## [1.9.1] - 2026-04-04

### Fixed

- Fix Storacha deprecation blog post URL in warning and docs.

## [1.9.0] - 2026-04-03

### Changed

- Deprecate Storacha: uploads will stop working on April 15, 2026. A warning is now shown in CI run summary when Storacha credentials are configured. See [Storacha announcement](https://medium.com/@storacha/an-update-on-storacha-and-important-news-for-you-and-your-data-15a5d10b7da0).
- Allow Filebase as a standalone CAR upload provider (no longer requires Kubo or IPFS Cluster alongside it).

## [1.8.0] - 2026-01-14

### Changed

- Migrate from deprecated `@web3-storage/w3cli` to `@storacha/cli`. Existing workflows continue to work without changes.

## [1.7.0] - 2025-08-25

### Fixed

- Add support for using ipfs-deploy-action in workflows triggered by `workflow_run` events to allow secure usage in PRs from forks.

## [1.6.0] - 2025-05-16

### Added

- Add optional `github-status-gw` input to allow for customizing the gateway used for the commit status updates.

## [1.5.0] - 2025-03-07

### Added

- Add `set-pr-comment` input to control PR comment creation separately from GitHub commit status

### Fixed

- Fix bug where GitHub commit status was still being set when `set-github-status` was set to 'false'
- Update descriptions to clarify that string values 'true' and 'false' are expected for boolean inputs

## [1.4.1] - 2025-03-06

### Fixed

- Fix commit status and PR comment when action is triggered by `pull_request_target` event.
- Fix bug in `ipfs-cluster-ctl-version` input not being used correctly.

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

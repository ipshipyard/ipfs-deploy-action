# Deploy to IPFS Action

This GitHub Action automates the deployment of static sites to IPFS using [CAR files](https://docs.ipfs.tech/concepts/glossary/#car). It uploads CAR files to [Storacha](https://storacha.network), [Pinata](https://pinata.cloud), Kubo, or IPFS Cluster. The action will automatically create a preview link and update your PR/commit status with the deployment information.

This action is built and maintained by [Interplanetary Shipyard](http://ipshipyard.com/).
<a href="http://ipshipyard.com/"><img align="right" src="https://github.com/user-attachments/assets/39ed3504-bb71-47f6-9bf8-cb9a1698f272" /></a>

The [composite action](https://docs.github.com/en/actions/sharing-automations/creating-actions/about-custom-actions#composite-actions) makes no assumptions about your build process. You should just run your build and then call this action (as a step in an existing job) with the `path-to-deploy` input set to the path of your build output directory.

![Setting commit status](./screenshot-commit-status.png)

![PR comment with CID and preview links](./screenshot-pr-comment.png)

## Table of Contents

- [Features](#features)
- [How does this compare to the other IPFS actions?](#how-does-this-compare-to-the-other-ipfs-actions)
- [Storacha configuration](#storacha-configuration)
- [Inputs](#inputs)
  - [Required Inputs](#required-inputs)
  - [Optional Inputs](#optional-inputs)
- [Outputs](#outputs)
- [Usage](#usage)
  - [Simple Workflow (No Fork PRs)](#simple-workflow-no-fork-prs)
  - [Dual Workflows (With Fork PRs)](#dual-workflows-with-fork-prs)
- [FAQ](#faq)

## Features

- 📦 Merkleizes your static site into a CAR file
- 🚀 Uploads CAR file to Storacha, Pinata, IPFS Cluster, or Kubo
- 💾 Optional CAR file upload to Filebase
- 📤 CAR file attached to Github Action run Summary page
- 🔗 Automatic preview links
- 💬 Optional PR comments with CID and preview links
- ✅ Optional commit status updates with build CID

## How does this compare to the other IPFS actions?

This action encapsulates the established best practices for deploying static sites to IPFS in 2025

- Merkleizes the build into a CAR file in GitHub Actions using Kubo. This ensures that the CID is generated in the build process and is the same across multiple providers.
- Uploads the CAR file to IPFS via one or more providers: [Storacha](https://storacha.network), [Pinata](https://pinata.cloud), IPFS Cluster, or Kubo. Uploading locally created CAR ensures data is delivered directly without relying on network retrieval.
- Updates the PR/commit status with the deployment information and preview links.

## Storacha configuration

To set up the Storacha, you will need to install [w3cli](https://github.com/storacha/w3cli) and login with your Storacha account.

Once logged in:

- [Create a new space](https://docs.storacha.network/how-to/ci/#create-a-space) (like an S3 bucket) to which you will upload the merkleized CAR files.
- [Create a signing key](https://docs.storacha.network/how-to/ci/#create-a-signing-key) that will be used in CI to sign requests to Storacha.
- [Create a UCAN proof](https://docs.storacha.network/how-to/ci/#create-a-proof) that will be used in CI to sign requests to Storacha.

The signing key and proof will be used as [inputs](#inputs) to the action.

## Inputs

### Required Inputs

| Input              | Description                                                                                                                                                                                                  |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `path-to-deploy`   | Path to the directory containing the frontend build to merkleize into a CAR file and deploy to IPFS                                                                                                          |
| `github-token`     | GitHub token for updating commit status and PR comments                                                                                                                                                      |
| `kubo-api-url`     | Kubo RPC API URL to pass to `ipfs --api`, e.g. `/dns/YOUR_DOMAIN/tcp/443/https`                                                                                                                              |
| `kubo-api-auth`    | Kubo RPC API auth secret to pass to `ipfs --api-auth`, e.g. `basic:hello:world` (defined as `AuthSecret` in `API.Authorizations` config)                                                                     |
| `cluster-url`      | IPFS Cluster URL to pass to `ipfs-cluster-ctl --host`                                                                                                                                                        |
| `cluster-user`     | IPFS Cluster username for basic http auth                                                                                                                                                                    |
| `cluster-password` | IPFS Cluster password for basic http auth                                                                                                                                                                    |
| `storacha-key`     | Storacha base64 encoded key to use to sign UCAN invocations. Create one using `w3 key create --json` (and use `key` from the output). See: https://github.com/storacha/w3cli#w3_principal                    |
| `storacha-proof`   | Storacha Base64 encoded proof UCAN with capabilities for the space. Create one using `w3 delegation create did:key:DID_OF_KEY -c space/blob/add -c space/index/add -c filecoin/offer -c upload/add --base64` |
| `pinata-jwt-token` | Pinata JWT token for authentication                                                                                                                                                                          |

> [!IMPORTANT]
> To use this action, you must configure the inputs for at least one provider:
>
> - Storacha: `storacha-key` and `storacha-proof`
> - Pinata: `pinata-jwt-token`
> - Kubo: `kubo-api-url` and `kubo-api-auth`
> - IPFS Cluster: `cluster-url`, `cluster-user`, `cluster-password`
>
> You can configure multiple providers for redundancy.

### Optional Inputs

| Input                     | Description                                                                                                                                         | Default                                    |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| `kubo-version`            | Kubo CLI version to use for merkleizing and CAR creation                                                                                            | `'v0.33.0'`                                |
| `ipfs-add-options`        | Options to pass to `ipfs add` command that is used to merkleize the build. See [ipfs add docs](https://docs.ipfs.tech/reference/kubo/cli/#ipfs-add) | `'--cid-version 1 --chunker size-1048576'` |
| `pinata-retry-attempts`   | Number of retry attempts for Pinata uploads                                                                                                         | `'3'`                                      |
| `filebase-bucket`         | Filebase bucket name                                                                                                                                | -                                          |
| `filebase-access-key`     | Filebase access key                                                                                                                                 | -                                          |
| `filebase-secret-key`     | Filebase secret key                                                                                                                                 | -                                          |
| `set-github-status`       | Set GitHub commit status with build CID. Use "true" or "false" (as strings)                                                                         | `'true'`                                   |
| `set-pr-comment`          | Set PR comments with IPFS deployment information. Use "true" or "false" (as strings)                                                                | `'true'`                                   |
| `github-status-gw`        | Gateway to use for the links in commit status updates (The green checkmark with the CID)                                                            | `'inbrowser.link'`                         |
| `upload-car-artifact`     | Upload and publish the CAR file on GitHub Action Summary pages                                                                                      | `'true'`                                   |
| `cluster-ctl-version`     | IPFS Cluster CLI version to use                                                                                                                     | `'v1.1.2'`                                 |
| `cluster-retry-attempts`  | Number of retry attempts for IPFS Cluster uploads                                                                                                   | `'3'`                                      |
| `cluster-timeout-minutes` | Timeout in minutes for each IPFS Cluster upload attempt                                                                                             | `'5'`                                      |
| `cluster-pin-expire-in`   | Time duration after which the pin will expire in IPFS Cluster (e.g. 720h for 30 days). If unset, the CID will be pinned with no expiry.             | -                                          |
| `pin-name`                | Custom name for the pin. If unset, defaults to "{repo-name}-{commit-sha-short}" for IPFS Cluster and Pinata.                                        | -                                          |

## Outputs

| Output | Description                          |
| ------ | ------------------------------------ |
| `cid`  | The IPFS CID of the uploaded content |

## Usage

### Simple Workflow (No Fork PRs)

For repositories that don't accept PRs from forks, you can use a single workflow:

```yaml
name: Build and Deploy to IPFS

permissions:
  contents: read
  pull-requests: write
  statuses: write

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    outputs:
      cid: ${{ steps.deploy.outputs.cid }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build project
        run: npm run build

      - name: Deploy to IPFS
        uses: ipfs/ipfs-deploy-action@v1
        id: deploy
        with:
          path-to-deploy: out
          storacha-key: ${{ secrets.STORACHA_KEY }}
          storacha-proof: ${{ secrets.STORACHA_PROOF }}
          github-token: ${{ github.token }}
```

### Dual Workflows (With Fork PRs)

For secure deployments of PRs from forks, use two separate workflows that pass artifacts between them:

**`.github/workflows/build.yml`** - Builds without secrets access:
```yaml
name: Build

permissions:
  contents: read

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

env:
  BUILD_PATH: 'out'  # Update this to your build output directory

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event_name == 'pull_request' && github.event.pull_request.head.sha || github.sha }}


      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build project
        run: npm run build

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: website-build-${{ github.run_id }}
          path: ${{ env.BUILD_PATH }}
          retention-days: 1
```

**`.github/workflows/deploy.yml`** - Deploys with secrets access:
```yaml
name: Deploy

permissions:
  contents: read
  pull-requests: write
  statuses: write

on:
  workflow_run:
    workflows: ["Build"]
    types: [completed]

env:
  BUILD_PATH: 'website-build'  # Directory where artifact from build.yml will be unpacked

jobs:
  deploy-ipfs:
    if: github.event.workflow_run.conclusion == 'success'
    runs-on: ubuntu-latest
    outputs:
      cid: ${{ steps.deploy.outputs.cid }}
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: website-build-${{ github.event.workflow_run.id }}
          path: ${{ env.BUILD_PATH }}
          run-id: ${{ github.event.workflow_run.id }}
          github-token: ${{ github.token }}

      - name: Deploy to IPFS
        uses: ipfs/ipfs-deploy-action@v1
        id: deploy
        with:
          path-to-deploy: ${{ env.BUILD_PATH }}
          storacha-key: ${{ secrets.STORACHA_KEY }}
          storacha-proof: ${{ secrets.STORACHA_PROOF }}
          github-token: ${{ github.token }}
```

See real-world examples:
- [IPFS Specs](https://github.com/ipfs/specs/tree/main/.github/workflows) - Uses the secure two-workflow pattern
- [IPFS Docs](https://github.com/ipfs/ipfs-docs/tree/main/.github/workflows) - Uses the secure two-workflow pattern

## FAQ

- How can I safely build on PRs from forks?
  - Use the two-workflow pattern shown above. The build workflow runs on untrusted fork code without secrets access, while the deploy workflow only runs after a successful build and has access to secrets but never executes untrusted code. This pattern uses GitHub's `workflow_run` event to securely pass artifacts between workflows.
- What's the difference between uploading a CAR and using the Pinning API?
  - Since the CAR is like a tarball of the full build with some additional metadata (merkle proofs), the upload will be as big as the build output. Pinning with the [Pinning API](https://github.com/ipfs/pinning-services-api-spec) in contrast is just a request to instruct the pinning service to retrieve and pin the data from the IPFS network. This action uploads CAR files directly to all providers (Storacha, Pinata, IPFS Cluster, Kubo, Filebase), ensuring reliable data delivery without depending on network retrieval.
- How can I update DNSLink?
  - See https://github.com/ipfs/dnslink-action as a complement to this action.

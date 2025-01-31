# Deploy to IPFS Action

This GitHub Action allows you to deploy static sites to IPFS using Storacha with CAR files. It supports optional pinning to Pinata and uploading to Filebase. The action will automatically create a preview link and update your PR/commit status with the deployment information.

The [composite action](https://docs.github.com/en/actions/sharing-automations/creating-actions/about-custom-actions#composite-actions) makes no assumptions about your build process. You should just run your build and then call this action with the `path-to-deploy` input set to the path to the build directory.

## Features

- 📦 Merkleizes your static site into a CAR file
- 🚀 Uploads to IPFS via Storacha
- 📍 Optional pinning to Pinata
- 💾 Optional backup to Filebase
- 🔗 Automatic preview links
- 💬 PR comments with deployment info
- ✅ Commit status updates

## How does this compare to the other IPFS actions?

This action encapsulates some of the established best practices for deploying static sites to IPFS in 2025:

- Merkleizes the build into a CAR file in GitHub Actions using `ipfs-car`. This ensures that the CID is generated in the build process and is the same across multiple providers.
- Uploads the CAR file to IPFS via [Storacha](https://storacha.network).
- Optionally pins the CID of the CAR file to Pinata. This is useful for redundancy (multiple providers). The pinning here is done in the background and non-blocking. (When pinning, Pinata will fetch the data from Storacha.)
- Updates the PR/commit status with the deployment information and preview links.

## Inputs

### Required Inputs

| Input            | Description                                                                                                                                                                                |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `path-to-deploy` | Path to the directory containing the frontend build to merkleize into a CAR file and deploy to IPFS                                                                                        |
| `storacha-key`   | Storacha base64 encoded key to use to sign UCAN invocations. Create one using `w3 key create --json`. See: https://github.com/storacha/w3cli#w3_principal                                  |
| `storacha-proof` | Storacha Base64 encoded proof UCAN with capabilities for the space. Create one using `w3 delegation create did:key:DID_OF_KEY -c space/blob/add -c space/index/add -c filecoin/offer -c upload/add --base64` |
| `github-token`   | GitHub token for updating commit status and PR comments                                                                                                                                    |

### Optional Inputs

| Input                 | Description                              | Default                          |
| --------------------- | ---------------------------------------- | -------------------------------- |
| `node-version`        | Node.js version to use                   | `'20'`                           |
| `kubo-version`        | Kubo version to use for pinning          | `'v0.33.0'`                      |
| `pinata-pinning-url`  | Pinata Pinning Service URL               | `'https://api.pinata.cloud/psa'` |
| `pinata-jwt-token`    | Pinata JWT token for authentication      | -                                |
| `filebase-bucket`     | Filebase bucket name                     | -                                |
| `filebase-access-key` | Filebase access key                      | -                                |
| `filebase-secret-key` | Filebase secret key                      | -                                |
| `set-github-status`   | Set GitHub commit status and PR comments | `'true'`                         |

## Outputs

| Output | Description                          |
| ------ | ------------------------------------ |
| `cid`  | The IPFS CID of the uploaded content |

## Usage

Here's a basic example of how to use this action in your workflow:

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

      - uses: ./.github/actions/deploy-to-ipfs
        name: Deploy to IPFS
        id: deploy
        with:
          path-to-deploy: out
          storacha-key: ${{ secrets.STORACHA_KEY }}
          storacha-proof: ${{ secrets.STORACHA_PROOF }}
          github-token: ${{ github.token }}
```

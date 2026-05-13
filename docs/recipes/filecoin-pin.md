# Archive CAR to Filecoin via `filecoin-pin`

[`filecoin-pin`](https://github.com/filecoin-project/filecoin-pin) is a CLI that uploads a CAR to Filecoin Onchain Cloud (Synapse) using PDP, and tops up USDFC from a wallet when needed. This recipe runs `filecoin-pin import` as a follow-up step against the CAR that `ipfs-deploy-action` produces (path comes from the `car-path` output). The root CID is preserved end-to-end.

## Prerequisites

- A Filecoin wallet private key stored in a repository secret (e.g. `FILECOIN_WALLET_KEY`). The wallet must hold FIL for gas and either USDFC or FIL that `--auto-fund` can convert to USDFC.
- Decide on spend caps: `--min-runway-days` (how many days of storage runway the auto-fund must achieve) and `--max-balance` (cap on USDFC balance after top-up). Upstream intentionally does not default these, since they govern wallet spend.

## Recipe

```yaml
- name: Create IPFS CAR
  uses: ipfs/ipfs-deploy-action@v2
  id: deploy
  with:
    path-to-deploy: 'out'
    cid-profile: 'unixfs-v1-2025'  # IPIP-0499; switch to 'unixfs-v0-2015' only if you need legacy CIDv0
    github-token: ${{ github.token }}

- name: Archive CAR to Filecoin
  env:
    PRIVATE_KEY: ${{ secrets.FILECOIN_WALLET_KEY }}
    CAR_PATH: ${{ steps.deploy.outputs.car-path }}
  run: |
    npx -y filecoin-pin@0.20.1 import "$CAR_PATH" \
      --mainnet \
      --auto-fund \
      --min-runway-days 30 \
      --max-balance 5.0
```

The CAR path comes from `steps.deploy.outputs.car-path`. The root CID is available as `steps.deploy.outputs.cid` if your follow-up steps need it. Add Kubo or IPFS Cluster inputs to the deploy step if you also want to pin to your own infrastructure; see the [main README](https://github.com/ipshipyard/ipfs-deploy-action/blob/main/README.md#native-pinning-providers-optional).

## Notes

- **Gate this step yourself.** It spends wallet funds, so decide where it is allowed to run before adding it. Common gates: only on `push` to your default branch, only when same-repo (skip fork PRs), or only inside the secrets-bearing half of a dual workflow. The action does not gate for you so you stay in control.
- Pin the CLI version (`filecoin-pin@0.20.1` here) so a future upstream release does not change behavior under your CI without a code review.
- Use `--calibnet` instead of `--mainnet` for testnet runs.

## Upstream references

- CLI: https://github.com/filecoin-project/filecoin-pin
- Flag reference for `--auto-fund`, `--min-runway-days`, `--max-balance`: see the [`filecoin-pin` README](https://github.com/filecoin-project/filecoin-pin#cli).

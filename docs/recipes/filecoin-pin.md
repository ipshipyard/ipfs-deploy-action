# Archive CAR to Filecoin via `filecoin-pin`

[`filecoin-pin`](https://github.com/filecoin-project/filecoin-pin) uploads a CAR to Filecoin Onchain Cloud (Synapse) over PDP, topping up USDFC from a wallet when needed. This recipe feeds it the CAR that `ipfs-deploy-action` produces: `filecoin-pin import` archives that exact CAR and preserves its root CID end-to-end.

The upstream [CLI recipe](https://github.com/filecoin-project/filecoin-pin/tree/master/upload-action/examples/cli-recipe) is the source of truth for running the CLI in CI: wallet setup, spend caps, version pinning, network selection, and egress. This page covers only the wiring specific to `ipfs-deploy-action`.

## Prerequisites

- A Filecoin wallet private key in a repository secret (e.g. `FILECOIN_WALLET_KEY`). The wallet must hold FIL for gas and either USDFC or FIL that `--auto-fund` converts to USDFC.
- Spend caps `--min-runway-days` and `--max-balance`. See the upstream recipe for how to choose them.

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
    npx -y filecoin-pin@0.22.3 import "$CAR_PATH" \
      --network mainnet \
      --auto-fund \
      --min-runway-days 30 \
      --max-balance 5.0
```

The CAR path comes from `steps.deploy.outputs.car-path`; the root CID is `steps.deploy.outputs.cid`. Add Kubo or IPFS Cluster inputs to the deploy step to also pin to your own infrastructure; see the [main README](https://github.com/ipshipyard/ipfs-deploy-action/blob/main/README.md#native-pinning-providers-optional).

## Notes

- **Gate this step yourself.** It spends wallet funds, so decide where it runs before adding it: typically `push` to your default branch, or the secrets-bearing half of a dual workflow.
- Use `--network calibration` for testnet runs. There is no `--mainnet` / `--calibnet` shorthand.

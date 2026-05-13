# Upload CAR to Filebase via S3 endpoint

[Filebase](https://filebase.com) exposes an S3-compatible endpoint at `s3.filebase.com`. Uploading a CAR with the `import=car` object metadata tells Filebase to unpack and pin the contained root CID. This recipe uploads the CAR that `ipfs-deploy-action` produces (path comes from the `car-path` output).

## Prerequisites

- A Filebase bucket and S3 credentials stored in repository secrets (e.g. `FILEBASE_ACCESS_KEY`, `FILEBASE_SECRET_KEY`, `FILEBASE_BUCKET`).
- No setup step is required: the `aws` CLI is preinstalled on `ubuntu-latest` runners.

## Recipe

```yaml
- name: Create IPFS CAR
  uses: ipfs/ipfs-deploy-action@v2
  id: deploy
  with:
    path-to-deploy: 'out'
    cid-profile: 'unixfs-v1-2025'  # IPIP-0499; switch to 'unixfs-v0-2015' only if you need legacy CIDv0
    github-token: ${{ github.token }}

- name: Upload CAR to Filebase
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.FILEBASE_ACCESS_KEY }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.FILEBASE_SECRET_KEY }}
    FILEBASE_BUCKET: ${{ secrets.FILEBASE_BUCKET }}
    CID: ${{ steps.deploy.outputs.cid }}
    CAR_PATH: ${{ steps.deploy.outputs.car-path }}
  run: |
    OBJECT_KEY="${GITHUB_REPOSITORY//\//-}-${GITHUB_SHA:0:7}-${CID}.car"
    aws --endpoint https://s3.filebase.com s3 cp \
      "$CAR_PATH" "s3://${FILEBASE_BUCKET}/${OBJECT_KEY}" \
      --metadata 'import=car'
```

Add Kubo or IPFS Cluster inputs to the deploy step if you also want to pin to your own infrastructure; see the [main README](https://github.com/ipshipyard/ipfs-deploy-action/blob/main/README.md#native-pinning-providers-optional).

## Notes

- The `--metadata 'import=car'` flag is what tells Filebase to treat the upload as a CAR and pin its root CID. Without it the object is stored as opaque bytes.
- Object key includes the CID so re-deploys do not overwrite previous CARs; drop the CID suffix if you want the bucket to track only the latest deploy.

## Upstream references

- Filebase docs: https://docs.filebase.com/
- CAR upload via S3: https://docs.filebase.com/api-documentation/ipfs-pinning-service-api

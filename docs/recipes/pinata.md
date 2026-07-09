# Upload CAR to Pinata via V3 Files API

[Pinata](https://pinata.cloud)'s [V3 Files API](https://docs.pinata.cloud/api-reference/endpoint/upload-a-file) accepts a CAR upload directly, with `car=true` instructing Pinata to unpack it server-side and pin the contained root CID. This recipe uploads the CAR that `ipfs-deploy-action` produces (path comes from the `car-path` output), then verifies that the CID Pinata returns matches the one this action computed.

## Prerequisites

- A Pinata JWT token stored in a repository secret (e.g. `PINATA_JWT_TOKEN`). Create one at https://app.pinata.cloud/developers/api-keys.

## Recipe

```yaml
- name: Create IPFS CAR
  uses: ipfs/ipfs-deploy-action@v2
  id: deploy
  with:
    path-to-deploy: 'out'
    cid-profile: 'unixfs-v1-2025'  # IPIP-0499; switch to 'unixfs-v0-2015' only if you need legacy CIDv0
    github-token: ${{ github.token }}

- name: Upload CAR to Pinata
  env:
    PINATA_JWT_TOKEN: ${{ secrets.PINATA_JWT_TOKEN }}
    CID: ${{ steps.deploy.outputs.cid }}
    CAR_PATH: ${{ steps.deploy.outputs.car-path }}
  run: |
    PIN_NAME="${GITHUB_REPOSITORY//\//-}-${GITHUB_SHA:0:7}"

    response=$(curl -sS -w "\n%{http_code}" -X POST \
      "https://uploads.pinata.cloud/v3/files" \
      -H "Authorization: Bearer ${PINATA_JWT_TOKEN}" \
      -F "network=public" \
      -F "file=@${CAR_PATH}" \
      -F "name=${PIN_NAME}" \
      -F "car=true")

    http_code=$(echo "$response" | tail -n1)
    body=$(echo "$response" | sed '$d')

    if [ "$http_code" != "200" ]; then
      echo "::error::Pinata upload failed (HTTP $http_code): $body"
      exit 1
    fi

    returned_cid=$(echo "$body" | jq -r '.data.cid')
    if [ "$returned_cid" != "$CID" ]; then
      echo "::warning::Pinata returned CID $returned_cid, expected $CID"
    fi
    echo "Uploaded CAR to Pinata as ${PIN_NAME} (CID ${returned_cid})"
```

Add Kubo or IPFS Cluster inputs to the deploy step if you also want to pin to your own infrastructure; see the [main README](https://github.com/ipshipyard/ipfs-deploy-action/blob/main/README.md#native-pinning-providers-optional).

## Notes

- The recipe uses the **V3 Files API** with `car=true`, not the older PSA endpoint. PSA only registered a pin request and let Pinata fetch the data over the public IPFS network; V3 Files API uploads the CAR bytes directly, so the data is delivered to Pinata even if no other peer has it yet.
- The CID-match warning is informational. A mismatch typically means a chunker or CID-version difference between this action's `ipfs add` and Pinata's CAR parser; investigate before relying on the pin.
- `jq` is preinstalled on `ubuntu-latest` runners.

## Upstream references

- API docs: https://docs.pinata.cloud/api-reference/endpoint/upload-a-file

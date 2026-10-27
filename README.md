# Sapherbox build log

Public, append-only record of every web-client build that Sapherbox's CI produced.
One directory per build under `web/<tag>/`:

| File | Content |
|---|---|
| `build-manifest.json` | SHA-256 of every file the web client ships, a root hash over the sorted list, the source revision and the toolchain. The same file is served by every tenant at `/.well-known/build-manifest.json`. |
| `build-manifest.sigstore.json` | Sigstore bundle: the manifest's signature, the short-lived signing certificate bound to the GitHub Actions workflow of `jetned/sapherbox`, and the Rekor transparency-log inclusion proof. |
| `images.txt` | Registry digests of the backend and frontend container images built from the same commit. Both digests are signed with the same workflow identity. |

## Verify a tenant

1. Fetch what the tenant serves: `curl https://<tenant>/.well-known/build-manifest.json`.
2. Compare its `root_sha256` with the entry here for the same `revision`.
3. Check the signature of the entry here, without trusting this repository:

```bash
cosign verify-blob \
  --bundle web/<tag>/build-manifest.sigstore.json \
  --certificate-identity-regexp '^https://github.com/jetned/sapherbox/' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  web/<tag>/build-manifest.json
```

4. Check that the tenant really serves those bytes: `scripts/verify-build.sh https://<tenant> web/<tag>/build-manifest.json` from the product repository, or by hand: download each file listed in the manifest and compare its SHA-256.

## Rebuild it yourself

The build is reproducible. Check out the revision named in the manifest, run
`docker build -f frontend/Dockerfile --build-arg GIT_REVISION=<revision> --target builder-v2 .`
and read `/repo/frontend-v2/dist/frontend-v2/browser/.well-known/build-manifest.json` out of the
image. Its `root_sha256` must equal the published one. Details: `REPRODUCIBLE-BUILDS.md` in the
product repository.

Entries are written by CI only. Nothing here is edited by hand.

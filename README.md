# Cosign Testbed

Container images for testing GitHub attestations and Kyverno's cosign verification.

**Registry:** `ghcr.io/lucchmielowski/cosign-testbed`

**Platforms:** `linux/amd64`, `linux/arm64` (MacOS ARM/M1/M2/M3 supported)

## Available Images

| Image Tag | Cosign Version | Signing Method | Artifacts Created | Notes |
|-----------|----------------|----------------|-------------------|-------|
| `:latest` | N/A | GitHub Attestation | SLSA v1.0 build provenance attestation | Native GitHub attestation |
| `:unsigned` | N/A | None | No signatures | Baseline for comparison |
| `:v2-traditional` | v2.4.1 | Key-based (by tag) | Traditional OCI signature manifest (`.sig` image) | Original cosign format |
| `:v2-keyless` | v2.4.1 | Keyless OIDC (by tag) | Traditional OCI signature manifest + Fulcio cert + Rekor entry | Signature in transparency log |
| `:v3-traditional` | v3.0.4 | Key-based (by tag) | Traditional OCI signature manifest (`.sig` image) | Backward compatible with v2 |
| `:v3-keyless` | v3.0.4 | Keyless OIDC (by tag) | Traditional OCI signature manifest + Fulcio cert + Rekor entry | Signature in transparency log |
| `:v3-bundle` | v3.0.4 | Key-based (by digest) | Traditional OCI signature manifest (`.sig` image) | Signed by digest for multi-platform* |

**\*Note on v3-bundle:** Originally intended to demonstrate the cosign v3 bundle format (`.sigstore.json` as OCI referrer), but the `--bundle` flag has compatibility issues with multi-platform manifest lists. This image demonstrates digest-based signing instead, which ensures proper signature attachment to multi-architecture images.

## Important Notes

**Multi-Platform Support:** All images are built as multi-platform images supporting both `linux/amd64` and `linux/arm64` architectures.

## Verification Methods

### GitHub Attestation

```bash
# Using cosign to verify GitHub build provenance attestation (SLSA v1.0)
cosign verify-attestation \
  --type https://slsa.dev/provenance/v1 \
  --certificate-identity=https://github.com/lucchmielowski/cosign-testbed/.github/workflows/ci.yml@refs/heads/main \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com \
  ghcr.io/lucchmielowski/cosign-testbed:latest
```

### Cosign Key-Based Signatures

```bash
# v2-traditional (verify with any cosign v2 or v3)
cosign verify --key cosign.pub ghcr.io/lucchmielowski/cosign-testbed:v2-traditional

# v3-traditional (verify with cosign v3)
cosign verify --key cosign.pub ghcr.io/lucchmielowski/cosign-testbed:v3-traditional

# v3-bundle (verify with cosign v3)
cosign verify --key cosign.pub ghcr.io/lucchmielowski/cosign-testbed:v3-bundle
```

### Cosign Keyless Signatures

```bash
# v2-keyless (verify with GitHub Actions OIDC identity)
cosign verify \
  --certificate-identity=https://github.com/lucchmielowski/cosign-testbed/.github/workflows/ci.yml@refs/heads/main \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com \
  ghcr.io/lucchmielowski/cosign-testbed:v2-keyless

# v3-keyless (verify with GitHub Actions OIDC identity)
cosign verify \
  --certificate-identity=https://github.com/lucchmielowski/cosign-testbed/.github/workflows/ci.yml@refs/heads/main \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com \
  ghcr.io/lucchmielowski/cosign-testbed:v3-keyless
```

## Understanding Signature Artifacts

### Traditional OCI Signature Manifests (`.sig` images)
Created by: `:v2-traditional`, `:v3-traditional`, `:v3-bundle`

These store the signature as a separate OCI image in the registry with a `.sig` tag suffix. This is the original cosign format and is backward compatible across cosign versions.

**Example:**
- Image: `ghcr.io/lucchmielowski/cosign-testbed:v3-traditional`
- Signature: `ghcr.io/lucchmielowski/cosign-testbed:sha256-abc123.sig`

### Keyless Signatures (Fulcio + Rekor)
Created by: `:v2-keyless`, `:v3-keyless`

These use short-lived certificates from Fulcio (certificate authority) and store the signature in Rekor (transparency log). No long-lived signing keys are needed.

**Artifacts:**
- Traditional OCI signature manifest (`.sig` image)
- Fulcio certificate (embedded in signature)
- Rekor transparency log entry

### GitHub Attestations
Created by: `:latest`

Native GitHub attestations using SLSA v1.0 build provenance. Stored directly in GitHub's attestation registry.

**Artifacts:**
- SLSA build provenance attestation
- Signed with GitHub's signing infrastructure

---

**For setup instructions, see [DEVELOPMENT.md](DEVELOPMENT.md)**

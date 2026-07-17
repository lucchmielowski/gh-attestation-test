# Kyverno Cosign Testbed

Container images for testing Kyverno's cosign verification against multiple cosign use cases.

**Registry:** `ghcr.io/lucchmielowski/kyverno-cosign-testbed`

**Platforms:** `linux/amd64`, `linux/arm64` (MacOS ARM/M1/M2/M3 supported)

## Available Images

| Image Tag | Cosign Version | Signing Method | Artifacts Created | Notes |
|-----------|----------------|----------------|-------------------|-------|
| `:github-attestation` | N/A | GitHub Attestation | SLSA v1.0 build provenance attestation | Native GitHub attestation |
| `:github-sbom` | N/A | GitHub Attestation | SPDX SBOM attestation | Software Bill of Materials attestation |
| `:unsigned` | N/A | None | No signatures | Baseline for comparison |
| `:v2-traditional` | v2.4.1 | Key-based (by tag) | Traditional OCI signature manifest (`.sig` image) | Original cosign format |
| `:v2-keyless` | v2.4.1 | Keyless OIDC (by tag) | Traditional OCI signature manifest + Fulcio cert + Rekor entry | Signature in transparency log |
| `:v3-traditional` | v3.0.4 | Key-based (by tag) | Traditional OCI signature manifest (`.sig` image) | Backward compatible with v2 |
| `:v3-keyless` | v3.0.4 | Keyless OIDC (by tag) | Traditional OCI signature manifest + Fulcio cert + Rekor entry | Signature in transparency log |
| `:v3-bundle` | v3.0.4 | Key-based (by digest) | Traditional OCI signature manifest (`.sig` image) | Signed by digest for multi-platform* |
| `:v3-rekorv2-keyless` | v3.0.6 | Keyless OIDC (by tag) | Traditional OCI signature manifest + Fulcio cert + **Rekor v2** entry | Signed against a Rekor v2 (majorApiVersion 2) log via `--signing-config`; see [kyverno#15557](https://github.com/kyverno/kyverno/issues/15557) |

**\*Note on v3-bundle:** Originally intended to demonstrate the cosign v3 bundle format (`.sigstore.json` as OCI referrer), but the `--bundle` flag has compatibility issues with multi-platform manifest lists. This image demonstrates digest-based signing instead, which ensures proper signature attachment to multi-architecture images.

## Policy Compatibility Matrix

The following table shows which sample policies work with which image types:

| Policy File | Policy Type | Signing Method | `:github-attestation` | `:github-sbom` | `:v2-traditional` | `:v2-keyless` | `:v3-traditional` | `:v3-keyless` | `:v3-bundle` |
|------------|-------------|----------------|:---------------------:|:--------------:|:-----------------:|:-------------:|:----------------:|:-------------:|:------------:|
| `cpol.yaml` | ClusterPolicy | Keyless | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ |
| `cpol-key.yaml` | ClusterPolicy | Key-based | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| `cpol-att.yaml` | ClusterPolicy | GitHub Attestation | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| `cpol-sbom.yaml` | ClusterPolicy | GitHub SBOM | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| `ivpol.yaml` | ImageValidatingPolicy | Keyless | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ | ❌ |
| `ivpol-key.yaml` | ImageValidatingPolicy | Key-based | ❌ | ❌ | ✅ | ❌ | ✅ | ❌ | ✅ |
| `ivpol-gh-att.yaml` | ImageValidatingPolicy | GitHub Attestation | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| `ivpol-sbom.yaml` | ImageValidatingPolicy | GitHub SBOM | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |

### Key Compatibility Rules:

1. **ClusterPolicy (CPOL) Limitations:**
   - ❌ **Does NOT support cosign v3** signatures (`:v3-traditional`, `:v3-keyless`, `:v3-bundle`)
   - ❌ By extension, does not support GH attestations verification
   - ✅ Only supports cosign v2 signatures (`:v2-traditional`, `:v2-keyless`)

2. **ImageValidatingPolicy (IVPOL) Capabilities:**
   - ✅ Supports both cosign v2 and v3 signatures
   - ✅ Supports GitHub attestations (`:github-attestation`, `:github-sbom`)
   - ✅ More flexible and recommended for newer Kyverno versions

### Policy Files Location

All sample policies are located in the [`sample-policies/`](sample-policies/) directory:

**ClusterPolicy (CPOL) - v2 only:**
- `cpol.yaml` - Keyless signatures (v2 only)
- `cpol-key.yaml` - Key-based signatures (v2 only)
- `cpol-att.yaml` - GitHub SLSA provenance attestations
- `cpol-sbom.yaml` - GitHub SPDX SBOM attestations

**ImageValidatingPolicy (IVPOL) - v2 & v3:**
- `ivpol.yaml` - Keyless signatures (v2 & v3)
- `ivpol-key.yaml` - Key-based signatures (v2 & v3)
- `ivpol-gh-att.yaml` - GitHub SLSA provenance attestations
- `ivpol-sbom.yaml` - GitHub SPDX SBOM attestations


## Important Notes

**Multi-Platform Support:** All images are built as multi-platform images supporting both `linux/amd64` and `linux/arm64` architectures.

## Verification Methods

### GitHub Attestations

**Build Provenance (SLSA v1.0):**

```bash
# Using cosign to verify GitHub build provenance attestation
cosign verify-attestation \
  --type https://slsa.dev/provenance/v1 \
  --certificate-identity=https://github.com/lucchmielowski/kyverno-cosign-testbed/.github/workflows/ci.yml@refs/heads/main \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com \
  ghcr.io/lucchmielowski/kyverno-cosign-testbed:github-attestation
```

**SBOM (Software Bill of Materials):**

```bash
# Using cosign to verify GitHub SBOM attestation
cosign verify-attestation \
  --type https://spdx.dev/Document \
  --certificate-identity=https://github.com/lucchmielowski/kyverno-cosign-testbed/.github/workflows/ci.yml@refs/heads/main \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com \
  ghcr.io/lucchmielowski/kyverno-cosign-testbed:github-sbom
```

### Cosign Key-Based Signatures

```bash
# v2-traditional (verify with any cosign v2 or v3)
cosign verify --key cosign.pub ghcr.io/lucchmielowski/kyverno-cosign-testbed:v2-traditional

# v3-traditional (verify with cosign v3)
cosign verify --key cosign.pub ghcr.io/lucchmielowski/kyverno-cosign-testbed:v3-traditional

# v3-bundle (verify with cosign v3)
cosign verify --key cosign.pub ghcr.io/lucchmielowski/kyverno-cosign-testbed:v3-bundle
```

### Cosign Keyless Signatures

```bash
# v2-keyless (verify with GitHub Actions OIDC identity)
cosign verify \
  --certificate-identity=https://github.com/lucchmielowski/kyverno-cosign-testbed/.github/workflows/ci.yml@refs/heads/main \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com \
  ghcr.io/lucchmielowski/kyverno-cosign-testbed:v2-keyless

# v3-keyless (verify with GitHub Actions OIDC identity)
cosign verify \
  --certificate-identity=https://github.com/lucchmielowski/kyverno-cosign-testbed/.github/workflows/ci.yml@refs/heads/main \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com \
  ghcr.io/lucchmielowski/kyverno-cosign-testbed:v3-keyless

# v3-rekorv2-keyless (verify with GitHub Actions OIDC identity; entry is logged to a Rekor v2 instance)
cosign verify \
  --certificate-identity=https://github.com/lucchmielowski/kyverno-cosign-testbed/.github/workflows/ci.yml@refs/heads/main \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com \
  ghcr.io/lucchmielowski/kyverno-cosign-testbed:v3-rekorv2-keyless
```

## Understanding Signature Artifacts

### Traditional OCI Signature Manifests (`.sig` images)
Created by: `:v2-traditional`, `:v3-traditional`, `:v3-bundle`

These store the signature as a separate OCI image in the registry with a `.sig` tag suffix. This is the original cosign format and is backward compatible across cosign versions.

**Example:**
- Image: `ghcr.io/lucchmielowski/kyverno-cosign-testbed:v3-traditional`
- Signature: `ghcr.io/lucchmielowski/kyverno-cosign-testbed:sha256-abc123.sig`

### Keyless Signatures (Fulcio + Rekor)
Created by: `:v2-keyless`, `:v3-keyless`, `:v3-rekorv2-keyless`

These use short-lived certificates from Fulcio (certificate authority) and store the signature in Rekor (transparency log). No long-lived signing keys are needed.

**Artifacts:**
- Traditional OCI signature manifest (`.sig` image)
- Fulcio certificate (embedded in signature)
- Rekor transparency log entry

**Note on `:v3-rekorv2-keyless`:** Signed with `cosign sign --signing-config signing-config-with-rekor2.json`, pointing at a Rekor v2 (`majorApiVersion: 2`) log plus a timestamp authority instead of the default Rekor v1 instance. Rekor v2 entries are DSSE-based (`"kind":"dsse","version":"0.0.2"`) and don't carry an integrated/signed-entry timestamp like Rekor v1, so a TSA timestamp is required at signing time. Added to test Kyverno's IVPOL Rekor v2 support — see [kyverno#15557](https://github.com/kyverno/kyverno/issues/15557).

### GitHub Attestations
Created by: `:github-attestation`, `:github-sbom`

Native GitHub attestations stored directly in GitHub's attestation store and signed with GitHub's signing infrastructure.

**`:github-attestation` - Build Provenance:**
- SLSA v1.0 build provenance attestation
- Describes how the image was built (workflow, source, etc.)

**`:github-sbom` - Software Bill of Materials:**
- SPDX format SBOM attestation
- Lists all packages, dependencies, and components in the image
- Generated using Anchore's sbom-action

---

**For setup instructions, see [DEVELOPMENT.md](DEVELOPMENT.md)**

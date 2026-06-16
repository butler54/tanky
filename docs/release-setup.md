# Release Pipeline Setup

This document describes the GitHub configuration required for the tanky CI/CD release pipeline.

## How Releases Work

1. Push a conventional commit to `main` (e.g., `feat: add widget`)
2. `create-release.yml` runs Python Semantic Release, which:
   - Parses commit history for version bumps
   - Creates a git tag (e.g., `v0.1.0`) and GitHub Release
3. The tag push triggers `build-release.yml`, which:
   - Builds multi-arch images (amd64, arm64)
   - Pushes to GHCR (`ghcr.io/butler54/tanky`)
   - Signs with cosign and attests SBOM/provenance

## Required Configuration

### 1. GitHub App (for semantic release)

The release workflow uses a GitHub App token instead of a PAT to create tags and releases. This avoids triggering infinite workflow loops.

**Setup:**

1. Create a GitHub App in your org/account settings
2. Grant it **Contents: Read & Write** permission on the tanky repo
3. Install the App on the `butler54/tanky` repository
4. Note the **App ID** and generate a **Private Key**

**Repository configuration:**

| Type | Name | Value |
|------|------|-------|
| Variable | `RELEASE_APP_ID` | The GitHub App's App ID |
| Secret | `RELEASE_APP_PRIVATE_KEY` | The GitHub App's private key (PEM) |

Set these at **Settings > Secrets and variables > Actions**.

If these are not configured, the release job will skip with a log message: *"Release app credentials not set; skipping semantic release."*

### 2. Cosign (for image signing)

Image signing is **required**. The tanky bootc image ships a container policy (`/etc/containers/policy.json`) that enforces cosign signature verification for all image pulls and `bootc switch`/`bootc upgrade` operations. Without signing configured, the host will reject image updates.

| Type | Name | Value |
|------|------|-------|
| Secret | `COSIGN_PRIVATE_KEY` | The cosign private key |
| Secret | `COSIGN_PASSWORD` | The password for the cosign private key |
| Variable | `COSIGN_PUBLIC_KEY` | The cosign public key (embedded in the image at build time) |

**Generate a cosign key pair:**

```bash
cosign generate-key-pair
```

This creates `cosign.key` (private) and `cosign.pub` (public). Store the private key and password as secrets, and the public key as a variable.

The `COSIGN_PUBLIC_KEY` variable is passed as a build argument during image builds. The Containerfile writes it to `/etc/pki/containers/tanky-cosign.pub`, which the baked-in `policy.json` references for signature verification.

### 3. GHCR Access

The build workflow uses `GITHUB_TOKEN` for GHCR authentication. No additional configuration is needed as long as:

- **Settings > Actions > General > Workflow permissions** is set to "Read and write permissions"
- The repository is the source for the GHCR package (first push creates the package)

## Container Signature Policy

The tanky image ships a strict container policy at `/etc/containers/policy.json`:

- **Default:** All container image pulls require cosign signature verification using the embedded public key at `/etc/pki/containers/tanky-cosign.pub`
- **bootc system updates:** `bootc switch` and `bootc upgrade` also go through this policy, so the tanky image itself must be signed
- **Exempted images:** `ghcr.io/openclaw/openclaw` and `ghcr.io/cgwalters/service-gator` are explicitly allowed without signatures (these are unsigned third-party images used by Quadlet units)

The sigstore attachment lookup for GHCR is configured in `/etc/containers/registries.d/ghcr.io-butler54-tanky.yaml`.

### How it works

1. CI builds the image, passing `COSIGN_PUBLIC_KEY` as a build arg
2. The Containerfile writes the key to `/etc/pki/containers/tanky-cosign.pub`
3. The rootfs overlay installs `policy.json` requiring `sigstoreSigned` verification by default
4. After build, CI signs the image with the private key via cosign
5. On the host, any `podman pull` or `bootc upgrade` verifies the signature against the embedded public key

### What happens without signing

If cosign secrets are not configured:
- The image still builds and pushes to GHCR
- But the image will **not** be signed
- A running host with the enforced policy will **reject** `bootc upgrade` or `podman pull` for `ghcr.io/butler54/tanky` because no valid signature exists

## Key Rotation

To rotate the cosign keypair:

1. Generate a new keypair: `cosign generate-key-pair`
2. Update the GitHub secrets (`COSIGN_PRIVATE_KEY`, `COSIGN_PASSWORD`) and variable (`COSIGN_PUBLIC_KEY`)
3. Trigger a new release — the new public key is embedded in the image at build time
4. After the host updates to the new image (via `bootc upgrade`), it will have the new public key and accept images signed with the new private key

**Important:** The old image on the host still has the old public key. The transition works because the new image is signed with the **new** key and the new image also embeds the **new** public key. The first upgrade after rotation must be done before removing the old key from signing, or the host will reject the update.

## Verifying the Setup

After configuring secrets, push a conventional commit to `main`:

```bash
git commit --allow-empty -m "feat: trigger first release"
git push origin main
```

Check the **Actions** tab for:
1. "Validate Build and Create Release" — should create a `v0.0.1` tag
2. "Build and Release Container Image" — triggered by the tag, builds and pushes to GHCR

## Manual Verification

Verify a published image signature from any machine with cosign installed:

```bash
cosign verify --key cosign.pub ghcr.io/butler54/tanky:latest
```

On a running tanky host, the embedded key is at `/etc/pki/containers/tanky-cosign.pub`:

```bash
cosign verify --key /etc/pki/containers/tanky-cosign.pub ghcr.io/butler54/tanky:latest
```

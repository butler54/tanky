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

Image signing is optional but recommended. If cosign secrets are not configured, the build still succeeds — signing and verification steps are skipped.

| Type | Name | Value |
|------|------|-------|
| Secret | `COSIGN_PRIVATE_KEY` | The cosign private key |
| Secret | `COSIGN_PASSWORD` | The password for the cosign private key |
| Variable | `COSIGN_PUBLIC_KEY` | The cosign public key (for verification) |

**Generate a cosign key pair:**

```bash
cosign generate-key-pair
```

This creates `cosign.key` (private) and `cosign.pub` (public). Store the private key and password as secrets, and the public key as a variable.

### 3. GHCR Access

The build workflow uses `GITHUB_TOKEN` for GHCR authentication. No additional configuration is needed as long as:

- **Settings > Actions > General > Workflow permissions** is set to "Read and write permissions"
- The repository is the source for the GHCR package (first push creates the package)

## Verifying the Setup

After configuring secrets, push a conventional commit to `main`:

```bash
git commit --allow-empty -m "feat: trigger first release"
git push origin main
```

Check the **Actions** tab for:
1. "Validate Build and Create Release" — should create a `v0.0.1` tag
2. "Build and Release Container Image" — triggered by the tag, builds and pushes to GHCR

## Local Verification

After a release, verify the published image:

```bash
podman pull ghcr.io/butler54/tanky:latest

# If cosign is configured:
make verify COSIGN_PUBLIC_KEY="$(cat cosign.pub)" IMAGE_OWNER=butler54
```

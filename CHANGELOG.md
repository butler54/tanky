# CHANGELOG

<!-- version list -->

## v0.1.2 (2026-06-17)

### Bug Fixes

- Repair cosign signature verification for bootc upgrades
  ([#14](https://github.com/butler54/tanky/pull/14),
  [`6acebb3`](https://github.com/butler54/tanky/commit/6acebb3de19768ba5279924651032f2340f1b285))


## v0.1.1 (2026-06-17)

### Bug Fixes

- Activate network-online.target at boot for Quadlet services
  ([#13](https://github.com/butler54/tanky/pull/13),
  [`f002a81`](https://github.com/butler54/tanky/commit/f002a8198ee40cee11cc53d48fbf13fcbd574a72))

### Chores

- **deps**: Bump actions/checkout from 4.3.1 to 6.0.3
  ([#6](https://github.com/butler54/tanky/pull/6),
  [`0b76ee3`](https://github.com/butler54/tanky/commit/0b76ee338ebbde0a05749f3c7b4d25836c73cd69))

- **deps**: Bump docker/login-action from 4.0.0 to 4.2.0
  ([#7](https://github.com/butler54/tanky/pull/7),
  [`2470605`](https://github.com/butler54/tanky/commit/247060513c6dd72ad1e61ad17be994c4b03a6e9e))

- **deps**: Bump github/codeql-action from 4.35.4 to 4.36.2
  ([#8](https://github.com/butler54/tanky/pull/8),
  [`7e5b107`](https://github.com/butler54/tanky/commit/7e5b107cb0c1e5caf3abf73e8592b06eaaae3dc5))

- **deps**: Bump step-security/harden-runner from 2.19.1 to 2.19.4
  ([#9](https://github.com/butler54/tanky/pull/9),
  [`37b44ce`](https://github.com/butler54/tanky/commit/37b44cecb79c5a263b3ca92c7629e1aea2d4cc77))

### Continuous Integration

- SHA-pin checkout action and pin commitlint versions
  ([#12](https://github.com/butler54/tanky/pull/12),
  [`bfa4360`](https://github.com/butler54/tanky/commit/bfa43601088063777624801285f6e0b7bc940d69))

- Update Cosign and migrate to unified attestation action
  ([#11](https://github.com/butler54/tanky/pull/11),
  [`14ea43b`](https://github.com/butler54/tanky/commit/14ea43bdec5dd0375cf98689a62dcad6d26b05fc))

- **02a**: Migrate build provenance to actions/attest
  ([#11](https://github.com/butler54/tanky/pull/11),
  [`14ea43b`](https://github.com/butler54/tanky/commit/14ea43bdec5dd0375cf98689a62dcad6d26b05fc))

- **02a**: Migrate SBOM attestation to actions/attest
  ([#11](https://github.com/butler54/tanky/pull/11),
  [`14ea43b`](https://github.com/butler54/tanky/commit/14ea43bdec5dd0375cf98689a62dcad6d26b05fc))

- **02a**: Update Cosign from v2.2.4 to v2.6.3 ([#11](https://github.com/butler54/tanky/pull/11),
  [`14ea43b`](https://github.com/butler54/tanky/commit/14ea43bdec5dd0375cf98689a62dcad6d26b05fc))

- **02b**: Pin commitlint npm versions to v20.5.3 ([#12](https://github.com/butler54/tanky/pull/12),
  [`bfa4360`](https://github.com/butler54/tanky/commit/bfa43601088063777624801285f6e0b7bc940d69))

- **02b**: SHA-pin actions/checkout in block-ide-dirs.yml
  ([#12](https://github.com/butler54/tanky/pull/12),
  [`bfa4360`](https://github.com/butler54/tanky/commit/bfa43601088063777624801285f6e0b7bc940d69))


## v0.1.0 (2026-06-16)

- Initial Release

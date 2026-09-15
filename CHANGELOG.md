# Changelog — `armature-storage`

All notable changes to this crate will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this crate adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Earlier changes are recorded in the workspace [`CHANGELOG.md`](../CHANGELOG.md).

## [Unreleased]

### Changed

- Bumped dependencies: `tokio` 1.53, `http` 1.5, `uuid` 1.26, `azure_storage_blob`/`azure_core` 1.1, `google-cloud-storage` 1.18, `google-cloud-gax` 1.14, `google-cloud-auth` 1.16, `base64` 0.23. No source changes were needed -- the existing S3/GCS/Azure backends already targeted the current API surface of each SDK. `aws-sdk-s3` is held at 1.146 (one release back): newer `aws-sdk-*` releases require `aws-smithy-types` 1.7, whose reshaped `Document::Object` does not compile against the `aws-smithy-json` 0.63 that the newest `aws-config` (1.12) still depends on.
- A direct `aws-smithy-types >=1.6.3, <1.7` requirement keeps a fresh resolve on the SDK releases held back above; without it the resolver picks `aws-sdk-*`/`aws-runtime` releases that need `aws-smithy-types` 1.7 and fail to build against `aws-config` 1.12.
- AWS SDK dependencies no longer enable their default features, dropping the SDK's legacy hyper-0.14 client and its `h2 0.3` (RUSTSEC-2026-0258); the hyper-1 `default-https-client` and `rt-tokio` (plus `sigv4a`/`http-1x` where the SDK enabled them by default) are kept.
- The MSRV CI job also checks `--all-features`, so the optional AWS SDK dependencies are built on the MSRV toolchain.

### Added

- Adopted the `storage` criterion benchmark (file validation, metadata, local storage, uploaded-file handling) from the root package's `benches/`. Run it with `cargo bench -p armature-storage --bench storage`. The crate now sets `autobenches = false`, so a new file under `benches/` needs an explicit `[[bench]]` entry.

### Fixed

- The fully-buffered API's size ceiling is documented: every object is materialized in memory in both directions, and single-request `PutObject` caps S3 objects at 5 GiB.

## [0.2.2] - 2026-08-04

### Fixed

- Requirements on sibling armature crates name a minor instead of `0`. Under
  Cargo's 0.x rules `version = "0"` matches any release ever made, and edition
  2024 selects the MSRV-aware resolver, so a consumer declaring an older
  `rust-version` was handed the oldest version satisfying it — resolving
  `armature-core = "0"` on Rust 1.89 produced `armature-core 0.2.3` while an
  explicit `armature-core = "0.8"` elsewhere in the same graph pulled 0.8.2.
  Two copies of core, and a build failing on symbols the older one lacks. Each
  0.x minor in this family is a breaking change, so the requirement now names
  one. No API change.

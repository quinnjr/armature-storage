# Changelog — `armature-storage`

All notable changes to this crate will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this crate adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Earlier changes are recorded in the workspace [`CHANGELOG.md`](../CHANGELOG.md).

## [Unreleased]

## [0.3.0] - 2026-09-15

### Changed

- **Breaking:** `AzureBlobStorage::from_azure_services` takes `armature_azure::AzureServices`, so the `armature-azure` 0.3 requirement is breaking here and the minor moves.
- **Breaking:** requires `armature-azure` 0.3 (was `0.2`); its types appear in this crate's API, so the requirement change is breaking here and the minor moves. Part of the `armature-core` 0.10 release train.
- Bumped dependencies: `tokio` 1.53, `http` 1.5, `uuid` 1.26, `azure_storage_blob`/`azure_core` 1.1, `google-cloud-storage` 1.18, `google-cloud-gax` 1.14, `google-cloud-auth` 1.16, `base64` 0.23. No source changes were needed -- the existing S3/GCS/Azure backends already targeted the current API surface of each SDK. `aws-sdk-s3` is held at 1.146 (one release back): newer `aws-sdk-*` releases require `aws-smithy-types` 1.7, whose reshaped `Document::Object` does not compile against the `aws-smithy-json` 0.63 that the newest `aws-config` (1.12) still depends on.
- A direct `aws-smithy-types >=1.6.3, <1.7` requirement keeps a fresh resolve on the SDK releases held back above; without it the resolver picks `aws-sdk-*`/`aws-runtime` releases that need `aws-smithy-types` 1.7 and fail to build against `aws-config` 1.12.
- AWS SDK dependencies no longer enable their default features, dropping the SDK's legacy hyper-0.14 client and its `h2 0.3` (RUSTSEC-2026-0258); the hyper-1 `default-https-client` and `rt-tokio` (plus `sigv4a`/`http-1x` where the SDK enabled them by default) are kept.
- The MSRV CI job also checks `--all-features`, so the optional AWS SDK dependencies are built on the MSRV toolchain.

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

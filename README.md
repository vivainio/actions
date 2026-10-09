# actions

Reusable GitHub Actions for Python projects.

## publish-pypi-full

Checkout, build with uv, and publish to PyPI using trusted publishing.

```yaml
name: Publish to PyPI

on:
  release:
    types: [published]

jobs:
  publish:
    runs-on: ubuntu-latest
    environment: pypi
    permissions:
      id-token: write
    steps:
      - uses: vivainio/actions/publish-pypi-full@main
```

Requires PyPI trusted publisher configured for your repo.

## maturin-wheels reusable workflow

Build Rust CLI packages with maturin (`bindings = "bin"`). The workflow owns
version setting, the platform matrix, and artifact uploads. It checks out the
calling repository and strips a leading `v` from the version. Use it on release
tags, or pass an explicit Cargo-compatible `version` for other events.

```yaml
name: Release wheels
on:
  push:
    tags: ['v*']
jobs:
  wheels:
    uses: vivainio/actions/.github/workflows/maturin-wheels.yml@main

  publish:
    needs: wheels
    runs-on: ubuntu-latest
    environment: pypi
    permissions:
      id-token: write
      actions: read
    steps:
      - uses: vivainio/actions/publish-pypi-artifacts@main
```

Keep your existing tests and add `needs: test` to the calling `wheels` job where
appropriate. Publishing uses the calling repo's PyPI trusted publisher and
`pypi` environment. Pin the shared workflow to a commit for reproducible use.

| Input | Default | Purpose |
| --- | --- | --- |
| `version` | Caller ref name | Cargo package version; optional leading `v` |
| `platforms` | `desktop` | `desktop` or `linux`; desktop adds Windows x64 and macOS x64/ARM64 to Linux x64 |
| `linux-arm64` | `false` | Add native Linux ARM64 builds |
| `android` | `false` | Add Android ARM64, API 24, with manylinux off |
| `sdist` | `false` | Also upload a source distribution |
| `manylinux` | `auto` | Linux wheel compatibility |
| `maturin-version` | Unspecified | Use maturin-action's default, or pin a version |
| `build-args` | Empty | Additional maturin build arguments |
| `linux-cflags` | Empty | CFLAGS for Linux builds, excluding Android |

For tempkeys, use `platforms: linux`, `linux-arm64: true`, `manylinux: '2_28'`,
and `sdist: true`. For zipget, use `linux-arm64: true`. For leo-cub, use
`android: true`, `linux-cflags: -D_BSD_SOURCE`, and `maturin-version: v1.12.4`.

Artifacts are named `wheels-<target>` and `wheels-sdist`. The workflow builds
with `--release --locked`; commit Cargo.lock. It currently expects Cargo.toml
and pyproject.toml at the repository root and does not run package tests.

## publish-pypi-artifacts

Download `wheels-*` artifacts from the current workflow run, check that wheel
or source distributions exist, and publish them through PyPI Trusted Publishing.
Publishing uses `uv publish --trusted-publishing always`, which requires OIDC
authentication and does not fall back to stored credentials.
Use the local publishing job shown above; no checkout or API token is needed.
The job must run on Linux, declare `id-token: write` and `actions: read`, and use the environment
configured in its Trusted Publisher (the examples use `pypi`).

Configure PyPI with the **app repository and its caller workflow filename**,
for example `vivainio/unxml-rs`, `release.yml`, and environment `pypi`.
The composite action runs inside that local job and preserves its OIDC identity.
Keep the publishing job in the app: PyPI currently does not support publishing
from reusable workflows. See [PyPI's reusable workflow limitation](https://docs.pypi.org/trusted-publishers/troubleshooting/#reusable-workflows-on-github).

| Input | Default | Purpose |
| --- | --- | --- |
| `artifact-pattern` | `wheels-*` | Artifacts to download and merge |
| `packages-dir` | `dist` | Download directory and publish source |
| `run-id` | Current run | Download artifacts from an earlier run to retry publishing |
| `repository-url` | `https://upload.pypi.org/legacy/` | Override for TestPyPI or another index |

For TestPyPI, pass `repository-url: https://test.pypi.org/legacy/` and configure
a Trusted Publisher there as well. Use a fresh publishing job directory so
only the intended distributions are uploaded.

The uv publisher currently uploads distributions without PEP 740 attestations.

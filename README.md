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

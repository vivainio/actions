# actions

Reusable GitHub Action for publishing Python packages to PyPI.

## Usage

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
      - uses: vivainio/actions@main
```

Requires PyPI trusted publisher configured for your repo.

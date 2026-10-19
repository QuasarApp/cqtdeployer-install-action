# CQtDeployer Install Action

A GitHub Action to automatically download, extract, and install **CQtDeployer** for Linux (x64 / ARM64), Windows (x64) and macOS (ARM64) runners.

This action eliminates boilerplate code in your workflows, allowing you to install CQtDeployer with a single line of code across different operating systems and CPU architectures.


## Usage

Add the following step to your GitHub Actions workflow file (e.g., `.github/workflows/build.yml`):

```yaml
- name: Install CQtDeployer
  uses: QuasarApp/cqtdeployer-install-action@v1.1.0
```

By default the step installs the latest stable **CQtDeployer** release.
Starting with **v1.1.0** the downloaded release can be controlled with the
`latest` and `version` options (see [Options](#options)).


## Options

| Option    | Type    | Default   | Description |
|-----------|---------|-----------|-------------|
| `latest`  | boolean | `true`    | Download the latest stable release. Only consulted when `version` is not set. |
| `version` | string  | *(empty)* | Download a specific release tag instead of the latest release, e.g. `v1.6.2365`. When set, it takes precedence over `latest`. |

By default the action downloads the latest stable **CQtDeployer** release. To pin
a specific release, set the `version` input to the desired release tag:

```yaml
- name: Install a specific CQtDeployer version
  uses: QuasarApp/cqtdeployer-install-action@v1.1.0
  with:
    version: v1.6.2365
```

> **Note:** `version` must be the exact GitHub release **tag** (for example
> `v1.6.2365`). When `version` is set, the `latest` flag is ignored.

> **macOS:** the CQtDeployer releases ship the macOS package as
> `*_Darwin_arm64.zip` (a flat archive with the `CQtDeployer` executable, the
> `CQtDeployer.sh` launcher and the Qt frameworks). The first release with the
> macOS assets (`v1.6.2504.659185d`) is a pre-release, and the GitHub
> "latest release" endpoint ignores pre-releases, so when `version` is not set
> the action resolves the macOS tag by selecting the newest release that
> ships the `Darwin_arm64` package.

## Full Workflow Example
Here is a complete example of how to use this action in a matrix build for both Linux and Windows:

```yaml

name: CI Build and Deploy

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

env:
  # Leave empty to install the latest stable release,
  # or set an exact release tag (e.g. "v1.6.2365") to pin the version.
  CQT_VERSION: ""

jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
    runs-on: ${{ matrix.os }}

    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v4

      # 1. Install CQtDeployer using this custom action.
      #    With an empty CQT_VERSION the latest stable release is installed.
      #    To pin a specific release, set CQT_VERSION to the exact release
      #    tag (e.g. "v1.6.2365") or pass "version: <tag>" directly.
      - name: Setup CQtDeployer
        uses: QuasarApp/cqtdeployer-install-action@v1.1.0
        with:
          version: ${{ env.CQT_VERSION }}

      # 2. Run deployment for your Qt Application (Linux)
      - name: Deploy Application (Linux)
        if: runner.os == 'Linux'
        shell: bash
        run: |
          cqtdeployer -bin my_qt_app -qmake /path/to/qt/bin/qmake

      # 3. Run deployment for your Qt Application (Windows)
      - name: Deploy Application (Windows)
        if: runner.os == 'Windows'
        shell: pwsh
        run: |
          cqtdeployer -bin my_qt_app.exe -qmake C:\path\to\qt\bin\qmake.exe
```

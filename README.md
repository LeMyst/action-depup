# depup - Action which updates dependencies automatically

[![Test](https://github.com/reviewdog/action-depup/workflows/Test/badge.svg)](https://github.com/reviewdog/action-depup/actions?query=workflow%3ATest)
[![reviewdog](https://github.com/reviewdog/action-depup/workflows/reviewdog/badge.svg)](https://github.com/reviewdog/action-depup/actions?query=workflow%3Areviewdog)
[![depup](https://github.com/reviewdog/action-depup/workflows/depup/badge.svg?branch=master&event=push)](https://github.com/reviewdog/action-depup/actions?query=workflow%3Adepup+event%3Apush+branch%3Amaster)
[![release](https://github.com/reviewdog/action-depup/workflows/release/badge.svg)](https://github.com/reviewdog/action-depup/actions?query=workflow%3Arelease)
[![GitHub release (latest SemVer)](https://img.shields.io/github/v/release/reviewdog/action-depup?logo=github&sort=semver)](https://github.com/reviewdog/action-depup/releases)
[![action-bumpr supported](https://img.shields.io/badge/bumpr-supported-ff69b4?logo=github&link=https://github.com/haya14busa/action-bumpr)](https://github.com/haya14busa/action-bumpr)

depup action updates version in a given file automatically.

[![demo](https://user-images.githubusercontent.com/3797062/72677595-7ac4ec80-3ae1-11ea-8b49-163bb72f822c.png)](https://github.com/reviewdog/action-depup/pull/4)

**Supported patterns example:**

```
REVIEWDOG_VERSION=0.1.0
# v prefix is supported as well.
REVIEWDOG_VERSION=v0.1.0
```

```Dockerfile
# Dockerfile sample
ENV REVIEWDOG_VERSION=0.1.0
# space is supported as well.
ENV REVIEWDOG_VERSION 0.1.0
ARG REVIEWDOG_VERSION=0.1.0
```

```yaml
# yaml sample
yaml:
  REVIEWDOG_VERSION: 0.1.0
```

demo: https://github.com/reviewdog/action-depup/pull/4

## Inputs

```yaml
inputs:
  github_token:
    description: "GITHUB_TOKEN to get latest version with GitHub Release API"
    default: "${{ github.token }}"
  file:
    description: "target file"
    required: true
  version_name:
    description: "target version name. e.g. REVIEWDOG_VERSION"
    required: true
  repo:
    description: "target GitHub repository. e.g. reviewdog/reviewdog"
    required: true
  tag:
    description: "Check tags instead of releases."
    default: "false"
    required: false
  allow-prerelease:
    description: 'Allow releases tagged as prerelease. Does not work with tags.'
    default: 'false'
    required: false
```

## Example usage

### [.github/workflows/depup.yml](.github/workflows/depup.yml)

```yml
name: depup
on:
  schedule:
    - cron: "14 14 * * *"
  workflow_dispatch:

jobs:
  reviewdog:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: reviewdog/action-depup@v1
        id: depup
        with:
          file: testdata/testfile
          version_name: REVIEWDOG_VERSION
          repo: reviewdog/reviewdog

      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v3
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          title: "chore(deps): update ${{ steps.depup.outputs.repo }} to ${{ steps.depup.outputs.latest }}"
          commit-message: "chore(deps): update ${{ steps.depup.outputs.repo }} to ${{ steps.depup.outputs.latest }}"
          body: |
            Update ${{ steps.depup.outputs.repo }} to [${{ steps.depup.outputs.latest }}](https://github.com/${{ steps.depup.outputs.repo }}/releases/tag/v${{ steps.depup.outputs.latest }})

            This PR is auto generated by [depup workflow](https://github.com/${{ github.repository }}/actions?query=workflow%3A${{ github.workflow }}).
          branch: depup/${{ steps.depup.outputs.repo }}
```

### Run depup, and then create PR

If you want to create a PR after, you can use `reviewdog/action-depup/with-pr@v1` action.

```yml
name: depup
on:
  schedule:
    - cron: "14 14 * * *"
  workflow_dispatch:

jobs:
  reviewdog:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: reviewdog/action-depup/with-pr@v1
        with:
          file: testdata/testfile
          version_name: REVIEWDOG_VERSION
          repo: reviewdog/reviewdog
```

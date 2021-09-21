# release-please-manifest-demo

This is a demo of using [release-please](https://github.com/googleapis/release-please) with a [manifest](https://github.com/googleapis/release-please/blob/main/docs/manifest-releaser.md) for monorepo packages (Node.js).

## Config

Create empty [`.release-please-manifest.json`](https://github.com/googleapis/release-please/blob/main/docs/manifest-releaser.md#quick-start):

```sh
echo '{}' > .release-please-manifest.json
```

Create `release-please-config.json` with [bootstrap-sha](https://github.com/googleapis/release-please/blob/main/docs/manifest-releaser.md#starting-commit):

```json
{
  "bootstrap-sha": "555bc4fb394409cb4ef342c7bb151c6e88d0d351"
}
```

For a new repo, use the first commit sha:

```sh
git rev-list HEAD | tail -n 1
```

## Packages

Given the directory structure:

```
.
└── packages
    └── package-a
        └── package.json

2 directories, 1 file
```

Add the package to release-please-config.json`:

```json
{
  "bootstrap-sha": "555bc4fb394409cb4ef342c7bb151c6e88d0d351",
  "packages": {
    "packages/package-a": {}
  }
}
```

By default, the first release will always set the version to `1.0.0`.

To change the first release-as version:

```json
{
  "bootstrap-sha": "555bc4fb394409cb4ef342c7bb151c6e88d0d351",
  "packages": {
    "packages/package-a": {
      "release-as": "2.0.0"
    }
  }
}
```

## GitHub Actions

Create `.github/workflows/release-please.yml`:

```yml
on:
  push:
    branches:
      - master

name: release-please
jobs:
  release-please:
    runs-on: ubuntu-latest

    steps:
      - name: Release Please
        uses: GoogleCloudPlatform/release-please-action@v2
        id: release
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          command: manifest

      # true for single package
      - name: Echo release_created
        run: echo "${{ steps.release.outputs.release_created }}"

      # true for multiple packages
      - name: Echo releases_created
        run: echo "${{ steps.release.outputs.releases_created }}"
```

If you committed changes to your package with one of the following in your [commit message](https://github.com/googleapis/release-please#how-should-i-write-my-commits):

- feat
- fix
- BREAKING CHANGE
- Release-As

The `release-please` workflow will generate a release PR. Merge the PR to `master` and the `release-please-action` step may generate outputs.

`release_created` will be `true` if there is a single package:

```yml
- if: ${{ steps.release.outputs.release_created }}
```

`releases_created` will be `true` if there are multiple packages:

```yml
- if: ${{ steps.release.outputs.releases_created }}
```

## Shortcomings

[Manifest releaser does not update dependent modules](https://github.com/googleapis/release-please/issues/1032)

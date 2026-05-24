<p align="left">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/logo-dark.svg">
    <img src="assets/logo-light.svg" alt="Trestle logo" height="18">
  </picture>
</p>

Scan your repository for leaked secrets on every push and pull request, using
the [Trestle][trestle] Community secret scanner.

Trestle detects API keys, access tokens, passwords, private keys, and
certificates. It runs as a single self-contained
binary downloaded from the official GitHub release, with the download verified
against a sha256 checksum published with each release.

[trestle]: https://trestlescan.com

## Quick start

```yaml
name: Secret scan

on:
  push:
  pull_request:

jobs:
  trestle:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: toro-guapo/trestle-action@v1
```

## Upload results to the GitHub Security tab

Trestle can emit [SARIF][sarif], which GitHub renders in the Security tab as
code scanning alerts.

```yaml
jobs:
  trestle:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4
      - uses: toro-guapo/trestle-action@v1
        with:
          args: --output-format=sarif --output-file=trestle.sarif
          fail-on-findings: false
        id: trestle
      - uses: github/codeql-action/upload-sarif@v4
        if: always()
        with:
          sarif_file: trestle.sarif
```

[sarif]: https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/sarif-support-for-code-scanning

## Inputs

| Name               | Default              | Description |
| ------------------ | -------------------- | ----------- |
| `version`          | `latest`             | Trestle Community version, such as `1.2.3`, or `latest`. |
| `path`             | `.`                  | Path to scan. |
| `args`             | _empty_              | Extra arguments passed to `trestle scan`. Space-separated; values cannot contain spaces. |
| `fail-on-findings` | `true`               | When `true`, fail the job if any secrets are found. |
| `repository`       | `toro-guapo/trestle` | GitHub repository to download trestle from. |

## Outputs

| Name           | Description |
| -------------- | ----------- |
| `exit-code`    | Trestle exit code: `0` (no findings), `1` (findings), `2` (error). |
| `has-findings` | `"true"` if any findings were reported, otherwise `"false"`. |

## Supported runners

The action runs on the GitHub-hosted runners listed below. Self-hosted runners
on the same operating system and architecture also work.

| Operating system | Architecture | Example runner labels        |
| ---------------- | ------------ | ---------------------------- |
| Linux            | x86-64       | `ubuntu-latest`, `ubuntu-24.04` |
| Linux            | arm64        | `ubuntu-24.04-arm`           |
| macOS            | arm64        | `macos-latest`, `macos-14`   |
| macOS            | x86-64       | `macos-13`                   |
| Windows          | x86-64       | `windows-latest`             |
| Windows          | arm64        | `windows-11-arm`             |

## How the binary is verified

For each release of Trestle, a `checksums.txt` file is published alongside the
platform binaries. The action downloads both files from the same release tag
and rejects the binary if its sha256 hash does not match the entry in
`checksums.txt`.

The action requires a release that ships `checksums.txt`.

## License

Apache License 2.0. See [LICENSE](LICENSE).

Issues and discussion are welcome on GitHub. Pull requests are not accepted at
this time.

# mcpb-pack

A GitHub Action that packs a compiled MCP server as an [MCP Bundle](https://github.com/modelcontextprotocol/mcpb)
(`.mcpb`), the one-file install Claude Desktop opens with a click. It runs the
[`mcpb`](https://github.com/Synphonyte/mcpb) command, so no Node or official `mcpb` CLI is needed.

A binary only runs on the OS and CPU it was built for, so the action packs the binary a job just
built for that job's runner, and a matrix of runners gives one bundle per platform:

```yaml
jobs:
  bundle:
    strategy:
      matrix:
        os: [ubuntu-latest, ubuntu-24.04-arm, macos-latest, windows-latest, windows-11-arm]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v6
      - run: cargo build --release --locked
      - uses: Synphonyte/mcpb-pack@v1
        id: mcpb
        with:
          executable: target/release/my-server # `.exe` is added on Windows
          out: dist/
      - uses: actions/upload-artifact@v4
        with:
          name: ${{ steps.mcpb.outputs.name }}
          path: ${{ steps.mcpb.outputs.bundle }}
```

The action packs `manifest.json` for the runner's platform only, checks the bundle and prints what
it holds. [mcpb-template](https://github.com/Synphonyte/mcpb-template) shows the whole thing: a
Rust server built for six targets, then one GitHub release with every bundle.

## Inputs

| Input | Default | |
| --- | --- | --- |
| `manifest` | `manifest.json` | the manifest, or its directory; its paths are relative to it |
| `executable` | the entry point | the built binary, relative to the workspace |
| `platform` | the runner's OS | `darwin`, `linux` or `win32` |
| `arch` | the runner's CPU | `x64`, `arm64` …; set it when cross-compiling, as for `x86_64-apple-darwin` on an arm64 Mac |
| `out` | the bundle's name | a file, or a directory ending in `/`; the bundle is named `<name>-<version>-<platform>-<arch>.mcpb` |
| `args` | | more `mcpb pack` arguments, such as `--tree assets --data data` |
| `mcpb-version` | `latest` | a release of `mcpb` such as `0.3.0`, or `git` for its newest commit |

## Outputs

| Output | |
| --- | --- |
| `bundle` | the bundle's path |
| `name` | its file name, which suits an artifact name |

The action downloads the `mcpb` binary of that release, and builds it with cargo where the release
has none for the runner.

## Releasing

Tag and push; `.github/workflows/release.yml` moves `v1` to the tag and creates a GitHub release:

```sh
git tag -a v1.0.0 -m v1.0.0
git push origin v1.0.0
```

To list a release on the GitHub Marketplace, edit it on GitHub and tick **Publish this Action to
the GitHub Marketplace**. The first time, that asks you to accept the Marketplace developer
agreement, needs two-factor authentication, and checks the action's `name` is not taken.

## License

MIT — see [LICENSE](LICENSE).

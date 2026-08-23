# Installation

The `mlx` CLI is a Python package that installs with uv, directly from the platform's GitHub repository.
It runs on Linux, macOS, and Windows, and needs no platform-specific setup.

## Prerequisites

| Requirement | Notes |
|---|---|
| Python 3.12 or newer | The CLI is a standard Python package |
| [uv](https://docs.astral.sh/uv/) | Used to install and update the CLI |
| An API server URL and token | Provided by your platform team; needed only to use the CLI, not to install it |

If you do not have uv yet:

```shell
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## Install the CLI

Install the CLI as a uv tool from the repository your platform team points you at:

```shell
uv tool install git+https://github.com/<your-org>/ml-platform.git
```

Verify the installation:

```shell
mlx --version
```

The `mlx` binary is now on your `PATH` and usable from any directory.

## Update the CLI

```shell
uv tool upgrade mlx
```

## Uninstall the CLI

```shell
uv tool uninstall mlx
```

## Shell completion

The CLI can emit completion scripts for bash, zsh, and fish:

```shell
mlx completion bash
mlx completion zsh
mlx completion fish
```

To install completions for bash, append the script to your `~/.bashrc`:

```shell
echo 'eval "$(mlx completion bash)"' >> ~/.bashrc
```

For fish, write the script to the completions directory:

```shell
mlx completion fish > ~/.config/fish/completions/mlx.fish
```

See [`mlx completion`](reference/cli/completion.md) for details.

## Next steps

1. Walk through the [Quickstart](quickstart.md).
2. Read the [CLI conventions](guides/cli-conventions.md) so the command surface feels predictable.

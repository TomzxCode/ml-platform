# mlx completion

Shell completion scripts, generated from the registered command tree so they stay in sync with the CLI.

## Usage

```shell
mlx completion <shell>
```

| Argument | Values |
|---|---|
| shell | `bash`, `zsh`, `fish` |

An unsupported shell name exits 1 and lists the supported shells.

## Installing

The script goes to stdout as-is, since it is consumed by the shell:

```shell
echo 'eval "$(mlx completion bash)"' >> ~/.bashrc
mlx completion fish > ~/.config/fish/completions/mlx.fish
```

See [Installation](../../installation.md#shell-completion).

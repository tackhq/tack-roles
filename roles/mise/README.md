# mise

Installs [mise](https://mise.jdx.dev) — a polyglot tool version manager —
from its signed apt repository, and gives each listed user a ready
toolchain: languages (Go, Node.js, Python, …) and developer CLIs (task,
golangci-lint, uv, …), all version-pinnable per project.

## What it does

1. Adds mise's apt repository (`/etc/apt/sources.list.d/mise.sources`, key
   in `/etc/apt/keyrings`) and installs `mise`.
2. For each user in `mise_users`:
   - puts `~/.local/share/mise/shims` on `PATH` for **every** shell —
     `~/.zshenv` for zsh, the top of `~/.bashrc` for bash (before Ubuntu's
     non-interactive guard) — so tools also work in non-interactive SSH
     commands such as `ssh host go test ./...` or `pmox exec vm -- task test`;
   - installs the global tool set with `mise use -g`, recorded in
     `~/.config/mise/config.toml`.

Shims pick versions per directory, so a project's own `mise.toml`
(`mise use go@1.25` inside the repo) overrides the global set — run
`mise install` there once to fetch them.

## Requirements

- Ubuntu (uses `apt`).
- Root / privilege escalation.
- `mise_users` must already exist.
- Internet access from the host (tools download from their upstreams).

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `mise_users` | `[]` | Users to set up. Empty installs mise only. |
| `mise_tools` | `go@latest node@lts python@3.12 uv@latest task@latest golangci-lint@latest` | Space-separated `tool@version` list installed globally per user. Any `mise registry` tool works, plus `npm:`, `pipx:`, `go:`, `cargo:` packages. |
| `mise_home_base` | `/home` | Parent of the users' home directories. |
| `mise_keyrings_dir` | `/etc/apt/keyrings` | Where the apt signing key is stored. |
| `mise_key_url` | `https://mise.jdx.dev/gpg-key.pub` | mise's apt signing key. |

The repository architecture is derived from `facts.arch`.

## Example

```yaml
name: Go + Python + Node dev box
hosts: all
sudo: true
vars:
  mise_users: [ubuntu]
  mise_tools: "go@1.25 node@22 python@3.12 uv@latest pnpm@latest task@latest golangci-lint@latest npm:@fission-ai/openspec@latest"
roles:
  - role: https://github.com/tackhq/tack-roles.git//roles/mise
    tags: [mise]
```

Re-running converges: already-installed tools are skipped and new entries
in `mise_tools` are added.

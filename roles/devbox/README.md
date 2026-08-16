# devbox

Sets up a developer environment on Ubuntu: CLI tools, editors, terminal
multiplexers, and (opt-in, per user) a zsh + oh-my-zsh + vim configuration.

By default the role installs packages only. Add users to `devbox_users` to also
configure their shell and editor.

## What it does

**Always (system packages):**
- Installs a curated CLI toolset: `zsh`, `vim`, `neovim`, `tmux`, `git`,
  `fd-find`, `silversearcher-ag`, `ripgrep`, `fzf`, `git-delta`, `bat`, `gh`,
  `zoxide`, `eza`, `jq`, `htop`, `tree`, `direnv`, and build essentials.
- Symlinks `fd` → `fdfind` and `bat` → `batcat` onto the PATH.
- Installs `zellij` from its official GitHub release (not in apt).

**Per user in `devbox_users`:**
- Installs oh-my-zsh + the powerlevel10k theme + the `zsh-fzf-history-search`
  plugin.
- Writes a managed `~/.zshrc` (theme + plugin set) and sets zsh as the login
  shell. An existing `.zshrc` is backed up once to `.zshrc.orig`.
- Installs vim-plug and a managed `~/.vimrc` with a lightweight plugin set, then
  runs `:PlugInstall`. An existing `.vimrc` is backed up once to `.vimrc.orig`.

The vim config is intentionally **not an IDE** — editing-comfort plugins only
(NERDTree, fugitive, gitgutter, commentary, surround, fzf.vim, lightline,
editorconfig, polyglot, gruvbox). No LSP or completion engines.

## Requirements

- Ubuntu 24.04 (noble) or newer — all packages come from `universe`.
- Root / privilege escalation.
- `devbox_users` must already exist and have home directories under
  `devbox_home_base`.

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `devbox_packages` | see [`defaults`](defaults/main.yaml) | System packages to install. |
| `devbox_bin_dir` | `/usr/local/bin` | PATH dir for the fd/bat symlinks and zellij. |
| `devbox_zellij_arch` | `x86_64` | zellij release arch (`x86_64` or `aarch64`). |
| `devbox_zellij_url` | latest musl release | Override to pin a zellij version. |
| `devbox_users` | `[]` | Users to configure. Empty = packages only. |
| `devbox_home_base` | `/home` | Base path for user home directories. |
| `devbox_login_shell` | `/usr/bin/zsh` | Login shell set for each user. |
| `devbox_zsh_theme` | `powerlevel10k/powerlevel10k` | oh-my-zsh theme. |
| `devbox_zsh_plugins` | `git terraform kubectl aws wd zsh-fzf-history-search` | Space-separated oh-my-zsh plugin list. |
| `devbox_omz_repo` / `devbox_p10k_repo` / `devbox_fzf_hist_repo` | upstream URLs | Sources for oh-my-zsh, p10k, and the history plugin. |

## Notes

- **Install-once config.** oh-my-zsh, p10k, plugins, and vim-plug are cloned
  with a `creates:` guard, so they aren't auto-updated on later runs — stable,
  pinned config. Update them out of band (or remove the dir to re-clone).
- **powerlevel10k prompt.** Run `p10k configure` once per user to generate
  `~/.p10k.zsh`; the managed `.zshrc` sources it if present.
- **neovim** is installed but left stock. To reuse the vim config, add an
  `~/.config/nvim/init.vim` containing `source ~/.vimrc` (and point vim-plug at
  nvim's paths) — left to the user.

## Example

```yaml
name: Set up a dev box
hosts: workstations
roles:
  - role: https://github.com/tackhq/tack-roles.git//devbox
    vars:
      devbox_users:
        - eugene
```

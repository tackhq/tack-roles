# tailscale

Installs [Tailscale](https://tailscale.com/) on Ubuntu from the official apt
repository, enables the `tailscaled` service, and optionally joins a tailnet.

## What it does

1. Adds Tailscale's GPG key to `/usr/share/keyrings/tailscale-archive-keyring.gpg`.
2. Writes the apt source to `/etc/apt/sources.list.d/tailscale.list`.
3. Installs the `tailscale` package.
4. Enables and starts `tailscaled`.
5. If `tailscale_authkey` is set, runs `tailscale up` to join a tailnet
   (skipped when the node is already connected).

By default (no auth key) it just installs the client and daemon — you run
`tailscale up` yourself.

## Requirements

- Ubuntu (uses `apt` and `systemd`).
- Root / privilege escalation.

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `tailscale_apt_release` | `noble` | Ubuntu codename for the apt repo. Tack has no codename fact — set this for non-24.04 releases (e.g. `jammy`). |
| `tailscale_keyring` | `/usr/share/keyrings/tailscale-archive-keyring.gpg` | Keyring path referenced by the apt source. |
| `tailscale_packages` | `[tailscale]` | Packages to install. |
| `tailscale_service_state` | `started` | Desired `tailscaled` state. |
| `tailscale_service_enabled` | `true` | Start `tailscaled` on boot. |
| `tailscale_authkey` | `""` | Auth key to join a tailnet non-interactively. **Secret** — supply from a vault, never commit. Empty = install only. |
| `tailscale_up_args` | `""` | Extra `tailscale up` flags, e.g. `--ssh --accept-routes --hostname=web1`. |

## Example

```yaml
name: Install Tailscale
hosts: servers
roles:
  - role: https://github.com/tackhq/tack-roles.git//roles/tailscale
    vars:
      tailscale_authkey: "{{ my_tailscale_authkey }}"
      tailscale_up_args: "--ssh --accept-routes"
```

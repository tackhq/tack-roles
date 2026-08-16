# wireguard

Installs [WireGuard](https://www.wireguard.com/) and its tooling
(`wireguard`, `wireguard-tools`) on Ubuntu.

This role only installs the packages — it does not deploy tunnel configs or
manage `wg-quick@` services. Bring your own `/etc/wireguard/*.conf` and manage
the interface however suits your setup.

## Requirements

- Ubuntu (uses `apt`).
- Root / privilege escalation for package installation.

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `wireguard_packages` | `[wireguard, wireguard-tools]` | Packages to install. |

## Example

```yaml
name: Install WireGuard
hosts: clients
roles:
  - role: https://github.com/tackhq/tack-roles.git//wireguard
```

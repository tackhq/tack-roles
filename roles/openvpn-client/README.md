# openvpn-client

Installs the [OpenVPN](https://openvpn.net/) client (`openvpn` package) on
Ubuntu.

This role only installs the package — it does not deploy client profiles or
manage `openvpn-client@` services. Bring your own
`/etc/openvpn/client/*.conf` and manage the connection however suits your
setup.

## Requirements

- Ubuntu (uses `apt`).
- Root / privilege escalation for package installation.

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `openvpn_packages` | `[openvpn]` | Packages to install. |

## Example

```yaml
name: Install OpenVPN client
hosts: clients
roles:
  - role: https://github.com/tackhq/tack-roles.git//openvpn-client
```

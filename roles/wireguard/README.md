# wireguard

Sets up a [WireGuard](https://www.wireguard.com/) client (peer) connection on
Ubuntu: installs WireGuard, deploys a tunnel config to
`/etc/wireguard/<interface>.conf`, and brings it up with `wg-quick`.

## What it does

1. Installs `wireguard` and `wireguard-tools`.
2. Writes your tunnel config to `/etc/wireguard/{{ wireguard_interface }}.conf`
   (mode `0600`).
3. Enables and starts `wg-quick@{{ wireguard_interface }}`, and restarts it when
   the config changes.

## Configuration

You must provide the full tunnel config via `wireguard_config` — the `[Interface]`
and `[Peer]` sections you'd normally hand-write. It contains the interface
**PrivateKey**, so pass it from a secret store (vault / encrypted vars); the
role never ships keys and writes the file `0600`.

```yaml
wireguard_config: |
  [Interface]
  PrivateKey = <your-private-key>
  Address = 10.0.0.2/32
  DNS = 10.0.0.1

  [Peer]
  PublicKey = <server-public-key>
  Endpoint = vpn.example.com:51820
  AllowedIPs = 0.0.0.0/0
  PersistentKeepalive = 25
```

> If your config sets `DNS =`, `wg-quick` needs a `resolvconf` provider
> (e.g. the `openresolv` package) present on the host.

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `wireguard_interface` | `wg0` | Interface / config name (`<name>.conf`, `wg-quick@<name>`). |
| `wireguard_config` | `""` | **Required.** Full tunnel config contents. |
| `wireguard_config_dir` | `/etc/wireguard` | Config directory. |
| `wireguard_service_state` | `started` | Desired `wg-quick@` service state. |
| `wireguard_service_enabled` | `true` | Start the tunnel on boot. |

## Example

```yaml
name: Connect to WireGuard
hosts: clients
roles:
  - role: https://github.com/tackhq/tack-roles.git//wireguard
    vars:
      wireguard_interface: wg0
      wireguard_config: "{{ my_wg_config }}"
```

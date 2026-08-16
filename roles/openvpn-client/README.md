# openvpn-client

Sets up an [OpenVPN](https://openvpn.net/) client on Ubuntu: installs OpenVPN,
deploys a client profile to `/etc/openvpn/client/<name>.conf`, and runs it via
the `openvpn-client@<name>` systemd service.

## What it does

1. Installs `openvpn`.
2. Writes your client profile to
   `/etc/openvpn/client/{{ openvpn_client_name }}.conf` (mode `0600`).
3. Enables and starts `openvpn-client@{{ openvpn_client_name }}`, and restarts
   it when the profile changes.

## Configuration

Provide the full client profile via `openvpn_client_config` — the same contents
as a `.ovpn` file, including any inline `<ca>` / `<cert>` / `<key>` /
`<tls-auth>` blocks. It may carry private keys and credentials, so pass it from
a secret store; the role ships no secrets and writes the file `0600`.

Embedding certs and keys inline keeps everything in one profile:

```
client
dev tun
proto udp
remote vpn.example.com 1194
resolv-retry infinite
nobind
remote-cert-tls server
cipher AES-256-GCM
verb 3
<ca>
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
</ca>
<cert>
...
</cert>
<key>
...
</key>
```

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `openvpn_client_name` | `client` | Profile name (`<name>.conf`, `openvpn-client@<name>`). |
| `openvpn_client_config` | `""` | **Required.** Full client profile contents. |
| `openvpn_config_dir` | `/etc/openvpn/client` | Profile directory. |
| `openvpn_service_state` | `started` | Desired service state. |
| `openvpn_service_enabled` | `true` | Connect on boot. |

## Example

```yaml
name: Connect to OpenVPN
hosts: clients
roles:
  - role: https://github.com/tackhq/tack-roles.git//openvpn-client
    vars:
      openvpn_client_name: work
      openvpn_client_config: "{{ work_ovpn_profile }}"
```

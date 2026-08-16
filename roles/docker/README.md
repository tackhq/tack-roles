# docker

Installs [Docker Engine](https://docs.docker.com/engine/install/ubuntu/) on
Ubuntu from Docker's **official apt repository** — `docker-ce`, the CLI,
`containerd.io`, and the Buildx and Compose plugins.

Targets the latest Ubuntu LTS by default and is idempotent: the GPG key is
only downloaded once, and the repository, packages, and service state converge
on repeated runs.

## What it does

1. Installs `ca-certificates` and `curl`.
2. Adds Docker's GPG key to `/etc/apt/keyrings/docker.asc`.
3. Writes a deb822 apt source to `/etc/apt/sources.list.d/docker.sources`.
4. Installs Docker Engine and the Buildx/Compose plugins.
5. Enables and starts the `docker` service.
6. Optionally adds users to the `docker` group.

## Requirements

- Ubuntu (uses `apt` and `systemd`).
- Root / privilege escalation, as with any package install.

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `docker_apt_release` | `noble` | Ubuntu codename for the apt repo. Tack has no codename fact — set this for non-24.04 releases (e.g. `jammy`). |
| `docker_apt_channel` | `stable` | Repository channel: `stable`, `test`, or `nightly`. |
| `docker_packages` | see [`defaults`](defaults/main.yaml) | Packages installed from the Docker repo. |
| `docker_service_state` | `started` | Desired `docker` service state. |
| `docker_service_enabled` | `true` | Start `docker` on boot. |
| `docker_users` | `[]` | Users to add to the `docker` group. Takes effect on their next login. |

The repository architecture is derived automatically from `facts.arch`.

## Example

```yaml
name: Install Docker
hosts: servers
roles:
  - role: https://github.com/tackhq/tack-roles.git//docker
    vars:
      docker_users:
        - deploy
```

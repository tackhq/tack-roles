# ai-tools

Provisions an AI stack on Ubuntu: two terminal coding agents plus a self-hosted
MCP gateway. MCP servers run in Docker, and you supply their credentials.

## What it does

- **Claude Code** — installed system-wide from Anthropic's signed apt repo.
- **omp** ([oh-my-pi](https://omp.sh)) — an open-source terminal coding agent;
  self-contained prebuilt binary in `/usr/local/bin`.
- **[MCPJungle](https://github.com/mcpjungle/MCPJungle)** — an MCP gateway that
  puts every MCP server behind one endpoint, `http://<host>:8080/mcp`. It runs
  as a systemd service under `ai_tools_mcp_user`, with SQLite in dev mode.
- **MCP servers** — installs a catalog and `mcp-sync`, then registers the
  servers in `ai_tools_mcp_servers`. Each stdio server runs as a
  `docker run -i --rm` container.
- Optionally points each user's Claude Code at the gateway.

## Requirements

- Ubuntu 24.04 (noble) or newer, with root / privilege escalation.
- **Docker** — apply the [`docker`](../docker) role first.
- `ai_tools_mcp_user` must be set to an existing user. The gateway runs as that
  user and reads credentials from that user's `~/.mcp`.
- Claude Code still needs a per-user login (`claude`). The role installs and
  wires it but does not log in.

## Credentials: `~/.mcp/<name>/env`

Every enabled server gets a directory `~/.mcp/<name>/` (mode 700) in the MCP
user's home. The directory holds an `env` file (mode 600) pre-filled with the
keys the server needs:

```
~/.mcp/
├── servers              # enabled servers (written by mcp-sync)
├── jira/env             # JIRA_URL=… JIRA_USERNAME=… JIRA_API_TOKEN=…
├── aws-api/
│   ├── env              # AWS_REGION=…, keys or AWS_PROFILE
│   ├── config           # optional, mounted into the container at /mcp
│   └── credentials
└── jenkins/env
```

A server is **registered only once its required keys are filled in**. Until
then, the play reports `waiting for credentials`. To finish setup:

1. Fill in the file, as `KEY=value` lines without quotes.
2. Run `mcp-sync` as the MCP user (or re-run the play).

Docker servers read `env` each time they start, so a changed token needs no
re-registration. The directory is also mounted read-only at `/mcp` in the
container, for servers that need more than environment variables.

## Catalog

| Name | Runs as | Required keys |
| --- | --- | --- |
| `context7` | remote HTTP | — |
| `sequential-thinking` | `mcp/sequentialthinking` | — |
| `jira` | `ghcr.io/sooperset/mcp-atlassian:0.23.1` | `JIRA_URL`, `JIRA_USERNAME`, `JIRA_API_TOKEN` (optional `CONFLUENCE_*`) |
| `aws-api` | `public.ecr.aws/awslabs-mcp/awslabs/aws-api-mcp-server:1.5.6` | `AWS_REGION`, plus keys or `AWS_PROFILE` with `config`/`credentials` files |
| `jenkins` | your Jenkins, via the [MCP Server plugin](https://plugins.jenkins.io/mcp-server/) (HTTP) | `JENKINS_URL`, `JENKINS_USER`, `JENKINS_API_TOKEN` |

There is no Jenkins image: the plugin serves MCP from the Jenkins controller.
`mcp-sync` registers `<JENKINS_URL>/mcp-server/mcp` with basic auth. For HTTP
servers, the credentials are stored in the gateway's database. Re-run
`mcp-sync` after rotating the token.

## Your own servers

Create `~/.mcp/<name>/spec`. Any directory with a `spec` is enabled
automatically. The format matches [`files/mcp-catalog`](files/mcp-catalog):

```sh
# ~/.mcp/github/spec
DESCRIPTION="GitHub"
KIND=docker                          # docker | http
IMAGE=ghcr.io/github/github-mcp-server:latest
REQUIRED="GITHUB_PERSONAL_ACCESS_TOKEN"
# DOCKER_ARGS="-e EXTRA=1"           # extra `docker run` flags
skeleton() { echo "GITHUB_PERSONAL_ACCESS_TOKEN="; }
```

HTTP servers use `KIND=http` with either `URL=` or `URL_VAR=`/`URL_SUFFIX=`.
They also take `AUTH=none|bearer|basic` with
`AUTH_TOKEN_VAR=` and `AUTH_USER_VAR=`.

The catalog and `mcp-sync` are also shipped by pmox's on-VM `devbox-setup`,
which asks for these keys interactively and writes the same `env` files.

## mcp-sync

```sh
mcp-sync jira aws-api   # enable exactly these (plus ~/.mcp/*/spec), then sync
mcp-sync                # sync the saved list
```

`mcp-sync` creates missing `env` skeletons and pulls images. It registers newly
ready servers and re-registers (`--force`) servers whose generated config
changed. It deregisters servers that are no longer enabled, but keeps their
credentials. Registration fails if MCPJungle cannot start the server. That
usually means a wrong credential, and the error is printed.

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `ai_tools_mcp_user` | `""` (required) | User the gateway runs as; owns `~/.mcp`; added to `docker`. |
| `ai_tools_mcp_servers` | `context7 sequential-thinking` | Space-separated servers to enable. |
| `ai_tools_mcpjungle_listen` | `127.0.0.1` | Listen address; `0.0.0.0` (or `""`) for all interfaces. |
| `ai_tools_mcpjungle_port` | `8080` | Gateway port. |
| `ai_tools_mcp_init_timeout` | `120` | Seconds to wait for a server to start. |
| `ai_tools_home_base` | `/home` | Parent of the MCP user's home. |
| `ai_tools_mcp_catalog_dir` | `/usr/local/share/mcp-catalog` | Where the catalog is installed. |
| `ai_tools_mcpjungle_version` | `0.4.6` | Pinned MCPJungle release. |
| `ai_tools_mcpjungle_arch` | `x86_64` | Release arch (`x86_64`/`arm64`). |
| `ai_tools_claude_channel` | `stable` | Claude Code apt channel (`stable`/`latest`). |
| `ai_tools_omp_install_dir` | `/usr/local/bin` | Install dir for the omp binary. |
| `ai_tools_users` | `[]` | Users whose Claude Code is wired to the gateway. |

## Security

- Membership in the `docker` group is **root-equivalent**. The MCP user can
  control the machine.
- The gateway runs in dev mode with **no authentication**. Anyone who reaches
  the port can call every registered tool with your credentials. The default
  is localhost-only. Set `ai_tools_mcpjungle_listen: 0.0.0.0` only on a trusted
  network (e.g. Tailscale), or reach the gateway through an SSH tunnel.
- Use least-privilege tokens. For example, `READ_ONLY_MODE=true` for Jira, a
  read-only IAM user, and `READ_OPERATIONS_ONLY=true` for AWS.

## Example

```yaml
name: Set up AI tooling
hosts: workstations
roles:
  - role: https://github.com/tackhq/tack-roles.git//docker
  - role: https://github.com/tackhq/tack-roles.git//ai-tools
    vars:
      ai_tools_mcp_user: eugene
      ai_tools_mcp_servers: "context7 sequential-thinking jira aws-api jenkins"
      ai_tools_mcpjungle_listen: 0.0.0.0     # optional: all interfaces
      ai_tools_users:
        - eugene
```

The first run creates `/home/eugene/.mcp/{jira,aws-api,jenkins}/env`. Fill them in,
then run `mcp-sync`.

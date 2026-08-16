# ai-tools

Provisions an AI stack on Ubuntu: two terminal coding agents plus a self-hosted
MCP gateway with servers registered into it.

## What it does

- **Claude Code** — installed system-wide from Anthropic's signed apt repo.
- **omp** ([oh-my-pi](https://omp.sh)) — an open-source terminal coding agent;
  self-contained prebuilt binary in `/usr/local/bin`.
- **Node.js** (NodeSource) — so the gateway can spawn `npx`-based MCP servers.
- **[MCPJungle](https://github.com/mcpjungle/MCPJungle)** — an MCP gateway that
  aggregates multiple MCP servers behind one endpoint. Installed from its GitHub
  release and run as a systemd service (`mcpjungle`, SQLite, dev mode) on
  `:8080/mcp`.
- Registers a default set of MCP servers into the gateway:
  [context7](https://context7.com) (HTTP) and `sequential-thinking` (stdio/npx).
- Optionally points each user's Claude Code at the gateway.

## Architecture

Register MCP servers once in MCPJungle; every client connects to the single
gateway endpoint instead of configuring servers individually. For users listed
in `ai_tools_users`, the role runs `claude mcp add --transport http mcpjungle
http://localhost:8080/mcp` so their Claude Code sees every gateway tool.

## Requirements

- Ubuntu 24.04 (noble) or newer.
- Root / privilege escalation.
- `ai_tools_users` must already exist (their Claude Code config lives in
  `~/.claude.json`).
- Claude Code still needs authentication (`claude` login) per user; the role
  installs and wires it but does not log in.

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `ai_tools_claude_channel` | `stable` | Claude Code apt channel (`stable`/`latest`). |
| `ai_tools_node_major` | `22` | Node.js major version (NodeSource). |
| `ai_tools_omp_install_dir` | `/usr/local/bin` | Install dir for the omp binary. |
| `ai_tools_mcpjungle_version` | `0.4.6` | Pinned MCPJungle release. |
| `ai_tools_mcpjungle_arch` | `x86_64` | Release arch (`x86_64`/`arm64`). |
| `ai_tools_mcpjungle_port` | `8080` | Gateway port. |
| `ai_tools_mcpjungle_home` | `/var/lib/mcpjungle` | Service state dir (SQLite DB). |
| `ai_tools_mcpjungle_user` | `mcpjungle` | Service system user. |
| `ai_tools_context7_url` | `https://mcp.context7.com/mcp` | context7 MCP endpoint. |
| `ai_tools_users` | `[]` | Users whose Claude Code is wired to the gateway. |

## Adding more MCP servers

- **HTTP:** `mcpjungle --registry http://127.0.0.1:8080 register --name <n> --url <url>`
- **stdio:** drop a JSON config (see
  [`files/mcp-sequential-thinking.json`](files/mcp-sequential-thinking.json))
  and `mcpjungle --registry http://127.0.0.1:8080 register -c <file>`

All registered servers are exposed through the same gateway endpoint, so clients
need no further changes.

## Example

```yaml
name: Set up AI tooling
hosts: workstations
roles:
  - role: https://github.com/tackhq/tack-roles.git//ai-tools
    vars:
      ai_tools_users:
        - eugene
```

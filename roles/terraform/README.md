# terraform

Installs [Terraform](https://developer.hashicorp.com/terraform) via
[tfenv](https://github.com/tfutils/tfenv), so hosts can install and switch
between Terraform versions on demand.

tfenv is installed **system-wide** under `/opt/tfenv` and its `tfenv` and
`terraform` shims are symlinked into `/usr/local/bin` — available to every user
and to non-interactive runs, with no shell-rc changes.

## What it does

1. Installs prerequisites: `git`, `curl`, `unzip`, `gnupg`.
2. Clones tfenv into `/opt/tfenv` at a pinned release tag.
3. Symlinks `tfenv` and `terraform` into `/usr/local/bin`.
4. Installs the requested Terraform version (`tfenv install`).
5. Sets it as the global default (`tfenv use`).

Once installed, switch versions on the host with `tfenv install <ver>` /
`tfenv use <ver>`, or per-project with a `.terraform-version` file.

## Requirements

- Ubuntu (uses `apt`).
- Root / privilege escalation for the system-wide install.

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `tfenv_root` | `/opt/tfenv` | tfenv install directory. |
| `tfenv_repo` | `https://github.com/tfutils/tfenv.git` | tfenv source repository. |
| `tfenv_version` | `v3.2.2` | Pinned tfenv release tag. |
| `tfenv_bin_dir` | `/usr/local/bin` | PATH directory for the `tfenv`/`terraform` symlinks. |
| `terraform_version` | `latest` | Terraform version to install and default. Accepts an exact version, `latest`, or a tfenv regex like `latest:^1.9`. |

## Example

```yaml
name: Install Terraform
hosts: servers
roles:
  - role: https://github.com/tackhq/tack-roles.git//terraform
    vars:
      terraform_version: "1.9.8"
```

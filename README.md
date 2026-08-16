# tack-roles

A community collection of reusable [Tack](https://github.com/tackhq/tack) roles.

Tack is a single-binary, dependency-free configuration management and
system-bootstrapping tool inspired by Ansible. **Roles** package tasks,
handlers, variables, templates, and files into a self-contained unit you can
share across playbooks and projects. This repository is a home for such roles.

## Available roles

| Role | Description |
| --- | --- |
| [`docker`](roles/docker) | Installs Docker Engine on Ubuntu from Docker's official apt repository. |

## Using a role

Tack resolves a plain role name against the `roles/` directory next to your
playbook, but it can also fetch a role straight from a git or HTTPS URL — so
you can consume anything in this repo without vendoring it first.

```yaml
name: Set up a host
hosts: servers
roles:
  # A role published in this repository, pulled directly from GitHub.
  - role: https://github.com/tackhq/tack-roles.git//docker
    tags: [docker]

  # A local role in ./roles/, for comparison.
  - role: firewall
    tags: [net]
```

Remote role references support:

- HTTPS — `https://github.com/tackhq/tack-roles.git`
- SSH — `git@github.com:tackhq/tack-roles.git`
- Subdirectory — append `//path/to/role`
- Pinned ref — append `@branch-or-tag` to lock to a release

Tack clones remote roles into a temporary directory before the play runs and
removes them afterward. Pin to a tag when you want reproducible runs.

## Role layout

Each role follows Tack's Ansible-compatible directory structure:

```
roles/
└── <role-name>/
    ├── tasks/
    │   └── main.yaml      # what the role does
    ├── handlers/
    │   └── main.yaml      # actions triggered via notify
    ├── defaults/
    │   └── main.yaml      # low-priority default variables
    ├── vars/
    │   └── main.yaml      # higher-priority variables
    ├── files/             # static files to copy
    └── templates/         # templated files to render
```

Only the directories a role actually needs have to be present.

## Contributing a role

1. Scaffold the boilerplate:

   ```
   tack scaffold <role-name>
   ```

2. Fill in `tasks/main.yaml` and any variables, handlers, files, or templates
   the role needs.
3. Keep roles focused — one responsibility per role — and prefer overridable
   `defaults/` over hard-coded `vars/` so consumers can tune behavior.
4. Document any required variables at the top of the role's `tasks/main.yaml`
   or in a role-level README.
5. Open a pull request.

## License

Released under the [MIT License](LICENSE).

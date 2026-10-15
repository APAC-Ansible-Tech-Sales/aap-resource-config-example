# aap-resource-config-example

An example of using Configuration as Code (CaC) to create and manage Ansible Automation Platform (AAP) resources.

## What it does

The playbook in `playbooks/main.yml` configures AAP Controller resources by applying YAML definitions as the desired state. It runs on `localhost`, connects to an AAP instance, and uses the [`infra.aap_configuration.dispatch`](https://galaxy.ansible.com/ui/repo/published/infra/aap_configuration/) role to create or update the selected resources.

You choose which resource types to deploy by passing a `resources` list. Each entry maps to a vars file under `playbooks/group_vars/`.

## Resources managed

| Resource key | Vars file | Purpose |
|---|---|---|
| `controller_projects` | `group_vars/controller_projects.yml` | SCM projects (Git repos, branches, credentials, EEs) |
| `controller_credentials` | `group_vars/controller_credentials.yml` | Credentials (Machine, Vault, etc.) |
| `controller_templates` | `group_vars/controller_templates.yml` | Job templates (playbooks, surveys, launch prompts) |
| `controller_schedules` | `group_vars/controller_schedules.yml` | Recurring schedules for job templates |

Edit the corresponding vars file to change what gets applied. Sensitive values (such as the AAP password) live in the Ansible Vault file at `playbooks/group_vars/localhost/vault.yml`.

## How it runs

A GitHub Actions workflow (`.github/workflows/aap_resource_config.yml`) is triggered manually via `workflow_dispatch`. It:

1. Installs the `infra.aap_configuration` collection from Automation Hub
2. Runs `ansible-playbook playbooks/main.yml` with:
   - `aap_hostname` from the `AAP_HOSTNAME` repository variable
   - `resources` from the workflow input (defaults to all four resource types above)
   - Vault unlock via the `ANSIBLE_VAULT` repository secret

### Required GitHub configuration

| Type | Name | Purpose |
|---|---|---|
| Variable | `AAP_HOSTNAME` | AAP Controller hostname |
| Secret | `ANSIBLE_VAULT` | Password to decrypt `vault.yml` |

## Local usage

```bash
cd playbooks
ansible-galaxy collection install infra.aap_configuration --server https://console.redhat.com/api/automation-hub/ --token '<TOKEN_FROM_HUB>'
ansible-playbook main.yml \
  --extra-vars "aap_hostname=https://your-aap.example.com" \
  --extra-vars '{"resources": ["controller_projects", "controller_credentials", "controller_templates", "controller_schedules"]}' \
  --ask-vault-pass
```

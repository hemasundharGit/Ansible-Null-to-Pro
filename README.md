# Ansible Null to Pro

A hands-on collection of Ansible playbooks and a custom role, built to go from zero Ansible knowledge to practical, working automation. Each file targets a specific core concept — ad-hoc tasks, package management, loops and conditionals, roles, handlers, and Ansible Vault for secrets.

## 📁 Repository Structure

```
Ansible-Null-to-Pro/
├── ansible.cfg                     # Project-level Ansible configuration
├── hosts.ini                       # Inventory file (worker nodes)
├── hello.yml                       # First playbook — variables & command module
├── install-package.yml             # Installing a single package (apt)
├── multiple-package.yml            # Installing multiple packages with a loop + conditional
├── setup-nginx.yml                 # Deploying nginx + a static index.html to servers
├── install_docker_with_role.yml    # Playbook that consumes the docker role
├── secrets.yml                     # Ansible Vault–encrypted variables (password, api_key)
├── show_secrets.yml                # Reads vault secrets with no_log enabled
├── encrypt.txt                     # Sample vault-encrypted text file
├── index.html                      # Sample static page deployed by setup-nginx.yml
└── roles/
    └── docker/                     # Custom role: installs & configures Docker
        ├── tasks/main.yml
        ├── handlers/main.yml
        ├── vars/main.yml
        ├── defaults/main.yml
        ├── meta/main.yml
        ├── tests/
        │   ├── inventory
        │   └── test.yml
        └── README.md
```

## ⚙️ Prerequisites

- Ansible installed on the control node (`pip install ansible` or your distro's package manager)
- SSH access to the target hosts, with the private key referenced in `hosts.ini`
- Target hosts running Ubuntu (playbooks use `apt` and Ubuntu-specific facts)
- Python 3 on the target hosts (`ansible_python_interpreter=/usr/bin/python3`, already set in `hosts.ini`)

## 🖥️ Inventory

`hosts.ini` defines a `servers` group with two worker nodes, connecting as the `ubuntu` user via a shared private key:

```ini
[servers]
worker-node-1 ansible_host=<ip> ansible_user=ubuntu
worker-node-2 ansible_host=<ip> ansible_user=ubuntu

[all:vars]
ansible_ssh_private_key_file=/home/ubuntu/ansible-master-key
ansible_python_interpreter=/usr/bin/python3
```

Update the hostnames/IPs and key path to match your own environment before running any playbook.

## ▶️ Playbooks

| Playbook | What it teaches | Run it |
|---|---|---|
| `hello.yml` | Basic playbook structure, `vars`, and the `command` module | `ansible-playbook -i hosts.ini hello.yml` |
| `install-package.yml` | Installing a single package with the `apt` module | `ansible-playbook -i hosts.ini install-package.yml` |
| `multiple-package.yml` | Looping over a list of packages, `when` conditionals, `debug` output | `ansible-playbook -i hosts.ini multiple-package.yml` |
| `setup-nginx.yml` | Installing nginx, copying a file with `copy`, restarting/enabling a service | `ansible-playbook -i hosts.ini setup-nginx.yml` |
| `install_docker_with_role.yml` | Applying a reusable role to a set of hosts | `ansible-playbook -i hosts.ini install_docker_with_role.yml` |
| `show_secrets.yml` | Loading encrypted variables via `vars_files` and hiding output with `no_log` | `ansible-playbook -i hosts.ini show_secrets.yml --ask-vault-pass` |

## 🔐 Secrets with Ansible Vault

`secrets.yml` and `encrypt.txt` are encrypted with Ansible Vault and hold sample values (`password`, `api_key`). To work with them:

```bash
# View/edit an encrypted file
ansible-vault edit secrets.yml

# Encrypt a new file
ansible-vault encrypt <file>

# Run a playbook that needs the vault password
ansible-playbook -i hosts.ini show_secrets.yml --ask-vault-pass
```

`show_secrets.yml` demonstrates the important pairing of reading a secret and setting `no_log: true` so the value never appears in console output or logs.

## 📦 The `docker` Role

`roles/docker` is a standard Ansible Galaxy–style role skeleton that:

- Updates all system packages (`apt: name=* state=latest`)
- Installs `docker.io`
- Adds the users listed in `vars/main.yml` (`docker_users`, defaults to `ubuntu`) to the `docker` group
- Restarts the Docker service (also wired up as a handler)
- Registers and prints the installed Docker version
- Lists currently running containers

Role variables (`roles/docker/vars/main.yml`):

```yaml
docker_users:
  - ubuntu
```

`roles/docker/meta/main.yml` and `roles/docker/README.md` are still the default Ansible Galaxy scaffolding (`ansible-galaxy init` placeholders) and are good candidates to fill in as this repo grows — see below.

## 🚀 Getting Started

```bash
git clone https://github.com/hemasundharGit/Ansible-Null-to-Pro.git
cd Ansible-Null-to-Pro

# Update hosts.ini with your own server IPs and SSH key path

# Test connectivity
ansible -i hosts.ini servers -m ping

# Run a playbook
ansible-playbook -i hosts.ini hello.yml
```

## 🗺️ Roadmap / Ideas to Add

This repo is a learning-in-progress project. Some natural next steps:

- [ ] Fill in `roles/docker/meta/main.yml` and `roles/docker/README.md` with real author/license/description info
- [ ] Add a `templates/` directory and use `template` (Jinja2) instead of the static `copy` in `setup-nginx.yml`
- [ ] Add tags to plays/tasks and show `--tags` / `--skip-tags` usage
- [ ] Add an `ansible-lint` / `yamllint` config and a CI workflow (GitHub Actions) to lint playbooks on push
- [ ] Add a role for nginx (mirroring the `docker` role) instead of the flat `setup-nginx.yml` playbook
- [ ] Demonstrate `group_vars/` and `host_vars/` instead of inline `vars`
- [ ] Add a playbook using `blockinfile` or `lineinfile` for config management
- [ ] Add error handling examples (`rescue`, `block`, `ignore_errors`, `failed_when`)
- [ ] Document vault password management (`--vault-password-file`, multiple vault IDs)

## 📄 License

This project is licensed under the [MIT License](LICENSE) — free to use, modify, and distribute with attribution.

# Ansible Configuration for Raspberry Pi

Ansible playbooks and configuration for provisioning Raspberry Pi devices. Sets up SSH key authentication, essential packages, rootless Docker, and Tailscale VPN — inspired by [geerlingguy/internet-pi](https://github.com/geerlingguy/internet-pi).

This is a **template** — all personal configuration has been replaced with dummy values. Clone it, customize the config, and deploy.

## Directory Structure

```
ansible-public/
├── main.yml                  # Main playbook that orchestrates all tasks
├── ansible.cfg               # Ansible configuration
├── requirements.yml          # Required Ansible collections
├── example.config.yml        # Example configuration (copy to config.yml)
├── example.inventory.ini     # Example inventory (copy to inventory.ini)
├── VAULT.md                  # Guide for managing secrets with Ansible Vault
├── .gitignore
├── group_vars/
│   └── rpi/
│       ├── vars.yml          # Non-sensitive group variables
│       └── vault.example.yml # Example vault file for secrets
└── tasks/
    ├── ssh-key-setup.yml     # SSH key authentication setup
    ├── initial-setup.yml     # System updates & essential packages
    ├── docker-install.yml    # Rootless Docker installation
    └── tailscale-install.yml # Tailscale VPN installation & configuration
```

## Features

- **SSH Key Authentication** — Copies your public key to the Pi, eliminating password-based SSH
- **Initial System Setup** — Updates packages and installs essential tools (vim, git, curl, htop, etc.)
- **Rootless Docker** — Installs Docker in rootless mode for improved security
- **Tailscale VPN** — Installs and configures Tailscale with subnet routing, exit node, and SSH support
- **Modular Design** — Each feature is a separate task file, individually toggleable via config

## Quick Start

1. **Install Ansible**:
   ```bash
   pip3 install ansible
   ```

2. **Install required collections**:
   ```bash
   ansible-galaxy collection install -r requirements.yml
   ```

3. **Set up your inventory**:
   ```bash
   cp example.inventory.ini inventory.ini
   ```
   Edit `inventory.ini` and add your Raspberry Pi's IP address and SSH user:
   ```ini
   [rpi]
   raspberrypi ansible_host=192.168.1.100 ansible_user=pi
   ```

4. **Set up your configuration**:
   ```bash
   cp example.config.yml config.yml
   ```
   Edit `config.yml` to customize settings (SSH key path, username, packages, subnet routes, etc.)

5. **Generate an SSH key** (if you don't have one):
   ```bash
   ssh-keygen -t ed25519 -C "ansible@raspberry-pi"
   ```

6. **Test connectivity**:
   ```bash
   ansible rpi -m ping --ask-pass
   ```

7. **Run the playbook**:
   ```bash
   ansible-playbook main.yml --ask-pass --ask-become-pass
   ```

After the first run, SSH keys will be set up and you can drop `--ask-pass`:
```bash
ansible-playbook main.yml --ask-become-pass
```

## Configuration

All settings live in `config.yml`. Toggle features on/off and customize values:

| Setting | Description | Default |
|---------|-------------|---------|
| `ssh_key_setup_enable` | Set up SSH key authentication | `true` |
| `ssh_public_key` | Path to your local SSH public key | `~/.ssh/id_rsa.pub` |
| `ssh_authorized_keys_user` | User to configure SSH keys for | `pi` |
| `initial_setup_enable` | Run system updates & install packages | `true` |
| `essential_packages` | List of packages to install | vim, git, curl, wget, htop, ufw |
| `docker_install_enable` | Install rootless Docker | `true` |
| `docker_user` | User to install Docker for | `pi` |
| `tailscale_install_enable` | Install and configure Tailscale | `true` |
| `tailscale_subnet_routes` | Subnets to advertise via Tailscale | `["10.0.0.0/24", "192.168.1.0/24"]` |
| `tailscale_enable_exit_node` | Enable as a Tailscale exit node | `true` |
| `tailscale_enable_ssh` | Enable Tailscale SSH | `true` |

## Managing Secrets

For secure password management, use **Ansible Vault**. See [VAULT.md](VAULT.md) for a full guide.

Quick version:
```bash
# Create encrypted vault
ansible-vault create group_vars/rpi/vault.yml

# Add secrets (editor opens):
#   ansible_ssh_pass: "your_ssh_password"
#   ansible_become_pass: "your_sudo_password"

# Run with vault
ansible-playbook main.yml --ask-vault-pass
```

## Tailscale Auth Key

To automatically authenticate Tailscale (instead of manual `tailscale up`), pass your auth key as an extra variable:

```bash
ansible-playbook main.yml --ask-become-pass -e "ts_key=tskey-auth-xxxxx"
```

Generate an auth key at [Tailscale Admin Console](https://login.tailscale.com/admin/settings/keys).

> **Note:** After connecting, you may need to approve subnet routes and exit node in the Tailscale admin console.

## Prerequisites

- Ansible installed on your control machine
- SSH access to your Raspberry Pi (password authentication initially)
- Python installed on the Raspberry Pi (included in Raspberry Pi OS)
- SSH public key on your control machine
# raspberry-pi-homelab

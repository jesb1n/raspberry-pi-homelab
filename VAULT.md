# Ansible Vault Guide

Ansible Vault is the recommended way to handle secrets (passwords, API keys, etc.) in Ansible.

## Quick Start

### 1. Create a Vault File

```bash
# Create the vault file (will prompt for vault password)
ansible-vault create group_vars/rpi/vault.yml
```

### 2. Add Your Secrets

When the editor opens, add your secrets:

```yaml
---
ansible_ssh_pass: "your_ssh_password"
ansible_become_pass: "your_sudo_password"
```

Save and close. The file will be encrypted.

### 3. Use the Vault

Run playbooks with the vault password:

```bash
# Option 1: Prompt for vault password
ansible-playbook main.yml --ask-vault-pass

# Option 2: Use password file (more secure for automation)
echo "your_vault_password" > .vault_pass
chmod 600 .vault_pass
ansible-playbook main.yml --vault-password-file .vault_pass

# Option 3: Use environment variable
export ANSIBLE_VAULT_PASSWORD_FILE=.vault_pass
ansible-playbook main.yml
```

## Common Vault Operations

### Edit Encrypted File
```bash
ansible-vault edit group_vars/rpi/vault.yml
```

### View Encrypted File
```bash
ansible-vault view group_vars/rpi/vault.yml
```

### Change Vault Password
```bash
ansible-vault rekey group_vars/rpi/vault.yml
```

### Encrypt Existing File
```bash
ansible-vault encrypt group_vars/rpi/vault.yml
```

### Decrypt File (temporarily)
```bash
ansible-vault decrypt group_vars/rpi/vault.yml
```

## Best Practices

1. **Never commit vault passwords** - Add `.vault_pass` to `.gitignore`
2. **Use different vault passwords** for different environments (dev, prod)
3. **Rotate vault passwords** periodically
4. **Use vault password files** for CI/CD pipelines (store securely)
5. **Keep vault files separate** from regular variables

## File Structure

```
ansible/
├── group_vars/
│   └── rpi/
│       ├── vars.yml          # Non-sensitive variables
│       └── vault.yml          # Encrypted secrets (gitignored)
├── host_vars/
│   └── hostname/
│       ├── vars.yml          # Host-specific non-sensitive vars
│       └── vault.yml         # Host-specific secrets
└── .vault_pass               # Vault password file (gitignored)
```

## Alternative: Environment Variables

For CI/CD, you can also use environment variables:

```bash
export ANSIBLE_VAULT_PASSWORD="your_vault_password"
ansible-playbook main.yml
```

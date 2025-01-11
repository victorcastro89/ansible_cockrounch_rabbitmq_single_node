# CockroachDB and RabbitMQ Single-Node Deployment

## Project Overview
This Ansible project automates the deployment of a single-node CockroachDB instance and RabbitMQ server with SSL/TLS encryption. It includes complete setup of security configurations, system optimizations, and SSL certificate generation for both services.

## Prerequisites
- Ubuntu 22.04 (Jammy) target server
- Ansible 2.9 or higher on the control node
- Python 3.x
- Minimum system requirements:
  - 2 CPU cores
  - 4GB RAM
  - 20GB available disk space
- SSH access to target servers
- Sudo privileges on target servers

## Installation

1. Install Ansible and required collection:
```bash
pip install ansible
ansible-galaxy collection install community.rabbitmq
```

2. Clone this repository:
```bash
git clone <repository-url>
cd <repository-name>
```

3. Set up configuration files from examples:
```bash
# Copy example files
cp inventories/production/group_vars/all/vault.yml.example inventories/production/group_vars/all/vault.yml
cp .env.example .env
cp inventories/production/hosts.yml.example inventories/production/hosts.yml

# Set proper permissions
chmod 600 .env
```

4. Configure your inventory hosts in `inventories/production/hosts.yml`:
```yaml
[cockroachdb_cluster]
node1 ansible_host=<your-server-ip> ansible_ssh_private_key_file=~/.ssh/id_rsa
```

4. Update group variables in `inventories/production/group_vars/all/main.yml`:

Required changes:
- Network configurations:
  ```yaml
  trusted_networks: "192.168.1.0/24"    # Network range for database access
  admin_networks: "192.168.1.1/24"      # Network range for admin access
  monitoring_networks: "192.168.1.1/24"  # Network range for monitoring access
  ansible_user: "your-ssh-user"         # SSH user for deployment
  ```

- CockroachDB settings:
  ```yaml
  cockroachdb:
    version: "24.3.2"                   # Verify latest stable version
    max_memory: "25%"                   # Adjust based on server RAM
    cache_size: "25%"                   # Adjust based on server RAM
    locality: "region=us-east,zone=us-east-1"  # Update to match your region
  ```

- RabbitMQ settings:
  ```yaml
  rabbitmq:
    admin_user: "admin"                 # Change admin username
    web_admin_user: "web_admin"         # Change web UI username
  ```

5. Update vault variables in `inventories/production/group_vars/all/vault.yml`:
```yaml
vault_cockroachdb_admin_password: "change-this-password"
vault_rabbitmq_admin_password: "change-this-password"
vault_rabbitmq_web_admin_password: "change-this-password"
```

5. Set up Ansible Vault:
```bash
# Create vault password file
echo "your-vault-password" > .vault_pass

# Encrypt the vault file
ansible-vault encrypt inventories/production/group_vars/all/vault.yml
```

## Deployment Instructions

1. Verify connectivity to target servers:
```bash
ansible cockroachdb_cluster -m ping
```

2. Run the deployment playbook:
```bash
ansible-playbook site.yml
```

3. Access the deployed services:
- CockroachDB UI: https://<server-ip>:8080
- RabbitMQ Management UI: https://<server-ip>:15672

## Additional Notes

- Default ports:
  - CockroachDB SQL: 26258
  - CockroachDB HTTP: 8080
  - CockroachDB Internal: 26257
  - RabbitMQ AMQP: 5672
  - RabbitMQ Management: 15672

- SSL certificates are automatically generated and configured for both services
- The deployment includes UFW firewall configuration
- System is optimized for database workloads with appropriate sysctl settings
- Default credentials are stored in the vault file
- For production use, remember to:
  - Change default passwords in vault.yml
  - Configure appropriate firewall rules
  - Review and adjust system resource allocations
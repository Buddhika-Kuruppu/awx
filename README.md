# Ansible AWX Deployments

A well-organized collection of Ansible playbooks for AWX/Tower deployments, featuring Docker installation, LAMP stack deployment, and Nginx reverse proxy configuration.

## Repository Structure

```
ansible-awx-deployments/
├── ansible.cfg                 # Ansible configuration
├── requirements.yml            # Required collections and roles
├── playbooks/                  # Main playbooks directory
│   ├── docker-installation.yaml
│   ├── lamp-deployment.yaml
│   └── nginx-reverse-proxy.yaml
├── inventories/               # Environment inventories
│   ├── production.ini
│   └── staging.ini
├── group_vars/               # Group-specific variables
│   ├── all.yml
│   ├── webservers.yml
│   ├── lamp_servers.yml
│   └── docker_hosts.yml
├── host_vars/               # Host-specific variables
│   ├── web01.yml
│   └── lamp01.yml
├── templates/               # Jinja2 templates
│   ├── nginx.conf.j2
│   ├── upstream.conf.j2
│   └── reverse-proxy.conf.j2
├── roles/                   # Custom roles (empty)
└── files/                   # Static files (empty)
```

## Playbooks

### 1. Docker Installation (`docker-installation.yaml`)
Installs Docker CE on Ubuntu systems with all necessary components.

**Features:**
- Docker CE, CLI, and Compose plugin installation
- User group configuration
- Service management
- Installation verification

**Usage in AWX:**
- **Inventory:** `docker_hosts` group
- **Credentials:** SSH key for target hosts
- **Variables:** Configure via `group_vars/docker_hosts.yml`

### 2. LAMP Stack Deployment (`lamp-deployment.yaml`)
Deploys a complete LAMP (Linux, Apache, MySQL, PHP) stack with cross-platform support.

**Features:**
- Apache/HTTPD installation and configuration
- MySQL server setup with secure installation
- PHP with common extensions
- Database and user creation
- Firewall configuration

**Usage in AWX:**
- **Inventory:** `lamp_servers` group
- **Credentials:** SSH key + Vault for database passwords
- **Variables:** Configure database settings in `group_vars/lamp_servers.yml`

### 3. Nginx Reverse Proxy (`nginx-reverse-proxy.yaml`)
Configures Nginx as a reverse proxy with SSL support and load balancing.

**Features:**
- Nginx installation with extras
- SSL certificate generation (self-signed)
- Load balancing configuration
- Security headers
- Log rotation
- Health check endpoints

**Usage in AWX:**
- **Inventory:** `webservers` group
- **Credentials:** SSH key
- **Variables:** Configure domains and SSL in `group_vars/webservers.yml`

## AWX Integration

### Job Templates Configuration

1. **Docker Installation Job**
   - **Playbook:** `playbooks/docker-installation.yaml`
   - **Inventory:** Select appropriate inventory
   - **Limit:** `docker_hosts`

2. **LAMP Stack Job**
   - **Playbook:** `playbooks/lamp-deployment.yaml`
   - **Inventory:** Select appropriate inventory
   - **Limit:** `lamp_servers`
   - **Extra Variables:**
     ```yaml
     vault_mysql_root_password: "{{ vault_mysql_root_password }}"
     vault_db_password: "{{ vault_db_password }}"
     ```

3. **Nginx Reverse Proxy Job**
   - **Playbook:** `playbooks/nginx-reverse-proxy.yaml`
   - **Inventory:** Select appropriate inventory
   - **Limit:** `webservers`
   - **Survey Variables:**
     - `domain_name`: Target domain
     - `enable_ssl`: Enable HTTPS (true/false)
     - `configure_firewall`: Configure UFW (true/false)

### Prerequisites

1. **Install Required Collections:**
   ```bash
   ansible-galaxy install -r requirements.yml
   ```

2. **Configure Vault for Sensitive Data:**
   ```bash
   ansible-vault encrypt_string 'your_password' --name 'vault_mysql_root_password'
   ```

3. **Update Inventory Files:**
   - Edit `inventories/production.ini` or `inventories/staging.ini`
   - Set correct IP addresses and SSH configurations

### Security Considerations

- Use Ansible Vault for all passwords and sensitive data
- Store vault passwords securely in AWX credentials
- Use SSH key authentication (disable password auth)
- Regularly update SSL certificates in production
- Review firewall rules before deployment

### Testing

Test playbooks individually before running in AWX:

```bash
# Test Docker installation
ansible-playbook -i inventories/staging.ini playbooks/docker-installation.yaml --limit docker_hosts --check

# Test LAMP stack
ansible-playbook -i inventories/staging.ini playbooks/lamp-deployment.yaml --limit lamp_servers --check

# Test Nginx reverse proxy
ansible-playbook -i inventories/staging.ini playbooks/nginx-reverse-proxy.yaml --limit webservers --check
```

## Customization

### Adding New Hosts
1. Update inventory files in `inventories/`
2. Create host-specific variables in `host_vars/`
3. Adjust group variables in `group_vars/` as needed

### Custom SSL Certificates
Replace self-signed certificate generation in the Nginx playbook with your certificate provider's process.

### Additional PHP Extensions
Modify `php_packages_debian` and `php_packages_redhat` in `group_vars/lamp_servers.yml`

## Support

For issues and contributions, please use the repository's issue tracker.
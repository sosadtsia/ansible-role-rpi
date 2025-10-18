Ansible Role: Raspberry Pi Management
======================================

A comprehensive Ansible role for managing Raspberry Pi devices (RPi2, RPi3, and newer models). This role handles system updates, package management, security hardening, and common Raspberry Pi configurations across Debian-based distributions (Raspberry Pi OS, Raspbian, Debian).

## Features

- **System Updates**: Keeps the OS up to date with automatic apt cache updates and package upgrades
- **Package Management**: Installs essential packages and removes unnecessary ones
- **Security Hardening**: SSH configuration, firewall setup (UFW), and fail2ban
- **User Management**: Creates and manages users with sudo access
- **System Configuration**: Timezone, locale, hostname, and network settings
- **Monitoring**: Optional installation of monitoring tools (htop, ncdu, iotop)
- **Docker Support**: Optional Docker installation for container workloads
- **Automatic Cleanup**: Removes old kernels and unused packages
- **Reboot Management**: Detects when reboot is required and can optionally auto-reboot

## Requirements

- Ansible 2.10 or higher
- Target system: Raspberry Pi 2, Pi 3, or newer models running Debian-based OS (Raspberry Pi OS, Raspbian, Debian)
- SSH access to the Raspberry Pi with sudo privileges
- Python 3 installed on the target system (usually pre-installed on modern Raspberry Pi OS)

## Role Variables

### System Configuration

```yaml
# System update settings
rpi_update_cache: true
rpi_cache_valid_time: 3600  # seconds
rpi_upgrade_type: "dist"     # Options: dist, full, safe
rpi_autoremove: true
rpi_autoclean: true

# Hostname configuration
rpi_hostname: "{{ inventory_hostname }}"
rpi_set_hostname: false

# Timezone and locale
rpi_timezone: "America/New_York"
rpi_locale: "en_US.UTF-8"
rpi_set_locale: false
```

### Package Management

```yaml
# Essential packages to install
rpi_essential_packages:
  - curl
  - wget
  - git
  - vim
  - htop
  - ncdu
  - tree
  - tmux
  - ufw
  - fail2ban
  - unattended-upgrades

# Additional packages (customize per environment)
rpi_additional_packages: []

# GPIO and hardware packages (optional)
rpi_gpio_packages:
  - python3-gpiozero    # High-level GPIO library
  - python3-rpi.gpio    # Low-level GPIO library
  - i2c-tools           # I2C utilities
  - python3-smbus       # Python I2C library
rpi_install_gpio_packages: false

# Packages to remove
rpi_packages_to_remove: []
```

### Security Settings

```yaml
# SSH configuration
rpi_ssh_port: 22
rpi_ssh_permit_root_login: "no"
rpi_ssh_password_authentication: "no"
rpi_ssh_pubkey_authentication: "yes"
rpi_configure_ssh: true

# UFW firewall
rpi_enable_ufw: true
rpi_ufw_allow_ports:
  - "22/tcp"    # SSH
  - "80/tcp"    # HTTP
  - "443/tcp"   # HTTPS

# Fail2ban
rpi_enable_fail2ban: true
rpi_fail2ban_maxretry: 5
rpi_fail2ban_bantime: 3600
```

### User Management

```yaml
# Create additional users
rpi_users: []
  # - name: piuser
  #   groups: sudo
  #   shell: /bin/bash
  #   ssh_key: "ssh-rsa AAAA..."

# Disable default pi user (recommended for security)
rpi_disable_pi_user: false
```

### Optional Features

```yaml
# Docker installation
rpi_install_docker: false
rpi_docker_users:
  - "{{ ansible_user }}"

# Enable automatic security updates
rpi_enable_unattended_upgrades: true

# Reboot management
rpi_auto_reboot: false
rpi_reboot_time: "03:00"  # Time for scheduled reboots if required
```

### Performance and Hardware

```yaml
# Memory split for GPU (useful for headless systems)
rpi_gpu_mem: 16  # MB, set to 16 for headless, 128+ for desktop

# Overclocking (use with caution)
rpi_overclock: false
rpi_arm_freq: 1500
rpi_gpu_freq: 500

# Enable hardware interfaces (GPIO, I2C, SPI, etc.)
rpi_enable_i2c: false          # I2C interface for sensors, displays
rpi_enable_spi: false          # SPI interface for displays, ADCs
rpi_enable_camera: false       # Camera module support
rpi_enable_serial: false       # Serial console on GPIO pins
rpi_enable_1wire: false        # 1-Wire interface (DS18B20 temperature sensors)

# GPIO access permissions
rpi_gpio_group: "gpio"         # Group for GPIO access
rpi_add_users_to_gpio: true    # Add ansible_user to gpio group
```

## Dependencies

None.

## Example Playbook

### Basic Usage

```yaml
---
- hosts: raspberry_pis
  become: true
  roles:
    - role: ansible-role-rpi
```

### Advanced Configuration

```yaml
---
- hosts: raspberry_pis
  become: true
  roles:
    - role: ansible-role-rpi
      vars:
        rpi_hostname: "rpi-home-01"
        rpi_set_hostname: true
        rpi_timezone: "America/New_York"
        rpi_essential_packages:
          - curl
          - git
          - vim
          - htop
          - docker.io
        rpi_enable_ufw: true
        rpi_ufw_allow_ports:
          - "22/tcp"
          - "8080/tcp"
        rpi_install_docker: true
        rpi_users:
          - name: admin
            groups: sudo,docker
            shell: /bin/bash
            ssh_key: "ssh-rsa AAAAB3NzaC1yc2E..."
        rpi_auto_reboot: false
```

### Multi-Environment Setup

```yaml
# production.yml
---
- hosts: production_rpis
  become: true
  roles:
    - role: ansible-role-rpi
      vars:
        rpi_upgrade_type: "safe"
        rpi_enable_fail2ban: true
        rpi_ssh_password_authentication: "no"
        rpi_enable_unattended_upgrades: true

# development.yml
---
- hosts: dev_rpis
  become: true
  roles:
    - role: ansible-role-rpi
      vars:
        rpi_install_docker: true
        rpi_additional_packages:
          - build-essential
          - python3-pip
          - python3-venv
```

## Inventory Example

```ini
[raspberry_pis]
rpi-01 ansible_host=192.168.1.100
rpi-02 ansible_host=192.168.1.101
rpi-03 ansible_host=192.168.1.102

[raspberry_pis:vars]
ansible_user=pi
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
```

## Post-Installation Tasks

After running this role, you should:

1. **Verify SSH Access**: Ensure you can still connect after SSH hardening
2. **Check Firewall Rules**: Verify UFW is configured correctly
3. **Test Services**: Confirm all required services are running
4. **Monitor Logs**: Check `/var/log/auth.log` for fail2ban activity
5. **Reboot if Required**: Check if `/var/run/reboot-required` exists

## Additional Configurations

### 1. Monitoring and Alerting
- Install and configure Prometheus node exporter
- Set up Grafana dashboards for RPi metrics
- Configure email alerts for system issues

### 2. Backup Strategy
- Implement automated backup of critical data
- Use tools like rsync to backup data to pcloud storage

### 3. Log Management
- Configure log rotation (logrotate)
- Monitor disk space for log files

### 4. Network Configuration
- Static IP configuration
- DHCP configuration

### 5. Storage Management
- Configure automatic mounting via fstab

### 7. Optional Service-Specific Roles
- Web server (Nginx, Apache)
- Database (PostgreSQL, MariaDB, Redis)
- pyhole (Pi-hole for ad blocking)
- NAS (Samba, NFS)
- Mail server (Postfix)
- VPN server (WireGuard)

## Security Best Practices

1. **Change Default Passwords**: Never use default credentials
2. **Use SSH Keys**: Disable password authentication
3. **Enable Firewall**: Use UFW with minimal open ports
4. **Keep Updated**: Run updates regularly
5. **Enable Fail2ban**: Protect against brute force attacks
6. **Disable Unused Services**: Reduce attack surface
7. **Monitor Logs**: Regularly check for suspicious activity
8. **Use Strong Passwords**: For any services requiring authentication
9. **Enable Automatic Security Updates**: Keep system patched
10. **Regular Backups**: Ensure you can recover from incidents

## Troubleshooting

### SSH Connection Issues
```bash
# Test SSH connection with verbose output
ssh -vvv user@raspberry-pi-ip

# Check SSH service status on RPi
sudo systemctl status ssh
```

### Package Installation Failures
```bash
# Update apt cache manually
sudo apt update

# Fix broken packages
sudo apt --fix-broken install

# Check disk space
df -h
```

### Firewall Blocking Access
```bash
# Check UFW status
sudo ufw status verbose

# Allow port temporarily
sudo ufw allow 22/tcp

# Disable UFW if locked out (physical access required)
sudo ufw disable
```

## Testing

This role includes molecule tests for verification:

```bash
# Install molecule
pip install molecule molecule-docker

# Run tests
cd ansible-role-rpi
molecule test
```

## License

Apache 2.0

## Author Information

Created for managing Raspberry Pi 2 and Pi 3 infrastructure running Debian-based operating systems.

## Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## Changelog



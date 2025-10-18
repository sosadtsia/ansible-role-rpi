# Ansible Role: Raspberry Pi

Ansible role for managing Raspberry Pi devices with security hardening, system updates, and common configurations.

## Features

- System updates and package management
- SSH hardening and firewall (UFW)
- Fail2ban intrusion prevention
- User management with sudo access
- Docker support (optional)
- GPIO and hardware interfaces
- Automated reboot handling

## Requirements

- Ansible 2.10+
- Raspberry Pi 2/3 or newer
- Debian-based OS (Raspberry Pi OS, Raspbian, Debian)
- SSH access with sudo privileges

## Quick Start

```yaml
- hosts: raspberry_pis
  become: true
  roles:
    - role: ansible-role-rpi
      vars:
        rpi_hostname: "rpi-01"
        rpi_set_hostname: true
        rpi_enable_ufw: true
        rpi_install_docker: true
```

## Key Variables

### System
```yaml
rpi_hostname: "{{ inventory_hostname }}"
rpi_timezone: "America/New_York"
rpi_upgrade_type: "dist"  # dist, full, safe
```

### Security
```yaml
rpi_ssh_port: 22
rpi_ssh_password_authentication: "no"
rpi_enable_ufw: true
rpi_ufw_allow_ports:
  - "22/tcp"
  - "80/tcp"
rpi_enable_fail2ban: true
```

### Users
```yaml
rpi_users:
  - name: admin
    groups: sudo
    shell: /bin/bash
    ssh_key: "ssh-rsa AAAA..."
```

### Optional Features
```yaml
rpi_install_docker: false
rpi_install_gpio_packages: false
rpi_enable_i2c: false
rpi_enable_spi: false
rpi_auto_reboot: false
```

## Example Playbook

```yaml
- hosts: raspberry_pis
  become: true
  roles:
    - role: ansible-role-rpi
      vars:
        rpi_hostname: "rpi-home-01"
        rpi_set_hostname: true
        rpi_timezone: "America/New_York"
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
```

## Security Best Practices

1. Change default passwords
2. Use SSH keys (disable password auth)
3. Enable UFW firewall
4. Enable fail2ban
5. Keep system updated
6. Disable unused services

## Documentation

For complete variable documentation, see `defaults/main.yml`.

## License

Apache 2.0

## Author

svosadtsia

# Guacamole 1.6.0 Complete Installation Tutorial

**English** | [Français](README.fr.md)

---

## Overview

This repository contains a **complete, production-ready tutorial** for installing and configuring **Apache Guacamole 1.6.0** on Ubuntu Server 24.04.3 with:

- ✅ Apache Guacamole Server & Client (1.6.0)
- ✅ Tomcat 9 (via Jammy 22.04 repository)
- ✅ MariaDB 10.11 with proper initialization
- ✅ nginx reverse proxy for HTTPS
- ✅ Let's Encrypt SSL/TLS certificates with auto-renewal
- ✅ Firewall configuration (UFW)
- ✅ Multi-user authentication (TOTP support)
- ✅ Session recording capability

---

## Key Features

### 🔒 Security First
- **HTTPS/SSL/TLS** with automatic certificate renewal
- **Let's Encrypt** integration (90-day certificates auto-renewing)
- **UFW firewall** with network isolation
- **TOTP 2FA** support for additional authentication
- **Session recording** with audit trails

### 🚀 Production Ready
- Comprehensive database initialization (no "unit file not found" errors)
- Complete reverse proxy configuration
- Auto-scaling firewall rules
- Professional documentation with real-world examples

### 📚 Well-Documented
- Step-by-step instructions (100+ detailed steps)
- Troubleshooting section with 15+ common issues
- Example configurations with dummy IPs/domains
- Bilingual documentation (English/French)

---

## Quick Start

### Prerequisites
- Ubuntu Server 24.04.3 (Minimized)
- Static IP address (e.g., `192.168.1.100`)
- Domain name (e.g., `guacamole.example.com`)
- Public IP access (optional, for remote access)
- Router/Firewall access (for port forwarding)

### Installation Overview

```bash
# 1. Install dependencies
sudo apt update && sudo apt upgrade -y
sudo apt-get install -y build-essential ... # (see tutorial for full list)

# 2. Install Guacamole Server
cd /tmp
wget https://downloads.apache.org/guacamole/1.6.0/source/guacamole-server-1.6.0.tar.gz
tar -xzf guacamole-server-1.6.0.tar.gz
cd guacamole-server-1.6.0/
sudo ./configure --with-systemd-dir=/etc/systemd/system/
sudo make && sudo make install
sudo ldconfig && sudo systemctl daemon-reload
sudo systemctl enable --now guacd

# 3. Install Tomcat 9
echo "deb http://archive.ubuntu.com/ubuntu/ jammy main universe" | sudo tee /etc/apt/sources.list.d/jammy.list
sudo apt-get update
sudo apt-get install -y tomcat9 tomcat9-admin tomcat9-common tomcat9-user

# 4. Deploy Guacamole Client
cd /tmp
wget https://downloads.apache.org/guacamole/1.6.0/binary/guacamole-1.6.0.war
sudo cp guacamole-1.6.0.war /var/lib/tomcat9/webapps/guacamole.war
sudo systemctl restart tomcat9 guacd

# 5. Configure MariaDB
sudo apt-get install -y mariadb-server mariadb-client
sudo mariadb-install-db  # CRITICAL STEP
sudo systemctl enable mariadb && sudo systemctl start mariadb

# 6. Setup nginx + Let's Encrypt
sudo apt-get install -y nginx certbot python3-certbot-nginx
sudo systemctl stop nginx tomcat9
sudo certbot certonly --standalone -d guacamole.example.com
sudo systemctl start nginx tomcat9

# 7. Configure firewall (optional)
sudo apt-get install -y ufw
sudo ufw allow from 192.168.1.0/24
sudo ufw enable
```

For detailed instructions, see the **complete tutorial** in this repository.

---

## Documentation Structure

| File | Purpose |
|------|---------|
| `README.md` | English overview (this file) |
| `README.fr.md` | French overview |
| `TUTORIAL.md` | Complete English installation guide |
| `TUTORIAL.fr.md` | Complete French installation guide |

---

## Supported Protocols

- **RDP** - Remote Desktop Protocol (Windows)
- **SSH** - Secure Shell (Linux/Unix)
- **VNC** - Virtual Network Computing (various systems)
- **TELNET** - Legacy terminal access

---

## System Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| **CPU** | 1 vCore | 2 vCores |
| **RAM** | 2 GB | 4 GB |
| **Storage** | 20 GB | 50 GB |
| **OS** | Ubuntu 22.04 LTS | Ubuntu 24.04.3 LTS |
| **Network** | 100 Mbps | 1 Gbps |

---

## Network Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        INTERNET                              │
└───────────────────────┬─────────────────────────────────────┘
                        │ Port 80/443 (Public IP)
                        ▼
            ┌───────────────────────┐
            │  Router/Firewall │
            │  192.168.1.1          │
            └───────────┬───────────┘
                        │ Port Forwarding
                        │ 80→80, 443→443
                        ▼
            ┌───────────────────────┐
            │  nginx Reverse Proxy   │
            │  192.168.1.100:80/443 │
            │  - SSL/TLS            │
            │  - Let's Encrypt      │
            │  - Auto-renewing      │
            └───────────┬───────────┘
                        │ Local Proxy
                        │ :8080
                        ▼
            ┌───────────────────────┐
            │  Tomcat 9             │
            │  localhost:8080       │
            │  - Guacamole WAR      │
            └───────────┬───────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
    ┌────────┐   ┌─────────┐   ┌──────────┐
    │ guacd  │   │ MariaDB │   │ Services │
    │ :4822  │   │ :3306   │   │ Config   │
    └────────┘   └─────────┘   └──────────┘
```

---

## Key Improvements in This Tutorial

### ✅ MariaDB Initialization Fix
The standard Ubuntu 24.04.3 installation **doesn't automatically initialize** the MariaDB data directory. This tutorial includes the critical `sudo mariadb-install-db` command that prevents the dreaded "Failed to enable unit: Unit file mariadb.service does not exist" error.

### ✅ Tomcat 9 on Ubuntu 24.04
Ubuntu 24.04 removed Tomcat 9 from the default repositories in favor of Tomcat 10+. This tutorial adds the Jammy (22.04) repository to properly install compatible Tomcat 9.

### ✅ nginx Reverse Proxy Configuration
Complete reverse proxy setup for:
- HTTP → HTTPS redirection
- WebSocket support (required for Guacamole)
- Proper header forwarding
- SSL/TLS termination

### ✅ Let's Encrypt Auto-Renewal
Automatic certificate renewal configured via `certbot.timer` with:
- Automatic renewal 30 days before expiration
- Daily renewal checks
- Zero-downtime renewal

### ✅ UFW Firewall Integration
Optional firewall configuration to restrict access to internal networks only, preventing unwanted external access.

---

## Configuration Examples

### Example 1: Internal Network Only
```bash
# Access via domain (internal DNS)
https://guacamole.example.com/

# OR access via IP
https://192.168.1.100/guacamole/

# Firewall restricts to internal network (192.168.1.0/24)
```

### Example 2: Create RDP Connection
```
Name:                SRV-WIN-01 (Production)
Protocol:            RDP
Hostname:            192.168.1.50
Port:                3389
Username:            ADMIN
Password:            YourPassword123!
Keyboard Layout:     French (Azerty)
Timezone:            Europe/Paris
Ignore Certificate:  ✓ (if self-signed)
```

### Example 3: Create SSH Connection
```
Name:                SRV-LINUX-01 (Production)
Protocol:            SSH
Hostname:            192.168.1.51
Port:                22
Username:            root
Password:            LinuxPassword2025!
```

---

## Troubleshooting

### Common Issues

**1. MariaDB Service Won't Start**
```bash
# Error: "Unit file mariadb.service does not exist"
# Solution: Run initialization
sudo mariadb-install-db
```

**2. Tomcat Not Found**
```bash
# Error: "Unable to locate package tomcat9"
# Solution: Add Jammy repository
echo "deb http://archive.ubuntu.com/ubuntu/ jammy main universe" | sudo tee /etc/apt/sources.list.d/jammy.list
sudo apt-get update
```

**3. nginx Returns 404**
```bash
# Error: "404 Not Found"
# Solution: Remove default site
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl restart nginx
```

**4. Certificate Won't Renew**
```bash
# Check timer status
sudo systemctl status certbot.timer
# Test renewal
sudo certbot renew --dry-run
```

---

## Default Credentials

⚠️ **CHANGE THESE IMMEDIATELY IN PRODUCTION**

| Component | Username | Password |
|-----------|----------|----------|
| Guacamole | `guacadmin` | `guacadmin` |
| MariaDB Root | `root` | *(set during installation)* |
| Example DB User | `gua_admin` | `SecurePass2025!` |

---

## Security Recommendations

✅ **Implemented in this tutorial:**
- HTTPS/SSL with auto-renewing certificates
- Firewall with network isolation
- Secure database initialization
- MariaDB hardening steps
- Credential management

🔐 **Additional recommended actions:**
- Enable TOTP 2FA for all users
- Configure session recording
- Setup automated backups
- Monitor authentication logs
- Regular security updates

---

## Verification Checklist

After installation, verify:

```bash
# 1. Services running
sudo systemctl status tomcat9 guacd mariadb nginx

# 2. Certificate valid
sudo certbot certificates

# 3. Auto-renewal active
sudo systemctl status certbot.timer

# 4. Ports listening
sudo ss -tulpn | grep -E '80|443|3306|4822|8080'

# 5. Database accessible
sudo mysql -u root -p -e "SHOW DATABASES;"

# 6. Test HTTP → HTTPS redirect
curl -I http://guacamole.example.com/

# 7. Test HTTPS access
curl -I https://guacamole.example.com/ 2>/dev/null | grep "HTTP"
```

---

## Contributors

- **Tutorial Creator**: [Perplexity]
- **Technical Advisor**: Maxence Dulche ([@maxencedulche](https://github.com/MDulche))
---

## References & Credits

### Official Documentation
- [Apache Guacamole Official Documentation](https://guacamole.apache.org/)
- [Tomcat 9 Documentation](https://tomcat.apache.org/tomcat-9.0-doc/)
- [MariaDB Official Documentation](https://mariadb.com/kb/en/documentation/)
- [nginx Documentation](https://nginx.org/en/docs/)

### External Resources
- [Let's Encrypt / Certbot Documentation](https://certbot.eff.org/)
- [Ubuntu Server Documentation](https://help.ubuntu.com/community/ApacheGuacamole)
- [UFW (Uncomplicated Firewall) Guide](https://help.ubuntu.com/community/UFW)
- [systemd Documentation](https://www.freedesktop.org/software/systemd/man/)

### Related Projects
- [Guacamole Docker Images](https://hub.docker.com/r/guacamole/guacamole)
- [Awesome Guacamole](https://github.com/iamckn/awesome-guacamole)
- [Guacamole Client Examples](https://github.com/apache/guacamole-client)

### Tools & Technologies Used
- **OS**: Ubuntu Server 24.04.3 LTS
- **Web Server**: nginx 1.24 (Ubuntu)
- **Application Server**: Apache Tomcat 9
- **Database**: MariaDB 10.11
- **Certificate Authority**: Let's Encrypt
- **Proxy**: Certbot (Let's Encrypt)
- **Firewall**: UFW (Uncomplicated Firewall)

---

## License

This tutorial and all documentation are provided under the **MIT License**.

```
MIT License

Copyright (c) 2025 [Maxence Dulche]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
```

---

## Changelog

### Version 1.0.0 (December 2025)
- ✅ Initial release
- ✅ Ubuntu 24.04.3 support
- ✅ Guacamole 1.6.0 installation
- ✅ nginx + Let's Encrypt configuration
- ✅ Complete troubleshooting guide
- ✅ Bilingual documentation (EN/FR)

---

## Disclaimer

This tutorial is provided "as-is" for educational purposes. While every effort has been made to ensure accuracy, no warranty is provided. Always test in a non-production environment first. The authors are not responsible for any data loss, system damage, or other consequences resulting from following this tutorial.

---

**Last Updated**: December 3, 2025

🚀 **Happy secure remote access!**

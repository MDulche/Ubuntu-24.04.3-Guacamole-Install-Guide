# Apache Guacamole 1.6.0 Complete Installation Tutorial

**English Version** | [Français](TUTORIAL.fr.md)

**Last Updated**: December 3, 2025  
**Status**: Production Ready ✅

---

## Table of Contents

1. [Prerequisites and Preparation](#prerequisites-and-preparation)
2. [Installing Dependencies](#installing-dependencies)
3. [Installing Apache Guacamole Server](#installing-apache-guacamole-server)
4. [Configuring Guacamole Directory](#configuring-guacamole-directory)
5. [Installing Tomcat 9 via Jammy Repository](#installing-tomcat-9-via-jammy-repository)
6. [Installing Apache Guacamole Client](#installing-apache-guacamole-client)
7. [Installing and Configuring MariaDB](#installing-and-configuring-mariadb)
8. [First Steps and Initial Configuration](#first-steps-and-initial-configuration)
9. [Installation Improvements](#installation-improvements)
10. [Configuring nginx Reverse Proxy with SSL/TLS](#configuring-nginx-reverse-proxy-with-ssltls)

---

## Example Configuration Used in This Tutorial

| Parameter | Dummy Value |
|-----------|-------------|
| **Guacamole VM IP** | `192.168.1.100` |
| **Domain** | `guacamole.example.com` |
| **Public IP** | `203.0.113.45` |
| **DB User** | `gua_admin` |
| **DB Password** | `SecurePass2025!` |
| **DB Name** | `guacamole_db` |
| **Let's Encrypt Email** | `admin@example.com` |
| **Router/Bbox IP** | `192.168.1.1` |

⚠️ **Adapt these values to your environment!**

---

## Prerequisites and Preparation

### Minimum Recommended Configuration
- **CPU**: 2 vCores
- **RAM**: 4 GB (minimum 2 GB)
- **Disk**: 50 GB (minimum 20 GB)
- **IP**: `192.168.1.100` (static - adapt to your network)
- **OS**: Ubuntu Server 24.04.3 LTS Minimized

### Initial VM Setup on ESXi

1. Create new Ubuntu Server 24.04.3 Minimized VM
2. Allocate recommended resources above
3. Assign static IP address: `192.168.1.100` (adapt to your network)
4. Verify network connectivity

### SSH Connection to VM

```bash
ssh ubuntu@192.168.1.100
```

At first connection, accept the SSH key and use the password set during installation.

### Create a Sudoer User (Recommended)

```bash
sudo apt-get install sudo -y
sudo useradd -m -s /bin/bash guacadmin
sudo usermod -aG sudo guacadmin
```

For following steps, prefix commands with `sudo` if using a non-root user.

---

## Installing Dependencies

Update system:

```bash
sudo apt update
sudo apt upgrade -y
```

Install all required packages for Apache Guacamole. **Important**: Do not install `mariadb-server` in this step:

```bash
sudo apt-get install -y build-essential libcairo2-dev libjpeg-turbo8-dev \
  libpng-dev libtool-bin uuid-dev libossp-uuid-dev libavcodec-dev \
  libavformat-dev libavutil-dev libswscale-dev freerdp2-dev \
  libpango1.0-dev libssh2-1-dev libtelnet-dev libvncserver-dev \
  libwebsockets-dev libpulse-dev libssl-dev libvorbis-dev libwebp-dev \
  libjpeg-turbo-progs pwgen fonts-dejavu fonts-liberation pkg-config \
  git autoconf automake default-jdk wget curl nano
```

Wait for installation to complete.

---

## Installing Apache Guacamole Server

### Step 1: Download Sources

```bash
cd /tmp
wget https://downloads.apache.org/guacamole/1.6.0/source/guacamole-server-1.6.0.tar.gz
```

### Step 2: Extraction and Preparation

```bash
tar -xzf guacamole-server-1.6.0.tar.gz
cd guacamole-server-1.6.0/
```

### Step 3: Configuration for Compilation

```bash
sudo ./configure --with-systemd-dir=/etc/systemd/system/
```

If error on `guacenc_video_alloc`:

```bash
sudo ./configure --with-systemd-dir=/etc/systemd/system/ --disable-guacenc
```

### Step 4: Compilation and Installation

```bash
sudo make
sudo make install
```

### Step 5: Service Finalization

```bash
sudo ldconfig
sudo systemctl daemon-reload
sudo systemctl enable --now guacd
sudo systemctl status guacd
```

Verification:

```bash
● guacd.service - Guacamole proxy daemon
     Loaded: loaded (/etc/systemd/system/guacd.service; enabled; vendor preset: enabled)
     Active: active (running) since ...
```

---

## Configuring Guacamole Directory

```bash
sudo mkdir -p /etc/guacamole/{extensions,lib}
ls -la /etc/guacamole/
```

---

## Installing Tomcat 9 via Jammy Repository

### Add Jammy (22.04) Repository

```bash
echo "deb http://archive.ubuntu.com/ubuntu/ jammy main universe" | sudo tee /etc/apt/sources.list.d/jammy.list
sudo apt-get update
```

### Install Tomcat 9

```bash
sudo apt-get install -y tomcat9 tomcat9-admin tomcat9-common tomcat9-user
```

### Enable and Start

```bash
sudo systemctl enable tomcat9
sudo systemctl start tomcat9
sudo systemctl status tomcat9
```

### Verify Access

```bash
sudo ss -tulpn | grep 8080
curl http://localhost:8080/
```

---

## Installing Apache Guacamole Client

### Download WAR File

```bash
cd /tmp
wget https://downloads.apache.org/guacamole/1.6.0/binary/guacamole-1.6.0.war
```

### Deploy to Tomcat

```bash
sudo cp guacamole-1.6.0.war /var/lib/tomcat9/webapps/guacamole.war
```

### Restart Services

```bash
sudo systemctl restart tomcat9 guacd
sleep 10
```

### Verify Access

Browser: `http://192.168.1.100:8080/guacamole/`

Default credentials:
- **Username**: `guacadmin`
- **Password**: `guacadmin`

---

## Installing and Configuring MariaDB

### Install MariaDB Properly

Completely uninstall first:

```bash
sudo apt-get remove -y mariadb-server mariadb-client mysql-server mysql-client
sudo apt-get purge -y mariadb-server mariadb-client mysql-server mysql-client
sudo apt-get autoremove -y
```

Reinstall:

```bash
sudo apt-get install -y mariadb-server mariadb-client
```

### Initialize Database (CRITICAL)

```bash
sudo mariadb-install-db
sudo ls -la /var/lib/mysql/ | head -10
```

### Enable and Start

```bash
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo systemctl status mariadb
```

### Secure Installation

```bash
sudo mysql_secure_installation
```

Answer prompts:
- Root password: Enter strong password (ex: `MariaDBRoot2025!`)
- Switch to unix_socket: `n`
- Remove anonymous users: `Y`
- Disable root login remotely: `Y`
- Remove test database: `Y`
- Reload privilege tables: `Y`

### Create Guacamole Database and User

```bash
sudo mysql -u root -p
```

Enter root password, then execute:

```sql
CREATE DATABASE guacamole_db;
CREATE USER 'gua_admin'@'localhost' IDENTIFIED BY 'SecurePass2025!';
GRANT SELECT,INSERT,UPDATE,DELETE ON guacamole_db.* TO 'gua_admin'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Install JDBC MySQL Extension

```bash
cd /tmp
wget https://downloads.apache.org/guacamole/1.6.0/binary/guacamole-auth-jdbc-1.6.0.tar.gz
tar -xzf guacamole-auth-jdbc-1.6.0.tar.gz
sudo mv guacamole-auth-jdbc-1.6.0/mysql/guacamole-auth-jdbc-mysql-1.6.0.jar /etc/guacamole/extensions/
```

### Install MySQL Connector

```bash
cd /tmp
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-j-9.5.0.tar.gz
tar -xzf mysql-connector-j-9.5.0.tar.gz
sudo cp mysql-connector-j-9.5.0/mysql-connector-j-9.5.0.jar /etc/guacamole/lib/
```

### Import Schema

```bash
cd /tmp/guacamole-auth-jdbc-1.6.0/mysql/schema/
cat *.sql | sudo mysql -u root -p guacamole_db
```

### Configure guacamole.properties

```bash
sudo nano /etc/guacamole/guacamole.properties
```

Add:

```properties
# MySQL Configuration
mysql-hostname: 127.0.0.1
mysql-port: 3306
mysql-database: guacamole_db
mysql-username: gua_admin
mysql-password: SecurePass2025!
```

Save (Ctrl+X, Y, Enter).

### Configure guacd.conf

```bash
sudo nano /etc/guacamole/guacd.conf
```

Add:

```ini
[server]
bind_host = 0.0.0.0
bind_port = 4822
```

### Restart All Services

```bash
sudo systemctl restart tomcat9 guacd mariadb
sleep 10
sudo systemctl status tomcat9 guacd mariadb
```

---

## First Steps and Initial Configuration

### Access Apache Guacamole

Browser: `http://192.168.1.100:8080/guacamole/`

Default credentials:
- **Username**: `guacadmin`
- **Password**: `guacadmin`

### Create Administrator Account

1. Click top right → **Settings**
2. Tab **Users** → **New User**
3. Fill:
   - **Username**: `admin-prod`
   - **Password**: `AdminGuacamole2025!`
   - **Confirm Password**: same
4. Check all permissions
5. Click **Create**

### Delete Default Account

1. **Settings** → **Users**
2. Click **guacadmin**
3. Click **Delete**

### Create Connection Group

1. **Settings** → **Connections** → **New Group**
2. **Name**: `Production Servers`
3. **Type**: `Organizational`
4. Click **Create**

### Add RDP Connection

1. **Settings** → **Connections** → **New Connection**
2. Fill:
   - **Name**: `SRV-WIN-01 (Production)`
   - **Group**: `Production Servers`
   - **Protocol**: `RDP`
   - **Hostname**: `192.168.1.50`
   - **Port**: `3389`
   - **Username**: `ADMIN`
   - **Password**: `YourWindowsPassword123!`
   - **Keyboard Layout**: `English (US)`
   - **Timezone**: `UTC`
   - **Ignore Certificate**: ✓ (if self-signed)
3. Click **Create**

### Add SSH Connection

1. **Settings** → **Connections** → **New Connection**
2. Fill:
   - **Name**: `SRV-LINUX-01 (Production)`
   - **Group**: `Production Servers`
   - **Protocol**: `SSH`
   - **Hostname**: `192.168.1.51`
   - **Port**: `22`
   - **Username**: `root`
   - **Password**: `LinuxPassword2025!`
3. Click **Create**

---

## Configuring nginx Reverse Proxy with SSL/TLS

### Prerequisites

- DNS Domain: `guacamole.example.com`
- Public IP: `203.0.113.45`
- DNS Access (GoDaddy, Namecheap, etc.)
- Router/Bbox accessible

### Step 1: Configure DNS

At your DNS registrar (GoDaddy, Namecheap):

Add A record:
```
Type: A
Name: guacamole
Value: 203.0.113.45
TTL: 3600
```

Verify propagation (10-15 min):

```bash
nslookup guacamole.example.com
```

### Step 2: Configure Router/Bbox Port Forwarding

On your **Router/Bbox** (`192.168.1.1`):

**Forwarding 1**:
```
External Port: 80
Internal Port: 80
Local IP: 192.168.1.100
```

**Forwarding 2**:
```
External Port: 443
Internal Port: 443
Local IP: 192.168.1.100
```

**IMPORTANT**: Disable "Remote Access to Bbox" on port 443.

Restart router and wait 2-3 minutes.

### Step 3: Install nginx and Certbot

```bash
sudo apt-get update
sudo apt-get install -y nginx certbot python3-certbot-nginx
```

### Step 4: Generate Let's Encrypt Certificate

Stop services:

```bash
sudo systemctl stop nginx tomcat9
sleep 2
```

Generate certificate:

```bash
sudo certbot certonly --standalone -d guacamole.example.com
```

At email prompt, enter: `admin@example.com`

Answer `Y` to prompts.

You should see:

```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/guacamole.example.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/guacamole.example.com/privkey.pem
This certificate expires on 2026-03-02.
```

Restart services:

```bash
sudo systemctl start nginx tomcat9
```

### Step 5: Configure nginx as Reverse Proxy

```bash
sudo tee /etc/nginx/sites-available/guacamole > /dev/null << 'EOF'
server {
    listen 80;
    server_name guacamole.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name guacamole.example.com;

    ssl_certificate /etc/letsencrypt/live/guacamole.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/guacamole.example.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Redirect / to /guacamole/
    location = / {
        return 301 /guacamole/;
    }

    # Proxy /guacamole/
    location /guacamole/ {
        proxy_pass http://localhost:8080/guacamole/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_buffering off;
    }
}
EOF
```

Enable:

```bash
sudo rm -f /etc/nginx/sites-enabled/default
sudo ln -sf /etc/nginx/sites-available/guacamole /etc/nginx/sites-enabled/guacamole
sudo nginx -t
sudo systemctl restart nginx
```

### Step 6: Verify Automatic Renewal

```bash
sudo systemctl status certbot.timer
```

You should see: `Active: active (running)`

Test:

```bash
sudo certbot renew --dry-run
```

Check expiration date:

```bash
sudo certbot certificates
```

---

## Step 7: Firewall (Internal Access Only)

If you want **no external access**:

```bash
sudo apt-get install -y ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.168.1.0/24
sudo ufw allow 22/tcp
sudo ufw enable
```

---

## Final Access

**Via Domain**:
```
https://guacamole.example.com/
```

**Via Internal IP**:
```
https://192.168.1.100/guacamole/
```

Both should display:
- ✅ Green lock (HTTPS)
- ✅ Guacamole login page
- ✅ Secure connection

---

## Useful Commands

### Verify Services

```bash
sudo systemctl status tomcat9 guacd mariadb nginx
```

### Restart All Services

```bash
sudo systemctl restart tomcat9 guacd mariadb nginx
```

### Check Listening Ports

```bash
sudo ss -tulpn | grep -E '80|443|3306|4822|8080'
```

### View Certificates

```bash
sudo certbot certificates
```

### nginx Logs

```bash
sudo tail -20 /var/log/nginx/error.log
sudo tail -20 /var/log/nginx/access.log
```

---

## Troubleshooting

### Guacamole Won't Load

```bash
sudo systemctl restart tomcat9
sleep 15
```

### nginx Returns 404

```bash
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl restart nginx
```

### Invalid Certificate

```bash
sudo certbot renew --force-renewal
sudo systemctl restart nginx
```

### Verify Database

```bash
sudo mysql -u root -p
SHOW DATABASES;
EXIT;
```

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

## Conclusion

✅ Apache Guacamole 1.6.0 installed
✅ HTTPS secured with Let's Encrypt
✅ Certificate auto-renewable
✅ Internal access protected by firewall
✅ Production ready

Enjoy secure remote access! 🚀

---

**Last Updated**: December 3, 2025  
**Version**: 1.0.0  
**Status**: Production Ready ✅

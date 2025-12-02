# Tutoriel Complet d'Installation d'Apache Guacamole 1.6.0

[English](TUTORIAL.md) | **Français**

**Dernière mise à jour** : 3 décembre 2025  
**Statut** : Prêt pour la production ✅

---

## Table des matières

1. [Prérequis et préparation](#prérequis-et-préparation)
2. [Installation des dépendances](#installation-des-dépendances)
3. [Installation du serveur Apache Guacamole](#installation-du-serveur-apache-guacamole)
4. [Configuration du répertoire Guacamole](#configuration-du-répertoire-guacamole)
5. [Installation de Tomcat 9 via le dépôt Jammy](#installation-de-tomcat-9-via-le-dépôt-jammy)
6. [Installation du client Apache Guacamole](#installation-du-client-apache-guacamole)
7. [Installation et configuration de MariaDB](#installation-et-configuration-de-mariadb)
8. [Premiers pas et configuration initiale](#premiers-pas-et-configuration-initiale)
9. [Améliorations de l'installation](#améliorations-de-linstallation)
10. [Configuration du proxy inverse nginx avec SSL/TLS](#configuration-du-proxy-inverse-nginx-avec-ssltls)

---

## Exemple de configuration utilisée dans ce tutoriel

| Paramètre | Valeur fictive |
|-----------|---|
| **IP VM Guacamole** | `192.168.1.100` |
| **Domaine** | `guacamole.example.com` |
| **IP publique** | `203.0.113.45` |
| **Utilisateur BD** | `gua_admin` |
| **Mot de passe BD** | `SecurePass2025!` |
| **Nom BD** | `guacamole_db` |
| **Email Let's Encrypt** | `admin@example.com` |
| **IP Routeur/Bbox** | `192.168.1.1` |

⚠️ **Adaptez ces valeurs à votre environnement !**

---

## Prérequis et préparation

### Configuration minimale recommandée
- **CPU** : 2 vCores
- **RAM** : 4 GB (minimum 2 GB)
- **Disque** : 50 GB (minimum 20 GB)
- **IP** : `192.168.1.100` (statique - à adapter à votre réseau)
- **OS** : Ubuntu Server 24.04.3 LTS Minimized

### Préparation initiale de la VM sur ESXi

1. Créer une nouvelle VM Ubuntu Server 24.04.3 Minimized
2. Allouer les ressources minimales recommandées ci-dessus
3. Assigner l'adresse IP statique : `192.168.1.100` (adapter à votre réseau)
4. Vérifier la connectivité réseau

### Connexion SSH à la VM

```bash
ssh ubuntu@192.168.1.100
```

À la première connexion, acceptez la clé SSH et utilisez le mot de passe défini lors de l'installation.

### Créer un utilisateur sudoer (recommandé)

```bash
sudo apt-get install sudo -y
sudo useradd -m -s /bin/bash guacadmin
sudo usermod -aG sudo guacadmin
```

Pour les étapes suivantes, préfixez les commandes par `sudo` si vous utilisez un utilisateur non-root.

---

## Installation des dépendances

Mise à jour du système :

```bash
sudo apt update
sudo apt upgrade -y
```

Installation de tous les paquets indispensables pour Apache Guacamole. **Important** : Ne pas installer `mariadb-server` dans cette étape :

```bash
sudo apt-get install -y build-essential libcairo2-dev libjpeg-turbo8-dev \
  libpng-dev libtool-bin uuid-dev libossp-uuid-dev libavcodec-dev \
  libavformat-dev libavutil-dev libswscale-dev freerdp2-dev \
  libpango1.0-dev libssh2-1-dev libtelnet-dev libvncserver-dev \
  libwebsockets-dev libpulse-dev libssl-dev libvorbis-dev libwebp-dev \
  libjpeg-turbo-progs pwgen fonts-dejavu fonts-liberation pkg-config \
  git autoconf automake default-jdk wget curl nano
```

Attendre la fin de l'installation complète.

---

## Installation du serveur Apache Guacamole

### Étape 1 : Télécharger les sources

```bash
cd /tmp
wget https://downloads.apache.org/guacamole/1.6.0/source/guacamole-server-1.6.0.tar.gz
```

### Étape 2 : Décompression et préparation

```bash
tar -xzf guacamole-server-1.6.0.tar.gz
cd guacamole-server-1.6.0/
```

### Étape 3 : Configuration pour la compilation

```bash
sudo ./configure --with-systemd-dir=/etc/systemd/system/
```

Si erreur sur `guacenc_video_alloc` :

```bash
sudo ./configure --with-systemd-dir=/etc/systemd/system/ --disable-guacenc
```

### Étape 4 : Compilation et installation

```bash
sudo make
sudo make install
```

### Étape 5 : Finalisation du service

```bash
sudo ldconfig
sudo systemctl daemon-reload
sudo systemctl enable --now guacd
sudo systemctl status guacd
```

Vérification :

```bash
● guacd.service - Guacamole proxy daemon
     Loaded: loaded (/etc/systemd/system/guacd.service; enabled; vendor preset: enabled)
     Active: active (running) since ...
```

---

## Configuration du répertoire Guacamole

```bash
sudo mkdir -p /etc/guacamole/{extensions,lib}
ls -la /etc/guacamole/
```

---

## Installation de Tomcat 9 via le dépôt Jammy

### Ajouter le dépôt Jammy (22.04)

```bash
echo "deb http://archive.ubuntu.com/ubuntu/ jammy main universe" | sudo tee /etc/apt/sources.list.d/jammy.list
sudo apt-get update
```

### Installer Tomcat 9

```bash
sudo apt-get install -y tomcat9 tomcat9-admin tomcat9-common tomcat9-user
```

### Activer et démarrer

```bash
sudo systemctl enable tomcat9
sudo systemctl start tomcat9
sudo systemctl status tomcat9
```

### Vérifier l'accès

```bash
sudo ss -tulpn | grep 8080
curl http://localhost:8080/
```

---

## Installation du client Apache Guacamole

### Télécharger le fichier WAR

```bash
cd /tmp
wget https://downloads.apache.org/guacamole/1.6.0/binary/guacamole-1.6.0.war
```

### Déployer dans Tomcat

```bash
sudo cp guacamole-1.6.0.war /var/lib/tomcat9/webapps/guacamole.war
```

### Redémarrer les services

```bash
sudo systemctl restart tomcat9 guacd
sleep 10
```

### Vérifier l'accès

Navigateur : `http://192.168.1.100:8080/guacamole/`

Identifiants par défaut :
- **Utilisateur** : `guacadmin`
- **Mot de passe** : `guacadmin`

---

## Installation et configuration de MariaDB

### Installer MariaDB proprement

Désinstaller complètement d'abord :

```bash
sudo apt-get remove -y mariadb-server mariadb-client mysql-server mysql-client
sudo apt-get purge -y mariadb-server mariadb-client mysql-server mysql-client
sudo apt-get autoremove -y
```

Réinstaller :

```bash
sudo apt-get install -y mariadb-server mariadb-client
```

### Initialiser la base de données (CRITIQUE)

```bash
sudo mariadb-install-db
sudo ls -la /var/lib/mysql/ | head -10
```

### Activer et démarrer

```bash
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo systemctl status mariadb
```

### Sécuriser l'installation

```bash
sudo mysql_secure_installation
```

Répondre aux questions :
- Mot de passe root : Entrer un mot de passe robuste (ex: `MariaDBRoot2025!`)
- Switch to unix_socket : `n`
- Remove anonymous users : `Y`
- Disable root login remotely : `Y`
- Remove test database : `Y`
- Reload privilege tables : `Y`

### Créer la base de données et l'utilisateur Guacamole

```bash
sudo mysql -u root -p
```

Entrer le mot de passe root, puis exécuter :

```sql
CREATE DATABASE guacamole_db;
CREATE USER 'gua_admin'@'localhost' IDENTIFIED BY 'SecurePass2025!';
GRANT SELECT,INSERT,UPDATE,DELETE ON guacamole_db.* TO 'gua_admin'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Installer l'extension JDBC MySQL

```bash
cd /tmp
wget https://downloads.apache.org/guacamole/1.6.0/binary/guacamole-auth-jdbc-1.6.0.tar.gz
tar -xzf guacamole-auth-jdbc-1.6.0.tar.gz
sudo mv guacamole-auth-jdbc-1.6.0/mysql/guacamole-auth-jdbc-mysql-1.6.0.jar /etc/guacamole/extensions/
```

### Installer le connecteur MySQL

```bash
cd /tmp
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-j-9.5.0.tar.gz
tar -xzf mysql-connector-j-9.5.0.tar.gz
sudo cp mysql-connector-j-9.5.0/mysql-connector-j-9.5.0.jar /etc/guacamole/lib/
```

### Importer le schéma

```bash
cd /tmp/guacamole-auth-jdbc-1.6.0/mysql/schema/
cat *.sql | sudo mysql -u root -p guacamole_db
```

### Configurer guacamole.properties

```bash
sudo nano /etc/guacamole/guacamole.properties
```

Ajouter :

```properties
# Configuration MySQL
mysql-hostname: 127.0.0.1
mysql-port: 3306
mysql-database: guacamole_db
mysql-username: gua_admin
mysql-password: SecurePass2025!
```

Enregistrer (Ctrl+X, Y, Entrée).

### Configurer guacd.conf

```bash
sudo nano /etc/guacamole/guacd.conf
```

Ajouter :

```ini
[server]
bind_host = 0.0.0.0
bind_port = 4822
```

### Redémarrer tous les services

```bash
sudo systemctl restart tomcat9 guacd mariadb
sleep 10
sudo systemctl status tomcat9 guacd mariadb
```

---

## Premiers pas et configuration initiale

### Accès à Apache Guacamole

Navigateur : `http://192.168.1.100:8080/guacamole/`

Identifiants par défaut :
- **Utilisateur** : `guacadmin`
- **Mot de passe** : `guacadmin`

### Créer un compte administrateur

1. Cliquer en haut à droite → **Paramètres**
2. Onglet **Utilisateurs** → **Nouvel utilisateur**
3. Remplir :
   - **Nom d'utilisateur** : `admin-prod`
   - **Mot de passe** : `AdminGuacamole2025!`
   - **Confirmer le mot de passe** : idem
4. Cocher toutes les permissions
5. Cliquer sur **Créer**

### Supprimer le compte par défaut

1. **Paramètres** → **Utilisateurs**
2. Cliquer sur **guacadmin**
3. Cliquer sur **Supprimer**

### Créer un groupe de connexion

1. **Paramètres** → **Connexions** → **Nouveau groupe**
2. **Nom** : `Serveurs Production`
3. **Type** : `Organisationnel`
4. Cliquer sur **Créer**

### Ajouter une connexion RDP

1. **Paramètres** → **Connexions** → **Nouvelle connexion**
2. Remplir :
   - **Nom** : `SRV-WIN-01 (Production)`
   - **Groupe** : `Serveurs Production`
   - **Protocole** : `RDP`
   - **Nom d'hôte** : `192.168.1.50`
   - **Port** : `3389`
   - **Nom d'utilisateur** : `ADMIN`
   - **Mot de passe** : `YourWindowsPassword123!`
   - **Disposition clavier** : `Français (Azerty)`
   - **Fuseau horaire** : `Europe/Paris`
   - **Ignorer le certificat du serveur** : ✓ (si auto-signé)
3. Cliquer sur **Créer**

### Ajouter une connexion SSH

1. **Paramètres** → **Connexions** → **Nouvelle connexion**
2. Remplir :
   - **Nom** : `SRV-LINUX-01 (Production)`
   - **Groupe** : `Serveurs Production`
   - **Protocole** : `SSH`
   - **Nom d'hôte** : `192.168.1.51`
   - **Port** : `22`
   - **Nom d'utilisateur** : `root`
   - **Mot de passe** : `LinuxPassword2025!`
3. Cliquer sur **Créer**

---

## Configuration du proxy inverse nginx avec SSL/TLS

### Prérequis

- Domaine DNS : `guacamole.example.com`
- IP publique : `203.0.113.45`
- Accès DNS (ex: GoDaddy, Namecheap, etc.)
- Routeur/Bbox accessible

### Étape 1 : Configurer le DNS

Chez votre registrar DNS (ex: GoDaddy, Namecheap) :

Ajouter enregistrement A :
```
Type : A
Nom : guacamole
Valeur : 203.0.113.45
TTL : 3600
```

Vérifier la propagation (10-15 min) :

```bash
nslookup guacamole.example.com
```

### Étape 2 : Configurer la redirection de ports sur le routeur/Bbox

Sur votre **Routeur/Bbox** (`192.168.1.1`) :

**Redirection 1** :
```
Port externe : 80
Port interne : 80
IP locale : 192.168.1.100
```

**Redirection 2** :
```
Port externe : 443
Port interne : 443
IP locale : 192.168.1.100
```

**IMPORTANT** : Désactiver "Accès à distance de la Bbox" sur le port 443.

Redémarrer le routeur et attendre 2-3 minutes.

### Étape 3 : Installer nginx et Certbot

```bash
sudo apt-get update
sudo apt-get install -y nginx certbot python3-certbot-nginx
```

### Étape 4 : Générer le certificat Let's Encrypt

Arrêter les services :

```bash
sudo systemctl stop nginx tomcat9
sleep 2
```

Générer le certificat :

```bash
sudo certbot certonly --standalone -d guacamole.example.com
```

À la question email, entrer : `admin@example.com`

Répondre `Y` aux questions.

Vous devez voir :

```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/guacamole.example.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/guacamole.example.com/privkey.pem
This certificate expires on 2026-03-02.
```

Redémarrer les services :

```bash
sudo systemctl start nginx tomcat9
```

### Étape 5 : Configurer nginx en tant que proxy inverse

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

    # Rediriger / vers /guacamole/
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

Activer :

```bash
sudo rm -f /etc/nginx/sites-enabled/default
sudo ln -sf /etc/nginx/sites-available/guacamole /etc/nginx/sites-enabled/guacamole
sudo nginx -t
sudo systemctl restart nginx
```

### Étape 6 : Vérifier le renouvellement automatique

```bash
sudo systemctl status certbot.timer
```

Vous devez voir : `Active: active (running)`

Tester :

```bash
sudo certbot renew --dry-run
```

Vérifier la date d'expiration :

```bash
sudo certbot certificates
```

---

## Étape 7 : Pare-feu (accès interne uniquement)

Si vous ne voulez **pas d'accès externe** :

```bash
sudo apt-get install -y ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.168.1.0/24
sudo ufw allow 22/tcp
sudo ufw enable
```

---

## Accès final

**Via le domaine** :
```
https://guacamole.example.com/
```

**Via l'IP interne** :
```
https://192.168.1.100/guacamole/
```

Les deux doivent afficher :
- ✅ Cadenas vert (HTTPS)
- ✅ Page de connexion Guacamole
- ✅ Connexion sécurisée

---

## Commandes utiles

### Vérifier les services

```bash
sudo systemctl status tomcat9 guacd mariadb nginx
```

### Redémarrer tous les services

```bash
sudo systemctl restart tomcat9 guacd mariadb nginx
```

### Vérifier les ports en écoute

```bash
sudo ss -tulpn | grep -E '80|443|3306|4822|8080'
```

### Voir les certificats

```bash
sudo certbot certificates
```

### Logs nginx

```bash
sudo tail -20 /var/log/nginx/error.log
sudo tail -20 /var/log/nginx/access.log
```

---

## Dépannage

### Guacamole ne se charge pas

```bash
sudo systemctl restart tomcat9
sleep 15
```

### nginx retourne 404

```bash
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl restart nginx
```

### Certificat invalide

```bash
sudo certbot renew --force-renewal
sudo systemctl restart nginx
```

### Vérifier la base de données

```bash
sudo mysql -u root -p
SHOW DATABASES;
EXIT;
```

---

## Liste de vérification après installation

Après l'installation, vérifiez :

```bash
# 1. Services en cours d'exécution
sudo systemctl status tomcat9 guacd mariadb nginx

# 2. Certificat valide
sudo certbot certificates

# 3. Auto-renouvellement actif
sudo systemctl status certbot.timer

# 4. Ports en écoute
sudo ss -tulpn | grep -E '80|443|3306|4822|8080'

# 5. Base de données accessible
sudo mysql -u root -p -e "SHOW DATABASES;"

# 6. Tester la redirection HTTP → HTTPS
curl -I http://guacamole.example.com/

# 7. Tester l'accès HTTPS
curl -I https://guacamole.example.com/ 2>/dev/null | grep "HTTP"
```

---

## Conclusion

✅ Apache Guacamole 1.6.0 installé
✅ HTTPS sécurisé avec Let's Encrypt
✅ Certificat auto-renouvelable
✅ Accès interne protégé par pare-feu
✅ Prêt pour la production

Bon accès distant sécurisé ! 🚀

---

## Support et problèmes

**Avez-vous des problèmes ?**

1. Consultez d'abord [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
2. Vérifiez les logs pertinents
3. Assurez-vous que tous les prérequis sont satisfaits
4. Ouvrez un problème GitHub avec :
   - Version d'Ubuntu
   - Message d'erreur complet
   - Étapes complétées
   - L'étape où vous avez rencontré un problème

---

**Dernière mise à jour** : 3 décembre 2025  
**Version** : 1.0.0  
**Statut** : Prêt pour la production ✅

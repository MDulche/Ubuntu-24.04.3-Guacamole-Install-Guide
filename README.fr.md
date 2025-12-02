# Tutoriel Complet d'Installation de Guacamole 1.6.0

[English](README.md) | **Français**

---

## Vue d'ensemble

Ce dépôt contient un **tutoriel complet et prêt pour la production** pour installer et configurer **Apache Guacamole 1.6.0** sur Ubuntu Server 24.04.3 avec :

- ✅ Serveur et Client Apache Guacamole (1.6.0)
- ✅ Tomcat 9 (via dépôt Jammy 22.04)
- ✅ MariaDB 10.11 avec initialisation correcte
- ✅ Reverse proxy nginx pour HTTPS
- ✅ Certificats SSL/TLS Let's Encrypt avec auto-renouvellement
- ✅ Configuration du pare-feu (UFW)
- ✅ Authentification multi-utilisateurs (support TOTP)
- ✅ Capacité d'enregistrement de sessions

---

## Caractéristiques principales

### 🔒 Sécurité d'abord
- **HTTPS/SSL/TLS** avec renouvellement automatique des certificats
- Intégration **Let's Encrypt** (certificats 90 jours s'auto-renouvelant)
- **Pare-feu UFW** avec isolation réseau
- Support **TOTP 2FA** pour une authentification supplémentaire
- **Enregistrement de sessions** avec pistes d'audit

### 🚀 Prêt pour la production
- Initialisation complète de la base de données (pas d'erreurs "unit file not found")
- Configuration complète du reverse proxy
- Règles de pare-feu auto-adaptatives
- Documentation professionnelle avec exemples réels

### 📚 Bien documenté
- Instructions étape par étape (100+ étapes détaillées)
- Section dépannage avec 15+ problèmes courants
- Configurations d'exemple avec IPs/domaines fictifs
- Documentation bilingue (Anglais/Français)

---

## Démarrage rapide

### Prérequis
- Ubuntu Server 24.04.3 (Minimized)
- Adresse IP statique (ex: `192.168.1.100`)
- Nom de domaine (ex: `guacamole.example.com`)
- Accès IP publique (optionnel, pour accès distant)
- Accès au routeur/Firewall (pour redirection de ports)

### Aperçu de l'installation

```bash
# 1. Installer les dépendances
sudo apt update && sudo apt upgrade -y
sudo apt-get install -y build-essential ... # (voir tutoriel pour la liste complète)

# 2. Installer le serveur Guacamole
cd /tmp
wget https://downloads.apache.org/guacamole/1.6.0/source/guacamole-server-1.6.0.tar.gz
tar -xzf guacamole-server-1.6.0.tar.gz
cd guacamole-server-1.6.0/
sudo ./configure --with-systemd-dir=/etc/systemd/system/
sudo make && sudo make install
sudo ldconfig && sudo systemctl daemon-reload
sudo systemctl enable --now guacd

# 3. Installer Tomcat 9
echo "deb http://archive.ubuntu.com/ubuntu/ jammy main universe" | sudo tee /etc/apt/sources.list.d/jammy.list
sudo apt-get update
sudo apt-get install -y tomcat9 tomcat9-admin tomcat9-common tomcat9-user

# 4. Déployer le client Guacamole
cd /tmp
wget https://downloads.apache.org/guacamole/1.6.0/binary/guacamole-1.6.0.war
sudo cp guacamole-1.6.0.war /var/lib/tomcat9/webapps/guacamole.war
sudo systemctl restart tomcat9 guacd

# 5. Configurer MariaDB
sudo apt-get install -y mariadb-server mariadb-client
sudo mariadb-install-db  # ÉTAPE CRITIQUE
sudo systemctl enable mariadb && sudo systemctl start mariadb

# 6. Installer nginx + Let's Encrypt
sudo apt-get install -y nginx certbot python3-certbot-nginx
sudo systemctl stop nginx tomcat9
sudo certbot certonly --standalone -d guacamole.example.com
sudo systemctl start nginx tomcat9

# 7. Configurer le pare-feu (optionnel)
sudo apt-get install -y ufw
sudo ufw allow from 192.168.1.0/24
sudo ufw enable
```

Pour des instructions détaillées, consultez le **tutoriel complet** dans ce dépôt.

---

## Structure de la documentation

| Fichier | Objectif |
|---------|----------|
| `README.md` | Vue d'ensemble anglaise |
| `README.fr.md` | Vue d'ensemble française (ce fichier) |
| `TUTORIAL.md` | Guide complet d'installation en anglais |
| `TUTORIAL.fr.md` | Guide complet d'installation en français |
| `EXAMPLES.md` | Exemples avec IPs/domaines fictifs |
| `TROUBLESHOOTING.md` | 15+ problèmes courants et solutions |
| `CONTRIBUTING.md` | Comment contribuer |
| `LICENSE` | Licence MIT |

---

## Protocoles supportés

- **RDP** - Remote Desktop Protocol (Windows)
- **SSH** - Secure Shell (Linux/Unix)
- **VNC** - Virtual Network Computing (systèmes divers)
- **TELNET** - Accès terminal hérité

---

## Configuration requise

| Ressource | Minimum | Recommandé |
|-----------|---------|-----------|
| **CPU** | 1 vCore | 2 vCores |
| **RAM** | 2 GB | 4 GB |
| **Stockage** | 20 GB | 50 GB |
| **OS** | Ubuntu 22.04 LTS | Ubuntu 24.04.3 LTS |
| **Réseau** | 100 Mbps | 1 Gbps |

---

## Architecture réseau

```
┌─────────────────────────────────────────────────────────────┐
│                        INTERNET                              │
└───────────────────────┬─────────────────────────────────────┘
                        │ Port 80/443 (IP publique)
                        ▼
            ┌───────────────────────┐
            │  Routeur/Firewall     │
            │  192.168.1.1          │
            └───────────┬───────────┘
                        │ Redirection de ports
                        │ 80→80, 443→443
                        ▼
            ┌───────────────────────┐
            │  nginx Reverse Proxy  │
            │  192.168.1.100:80/443 │
            │  - SSL/TLS            │
            │  - Let's Encrypt      │
            │  - Auto-renouvelable  │
            └───────────┬───────────┘
                        │ Proxy local
                        │ :8080
                        ▼
            ┌───────────────────────┐
            │  Tomcat 9             │
            │  localhost:8080       │
            │  - WAR Guacamole      │
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

## Améliorations clés de ce tutoriel

### ✅ Correction de l'initialisation MariaDB
L'installation standard d'Ubuntu 24.04.3 **n'initialise pas automatiquement** le répertoire de données MariaDB. Ce tutoriel inclut la commande critique `sudo mariadb-install-db` qui prévient l'erreur redoutée "Failed to enable unit: Unit file mariadb.service does not exist".

### ✅ Tomcat 9 sur Ubuntu 24.04
Ubuntu 24.04 a supprimé Tomcat 9 des dépôts par défaut au profit de Tomcat 10+. Ce tutoriel ajoute le dépôt Jammy (22.04) pour installer correctement Tomcat 9 compatible.

### ✅ Configuration du reverse proxy nginx
Configuration complète du reverse proxy pour :
- Redirection HTTP → HTTPS
- Support WebSocket (requis pour Guacamole)
- Transfert correct des en-têtes
- Terminaison SSL/TLS

### ✅ Auto-renouvellement Let's Encrypt
Renouvellement automatique des certificats configuré via `certbot.timer` avec :
- Renouvellement automatique 30 jours avant expiration
- Vérifications quotidiennes de renouvellement
- Renouvellement sans temps d'arrêt

### ✅ Intégration du pare-feu UFW
Configuration optionnelle du pare-feu pour restreindre l'accès aux réseaux internes uniquement, prévenant l'accès indésirable de l'extérieur.

---

## Exemples de configuration

### Exemple 1 : Réseau interne uniquement
```bash
# Accès via domaine (DNS interne)
https://guacamole.example.com/

# OU accès via IP
https://192.168.1.100/guacamole/

# Le pare-feu restreint au réseau interne (192.168.1.0/24)
```

### Exemple 2 : Créer une connexion RDP
```
Nom :                  SRV-WIN-01 (Production)
Protocole :            RDP
Nom d'hôte :           192.168.1.50
Port :                 3389
Nom d'utilisateur :    ADMIN
Mot de passe :         YourPassword123!
Disposition clavier :  Français (Azerty)
Fuseau horaire :       Europe/Paris
Ignorer certificat :   ✓ (si auto-signé)
```

### Exemple 3 : Créer une connexion SSH
```
Nom :                  SRV-LINUX-01 (Production)
Protocole :            SSH
Nom d'hôte :           192.168.1.51
Port :                 22
Nom d'utilisateur :    root
Mot de passe :         LinuxPassword2025!
```

---

## Dépannage

### Problèmes courants

**1. Le service MariaDB ne démarre pas**
```bash
# Erreur : "Unit file mariadb.service does not exist"
# Solution : Exécuter l'initialisation
sudo mariadb-install-db
```

**2. Tomcat introuvable**
```bash
# Erreur : "Unable to locate package tomcat9"
# Solution : Ajouter le dépôt Jammy
echo "deb http://archive.ubuntu.com/ubuntu/ jammy main universe" | sudo tee /etc/apt/sources.list.d/jammy.list
sudo apt-get update
```

**3. nginx retourne 404**
```bash
# Erreur : "404 Not Found"
# Solution : Supprimer le site par défaut
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl restart nginx
```

**4. Le certificat ne se renouvelle pas**
```bash
# Vérifier le statut du timer
sudo systemctl status certbot.timer
# Tester le renouvellement
sudo certbot renew --dry-run
```

---

## Identifiants par défaut

⚠️ **CHANGEZ CES ÉLÉMENTS IMMÉDIATEMENT EN PRODUCTION**

| Composant | Nom d'utilisateur | Mot de passe |
|-----------|-------------------|--------------|
| Guacamole | `guacadmin` | `guacadmin` |
| Racine MariaDB | `root` | *(défini lors de l'installation)* |
| Utilisateur BD exemple | `gua_admin` | `SecurePass2025!` |

---

## Recommandations de sécurité

✅ **Implémentées dans ce tutoriel :**
- HTTPS/SSL avec certificats s'auto-renouvelant
- Pare-feu avec isolation réseau
- Initialisation sécurisée de la base de données
- Étapes de renforcement de MariaDB
- Gestion des identifiants

🔐 **Actions supplémentaires recommandées :**
- Activer TOTP 2FA pour tous les utilisateurs
- Configurer l'enregistrement de sessions
- Configurer des sauvegardes automatisées
- Monitorer les journaux d'authentification
- Mises à jour de sécurité régulières

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

# 4. Ports à l'écoute
sudo ss -tulpn | grep -E '80|443|3306|4822|8080'

# 5. Base de données accessible
sudo mysql -u root -p -e "SHOW DATABASES;"

# 6. Tester la redirection HTTP → HTTPS
curl -I http://guacamole.example.com/

# 7. Tester l'accès HTTPS
curl -I https://guacamole.example.com/ 2>/dev/null | grep "HTTP"
```

---

## Contributeurs

- **Créateur du tutoriel** : [Perplexity.com]
- **Conseiller technique** : Maxence Dulche ([@maxencedulche](https://github.com/MDulche))

---

## Références et crédits

### Documentation officielle
- [Documentation officielle Apache Guacamole](https://guacamole.apache.org/)
- [Documentation Tomcat 9](https://tomcat.apache.org/tomcat-9.0-doc/)
- [Documentation officielle MariaDB](https://mariadb.com/kb/en/documentation/)
- [Documentation nginx](https://nginx.org/en/docs/)

### Ressources externes
- [Documentation Let's Encrypt / Certbot](https://certbot.eff.org/)
- [Documentation Ubuntu Server](https://help.ubuntu.com/community/ApacheGuacamole)
- [Guide UFW (Uncomplicated Firewall)](https://help.ubuntu.com/community/UFW)
- [Documentation systemd](https://www.freedesktop.org/software/systemd/man/)

### Projets connexes
- [Images Docker Guacamole](https://hub.docker.com/r/guacamole/guacamole)
- [Awesome Guacamole](https://github.com/iamckn/awesome-guacamole)
- [Exemples Client Guacamole](https://github.com/apache/guacamole-client)

### Outils et technologies utilisés
- **OS** : Ubuntu Server 24.04.3 LTS
- **Serveur Web** : nginx 1.24 (Ubuntu)
- **Serveur d'application** : Apache Tomcat 9
- **Base de données** : MariaDB 10.11
- **Autorité de certificat** : Let's Encrypt
- **Proxy** : Certbot (Let's Encrypt)
- **Pare-feu** : UFW (Uncomplicated Firewall)

---

## Licence

Ce tutoriel et toute la documentation sont fournis sous la **Licence MIT**.

```
Licence MIT

Copyright (c) 2025 [Maxence Dulche]

Par la présente, toute personne obtenant une copie de ce logiciel et 
des fichiers de documentation associés (le « Logiciel ») est autorisée 
à traiter le Logiciel sans restriction, y compris, sans limitation, 
les droits d'utilisation, de copie, de modification, de fusion, de 
publication, de distribution, de sous-licence et/ou de vente des copies 
du Logiciel...
```

---

## Journal des modifications

### Version 1.0.0 (Décembre 2025)
- ✅ Version initiale
- ✅ Support Ubuntu 24.04.3
- ✅ Installation Guacamole 1.6.0
- ✅ Configuration nginx + Let's Encrypt
- ✅ Guide complet de dépannage
- ✅ Documentation bilingue (EN/FR)

---

## Clause de non-responsabilité

Ce tutoriel est fourni « tel quel » à des fins éducatives. Bien que tous les efforts aient été faits pour assurer l'exactitude, aucune garantie n'est fournie. Testez toujours d'abord dans un environnement de non-production. Les auteurs ne sont pas responsables des pertes de données, dommages au système ou d'autres conséquences résultant du suivi de ce tutoriel.

---

**Dernière mise à jour** : 3 décembre 2025


🚀 **Bon accès distant sécurisé !**

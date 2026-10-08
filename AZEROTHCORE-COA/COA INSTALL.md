Voici un guide détaillé pour installer un serveur dédié AzerothCore WotLK (fork CoA) sur une machine virtuelle Oracle VM VirtualBox. Ce guide couvre l'installation complète, depuis la création de la VM jusqu'au démarrage du serveur.

---

# 📘 Guide d'installation complet : Serveur dédié AzerothCore WotLK (CoA) sur Oracle VM VirtualBox

## 📋 Table des matières

1. [Présentation et choix techniques](#1-présentation-et-choix-techniques)
2. [Prérequis matériels et logiciels](#2-prérequis-matériels-et-logiciels)
3. [Création de la machine virtuelle VirtualBox](#3-création-de-la-machine-virtuelle-virtualbox)
4. [Installation du système d'exploitation Ubuntu Server](#4-installation-du-système-dexploitation-ubuntu-server)
5. [Configuration post-installation et outils invités](#5-configuration-post-installation-et-outils-invités)
6. [Installation des dépendances AzerothCore](#6-installation-des-dépendances-azerothcore)
7. [Configuration de MySQL](#7-configuration-de-mysql)
8. [Compilation du core AzerothCore (fork CoA)](#8-compilation-du-core-azerothcore-fork-coa)
9. [Configuration des bases de données](#9-configuration-des-bases-de-données)
10. [Configuration des fichiers .conf](#10-configuration-des-fichiers-conf)
11. [Extraction des données client (DBC, Maps, VMaps, MMaps)](#11-extraction-des-données-client)
12. [Démarrage et test du serveur](#12-démarrage-et-test-du-serveur)
13. [Configuration réseau et accès externe](#13-configuration-réseau-et-accès-externe)
14. [Maintenance et mises à jour](#14-maintenance-et-mises-à-jour)
15. [Dépannage et erreurs courantes](#15-dépannage-et-erreurs-courantes)

---

## 1. Présentation et choix techniques

### 1.1 Qu'est-ce qu'AzerothCore ?

AzerothCore est un serveur de jeu open-source et un framework conçu pour héberger des MMORPG. Il est basé sur le populaire World of Warcraft et cherche à recréer l'expérience de jeu de la version 3.3.5a (Wrath of the Lich King). Le code original est basé sur MaNGOS, TrinityCore et SunwellCore, et a depuis fait l'objet d'un développement intensif pour améliorer la stabilité, les mécaniques de jeu et la modularité.

### 1.2 Pourquoi un serveur dédié sur VM ?

L'option du serveur dédié sur machine virtuelle offre plusieurs avantages :
- **Isolation totale** : Aucun conflit avec une installation existante
- **Flexibilité** : Possibilité de sauvegarder, cloner ou migrer la VM facilement
- **Sécurité** : Environnement sandboxé, isolé du système hôte
- **Test et développement** : Idéal pour expérimenter sans risque

### 1.3 Architecture cible

```
┌─────────────────────────────────────────────────────────────┐
│                    Machine Hôte (Windows/Linux)              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              Oracle VM VirtualBox                      │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │           VM Ubuntu Server 22.04/24.04          │  │  │
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌────────┐  │  │  │
│  │  │  │  authserver  │  │ worldserver │  │ MySQL  │  │  │  │
│  │  │  │  (port 3724) │  │  (port 8085)│  │ (3306) │  │  │  │
│  │  │  └─────────────┘  └─────────────┘  └────────┘  │  │  │
│  │  │  ┌─────────────────────────────────────────────┐ │  │  │
│  │  │  │        Bases de données                     │ │  │  │
│  │  │  │  acore_auth │ acore_world │ acore_characters│ │  │  │
│  │  │  └─────────────────────────────────────────────┘ │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Prérequis matériels et logiciels

### 2.1 Configuration matérielle recommandée pour l'hôte

| Composant | Minimum | Recommandé | Idéal |
|-----------|---------|------------|-------|
| **CPU** | 4 cœurs | 6-8 cœurs | 12+ cœurs |
| **RAM** | 8 Go | 16 Go | 32 Go |
| **Stockage** | 50 Go libres | 100 Go SSD | 200 Go SSD NVMe |
| **Réseau** | 10 Mbps | 100 Mbps | 1 Gbps |

> ⚠️ **Important** : La compilation d'AzerothCore est gourmande en CPU et en RAM. Prévoyez au moins 4 Go de RAM dédiés à la VM pour la compilation, et 2 Go minimum pour l'exécution.

### 2.2 Configuration recommandée pour la VM

| Ressource | Allocation recommandée |
|-----------|----------------------|
| **CPU** | 4 cœurs (ne pas dépasser 50% des cœurs physiques de l'hôte) |
| **RAM** | 4-8 Go (30% de la RAM de l'hôte) |
| **Disque** | 60-80 Go (VDI, dynamiquement alloué) |
| **Carte réseau** | Accès par pont (Bridged) ou NAT avec redirection de ports |

### 2.3 Logiciels nécessaires

| Logiciel | Version | Téléchargement |
|----------|---------|----------------|
| Oracle VM VirtualBox | 7.0+ | [virtualbox.org](https://www.virtualbox.org/) |
| Ubuntu Server ISO | 22.04 LTS ou 24.04 LTS | [ubuntu.com](https://ubuntu.com/download/server) |
| Client WoW 3.3.5a | Build 12340 | (légalement possédé) |

### 2.4 Prérequis techniques AzerothCore

| Composant | Version minimale |
|-----------|------------------|
| MySQL | 8.4 LTS (MySQL 26.x non supporté) |
| Boost | ≥ 1.74 |
| OpenSSL | ≥ 3.0.x |
| CMake | ≥ 3.16 |
| Clang | ≥ 10 |

---

## 3. Création de la machine virtuelle VirtualBox

### 3.1 Ouvrir VirtualBox et créer une nouvelle VM

1. Lancez **Oracle VM VirtualBox**
2. Cliquez sur **"Nouvelle"** (ou `Ctrl+N`)
3. Remplissez les informations :

| Champ | Valeur |
|-------|--------|
| **Nom** | `AzerothCore-Server` |
| **Dossier** | Chemin où stocker la VM |
| **Type** | Linux |
| **Version** | Ubuntu (64-bit) |
| **ISO Image** | Sélectionnez l'ISO Ubuntu Server téléchargée |

4. Cochez **"Skip Unattended Installation"** (installation manuelle recommandée)

### 3.2 Configuration de la mémoire et du CPU

1. **Mémoire de base** : Allouez **4096 Mo (4 Go)** minimum, **8192 Mo (8 Go)** recommandé
2. **Processeurs** : Allouez **4 cœurs** (ou plus si disponible)
   - Ne dépassez pas 50% des cœurs physiques de l'hôte

### 3.3 Configuration du disque dur

1. Sélectionnez **"Créer un disque dur virtuel maintenant"**
2. Type de disque : **VDI (VirtualBox Disk Image)**
3. Stockage : **Dynamiquement alloué**
4. Taille : **80 Go** (minimum 60 Go)
5. Emplacement : Choisissez un disque SSD si possible pour de meilleures performances

### 3.4 Configuration réseau

1. Allez dans **Configuration → Réseau**
2. **Mode d'accès réseau** : 
   - **Accès par pont (Bridged)** : La VM obtient une IP sur votre réseau local (recommandé pour un serveur)
   - **NAT** : La VM est derrière un NAT, nécessite une redirection de ports
3. **Type de carte** : `Intel PRO/1000 MT Desktop`

### 3.5 Configuration de l'affichage

1. **Mémoire vidéo** : 16-32 Mo (pas besoin de plus pour un serveur headless)
2. **Contrôleur graphique** : VMSVGA
3. Décochez **"Activer l'accélération 3D"** (inutile pour un serveur)

### 3.6 Configuration système

1. **Carte mère** : 
   - Chipset : PIIX3
   - Décochez "Activer EFI" (sauf si vous utilisez un disque GPT)
2. **Processeur** : 
   - Cochez **"Activer PAE/NX"**
   - Cochez **"Activer VT-x/AMD-V"** et **"Activer Nested Paging"**

---

## 4. Installation du système d'exploitation Ubuntu Server

### 4.1 Démarrage de l'installation

1. Sélectionnez la VM et cliquez sur **"Démarrer"**
2. L'installateur Ubuntu Server devrait démarrer

### 4.2 Étapes de l'installation

| Étape | Choix recommandé |
|-------|------------------|
| **Langue** | Français ou English |
| **Clavier** | Français (ou votre disposition) |
| **Type d'installation** | Ubuntu Server (pas minimized) |
| **Réseau** | Détection automatique DHCP |
| **Proxy** | Laisser vide |
| **Miroir** | Laisser par défaut |
| **Stockage** | Use an entire disk (avec LVM) |
| **Utilisateur** | `azeroth` (ou `admin`) |
| **Mot de passe** | Mot de passe fort |
| **Serveur SSH** | ✅ Installer OpenSSH server |
| **Snaps** | Ne rien sélectionner |

### 4.3 Finalisation

1. Attendez la fin de l'installation
2. **Redémarrez** la VM
3. Retirez l'ISO du lecteur virtuel (ou appuyez sur Entrée)
4. Connectez-vous avec vos identifiants

### 4.4 Première connexion et mise à jour

```bash
# Mettre à jour le système
sudo apt update && sudo apt upgrade -y

# Vérifier la version d'Ubuntu
lsb_release -a

# Vérifier l'espace disque
df -h
```

---

## 5. Configuration post-installation et outils invités

### 5.1 Installation des outils invités VirtualBox (Guest Additions)

Les Guest Additions améliorent les performances et permettent le partage de dossiers.

```bash
# Installer les dépendances nécessaires
sudo apt update
sudo apt install -y build-essential dkms linux-headers-$(uname -r)

# Dans le menu VirtualBox : Périphériques → Insérer l'image CD des Guest Additions
# Puis monter le CD
sudo mkdir -p /mnt/cdrom
sudo mount /dev/cdrom /mnt/cdrom

# Exécuter l'installateur
cd /mnt/cdrom
sudo ./VBoxLinuxAdditions.run

# Redémarrer
sudo reboot
```

### 5.2 Désactiver les mises à jour automatiques (critique)

AzerothCore peut être perturbé par les mises à jour automatiques du système, notamment MySQL qui peut être mis à jour pendant que le serveur tourne.

```bash
# Éditer le fichier de configuration des mises à jour automatiques
sudo nano /etc/apt/apt.conf.d/20auto-upgrades
```

Commentez toutes les lignes en ajoutant `//` au début :

```
// APT::Periodic::Update-Package-Lists "1";
// APT::Periodic::Unattended-Upgrade "1";
```

Sauvegardez (`Ctrl+X`, `Y`, `Entrée`) et redémarrez :

```bash
sudo reboot
```

### 5.3 Créer un utilisateur dédié (optionnel)

```bash
# Créer un utilisateur pour AzerothCore
sudo adduser azeroth
sudo usermod -aG sudo azeroth

# Se connecter en tant qu'azeroth
su - azeroth
```

### 5.4 Configuration du fuseau horaire

```bash
sudo timedatectl set-timezone Europe/Paris
```

---

## 6. Installation des dépendances AzerothCore

### 6.1 Mise à jour des dépôts

```bash
sudo apt update
```

### 6.2 Installation des paquets de base

Pour **Ubuntu 22.04 LTS** :

```bash
sudo apt install -y git cmake make gcc g++ clang libssl-dev libbz2-dev \
  libreadline-dev libncurses-dev libboost-all-dev lsb-release gnupg wget screen \
  libmysqlclient-dev
```

Pour **Ubuntu 24.04 LTS** :

```bash
sudo apt install -y git cmake make gcc g++ clang libssl-dev libbz2-dev \
  libreadline-dev libncurses-dev libboost-all-dev lsb-release gnupg wget
```

> 📝 **Note** : Ubuntu 24.04 ne fournit pas MySQL dans ses dépôts par défaut. Nous l'installerons manuellement à l'étape suivante.

### 6.3 Vérification des versions

```bash
# Vérifier les versions installées
cmake --version
clang --version
gcc --version

# Vérifier Boost
dpkg -l | grep libboost
```

### 6.4 Installation de MySQL 8.4 LTS (Ubuntu 24.04 uniquement)

Si vous utilisez Ubuntu 24.04, suivez ces étapes pour installer MySQL 8.4 LTS :

```bash
# Télécharger le package de configuration du dépôt MySQL
export MYSQL_APT_CONFIG_VERSION=0.8.36-1
wget https://dev.mysql.com/get/mysql-apt-config_${MYSQL_APT_CONFIG_VERSION}_all.deb

# Installer le package de configuration
sudo dpkg -i mysql-apt-config_${MYSQL_APT_CONFIG_VERSION}_all.deb
# Sélectionnez "MySQL 8.4 LTS" dans le menu

# Mettre à jour et installer MySQL
sudo apt update
sudo apt install -y mysql-server

# Vérifier l'installation
mysql --version
```

---

## 7. Configuration de MySQL

### 7.1 Sécurisation de l'installation MySQL

```bash
sudo mysql_secure_installation
```

Répondez aux questions :
- **VALIDATE PASSWORD COMPONENT** : `N` (ou `Y` selon vos préférences)
- **Change root password** : `Y` (choisissez un mot de passe fort)
- **Remove anonymous users** : `Y`
- **Disallow root login remotely** : `Y`
- **Remove test database** : `Y`
- **Reload privilege tables** : `Y`

### 7.2 Création de l'utilisateur AzerothCore

Connectez-vous à MySQL en tant que root :

```bash
sudo mysql -u root -p
```

Exécutez les commandes SQL suivantes :

```sql
-- Créer un utilisateur dédié
CREATE USER 'acore'@'localhost' IDENTIFIED BY 'VotreMotDePasseFort';

-- Accorder tous les privilèges sur les bases acore_*
GRANT ALL PRIVILEGES ON acore_auth.* TO 'acore'@'localhost';
GRANT ALL PRIVILEGES ON acore_world.* TO 'acore'@'localhost';
GRANT ALL PRIVILEGES ON acore_characters.* TO 'acore'@'localhost';

-- Appliquer les changements
FLUSH PRIVILEGES;

-- Quitter
EXIT;
```

### 7.3 Création des bases de données vides

```bash
sudo mysql -u root -p
```

```sql
-- Créer les trois bases de données avec le bon charset
CREATE DATABASE IF NOT EXISTS `acore_world` 
  DEFAULT CHARACTER SET UTF8MB4 COLLATE utf8mb4_unicode_ci;

CREATE DATABASE IF NOT EXISTS `acore_characters` 
  DEFAULT CHARACTER SET UTF8MB4 COLLATE utf8mb4_unicode_ci;

CREATE DATABASE IF NOT EXISTS `acore_auth` 
  DEFAULT CHARACTER SET UTF8MB4 COLLATE utf8mb4_unicode_ci;

-- Vérifier
SHOW DATABASES;
EXIT;
```

> 📝 **Note** : Le charset `utf8mb4_unicode_ci` est recommandé pour AzerothCore.

### 7.4 Vérification de la connexion

```bash
# Tester la connexion avec l'utilisateur acore
mysql -u acore -p -e "SHOW DATABASES;"
```

---

## 8. Compilation du core AzerothCore (fork CoA)

### 8.1 Clonage du dépôt CoA

> ⚠️ **Important** : Nous clonons le **fork CoA** (et non le dépôt officiel) car c'est ce fork qui contient les fonctionnalités spécifiques CoA.

```bash
# Se placer dans le répertoire home
cd ~

# Cloner le fork CoA
git clone https://github.com/jealous-sound/azerothcore-wotlk-coa.git azerothcore

# Entrer dans le répertoire
cd azerothcore

# Vérifier le dépôt distant
git remote -v
```

### 8.2 Création du répertoire de build

```bash
# Créer et entrer dans le répertoire de build
mkdir build
cd build
```

### 8.3 Configuration avec CMake

```bash
# Configurer la compilation
cmake ../ \
  -DCMAKE_INSTALL_PREFIX=$HOME/azerothcore/env/dist/ \
  -DCMAKE_C_COMPILER=/usr/bin/clang \
  -DCMAKE_CXX_COMPILER=/usr/bin/clang++ \
  -DWITH_WARNINGS=1 \
  -DTOOLS_BUILD=all \
  -DSCRIPTS=static \
  -DMODULES=static
```

**Explication des options** :
- `CMAKE_INSTALL_PREFIX` : Répertoire d'installation des binaires
- `CMAKE_C_COMPILER` / `CMAKE_CXX_COMPILER` : Compilateurs Clang
- `WITH_WARNINGS` : Active les avertissements de compilation
- `TOOLS_BUILD=all` : Compile tous les outils (extracteurs, etc.)
- `SCRIPTS=static` : Scripts liés statiquement
- `MODULES=static` : Modules liés statiquement

### 8.4 Compilation

```bash
# Compiler avec tous les cœurs disponibles
make -j$(nproc)

# Ou spécifier un nombre de threads (ex: 4)
# make -j4
```

> ⏱️ **Durée** : La compilation prend généralement **30 à 90 minutes** selon la puissance de la machine.

### 8.5 Installation

```bash
# Installer les binaires
make install
```

### 8.6 Vérification de l'installation

```bash
# Lister les binaires installés
ls -la $HOME/azerothcore/env/dist/bin/

# Vous devriez voir :
# authserver, worldserver, mapextractor, vmap4extractor, 
# vmap4assembler, mmaps_generator
```

---

## 9. Configuration des bases de données

### 9.1 Structure des données SQL

Les fichiers SQL se trouvent dans le répertoire `data/sql/` du dépôt cloné :

```bash
cd ~/azerothcore
ls -la data/sql/
# base/     - Structure de base des 3 bases de données
# updates/  - Mises à jour incrémentielles
```

### 9.2 Importation manuelle des bases (méthode simple)

#### 9.2.1 Base auth (authentification)

```bash
# Importer la structure de base
sudo mysql -u root -p acore_auth < data/sql/base/auth_database.sql
```

#### 9.2.2 Base characters (personnages)

```bash
sudo mysql -u root -p acore_characters < data/sql/base/characters_database.sql
```

#### 9.2.3 Base world (monde) - Étape spécifique CoA

> ⚠️ **IMPORTANT** : Pour ce fork CoA, le README indique explicitement :
> *"For this CoA fork, install the world database package into an empty world schema before the first worldserver startup."*

Le fork CoA fournit un outil Python dédié pour importer le package world :

```bash
# Vérifier que Python est installé
python3 --version

# Se placer dans le répertoire du fork
cd ~/azerothcore

# Vérifier le package world
python3 apps/coa-world/world_data.py verify

# Créer un fichier de connexion MySQL
cat > ~/world-client.cnf << 'EOF'
[client]
user=acore
password=VotreMotDePasseFort
host=127.0.0.1
port=3306
EOF

# Importer le package world dans le schéma vide
python3 apps/coa-world/world_data.py bootstrap \
  --defaults-file ~/world-client.cnf \
  --database acore_world
```

> 📝 **Note** : Si le script Python n'est pas disponible, vous pouvez utiliser la méthode standard avec les fichiers SQL :

```bash
# Méthode alternative : importer la structure de base world standard
sudo mysql -u root -p acore_world < data/sql/base/world_database.sql
```

### 9.3 Application des mises à jour SQL

Les mises à jour sont automatiquement appliquées par le worldserver au démarrage. Cependant, vous pouvez aussi les appliquer manuellement :

```bash
# Utiliser le script d'assemblage des mises à jour
cd ~/azerothcore
bash apps/db_assembler/db_assembler.sh

# Puis importer les fichiers générés
sudo mysql -u root -p acore_world < data/sql/updates/world/*.sql
```

### 9.4 Vérification des bases

```bash
# Vérifier le contenu des bases
mysql -u acore -p -e "USE acore_auth; SHOW TABLES;"
mysql -u acore -p -e "USE acore_world; SHOW TABLES;"
mysql -u acore -p -e "USE acore_characters; SHOW TABLES;"
```

---

## 10. Configuration des fichiers .conf

### 10.1 Copie des fichiers de configuration

```bash
# Se placer dans le répertoire d'installation
cd ~/azerothcore/env/dist/etc/

# Copier les fichiers .dist vers .conf
cp authserver.conf.dist authserver.conf
cp worldserver.conf.dist worldserver.conf

# Vérifier
ls -la *.conf
```

### 10.2 Configuration de authserver.conf

```bash
nano authserver.conf
```

Modifiez les paramètres suivants :

```ini
# Connexion à la base de données d'authentification
LoginDatabaseInfo = "127.0.0.1;3306;acore;VotreMotDePasseFort;acore_auth"

# Port d'écoute de l'authserver
WorldServerPort = 8085

# Bind IP (0.0.0.0 pour écouter sur toutes les interfaces)
BindIP = "0.0.0.0"
```

### 10.3 Configuration de worldserver.conf

```bash
nano worldserver.conf
```

Modifiez les paramètres suivants :

```ini
# ─── Connexions aux bases de données ───
LoginDatabaseInfo = "127.0.0.1;3306;acore;VotreMotDePasseFort;acore_auth"
WorldDatabaseInfo = "127.0.0.1;3306;acore;VotreMotDePasseFort;acore_world"
CharacterDatabaseInfo = "127.0.0.1;3306;acore;VotreMotDePasseFort;acore_characters"

# ─── Paramètres du royaume ───
RealmID = 1
WorldServerPort = 8085
BindIP = "0.0.0.0"

# ─── Répertoire des données (DBC, Maps, VMaps, MMaps) ───
DataDir = "$HOME/azerothcore/env/dist/data/"

# ─── Logs ───
LogsDir = "$HOME/azerothcore/env/dist/logs/"

# ─── Sécurité ───
# Désactiver l'authentification par console (optionnel)
Console.Enable = 1
```

> 📝 **Structure de connexion** : `"IP;Port;Utilisateur;MotDePasse;BaseDeDonnées"`

### 10.4 Déclaration du royaume dans la base auth

Connectez-vous à MySQL et insérez le royaume :

```bash
sudo mysql -u root -p
```

```sql
USE acore_auth;

-- Vérifier les royaumes existants
SELECT * FROM realmlist;

-- Insérer le royaume (si ce n'est pas déjà fait)
INSERT INTO realmlist (id, name, address, localAddress, localSubnetMask, port, icon, flag, timezone, allowedSecurityLevel, population)
VALUES (1, 'CoA Server', '127.0.0.1', '127.0.0.1', '255.255.255.0', 8085, 0, 0, 1, 0, 0);

-- Vérifier
SELECT * FROM realmlist;
EXIT;
```

> 📝 **Note** : `RealmID` dans `worldserver.conf` doit correspondre à l'`id` dans la table `realmlist`.

### 10.5 Création du répertoire de données

```bash
# Créer le répertoire de données
mkdir -p ~/azerothcore/env/dist/data
mkdir -p ~/azerothcore/env/dist/logs

# Vérifier
ls -la ~/azerothcore/env/dist/
```

---

## 11. Extraction des données client (DBC, Maps, VMaps, MMaps)

### 11.1 Localisation des outils d'extraction

Les outils d'extraction sont dans le répertoire `bin/` de l'installation :

```bash
ls -la ~/azerothcore/env/dist/bin/
# mapextractor, vmap4extractor, vmap4assembler, mmaps_generator
```

### 11.2 Copie des outils vers le client WoW

Vous devez avoir un client WoW 3.3.5a (build 12340) sur une machine accessible. Copiez les outils dans le répertoire du client :

```bash
# Sur la VM, copier les outils vers un dossier partagé ou via SCP
# Exemple avec SCP (depuis une autre machine)
scp ~/azerothcore/env/dist/bin/mapextractor user@client-machine:/chemin/vers/wow/
scp ~/azerothcore/env/dist/bin/vmap4extractor user@client-machine:/chemin/vers/wow/
scp ~/azerothcore/env/dist/bin/vmap4assembler user@client-machine:/chemin/vers/wow/
scp ~/azerothcore/env/dist/bin/mmaps_generator user@client-machine:/chemin/vers/wow/
```

### 11.3 Extraction des DBC et Maps

Sur la machine cliente, dans le répertoire WoW :

```bash
# Extraire les DBC et Maps
./mapextractor

# Cela crée les dossiers dbc/ et maps/
```

### 11.4 Extraction des VMaps

```bash
# Extraire les VMaps
./vmap4extractor

# Assembler les VMaps
mkdir vmaps
./vmap4assembler Buildings vmaps
```

> ⏱️ Cette étape peut prendre **plusieurs heures** selon la machine.

### 11.5 Extraction des MMaps (recommandé)

```bash
# Extraire les MMaps
mkdir mmaps
./mmaps_generator
```

> ⏱️ Cette étape peut également prendre **plusieurs heures**.

### 11.6 Transfert des données vers la VM

```bash
# Transférer les dossiers extraits vers la VM
# Depuis la machine cliente :
scp -r dbc maps vmaps mmaps user@vm-ip:~/azerothcore/env/dist/data/

# Ou utiliser un dossier partagé VirtualBox
```

### 11.7 Vérification des données

```bash
# Sur la VM
ls -la ~/azerothcore/env/dist/data/
# Vous devriez voir : dbc/  maps/  vmaps/  mmaps/
```

---

## 12. Démarrage et test du serveur

### 12.1 Démarrage de authserver

Ouvrez un premier terminal :

```bash
cd ~/azerothcore/env/dist/bin/
./authserver
```

Vous devriez voir :
```
Starting AuthServer...
AuthServer listening on 0.0.0.0:3724
```

### 12.2 Démarrage de worldserver

Ouvrez un second terminal (ou utilisez `screen` / `tmux`) :

```bash
cd ~/azerothcore/env/dist/bin/
./worldserver
```

Vous devriez voir :
```
Starting WorldServer...
WorldServer listening on 0.0.0.0:8085
...
World initialized in X seconds
```

### 12.3 Utilisation de screen pour la persistance

Pour que le serveur continue de tourner après déconnexion SSH :

```bash
# Créer une session screen pour authserver
screen -S authserver
cd ~/azerothcore/env/dist/bin/
./authserver
# Détacher : Ctrl+A, puis D

# Créer une session screen pour worldserver
screen -S worldserver
cd ~/azerothcore/env/dist/bin/
./worldserver
# Détacher : Ctrl+A, puis D

# Pour lister les sessions
screen -ls

# Pour se rattacher
screen -r authserver
```

### 12.4 Test de connexion

1. Configurez votre client WoW 3.3.5a avec l'IP de la VM dans `realmlist.wtf` :
   ```
   set realmlist <IP_DE_LA_VM>
   ```
2. Lancez WoW et connectez-vous avec un compte existant (ou créez-en un)

### 12.5 Création d'un compte (optionnel)

Si vous n'avez pas de compte, vous pouvez en créer un via la console du worldserver :

```
account create <nom> <motdepasse>
account set gmlevel <nom> 3
```

---

## 13. Configuration réseau et accès externe

### 13.1 Configuration du mode réseau VirtualBox

| Mode | Utilisation | Configuration |
|------|-------------|---------------|
| **NAT** | Accès Internet sortant uniquement | Redirection de ports nécessaire |
| **Accès par pont** | La VM est sur le réseau local | IP directe, idéal pour un serveur |
| **Réseau privé hôte** | Communication hôte ↔ VM | Pas d'accès externe |

### 13.2 Redirection de ports (si NAT)

Dans VirtualBox : **Configuration → Réseau → Avancé → Redirection de ports**

| Nom | Protocole | IP hôte | Port hôte | IP invité | Port invité |
|-----|-----------|---------|-----------|-----------|-------------|
| Auth | TCP | 127.0.0.1 | 3724 | 10.0.2.15 | 3724 |
| World | TCP | 127.0.0.1 | 8085 | 10.0.2.15 | 8085 |

### 13.3 Configuration du pare-feu Ubuntu

```bash
# Ouvrir les ports nécessaires
sudo ufw allow 3724/tcp
sudo ufw allow 8085/tcp
sudo ufw allow 22/tcp

# Activer le pare-feu
sudo ufw enable

# Vérifier
sudo ufw status
```

### 13.4 Configuration de la realmlist pour accès externe

Si vous souhaitez que des joueurs externes se connectent, modifiez l'adresse dans la base `acore_auth` :

```sql
USE acore_auth;
UPDATE realmlist SET address = 'VOTRE_IP_PUBLIQUE' WHERE id = 1;
```

> ⚠️ **Important** : Vous devrez également configurer la redirection de ports sur votre routeur (ports 3724 et 8085 TCP).

---

## 14. Maintenance et mises à jour

### 14.1 Sauvegarde des bases de données

```bash
# Créer un script de sauvegarde
cat > ~/backup-db.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="$HOME/backups/$(date +%Y-%m-%d)"
mkdir -p $BACKUP_DIR

mysqldump -u acore -p'VotreMotDePasseFort' acore_auth > $BACKUP_DIR/acore_auth.sql
mysqldump -u acore -p'VotreMotDePasseFort' acore_world > $BACKUP_DIR/acore_world.sql
mysqldump -u acore -p'VotreMotDePasseFort' acore_characters > $BACKUP_DIR/acore_characters.sql

echo "Sauvegarde terminée dans $BACKUP_DIR"
EOF

chmod +x ~/backup-db.sh

# Ajouter au crontab pour une sauvegarde quotidienne à 3h du matin
(crontab -l 2>/dev/null; echo "0 3 * * * $HOME/backup-db.sh") | crontab -
```

### 14.2 Mise à jour du core

```bash
# Arrêter les serveurs (screen -r puis Ctrl+C)

# Mettre à jour le dépôt
cd ~/azerothcore
git pull

# Recompiler
cd build
cmake ../ -DCMAKE_INSTALL_PREFIX=$HOME/azerothcore/env/dist/ \
  -DCMAKE_C_COMPILER=/usr/bin/clang \
  -DCMAKE_CXX_COMPILER=/usr/bin/clang++ \
  -DTOOLS_BUILD=all -DSCRIPTS=static -DMODULES=static
make -j$(nproc)
make install

# Redémarrer les serveurs
```

### 14.3 Surveillance des logs

```bash
# Voir les logs en temps réel
tail -f ~/azerothcore/env/dist/logs/Server.log
tail -f ~/azerothcore/env/dist/logs/Errors.log
```

---

## 15. Dépannage et erreurs courantes

### 15.1 Erreur de connexion MySQL

**Symptôme** : `Can't connect to MySQL server on '127.0.0.1'`

**Solution** :
```bash
# Vérifier que MySQL tourne
sudo systemctl status mysql

# Redémarrer si nécessaire
sudo systemctl restart mysql
```

### 15.2 Erreur "Table doesn't exist"

**Symptôme** : `Table 'acore_world.xxx' doesn't exist`

**Solution** : Vérifiez que vous avez bien importé le package world CoA avant le premier démarrage.

### 15.3 Erreur de compilation "cannot find -lstdc++"

**Solution** (Ubuntu 24.04) :
```bash
sudo apt install libstdc++-13-dev
```

### 15.4 Le worldserver crash au démarrage

**Vérifications** :
1. Les bases de données sont-elles correctement configurées ?
2. Le fichier `DataDir` pointe-t-il vers le bon répertoire ?
3. Les dossiers `dbc`, `maps`, `vmaps`, `mmaps` sont-ils présents ?

### 15.5 Port déjà utilisé

```bash
# Vérifier quel processus utilise le port
sudo lsof -i :8085
sudo lsof -i :3724

# Tuer le processus si nécessaire
sudo kill -9 <PID>
```

### 15.6 Erreur "Realm ID not defined"

**Solution** : Vérifiez que `RealmID` dans `worldserver.conf` correspond à l'`id` dans la table `realmlist`.

---

## 📎 Annexes

### A. Commandes utiles

```bash
# Voir l'utilisation CPU/RAM
htop

# Voir l'espace disque
df -h

# Voir les connexions réseau
ss -tulpn

# Voir les processus AzerothCore
ps aux | grep -E "authserver|worldserver"
```

### B. Fichiers de configuration importants

| Fichier | Emplacement | Rôle |
|---------|-------------|------|
| `authserver.conf` | `~/azerothcore/env/dist/etc/` | Config authserver |
| `worldserver.conf` | `~/azerothcore/env/dist/etc/` | Config worldserver |
| `realmlist` (table) | `acore_auth` | Liste des royaumes |
| `DataDir` | Variable dans `worldserver.conf` | Chemin des données client |

### C. Ressources

- [Documentation officielle AzerothCore](https://www.azerothcore.org/wiki/)
- [Dépôt GitHub du fork CoA](https://github.com/jealous-sound/azerothcore-wotlk-coa)
- [Discord AzerothCore](https://discord.gg/gkt4y2x)
- [Forum AzerothCore](https://forum.azerothcore.org/)

### D. Licence

Le code source AzerothCore est publié sous **GNU GPL v2**. AzerothCore n'est pas un produit officiel de Blizzard Entertainment et n'est pas affilié à World of Warcraft.

---

## ✅ Résumé des étapes

1. **Créer la VM** VirtualBox (4 CPU, 4-8 Go RAM, 80 Go disque)
2. **Installer Ubuntu Server** 22.04/24.04 LTS
3. **Installer les Guest Additions** et désactiver les mises à jour auto
4. **Installer les dépendances** (git, cmake, clang, boost, etc.)
5. **Installer et sécuriser MySQL** 8.4 LTS
6. **Créer les bases** `acore_auth`, `acore_world`, `acore_characters`
7. **Cloner le fork CoA** et compiler le core
8. **Importer les bases de données** (package world CoA inclus)
9. **Configurer** `authserver.conf` et `worldserver.conf`
10. **Extraire les données client** (DBC, Maps, VMaps, MMaps)
11. **Démarrer** authserver et worldserver
12. **Configurer le réseau** et tester la connexion

---

Ce guide vous permet d'installer un serveur dédié AzerothCore complet et fonctionnel sur une VM VirtualBox. Pour toute question ou problème, n'hésitez pas à consulter la documentation officielle ou la communauté AzerothCore.

# Audit complet
Pour réaliser un audit complet de votre VM Debian contenant AzerothCore, vous devez collecter systématiquement toutes les informations relatives à l’environnement, à l’installation du serveur, à la configuration des bases de données, aux services et aux éventuelles personnalisations. Voici une méthodologie détaillée, accompagnée des commandes à exécuter, pour produire un fichier de documentation du projet.

---

## 1. Informations générales sur le système

Documentez l’OS, le matériel virtuel alloué, les utilisateurs et l’état du système.

```bash
# Version de Debian et noyau
cat /etc/os-release
uname -a

# Ressources matérielles
free -h
df -h
lscpu
lsblk

# Adresses IP et interfaces réseau
ip a

# Utilisateurs présents
cat /etc/passwd
```

**À noter dans la documentation :**  
- Nom et version de l’OS, version du noyau.  
- Quantité de RAM, nombre de CPU, espace disque utilisé/disponible.  
- Configuration réseau (IP statique, passerelle, DNS).  
- Liste des utilisateurs système et leurs rôles.

---

## 2. Audit de l’installation d’AzerothCore

### 2.1. Emplacement du code source et version

```bash
# Localiser le dossier d'installation (souvent /home/azerothcore ou /opt/azerothcore)
find / -type d -name "azerothcore*" 2>/dev/null

# Si un dépôt Git est présent, obtenir la version exacte
cd /chemin/vers/azerothcore
git log -1 --oneline
git branch -v
git remote -v
```

**À documenter :**  
- Chemin du répertoire source.  
- Commit, branche et URL du dépôt Git.  
- Date du dernier pull / build.

### 2.2. Configuration du build (CMake)

Si le projet a été compilé, les options CMake sont importantes.

```bash
# Regarder dans le répertoire build s'il existe (souvent build/)
cd /chemin/vers/azerothcore/build
cat CMakeCache.txt | grep -E "CMAKE_INSTALL_PREFIX|TOOLS|SCRIPTS|MODULES|WITH_WARNINGS|BUILD_TESTING"
```

**À documenter :**  
- Options de compilation utilisées (ex : `-DTOOLS=1 -DSCRIPTS=static`).  
- Répertoire d’installation des binaires (`CMAKE_INSTALL_PREFIX`).  
- Modules supplémentaires activés.

### 2.3. Binaires et exécutables

```bash
# Localiser les exécutables worldserver et authserver
which worldserver authserver 2>/dev/null
find / -name "worldserver" -type f 2>/dev/null
find / -name "authserver" -type f 2>/dev/null

# Vérifier les permissions et l'utilisateur propriétaire
ls -l /chemin/vers/worldserver /chemin/vers/authserver
```

**À documenter :**  
- Emplacements exacts des binaires.  
- Utilisateur sous lequel ils s’exécutent.

---

## 3. Audit des bases de données

AzerothCore utilise MySQL ou MariaDB. Il faut inventorier les bases et leur configuration.

### 3.1. Version du SGBD

```bash
mysql --version
# ou
mariadb --version
```

### 3.2. Connexion et liste des bases

```bash
# Se connecter en root (ou utiliser l'utilisateur MySQL de l'application)
mysql -u root -p
```

Dans l’invite MySQL :

```sql
SHOW DATABASES;
SELECT user, host FROM mysql.user;
SHOW GRANTS FOR 'acore'@'localhost';  -- remplacer par le user utilisé
EXIT;
```

**À documenter :**  
- Version exacte du SGBD.  
- Noms des bases (généralement `acore_auth`, `acore_characters`, `acore_world`).  
- Utilisateurs MySQL et leurs hôtes autorisés (sans les mots de passe, mais indiquer s’ils sont définis).  
- Privilèges accordés.

### 3.3. Taille des bases

```sql
SELECT table_schema AS "Database", 
ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS "Size (MB)" 
FROM information_schema.tables 
GROUP BY table_schema;
```

---

## 4. Fichiers de configuration du serveur

Les fichiers principaux sont `authserver.conf` et `worldserver.conf`. Ils contiennent tous les réglages.

```bash
# Localiser les .conf
find / -name "*.conf" | grep -E "authserver|worldserver"

# Afficher le contenu (en masquant les mots de passe si nécessaire)
cat /chemin/vers/etc/authserver.conf
cat /chemin/vers/etc/worldserver.conf
```

**À documenter :**  
- Chemin exact des fichiers.  
- Paramètres importants :  
  - Ports d’écoute (auth : 3724, world : 8085 par défaut).  
  - Base de données utilisée (nom, hôte, utilisateur).  
  - Taux d’expérience, de drop, etc.  
  - Paramètres de logs.  
  - Activation de modules ou scripts personnalisés.  
- Toute modification par rapport aux valeurs par défaut doit être notée.

Si les fichiers contiennent des mots de passe, remplacez-les par `***` dans la documentation.

---

## 5. Services système et automatisation

Comment les serveurs sont-ils lancés ? Via systemd, des scripts, ou manuellement ?

```bash
# Lister les services liés à AzerothCore
systemctl list-units --type=service | grep -i azeroth
systemctl status <nom_du_service>

# Rechercher des scripts de démarrage
ls -l /etc/init.d/
ls -l /etc/systemd/system/ | grep -i world
cat /etc/systemd/system/worldserver.service 2>/dev/null
```

**À documenter :**  
- Nom des services systemd (ou scripts init) qui gèrent `authserver` et `worldserver`.  
- Contenu des fichiers unités (sans les secrets).  
- Utilisateur sous lequel le service tourne (`User=`).  
- Options de redémarrage automatique éventuelles.

---

## 6. Logs et surveillance

```bash
# Localiser les fichiers de logs
find / -name "*.log" | grep -E "world|auth|azeroth"

# Vérifier la rotation des logs
ls -l /etc/logrotate.d/
cat /etc/logrotate.d/azerothcore 2>/dev/null
```

**À documenter :**  
- Emplacement des logs et leur niveau de verbosité.  
- Politique de rotation.  
- Outils de surveillance éventuels (cron, monit, etc.).

---

## 7. Sécurité et réseau

### 7.1. Pare-feu

```bash
# Si iptables ou nftables est utilisé
iptables -L -n -v
nft list ruleset

# Si ufw est actif
ufw status verbose
```

### 7.2. Ports ouverts

```bash
ss -tulpn | grep -E "3724|8085|3306"
```

### 7.3. Permissions des fichiers

Vérifiez que les fichiers de configuration et les binaires ne sont accessibles que par les utilisateurs appropriés.

```bash
ls -l /chemin/vers/etc/
ls -l /chemin/vers/bin/
```

**À documenter :**  
- Règles de pare-feu actives (ports autorisés).  
- Ports en écoute et processus associés.  
- Propriétaires et permissions des fichiers critiques.

---

## 8. Personnalisations et modules

Si vous avez ajouté des modules Lua, C++ ou des scripts SQL personnalisés, listez-les.

```bash
# Modules Lua (souvent dans lua_scripts/)
find /chemin/vers/lua_scripts -type f -name "*.lua"

# Scripts SQL personnalisés
# Recherchez dans le répertoire sql/ du projet ou dans un dossier dédié
find /chemin/vers/sql -type f -name "*.sql" | grep -v "base"
```

**À documenter :**  
- Liste des modules supplémentaires avec leur version et auteur.  
- Toute modification du code source (patchs, commits personnels).  
- Scripts SQL exécutés en plus des fichiers officiels.

---

## 9. Configuration de la VM VirtualBox

Enfin, documentez les paramètres de la machine virtuelle elle-même.

Depuis l’hôte VirtualBox :

```bash
VBoxManage showvminfo "Nom_de_la_VM"
```

**À noter :**  
- Mémoire RAM allouée.  
- Nombre de CPU virtuels.  
- Taille et type des disques virtuels.  
- Cartes réseau (mode NAT, pont, etc.).  
- Dossiers partagés éventuels.

---

## 10. Génération automatique d’un rapport

Pour simplifier, vous pouvez créer un script qui rassemble toutes ces informations dans un fichier texte ou markdown.

```bash
#!/bin/bash
OUTPUT="audit_azerothcore_$(date +%Y%m%d).md"
{
    echo "# Rapport d'audit AzerothCore - $(date)"
    echo "## Système"
    uname -a
    cat /etc/os-release
    echo "## Ressources"
    free -h
    df -h
    echo "## Réseau"
    ip a
    echo "## Processus"
    ps aux | grep -E "worldserver|authserver"
    echo "## Bases de données"
    mysql --version
    mysql -u root -e "SHOW DATABASES;"
    # ... ajoutez d'autres commandes
} > "$OUTPUT"
```

Exécutez ce script et conservez le fichier généré dans un endroit sûr (par exemple, un dépôt Git de documentation).

---

# **Prompt idéal pour générer une documentation complète sur la gestion de serveur AzerothCore**

---

Rédige une documentation exhaustive et structurée sur la gestion d’un serveur AzerothCore (version 3.3.5a / Wrath of the Lich King) installé sur une machine virtuelle Debian 12. Cette documentation doit servir de référence unique pour un administrateur de serveur privé, couvrant l’installation, la configuration, l’administration quotidienne, la maintenance, la personnalisation et le dépannage.

**Contexte technique**  
- Serveur AzerothCore compilé depuis les sources officielles, installé dans `/azerothcore` sur Debian 12.  
- Base de données MySQL/MariaDB (bases `acore_auth`, `acore_characters`, `acore_world`).  
- Services `authserver` et `worldserver` gérés par systemd.  
- Remote Access (RA) activé sur le port 3443 avec mot de passe.  
- Accès SSH à la VM, avec éventuellement une interface web d’administration à mettre en place.

**Objectifs de la documentation**  
1. **Installation et mise à jour** : procédure pas à pas pour compiler AzerothCore depuis zéro, configurer les bases de données, extraire les données client (maps, dbc, vmaps, mmaps), et mettre à jour le serveur (pull git, recompilation, migration SQL).  
2. **Configuration complète** : explication de tous les paramètres importants des fichiers `worldserver.conf` et `authserver.conf`, avec recommandations pour un serveur de jeu privé (taux d’XP, drop, annonces, sécurité, etc.).  
3. **Administration via commandes GM** : liste des commandes essentielles (téléportation, apparition d’objets/PNJ, modification de personnages, gestion des quêtes, annonces, etc.), avec exemples et cas d’usage.  
4. **Gestion de la base de données** : explication des tables principales (`creature_template`, `gameobject_template`, `item_template`, `quest_template`, `smart_scripts`, etc.), et méthodes pour insérer/modifier du contenu (SQL direct, outils comme Keira3, scripts personnalisés).  
5. **Personnalisation du gameplay** : introduction aux modules C++ (création, compilation, activation), aux scripts Lua (Eluna), et aux modifications SQL pour ajouter des fonctionnalités (téléporteurs, vendeurs, boss customs, événements).  
6. **Sécurité** : bonnes pratiques pour sécuriser le serveur (accès RA, pare-feu, permissions fichiers, sauvegardes, protection contre les failles courantes).  
7. **Maintenance et sauvegardes** : planification des sauvegardes de base de données, rotation des logs, surveillance des ressources, redémarrage propre.  
8. **Dépannage** : erreurs fréquentes (connexion, crashs, problèmes de extraction de données, conflits de ports) et solutions.  
9. **Annexes** : chemins utiles, commandes récapitulatives, liens vers la documentation officielle AzerothCore.

**Format demandé**  
- Document Markdown bien structuré avec titres, sous-titres, tableaux, blocs de code (bash, SQL, C++).  
- Chaque section doit être autonome et comporter des exemples concrets.  
- Inclure des captures d’écran ou schémas textuels si pertinents.  
- Rédiger en français, avec un ton professionnel et pédagogique.

**Public cible**  
Administrateur intermédiaire, connaissant Linux et les bases de WoW, mais pas forcément expert en AzerothCore.

**Contraintes**  
- Se baser uniquement sur la version 3.3.5a d’AzerothCore, sans mentionner d’autres expansions.  
- Les commandes et chemins doivent être adaptés à Debian 12 et à une installation dans `/opt/azerothcore`.  
- Pour les modules C++, expliquer brièvement le processus de compilation, en supposant que le lecteur a déjà les outils de build installés.  
- La documentation doit être prête à être sauvegardée dans un fichier `README.md` ou `GESTION_SERVEUR.md` et utilisée comme manuel de référence.

**Livrable attendu**  
Un document complet d’environ 5000 à 8000 mots, couvrant tous les points ci-dessus de manière claire et détaillée.

# GM COMMANDE
### USAGE: reload
Possible subcommands:
- reload achievement criteria data
- reload achievement reward
- reload achievement_reward_locale
- reload acore_string
- reload all
- reload areatrigger
- reload areatrigger_involvedrelation
- reload areatrigger_tavern
- reload areatrigger_teleport
- reload auctions
- reload autobroadcast
- reload battleground_template
- reload broadcast_text
- reload command
- reload conditions
- reload config
- reload creature_linked_respawn
- reload creature_loot_template
- reload creature_movement_override
- reload creature_onkill_reputation
- reload creature_questender
- reload creature_queststarter
- reload creature_template
- reload creature_template_locale
- reload creature_text
- reload creature text locale
- reload disables
reload disenchant_loot_template
- reload dungeon_access_requirement
- reload dungeon_access_template
reload event_scripts
reload fishing_loot_template
reload game_graveyard
- reload game_tele
- reload gameobject_loot_template
- reload gameobject_questender
- reload gameobject_queststarter
reload gameobject_template_locale
- reload gm_tickets
reload gossip_menu
- reload gossip_menu_option
- reload gossip_menu_option_locale
reload graveyard zone
- reload item_enchantment_template
reload item_loot_template
reload item_set_name_locale
reload item_set_names
- reload item_template_locale
- reload Ifg_dungeon_rewards
- reload mail_level_reward
reload mail_loot_template
reload mail_server_template
- reload milling loot_template
- reload module_string
- reload motd
- reload npc_spellclick_spells
- reload npc_text_locale
- reload npc_trainer
- reload npc_vendor
- reload page_text
- reload page_text_locale
- reload pickpocketing_loot_template
- reload player_loot_template
- reload points_of_interest
- reload points_of interest locale
- reload profanity_name
- reload prospecting_loot_template
- reload quest_greeting
- reload quest_offer_reward_locale
- reload quest_poi
- reload quest_request_item_locale
- reload quest_template
- reload quest_template_locale
- reload reference_loot_template
- reload reputation_reward_rate
- reload reputation_spillover_template
- reload reserved_name
- reload skill_discovery_template
- reload skill_extra item template

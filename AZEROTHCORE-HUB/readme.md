
---

## 📋 Vue d'ensemble du plan

On va suivre cette feuille de route :

1. **Préparation de l'environnement** (Python, venv, PostgreSQL, Git)
2. **Création du projet Django** et de son architecture
3. **Configuration multi-bases de données** (world, auth, characters, dbc)
4. **Modélisation des premières entités** (items, créatures, sorts)
5. **Import des données** depuis AzerothCore
6. **Parsing des fichiers DBC** (noms localisés, icônes)
7. **Interface d'admin Django** pour naviguer dans les données
8. **API REST** avec Django REST Framework
9. **Templates et tooltips** frontend
10. **Recherche globale** (Meilisearch ou PostgreSQL FTS)
11. **Déploiement** (Docker, Nginx, Cloudflare)

---

## 🎯 Étape 1 — Préparation de l'environnement

### Le quoi
Installer tous les outils nécessaires sur ta machine pour développer le projet.

### Le pourquoi
Un environnement propre et isolé évite les conflits entre projets, facilite la reproduction sur d'autres machines, et te permet de monter en compétence sur les outils standards de l'industrie.

### Le comment

**1.1 — Vérifier Python**

On a besoin de Python 3.11 ou supérieur (Django 5.x exige 3.10+, mais 3.11+ est plus confortable).

```bash
python3 --version
```

Si tu as une version inférieure, installe la bonne via `pyenv` (recommandé) ou le gestionnaire de paquets de ton OS.

**1.2 — Créer le dossier du projet**

```bash
mkdir wotlkdb-django
cd wotlkdb-django
```

**1.3 — Créer un environnement virtuel**

```bash
python3 -m venv .venv
source .venv/bin/activate   # Linux/macOS
# .venv\Scripts\activate    # Windows
```

> 💡 Le venv isole les dépendances du projet. Sans lui, tu pollues ton Python système et tu risques des conflits.

**1.4 — Installer PostgreSQL**

On choisit PostgreSQL plutôt que MySQL pour plusieurs raisons :
- Meilleure recherche plein texte native (`tsvector`, `tsquery`)
- Meilleur support JSON (utile pour les données DBC flexibles)
- Support solide dans Django

```bash
# Ubuntu/Debian
sudo apt install postgresql postgresql-contrib

# macOS
brew install postgresql@16
```

Crée une base de données pour le projet :

```bash
sudo -u postgres psql
CREATE DATABASE wotlkdb OWNER wotlk_user;
CREATE USER wotlk_user WITH PASSWORD 'motdepasse';
GRANT ALL PRIVILEGES ON DATABASE wotlkdb TO wotlk_user;
\q
```

**1.5 — Initialiser Git**

```bash
git init
```

Crée un `.gitignore` :

```gitignore
.venv/
__pycache__/
*.pyc
.env
db.sqlite3
/media/
/staticfiles/
*.log
```

**1.6 — Créer un `requirements.txt` initial**

```
Django>=5.0,<5.1
psycopg[binary]>=3.1
python-dotenv>=1.0
djangorestframework>=3.15
django-environ>=0.11
```

Installe :

```bash
pip install -r requirements.txt
```

---

## 🎯 Étape 2 — Création du projet Django

### Le quoi
Générer la structure de base Django et organiser le code en applications distinctes.

### Le pourquoi
Django fonctionne avec un système de **projet** (contenant la config globale) et d'**applications** (modules fonctionnels). Cette séparation permet de garder un code propre et modulaire. On ne veut pas tout mettre dans une seule appli — on sépare par domaine métier.

### Le comment

**2.1 — Créer le projet**

```bash
django-admin startproject wotlkdb .
```

> 💡 Le `.` à la fin évite de créer un dossier `wotlkdb/wotlkdb/`. On reste à la racine.

Tu obtiens :

```
wotlkdb-django/
├── manage.py
├── wotlkdb/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
└── requirements.txt
```

**2.2 — Créer les applications**

On va découper le projet en apps logiques :

| App | Rôle |
|-----|------|
| `core` | Pages statiques, accueil, recherche globale |
| `items` | Objets (`item_template`) |
| `creatures` | PNJs (`creature_template`) |
| `spells` | Sorts (`spell_template`) |
| `quests` | Quêtes (`quest_template`) |
| `dbc` | Données DBC (noms localisés, icônes) |
| `accounts` | Utilisateurs, profils, favoris |
| `api` | API REST centralisée |

```bash
python manage.py startapp core
python manage.py startapp items
python manage.py startapp creatures
python manage.py startapp spells
python manage.py startapp quests
python manage.py startapp dbc
python manage.py startapp accounts
python manage.py startapp api
```

**2.3 — Organiser les settings**

Crée un dossier `wotlkdb/settings/` avec :

```
wotlkdb/settings/
├── __init__.py
├── base.py       # settings communs
├── dev.py        # développement
└── prod.py       # production
```

Dans `base.py`, déplace le contenu de `settings.py` et ajoute :

```python
from pathlib import Path
import environ

BASE_DIR = Path(__file__).resolve().parent.parent.parent

env = environ.Env()
environ.Env.read_env(BASE_DIR / ".env")

SECRET_KEY = env("DJANGO_SECRET_KEY")
DEBUG = env.bool("DJANGO_DEBUG", default=False)
ALLOWED_HOSTS = env.list("DJANGO_ALLOWED_HOSTS", default=[])
```

Dans `dev.py` :

```python
from .base import *

DEBUG = True
ALLOWED_HOSTS = ["*"]
```

Modifie `manage.py` et `wsgi.py` pour pointer sur `wotlkdb.settings.dev` par défaut.

**2.4 — Créer le fichier `.env`**

```env
DJANGO_SECRET_KEY=change-moi-en-production
DJANGO_DEBUG=True
DJANGO_ALLOWED_HOSTS=localhost,127.0.0.1
```

> 💡 Le `.env` ne doit **jamais** être commité. C'est pour ça qu'il est dans `.gitignore`.

**2.5 — Test de démarrage**

```bash
python manage.py migrate
python manage.py runserver
```

Tu dois voir la page de bienvenue Django. ✅

---

## 🎯 Étape 3 — Configuration multi-bases de données

### Le quoi
Configurer Django pour se connecter à **plusieurs bases** : celle de Django (utilisateurs, favoris) et celles du serveur WoW (world, auth, characters).

### Le pourquoi
Dans l'écosystème WoW privé, il y a historiquement **trois bases séparées** :
- `auth` : comptes, sessions, bans
- `characters` : personnages, inventaires, guildes
- `world` : tout le contenu statique (objets, PNJs, quêtes, sorts)

Pour wotlkdb.com, on a surtout besoin de `world` (données de contenu) et éventuellement `dbc` (données client parsées). On garde les autres pour de futures fonctionnalités (armurerie, etc.).

### Le comment

**3.1 — Créer un router de base de données**

Django permet de router chaque modèle vers une base spécifique. Crée `wotlkdb/db_routers.py` :

```python
class WoWRouter:
    """
    Route les modèles des apps WoW vers leur base respective.
    L'app 'default' reste sur la base Django classique.
    """
    wow_apps = {"items", "creatures", "spells", "quests", "dbc"}
    django_apps = {"core", "accounts", "api", "admin", "auth", "contenttypes", "sessions"}

    def db_for_read(self, model, **hints):
        if model._meta.app_label in self.wow_apps:
            return "world"
        if model._meta.app_label in self.django_apps:
            return "default"
        return None

    def db_for_write(self, model, **hints):
        # On interdit l'écriture sur la base world depuis Django
        if model._meta.app_label in self.wow_apps:
            return None  # lecture seule
        if model._meta.app_label in self.django_apps:
            return "default"
        return None

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        if app_label in self.wow_apps:
            return False  # Django ne doit PAS migrer ces tables
        if app_label in self.django_apps:
            return db == "default"
        return None
```

> 💡 **Point crucial** : `allow_migrate` renvoie `False` pour les apps WoW. On ne veut **jamais** que Django crée ou modifie les tables `item_template`, `creature_template`, etc. Elles existent déjà dans AzerothCore. Django doit seulement les **lire**.

**3.2 — Déclarer les bases dans `base.py`**

```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": env("DB_NAME"),
        "USER": env("DB_USER"),
        "PASSWORD": env("DB_PASSWORD"),
        "HOST": env("DB_HOST", default="localhost"),
        "PORT": env("DB_PORT", default="5432"),
    },
    "world": {
        "ENGINE": "django.db.backends.mysql",
        "NAME": env("WOW_WORLD_DB", default="acore_world"),
        "USER": env("WOW_DB_USER", default="acore"),
        "PASSWORD": env("WOW_DB_PASSWORD", default="acore"),
        "HOST": env("WOW_DB_HOST", default="127.0.0.1"),
        "PORT": env("WOW_DB_PORT", default="3306"),
        "OPTIONS": {
            "charset": "utf8mb4",
            "init_command": "SET sql_mode='STRICT_TRANS_TABLES'",
        },
    },
}

DATABASE_ROUTERS = ["wotlkdb.db_routers.WoWRouter"]
```

> 💡 On utilise MySQL pour `world` car AzerothCore l'utilise. On ne peut pas convertir sans casser la compatibilité avec le serveur.

**3.3 — Ajouter `pymysql` ou `mysqlclient`**

```bash
pip install mysqlclient
```

Ajoute-le à `requirements.txt`.

**3.4 — Mettre à jour `.env`**

```env
DB_NAME=wotlkdb
DB_USER=wotlk_user
DB_PASSWORD=motdepasse
DB_HOST=localhost
DB_PORT=5432

WOW_WORLD_DB=acore_world
WOW_DB_USER=acore
WOW_DB_PASSWORD=acore
WOW_DB_HOST=127.0.0.1
WOW_DB_PORT=3306
```

**3.5 — Vérifier la connexion**

```bash
python manage.py shell
```

```python
from django.db import connections
connections["world"].cursor()  # doit fonctionner si AzerothCore est installé
```

Si tu n'as pas encore AzerothCore installé, on verra ça à l'étape 5. Pour l'instant, on continue la structure.

---

## 🎯 Étape 4 — Modélisation des premières entités

### Le quoi
Créer les modèles Django qui reflètent les tables WoW, en **lecture seule**.

### Le pourquoi
Django a besoin de modèles Python pour interroger la base. Puisque les tables existent déjà (créées par AzerothCore), on utilise `managed = False` et on mappe `db_table`. On obtient ainsi un ORM complet sur des tables qu'on ne contrôle pas.

### Le comment

**4.1 — Modèle `Item` (app `items`)**

Dans `items/models.py` :

```python
from django.db import models


class ItemTemplate(models.Model):
    entry = models.AutoField(primary_key=True)
    class_ = models.IntegerField(db_column="class")
    subclass = models.IntegerField()
    name = models.CharField(max_length=255)
    displayid = models.IntegerField()
    Quality = models.IntegerField(db_column="Quality")
    Flags = models.IntegerField(db_column="Flags")
    ItemLevel = models.IntegerField(db_column="ItemLevel")
    RequiredLevel = models.IntegerField(db_column="RequiredLevel")
    inventory_type = models.IntegerField(db_column="InventoryType")
    stat_type1 = models.IntegerField(null=True)
    stat_value1 = models.IntegerField(null=True)
    # ... ajoute les colonnes dont tu as besoin

    class Meta:
        managed = False
        db_table = "item_template"
        ordering = ["entry"]

    def __str__(self):
        return f"[{self.entry}] {self.name}"
```

> 💡 **Pourquoi `managed = False` ?** Cela dit à Django : "ne crée pas, ne modifie pas cette table. Contente-toi de la lire." Sans ça, Django tenterait de générer des migrations destructrices.
>
> 💡 **Pourquoi `db_column` parfois ?** AzerothCore a des noms de colonnes parfois en camelCase (`ItemLevel`, `Quality`). Django les normalise en snake_case par défaut. On mappe explicitement.

**4.2 — Modèle `Creature` (app `creatures`)**

```python
class CreatureTemplate(models.Model):
    entry = models.AutoField(primary_key=True)
    name = models.CharField(max_length=255)
    subname = models.CharField(max_length=255, null=True)
    minlevel = models.IntegerField()
    maxlevel = models.IntegerField()
    faction = models.IntegerField()
    npcflag = models.BigIntegerField()
    speed_walk = models.FloatField()
    speed_run = models.FloatField()
    # ...
    class_: int

    class Meta:
        managed = False
        db_table = "creature_template"
```

**4.3 — Modèle `Spell` (app `spells`)**

```python
class SpellTemplate(models.Model):
    id = models.AutoField(primary_key=True, db_column="ID")
    name = models.CharField(max_length=255)
    description = models.TextField(null=True)
    tooltip = models.TextField(null=True)
    school = models.IntegerField()
    # ...

    class Meta:
        managed = False
        db_table = "spell_template"
```

> ⚠️ Le schéma exact dépend de ta version d'AzerothCore. Utilise `python manage.py inspectdb --database=world item_template` pour générer automatiquement un modèle à partir de la table réelle !

**4.4 — Astuce puissante : `inspectdb`**

Django peut générer les modèles automatiquement :

```bash
python manage.py inspectdb --database=world item_template > items/models.py
```

Puis tu nettoies :
- Ajoute `managed = False` (inspectdb le fait déjà normalement)
- Renomme les classes pour qu'elles soient plus lisibles
- Vérifie les `db_column` pour les colonnes camelCase

C'est **la méthode recommandée** pour ne pas te tromper sur les noms.

---

## 🎯 Étape 5 — Installer AzerothCore et importer les données

### Le quoi
Récupérer une base `acore_world` peuplée.

### Le pourquoi
Sans données, ton site est vide. AzerothCore fournit un dump complet de la base 3.3.5a, exactement comme wotlkdb.com.

### Le comment

**5.1 — Récupérer les dumps**

Le plus simple est d'utiliser les **releases officielles d'AzerothCore** :

```bash
git clone https://github.com/azerothcore/azerothcore-wotlk.git
cd azerothcore-wotlk
```

Les dumps SQL sont dans `data/sql/`. Mais le plus pratique reste les **DB releases précompilées** :

```bash
wget https://github.com/azerothcore/azerothcore-wotlk/releases/download/.../acore_world.sql
```

Ou via le dépôt **`azerothcore/database`** qui publie régulièrement des archives.

**5.2 — Importer dans MySQL**

```bash
mysql -u acore -p -e "CREATE DATABASE acore_world CHARACTER SET utf8mb4"
mysql -u acore -p acore_world < acore_world.sql
```

**5.3 — Vérifier**

```bash
mysql -u acore -p acore_world -e "SELECT COUNT(*) FROM item_template;"
```

Tu devrais avoir ~50 000 lignes. ✅

**5.4 — Tester depuis Django**

```bash
python manage.py shell
```

```python
from items.models import ItemTemplate
ItemTemplate.objects.count()
ItemTemplate.objects.filter(name__icontains="épée").first()
```

---

## 🎯 Étape 6 — Parsing des fichiers DBC

### Le quoi
Lire les fichiers DBC du client WoW pour obtenir les **noms localisés**, les **icônes**, et d'autres données absentes de la base world.

### Le pourquoi
Exemple concret : dans `item_template`, la colonne `name` n'existe pas toujours — c'est souvent `name` en anglais, mais les noms français/allemands sont dans `Item.dbc`. De même, les icônes (`displayid`) pointent vers `ItemDisplayInfo.dbc`.

### Le comment

**6.1 — Installer `dbcraft`**

```bash
pip install dbcraft
```

**6.2 — Créer un script d'import**

Dans `dbc/management/commands/import_dbc.py` :

```python
from django.core.management.base import BaseCommand
from dbcraft import DBCFile
from pathlib import Path
import struct


class Command(BaseCommand):
    help = "Importe un fichier DBC dans la base Django"

    def add_arguments(self, parser):
        parser.add_argument("dbc_path", type=str)
        parser.add_argument("table_name", type=str)

    def handle(self, *args, **options):
        path = Path(options["dbc_path"])
        self.stdout.write(f"Lecture de {path}...")

        dbc = DBCFile(str(path))
        self.stdout.write(f"Records: {dbc.get_record_count()}")

        for i in range(dbc.get_record_count()):
            record = dbc.get_record(i)
            # À adapter selon le fichier
            # print(record.get_uint(0), record.get_string(1))
```

> 💡 `dbcraft` est encore jeune. Si tu rencontres des limitations, `pywowlib` est plus mature mais plus complexe. Une alternative pragmatique : parser les DBC en te basant sur les définitions de `WoWDBDefs`.

**6.3 — Stratégie recommandée**

Plutôt que parser les DBC à chaque démarrage, on les **importe une fois** dans une table PostgreSQL dédiée :

```
dbc_item_name           # entry -> nom localisé
dbc_item_display_info   # displayid -> icône, modèle
dbc_spell_name          # id -> nom
dbc_map                 # map_id -> nom de zone
```

Puis on joint via l'ORM Django en utilisant `db_table` + `db_constraint`.

**6.4 — Alternative plus simple**

Certaines communautés publient déjà des **tables SQL converties** depuis les DBC. Cherche `wotlk dbc sql dump` sur GitHub. C'est souvent plus rapide que de parser toi-même.

---

## 🎯 Étape 7 — Interface d'admin Django

### Le quoi
Utiliser l'admin Django pour naviguer, rechercher et inspecter les données.

### Le pourquoi
L'admin Django est un outil gratuit, puissant, et sécurisé. En quelques lignes, tu obtiens une interface CRUD complète. Parfait pour explorer ta base avant même d'écrire un frontend.

### Le comment

**7.1 — Enregistrer les modèles**

Dans `items/admin.py` :

```python
from django.contrib import admin
from .models import ItemTemplate


@admin.register(ItemTemplate)
class ItemTemplateAdmin(admin.ModelAdmin):
    list_display = ("entry", "name", "Quality", "ItemLevel", "RequiredLevel")
    search_fields = ("name", "entry")
    list_filter = ("Quality", "class_")
    readonly_fields = [f.name for f in ItemTemplate._meta.fields]  # tout en lecture seule
```

**7.2 — Créer un superuser**

```bash
python manage.py createsuperuser
```

**7.3 — Tester**

Va sur `http://localhost:8000/admin/` et navigue dans tes objets. ✅

---

## 🎯 Étape 8 — API REST

### Le quoi
Exposer les données via une API JSON pour le frontend.

### Le pourquoi
Séparer le backend (données) du frontend (présentation) te permet d'évoluer vers un frontend moderne (React, Vue) plus tard, et facilite le développement des tooltips en AJAX.

### Le comment

**8.1 — Installer DRF**

Déjà dans `requirements.txt`. Ajoute `rest_framework` à `INSTALLED_APPS`.

**8.2 — Sérialiseur d'item**

Dans `api/serializers.py` :

```python
from rest_framework import serializers
from items.models import ItemTemplate


class ItemSerializer(serializers.ModelSerializer):
    quality_name = serializers.SerializerMethodField()

    class Meta:
        model = ItemTemplate
        fields = ["entry", "name", "Quality", "ItemLevel", "RequiredLevel", "quality_name"]

    def get_quality_name(self, obj):
        qualities = {0: "Poor", 1: "Common", 2: "Uncommon", 3: "Rare", 4: "Epic", 5: "Legendary"}
        return qualities.get(obj.Quality, "Unknown")
```

**8.3 — Vue**

```python
from rest_framework import viewsets
from items.models import ItemTemplate
from .serializers import ItemSerializer


class ItemViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = ItemTemplate.objects.all()
    serializer_class = ItemSerializer
    lookup_field = "entry"
```

**8.4 — URL**

```python
from rest_framework.routers import DefaultRouter
from .views import ItemViewSet

router = DefaultRouter()
router.register(r"items", ItemViewSet)
urlpatterns = router.urls
```

Test : `http://localhost:8000/api/items/12345/` ✅

---

## 🎯 Étape 9 — Tooltips frontend

### Le quoi
Afficher une infobulle au survol d'un lien d'item, comme Wowhead.

### Le pourquoi
C'est la signature d'un site de base de données WoW. Sans ça, l'expérience utilisateur est pauvre.

### Le comment

**9.1 — Endpoint tooltip**

```python
from django.shortcuts import render
from items.models import ItemTemplate


def item_tooltip(request, entry):
    item = ItemTemplate.objects.get(entry=entry)
    return render(request, "items/tooltip.html", {"item": item})
```

**9.2 — Template tooltip**

```html
<div class="tooltip quality-{{ item.Quality }}">
  <div class="tooltip-name">{{ item.name }}</div>
  <div class="tooltip-level">Niveau d'objet {{ item.ItemLevel }}</div>
  <div class="tooltip-required">Niveau requis {{ item.RequiredLevel }}</div>
</div>
```

**9.3 — JavaScript**

```javascript
document.querySelectorAll('[data-item]').forEach(el => {
  el.addEventListener('mouseenter', async (e) => {
    const entry = e.target.dataset.item;
    const res = await fetch(`/api/items/${entry}/tooltip/`);
    const html = await res.text();
    showTooltip(html, e.pageX, e.pageY);
  });
});
```

---

## 🎯 Étape 10 — Recherche globale

### Le quoi
Chercher dans tous les objets, PNJs, sorts, quêtes en même temps.

### Le pourquoi
C'est **la** fonctionnalité la plus utilisée sur ce type de site.

### Le comment

Deux approches :

**Option A — PostgreSQL FTS**

```python
from django.contrib.postgres.search import SearchVector, SearchQuery

results = ItemTemplate.objects.annotate(
    search=SearchVector("name")
).filter(search=SearchQuery("épée"))
```

Simple, intégré, mais monolingue et peu tolérant aux fautes.

**Option B — Meilisearch**

Meilleure option en production :
- Recherche floue (tolérance aux fautes)
- Très rapide (< 50 ms)
- Interface d'admin incluse
- Support multi-index

```bash
pip install meilisearch
docker run -p 7700:7700 getmeili/meilisearch
```

Puis un signal Django qui indexe chaque item lors de sa lecture.

---

## 🎯 Étape 11 — Déploiement

### Le quoi
Mettre en ligne.

### Le pourquoi
Un projet local c'est bien. Accessible au monde, c'est mieux.

### Le comment

**11.1 — Dockeriser**

`docker-compose.yml` avec :
- `web` (Django + Gunicorn)
- `db` (PostgreSQL)
- `wowdb` (MySQL existant ou externalisé)
- `nginx` (reverse proxy)
- `meilisearch`

**11.2 — Nginx + Gunicorn**

```nginx
server {
    listen 80;
    server_name wotlkdb.example.com;
    client_max_body_size 20M;

    location /static/ { alias /app/staticfiles/; }
    location / {
        proxy_pass http://web:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**11.3 — Cloudflare**

Comme wotlkdb.com : place Cloudflare devant. Tu gagnes :
- CDN mondial
- Protection DDoS
- Cache automatique des pages statiques
- HTTPS gratuit

---

Parfait, on avance. Voici le plan pour cette session :

1.  **Option A** : Installer AzerothCore, importer la base de données `acore_world` et vérifier que tout fonctionne.
2.  **Option B** : Configurer le routeur multi-bases dans Django et générer les premiers modèles à partir des vraies tables.

---

## 🎯 Étape A1 — Installer les dépendances système et créer l'utilisateur MySQL

### Le quoi
Installer les outils nécessaires et créer l'utilisateur `acore` qui possédera les bases de données.

### Le pourquoi
AzerothCore a besoin de MySQL (ou MariaDB) pour fonctionner. On crée un utilisateur dédié `acore` plutôt que d'utiliser `root` — c'est une bonne pratique de sécurité, et c'est aussi ce que fait l'écosystème AzerothCore par défaut.

### Le comment

**A1.1 — Installer les dépendances**

```bash
sudo apt update && sudo apt install -y git curl unzip sudo mysql-server
```

> 💡 Sur Ubuntu 22.04+, MySQL 8.x est disponible par défaut. AzerothCore le supporte.

**A1.2 — Créer l'utilisateur MySQL `acore`**

Connecte-toi à MySQL en root :

```bash
sudo mysql -u root
```

Puis exécute :

```sql
DROP USER IF EXISTS 'acore'@'localhost';
DROP USER IF EXISTS 'acore'@'127.0.0.1';
CREATE USER 'acore'@'localhost' IDENTIFIED BY 'acore';
CREATE USER 'acore'@'127.0.0.1' IDENTIFIED BY 'acore';
GRANT ALL PRIVILEGES ON *.* TO 'acore'@'localhost' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'acore'@'127.0.0.1' WITH GRANT OPTION;
FLUSH PRIVILEGES;
exit;
```

> ⚠️ Change le mot de passe `acore` en production. Pour le développement local, c'est acceptable.

**A1.3 — Créer les bases de données**

```bash
sudo mysql -u root -e "CREATE DATABASE acore_world CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
sudo mysql -u root -e "CREATE DATABASE acore_auth CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
sudo mysql -u root -e "CREATE DATABASE acore_characters CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

On crée les trois bases par convention, mais pour notre projet Django, seule `acore_world` nous intéresse vraiment.

---

## 🎯 Étape A2 — Récupérer et importer la base `acore_world`

### Le quoi
Télécharger les fichiers SQL officiels d'AzerothCore et les importer dans MySQL.

### Le pourquoi
La base `acore_world` contient tout le contenu statique du jeu : objets, créatures, quêtes, sorts, etc. C'est la matière première de notre site.

### Le comment

**A2.1 — Cloner le dépôt AzerothCore**

```bash
git clone https://github.com/azerothcore/azerothcore-wotlk.git
cd azerothcore-wotlk
```

> 💡 On clone le dépôt entier même si on n'a pas besoin de compiler le serveur. Les fichiers SQL se trouvent dans `data/sql/`.

**A2.2 — Comprendre la structure des SQL**

```
data/sql/
├── base/           # Schéma + données de base
│   ├── acore_world.sql
│   ├── acore_auth.sql
│   └── acore_characters.sql
└── updates/        # Mises à jour incrémentales
    └── ...
```

Le fichier `base/acore_world.sql` est volumineux (plusieurs centaines de Mo). Il contient à la fois le **schéma** (CREATE TABLE) et les **données** (INSERT).

**A2.3 — Importer la base world**

```bash
mysql -u acore -p'acore' acore_world < data/sql/base/acore_world.sql
```

> 💡 Cette commande peut prendre **plusieurs minutes** selon ta machine. Sois patient.

**A2.4 — Vérifier l'import**

```bash
mysql -u acore -p'acore' acore_world -e "SELECT COUNT(*) FROM item_template;"
```

Tu devrais obtenir un nombre autour de **50 000 à 60 000**. Si c'est le cas, l'import a réussi. ✅

Vérifie aussi les créatures et les sorts :

```bash
mysql -u acore -p'acore' acore_world -e "
  SELECT 'items' AS tbl, COUNT(*) AS n FROM item_template
  UNION ALL SELECT 'creatures', COUNT(*) FROM creature_template
  UNION ALL SELECT 'spells', COUNT(*) FROM spell_template
  UNION ALL SELECT 'quests', COUNT(*) FROM quest_template;"
```

---

## 🎯 Étape A3 — Tester la connexion depuis Django

### Le quoi
Vérifier que Django peut se connecter à `acore_world` et lire des données.

### Le pourquoi
Avant d'écrire des modèles, il faut valider que la connexion fonctionne. Une erreur ici signifierait que la configuration multi-base est incorrecte.

### Le comment

**A3.1 — Mettre à jour `.env`**

```env
WOW_WORLD_DB=acore_world
WOW_DB_USER=acore
WOW_DB_PASSWORD=acore
WOW_DB_HOST=127.0.0.1
WOW_DB_PORT=3306
```

**A3.2 — Tester la connexion**

```bash
python manage.py shell
```

```python
from django.db import connections
cursor = connections["world"].cursor()
cursor.execute("SELECT COUNT(*) FROM item_template")
print(cursor.fetchone())
```

Si tu obtiens un nombre (ex: `(54321,)`), la connexion est opérationnelle. ✅

> 💡 Si tu as une erreur `Unknown database 'acore_world'`, vérifie que MySQL écoute bien sur `127.0.0.1:3306` et que l'utilisateur `acore` a les droits.

---

## 🎯 Étape B1 — Finaliser le routeur multi-bases

### Le quoi
Créer le fichier `db_routers.py` qui dirige les lectures des apps WoW vers la base `world`, et empêche Django de migrer ces tables.

### Le pourquoi
Django doit savoir quelle base interroger selon le modèle. Sans routeur, il utilise `default` par défaut — ce qui échouerait pour les modèles WoW. Le routeur est aussi le garde-fou qui empêche Django de créer ou modifier des tables gérées par AzerothCore.

### Le comment

**B1.1 — Créer `wotlkdb/db_routers.py`**

```python
class WoWRouter:
    """
    Route les apps WoW vers la base 'world' (lecture seule).
    Les apps Django restent sur 'default'.
    """
    wow_apps = {"items", "creatures", "spells", "quests", "dbc"}
    django_apps = {"core", "accounts", "api", "admin", "auth", "contenttypes", "sessions"}

    def db_for_read(self, model, **hints):
        if model._meta.app_label in self.wow_apps:
            return "world"
        if model._meta.app_label in self.django_apps:
            return "default"
        return None

    def db_for_write(self, model, **hints):
        # Écriture interdite sur la base world
        if model._meta.app_label in self.wow_apps:
            return None
        if model._meta.app_label in self.django_apps:
            return "default"
        return None

    def allow_relation(self, obj1, obj2, **hints):
        # Pas de relations cross-database
        return None

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        if app_label in self.wow_apps:
            return False
        if app_label in self.django_apps:
            return db == "default"
        return None
```

**B1.2 — Vérifier que le routeur est bien déclaré dans `base.py`**

```python
DATABASE_ROUTERS = ["wotlkdb.db_routers.WoWRouter"]
```

**B1.3 — Tester qu'aucune migration n'est créée pour les apps WoW**

```bash
python manage.py makemigrations
```

Tu dois voir quelque chose comme :

```
No changes detected in app 'items'
No changes detected in app 'creatures'
...
```

> 💡 Si Django tente de créer des migrations pour `items`, c'est que `managed = False` n'est pas correctement appliqué aux modèles, ou que le routeur n'est pas pris en compte.

---

## 🎯 Étape B2 — Générer les modèles avec `inspectdb`

### Le quoi
Utiliser `inspectdb` pour générer automatiquement les modèles Django à partir des tables existantes.

### Le pourquoi
Écrire à la main les modèles pour `item_template` (plus de 100 colonnes) serait fastidieux et source d'erreurs. `inspectdb` lit le schéma MySQL et génère le code Python correspondant.

### Le comment

**B2.1 — Générer les modèles pour les items**

```bash
python manage.py inspectdb --database=world item_template > items/models.py
```

**B2.2 — Nettoyer le fichier généré**

Ouvre `items/models.py`. Tu vas voir quelque chose comme :

```python
class ItemTemplate(models.Model):
    entry = models.AutoField(primary_key=True)
    class_ = models.IntegerField(db_column='class')
    subclass = models.IntegerField()
    name = models.CharField(max_length=255)
    # ... 100+ champs
    class Meta:
        managed = False
        db_table = 'item_template'
```

> 💡 `inspectdb` ajoute déjà `managed = False` et `db_table`. C'est exactement ce qu'on veut.

**B2.3 — Ajouter une méthode `__str__`**

En haut du fichier, ajoute :

```python
class ItemTemplate(models.Model):
    # ... champs
    class Meta:
        managed = False
        db_table = 'item_template'
        ordering = ['entry']

    def __str__(self):
        return f"[{self.entry}] {self.name}"
```

**B2.4 — Répéter pour les autres tables**

```bash
python manage.py inspectdb --database=world creature_template > creatures/models.py
python manage.py inspectdb --database=world spell_template > spells/models.py
python manage.py inspectdb --database=world quest_template > quests/models.py
```

> ⚠️ Ces tables ont aussi des centaines de colonnes. Le fichier généré sera long, mais c'est normal.

**B2.5 — Ajouter les `__str__` pour chaque modèle**

Pour `CreatureTemplate` :

```python
def __str__(self):
    return f"[{self.entry}] {self.name}"
```

Pour `SpellTemplate` :

```python
def __str__(self):
    return f"[{self.id}] {self.name}"
```

---

## 🎯 Étape B3 — Tester les modèles depuis le shell

### Le quoi
Exécuter des requêtes ORM pour vérifier que tout fonctionne.

### Le pourquoi
C'est le moment de vérité. Si les modèles, le routeur et la connexion sont corrects, tu dois pouvoir interroger la base `world` comme n'importe quelle base Django.

### Le comment

**B3.1 — Lancer le shell**

```bash
python manage.py shell
```

**B3.2 — Tester les items**

```python
from items.models import ItemTemplate

# Compter
print(ItemTemplate.objects.count())

# Chercher un item connu (ex: "Epée de guerre")
print(ItemTemplate.objects.filter(name__icontains="Sword").first())

# Filtrer par qualité (4 = Epic)
print(ItemTemplate.objects.filter(Quality=4).count())
```

**B3.3 — Tester les créatures**

```python
from creatures.models import CreatureTemplate

# Compter
print(CreatureTemplate.objects.count())

# Chercher un PNJ connu
print(CreatureTemplate.objects.filter(name__icontains="Thrall").first())
```

**B3.4 — Tester les sorts**

```python
from spells.models import SpellTemplate

print(SpellTemplate.objects.count())
print(SpellTemplate.objects.filter(name__icontains="Fireball").first())
```

Si toutes ces requêtes retournent des résultats cohérents, **la fondation est posée**. ✅

---

## 🎯 Étape B4 — Enregistrer les modèles dans l'admin

### Le quoi
Exposer les modèles dans l'interface d'admin Django.

### Le pourquoi
L'admin te donne immédiatement une interface pour naviguer dans les données sans écrire une ligne de HTML. C'est idéal pour explorer et valider.

### Le comment

**B4.1 — `items/admin.py`**

```python
from django.contrib import admin
from .models import ItemTemplate


@admin.register(ItemTemplate)
class ItemTemplateAdmin(admin.ModelAdmin):
    list_display = ("entry", "name", "Quality", "ItemLevel", "RequiredLevel")
    search_fields = ("name", "entry")
    list_filter = ("Quality", "class_")
    readonly_fields = [f.name for f in ItemTemplate._meta.fields]
```

**B4.2 — `creatures/admin.py`**

```python
from django.contrib import admin
from .models import CreatureTemplate


@admin.register(CreatureTemplate)
class CreatureTemplateAdmin(admin.ModelAdmin):
    list_display = ("entry", "name", "minlevel", "maxlevel", "faction")
    search_fields = ("name", "entry")
    list_filter = ("minlevel", "faction")
    readonly_fields = [f.name for f in CreatureTemplate._meta.fields]
```

**B4.3 — `spells/admin.py`**

```python
from django.contrib import admin
from .models import SpellTemplate


@admin.register(SpellTemplate)
class SpellTemplateAdmin(admin.ModelAdmin):
    list_display = ("id", "name", "school")
    search_fields = ("name", "id")
    readonly_fields = [f.name for f in SpellTemplate._meta.fields]
```

**B4.4 — Lancer le serveur et tester**

```bash
python manage.py runserver
```

Va sur `http://localhost:8000/admin/` et connecte-toi. Tu devrais voir les sections **Items**, **Creatures**, **Spells** avec des données réelles. ✅

---

## 📊 Récapitulatif de la session

| Étape | Ce qui a été fait | Résultat |
|-------|-------------------|----------|
| A1 | Installation MySQL + user `acore` | Bases créées |
| A2 | Import `acore_world.sql` | ~50 000 items en base |
| A3 | Test connexion Django → MySQL | Connexion validée |
| B1 | Création du routeur multi-base | Django ne touche pas aux tables WoW |
| B2 | Génération des modèles via `inspectdb` | Modèles Python prêts |
| B3 | Test des requêtes ORM | Lecture des données fonctionnelle |
| B4 | Enregistrement dans l'admin | Interface de navigation opérationnelle |

---

Parfait, on attaque le frontend. Voici le plan pour cette session :

1. **C1** — Structure des URLs publiques
2. **C2** — Template de base (`base.html`) + CSS
3. **C3** — Page liste des objets avec pagination
4. **C4** — Page détail d'un objet
5. **C5** — Recherche basique
6. **C6** — Filtres (qualité, classe, niveau)

À la fin, tu auras un site navigable avec des vraies données.

---

## 🎯 Étape C1 — Structure des URLs publiques

### Le quoi
Définir les routes publiques du site, à la manière de wotlkdb.com.

### Le pourquoi
Les URLs sont l'ossature du site. Elles doivent être **lisibles**, **stables** (pour le SEO et le partage de liens) et **hiérarchiques**. wotlkdb.com utilise des URLs comme `/item=12345` ou `/?item=12345`. On va s'en inspirer mais avec une syntaxe plus propre et moderne.

### Le comment

**C1.1 — Choisir la convention d'URL**

Voici la convention que je te propose :

| Page | URL |
|------|-----|
| Accueil | `/` |
| Liste d'objets | `/items/` |
| Détail d'un objet | `/items/12345/` |
| Recherche | `/search/?q=sword` |
| Liste des créatures | `/creatures/` |
| Détail d'une créature | `/creatures/448/` |
| Liste des sorts | `/spells/` |
| Détail d'un sort | `/spells/133/` |

> 💡 **Pourquoi `/items/12345/` plutôt que `?item=12345` ?** Les URLs "propres" sont mieux référencées, plus faciles à retenir, et permettent une hiérarchie claire (`/items/12345/edit/` plus tard). C'est une pratique standard.

**C1.2 — Créer les URLs de l'app `items`**

Crée `items/urls.py` :

```python
from django.urls import path
from . import views

app_name = "items"

urlpatterns = [
    path("", views.item_list, name="list"),
    path("<int:entry>/", views.item_detail, name="detail"),
]
```

> 💡 `app_name = "items"` crée un **namespace**. Tu pourras écrire `{% url 'items:detail' entry=12345 %}` dans tes templates. C'est plus robuste que des URLs en dur.

**C1.3 — Créer les URLs des apps `creatures` et `spells`**

`creatures/urls.py` :

```python
from django.urls import path
from . import views

app_name = "creatures"

urlpatterns = [
    path("", views.creature_list, name="list"),
    path("<int:entry>/", views.creature_detail, name="detail"),
]
```

`spells/urls.py` :

```python
from django.urls import path
from . import views

app_name = "spells"

urlpatterns = [
    path("", views.spell_list, name="list"),
    path("<int:id>/", views.spell_detail, name="detail"),
]
```

**C1.4 — Créer les URLs de l'app `core`**

`core/urls.py` :

```python
from django.urls import path
from . import views

app_name = "core"

urlpatterns = [
    path("", views.home, name="home"),
    path("search/", views.search, name="search"),
]
```

**C1.5 — Brancher le tout dans `wotlkdb/urls.py`**

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("", include("core.urls")),
    path("items/", include("items.urls")),
    path("creatures/", include("creatures.urls")),
    path("spells/", include("spells.urls")),
]
```

---

## 🎯 Étape C2 — Template de base + CSS

### Le quoi
Créer un template parent dont tous les autres héritent, avec un style inspiré de Wowhead/wotlkdb.

### Le pourquoi
Sans template de base, tu dupliquerais le header, le footer et le CSS sur chaque page. L'héritage de templates Django (`{% extends %}`) permet de définir la structure une fois et de ne remplir que le contenu spécifique à chaque page.

### Le comment

**C2.1 — Configurer les templates**

Dans `base.py`, vérifie :

```python
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [BASE_DIR / "templates"],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
            ],
        },
    },
]
```

Crée le dossier `templates/` à la racine du projet :

```bash
mkdir -p templates
```

**C2.2 — Créer `templates/base.html`**

```html
{% load static %}
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{% block title %}WotLK DB{% endblock %}</title>
  <link rel="stylesheet" href="{% static 'css/main.css' %}">
</head>
<body>
  <header class="site-header">
    <div class="container">
      <a href="{% url 'core:home' %}" class="logo">WotLK DB</a>

      <nav class="main-nav">
        <a href="{% url 'items:list' %}">Objets</a>
        <a href="{% url 'creatures:list' %}">PNJs</a>
        <a href="{% url 'spells:list' %}">Sorts</a>
      </nav>

      <form action="{% url 'core:search' %}" method="get" class="search-form">
        <input type="text" name="q" placeholder="Rechercher..." value="{{ request.GET.q|default:'' }}">
        <button type="submit">🔍</button>
      </form>
    </div>
  </header>

  <main class="container">
    {% block content %}{% endblock %}
  </main>

  <footer class="site-footer">
    <div class="container">
      <p>WotLK DB — Base de données non officielle pour World of Warcraft 3.3.5a</p>
    </div>
  </footer>
</body>
</html>
```

> 💡 `{% load static %}` charge la bibliothèque qui donne accès au tag `{% static %}`. Sans ça, `{% static 'css/main.css' %}` échoue.

**C2.3 — Configurer les fichiers statiques**

Dans `base.py` :

```python
STATIC_URL = "static/"
STATICFILES_DIRS = [BASE_DIR / "static"]
STATIC_ROOT = BASE_DIR / "staticfiles"
```

Crée le dossier :

```bash
mkdir -p static/css
```

**C2.4 — Créer `static/css/main.css`**

```css
* { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  background: #1a1a1a;
  color: #d4d4d4;
  line-height: 1.5;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

/* Header */
.site-header {
  background: #0d0d0d;
  border-bottom: 1px solid #2a2a2a;
  padding: 15px 0;
}
.site-header .container {
  display: flex;
  align-items: center;
  gap: 30px;
}
.logo {
  font-size: 24px;
  font-weight: bold;
  color: #ffd100;
  text-decoration: none;
}
.main-nav {
  display: flex;
  gap: 20px;
  flex: 1;
}
.main-nav a {
  color: #d4d4d4;
  text-decoration: none;
  padding: 5px 10px;
  border-radius: 3px;
}
.main-nav a:hover {
  background: #2a2a2a;
  color: #ffd100;
}
.search-form {
  display: flex;
  gap: 5px;
}
.search-form input {
  background: #2a2a2a;
  border: 1px solid #3a3a3a;
  color: #d4d4d4;
  padding: 6px 12px;
  border-radius: 3px;
  width: 250px;
}
.search-form button {
  background: #ffd100;
  border: none;
  padding: 6px 12px;
  border-radius: 3px;
  cursor: pointer;
}

/* Main */
main {
  padding: 30px 20px;
}

/* Qualités d'objets (couleurs WoW) */
.q0 { color: #9d9d9d; } /* Poor */
.q1 { color: #ffffff; } /* Common */
.q2 { color: #1eff00; } /* Uncommon */
.q3 { color: #0070dd; } /* Rare */
.q4 { color: #a335ee; } /* Epic */
.q5 { color: #ff8000; } /* Legendary */
.q6 { color: #e6cc80; } /* Artifact */
.q7 { color: #e6cc80; } /* Heirloom */

/* Cartes / grilles */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 15px;
}
.card {
  background: #232323;
  border: 1px solid #2a2a2a;
  border-radius: 5px;
  padding: 12px;
}
.card a {
  text-decoration: none;
}

/* Footer */
.site-footer {
  background: #0d0d0d;
  border-top: 1px solid #2a2a2a;
  padding: 20px 0;
  margin-top: 40px;
  text-align: center;
  color: #666;
  font-size: 14px;
}
```

> 💡 Les couleurs de qualité (`q0` à `q7`) sont celles de Blizzard. On les réutilisera partout.

**C2.5 — Ajouter les URLs de l'admin dans le template**

Le lien vers l'admin n'est pas nécessaire. On garde le template épuré.

---

## 🎯 Étape C3 — Page liste des objets

### Le quoi
Une page qui affiche tous les objets, paginée, avec un rendu propre.

### Le pourquoi
C'est la page la plus utilisée : elle permet de parcourir le contenu. La pagination est indispensable — 50 000 items sur une seule page feraient planter le navigateur.

### Le comment

**C3.1 — Vue `item_list` dans `items/views.py`**

```python
from django.core.paginator import Paginator
from django.shortcuts import render
from .models import ItemTemplate

QUALITY_CHOICES = [
    (0, "Médiocre"), (1, "Commun"), (2, "Inhabituel"),
    (3, "Rare"), (4, "Épique"), (5, "Légendaire"),
    (6, "Artéfact"), (7, "Relique"),
]


def item_list(request):
    qs = ItemTemplate.objects.all()

    # Filtres
    quality = request.GET.get("quality")
    if quality not in (None, ""):
        qs = qs.filter(Quality=int(quality))

    class_filter = request.GET.get("class")
    if class_filter:
        qs = qs.filter(class_=int(class_filter))

    # Tri
    sort = request.GET.get("sort", "name")
    if sort == "level":
        qs = qs.order_by("-ItemLevel", "name")
    elif sort == "entry":
        qs = qs.order_by("entry")
    else:
        qs = qs.order_by("name")

    # Pagination
    paginator = Paginator(qs, 40)
    page_number = request.GET.get("page", 1)
    page_obj = paginator.get_page(page_number)

    return render(request, "items/item_list.html", {
        "page_obj": page_obj,
        "quality_choices": QUALITY_CHOICES,
        "current_quality": quality,
        "current_sort": sort,
    })
```

> 💡 `Paginator` limite à 40 items par page. `get_page` gère automatiquement les numéros de page invalides (renvoie la première ou la dernière page).

**C3.2 — Template `items/templates/items/item_list.html`**

Crée la structure :

```bash
mkdir -p items/templates/items
```

Puis :

```html
{% extends "base.html" %}

{% block title %}Objets — WotLK DB{% endblock %}

{% block content %}
<h1>Objets</h1>

<form method="get" class="filters">
  <label>Qualité :
    <select name="quality" onchange="this.form.submit()">
      <option value="">Toutes</option>
      {% for value, label in quality_choices %}
        <option value="{{ value }}" {% if current_quality == value|stringformat:"s" %}selected{% endif %}>
          {{ label }}
        </option>
      {% endfor %}
    </select>
  </label>

  <label>Trier par :
    <select name="sort" onchange="this.form.submit()">
      <option value="name" {% if current_sort == "name" %}selected{% endif %}>Nom</option>
      <option value="level" {% if current_sort == "level" %}selected{% endif %}>Niveau</option>
      <option value="entry" {% if current_sort == "entry" %}selected{% endif %}>ID</option>
    </select>
  </label>
</form>

<p class="result-count">{{ page_obj.paginator.count }} objets trouvés</p>

<div class="grid">
  {% for item in page_obj %}
    <div class="card">
      <a href="{% url 'items:detail' entry=item.entry %}" class="q{{ item.Quality }}">
        <strong>{{ item.name }}</strong>
      </a>
      <div class="item-meta">
        Niveau {{ item.ItemLevel }}
        {% if item.RequiredLevel %} · Requis {{ item.RequiredLevel }}{% endif %}
      </div>
    </div>
  {% empty %}
    <p>Aucun objet ne correspond à ces critères.</p>
  {% endfor %}
</div>

{% if page_obj.has_other_pages %}
<nav class="pagination">
  {% if page_obj.has_previous %}
    <a href="?page={{ page_obj.previous_page_number }}{% if current_quality %}&quality={{ current_quality }}{% endif %}{% if current_sort %}&sort={{ current_sort }}{% endif %}">← Précédent</a>
  {% endif %}

  <span>Page {{ page_obj.number }} / {{ page_obj.paginator.num_pages }}</span>

  {% if page_obj.has_next %}
    <a href="?page={{ page_obj.next_page_number }}{% if current_quality %}&quality={{ current_quality }}{% endif %}{% if current_sort %}&sort={{ current_sort }}{% endif %}">Suivant →</a>
  {% endif %}
</nav>
{% endif %}
{% endblock %}
```

> 💡 Note l'astuce `?page=X&quality=Y&sort=Z` : on **préserve les filtres** lors du changement de page. Sans ça, changer de page réinitialiserait les filtres.

**C3.3 — Ajouter du CSS pour les filtres et la pagination**

Ajoute à `main.css` :

```css
/* Filtres */
.filters {
  display: flex;
  gap: 20px;
  margin: 20px 0;
  padding: 15px;
  background: #232323;
  border-radius: 5px;
}
.filters select {
  background: #1a1a1a;
  border: 1px solid #3a3a3a;
  color: #d4d4d4;
  padding: 5px 10px;
  border-radius: 3px;
}

.result-count {
  color: #888;
  margin-bottom: 15px;
}

/* Pagination */
.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 15px;
  margin: 30px 0;
}
.pagination a {
  color: #ffd100;
  text-decoration: none;
  padding: 8px 15px;
  background: #232323;
  border-radius: 3px;
}
.pagination a:hover {
  background: #2a2a2a;
}

/* Meta info dans les cartes */
.item-meta {
  color: #888;
  font-size: 13px;
  margin-top: 5px;
}
```

**C3.4 — Tester**

```bash
python manage.py runserver
```

Va sur `http://localhost:8000/items/`. Tu devrais voir 40 objets paginés. ✅

---

## 🎯 Étape C4 — Page détail d'un objet

### Le quoi
Une page qui affiche toutes les informations d'un objet, dans un layout proche de Wowhead.

### Le pourquoi
C'est la page la plus importante du site : c'est celle qu'on partage, qu'on indexe, qu'on consulte. Elle doit contenir toutes les infos utiles et être lisible.

### Le comment

**C4.1 — Vue `item_detail`**

```python
from django.shortcuts import render, get_object_or_404
from .models import ItemTemplate


def item_detail(request, entry):
    item = get_object_or_404(ItemTemplate, entry=entry)
    return render(request, "items/item_detail.html", {"item": item})
```

> 💡 `get_object_or_404` renvoie une 404 propre si l'item n'existe pas. C'est plus élégant qu'un `try/except` manuel.

**C4.2 — Template `items/templates/items/item_detail.html`**

```html
{% extends "base.html" %}

{% block title %}{{ item.name }} — WotLK DB{% endblock %}

{% block content %}
<article class="item-detail">
  <header class="item-header q{{ item.Quality }}">
    <h1>{{ item.name }}</h1>
    <div class="item-subtitle">
      {% if item.ItemLevel %}Niveau d'objet {{ item.ItemLevel }}{% endif %}
      {% if item.RequiredLevel %} · Niveau requis {{ item.RequiredLevel }}{% endif %}
    </div>
  </header>

  <div class="item-body">
    <section class="item-stats">
      <h2>Statistiques</h2>
      <table class="stats-table">
        <tr><th>ID</th><td>{{ item.entry }}</td></tr>
        <tr><th>Qualité</th><td>{{ item.Quality }}</td></tr>
        <tr><th>Classe</th><td>{{ item.class_ }} / {{ item.subclass }}</td></tr>
        <tr><th>Type d'inventaire</th><td>{{ item.inventory_type }}</td></tr>
        {% if item.stat_type1 %}
          <tr><th>Stat 1</th><td>Type {{ item.stat_type1 }} : +{{ item.stat_value1 }}</td></tr>
        {% endif %}
        {% if item.armor %}
          <tr><th>Armure</th><td>{{ item.armor }}</td></tr>
        {% endif %}
        {% if item.dmg_min1 %}
          <tr><th>Dégâts</th><td>{{ item.dmg_min1 }} - {{ item.dmg_max1 }}</td></tr>
        {% endif %}
      </table>
    </section>

    <section class="item-description">
      <h2>Description</h2>
      <p>{{ item.description|default:"Aucune description." }}</p>
    </section>
  </div>

  <footer class="item-footer">
    <a href="{% url 'items:list' %}">← Retour à la liste</a>
  </footer>
</article>
{% endblock %}
```

**C4.3 — CSS pour le détail**

```css
.item-detail {
  max-width: 900px;
  margin: 0 auto;
}
.item-header {
  padding: 20px;
  background: #232323;
  border-radius: 5px 5px 0 0;
  border-left: 4px solid currentColor;
}
.item-header h1 {
  font-size: 28px;
  margin-bottom: 5px;
}
.item-subtitle {
  color: #888;
  font-size: 14px;
}
.item-body {
  background: #1e1e1e;
  padding: 20px;
  border-radius: 0 0 5px 5px;
}
.item-body h2 {
  font-size: 18px;
  margin-bottom: 10px;
  color: #ffd100;
}
.stats-table {
  width: 100%;
  border-collapse: collapse;
}
.stats-table th,
.stats-table td {
  padding: 8px 12px;
  text-align: left;
  border-bottom: 1px solid #2a2a2a;
}
.stats-table th {
  color: #888;
  font-weight: normal;
  width: 200px;
}
.item-footer {
  margin-top: 20px;
}
.item-footer a {
  color: #ffd100;
  text-decoration: none;
}
```

**C4.4 — Rendre les cartes cliquables**

Dans `item_list.html`, les cartes sont déjà des liens vers `{% url 'items:detail' entry=item.entry %}`. Test : clique sur un objet dans la liste, tu arrives sur sa page détail. ✅

---

## 🎯 Étape C5 — Recherche basique

### Le quoi
Une page de recherche qui interroge objets, créatures et sorts en même temps.

### Le pourquoi
C'est **la** fonctionnalité la plus utilisée. L'utilisateur veut taper "Thrall" ou "Epée" et trouver tout ce qui correspond.

### Le comment

**C5.1 — Vue `search` dans `core/views.py`**

```python
from django.shortcuts import render
from items.models import ItemTemplate
from creatures.models import CreatureTemplate
from spells.models import SpellTemplate


def search(request):
    q = request.GET.get("q", "").strip()
    results = {"items": [], "creatures": [], "spells": []}

    if q:
        results["items"] = ItemTemplate.objects.filter(name__icontains=q)[:20]
        results["creatures"] = CreatureTemplate.objects.filter(name__icontains=q)[:20]
        results["spells"] = SpellTemplate.objects.filter(name__icontains=q)[:20]

    total = sum(len(v) for v in results.values())

    return render(request, "core/search.html", {
        "q": q,
        "results": results,
        "total": total,
    })
```

> 💡 On limite à 20 résultats par catégorie. Pour une recherche plus poussée, on utilisera Meilisearch plus tard. Pour l'instant, `icontains` suffit.

**C5.2 — Template `core/templates/core/search.html`**

```bash
mkdir -p core/templates/core
```

```html
{% extends "base.html" %}

{% block title %}Recherche : {{ q }} — WotLK DB{% endblock %}

{% block content %}
<h1>Résultats pour « {{ q }} »</h1>
<p class="result-count">{{ total }} résultats</p>

{% if results.items %}
  <section class="search-section">
    <h2>Objets ({{ results.items|length }})</h2>
    <div class="grid">
      {% for item in results.items %}
        <div class="card">
          <a href="{% url 'items:detail' entry=item.entry %}" class="q{{ item.Quality }}">
            {{ item.name }}
          </a>
          <div class="item-meta">Niveau {{ item.ItemLevel }}</div>
        </div>
      {% endfor %}
    </div>
  </section>
{% endif %}

{% if results.creatures %}
  <section class="search-section">
    <h2>PNJs ({{ results.creatures|length }})</h2>
    <div class="grid">
      {% for creature in results.creatures %}
        <div class="card">
          <a href="{% url 'creatures:detail' entry=creature.entry %}">
            {{ creature.name }}
          </a>
          <div class="item-meta">Niveau {{ creature.minlevel }}-{{ creature.maxlevel }}</div>
        </div>
      {% endfor %}
    </div>
  </section>
{% endif %}

{% if results.spells %}
  <section class="search-section">
    <h2>Sorts ({{ results.spells|length }})</h2>
    <div class="grid">
      {% for spell in results.spells %}
        <div class="card">
          <a href="{% url 'spells:detail' id=spell.id %}">
            {{ spell.name }}
          </a>
        </div>
      {% endfor %}
    </div>
  </section>
{% endif %}

{% if not total %}
  <p>Aucun résultat. Essaie un autre terme.</p>
{% endif %}
{% endblock %}
```

**C5.3 — CSS**

```css
.search-section {
  margin: 30px 0;
}
.search-section h2 {
  font-size: 20px;
  color: #ffd100;
  margin-bottom: 15px;
  border-bottom: 1px solid #2a2a2a;
  padding-bottom: 8px;
}
```

**C5.4 — Vue `home` (accueil)**

Dans `core/views.py` :

```python
def home(request):
    return render(request, "core/home.html", {
        "item_count": ItemTemplate.objects.count(),
        "creature_count": CreatureTemplate.objects.count(),
        "spell_count": SpellTemplate.objects.count(),
    })
```

`core/templates/core/home.html` :

```html
{% extends "base.html" %}

{% block title %}WotLK DB — Accueil{% endblock %}

{% block content %}
<div class="hero">
  <h1>WotLK DB</h1>
  <p>Base de données pour World of Warcraft : Wrath of the Lich King 3.3.5a</p>
</div>

<div class="stats-grid">
  <a href="{% url 'items:list' %}" class="stat-card">
    <div class="stat-number">{{ item_count }}</div>
    <div class="stat-label">Objets</div>
  </a>
  <a href="{% url 'creatures:list' %}" class="stat-card">
    <div class="stat-number">{{ creature_count }}</div>
    <div class="stat-label">PNJs</div>
  </a>
  <a href="{% url 'spells:list' %}" class="stat-card">
    <div class="stat-number">{{ spell_count }}</div>
    <div class="stat-label">Sorts</div>
  </a>
</div>
{% endblock %}
```

**C5.5 — CSS pour l'accueil**

```css
.hero {
  text-align: center;
  padding: 60px 20px 40px;
}
.hero h1 {
  font-size: 48px;
  color: #ffd100;
  margin-bottom: 10px;
}
.hero p {
  color: #888;
  font-size: 18px;
}
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
  max-width: 800px;
  margin: 40px auto;
}
.stat-card {
  background: #232323;
  padding: 30px;
  border-radius: 5px;
  text-align: center;
  text-decoration: none;
  color: inherit;
  transition: background 0.2s;
}
.stat-card:hover {
  background: #2a2a2a;
}
.stat-number {
  font-size: 36px;
  font-weight: bold;
  color: #ffd100;
}
.stat-label {
  color: #888;
  margin-top: 5px;
}
```

---

## 🎯 Étape C6 — Ajouter les vues créatures et sorts

### Le quoi
Implémenter les mêmes pages pour les créatures et les sorts.

### Le pourquoi
Sans ça, les liens du menu retourneraient des erreurs 500.

### Le comment

**C6.1 — `creatures/views.py`**

```python
from django.core.paginator import Paginator
from django.shortcuts import render, get_object_or_404
from .models import CreatureTemplate


def creature_list(request):
    qs = CreatureTemplate.objects.all().order_by("name")
    paginator = Paginator(qs, 40)
    page_obj = paginator.get_page(request.GET.get("page", 1))
    return render(request, "creatures/creature_list.html", {"page_obj": page_obj})


def creature_detail(request, entry):
    creature = get_object_or_404(CreatureTemplate, entry=entry)
    return render(request, "creatures/creature_detail.html", {"creature": creature})
```

**C6.2 — Template liste des créatures**

`creatures/templates/creatures/creature_list.html` :

```html
{% extends "base.html" %}
{% block title %}PNJs — WotLK DB{% endblock %}
{% block content %}
<h1>PNJs</h1>
<p class="result-count">{{ page_obj.paginator.count }} PNJs</p>

<div class="grid">
  {% for creature in page_obj %}
    <div class="card">
      <a href="{% url 'creatures:detail' entry=creature.entry %}">
        {{ creature.name }}
      </a>
      <div class="item-meta">
        Niveau {{ creature.minlevel }}-{{ creature.maxlevel }}
      </div>
    </div>
  {% endfor %}
</div>

{% if page_obj.has_other_pages %}
<nav class="pagination">
  {% if page_obj.has_previous %}
    <a href="?page={{ page_obj.previous_page_number }}">← Précédent</a>
  {% endif %}
  <span>Page {{ page_obj.number }} / {{ page_obj.paginator.num_pages }}</span>
  {% if page_obj.has_next %}
    <a href="?page={{ page_obj.next_page_number }}">Suivant →</a>
  {% endif %}
</nav>
{% endif %}
{% endblock %}
```

**C6.3 — Template détail créature**

`creatures/templates/creatures/creature_detail.html` :

```html
{% extends "base.html" %}
{% block title %}{{ creature.name }} — WotLK DB{% endblock %}
{% block content %}
<article class="item-detail">
  <header class="item-header">
    <h1>{{ creature.name }}</h1>
    {% if creature.subname %}<div class="item-subtitle">{{ creature.subname }}</div>{% endif %}
  </header>
  <div class="item-body">
    <h2>Informations</h2>
    <table class="stats-table">
      <tr><th>ID</th><td>{{ creature.entry }}</td></tr>
      <tr><th>Niveau</th><td>{{ creature.minlevel }} - {{ creature.maxlevel }}</td></tr>
      <tr><th>Faction</th><td>{{ creature.faction }}</td></tr>
      <tr><th>Vitesse (marche)</th><td>{{ creature.speed_walk }}</td></tr>
      <tr><th>Vitesse (course)</th><td>{{ creature.speed_run }}</td></tr>
    </table>
  </div>
</article>
{% endblock %}
```

**C6.4 — Faire pareil pour les sorts**

`spells/views.py` :

```python
from django.core.paginator import Paginator
from django.shortcuts import render, get_object_or_404
from .models import SpellTemplate


def spell_list(request):
    qs = SpellTemplate.objects.all().order_by("name")
    paginator = Paginator(qs, 40)
    page_obj = paginator.get_page(request.GET.get("page", 1))
    return render(request, "spells/spell_list.html", {"page_obj": page_obj})


def spell_detail(request, id):
    spell = get_object_or_404(SpellTemplate, id=id)
    return render(request, "spells/spell_detail.html", {"spell": spell})
```

Crée les templates correspondants sur le même modèle.

---

## 🎯 Récapitulatif de la session

| Étape | Contenu | Résultat |
|-------|---------|----------|
| C1 | URLs propres et namespacées | Structure stable |
| C2 | Template de base + CSS | Design uniforme |
| C3 | Liste d'objets paginée + filtres qualité/tri | Navigation fluide |
| C4 | Page détail d'un objet | Fiches complètes |
| C5 | Recherche globale + accueil | Site utilisable |
| C6 | Créatures et sorts | Site complet |

À ce stade, tu as un **site fonctionnel** avec des vraies données. ✅

---

Excellent choix. On va construire les tooltips comme Wowhead, puis intégrer les vraies icônes du jeu. C'est ce qui va transformer ton site d'un simple projet Django en un outil crédible.

Voici le plan :

**Option D — Tooltips**
- D1 : Endpoint API pour le HTML du tooltip
- D2 : JavaScript qui intercepte le survol
- D3 : CSS stylisé aux couleurs de WoW
- D4 : Tooltips pour créatures et sorts
- D5 : Mise en cache HTTP

**Option F — Icônes**
- F1 : Comprendre la chaîne de données
- F2 : Récupérer les icônes (deux stratégies)
- F3 : Parser `ItemDisplayInfo.dbc`
- F4 : Modèle Django de mapping
- F5 : Servir les icônes
- F6 : Intégration dans les templates

---

## 🎯 Étape D1 — Endpoint API pour le tooltip

### Le quoi
Créer une vue Django qui retourne uniquement le **fragment HTML** du tooltip d'un item.

### Le pourquoi
Un tooltip n'est pas une page complète. C'est un petit bloc HTML injecté dans une infobulle flottante. Si on retournait la page entière, on chargerait le header, le footer, le CSS... tout ça pour 200 pixels de contenu. Un endpoint dédié est bien plus léger et rapide.

### Le comment

**D1.1 — Créer une app `api`**

```bash
python manage.py startapp api
```

Ajoute `"api"` à `INSTALLED_APPS` dans `base.py`.

**D1.2 — Créer `api/urls.py`**

```python
from django.urls import path
from . import views

app_name = "api"

urlpatterns = [
    path("item/<int:entry>/tooltip/", views.item_tooltip, name="item_tooltip"),
    path("creature/<int:entry>/tooltip/", views.creature_tooltip, name="creature_tooltip"),
    path("spell/<int:id>/tooltip/", views.spell_tooltip, name="spell_tooltip"),
]
```

**D1.3 — Brancher dans `wotlkdb/urls.py`**

```python
urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/", include("api.urls")),
    path("", include("core.urls")),
    path("items/", include("items.urls")),
    path("creatures/", include("creatures.urls")),
    path("spells/", include("spells.urls")),
]
```

**D1.4 — Vue `item_tooltip`**

Dans `api/views.py` :

```python
from django.shortcuts import get_object_or_404, render
from django.views.decorators.cache import cache_page
from items.models import ItemTemplate


@cache_page(60 * 60 * 24)  # cache HTTP 24h
def item_tooltip(request, entry):
    item = get_object_or_404(ItemTemplate, entry=entry)
    return render(request, "api/tooltips/item.html", {"item": item})
```

> 💡 `cache_page` ajoute un header `Cache-Control` qui autorise le navigateur et les proxies (Cloudflare) à mettre en cache la réponse. Un tooltip ne change jamais, c'est parfait.

**D1.5 — Template du tooltip item**

`api/templates/api/tooltips/item.html` :

```html
<div class="wow-tooltip">
  <div class="wow-tooltip-name q{{ item.Quality }}">{{ item.name }}</div>

  {% if item.ItemLevel %}
    <div class="wow-tooltip-line">Niveau d'objet {{ item.ItemLevel }}</div>
  {% endif %}

  {% if item.RequiredLevel %}
    <div class="wow-tooltip-line wow-tooltip-required">
      Niveau {{ item.RequiredLevel }} requis
    </div>
  {% endif %}

  {% if item.armor %}
    <div class="wow-tooltip-line">{{ item.armor }} Armure</div>
  {% endif %}

  {% if item.dmg_min1 %}
    <div class="wow-tooltip-line">
      {{ item.dmg_min1 }} - {{ item.dmg_max1 }} Dégâts
    </div>
  {% endif %}

  {% if item.stat_type1 and item.stat_value1 %}
    <div class="wow-tooltip-line wow-tooltip-stat">
      +{{ item.stat_value1 }} {{ item.stat_type1 }}
    </div>
  {% endif %}

  {% if item.description %}
    <div class="wow-tooltip-flavor">« {{ item.description }} »</div>
  {% endif %}
</div>
```

> 💡 Ce template est un **partial** : pas de `{% extends %}`, pas de balises `<html>`. Il sera injecté tel quel dans le DOM par JavaScript.

**D1.6 — Tester**

```bash
python manage.py runserver
```

Va sur `http://localhost:8000/api/item/12345/tooltip/`. Tu devrais voir le HTML brut du tooltip. ✅

---

## 🎯 Étape D2 — JavaScript qui intercepte le survol

### Le quoi
Un script qui détecte le survol des liens `[data-item]`, fetch le tooltip, et l'affiche près du curseur.

### Le pourquoi
L'utilisateur ne doit pas cliquer pour voir le tooltip. Le survol doit suffire, comme sur Wowhead. C'est ce qui rend l'expérience fluide et informative.

### Le comment

**D2.1 — Marquer les liens dans les templates**

Dans `items/item_list.html`, remplace :

```html
<a href="{% url 'items:detail' entry=item.entry %}" class="q{{ item.Quality }}">
```

par :

```html
<a href="{% url 'items:detail' entry=item.entry %}"
   class="q{{ item.Quality }}"
   data-item="{{ item.entry }}">
```

Le `data-item` est la clé : c'est lui que le JS va lire pour construire l'URL du tooltip.

**D2.2 — Créer `static/js/tooltip.js`**

```javascript
(function () {
  "use strict";

  const CACHE = new Map();
  let tooltipEl = null;
  let hoverTimer = null;
  let currentTarget = null;

  function getTooltipEl() {
    if (!tooltipEl) {
      tooltipEl = document.createElement("div");
      tooltipEl.id = "wow-tooltip";
      tooltipEl.style.display = "none";
      document.body.appendChild(tooltipEl);
    }
    return tooltipEl;
  }

  async function fetchTooltip(url) {
    if (CACHE.has(url)) return CACHE.get(url);
    const res = await fetch(url);
    if (!res.ok) throw new Error("Tooltip introuvable");
    const html = await res.text();
    CACHE.set(url, html);
    return html;
  }

  function positionTooltip(x, y) {
    const el = getTooltipEl();
    const margin = 15;
    let left = x + margin;
    let top = y + margin;

    // Empêche le débordement à droite/bas
    const rect = el.getBoundingClientRect();
    if (left + rect.width > window.innerWidth) {
      left = x - rect.width - margin;
    }
    if (top + rect.height > window.innerHeight) {
      top = y - rect.height - margin;
    }

    el.style.left = left + "px";
    el.style.top = top + "px";
  }

  async function showTooltip(target, x, y) {
    const url = target.dataset.tooltipUrl;
    try {
      const html = await fetchTooltip(url);
      const el = getTooltipEl();
      el.innerHTML = html;
      el.style.display = "block";
      positionTooltip(x, y);
    } catch (e) {
      console.warn(e);
    }
  }

  function hideTooltip() {
    if (tooltipEl) tooltipEl.style.display = "none";
  }

  function resolveUrl(target) {
    if (target.dataset.tooltipUrl) return target.dataset.tooltipUrl;
    if (target.dataset.item) return `/api/item/${target.dataset.item}/tooltip/`;
    if (target.dataset.creature) return `/api/creature/${target.dataset.creature}/tooltip/`;
    if (target.dataset.spell) return `/api/spell/${target.dataset.spell}/tooltip/`;
    return null;
  }

  document.addEventListener("mouseover", (e) => {
    const target = e.target.closest("[data-item], [data-creature], [data-spell], [data-tooltip-url]");
    if (!target) return;
    const url = resolveUrl(target);
    if (!url) return;
    target.dataset.tooltipUrl = url;

    currentTarget = target;
    clearTimeout(hoverTimer);
    hoverTimer = setTimeout(() => {
      showTooltip(target, e.pageX, e.pageY);
    }, 150); // délai anti-scintillement
  });

  document.addEventListener("mousemove", (e) => {
    if (!tooltipEl || tooltipEl.style.display !== "block") return;
    positionTooltip(e.pageX, e.pageY);
  });

  document.addEventListener("mouseout", (e) => {
    const target = e.target.closest("[data-item], [data-creature], [data-spell], [data-tooltip-url]");
    if (!target) return;
    clearTimeout(hoverTimer);
    hideTooltip();
    currentTarget = null;
  });
})();
```

> 💡 Points importants :
> - **Cache client** (`Map`) : évite de re-fetch un tooltip déjà vu.
> - **Délai 150ms** : empêche l'affichage si l'utilisateur ne fait que passer la souris.
> - **Position intelligente** : bascule à gauche/haut si le tooltip déborderait de l'écran.
> - **Délégation d'événements** : on n'attache pas un listener par lien, mais un seul sur `document`. Essentiel pour une page avec 40 items.

**D2.3 — Inclure le script dans `base.html`**

Avant `</body>` :

```html
<script src="{% static 'js/tooltip.js' %}" defer></script>
```

> 💡 `defer` charge le script en parallèle du HTML mais l'exécute après le parsing. Résultat : pas de blocage du rendu.

**D2.4 — Tester**

Sur `/items/`, survole un objet. Après 150ms, un tooltip doit apparaître. ✅

---

## 🎯 Étape D3 — CSS stylisé aux couleurs de WoW

### Le quoi
Donner au tooltip l'apparence exacte de ceux du jeu.

### Le pourquoi
L'apparence est **la** raison pour laquelle Wowhead est devenu culte. Un tooltip bien stylisé inspire confiance et reconnaissance immédiate.

### Le comment

**D3.1 — Ajouter à `main.css`**

```css
/* Tooltip */
#wow-tooltip {
  position: absolute;
  z-index: 10000;
  pointer-events: none;
  max-width: 320px;
  font-family: "Segoe UI", sans-serif;
  font-size: 13px;
  color: #fff;
  filter: drop-shadow(0 4px 12px rgba(0, 0, 0, 0.8));
}

.wow-tooltip {
  background: linear-gradient(180deg, #0a0a0a 0%, #05051a 100%);
  border: 2px solid #1a1a2e;
  border-radius: 4px;
  padding: 10px 12px;
  line-height: 1.4;
}

.wow-tooltip-name {
  font-size: 14px;
  font-weight: bold;
  margin-bottom: 6px;
}

.wow-tooltip-line {
  margin: 2px 0;
  color: #ffffff;
}

.wow-tooltip-required {
  color: #ff6b6b;
}

.wow-tooltip-stat {
  color: #1eff00;
}

.wow-tooltip-flavor {
  color: #ffd100;
  font-style: italic;
  margin-top: 8px;
  padding-top: 6px;
  border-top: 1px solid #2a2a2a;
  font-size: 12px;
}
```

**D3.2 — Couleurs par qualité**

Le nom de l'item hérite déjà de la classe `q{{ item.Quality }}`. Les couleurs sont déjà définies dans `main.css` (rappel : `q0` à `q7`). ✅

**D3.3 — Point technique : `pointer-events: none`**

> ⚠️ Cette règle est **cruciale**. Sans elle, le tooltip capte les événements de souris et empêche le `mouseout` de se déclencher correctement sur le lien, ce qui provoque des clignotements. Avec `pointer-events: none`, le tooltip est "transparent" aux événements.

---

## 🎯 Étape D4 — Tooltips pour créatures et sorts

### Le quoi
Étendre le système aux PNJs et aux sorts.

### Le pourquoi
La cohérence de l'expérience utilisateur. Partout où on mentionne une entité, on doit pouvoir la survoler.

### Le comment

**D4.1 — Vues `creature_tooltip` et `spell_tooltip`**

Dans `api/views.py` :

```python
from creatures.models import CreatureTemplate
from spells.models import SpellTemplate


@cache_page(60 * 60 * 24)
def creature_tooltip(request, entry):
    creature = get_object_or_404(CreatureTemplate, entry=entry)
    return render(request, "api/tooltips/creature.html", {"creature": creature})


@cache_page(60 * 60 * 24)
def spell_tooltip(request, id):
    spell = get_object_or_404(SpellTemplate, id=id)
    return render(request, "api/tooltips/spell.html", {"spell": spell})
```

**D4.2 — Template créature**

`api/templates/api/tooltips/creature.html` :

```html
<div class="wow-tooltip">
  <div class="wow-tooltip-name">{{ creature.name }}</div>
  {% if creature.subname %}
    <div class="wow-tooltip-line wow-tooltip-subname">{{ creature.subname }}</div>
  {% endif %}
  <div class="wow-tooltip-line">Niveau {{ creature.minlevel }} - {{ creature.maxlevel }}</div>
</div>
```

**D4.3 — Template sort**

`api/templates/api/tooltips/spell.html` :

```html
<div class="wow-tooltip">
  <div class="wow-tooltip-name">{{ spell.name }}</div>
  {% if spell.description %}
    <div class="wow-tooltip-line">{{ spell.description|truncatewords:30 }}</div>
  {% endif %}
</div>
```

**D4.4 — Marquer les liens dans les templates**

Partout où tu affiches un nom de créature, ajoute `data-creature="{{ creature.entry }}"`. Idem pour les sorts avec `data-spell="{{ spell.id }}"`.

---

## 🎯 Étape D5 — Mise en cache

### Le quoi
Optimiser les performances.

### Le pourquoi
Un tooltip populaire peut être demandé des milliers de fois par heure. Sans cache, chaque survol déclenche une requête SQL + un rendu de template.

### Le comment

**D5.1 — Cache HTTP (déjà en place)**

`@cache_page(86400)` est déjà dans les vues. Ça ajoute :

```
Cache-Control: max-age=86400
```

Cloudflare et le navigateur vont donc mettre en cache. ✅

**D5.2 — Cache serveur Django (optionnel mais recommandé)**

Dans `base.py` :

```python
CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.redis.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",
    }
}
```

Nécessite `redis-server`. Pour le développement local, on peut utiliser :

```python
CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.locmem.LocMemCache",
    }
}
```

**D5.3 — Préchargement client**

En bonus, on peut précharger certains tooltips avec `prefetch` sur la page :

```javascript
// Dans tooltip.js, au chargement
document.querySelectorAll('[data-item]').forEach(el => {
  const url = `/api/item/${el.dataset.item}/tooltip/`;
  fetch(url).catch(() => {}); // précharge en arrière-plan
});
```

> 💡 À utiliser avec parcimonie. Sur une page avec 40 items, ça fait 40 requêtes. Utilise plutôt `<link rel="prefetch">` pour les 5 premiers.

---

## 🎯 Étape F1 — Comprendre la chaîne de données

### Le quoi
Comprendre comment obtenir l'icône d'un item.

### Le pourquoi
C'est le point le plus délicat du projet. Contrairement à ce qu'on pourrait penser, `item_template` ne contient **pas** directement le nom de l'icône. Il y a une chaîne d'indirection.

### Le comment

**F1.1 — La chaîne complète**

```
item_template.displayid (ex: 12345)
    ↓
ItemDisplayInfo.dbc[id=12345].icon_name (ex: "INV_Sword_01")
    ↓
Interface\Icons\INV_Sword_01.blp (fichier dans les MPQ du client)
```

Donc pour afficher une icône, il faut :
1. La colonne `displayid` de l'item
2. Une table `ItemDisplayInfo.dbc` parsée
3. Les fichiers `.blp` extraits (ou déjà extraits)
4. Une conversion BLP → PNG (ou servir directement le PNG)

**F1.2 — Décision stratégique**

Deux options :

| Option | Avantages | Inconvénients |
|--------|-----------|---------------|
| **A. Servir les icônes via un CDN externe** (Wowhead, etc.) | Instantané, 0 effort | Dépendance externe, moins propre |
| **B. Extraire les icônes soi-même** | Indépendance totale, légal pour un serveur privé | Nécessite le client WoW 3.3.5a |

Pour le développement, on commence par **A**, puis on migrera vers **B**. C'est ce que je te propose.

---

## 🎯 Étape F2 — Récupérer les icônes

### Le quoi
Se procurer un ensemble d'icônes PNG.

### Le pourquoi
Il y a environ 5000 icônes différentes dans WoW 3.3.5a. Les héberger toi-même te rend indépendant.

### Le comment

**F2.1 — Option A : Wowhead CDN (développement uniquement)**

Wowhead expose les icônes via :

```
https://wow.zamimg.com/images/wow/icons/large/inv_sword_01.jpg
```

Le nom est **en minuscules** et sans l'extension `.blp`. On peut donc générer l'URL directement depuis `icon_name`.

> ⚠️ Utiliser le CDN de Wowhead en production est **déconseillé** (dépendance, risque de blocage, et flou juridique). C'est acceptable pour le prototypage.

**F2.2 — Option B : Extraction personnelle**

Si tu as le client WoW 3.3.5a installé :

1. **Extraire les BLP** avec `wow.export` ou `CascView` (pour 3.3.5, il faut plutôt `MPQEditor` ou `Ladik's MPQ Editor`)
2. Les fichiers sont dans `Interface\Icons\*.blp`
3. **Convertir BLP → PNG** avec `BLPConverter` ou en Python avec la lib `Pillow` + `blp` plugin

En pratique, la solution la plus rapide : télécharger un pack pré-extrait. Il existe plusieurs dépôts GitHub qui hébergent les ~5000 icônes en PNG (cherche `wow icons png` ou `wow 3.3.5 icons`).

**F2.3 — Organisation recommandée**

```
static/icons/
├── inv_sword_01.jpg
├── inv_sword_02.jpg
├── spell_fire_01.jpg
└── ...
```

Nom en minuscules, extension `.jpg` (c'est ce que fait Wowhead, et c'est plus léger que PNG pour des icônes 64×64).

Pour cette session, on va utiliser **le CDN Wowhead** en développement, avec une variable de config pour basculer plus tard.

---

## 🎯 Étape F3 — Parser `ItemDisplayInfo.dbc`

### Le quoi
Extraire le mapping `displayid → icon_name` depuis le fichier DBC.

### Le pourquoi
Sans ce mapping, on ne peut pas savoir quelle icône correspond à quel item. C'est la pièce manquante.

### Le comment

**F3.1 — Où trouver le fichier**

Le fichier est dans le client WoW :

```
Data/patch-3.MPQ  →  DBFilesClient/ItemDisplayInfo.dbc
```

Ou dans les patches :

```
Data/frFR/patch-frFR-3.MPQ  →  DBFilesClient/ItemDisplayInfo.dbc
```

Il faut l'extraire avec MPQEditor.

**F3.2 — Structure du DBC**

`ItemDisplayInfo.dbc` a la structure suivante (simplifiée) :

| Champ | Index | Contenu |
|-------|-------|---------|
| ID | 0 | = displayid |
| ModelName1 | 1 | Nom du modèle 3D |
| ModelName2 | 2 | ... |
| ModelTexture1 | 3 | ... |
| Icon1 | 4 | **Nom de l'icône** (ex: "INV_Sword_01") |
| Icon2 | 5 | ... |
| GeosetGroup1 | 6 | ... |

C'est le champ **Icon1** (index 4) qui nous intéresse.

**F3.3 — Installer `dbcraft`**

```bash
pip install dbcraft
```

Ajoute-le à `requirements.txt`.

**F3.4 — Créer une commande de management**

```bash
mkdir -p dbc/management/commands
touch dbc/management/__init__.py
touch dbc/management/commands/__init__.py
```

Puis `dbc/management/commands/import_item_display_info.py` :

```python
import struct
from pathlib import Path
from django.core.management.base import BaseCommand
from django.db import transaction
from dbc.models import DBCItemDisplayInfo


class Command(BaseCommand):
    help = "Importe ItemDisplayInfo.dbc dans la base Django"

    def add_arguments(self, parser):
        parser.add_argument("dbc_path", type=str)

    @transaction.atomic
    def handle(self, *args, **options):
        path = Path(options["dbc_path"])
        if not path.exists():
            self.stderr.write(f"Fichier introuvable : {path}")
            return

        data = path.read_bytes()

        # En-tête DBC
        magic, record_count, field_count, record_size, string_size = struct.unpack(
            "<4sIIII", data[:20]
        )
        if magic != b"WDBC":
            self.stderr.write("Fichier DBC invalide")
            return

        self.stdout.write(
            f"Records: {record_count}, champs: {field_count}, taille record: {record_size}"
        )

        records_start = 20
        strings_start = records_start + record_count * record_size

        def read_string(offset):
            if offset == 0:
                return ""
            end = data.index(b"\x00", strings_start + offset)
            return data[strings_start + offset : end].decode("utf-8", errors="replace")

        DBCItemDisplayInfo.objects.all().delete()

        objs = []
        for i in range(record_count):
            offset = records_start + i * record_size
            fields = struct.unpack(f"<{field_count}I", data[offset : offset + record_size])
            display_id = fields[0]
            icon_name = read_string(fields[4]) if field_count > 4 else ""

            objs.append(DBCItemDisplayInfo(display_id=display_id, icon_name=icon_name))

        DBCItemDisplayInfo.objects.bulk_create(objs, batch_size=1000)
        self.stdout.write(self.style.SUCCESS(f"{len(objs)} entrées importées."))
```

> 💡 **Points clés** :
> - L'en-tête DBC est toujours `WDBC` suivi de 4 entiers (record_count, field_count, record_size, string_size).
> - Les **strings** sont stockées dans un bloc séparé, après les records. Le champ DBC contient un **offset** dans ce bloc.
> - `bulk_create` insère 1000 entrées par requête — indispensable pour ~100 000 records.

**F3.5 — Lancer l'import**

```bash
python manage.py import_item_display_info /chemin/vers/ItemDisplayInfo.dbc
```

---

## 🎯 Étape F4 — Modèle Django de mapping

### Le quoi
Stocker le mapping dans la base Django.

### Le pourquoi
Une fois importé, on joint ce mapping avec `item_template` via l'ORM.

### Le comment

**F4.1 — `dbc/models.py`**

```python
from django.db import models


class DBCItemDisplayInfo(models.Model):
    display_id = models.IntegerField(primary_key=True)
    icon_name = models.CharField(max_length=100, db_index=True)

    class Meta:
        db_table = "dbc_item_display_info"
        ordering = ["display_id"]

    def __str__(self):
        return f"{self.display_id}: {self.icon_name}"
```

> ⚠️ Contrairement aux modèles de la base `world`, ce modèle-ci est **géré par Django**. Il faut donc retirer `"dbc"` de `wow_apps` dans le routeur. Ou plus simple : renomme l'app en `dbc_data` pour éviter l'ambiguïté, et laisse `dbc` comme app "world".

En fait, réfléchissons. La convention :
- Modèles de la base `world` → app `dbc` (lecture seule)
- Mapping parsé → app séparée `dbc_data` (géré par Django)

Pour simplifier, on garde `dbc` pour les DBC parsés (gérés par Django), et on met les modèles lus depuis `world` dans leurs apps respectives (déjà fait pour items/creatures/spells). On retire donc `"dbc"` de `wow_apps`.

**F4.2 — Ajuster le routeur**

```python
class WoWRouter:
    wow_apps = {"items", "creatures", "spells", "quests"}
    django_apps = {"core", "accounts", "api", "dbc", "admin", "auth", "contenttypes", "sessions"}
```

**F4.3 — Migration**

```bash
python manage.py makemigrations dbc
python manage.py migrate
```

**F4.4 — Helper `icon_url`**

On veut une fonction réutilisable pour générer l'URL de l'icône d'un item.

`items/utils.py` :

```python
from functools import lru_cache
from django.conf import settings
from dbc.models import DBCItemDisplayInfo


@lru_cache(maxsize=10000)
def icon_name_for_display_id(display_id):
    try:
        return DBCItemDisplayInfo.objects.get(display_id=display_id).icon_name
    except DBCItemDisplayInfo.DoesNotExist:
        return None


def item_icon_url(item):
    name = icon_name_for_display_id(item.displayid)
    if not name:
        return settings.ICON_FALLBACK_URL
    return f"{settings.ICON_BASE_URL}/{name.lower()}.jpg"
```

> 💡 `lru_cache` évite de faire une requête SQL pour le même `display_id` répété sur une page (fréquent : un même modèle 3D peut être partagé par plusieurs items).

**F4.5 — Config dans `base.py`**

```python
# En dev : CDN Wowhead
ICON_BASE_URL = "https://wow.zamimg.com/images/wow/icons/medium"
ICON_FALLBACK_URL = "https://wow.zamimg.com/images/wow/icons/medium/inv_misc_questionmark.jpg"
```

> 💡 En production, tu changeras ces URLs par `/static/icons` (local).

---

## 🎯 Étape F5 — Servir les icônes (migration vers local)

### Le quoi
Passer du CDN à tes propres icônes.

### Le pourquoi
Indépendance, performance (cache Cloudflare), et propreté.

### Le comment

**F5.1 — Récupérer les icônes**

Deux méthodes :

**Méthode A — Télécharger via Wowhead (script)**

```python
# dbc/management/commands/download_icons.py
import os
import requests
from pathlib import Path
from django.conf import settings
from django.core.management.base import BaseCommand
from dbc.models import DBCItemDisplayInfo


class Command(BaseCommand):
    def handle(self, *args, **options):
        out = Path(settings.BASE_DIR) / "static" / "icons"
        out.mkdir(parents=True, exist_ok=True)

        names = set(
            DBCItemDisplayInfo.objects.values_list("icon_name", flat=True)
        )
        self.stdout.write(f"{len(names)} icônes à télécharger.")

        for i, name in enumerate(names):
            if not name:
                continue
            slug = name.lower()
            dest = out / f"{slug}.jpg"
            if dest.exists():
                continue
            url = f"https://wow.zamimg.com/images/wow/icons/large/{slug}.jpg"
            try:
                r = requests.get(url, timeout=10)
                if r.status_code == 200:
                    dest.write_bytes(r.content)
                if i % 100 == 0:
                    self.stdout.write(f"{i}/{len(names)}")
            except Exception as e:
                self.stderr.write(f"{slug}: {e}")
```

> ⚠️ Ce script télécharge ~5000 fichiers. Utilise-le **une fois** en local, puis commit le dossier. Ou stocke-le dans un bucket S3.

**Méthode B — Extraire depuis le client WoW**

Si tu as le client 3.3.5a, utilise **BLPConverter** :
```bash
BLPConverter Interface/Icons/*.blp -o static/icons/
```

Puis convertis en JPG avec ImageMagick :
```bash
mogrify -format jpg -quality 85 static/icons/*.png
```

**F5.2 — Basculer la config**

```python
ICON_BASE_URL = "/static/icons"
ICON_FALLBACK_URL = "/static/icons/inv_misc_questionmark.jpg"
```

---

## 🎯 Étape F6 — Intégration dans les templates

### Le quoi
Afficher les icônes dans les listes et les tooltips.

### Le pourquoi
C'est ce qui rend le site immédiatement reconnaissable et pro.

### Le comment

**F6.1 — Filtre de template pour l'icône**

`items/templatetags/item_extras.py` :

```bash
mkdir -p items/templatetags
touch items/templatetags/__init__.py
```

```python
from django import template
from items.utils import item_icon_url

register = template.Library()


@register.filter
def item_icon(item):
    return item_icon_url(item)
```

**F6.2 — Utilisation dans `item_list.html`**

```html
{% load item_extras %}

<div class="card">
  <a href="{% url 'items:detail' entry=item.entry %}"
     class="q{{ item.Quality }}"
     data-item="{{ item.entry }}">
    <img src="{{ item|item_icon }}" alt="" class="item-icon" loading="lazy">
    <span>{{ item.name }}</span>
  </a>
  <div class="item-meta">Niveau {{ item.ItemLevel }}</div>
</div>
```

> 💡 `loading="lazy"` : le navigateur ne charge l'image que quand elle entre dans le viewport. Essentiel pour une liste de 40 items.

**F6.3 — CSS**

```css
.card a {
  display: flex;
  align-items: center;
  gap: 10px;
}
.item-icon {
  width: 36px;
  height: 36px;
  border: 1px solid #3a3a3a;
  border-radius: 3px;
  flex-shrink: 0;
  background: #1a1a1a;
}
```

> 💡 Une bordure grise autour de l'icône évite l'effet "flottant" et rappelle le style WoW.

**F6.4 — Dans le tooltip**

Modifie `api/templates/api/tooltips/item.html` :

```html
{% load item_extras %}
<div class="wow-tooltip">
  <div class="wow-tooltip-header">
    <img src="{{ item|item_icon }}" alt="" class="wow-tooltip-icon">
    <div class="wow-tooltip-name q{{ item.Quality }}">{{ item.name }}</div>
  </div>
  <!-- ... reste inchangé -->
</div>
```

Et le CSS :

```css
.wow-tooltip-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 8px;
}
.wow-tooltip-icon {
  width: 44px;
  height: 44px;
  border: 2px solid #3a3a3a;
  border-radius: 3px;
}
```

**F6.5 — Icônes pour créatures et sorts (bonus)**

Si tu as parsé `SpellIcon.dbc` (mapping `spellIconId → icon_name`), tu peux faire pareil pour les sorts. La chaîne est :
```
spell_template.SpellIconID → SpellIcon.dbc → icon_name
```

Pour les créatures, c'est plus complexe (modèles 3D, pas d'icône 2D). On peut utiliser un placeholder générique ou parser `CreatureDisplayInfo.dbc`.

---

## 📊 Récapitulatif de la session

| Étape | Contenu | Résultat |
|-------|---------|----------|
| D1 | Endpoint API tooltip | HTML fragmentaire |
| D2 | JavaScript de survol | Infobulles dynamiques |
| D3 | CSS stylisé WoW | Reconnaissable |
| D4 | Créatures + sorts | Cohérence |
| D5 | Cache HTTP + Django | Performance |
| F1–F2 | Chaîne de données + sources | Compréhension |
| F3 | Parser `ItemDisplayInfo.dbc` | Mapping disponible |
| F4 | Modèle + helper `lru_cache` | Intégration ORM |
| F5 | Téléchargement icônes | Autonomie |
| F6 | Templates + CSS | Icônes visibles |

---

On continue l'aventure. Ce segment va transformer notre base de données consultable en un outil d'analyse complet.

Voici le plan pour cette session :

1.  **Option E** — Recherche avancée avec Meilisearch : pour une recherche rapide, tolérante aux fautes et pertinente.
2.  **Option H** — Calculateur de talents : pour reproduire la fonctionnalité emblématique de wotlkdb.
3.  **Option I** — Cartes interactives : pour explorer les zones et y afficher les PNJs et objets.

---

## 🎯 Option E — Recherche avancée avec Meilisearch

### Le quoi
Remplacer la recherche basique (`icontains`) par un moteur dédié, Meilisearch, qui offrira une recherche instantanée, tolérante aux fautes de frappe, et capable de trier par pertinence.

### Le pourquoi
La recherche `icontains` est simple mais limitée. Elle ne gère pas les fautes ("Epée" vs "Epee"), n'offre pas de tri par pertinence, et devient lente sur de gros volumes. Meilisearch résout ces problèmes et est bien plus simple à configurer qu'Elasticsearch.

### Le comment

**E1. Lancer Meilisearch**
Le plus simple est d'utiliser Docker.

```bash
docker run -it --rm \
  -p 7700:7700 \
  -v $(pwd)/meili_data:/meili_data \
  getmeili/meilisearch:v1.10 \
  meilisearch --master-key=une_cle_secrete_a_changer
```

> 💡 Note bien ta `master-key`. Elle servira à sécuriser l'accès à l'instance.

**E2. Installer le client Python**
On utilise le SDK officiel.

```bash
pip install meilisearch
```

**E3. Configurer Django**
Dans `settings/base.py`, ajoute la configuration.

```python
MEILISEARCH = {
    "HOST": "http://localhost:7700",
    "MASTER_KEY": "une_cle_secrete_a_changer", # Utilise une variable d'environnement en prod
}
```

**E4. Définir les index**
On va créer un fichier `search.py` dans une nouvelle app `search` (ou dans `core`). L'idée est de définir quel modèle est indexé et quels champs sont recherchables.

```python
# core/search.py
import meilisearch
from django.conf import settings

client = meilisearch.Client(settings.MEILISEARCH["HOST"], settings.MEILISEARCH["MASTER_KEY"])

def index_items():
    index = client.index("items")
    # On configure les champs sur lesquels Meilisearch peut chercher
    index.update_searchable_attributes(["name", "description"])
    index.update_filterable_attributes(["Quality", "ItemLevel", "RequiredLevel"])
    index.update_sortable_attributes(["ItemLevel", "name"])
    
    from items.models import ItemTemplate
    # On prépare les documents à indexer
    documents = [
        {
            "id": item.entry,
            "name": item.name,
            "description": item.description or "",
            "Quality": item.Quality,
            "ItemLevel": item.ItemLevel,
            "RequiredLevel": item.RequiredLevel,
            "type": "item",
        }
        for item in ItemTemplate.objects.all().iterator()
    ]
    
    # On envoie les documents par paquets
    index.add_documents_in_batches(documents, batch_size=1000)

def index_spells():
    # ... même logique pour les sorts
    pass
```

> 💡 L'utilisation de `.iterator()` est cruciale : elle évite de charger les 60 000 items en mémoire d'un coup.

**E5. Créer une commande de management**
Pour lancer l'indexation depuis le terminal.

```python
# core/management/commands/reindex.py
from django.core.management.base import BaseCommand
from core.search import index_items, index_spells, index_creatures

class Command(BaseCommand):
    help = "Réindexe toutes les données dans Meilisearch"

    def handle(self, *args, **options):
        self.stdout.write("Indexation des objets...")
        index_items()
        self.stdout.write("Indexation des sorts...")
        index_spells()
        self.stdout.write("Indexation des PNJs...")
        index_creatures()
        self.stdout.write(self.style.SUCCESS("Indexation terminée."))
```

**E6. Mettre à jour la vue de recherche**
On remplace la vue actuelle pour interroger Meilisearch.

```python
# core/views.py
from core.search import client

def search(request):
    q = request.GET.get("q", "").strip()
    results = {"items": [], "creatures": [], "spells": []}
    
    if q:
        # Recherche fédérée : on interroge tous les index en même temps
        search_results = client.multi_search([
            {"indexUid": "items", "q": q, "limit": 20},
            {"indexUid": "creatures", "q": q, "limit": 20},
            {"indexUid": "spells", "q": q, "limit": 20},
        ])
        
        for res in search_results["results"]:
            if res["indexUid"] == "items":
                results["items"] = res["hits"]
            elif res["indexUid"] == "creatures":
                results["creatures"] = res["hits"]
            elif res["indexUid"] == "spells":
                results["spells"] = res["hits"]

    # On doit ensuite récupérer les objets Django à partir des IDs retournés par Meilisearch
    # ... (code de récupération des objets)
```

> 💡 **Point clé** : Meilisearch retourne des documents JSON, pas des objets Django. Il faut ensuite faire une requête `ItemTemplate.objects.filter(entry__in=[...])` pour récupérer les instances complètes.

---

## 🎯 Option H — Calculateur de talents

### Le quoi
Créer un outil interactif qui permet de simuler la répartition des points de talent d'un personnage, comme dans le jeu.

### Le pourquoi
C'est une fonctionnalité phare de wotlkdb. Pour la réaliser, nous devons parser les fichiers DBC `Talent.dbc` et `TalentTab.dbc`.

### Le comment

**H1. Parser `Talent.dbc` et `TalentTab.dbc`**
On utilise la même technique que pour les icônes d'objets. La structure de `Talent.dbc` est bien documentée.

- **`Talent.dbc`** : Chaque ligne représente un talent unique. Les colonnes importantes sont `TalentID`, `TalentTab` (l'onglet auquel il appartient), `Row` et `Col` (sa position dans la grille), et `RankID1` à `RankID5` (les IDs des sorts pour chaque rang).
- **`TalentTab.dbc`** : Chaque ligne décrit un onglet de talent (ex: "Feu", "Givre"). Les colonnes utiles sont `ID`, `Name`, `SpellIcon`, `RaceMask` et `ClassMask`.

On peut créer des modèles Django pour les stocker.

```python
# talents/models.py
from django.db import models

class TalentTab(models.Model):
    id = models.IntegerField(primary_key=True)
    name = models.CharField(max_length=100)
    icon = models.CharField(max_length=100)
    class_mask = models.IntegerField()
    # ...

class Talent(models.Model):
    id = models.IntegerField(primary_key=True)
    tab = models.ForeignKey(TalentTab, on_delete=models.DO_NOTHING)
    row = models.IntegerField()
    col = models.IntegerField()
    rank_ids = models.JSONField() # ex: [12345, 12346, 12347, 12348, 12349]
    # ...
```

**H2. Créer l'interface du calculateur**
On crée une vue et un template dédiés.

```python
# talents/views.py
def calculator(request, class_id):
    tabs = TalentTab.objects.filter(class_mask__bitand=class_id) # Utilise les opérations bit à bit
    # ...
```

Le template affichera une grille pour chaque onglet. Chaque cellule de la grille est un talent, avec son icône et les prérequis.

**H3. La logique de JavaScript**
Le cœur du calculateur est en JavaScript. Il doit gérer :
- **La répartition des points** : quand on clique sur un talent, on incrémente son rang et on décrémente le total de points disponibles.
- **Les prérequis** : un talent ne peut être amélioré que si le total de points investis dans son onglet atteint un certain seuil (généralement 5, 10, 15...).
- **Les dépendances** : certains talents nécessitent un autre talent à un certain rang pour être débloqués.
- **L'état de l'URL** : pour permettre le partage d'un build, on peut encoder la répartition des points dans l'URL (ex: `?talents=000...`).

---

## 🎯 Option I — Cartes interactives

### Le quoi
Afficher une carte d'une zone (ex: Durotar) avec des marqueurs cliquables pour chaque PNJ, objet ou quête.

### Le pourquoi
C'est une fonctionnalité visuelle très utile pour localiser les ressources. La difficulté réside dans la conversion des coordonnées du monde (celles de la base de données) en coordonnées sur une image de carte.

### Le comment

**I1. Récupérer les coordonnées des objets**
La base `acore_world` contient les coordonnées 3D du monde pour chaque objet et PNJ. La table `creature` contient les colonnes `position_x`, `position_y` et `map`.

```python
# maps/models.py
class CreatureSpawn(models.Model):
    guid = models.IntegerField(primary_key=True)
    id1 = models.IntegerField()
    map = models.IntegerField()
    position_x = models.FloatField()
    position_y = models.FloatField()
    # ...
    
    class Meta:
        managed = False
        db_table = "creature"
        # ...
```

**I2. Convertir les coordonnées**
Pour placer un point sur une image, il faut transformer les coordonnées du monde (`position_x`, `position_y`) en coordonnées de pixel (0-100%) sur l'image.

La conversion se fait grâce aux données des fichiers DBC `WorldMapArea.dbc` et `Map.dbc`. `WorldMapArea.dbc` définit les bornes (`x1`, `x2`, `y1`, `y2`) d'une zone sur la carte du monde.

La formule est la suivante :
```
pixel_x = (world_x - x1) / (x2 - x1) * 100
pixel_y = (y2 - world_y) / (y2 - y1) * 100
```

> 💡 L'axe Y est inversé car dans le jeu, l'origine est en bas à gauche, alors que sur une image, elle est en haut à gauche.

**I3. Afficher la carte**
Le frontend utilisera une bibliothèque JavaScript comme **Leaflet.js**. C'est une bibliothèque open source, légère et parfaite pour ça. On lui fournit une image de la carte (qu'on aura extraite des MPQ du jeu) et on place les marqueurs aux coordonnées calculées.

```javascript
// maps/static/js/map.js
const map = L.map('map', {
    crs: L.CRS.Simple, // On utilise un système de coordonnées simple, pas géographique
    minZoom: -2,
});

const bounds = [[0, 0], [100, 100]]; // On suppose une image de 100x100 unités
L.imageOverlay('/static/maps/durotar.jpg', bounds).addTo(map);
map.fitBounds(bounds);

// On récupère les points depuis une API Django
fetch('/api/maps/1/spawns/')
    .then(response => response.json())
    .then(spawns => {
        spawns.forEach(spawn => {
            const marker = L.marker([spawn.y, spawn.x]).addTo(map); // Utiliser les coordonnées converties
            marker.bindPopup(`<b>${spawn.name}</b><br>Niveau ${spawn.level}`);
        });
    });
```

---

## 📊 Récapitulatif de la session

| Étape | Contenu | Résultat |
|-------|---------|----------|
| E | Recherche Meilisearch | Recherche rapide, tolérante, pertinente |
| H | Calculateur de talents | Outil d'analyse complet |
| I | Cartes interactives | Localisation visuelle des données |

---


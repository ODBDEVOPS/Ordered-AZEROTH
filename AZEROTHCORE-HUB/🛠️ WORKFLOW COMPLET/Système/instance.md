# Guide complet : Créer une instance personnalisée dans AzerothCore 3.3.5

## Introduction

Ce guide détaille étape par étape la création d'une **instance personnalisée** (donjon ou raid) pour AzerothCore 3.3.5. Contrairement aux guides précédents (enchantements, métiers, réputations, classes), la création d'une instance est une opération **multi-niveaux** qui touche à la fois aux fichiers DBC côté client, aux tables de base de données côté serveur, et à la **programmation C++** pour la logique des boss et des événements.

La difficulté principale réside dans le fait que le noyau AzerothCore gère les instances via la classe `InstanceScript` et la fonction `GetInstanceScript()`. Vous devrez donc écrire et compiler un script C++ pour que votre instance soit pleinement fonctionnelle.

> **⚠️ Avertissement** : La création d'une instance personnalisée est une opération **avancée**. Elle nécessite des modifications des fichiers DBC, de la base de données, **et une recompilation du noyau** (core). Sauvegardez toujours vos fichiers avant modification.

---

## Prérequis

- **Éditeur DBC** : MyDBCEditor, DBC Editor, ou équivalent.
- **Client de base de données** : HeidiSQL, MySQL Workbench, etc.
- **Fichiers DBC de référence** : `Map.dbc`, `AreaTable.dbc`, `DungeonEncounter.dbc`, `LfgDungeon.dbc`, `LoadingScreens.dbc`.
- **Outil MPQ** : Ladik's MPQ Editor.
- **Environnement de développement C++** : Visual Studio, CMake, et les sources d'AzerothCore.
- **Outil de création de cartes** : Noggit (pour créer la géométrie du terrain) ou un logiciel 3D équivalent.
- **Modèles 3D** : Fichiers `.wmo` (bâtiments) et `.m2` (objets) pour le contenu de l'instance.
- **Accès serveur** : Base de données `world` et possibilité de **recompiler le noyau**.

---

## Vue d'ensemble du processus

La création d'une instance personnalisée implique **sept éléments interconnectés** :

1. **Map.dbc** (client) — Définit le « monde » de l'instance (type, nom, limites de joueurs).
2. **Fichiers de carte** (client) — Les fichiers `.map`, `.vmap`, `.mmap` générés à partir de la géométrie de votre instance.
3. **AreaTable.dbc** (client) — Définit la zone de l'instance (nom, drapeaux).
4. **instance_template** (serveur) — Définit les paramètres de l'instance (script, monture autorisée).
5. **instance_encounters** + **DungeonEncounter.dbc** (serveur + client) — Définissent les boss et les rencontres (utilisé par le LFG).
6. **creature_template** + **creature** (serveur) — Définissent et placent les créatures (boss, trash mobs).
7. **InstanceScript** (C++) — Programme la logique de l'instance (événements, portes, boss).

### Schéma des relations

```
┌─────────────────────────┐
│       Map.dbc            │  ← Définit le monde de l'instance
│  ID: 1000               │
│  Type: 1 (Party)        │
│  MaxPlayers: 5          │
└───────────┬─────────────┘
            │  référencé par
            ▼
┌─────────────────────────┐
│   instance_template      │  ← Paramètres serveur
│  map: 1000              │
│  script: "instance_nexus"│
│  allowMount: 0          │
└───────────┬─────────────┘
            │  lié à
            ▼
┌─────────────────────────┐
│  InstanceScript (C++)    │  ← Logique de l'instance
│  class instance_nexus   │
│  : public InstanceScript│
└───────────┬─────────────┘
            │  gère
            ▼
┌─────────────────────────┐
│  creature + creature_template│  ← Créatures de l'instance
│  entry: 90001 (Boss)    │
│  map: 1000              │
└─────────────────────────┘
```

---

## Étape 1 : Définir le monde dans Map.dbc

Le fichier `Map.dbc` contient toutes les cartes du jeu. C'est ici que vous définissez le « monde » de votre instance.

### 1.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique de la carte. |
| 2 | **InternalName** | String | Nom interne (référence au dossier `World\Map\`). |
| 3 | **Flags** | Integer | Drapeaux de la carte (`0x100` = changement de difficulté). |
| 4 | **Type** | Integer | `0` = Aucun, `1` = Groupe, `2` = Raid, `3` = PvP, `4` = Arène. |
| 5 | **IsBattleground** | Integer | `1` = Champ de bataille, `0` = Non. |
| 6-22 | **Name** | Loc | Nom de la carte (affiché sur la carte du monde). |
| 23 | **AreaTableID** | iRefID | Référence à `AreaTable.dbc`. |
| 24-40 | **MapDescriptionA** | Loc | Description (Alliance). |
| 41-57 | **MapDescriptionH** | Loc | Description (Horde). |
| 58 | **LoadingScreen** | iRefID | Référence à `LoadingScreens.dbc` (écran de chargement). |
| 59 | **BGMapIconScale** | Float | Échelle de l'icône (BG). |
| 60 | **GhostEntranceMap** | iRefID | Carte d'entrée fantôme. |
| 61 | **GhostEntranceX** | Float | Coordonnée X de l'entrée fantôme. |
| 62 | **GhostEntranceY** | Float | Coordonnée Y de l'entrée fantôme. |
| 63 | **TimeOfDayOverride** | Integer | `-1` par défaut. |
| 64 | **Expansion** | Integer | `0` = Classic, `1` = BC, `2` = WotLK. |
| 65 | **RaidOffset** | Integer | Décalage de raid. |
| 66 | **MaxPlayers** | Integer | **Nombre maximum de joueurs** (ex: 5 pour un donjon, 25 pour un raid). |



### 1.2. Exemple concret

**Instance « Nexus de l'Éternité »**

- **ID** : `1000` (doit être unique, les IDs 1-999 sont utilisés)
- **InternalName** : `NexusEternity`
- **Flags** : `0`
- **Type** : `1` (Groupe)
- **IsBattleground** : `0`
- **Name** : `Nexus de l'Éternité`
- **AreaTableID** : `5000` (doit être créé dans `AreaTable.dbc`)
- **LoadingScreen** : `1` (ou un écran existant)
- **Expansion** : `2` (WotLK)
- **MaxPlayers** : `5`

> **📝 Note** : Le champ **MaxPlayers** détermine la taille maximale du groupe. Pour un raid, utilisez `10`, `25` ou `40`. Pour un donjon, `5`.

---

## Étape 2 : Créer la géométrie de l'instance (fichiers de carte)

Cette étape est **la plus complexe** car elle nécessite la création de la géométrie 3D de votre instance. Vous devez :

1. **Créer le terrain** avec Noggit ou un logiciel 3D.
2. **Placer les bâtiments** (fichiers `.wmo`).
3. **Placer les objets** (fichiers `.m2`).
4. **Générer les fichiers de carte** (`.map`, `.vmap`, `.mmap`) avec les outils d'extraction d'AzerothCore.

### 2.1. Structure des dossiers

```
World/
└── Map/
    └── NexusEternity/
        ├── NexusEternity.wdt
        ├── NexusEternity_00_00.adt
        └── ...
```

> **📝 Note** : Les fichiers `.map`, `.vmap` et `.mmap` sont générés côté serveur à partir des fichiers du client. Vous devez extraire ces fichiers avec `mapextractor`, `vmap4extractor` et `mmaps_generator`.

---

## Étape 3 : Définir la zone dans AreaTable.dbc

Le fichier `AreaTable.dbc` définit les zones du jeu. Créez une nouvelle entrée pour votre instance.

### 3.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique de la zone. |
| 2 | **ContinentID** | iRefID | Référence à `Map.dbc` (la carte à laquelle appartient la zone). |
| 3 | **ParentAreaID** | iRefID | Zone parente (`0` = aucune). |
| 4 | **AreaBit** | Integer | Bit de zone. |
| 5 | **Flags** | Integer | Drapeaux de zone (`0x02` = instance). |
| 6 | **SoundProviderPref** | Integer | Préférence de son. |
| 7 | **SoundProviderPrefUnderwater** | Integer | Son sous-marin. |
| 8 | **AmbienceID** | Integer | Ambiance. |
| 9 | **ZoneMusic** | Integer | Musique de zone. |
| 10 | **IntroSound** | Integer | Son d'introduction. |
| 11 | **ExplorationLevel** | Integer | Niveau d'exploration. |
| 12-28 | **AreaName** | Loc | Nom de la zone. |
| 29-45 | **AreaName_lang** | Loc | Nom de la zone (langues). |
| 46 | **FactionGroupMask** | Integer | Masque de faction. |
| 47-50 | **LiquidTypeID** | Integer | Type de liquide. |
| 51 | **MinElevation** | Float | Élévation minimale. |
| 52 | **AmbientMultiplier** | Float | Multiplicateur d'ambiance. |
| 53 | **Lightid** | Integer | ID de lumière. |

### 3.2. Exemple concret

**Zone « Nexus de l'Éternité »**

- **ID** : `5000`
- **ContinentID** : `1000` (référence à `Map.dbc`)
- **ParentAreaID** : `0`
- **Flags** : `0x02` (instance)
- **AreaName** : `Nexus de l'Éternité`

---

## Étape 4 : Configurer l'instance dans instance_template

La table `instance_template` définit les paramètres de l'instance côté serveur.

### 4.1. Structure de la table

| Champ | Type | Description |
|-------|------|-------------|
| **map** | INT UNSIGNED | ID du map (référence `Map.dbc`). |
| **parent** | BIGINT UNSIGNED | ID du map parent si sous-instance (`0` = aucun). |
| **script** | VARCHAR(128) | Nom du script d'instance (référence au script C++). |
| **allowMount** | TINYINT | `0` = pas de monture, `1` = monture autorisée. |



### 4.2. Exemple concret

```sql
INSERT INTO `instance_template` (
    `map`, `parent`, `script`, `allowMount`
) VALUES (
    1000, 0, 'instance_nexus_eternity', 0
);
```

| Champ | Valeur | Description |
|-------|--------|-------------|
| `map` | `1000` | ID du map. |
| `parent` | `0` | Pas de parent. |
| `script` | `instance_nexus_eternity` | Nom du script C++. |
| `allowMount` | `0` | Monture interdite. |

> **📝 Note** : Le champ **script** doit correspondre exactement au nom de votre classe `InstanceScript` dans le code C++.

---

## Étape 5 : Définir les rencontres dans DungeonEncounter.dbc et instance_encounters

### 5.1. DungeonEncounter.dbc (client)

Ce fichier définit les boss et les rencontres pour le système de recherche de groupe (LFG).

| Colonne | Champ | Description |
|---------|-------|-------------|
| 1 | **ID** | ID unique de la rencontre. |
| 2 | **MapID** | ID du map (référence `Map.dbc`). |
| 3 | **Difficulty** | Difficulté (`0` = Normal, `1` = Héroïque). |
| 4 | **OrderIndex** | Ordre de la rencontre. |
| 5 | **EncounterName** | Nom du boss. |
| 6 | **SpellIconID** | Icône du boss. |

### 5.2. instance_encounters (serveur)

Cette table lie les rencontres à des crédits (kill de créature ou sort).

| Champ | Type | Description |
|-------|------|-------------|
| **entry** | INT UNSIGNED | ID unique (référence `DungeonEncounter.dbc`). |
| **creditType** | TINYINT | `0` = Kill de créature, `1` = Sort. |
| **creditEntry** | INT UNSIGNED | ID de la créature (`creature_template.entry`) ou du sort. |
| **lastEncounterDungeon** | SMALLINT | `0` = pas le dernier boss, sinon ID LFG. |
| **comment** | VARCHAR(255) | Commentaire. |



### 5.3. Exemple concret

**DungeonEncounter.dbc** :

| ID | MapID | Difficulty | OrderIndex | EncounterName |
|----|-------|------------|------------|---------------|
| 1000 | 1000 | 0 | 0 | Gardien des Arcanes |
| 1001 | 1000 | 0 | 1 | Seigneur du Néant |

**instance_encounters** :

```sql
INSERT INTO `instance_encounters` (
    `entry`, `creditType`, `creditEntry`, `lastEncounterDungeon`, `comment`
) VALUES
(1000, 0, 90001, 0, 'Gardien des Arcanes'),
(1001, 0, 90002, 0, 'Seigneur du Néant');
```

---

## Étape 6 : Créer les créatures de l'instance

### 6.1. creature_template

Créez les modèles de vos créatures (boss, trash mobs).

```sql
INSERT INTO `creature_template` (
    `entry`, `name`, `subname`, `minlevel`, `maxlevel`, `faction`, `npcflag`,
    `rank`, `HealthModifier`, `DamageModifier`, `ScriptName`
) VALUES (
    90001, 'Gardien des Arcanes', 'Boss du Nexus', 80, 80, 16, 0,
    3, 50.0, 5.0, 'npc_gardien_arcanes'
);
```

| Champ | Valeur | Description |
|-------|--------|-------------|
| `entry` | `90001` | ID unique de la créature. |
| `name` | `Gardien des Arcanes` | Nom du boss. |
| `minlevel` / `maxlevel` | `80` | Niveau du boss. |
| `faction` | `16` | Faction hostile. |
| `rank` | `3` | Boss (crâne). |
| `HealthModifier` | `50.0` | 50x plus de vie qu'un mob normal. |
| `DamageModifier` | `5.0` | 5x plus de dégâts. |
| `ScriptName` | `npc_gardien_arcanes` | Nom du script C++ de l'IA. |



### 6.2. creature (placement)

Placez les créatures dans l'instance.

```sql
INSERT INTO `creature` (
    `guid`, `id`, `map`, `zoneId`, `areaId`, `spawnMask`, `phaseMask`,
    `modelid`, `position_x`, `position_y`, `position_z`, `orientation`,
    `spawntimesecs`, `wander_distance`, `movement_type`
) VALUES (
    900001, 90001, 1000, 5000, 5000, 1, 1,
    12345, -100.0, 0.0, 10.0, 0.0,
    7200, 0, 0
);
```

| Champ | Description |
|-------|-------------|
| `guid` | ID unique de l'instance de créature. |
| `id` | ID du template (`creature_template.entry`). |
| `map` | ID du map (`Map.dbc`). |
| `position_x/y/z` | Coordonnées dans l'instance. |
| `spawntimesecs` | Temps de réapparition (secondes). |

---

## Étape 7 : Programmer le script d'instance (C++)

C'est **l'étape la plus critique**. Vous devez écrire une classe `InstanceScript` pour gérer la logique de votre instance.

### 7.1. Structure d'un script d'instance

```cpp
#include "InstanceScript.h"
#include "CreatureScript.h"
#include "GameObjectScript.h"
#include "Player.h"

// Définition des IDs des créatures et objets
enum Creatures
{
    BOSS_GARDIEN_ARCANES = 90001,
    BOSS_SEIGNEUR_NEANT   = 90002,
};

enum GameObjects
{
    GO_PORTAIL_NEXUS = 900001,
};

// Structure pour les données sauvegardées
struct instance_nexus_eternity : public InstanceScript
{
    instance_nexus_eternity(Map* map) : InstanceScript(map) { }

    // Initialisation
    void Initialize() override
    {
        SetBossNumber(2); // Nombre de boss
        LoadObjectData(creatureData, gameObjectData);
    }

    // Appelé quand une créature est créée
    void OnCreatureCreate(Creature* creature) override
    {
        InstanceScript::OnCreatureCreate(creature);
        // Enregistrer les créatures importantes
    }

    // Appelé quand un objet est créé
    void OnGameObjectCreate(GameObject* go) override
    {
        InstanceScript::OnGameObjectCreate(go);
        // Gérer les portes, etc.
    }

    // Appelé quand un boss est tué
    void OnBossKilled(Creature* boss) override
    {
        // Ouvrir la porte du boss suivant, etc.
    }

    // Sauvegarde des données
    std::string GetSaveData() override
    {
        OUT_SAVE_INST_DATA;
        std::ostringstream saveStream;
        saveStream << "NEX " << GetBossSaveData();
        OUT_SAVE_INST_DATA_COMPLETE;
        return saveStream.str();
    }

    // Chargement des données
    void Load(char const* data) override
    {
        if (!data)
        {
            OUT_LOAD_INST_DATA_FAIL;
            return;
        }
        OUT_LOAD_INST_DATA(data);
        std::istringstream loadStream(data);
        std::string header;
        loadStream >> header;
        if (header == "NEX")
        {
            LoadBossState(loadStream);
        }
        else
        {
            OUT_LOAD_INST_DATA_FAIL;
        }
    }

    // Données des créatures et objets
    static ObjectData const creatureData[];
    static ObjectData const gameObjectData[];
};

// Initialisation des données statiques
ObjectData const instance_nexus_eternity::creatureData[] =
{
    { BOSS_GARDIEN_ARCANES, DATA_GARDIEN_ARCANES },
    { BOSS_SEIGNEUR_NEANT,   DATA_SEIGNEUR_NEANT },
    { 0, 0 }
};

ObjectData const instance_nexus_eternity::gameObjectData[] =
{
    { GO_PORTAIL_NEXUS, DATA_PORTAIL_NEXUS },
    { 0, 0 }
};

// Enregistrement du script
void AddSC_instance_nexus_eternity()
{
    RegisterInstanceScript(instance_nexus_eternity, 1000);
}
```



### 7.2. Enregistrement du script

Ajoutez la déclaration dans le fichier d'en-tête de votre module :

```cpp
void AddSC_instance_nexus_eternity();
```

Et appelez la fonction dans `AddSC_scripts()` :

```cpp
void AddSC_scripts()
{
    // ... autres scripts ...
    AddSC_instance_nexus_eternity();
}
```



### 7.3. Compilation

Recompilez le noyau avec CMake et votre compilateur (Visual Studio, GCC, etc.).

---

## Étape 8 : Créer le patch MPQ côté client

1. Créez la structure :
   ```
   patch-4/
   └── DBFilesClient/
       ├── Map.dbc
       ├── AreaTable.dbc
       ├── DungeonEncounter.dbc
       └── LoadingScreens.dbc
   ```
2. Placez les fichiers modifiés dans `DBFilesClient`.
3. Créez `patch-4.MPQ` avec Ladik's MPQ Editor.
4. Placez-le dans le dossier `Data` du client.

> **⚠️ Important** : Le nom du patch doit être supérieur au dernier patch existant dans votre dossier `Data`.

---

## Étape 9 : Redémarrer et tester

1. **Recompilez le noyau** avec vos scripts C++.
2. **Redémarrez le serveur** pour appliquer les modifications de la base de données.
3. **Reconnectez-vous** avec le client patché.
4. **Entrez dans l'instance** via un portail ou la commande `.go xyz` (coordonnées de l'instance).
5. **Testez** les boss, les portes, les événements.

---

## Dépannage

| Problème | Cause possible | Solution |
|----------|----------------|----------|
| L'instance ne charge pas | `Map.dbc` non patché ou fichiers de carte manquants | Vérifiez le MPQ et les fichiers `.map`. |
| Le script ne se charge pas | Nom du script incorrect dans `instance_template` | Vérifiez la casse et l'orthographe. |
| Les boss ne réapparaissent pas | `spawntimesecs` trop court ou données corrompues | Vérifiez la table `creature` et `instance`. |
| Erreur de compilation | Syntaxe C++ incorrecte ou includes manquants | Vérifiez le code et les includes. |
| Le LFG ne trouve pas l'instance | `DungeonEncounter.dbc` ou `LfgDungeon.dbc` manquant | Vérifiez les entrées. |

---

## Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| Map (monde) | `1000` | `Map.dbc` |
| Zone (AreaTable) | `5000` | `AreaTable.dbc` |
| Instance template | `1000` | `instance_template` |
| Rencontre (DungeonEncounter) | `1000` | `DungeonEncounter.dbc` |
| Boss (creature_template) | `90001` | `creature_template` |
| Portail (gameobject_template) | `900001` | `gameobject_template` |

---

## Conclusion

Vous savez maintenant créer une instance personnalisée complète dans AzerothCore 3.3.5. Le processus implique la modification de fichiers DBC (`Map.dbc`, `AreaTable.dbc`, `DungeonEncounter.dbc`), de tables de base de données (`instance_template`, `instance_encounters`, `creature_template`, `creature`), et l'écriture d'un script C++ (`InstanceScript`). La création de la géométrie 3D est l'étape la plus complexe et nécessite des outils spécialisés. Testez toujours progressivement et consultez les logs serveur en cas d'erreur.

# Guide complet : Créer une gemme (Gem) personnalisée dans AzerothCore 3.3.5

## Introduction

Ce guide détaille étape par étape la création d'une **gemme (gem) personnalisée** pour AzerothCore 3.3.5. Contrairement aux guides précédents (enchantements, recettes de métier, réputations, classes), la création d'une gemme est une opération **multi-niveaux** qui touche à la fois aux fichiers DBC côté client, aux tables de base de données côté serveur, et qui nécessite une bonne compréhension du système de **sockets** et d'**enchantements d'objets**.

La particularité des gemmes réside dans le fait que leur **effet** n'est pas stocké dans `item_template` mais dans les fichiers DBC `GemProperties.dbc` et `SpellItemEnchantment.dbc`. La table `item_template` ne contient qu'une **référence** vers `GemProperties.dbc` via le champ `GemProperties`.

> **⚠️ Avertissement** : Toute modification des fichiers DBC nécessite la création d'un patch MPQ côté client. Sauvegardez toujours vos fichiers avant modification.

---

## Prérequis

- **Éditeur DBC** : MyDBCEditor, DBC Editor, ou équivalent.
- **Client de base de données** : HeidiSQL, MySQL Workbench, etc.
- **Fichiers DBC de référence** : `GemProperties.dbc`, `SpellItemEnchantment.dbc`, `Item.dbc`, `Spell.dbc`.
- **Outil MPQ** : Ladik's MPQ Editor.
- **Accès serveur** : Base de données `world` et redémarrage possible.
- **Outil de création d'objets** : Keira3 (recommandé) ou édition manuelle SQL.

---

## Vue d'ensemble du processus

La création d'une gemme personnalisée implique **cinq éléments interconnectés** :

1. **SpellItemEnchantment.dbc** (client) — Définit l'**effet** de la gemme (statistiques, proc, etc.).
2. **GemProperties.dbc** (client) — Définit les **propriétés** de la gemme (couleur, enchantement associé).
3. **Item.dbc** (client) — Référence l'objet gemme côté client.
4. **item_template** (serveur) — Définit l'objet gemme et son champ `GemProperties`.
5. **Sockets de l'équipement** — L'équipement cible doit avoir des sockets pour recevoir la gemme.

### Schéma des relations

```
┌──────────────────────────────┐
│   SpellItemEnchantment.dbc   │  ← Définit l'effet de la gemme
│  ID: 4000                    │
│  Effect Type: 5 (Statistiques)│
│  Effect: +20 Intelligence    │
└───────────┬──────────────────┘
            │  référencé par
            ▼
┌──────────────────────────────┐
│     GemProperties.dbc         │  ← Définit les propriétés
│  ID: 4030                    │
│  spellitemenchantement: 4000 │  ← ID de l'enchantement
│  color: 14 (Prismatique)     │  ← Couleurs acceptées
└───────────┬──────────────────┘
            │  référencé par
            ▼
┌──────────────────────────────┐
│      item_template            │  ← Définit l'objet gemme
│  entry: 90020                │
│  class: 3 (Gem)              │
│  GemProperties: 4030         │  ← ID de GemProperties
└───────────┬──────────────────┘
            │  s'insère dans
            ▼
┌──────────────────────────────┐
│   Équipement avec sockets    │
│  socketColor_1: 2 (Rouge)    │
│  socketColor_2: 4 (Jaune)    │
│  socketColor_3: 8 (Bleu)     │
└──────────────────────────────┘
```

---

## Étape 1 : Définir l'effet de la gemme dans SpellItemEnchantment.dbc

Le fichier `SpellItemEnchantment.dbc` contient toutes les définitions d'enchantements d'objets. C'est ici que vous définissez **ce que fait** votre gemme.

### 1.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique de l'enchantement. |
| 2 | **Charges** | Integer | Nombre de charges (généralement 0). |
| 3-5 | **Type** | Integer[3] | Type d'effet pour chaque slot. `5` = statistiques, `3` = compétence passive, `7` = compétence active. |
| 6-8 | **AmountMin** | Integer[3] | Valeur minimale de l'effet. |
| 9-11 | **AmountMax** | Integer[3] | Valeur maximale de l'effet. |
| 12-14 | **SpellID** | Integer[3] | ID du sort (si `Type = 7`). Pour `Type = 5`, cette colonne est ignorée. |
| 15-30 | **Description** | Loc | Description de l'enchantement. |
| 31 | **Description_lang** | Integer | Drapeaux de langue. |
| 32 | **AuraID** | iRefID | Référence à `ItemVisuals.dbc` (effet visuel). |
| 33 | **Slot** | Integer | Emplacement d'enchantement. |
| 34 | **GemID** | Integer | **Référence à l'ID de l'objet gemme** (ajouté en 2.0.0). |
| 35 | **EnchantmentCondition** | iRefID | Référence à `SpellItemEnchantmentCondition.dbc`. |
| 36 | **RequiredSkill** | Integer | Compétence requise. |
| 37 | **RequiredSkillValue** | Integer | Valeur de compétence requise. |
| 38 | **RequiredLevel** | Integer | Niveau requis. |

### 1.2. Types d'effet courants

| Type | Description | Exemple |
|------|-------------|---------|
| `0` | Aucun effet | — |
| `1` | Effet de combat | Proc au coup |
| `3` | Compétence passive | +20 au score de hâte |
| `5` | Statistiques | +20 Intelligence, +10 Force |
| `7` | Compétence active | Utilisable (clic droit) |

### 1.3. Exemple concret

**Gemme « Rubis de l'Intelligence » : +20 Intelligence**

- **ID** : `4000`
- **Type** : colonne 3 = `5` (statistiques)
- **AmountMin** : colonne 6 = `20`
- **AmountMax** : colonne 9 = `20`
- **Description** : `+20 Intelligence`
- **GemID** : `90020` (ID de l'objet gemme)

> **📝 Note importante** : La colonne **34 (GemID)** doit contenir l'**ID de l'objet gemme** (`item_template.entry`). Sans cette référence, la gemme ne s'affichera pas correctement dans les sockets. Comme l'indique la documentation de wowdev.wiki : « Reference to the Gem that has this ability ».

---

## Étape 2 : Définir les propriétés dans GemProperties.dbc

Le fichier `GemProperties.dbc` fait le **lien** entre l'effet (`SpellItemEnchantment.dbc`) et la **couleur** de la gemme (rouge, jaune, bleu, prismatique).

### 2.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique de la propriété. |
| 2 | **spellitemenchantement** | iRefID | **ID de l'enchantement** (référence `SpellItemEnchantment.dbc`). |
| 3 | **color** | Integer | **Couleur de la gemme** (bitmask). |
| 4 | **maxcount_inv** | Integer | Nombre maximum en inventaire (inutilisé en 3.3.5). |
| 5 | **maxcount_item** | Integer | Nombre maximum équipé (inutilisé en 3.3.5). |

### 2.2. Couleurs de gemmes (bitmask)

| Couleur | Valeur | Description |
|---------|--------|-------------|
| **Meta** | `1` | Gemme métagemme. |
| **Rouge** | `2` | Gemme rouge. |
| **Jaune** | `4` | Gemme jaune. |
| **Bleu** | `8` | Gemme bleue. |
| **Prismatique** | `14` | Rouge + Jaune + Bleu (s'insère dans n'importe quel socket). |
| **Orange** | `6` | Rouge + Jaune. |
| **Vert** | `10` | Jaune + Bleu. |
| **Violet** | `12` | Rouge + Bleu. |

> **📝 Note** : La valeur `14` est une combinaison de `2 + 4 + 8` (Rouge + Jaune + Bleu) qui correspond à une gemme **prismatique**, insérable dans n'importe quel socket. Comme l'explique un moddeur sur modcraft : « Keep in mind that the wiki say "Gem Color (a combination of Meta = 1, Red = 2, Yellow = 4 and Blue = 8)" but you can add "14" as the prismatic one ».

### 2.3. Exemple concret

**Gemme « Rubis de l'Intelligence » (rouge)**

- **ID** : `4030`
- **spellitemenchantement** : `4000` (ID de l'enchantement défini à l'étape 1)
- **color** : `2` (Rouge)

---

## Étape 3 : Créer l'objet gemme (Item.dbc + item_template)

### 3.1. Item.dbc (côté client)

Dupliquez une ligne d'objet existant dans `Item.dbc` et modifiez son **ID** (colonne 1) pour `90020`. Vérifiez que :
- **Class** (colonne 2) : `3` (Gem)
- **SubClass** (colonne 3) : `0` (ou `1` à `6` selon le type de gemme)
- **DisplayInfoID** (colonne 6) : Un ID d'affichage de gemme existant.

### 3.2. item_template (côté serveur)

Insérez une nouvelle ligne dans la table `item_template` :

```sql
INSERT INTO `item_template` (
    `entry`, `class`, `subclass`, `name`, `displayid`, `Quality`,
    `BuyCount`, `BuyPrice`, `SellPrice`, `InventoryType`, `ItemLevel`,
    `RequiredLevel`, `bonding`, `GemProperties`, `description`
) VALUES (
    90020, 3, 0, 'Rubis de l\'Intelligence', 12345, 4,
    1, 50000, 12500, 0, 80,
    0, 1, 4030, '+20 Intelligence'
);
```

| Champ | Valeur | Description |
|-------|--------|-------------|
| `entry` | `90020` | ID unique de l'objet gemme. |
| `class` | `3` | **Gem** (Gemme). |
| `subclass` | `0` | Rouge (ou `1` Jaune, `2` Bleu, `3` Violet, `4` Vert, `5` Orange, `6` Meta, `7` Prismatique). |
| `name` | `Rubis de l'Intelligence` | Nom de la gemme. |
| `displayid` | `12345` | ID d'affichage (icône). |
| `Quality` | `4` | Épique (violet). |
| `InventoryType` | `0` | Non équipable. |
| `ItemLevel` | `80` | Niveau d'objet. |
| `RequiredLevel` | `0` | Aucun niveau requis. |
| `bonding` | `1` | Lié quand ramassé. |
| **`GemProperties`** | **`4030`** | **ID de GemProperties.dbc** (le lien critique). |
| `description` | `+20 Intelligence` | Description. |

> **📝 Note cruciale** : Le champ **`GemProperties`** dans `item_template` est le **lien** vers `GemProperties.dbc`. C'est ce champ qui détermine l'effet de la gemme. Comme l'explique un utilisateur sur uiwow : « 在item_template表有一列叫GemProperties，通过这一列的数值在GemProperties.dbc查找对应行，对应行的第二列取值在spellitemenchantement.dbc查找 ».

---

## Étape 4 : Configurer les sockets de l'équipement

Pour que la gemme puisse être insérée, l'équipement cible doit avoir des **sockets**. Les sockets sont définis dans `item_template` via les champs suivants :

| Champ | Description | Valeurs |
|-------|-------------|---------|
| **`socketColor_1`** | Couleur du socket 1 | `1` Meta, `2` Rouge, `4` Jaune, `8` Bleu |
| **`socketColor_2`** | Couleur du socket 2 | Idem |
| **`socketColor_3`** | Couleur du socket 3 | Idem |
| **`socketContent_1`** | Contenu initial du socket 1 | `0` (vide) |
| **`socketContent_2`** | Contenu initial du socket 2 | `0` |
| **`socketContent_3`** | Contenu initial du socket 3 | `0` |
| **`socketBonus`** | ID du bonus de socket | Référence `SpellItemEnchantment.dbc` |

### 4.1. Exemple : ajouter des sockets à un équipement existant

```sql
UPDATE `item_template`
SET `socketColor_1` = 2,   -- Socket Rouge
    `socketColor_2` = 4,   -- Socket Jaune
    `socketColor_3` = 8,   -- Socket Bleu
    `socketContent_1` = 0,
    `socketContent_2` = 0,
    `socketContent_3` = 0,
    `socketBonus` = 3000   -- ID du bonus de socket
WHERE `entry` = 50000;  -- ID de l'équipement
```

> **📝 Note** : Le **socket bonus** est un enchantement supplémentaire accordé lorsque tous les sockets sont remplis avec des gemmes de la couleur correspondante. Il est défini dans `SpellItemEnchantment.dbc`.

### 4.2. Types de sockets

| Valeur | Socket | Couleur acceptée |
|--------|--------|------------------|
| `1` | Meta | Gemme métagemme uniquement |
| `2` | Rouge | Rouge ou prismatique |
| `4` | Jaune | Jaune, orange, vert ou prismatique |
| `8` | Bleu | Bleu, vert, violet ou prismatique |

---

## Étape 5 : (Optionnel) Créer un sort pour la gemme

Si votre gemme doit avoir un **effet actif** (utilisable), vous devez créer un sort dans `Spell.dbc` et le référencer dans `SpellItemEnchantment.dbc`.

### 5.1. Champs essentiels

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97000` | ID unique du sort. |
| 5 | **Attributes** | `0x00000040` | `SPELL_ATTR0_PASSIVE` si passif. |
| 72 | **Effect_1** | `6` | `SPELL_EFFECT_APPLY_AURA`. |
| 108 | **EffectApplyAuraName** | Variable | Type d'aura. |
| 128 | **SpellIconID** | `1` | Icône du sort. |
| 131-162 | **Name** | `Effet du Rubis` | Nom du sort. |

### 5.2. Lien avec SpellItemEnchantment.dbc

Dans `SpellItemEnchantment.dbc`, colonne **12-14** (`SpellID`), référencez l'ID du sort créé. Le `Type` (colonne 3-5) doit être `7` (compétence active) ou `3` (compétence passive).

---

## Étape 6 : Créer le patch MPQ côté client

1. Créez la structure :
   ```
   patch-4/
   └── DBFilesClient/
       ├── SpellItemEnchantment.dbc
       ├── GemProperties.dbc
       └── Item.dbc
   ```
2. Placez les fichiers modifiés dans `DBFilesClient`.
3. Créez `patch-4.MPQ` avec Ladik's MPQ Editor.
4. Placez-le dans le dossier `Data` du client.

> **⚠️ Important** : Le nom du patch doit être supérieur au dernier patch existant dans votre dossier `Data`. Si vous avez `patch-3.MPQ`, nommez le vôtre `patch-4.MPQ` ou `patch-Z.MPQ`.

---

## Étape 7 : Redémarrer et tester

1. **Redémarrez votre serveur** pour que les modifications de la base de données soient prises en compte.
2. **Reconnectez-vous** au jeu avec le client patché.
3. **Obtenez la gemme** via la commande `.additem 90020` ou via un marchand.
4. **Obtenez un équipement avec sockets** et insérez la gemme.
5. **Vérifiez** que les statistiques sont appliquées et que l'icône s'affiche correctement.

---

## Dépannage

| Problème | Cause possible | Solution |
|----------|----------------|----------|
| La gemme n'apparaît pas dans le socket | `GemID` non renseigné dans `SpellItemEnchantment.dbc` | Remplissez la colonne 34 avec l'ID de l'objet gemme. |
| Les statistiques ne s'appliquent pas | `GemProperties` incorrect dans `item_template` | Vérifiez que le champ `GemProperties` pointe vers un ID valide dans `GemProperties.dbc`. |
| L'icône de la gemme disparaît après insertion | Problème de texture `.blp` | Vérifiez que votre texture de gemme possède une couche alpha. |
| La gemme ne s'insère pas | `socketColor` incompatible | Vérifiez que la couleur de la gemme (`color` dans `GemProperties.dbc`) correspond au type de socket. |
| Erreur « GemProperties not found » | `GemProperties.dbc` non patché | Vérifiez le MPQ et l'ID. |
| Le serveur plante | Conflit d'ID | Utilisez des IDs uniques (90000+). |

---

## Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| Enchantement (effet) | `4000` | `SpellItemEnchantment.dbc` |
| Propriétés de gemme | `4030` | `GemProperties.dbc` |
| Objet gemme | `90020` | `Item.dbc` + `item_template` |
| Sort (optionnel) | `97000` | `Spell.dbc` |
| Équipement avec sockets | `50000` | `item_template` |

---

## Conclusion

Vous savez maintenant créer une gemme personnalisée complète dans AzerothCore 3.3.5. Le processus implique la modification de fichiers DBC (`SpellItemEnchantment.dbc`, `GemProperties.dbc`, `Item.dbc`) et de tables de base de données (`item_template`, avec le champ critique `GemProperties`). La particularité des gemmes réside dans le **chaînage** entre ces éléments : l'effet est dans `SpellItemEnchantment.dbc`, les propriétés (couleur) dans `GemProperties.dbc`, et l'objet dans `item_template`. Testez toujours progressivement et vérifiez que vos sockets sont correctement configurés.

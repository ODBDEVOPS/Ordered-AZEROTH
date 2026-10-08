# Guide complet : Créer une mascotte (compagnon) personnalisée dans AzerothCore 3.3.5

## Introduction

Ce guide détaille étape par étape la création d'une **mascotte (compagnon non-combattant)** personnalisée pour AzerothCore 3.3.5. Contrairement aux mascottes de combat (chasseur, démoniste), une mascotte de compagnie est une créature **purement cosmétique** qui suit le joueur, ne participe pas au combat, et est invoquée via un sort ou un objet.

Ce type de mascotte est plus simple à créer que les autres éléments couverts précédemment (instance, classe, forme de druide), car elle ne nécessite **aucune modification du noyau (core)** ni de script C++. Elle repose uniquement sur des fichiers DBC côté client et des tables de base de données côté serveur.

> **⚠️ Avertissement** : Toute modification des fichiers DBC nécessite la création d'un patch MPQ côté client. Sauvegardez toujours vos fichiers avant modification.

---

## Prérequis

- **Éditeur DBC** : MyDBCEditor, DBC Editor, ou équivalent.
- **Client de base de données** : HeidiSQL, MySQL Workbench, etc.
- **Fichiers DBC de référence** : `Spell.dbc`, `Item.dbc`, `CreatureDisplayInfo.dbc`, `SummonProperties.dbc`.
- **Outil MPQ** : Ladik's MPQ Editor.
- **Accès serveur** : Base de données `world` et redémarrage possible.

---

## Vue d'ensemble du processus

La création d'une mascotte de compagnie implique **quatre éléments interconnectés** :

1. **Creature_template** (serveur) — Définit la créature mascotte (nom, modèle, faction, IA passive).
2. **Spell.dbc** (client) — Définit le sort d'invocation de la mascotte (effet `28` : `SPELL_EFFECT_SUMMON`).
3. **SummonProperties.dbc** (client) — Définit les propriétés d'invocation (type `Mini pet`).
4. **Item.dbc** + **item_template** (client + serveur) — Définissent l'objet qui apprend ou invoque la mascotte (optionnel).

### Schéma des relations

```
┌─────────────────────────┐
│   creature_template      │  ← Définit la créature mascotte
│  entry: 90010           │
│  name: "Bébé tigre"     │
│  faction: 35 (amical)   │
│  AIName: PassiveAI      │
└───────────┬─────────────┘
            │  référencé par
            ▼
┌─────────────────────────┐
│       Spell.dbc          │  ← Sort d'invocation
│  ID: 97000              │
│  Effect_1: 28           │  ← SPELL_EFFECT_SUMMON
│  EffectMiscValue: 90010 │  ← ID de la créature
│  EffectMiscValueB: 5    │  ← Mini pet (SummonProperties)
└───────────┬─────────────┘
            │  référencé par
            ▼
┌─────────────────────────┐
│   item_template          │  ← Objet qui apprend/invoque
│  entry: 90011           │
│  spellid_1: 97000       │
│  spelltrigger_1: 0      │  ← Utilisation
└─────────────────────────┘
```

---

## Étape 1 : Définir la créature mascotte dans `creature_template`

La table `creature_template` définit l'apparence et le comportement de votre mascotte. Insérez une nouvelle ligne dans la base `world`.

### 1.1. Champs essentiels

| Champ | Valeur | Description |
|-------|--------|-------------|
| **entry** | `90010` | ID unique de la créature. |
| **name** | `Bébé tigre` | Nom de la mascotte. |
| **subname** | `Compagnon` | Sous-titre (optionnel). |
| **minlevel** / **maxlevel** | `1` / `1` | Niveau de la créature (1 pour une mascotte). |
| **faction** | `35` | Faction amicale (35 = amical avec tous les joueurs). |
| **npcflag** | `0` | Aucun drapeau PNJ. |
| **speed_walk** | `0.7` | Vitesse de marche (légèrement plus rapide que le joueur). |
| **speed_run** | `0.7` | Vitesse de course (doit être égale ou supérieure au joueur pour suivre). |
| **rank** | `0` | Normal (pas d'élite). |
| **dmgschool** | `0` | École de dégâts (sans importance). |
| **BaseAttackTime** | `2000` | Temps d'attaque de base. |
| **AIName** | `PassiveAI` | **IA passive** : la créature n'attaque jamais. |
| **MovementType** | `1` | Mouvement aléatoire (ou `0` pour statique). |
| **HealthModifier** | `1` | Modificateur de vie. |
| **ManaModifier** | `1` | Modificateur de mana. |
| **ArmorModifier** | `1` | Modificateur d'armure. |
| **DamageModifier** | `1` | Modificateur de dégâts. |

> **📝 Note importante** : Le champ `AIName` doit impérativement être `PassiveAI` pour que la mascotte ne participe pas au combat. Vous pouvez aussi utiliser `NullCreatureAI` si vous voulez qu'elle ne fasse absolument rien.

### 1.2. Requête SQL d'exemple

```sql
INSERT INTO `creature_template` (
    `entry`, `name`, `subname`, `minlevel`, `maxlevel`, `faction`, `npcflag`,
    `speed_walk`, `speed_run`, `rank`, `dmgschool`,
    `BaseAttackTime`, `AIName`, `MovementType`,
    `HealthModifier`, `ManaModifier`, `ArmorModifier`, `DamageModifier`
) VALUES (
    90010, 'Bébé tigre', 'Compagnon', 1, 1, 35, 0,
    0.7, 0.7, 0, 0,
    2000, 'PassiveAI', 1,
    1, 1, 1, 1
);
```

### 1.3. Définir le modèle de la mascotte

Le modèle visuel de la mascotte est défini par `modelid` dans la table `creature` (placement) ou par `modelid` dans `creature_template` (pour les créatures invoquées). Pour une mascotte invoquée, le modèle est généralement référencé dans `CreatureDisplayInfo.dbc`.

Si vous utilisez un modèle existant (par exemple, un bébé tigre du jeu), trouvez son `DisplayID` dans `CreatureDisplayInfo.dbc`. Sinon, vous devrez ajouter une nouvelle entrée dans `CreatureDisplayInfo.dbc` et `CreatureModelData.dbc` pour un modèle personnalisé.

---

## Étape 2 : Créer le sort d'invocation dans `Spell.dbc`

Le sort d'invocation est ce qui fait apparaître la mascotte. Il utilise l'effet `28` (`SPELL_EFFECT_SUMMON`) et référence l'ID de la créature.

### 2.1. Champs essentiels

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97000` | ID unique du sort. |
| 5 | **Attributes** | `0x00000000` | Attributs du sort (laissez `0` pour un sort de mascotte). |
| 18-19 | **TargetType** | `0` | Aucune cible (sort sur soi-même). |
| 39 | **BaseLevel** | `1` | Niveau de base. |
| 72 | **Effect_1** | `28` | `SPELL_EFFECT_SUMMON` (invoquer une créature). |
| 75 | **EffectBasePoints** | `0` | Points de base (0 = durée illimitée jusqu'à annulation). |
| 108 | **EffectMiscValue** | `90010` | **ID de la créature** (référence `creature_template.entry`). |
| 109 | **EffectMiscValueB** | `5` | **ID de SummonProperties** (`5` = Mini pet, voir `SummonProperties.dbc`). |
| 128 | **SpellIconID** | `1` | Icône du sort. |
| 131-162 | **Name** | `Invocation : Bébé tigre` | Nom du sort. |
| 171-178 | **Description** | `Invoque un bébé tigre qui vous suit partout.` | Description. |

> **📝 Note cruciale** : Le champ `EffectMiscValueB` (colonne 109) doit être `5` (Mini pet) pour que la créature soit traitée comme une **mascotte de compagnie** et non comme un familier de combat. Cela empêche le client d'afficher les barres de vie/mana et les commandes de familier.

### 2.2. Exemple concret

- **ID** : `97000`
- **Effect_1** : `28` (SUMMON)
- **EffectMiscValue** : `90010` (ID de la créature mascotte)
- **EffectMiscValueB** : `5` (Mini pet)
- **Name** : `Invocation : Bébé tigre`

---

## Étape 3 : Vérifier les propriétés d'invocation dans `SummonProperties.dbc`

Le fichier `SummonProperties.dbc` définit le comportement de l'invocation. Pour une mascotte de compagnie, vous devez utiliser l'ID `5` (Mini pet).

### 3.1. Structure de `SummonProperties.dbc`

| Colonne | Champ | Description |
|---------|-------|-------------|
| 1 | **ID** | ID unique. |
| 2 | **Control** | Type de contrôle (`0` = None, `1` = Guardian, `2` = Pet). |
| 3 | **Faction** | Faction de l'invocation. |
| 4 | **Title** | Type d'invocation (`5` = Mini pet). |
| 5 | **Slot** | Emplacement (`5` = Critter). |
| 6 | **Flags** | Drapeaux (voir ci-dessous). |

### 3.2. ID `5` : Mini pet (Compagnon)

L'entrée ID `5` dans `SummonProperties.dbc` correspond à une mascotte de compagnie :

| Champ | Valeur | Signification |
|-------|--------|---------------|
| **ID** | `5` | Mini pet |
| **Control** | `0` | None (pas de contrôle par le joueur) |
| **Title** | `5` | Mini pet |
| **Slot** | `5` | Critter |
| **Flags** | `0x00000010` | Only Visible to Summoner (visible uniquement par l'invocateur) |

> **📝 Note** : L'entrée ID `5` existe déjà dans le DBC vanilla. Vous n'avez pas besoin de la modifier, utilisez simplement `5` comme valeur de `EffectMiscValueB` dans votre sort.

---

## Étape 4 : Créer l'objet d'apprentissage/invocation (optionnel)

Il existe **deux méthodes** pour que le joueur obtienne la mascotte.

### Méthode A : Via un sort directement appris

Le joueur apprend le sort d'invocation `97000` via un entraîneur, une quête, ou la commande `.learn 97000`.

### Méthode B : Via un objet consommable

Créez un objet qui apprend ou invoque la mascotte. C'est la méthode la plus courante (comme les mascottes du jeu).

**Item.dbc** (côté client) :

Dupliquez une ligne d'objet existant et modifiez son **ID** (colonne 1) pour `90011`. Vérifiez que `class` (colonne 2) est `15` (Miscellaneous) ou `0` (Consumable) et `subclass` (colonne 3) est `0`.

**item_template** (côté serveur) :

```sql
INSERT INTO `item_template` (
    `entry`, `class`, `subclass`, `name`, `displayid`, `Quality`,
    `BuyCount`, `BuyPrice`, `SellPrice`, `InventoryType`,
    `ItemLevel`, `RequiredLevel`, `bonding`,
    `spellid_1`, `spelltrigger_1`, `spellcharges_1`, `description`
) VALUES (
    90011, 15, 0, 'Bébé tigre', 12345, 3,
    1, 10000, 2500, 0,
    1, 0, 1,
    97000, 0, -1, 'Apprend à invoquer un bébé tigre.'
);
```

| Champ | Valeur | Description |
|-------|--------|-------------|
| `entry` | `90011` | ID unique de l'objet. |
| `class` | `15` | Miscellaneous (objet divers). |
| `subclass` | `0` | Miscellaneous. |
| `name` | `Bébé tigre` | Nom de l'objet. |
| `displayid` | `12345` | ID d'affichage (icône). |
| `Quality` | `3` | Rare (bleu). |
| `bonding` | `1` | Lié quand ramassé. |
| `spellid_1` | `97000` | **ID du sort d'invocation**. |
| `spelltrigger_1` | `0` | `ON_USE` (utilisation). |
| `spellcharges_1` | `-1` | Consommé à l'utilisation. |

> **📝 Note** : Avec `spelltrigger_1 = 0`, l'utilisation de l'objet lance directement le sort d'invocation. Si vous voulez que l'objet **apprenne** le sort (pour invocation permanente via le grimoire), utilisez `spelltrigger_1 = 6` (`ON_LEARN`).

---

## Étape 5 : (Optionnel) Ajouter la mascotte aux nouveaux personnages

Pour que les nouveaux personnages possèdent automatiquement la mascotte, ajoutez une entrée dans `playercreateinfo_spell` (ou `playercreateinfo_spell_custom` si vous utilisez `PlayerStart.CustomSpells`).

```sql
INSERT INTO `playercreateinfo_spell_custom` (
    `racemask`, `classmask`, `Spell`, `Note`
) VALUES (
    0, 0, 97000, 'Invocation : Bébé tigre'
);
```

| Champ | Description |
|-------|-------------|
| `racemask` | `0` = toutes les races. |
| `classmask` | `0` = toutes les classes. |
| `Spell` | ID du sort d'invocation (`97000`). |
| `Note` | Commentaire. |

> **📝 Note** : Vous devez activer `PlayerStart.CustomSpells = 1` dans `worldserver.conf` pour que cette table soit prise en compte.

---

## Étape 6 : Créer le patch MPQ côté client

1. Créez la structure :
   ```
   patch-4/
   └── DBFilesClient/
       ├── Spell.dbc
       └── Item.dbc
   ```
2. Placez les fichiers modifiés dans `DBFilesClient`.
3. Créez `patch-4.MPQ` avec Ladik's MPQ Editor.
4. Placez-le dans le dossier `Data` du client.

> **⚠️ Important** : Le nom du patch doit être supérieur au dernier patch existant dans votre dossier `Data`. Si vous avez `patch-3.MPQ`, nommez le vôtre `patch-4.MPQ`.

---

## Étape 7 : Redémarrer et tester

1. **Redémarrez votre serveur** pour que les modifications de la base de données soient prises en compte.
2. **Reconnectez-vous** au jeu avec le client patché.
3. **Obtenez l'objet** ou apprenez le sort via `.learn 97000`.
4. **Utilisez l'objet** ou lancez le sort.
5. **Vérifiez** que la mascotte apparaît, vous suit, et ne participe pas au combat.

---

## Dépannage

| Problème | Cause possible | Solution |
|----------|----------------|----------|
| La mascotte n'apparaît pas | `CreatureDisplayInfo` invalide ou modèle non patché | Vérifiez le `modelid` et le MPQ. |
| La mascotte est hostile | `faction` incorrecte | Utilisez `35` (amical) dans `creature_template`. |
| La mascotte attaque | `AIName` n'est pas `PassiveAI` | Vérifiez le champ `AIName`. |
| La mascotte ne suit pas | `MovementType` incorrect ou vitesse trop basse | Utilisez `MovementType = 1` et `speed_run >= 0.7`. |
| La mascotte disparaît en montant | Comportement normal des mini-pets | Utilisez `SummonProperties` ID `5` (Mini pet). |
| L'objet ne donne pas la mascotte | `spelltrigger_1` incorrect | Utilisez `0` (utilisation) ou `6` (apprentissage). |
| Le serveur plante | Conflit d'ID ou entrée DBC manquante | Vérifiez les IDs et les logs serveur. |

---

## Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| Créature mascotte | `90010` | `creature_template` |
| Sort d'invocation | `97000` | `Spell.dbc` |
| Objet d'apprentissage | `90011` | `Item.dbc` + `item_template` |
| SummonProperties (Mini pet) | `5` | `SummonProperties.dbc` |
| Faction amicale | `35` | `Faction.dbc` |

---

## Conclusion

Vous savez maintenant créer une mascotte de compagnie personnalisée dans AzerothCore 3.3.5. Le processus implique la modification de `creature_template` (côté serveur), `Spell.dbc` et `Item.dbc` (côté client), et l'utilisation de `SummonProperties.dbc` ID `5` pour le comportement de mini-pet. Contrairement aux autres éléments couverts dans cette série de guides, la mascotte de compagnie ne nécessite **aucune modification du noyau C++**, ce qui en fait l'une des créations les plus accessibles. Testez toujours progressivement et consultez les logs serveur en cas d'erreur.

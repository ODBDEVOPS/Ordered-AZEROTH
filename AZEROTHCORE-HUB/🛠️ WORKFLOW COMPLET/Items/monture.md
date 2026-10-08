# Guide complet : Créer une monture personnalisée dans AzerothCore 3.3.5

## Introduction

Ce guide détaille étape par étape la création d'une **monture personnalisée** pour AzerothCore 3.3.5. La création d'une monture est une opération qui touche à la fois aux **fichiers DBC côté client** (`Spell.dbc`, `Item.dbc`) et aux **tables de base de données côté serveur** (`item_template`, `creature_template`), et qui peut nécessiter une **modification du noyau (core)** pour les montures adaptatives.

Il existe **deux approches principales** :

| Approche | Complexité | Modifications |
|----------|------------|---------------|
| **Monture simple** (invocation directe d'une créature) | Facile | `Spell.dbc` + `Item.dbc` + `item_template` + `creature_template` |
| **Monture adaptative** (vitesse variable selon la compétence de monte) | Avancée | + Modification du core C++ (`spell_generic.cpp`) |

> **⚠️ Avertissement** : Toute modification des fichiers DBC nécessite la création d'un patch MPQ côté client. Sauvegardez toujours vos fichiers avant modification.

---

## Prérequis

- **Éditeur DBC** : MyDBCEditor, DBC Editor, ou équivalent.
- **Client de base de données** : HeidiSQL, MySQL Workbench, etc.
- **Fichiers DBC de référence** : `Spell.dbc`, `Item.dbc`, `CreatureDisplayInfo.dbc`.
- **Outil MPQ** : Ladik's MPQ Editor.
- **Accès serveur** : Base de données `world` et possibilité de **recompiler le noyau** (pour les montures adaptatives).
- **Environnement de développement C++** (pour les montures adaptatives) : Visual Studio, CMake, sources d'AzerothCore.

---

## Vue d'ensemble du processus

La création d'une monture implique **quatre éléments interconnectés** :

1. **creature_template** (serveur) — Définit la créature qui sert de monture.
2. **Spell.dbc** (client) — Définit le sort d'invocation de la monture.
3. **Item.dbc** + **item_template** (client + serveur) — Définissent l'objet qui apprend ou invoque la monture.
4. **spell_generic.cpp** (C++, optionnel) — Pour les montures adaptatives dont la vitesse varie.

### Schéma des relations

```
┌─────────────────────────┐
│   creature_template      │  ← Définit la créature-monture
│  entry: 90020           │
│  name: "Tigre spectral" │
│  modelid: 12345         │  ← Modèle 3D
└───────────┬─────────────┘
            │  référencé par
            ▼
┌─────────────────────────┐
│       Spell.dbc          │  ← Sort d'invocation
│  ID: 97000              │
│  Effect_1: 78           │  ← SPELL_EFFECT_APPLY_MOUNT
│  EffectMiscValue: 90020 │  ← ID de la créature
│  Aura: 32               │  ← Aura de vitesse montée
└───────────┬─────────────┘
            │  référencé par
            ▼
┌─────────────────────────┐
│   item_template          │  ← Objet qui apprend la monture
│  entry: 90021           │
│  class: 15 (Misc)       │
│  spellid_1: 97000       │
│  spelltrigger_1: 6      │  ← Apprentissage
└─────────────────────────┘
```

---

## Étape 1 : Créer la créature-monture dans `creature_template`

La table `creature_template` définit l'apparence et le comportement de la créature qui servira de monture. Insérez une nouvelle ligne dans la base `world`.

### 1.1. Champs essentiels

| Champ | Valeur | Description |
|-------|--------|-------------|
| **entry** | `90020` | ID unique de la créature. |
| **name** | `Tigre spectral` | Nom de la monture. |
| **minlevel** / **maxlevel** | `1` / `1` | Niveau de la créature. |
| **faction** | `35` | Faction amicale. |
| **npcflag** | `0` | Aucun drapeau PNJ. |
| **speed_walk** | `1` | Vitesse de marche (non utilisée pour les montures). |
| **speed_run** | `1` | Vitesse de course (non utilisée pour les montures). |
| **rank** | `0` | Normal. |
| **dmgschool** | `0` | École de dégâts (sans importance). |
| **BaseAttackTime** | `2000` | Temps d'attaque de base. |
| **AIName** | `NullCreatureAI` | **IA nulle** : la créature ne fait rien. |
| **MovementType** | `0` | Statique (ne bouge pas). |
| **modelid** | `12345` | **ID du modèle 3D** (référence `CreatureDisplayInfo.dbc`). |
| **HealthModifier** | `1` | Modificateur de vie. |
| **ManaModifier** | `1` | Modificateur de mana. |
| **ArmorModifier** | `1` | Modificateur d'armure. |
| **DamageModifier** | `1` | Modificateur de dégâts. |

### 1.2. Requête SQL d'exemple

```sql
INSERT INTO `creature_template` (
    `entry`, `name`, `minlevel`, `maxlevel`, `faction`, `npcflag`,
    `speed_walk`, `speed_run`, `rank`, `dmgschool`,
    `BaseAttackTime`, `AIName`, `MovementType`, `modelid`,
    `HealthModifier`, `ManaModifier`, `ArmorModifier`, `DamageModifier`
) VALUES (
    90020, 'Tigre spectral', 1, 1, 35, 0,
    1, 1, 0, 0,
    2000, 'NullCreatureAI', 0, 12345,
    1, 1, 1, 1
);
```

### 1.3. Modèle 3D de la monture

Le modèle est défini par `modelid` (référence `CreatureDisplayInfo.dbc`). Vous pouvez :
- **Réutiliser un modèle existant** : trouvez son `DisplayID` dans `CreatureDisplayInfo.dbc` (par exemple, le tigre spectral du jeu).
- **Créer un modèle personnalisé** : ajoutez une nouvelle entrée dans `CreatureDisplayInfo.dbc` et `CreatureModelData.dbc`, puis placez le fichier `.m2` correspondant dans votre patch MPQ.

---

## Étape 2 : Créer le sort d'invocation dans `Spell.dbc`

Le sort d'invocation est ce qui fait apparaître la monture sous le joueur. Il utilise l'effet `78` (`SPELL_EFFECT_APPLY_MOUNT`) et référence l'ID de la créature.

### 2.1. Champs essentiels

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97000` | ID unique du sort. |
| 5 | **Attributes** | `0x00000000` | Attributs du sort. |
| 9 | **CastTimeIndex** | `1` | Temps d'incantation instantané. |
| 18-19 | **TargetType** | `0` | Aucune cible. |
| 39 | **BaseLevel** | `1` | Niveau de base. |
| 72 | **Effect_1** | `78` | `SPELL_EFFECT_APPLY_MOUNT` (appliquer une monture). |
| 75 | **EffectBasePoints** | `0` | Points de base (0 pour une monture permanente). |
| 108 | **EffectMiscValue** | `90020` | **ID de la créature** (référence `creature_template.entry`). |
| 109 | **EffectMiscValueB** | `0` | Valeur diverse B. |
| 128 | **SpellIconID** | `1` | Icône du sort. |
| 131-162 | **Name** | `Invocation : Tigre spectral` | Nom du sort. |
| 171-178 | **Description** | `Invoque un tigre spectral comme monture.` | Description. |

> **📝 Note** : L'effet `78` (`SPELL_EFFECT_APPLY_MOUNT`) est confirmé par la documentation : « SPELL_AURA_MOD_MOUNTED... 78 - SPELL_EFFECT_APPLY_MOUNT ». Le champ `EffectMiscValue` (colonne 108) doit pointer vers l'ID de la créature-monture. Un moddeur confirme : « 110 - EffectMiscValue - this becomes the creature_Template Id that you want to have the player mount ».

### 2.2. Exemple concret

- **ID** : `97000`
- **Effect_1** : `78` (APPLY_MOUNT)
- **EffectMiscValue** : `90020` (ID de la créature-monture)
- **Name** : `Invocation : Tigre spectral`

### 2.3. Ajouter l'aura de vitesse

Pour que la monture augmente la vitesse du joueur, le sort doit également appliquer une **aura de vitesse montée**. Cela se fait via une **aura supplémentaire** dans le même sort ou via un sort séparé.

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 108 | **EffectApplyAuraName** | `32` | `SPELL_AURA_MOD_INCREASE_MOUNTED_SPEED` |
| 75 | **EffectBasePoints** | `60` | Vitesse en % (60 = +60% de vitesse montée). |

> **📝 Note** : L'aura `32` (`SPELL_AURA_MOD_INCREASE_MOUNTED_SPEED`) est confirmée par la base de données : « Ram Riding <Passive> ... Auras: 32: Mod increase mounted speed Increases riding speed by 0. ». La valeur `EffectBasePoints` détermine le bonus de vitesse (100 = +100%, 280 = +280%, etc.).

---

## Étape 3 : Créer l'objet qui apprend la monture

### 3.1. Item.dbc (côté client)

Dupliquez une ligne d'objet existant dans `Item.dbc` et modifiez son **ID** (colonne 1) pour `90021`. Vérifiez que :
- **Class** (colonne 2) : `15` (Miscellaneous) ou `0` (Consumable)
- **SubClass** (colonne 3) : `0`
- **DisplayInfoID** (colonne 6) : Un ID d'affichage d'icône de monture.

### 3.2. item_template (côté serveur)

Insérez une nouvelle ligne dans la table `item_template` :

```sql
INSERT INTO `item_template` (
    `entry`, `class`, `subclass`, `name`, `displayid`, `Quality`,
    `BuyCount`, `BuyPrice`, `SellPrice`, `InventoryType`,
    `ItemLevel`, `RequiredLevel`, `bonding`,
    `spellid_1`, `spelltrigger_1`, `spellcharges_1`, `description`
) VALUES (
    90021, 15, 0, 'Tigre spectral', 12346, 4,
    1, 100000, 25000, 0,
    1, 20, 1,
    97000, 6, -1, 'Apprend à invoquer un tigre spectral.'
);
```

| Champ | Valeur | Description |
|-------|--------|-------------|
| `entry` | `90021` | ID unique de l'objet. |
| `class` | `15` | Miscellaneous. |
| `subclass` | `0` | Miscellaneous. |
| `name` | `Tigre spectral` | Nom de l'objet. |
| `displayid` | `12346` | ID d'affichage (icône). |
| `Quality` | `4` | Épique. |
| `bonding` | `1` | Lié quand ramassé. |
| **`spellid_1`** | **`97000`** | **ID du sort d'invocation**. |
| **`spelltrigger_1`** | **`6`** | **`ON_LEARN`** : apprend le sort. |
| `spellcharges_1` | `-1` | Consommé à l'utilisation. |

> **📝 Note importante** : Le `spelltrigger_1` doit être `6` (`ON_LEARN`) pour que l'objet **apprenne** le sort au joueur, et non qu'il l'invoque directement. Un moddeur confirme : « spelltrigger_1：法术触发类型，0-使用，1-装备，2-击中时可能，4-灵魂石，5-马上使用，无延迟，6-学习法术编号 ». Si vous préférez que l'objet **invoque** directement la monture (consommable), utilisez `spelltrigger_1 = 0`.

### 3.3. Monture à durée limitée (optionnel)

Pour créer une monture **temporaire** (qui disparaît après un certain temps), ajoutez une valeur dans la colonne `duration` de `item_template`. Comme l'explique un utilisateur : « item_template表添加一个物品，spellid_1填spell.dbc里坐骑的技能ID，spelltrigger_1=0,spellcharges_1=0,duration列填持续时间（单位是秒），正数为游戏时间,负数为现实时间 ».

---

## Étape 4 : (Avancé) Créer une monture adaptative

Les montures adaptatives (comme le **Coursier céleste** ou **Invincible**) adaptent leur vitesse en fonction de la compétence de monte du joueur. Pour créer ce type de monture, vous devez **modifier le noyau C++**.

### 4.1. Principe

Le noyau AzerothCore gère les montures adaptatives via la classe `spell_gen_mount` dans `src/server/scripts/Spells/spell_generic.cpp`. Cette classe prend en paramètre **cinq IDs de sorts** correspondant aux différentes vitesses :

```cpp
spell_gen_mount(uint32 mount0, uint32 mount60, uint32 mount100, uint32 mount150, uint32 mount280, uint32 mount310)
```

| Paramètre | Vitesse | Compétence de monte requise |
|-----------|---------|----------------------------|
| `mount0` | 0% | Aucune |
| `mount60` | 60% | Apprenti (75) |
| `mount100` | 100% | Compagnon (150) |
| `mount150` | 150% | Expert (225) |
| `mount280` | 280% | Artisan (300) |
| `mount310` | 310% | Maître (375) |

### 4.2. Créer les 5 sorts de vitesse

Vous devez créer **cinq sorts distincts** dans `Spell.dbc`, un pour chaque palier de vitesse. Chaque sort utilise l'effet `78` (APPLY_MOUNT) et une aura de vitesse `32` avec une valeur différente :

| Sort | ID | EffectMiscValue (créature) | Aura 32 (vitesse) |
|------|-----|---------------------------|-------------------|
| `mount0` | `97000` | `90020` | `0` |
| `mount60` | `97001` | `90020` | `60` |
| `mount100` | `97002` | `90020` | `100` |
| `mount150` | `97003` | `90020` | `150` |
| `mount280` | `97004` | `90020` | `280` |
| `mount310` | `97005` | `90020` | `310` |

> **📝 Note** : Un moddeur explique : « to make one new adaptive mount you'll need to make 5 new spells in Spell.dbc (each speed + catchall dummy spell) ».

### 4.3. Modifier le core C++

Dans `spell_generic.cpp`, ajoutez une nouvelle instance de `spell_gen_mount` pour votre monture :

```cpp
// Dans la fonction AddSC_spell_generic()
new spell_gen_mount("spell_tigre_spectral", 97000, 97001, 97002, 97003, 97004, 97005);
```

> **📝 Note** : Le premier paramètre est le nom du script, les suivants sont les IDs des sorts pour chaque palier de vitesse. Vous devez également enregistrer le sort principal (celui que le joueur apprend) dans `Spell.dbc` avec un effet **dummy** (`SPELL_EFFECT_DUMMY` = 3) qui sera intercepté par le script C++.

### 4.4. Recompiler le noyau

Après modification du C++, recompilez le noyau avec CMake et votre compilateur.

---

## Étape 5 : (Optionnel) Rendre la monture volante

Pour qu'une monture soit **volante**, elle doit avoir l'attribut `0x4000000` dans `Spell.dbc` (colonne 5, `Attributes`). Pour une monture **adaptative volante** (comme le Coursier céleste), vous devez également modifier la fonction `HandleMount` dans `spell_generic.cpp` pour retirer la restriction de zone :

```cpp
// Dans HandleMount, ajouter avant le switch :
canFly = true; // Toujours autorisé à voler
```

> **📝 Note** : Un tutoriel explique : « 在这几句代码前面添加 canFly = true;//此处将变量设置为始终true 去除飞行区域的判断 ». Cela permet à la monture de voler même dans les zones où le vol est normalement interdit (comme les royaumes de l'Est).

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

> **⚠️ Important** : Le nom du patch doit être supérieur au dernier patch existant dans votre dossier `Data`. Si vous avez `patch-3.MPQ`, nommez le vôtre `patch-4.MPQ` ou `patch-Z.MPQ`. Pour un patch localisé (français), utilisez `patch-frFR-4.MPQ` dans le dossier `Data/frFR`.

---

## Étape 7 : Redémarrer et tester

1. **Recompilez le noyau** (si vous avez modifié le C++).
2. **Redémarrez le serveur** pour que les modifications de la base de données soient prises en compte.
3. **Reconnectez-vous** avec le client patché.
4. **Obtenez l'objet** via la commande `.additem 90021` ou via un marchand.
5. **Utilisez l'objet** pour apprendre la monture.
6. **Lancez le sort** et vérifiez que la monture apparaît et que la vitesse est correcte.

---

## Dépannage

| Problème | Cause possible | Solution |
|----------|----------------|----------|
| La monture n'apparaît pas | `EffectMiscValue` incorrect dans `Spell.dbc` | Vérifiez que la colonne 108 pointe vers l'ID de la créature. |
| La monture est invisible | `modelid` incorrect dans `creature_template` | Vérifiez le `DisplayID` dans `CreatureDisplayInfo.dbc`. |
| La vitesse n'est pas augmentée | Aura `32` manquante ou incorrecte | Vérifiez `EffectApplyAuraName` (colonne 108) et `EffectBasePoints` (colonne 75). |
| L'objet n'apprend pas la monture | `spelltrigger_1` incorrect | Utilisez `6` pour l'apprentissage, `0` pour l'invocation directe. |
| Le joueur ne peut pas monter | Compétence de monte manquante | Le joueur doit avoir la compétence de monte requise (75, 150, 225, 300). |
| La monture ne vole pas | Attribut `0x4000000` manquant | Ajoutez l'attribut dans `Spell.dbc` (colonne 5) et recompilez. |
| Le serveur plante | Conflit d'ID ou entrée DBC manquante | Vérifiez les IDs et les logs serveur. |

---

## Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| Créature-monture | `90020` | `creature_template` |
| Sort d'invocation | `97000` | `Spell.dbc` |
| Objet d'apprentissage | `90021` | `Item.dbc` + `item_template` |
| Aura de vitesse montée | `32` | `Spell.dbc` (colonne 108) |
| Effet d'invocation de monture | `78` | `Spell.dbc` (colonne 72) |

---

## Conclusion

Vous savez maintenant créer une monture personnalisée dans AzerothCore 3.3.5. Le processus implique la modification de `creature_template` (côté serveur), `Spell.dbc` et `Item.dbc` (côté client), et pour les montures adaptatives, une modification du noyau C++ (`spell_generic.cpp`). Les concepts clés sont l'effet `78` (`SPELL_EFFECT_APPLY_MOUNT`), l'aura `32` (`SPELL_AURA_MOD_INCREASE_MOUNTED_SPEED`), et le champ `spelltrigger_1 = 6` pour l'apprentissage. Pour les montures volantes, ajoutez l'attribut `0x4000000` dans `Spell.dbc`. Testez toujours progressivement et consultez les logs serveur en cas d'erreur.

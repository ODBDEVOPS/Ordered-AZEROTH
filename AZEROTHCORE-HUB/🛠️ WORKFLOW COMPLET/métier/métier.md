# Guide complet : Créer un métier (Profession) personnalisé dans AzerothCore 3.3.5

## Introduction

Ce guide détaille étape par étape la création d'un **métier (profession) entièrement personnalisé** pour AzerothCore 3.3.5. Contrairement à la création d'une simple recette de métier existant, il s'agit ici de définir une **nouvelle compétence** (SkillLine) avec ses propres sorts de progression (Apprenti, Compagnon, Expert, Artisan, Maître), ses recettes, son entraîneur et son interface dans le livre de compétences.

> **⚠️ Avertissement** : Toute modification des fichiers DBC nécessite la création d'un patch MPQ côté client. Les modifications de base de données sont appliquées côté serveur et ne nécessitent pas de patch client. Sauvegardez toujours vos fichiers avant modification.

---

## Prérequis

- **Éditeur DBC** : MyDBCEditor, DBC Editor, ou équivalent.
- **Client de base de données** : HeidiSQL, MySQL Workbench, etc.
- **Fichiers DBC de référence** : `SkillLine.dbc`, `SkillLineAbility.dbc`, `SkillRaceClassInfo.dbc`, `Spell.dbc`, `Item.dbc`.
- **Outil MPQ** : Ladik's MPQ Editor.
- **Accès serveur** : Base de données `world` et redémarrage possible.

---

## Vue d'ensemble du processus

La création d'un métier personnalisé implique **six éléments interconnectés** :

1. **SkillLine.dbc** (client) — Définit la compétence elle-même (nom, icône, catégorie).
2. **SkillRaceClassInfo.dbc** (client) — Définit quelles races et classes peuvent apprendre le métier.
3. **Spell.dbc** (client) — Définit les sorts de progression (Apprenti, Compagnon, etc.) et les recettes.
4. **SkillLineAbility.dbc** (client) — Lie tous les sorts à la compétence.
5. **creature_template** + **npc_trainer** (serveur) — Crée l'entraîneur qui enseigne le métier.
6. **playercreateinfo_skills** (serveur, optionnel) — Donne le métier aux nouveaux personnages.

### Schéma des relations

```
┌─────────────────────────┐
│     SkillLine.dbc        │  ← Définit le métier (nom, icône)
│  ID: 1002               │
│  Name: "Runeforging"    │
└───────────┬─────────────┘
            │  référencé par
            ▼
┌─────────────────────────┐
│ SkillRaceClassInfo.dbc   │  ← Qui peut apprendre (races/classes)
│  SkillLine: 1002        │
│  ChrRaces: 0 (toutes)   │
│  ChrClasses: 0 (toutes) │
└───────────┬─────────────┘
            │  lié à
            ▼
┌─────────────────────────┐
│   SkillLineAbility.dbc   │  ← Lie sorts ↔ compétence
│  SkillLine: 1002        │
│  Spell: 95000           │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│       Spell.dbc          │  ← Sorts de progression + recettes
│  ID: 95000              │
│  Effect: 24 (CreateItem)│
│  Name: "Apprenti Runeforging"│
└─────────────────────────┘
```

---

## Étape 1 : Définir la compétence dans SkillLine.dbc

Le fichier `SkillLine.dbc` contient toutes les compétences du jeu. C'est ici que vous définissez **le nom** et les propriétés de base de votre métier.

### 1.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique de la compétence. |
| 2 | **CategoryId** | iRefID | Référence à `SkillLineCategory.dbc` (catégorie du métier). |
| 3 | **SkillCostId** | Integer | Référence à `SkillCostsData.dbc` (coût d'apprentissage). |
| 4 | **Name** | String + Loc | Nom du métier (ex: « Runeforging »). |
| 5 | **Description** | String + Loc | Description du métier. |
| 6 | **SpellIcon** | iRefID | Référence à `SpellIcon.dbc` (icône du métier). |
| 7 | **Verb** | String + Loc | Verbe utilisé dans l'interface (ex: « Forging »). |
| 8 | **CanLink** | Integer | `1` si le métier a des recettes (profession with recipes). |

### 1.2. Exemple concret

**Métier « Runeforging » (Forge runique)**

- **ID** : `1002` (doit être unique et non utilisé)
- **CategoryId** : `11` (catégorie « Professions »)
- **SkillCostId** : `0` (ou référence à un coût existant)
- **Name** : `Runeforging`
- **Description** : `La forge runique permet de graver des runes sur les armes.`
- **SpellIcon** : `1` (ou une icône existante)
- **Verb** : `Forging`
- **CanLink** : `1` (car ce métier a des recettes)

> **📝 Note** : Pour trouver un ID unique, utilisez la commande `.lookup skill` ou vérifiez le contenu de `SkillLine.dbc`. Les IDs des métiers existants sont : Alchimie = 171, Forge = 164, Travail du cuir = 165, Couture = 197, Ingénierie = 202, Joaillerie = 755, Calligraphie = 773, Enchantement = 333.

---

## Étape 2 : Définir les races et classes dans SkillRaceClassInfo.dbc

Le fichier `SkillRaceClassInfo.dbc` détermine **quelles races et classes** peuvent apprendre votre métier.

### 2.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 0 | **ID** | Integer | Identifiant unique de l'entrée. |
| 1 | **SkillLine** | iRefID | Référence à l'ID dans `SkillLine.dbc`. |
| 2 | **ChrRaces** | iRefMask | Bitmask des races autorisées (0 = toutes). |
| 3 | **ChrClasses** | iRefMask | Bitmask des classes autorisées (0 = toutes). |
| 4 | **Flags** | Integer | Drapeaux (voir ci-dessous). |
| 5 | **RequLvl** | Integer | Niveau minimum pour apprendre le métier. |
| 6 | **SkillTierId** | iRefID | Référence à `SkillTiers.dbc` (paliers de compétence). |

### 2.2. Drapeaux (Flags)

| Drapeau | Valeur | Description |
|---------|--------|-------------|
| `SKILL_FLAG_ALWAYS_MAX_VALUE` | `0x01` | La compétence est toujours au maximum. |
| `SKILL_FLAG_ABANDONABLE` | `0x02` | Le joueur peut abandonner le métier. |
| `SKILL_FLAG_IS_PROFESSION` | `0x04` | C'est une profession principale. |
| `SKILL_FLAG_NOT_TRAINABLE` | `0x08` | Non apprenable via entraîneur. |

### 2.3. Exemple concret

**Métier « Runeforging » accessible à toutes les races et classes**

- **ID** : `1002` (peut être identique à l'ID de SkillLine)
- **SkillLine** : `1002`
- **ChrRaces** : `0` (toutes les races)
- **ChrClasses** : `0` (toutes les classes)
- **Flags** : `0x06` (IS_PROFESSION + ABANDONABLE)
- **RequLvl** : `0` (accessible dès le niveau 1)
- **SkillTierId** : `0` (pas de palier spécial)

> **📝 Note** : Pour restreindre le métier à certaines races (par exemple, uniquement aux Nains et aux Gnomes), utilisez les bitmasks appropriés. Le bitmask des races est : Humain = 1, Orc = 2, Nain = 4, Elfe de la nuit = 8, Mort-vivant = 16, Tauren = 32, Gnome = 64, Troll = 128, etc.

---

## Étape 3 : Créer les sorts de progression dans Spell.dbc

Chaque métier possède des sorts de progression : Apprenti, Compagnon, Expert, Artisan et Maître. Vous devez créer ces cinq sorts (ou plus selon votre design).

### 3.1. Structure d'un sort de progression

Un sort de progression (par exemple « Apprenti Runeforging ») est un sort passif qui augmente la compétence maximale du joueur.

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `95000` | ID unique du sort. |
| 5 | **Attributes** | `0x00000040` | `SPELL_ATTR0_PASSIVE` (sort passif). |
| 18-19 | **TargetType** | `0` | Aucune cible. |
| 72 | **Effect_1** | `118` | `SPELL_EFFECT_SKILL` (augmente la compétence). |
| 75 | **EffectBasePoints** | `75` | Valeur de la compétence (ici 75 pour Apprenti). |
| 108 | **EffectItemType** | `1002` | ID de la compétence (référence `SkillLine.dbc`). |
| 128 | **SpellIconID** | `1` | Icône du sort. |
| 131-162 | **Name** | `Apprenti Runeforging` | Nom du sort. |
| 171-178 | **Description** | `Permet d'apprendre la Runeforging (75).` | Description. |

### 3.2. Exemple concret : les 5 sorts de progression

| Sort | ID | EffectBasePoints (colonne 75) | Nom |
|------|-----|-------------------------------|-----|
| Apprenti | `95000` | `75` | `Apprenti Runeforging` |
| Compagnon | `95001` | `150` | `Compagnon Runeforging` |
| Expert | `95002` | `225` | `Expert Runeforging` |
| Artisan | `95003` | `300` | `Artisan Runeforging` |
| Maître | `95004` | `450` | `Maître Runeforging` |

> **📝 Note** : La colonne `EffectItemType` (108) doit contenir l'ID de la compétence (`1002`). Le sort utilise l'effet `118` (`SPELL_EFFECT_SKILL`) pour augmenter la compétence maximale du joueur.

### 3.3. Créer des recettes pour le métier

Une fois les sorts de progression créés, vous pouvez créer des recettes. Une recette est un sort qui crée un objet à partir de réactifs (voir le guide précédent sur les recettes de métier pour plus de détails).

**Exemple de recette :**

- **ID** : `95100`
- **Effect_1** : `24` (SPELL_EFFECT_CREATE_ITEM)
- **EffectBasePoints** : `1`
- **EffectItemType** : `95001` (objet fabriqué)
- **Reagent** : `100001` x `5`
- **Name** : `Runeforging : Lame runique`


## Étape 4 : Lier les sorts à la compétence dans SkillLineAbility.dbc

Ce fichier est **essentiel** : il lie chaque sort (progression et recettes) à la compétence.

### 4.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique de l'entrée. |
| 2 | **SkillLine** | iRefID | Référence à l'ID dans `SkillLine.dbc`. |
| 3 | **Spell** | iRefID | Référence à l'ID du sort dans `Spell.dbc`. |
| 4 | **RaceMask** | BitMask | Races autorisées (0 = toutes). |
| 5 | **ClassMask** | BitMask | Classes autorisées (0 = toutes). |
| 6 | **ExcludeRace** | BitMask | Races exclues. |
| 7 | **ExcludeClass** | BitMask | Classes exclues. |
| 8 | **MinSkillLineRank** | Integer | Rang minimum de compétence pour utiliser le sort. |
| 9 | **SupercededBySpell** | iRefID | Sort qui remplace celui-ci (pour les rangs). |
| 10 | **AcquireMethod** | Integer | Méthode d'acquisition. |
| 11 | **TrivialSkillLineRankHigh** | Integer | Rang où la recette devient grise (max). |
| 12 | **TrivialSkillLineRankLow** | Integer | Rang où la recette devient jaune (min). |

### 4.2. Exemple concret

**Liaison des sorts de progression :**

| ID | SkillLine | Spell | MinSkillLineRank | TrivialHigh | TrivialLow |
|----|-----------|-------|------------------|-------------|------------|
| 95000 | 1002 | 95000 | 1 | 75 | 50 |
| 95001 | 1002 | 95001 | 50 | 150 | 100 |
| 95002 | 1002 | 95002 | 125 | 225 | 175 |
| 95003 | 1002 | 95003 | 200 | 300 | 250 |
| 95004 | 1002 | 95004 | 275 | 450 | 350 |

**Liaison d'une recette :**

| ID | SkillLine | Spell | MinSkillLineRank | TrivialHigh | TrivialLow |
|----|-----------|-------|------------------|-------------|------------|
| 95100 | 1002 | 95100 | 1 | 75 | 50 |

> **📝 Note** : La colonne `MinSkillLineRank` détermine le niveau de compétence minimum requis pour utiliser le sort. Pour les sorts de progression, il s'agit du rang auquel le sort devient disponible. Pour les recettes, il s'agit du niveau requis pour fabriquer l'objet.

---

## Étape 5 : Créer l'entraîneur (creature_template + npc_trainer)

Pour que les joueurs puissent apprendre votre métier, vous devez créer un PNJ entraîneur.

### 5.1. creature_template

Insérez une nouvelle ligne dans la table `creature_template` :

```sql
INSERT INTO `creature_template` (
    `entry`, `name`, `subname`, `minlevel`, `maxlevel`, `faction`, `npcflag`
) VALUES (
    90010, 'Maître Runeforging', 'Entraîneur de Runeforging', 80, 80, 35, 16
);
```

| Champ | Valeur | Description |
|-------|--------|-------------|
| `entry` | `90010` | ID unique du PNJ. |
| `name` | `Maître Runeforging` | Nom du PNJ. |
| `subname` | `Entraîneur de Runeforging` | Sous-titre. |
| `faction` | `35` | Faction amicale avec les joueurs. |
| `npcflag` | `16` | `UNIT_NPC_FLAG_TRAINER` (entraîneur). |

### 5.2. npc_trainer

Ajoutez les sorts de progression dans la table `npc_trainer` :

```sql
INSERT INTO `npc_trainer` (
    `entry`, `spell`, `spellcost`, `reqskill`, `reqskillvalue`, `reqlevel`
) VALUES
(90010, 95000, 100, 0, 0, 0),
(90010, 95001, 500, 1002, 75, 10),
(90010, 95002, 2500, 1002, 150, 25),
(90010, 95003, 10000, 1002, 225, 40),
(90010, 95004, 50000, 1002, 300, 55);
```

| Champ | Description |
|-------|-------------|
| `entry` | ID du PNJ entraîneur. |
| `spell` | ID du sort de progression. |
| `spellcost` | Coût en cuivre. |
| `reqskill` | Compétence requise (0 pour le premier rang). |
| `reqskillvalue` | Niveau de compétence requis. |
| `reqlevel` | Niveau du joueur requis. |

---

## Étape 6 : (Optionnel) Donner le métier aux nouveaux personnages

Pour que les nouveaux personnages aient automatiquement le métier, ajoutez une entrée dans `playercreateinfo_skills`.

```sql
INSERT INTO `playercreateinfo_skills` (
    `raceMask`, `classMask`, `skill`, `rank`, `comment`
) VALUES (
    0, 0, 1002, 1, 'Runeforging'
);
```

| Champ | Description |
|-------|-------------|
| `raceMask` | `0` = toutes les races. |
| `classMask` | `0` = toutes les classes. |
| `skill` | ID de la compétence (`1002`). |
| `rank` | Rang initial (`1`). |

---

## Étape 7 : Créer le patch MPQ côté client

1. Créez la structure :
   ```
   patch-4/
   └── DBFilesClient/
       ├── SkillLine.dbc
       ├── SkillRaceClassInfo.dbc
       ├── SkillLineAbility.dbc
       ├── Spell.dbc
       └── Item.dbc
   ```
2. Placez les fichiers modifiés dans `DBFilesClient`.
3. Créez `patch-4.MPQ` avec Ladik's MPQ Editor.
4. Placez-le dans le dossier `Data` du client.

> **⚠️ Important** : Le nom du patch doit être supérieur au dernier patch existant dans votre dossier `Data`. Par exemple, si vous avez `patch-3.MPQ`, nommez le vôtre `patch-4.MPQ` ou `patch-Z.MPQ`.

---

## Étape 8 : Redémarrer et tester

1. **Redémarrez votre serveur** pour que les modifications de la base de données soient prises en compte.
2. **Reconnectez-vous** au jeu avec le client patché.
3. **Parlez à l'entraîneur** et apprenez le métier.
4. **Vérifiez** que le métier apparaît dans votre livre de compétences (touche `K`).
5. **Testez** les recettes en utilisant les réactifs appropriés.

---

## Dépannage

| Problème | Cause possible | Solution |
|----------|----------------|----------|
| Le métier n'apparaît pas dans le livre | `SkillLine.dbc` non patché ou ID invalide | Vérifiez le MPQ et l'ID. |
| L'entraîneur ne propose pas le métier | `npc_trainer` mal configuré | Vérifiez les entrées et `reqskill`. |
| Le sort de progression ne fonctionne pas | `SkillLineAbility.dbc` manquant | Ajoutez les entrées pour chaque sort. |
| Erreur « compétence inconnue » | `SkillRaceClassInfo.dbc` manquant | Vérifiez que l'entrée existe. |
| Le métier n'apparaît pas dans l'interface | `CanLink` = 0 dans `SkillLine.dbc` | Mettez `CanLink` = 1. |
| Les recettes ne s'affichent pas | `SkillLineAbility.dbc` manquant pour les recettes | Ajoutez les entrées correspondantes. |

---

## Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| Compétence (SkillLine) | `1002` | `SkillLine.dbc` |
| SkillRaceClassInfo | `1002` | `SkillRaceClassInfo.dbc` |
| Sort de progression (Apprenti) | `95000` | `Spell.dbc` |
| Sort de progression (Maître) | `95004` | `Spell.dbc` |
| Recette | `95100` | `Spell.dbc` |
| Objet fabriqué | `95001` | `Item.dbc` + `item_template` |
| PNJ entraîneur | `90010` | `creature_template` + `npc_trainer` |

---

## Conclusion

Vous savez maintenant créer un métier entièrement personnalisé dans AzerothCore 3.3.5. Le processus implique la modification de plusieurs fichiers DBC (`SkillLine.dbc`, `SkillRaceClassInfo.dbc`, `SkillLineAbility.dbc`, `Spell.dbc`, `Item.dbc`) et de tables de base de données (`creature_template`, `npc_trainer`, `playercreateinfo_skills`). Les sorts de progression (Apprenti à Maître) et les recettes sont liés à la compétence via `SkillLineAbility.dbc`. Testez toujours progressivement et consultez les logs serveur en cas d'erreur.

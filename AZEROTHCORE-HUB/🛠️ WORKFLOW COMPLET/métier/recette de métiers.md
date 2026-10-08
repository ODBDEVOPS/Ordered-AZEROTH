# Guide complet : Créer une recette de métier (Profession) dans AzerothCore 3.3.5

## Introduction

Ce guide détaille étape par étape la création d'une **recette de métier** (Crafting Recipe) personnalisée pour AzerothCore 3.3.5. Contrairement aux enchantements, les recettes de métier (Forge, Alchimie, Travail du cuir, Couture, Ingénierie, Joaillerie, Calligraphie, etc.) créent un **nouvel objet** à partir de réactifs. Vous apprendrez à définir l'objet fabriqué, le sort de fabrication, la liaison avec la compétence, et la méthode d'apprentissage.

> **⚠️ Avertissement** : Toute modification des fichiers DBC nécessite la création d'un patch MPQ côté client. Les modifications de base de données sont appliquées côté serveur et ne nécessitent pas de patch client. Sauvegardez toujours vos fichiers avant modification.

---

## Prérequis

- **Éditeur DBC** : MyDBCEditor, DBC Editor, ou équivalent.
- **Client de base de données** : HeidiSQL, MySQL Workbench, etc.
- **Fichiers DBC de référence** : `Spell.dbc`, `Item.dbc`, `SkillLineAbility.dbc`, `SkillLine.dbc`.
- **Outil MPQ** : Ladik's MPQ Editor.
- **Accès serveur** : Base de données `world` et redémarrage possible.

---

## Vue d'ensemble du processus

La création d'une recette de métier implique **cinq éléments interconnectés** :

1. **Item.dbc** (client) + **item_template** (serveur) — Définissent l'objet fabriqué et l'objet « plan » (la recette elle-même).
2. **Spell.dbc** — Définit le sort de fabrication qui crée l'objet à partir des réactifs.
3. **SkillLineAbility.dbc** — Lie le sort de fabrication à la compétence (ex: Forge).
4. **npc_trainer** ou **item d'apprentissage** — Permet au joueur d'apprendre la recette.
5. **spell_required** (optionnel) — Définit des prérequis (spécialisation, autre sort).

### Schéma des relations

```
┌─────────────────────────┐
│   Spell.dbc (fabrication)│  ← Sort qui crée l'objet
│  ID: 95000              │
│  Effect: 24 (CreateItem)│
│  ItemType: 95001        │  ← ID de l'objet fabriqué
│  Reagent: 100001 x 5    │  ← ID des réactifs
└───────────┬─────────────┘
            │  lié à
            ▼
┌─────────────────────────┐
│  SkillLineAbility.dbc    │  ← Lie le sort à la compétence
│  Spell: 95000           │
│  SkillLine: 164 (Forge) │
└─────────────────────────┘
            │
            ▼
┌─────────────────────────┐
│  Plan/Recette (objet)   │  ← Apprise par le joueur
│  item_template 95002    │
│  spellid_1: 95000       │
└─────────────────────────┘
```

---

## Étape 1 : Créer l'objet fabriqué (Item.dbc + item_template)

### 1.1. Item.dbc (côté client)

Dupliquez une ligne d'objet existant dans `Item.dbc` et modifiez son **ID** (colonne 1) pour correspondre à l'`entry` de votre objet (ex: `95001`). Vérifiez que `class` (colonne 2), `subclass` (colonne 3) et `DisplayInfoID` (colonne 6) sont valides.

### 1.2. item_template (côté serveur)

Insérez une nouvelle ligne dans la table `item_template` de la base `world` :

```sql
INSERT INTO `item_template` (
    `entry`, `class`, `subclass`, `name`, `displayid`, `Quality`,
    `BuyCount`, `BuyPrice`, `SellPrice`, `InventoryType`, `ItemLevel`,
    `RequiredLevel`, `bonding`, `description`
) VALUES (
    95001, 2, 7, 'Épée runique flamboyante', 12345, 4,
    1, 500000, 125000, 21, 80,
    80, 1, 'Forgée avec les flammes du Noyau.'
);
```

| Champ | Valeur | Description |
|-------|--------|-------------|
| `entry` | `95001` | ID unique de l'objet. |
| `class` | `2` | Arme (Weapon). |
| `subclass` | `7` | Épée à deux mains (Two-Handed Sword). |
| `name` | `Épée runique flamboyante` | Nom de l'objet. |
| `displayid` | `12345` | ID d'affichage (référence `ItemDisplayInfo.dbc`). |
| `Quality` | `4` | Épique. |
| `InventoryType` | `21` | Main droite (Main Hand). |
| `ItemLevel` | `80` | Niveau d'objet. |
| `RequiredLevel` | `80` | Niveau minimum du joueur. |
| `bonding` | `1` | Lié quand équipé. |

---

## Étape 2 : Créer le sort de fabrication (Spell.dbc)

Le sort de fabrication est ce qui est lancé lorsque le joueur utilise la recette. Il consomme les réactifs et crée l'objet.

### 2.1. Champs essentiels

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `95000` | ID unique du sort. |
| 5 | **Attributes** | `0x00010000` | `SPELL_ATTR0_IS_ABILITY` (compétence). |
| 18-19 | **TargetType** | `0` | Aucune cible (le sort se lance sur soi-même). |
| 28-30 | **Reagent** | `100001, 100002, 0` | IDs des réactifs nécessaires. |
| 31-33 | **ReagentCount** | `5, 2, 0` | Quantités des réactifs. |
| 39 | **BaseLevel** | `1` | Niveau de base du sort. |
| 72 | **Effect_1** | `24` | `SPELL_EFFECT_CREATE_ITEM` (créer un objet). |
| 75 | **EffectBasePoints** | `1` | Nombre d'objets créés (1). |
| 112 | **EffectItemType** | `95001` | **ID de l'objet fabriqué** (référence `Item.dbc` / `item_template`). |
| 128 | **SpellIconID** | `1` | Icône du sort. |
| 131-162 | **Name** | `Forge : Épée runique flamboyante` | Nom du sort. |
| 171-178 | **Description** | `Crée une épée runique flamboyante.` | Description. |
| 237 | **SpellFamilyName** | `0` | Famille générique (ou `4` pour Warrior si arme de guerre). |
| 254 | **SchoolMask** | `1` | École physique (ou `4` pour Feu). |

> **📝 Note importante** : Pour un sort de métier, l'effet `24` (`SPELL_EFFECT_CREATE_ITEM`) utilise `EffectItemType` (colonne 112) pour définir l'objet créé et `EffectBasePoints` (colonne 75) pour le nombre d'objets créés.

### 2.2. Exemple concret

- **ID** : `95000`
- **Reagent 1** : `100001` (Minerai de titane) x `5`
- **Reagent 2** : `100002` (Étoffe de lune) x `2`
- **Effect_1** : `24`
- **EffectBasePoints** : `1`
- **EffectItemType** : `95001`
- **Name** : `Forge : Épée runique flamboyante`

---

## Étape 3 : Lier le sort à la compétence (SkillLineAbility.dbc)

Ce fichier DBC est **essentiel** : il lie le sort de fabrication à une compétence (Forge, Alchimie, etc.) pour qu'il apparaisse dans l'interface du métier.

### 3.1. Structure

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `95000` | ID unique de l'entrée (peut être identique au sort). |
| 2 | **SkillLine** | `164` | ID de la compétence (référence `SkillLine.dbc`). `164` = Forge. |
| 3 | **Spell** | `95000` | ID du sort de fabrication. |
| 4 | **RaceMask** | `0` | Toutes races. |
| 5 | **ClassMask** | `0` | Toutes classes. |
| 6 | **MinSkillLineRank** | `1` | Compétence minimale requise. |
| 7 | **SupercededBySpell** | `0` | Sort remplacé (pour les rangs). |
| 8 | **AcquireMethod** | `0` | Méthode d'acquisition (0 = appris par recette). |
| 9 | **TrivialSkillLineRankHigh** | `450` | Rang où la recette devient grise (max). |
| 10 | **TrivialSkillLineRankLow** | `400` | Rang où la recette devient jaune (min). |

> **📝 Note** : La colonne 2 doit correspondre à l'ID de la compétence dans `SkillLine.dbc` (ex: `171` = Alchimie, `164` = Forge, `333` = Enchantement). La colonne 3 est l'ID du sort de fabrication.

### 3.2. IDs des compétences courantes

| Compétence | ID SkillLine |
|------------|--------------|
| Alchimie | `171` |
| Forge | `164` |
| Travail du cuir | `165` |
| Couture | `197` |
| Ingénierie | `202` |
| Joaillerie | `755` |
| Calligraphie | `773` |

---

## Étape 4 : Rendre la recette apprenable

Il existe **deux méthodes** pour qu'un joueur apprenne la recette.

### Méthode A : Via un PNJ entraîneur (npc_trainer)

Ajoutez une ligne dans la table `npc_trainer` :

```sql
INSERT INTO `npc_trainer` (
    `entry`, `spell`, `spellcost`, `reqskill`, `reqskillvalue`, `reqlevel`
) VALUES (
    12345, 95000, 50000, 164, 400, 75
);
```

| Champ | Valeur | Description |
|-------|--------|-------------|
| `entry` | `12345` | ID du PNJ entraîneur (Forgeron). |
| `spell` | `95000` | ID du sort de fabrication. |
| `spellcost` | `50000` | Coût en cuivre (5 pièces d'or). |
| `reqskill` | `164` | Compétence requise (Forge). |
| `reqskillvalue` | `400` | Niveau de compétence requis. |
| `reqlevel` | `75` | Niveau du joueur requis. |

### Méthode B : Via un objet « Plan » (Recette)

Créez un objet « Plan » que le joueur utilise pour apprendre la recette.

**item_template** :

```sql
INSERT INTO `item_template` (
    `entry`, `class`, `subclass`, `name`, `displayid`, `Quality`,
    `RequiredLevel`, `RequiredSkill`, `RequiredSkillRank`,
    `spellid_1`, `spelltrigger_1`, `spellcharges_1`, `bonding`, `flags`
) VALUES (
    95002, 9, 6, 'Plan : Épée runique flamboyante', 12346, 4,
    75, 164, 400,
    95000, 6, -1, 1, 0x02000000
);
```

| Champ | Valeur | Description |
|-------|--------|-------------|
| `class` | `9` | Recette (Recipe). |
| `subclass` | `6` | Forge (Blacksmithing). |
| `spellid_1` | `95000` | Sort de fabrication à apprendre. |
| `spelltrigger_1` | `6` | `ON_USE` (apprentissage). |
| `spellcharges_1` | `-1` | Consommé à l'utilisation. |
| `flags` | `0x02000000` | `ITEM_FLAG_PROFESSION_RECIPE` : ne peut être ramassé que si les prérequis sont remplis et si le joueur ne connaît pas déjà la recette. |

N'oubliez pas d'ajouter également une entrée correspondante dans `Item.dbc` (côté client) pour l'objet « Plan ».

---

## Étape 5 : (Optionnel) Configurer les systèmes avancés

### 5.1. skill_extra_item_template (création multiple)

Permet d'avoir une chance de créer plusieurs objets à la fois (comme les potions en Alchimie).

```sql
INSERT INTO `skill_extra_item_template` (
    `spellId`, `requiredSpecialization`, `additionalCreateChance`, `additionalMaxNum`
) VALUES (
    95000, 0, 20.0, 3
);
```

Cela donne 20% de chance de créer jusqu'à 3 objets supplémentaires.

### 5.2. skill_discovery_template (découverte)

Permet de « découvrir » une recette en craftant (utilisé uniquement par l'Alchimie).

```sql
INSERT INTO `skill_discovery_template` (
    `spellId`, `reqSpell`, `reqSkillValue`, `chance`
) VALUES (
    95000, 0, 400, 2.0
);
```

### 5.3. spell_required (prérequis)

Si votre recette nécessite une spécialisation ou un autre sort :

```sql
INSERT INTO `spell_required` (`spell_id`, `req_spell`) VALUES (95000, 9787);
```

---

## Étape 6 : Créer le patch MPQ côté client

1. Créez la structure :
   ```
   patch-4/
   └── DBFilesClient/
       ├── Spell.dbc
       ├── Item.dbc
       └── SkillLineAbility.dbc
   ```
2. Placez les fichiers modifiés dans `DBFilesClient`.
3. Créez `patch-4.MPQ` avec Ladik's MPQ Editor.
4. Placez-le dans le dossier `Data` du client.

---

## Étape 7 : Redémarrer et tester

1. Redémarrez le serveur.
2. Reconnectez-vous avec le client patché.
3. Apprenez la recette (via PNJ ou objet).
4. Ouvrez l'interface du métier, sélectionnez la recette, cliquez sur « Créer ».
5. Vérifiez que l'objet apparaît dans votre sac et que les réactifs sont consommés.

---

## Dépannage

| Problème | Cause possible | Solution |
|----------|----------------|----------|
| La recette n'apparaît pas dans le métier | `SkillLineAbility.dbc` manquant ou incorrect | Vérifiez la colonne 2 (SkillLine) et colonne 3 (Spell). |
| Le sort ne crée rien | `Effect_1` ≠ 24 ou `EffectItemType` incorrect | Vérifiez les colonnes 72 et 112. |
| Les réactifs ne sont pas consommés | Colonnes Reagent/ReagentCount vides | Remplissez les colonnes 28-33. |
| Le plan n'est pas apprenable | `spelltrigger_1` ≠ 6 | Utilisez `6` pour l'apprentissage. |
| Erreur « Vous ne pouvez pas apprendre cette recette » | `flags` du plan incorrect | Ajoutez `0x02000000` à `flags`. |
| L'objet fabriqué est invisible | `Item.dbc` non patché | Vérifiez le MPQ. |

---

## Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| Objet fabriqué | `95001` | `Item.dbc` + `item_template` |
| Sort de fabrication | `95000` | `Spell.dbc` |
| Liaison compétence | `95000` | `SkillLineAbility.dbc` |
| Plan (recette) | `95002` | `Item.dbc` + `item_template` |
| PNJ entraîneur | `12345` | `npc_trainer` |
| Réactif 1 | `100001` | `item_template` |
| Réactif 2 | `100002` | `item_template` |

---

## Conclusion

Vous savez maintenant créer une recette de métier complète dans AzerothCore 3.3.5. Le processus implique la modification de fichiers DBC (`Spell.dbc`, `Item.dbc`, `SkillLineAbility.dbc`) et de tables de base de données (`item_template`, `npc_trainer`, etc.). Les systèmes avancés comme `skill_extra_item_template` et `skill_discovery_template` vous permettent d'ajouter des mécaniques de craft supplémentaires. Testez toujours progressivement et consultez les logs serveur en cas d'erreur.

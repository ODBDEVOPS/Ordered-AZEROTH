# Guide complet : Créer une forme de druide personnalisée dans AzerothCore 3.3.5

## Introduction

Ce guide détaille étape par étape la création d'une **forme de druide personnalisée** (Shapeshift Form) pour AzerothCore 3.3.5. Contrairement à une classe ou un métier, la forme de druide est un **état de transformation** qui modifie le modèle du personnage, son type de créature, ses statistiques et lui donne accès à des sorts spécifiques (attaques de mêlée, fureur, etc.).

La difficulté principale réside dans le fait que **le noyau (core) AzerothCore contrôle directement les modèles des formes de druide** pour les formes principales (Ours, Loup, etc.) via la fonction `GetModelForForm()`. Les fichiers DBC `SpellShapeshiftForm.dbc` ne contrôlent pas entièrement l'apparence pour ces formes en 3.3.5.

> **⚠️ Avertissement** : La création d'une forme de druide personnalisée nécessite des modifications des fichiers DBC **et** une recompilation du noyau (core) pour les formes principales. Sauvegardez toujours vos fichiers avant modification.

---

## Prérequis

- **Éditeur DBC** : MyDBCEditor, DBC Editor, ou équivalent.
- **Client de base de données** : HeidiSQL, MySQL Workbench, etc.
- **Fichiers DBC de référence** : `SpellShapeshiftForm.dbc`, `Spell.dbc`, `SkillLineAbility.dbc`, `CreatureDisplayInfo.dbc`, `CreatureModelData.dbc`.
- **Outil MPQ** : Ladik's MPQ Editor.
- **Accès serveur** : Base de données `world` et possibilité de **recompiler le noyau** (pour les formes principales).
- **Modèle 3D** : Un fichier `.m2` et `.skin` pour le nouveau modèle (optionnel si vous réutilisez un modèle existant).

---

## Vue d'ensemble du processus

La création d'une forme de druide personnalisée implique **cinq éléments interconnectés** :

1. **SpellShapeshiftForm.dbc** (client) — Définit la forme elle-même (nom, icône, drapeaux, sorts prédéfinis).
2. **Spell.dbc** (client) — Définit le sort de transformation qui active la forme.
3. **SkillLineAbility.dbc** (client) — Lie le sort de forme à la compétence de druide.
4. **player_shapeshift_model** (serveur) — Définit le modèle utilisé pour chaque forme, selon la race et le genre.
5. **GetModelForForm()** (noyau C++) — Pour les formes principales (Ours, Loup), le noyau ignore les DBC et utilise cette fonction pour déterminer le modèle.

### Schéma des relations

```
┌─────────────────────────┐
│ SpellShapeshiftForm.dbc  │  ← Définit la forme
│  ID: 50                 │
│  Name: "Forme d'Araignée"│
│  Flags: 0x00000001      │
│  DisplayID_A: 12345     │  ← Modèle Alliance
│  DisplayID_H: 12346     │  ← Modèle Horde
└───────────┬─────────────┘
            │  référencé par
            ▼
┌─────────────────────────┐
│       Spell.dbc          │  ← Sort de transformation
│  ID: 97000              │
│  Effect_1: 6            │  ← SPELL_EFFECT_APPLY_AURA
│  EffectApplyAuraName: 36│  ← SPELL_AURA_MOD_SHAPESHIFT
│  EffectMiscValue: 50    │  ← ID de SpellShapeshiftForm
└───────────┬─────────────┘
            │  lié à
            ▼
┌─────────────────────────┐
│  SkillLineAbility.dbc    │  ← Lie le sort à la compétence
│  SkillLine: 573 (Druide)│
│  Spell: 97000           │
│  ClassMask: 1024        │  ← Druide
└─────────────────────────┘
            │
            ▼
┌─────────────────────────┐
│  player_shapeshift_model │  ← Modèle utilisé par le serveur
│  ShapeshiftID: 50       │
│  RaceID: 4 (Elfe Nuit)  │
│  ModelID: 12345         │
└─────────────────────────┘
```

---

## Étape 1 : Définir la forme dans SpellShapeshiftForm.dbc

Le fichier `SpellShapeshiftForm.dbc` contient toutes les formes et postures du jeu. C'est ici que vous définissez les propriétés de base de votre forme.

### 1.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique de la forme. |
| 2 | **ActionBar** | Integer | Barre d'action bonus (0 = aucune, 1 = barre de forme). |
| 3-19 | **Name** | Loc | Nom de la forme (ex: « Forme d'Araignée »). |
| 20 | **Flags** | BitMask | Drapeaux de la forme (voir ci-dessous). |
| 21 | **CreatureType** | Integer | Type de créature (`-1` = hérité de la race, `0` = aucun, `1` = Bête, `8` = Démon, etc.). |
| 22 | **SpellIcon** | iRefID | Icône de la forme (référence `SpellIcon.dbc`). |
| 23 | **combatRoundTime** | Integer | Temps de round de combat (seules les formes de druide ont une valeur, `{2500, 1000}`). |
| 24-25 | **DisplayID[4]** | Integer | Modèles `{Alliance, Horde}`. **Note** : pour les formes principales (Ours, Loup), ces colonnes sont ignorées par le noyau en 3.3.5. |
| 28-35 | **presetSpellID[8]** | iRefID | Sorts prédéfinis disponibles dans cette forme (ex: pour « Zombie », « Goule »). |

### 1.2. Drapeaux de forme (colonne 20)

| Drapeau | Valeur | Description |
|---------|--------|-------------|
| **Stance** | `0x00000001` | Simule ne pas être transformé (permet de monter, utiliser des objets, interagir) — utilisé par les postures de guerrier. |
| **Not Toggleable** | `0x00000002` | Empêche l'annulation manuelle de la forme. |
| **Persist On Death** | `0x00000004` | La forme persiste après la mort. |
| **Can Interact NPC** | `0x00000008` | Permet d'interagir avec les PNJ. |
| **Don't Use Weapon** | `0x00000010` | N'utilise pas d'arme (attaque à mains nues). |
| **Agility Attack Bonus** | `0x00000020` | Active le bonus d'attaque basé sur l'agilité (comme la forme de loup). |
| **Can Use Equipped Items** | `0x00000040` | Peut utiliser les objets équipés. |
| **Can Use Items** | `0x00000080` | Peut utiliser des objets. |
| **Don't Auto-Unshift** | `0x00000100` | Empêche le client de sortir automatiquement de la forme. |
| **Considered Dead** | `0x00000200` | Considéré comme mort (empêche la téléportation LFG). |
| **Can Only Cast Shapeshift Spells** | `0x00000400` | Limite les sorts à ceux de la forme (comme pour le loup). |

### 1.3. Exemple concret

**Forme « Forme d'Araignée »**

- **ID** : `50` (doit être unique, les IDs 1-29 sont utilisés par les formes existantes)
- **ActionBar** : `1` (barre d'action bonus)
- **Name** : `Forme d'Araignée`
- **Flags** : `0x00000010` (Don't Use Weapon) | `0x00000020` (Agility Attack Bonus) | `0x00000200` (Considered Dead)
- **CreatureType** : `1` (Bête)
- **SpellIcon** : `1`
- **DisplayID_A** : `12345` (modèle Alliance — peut être ignoré pour les formes principales)
- **DisplayID_H** : `12346` (modèle Horde)
- **presetSpellID** : `97001` (Morsure empoisonnée), `97002` (Toile), etc.

---

## Étape 2 : Créer le sort de transformation dans Spell.dbc

Le sort de transformation active la forme. Il utilise l'aura `SPELL_AURA_MOD_SHAPESHIFT` (36) et référence l'ID de la forme dans `SpellShapeshiftForm.dbc` via `EffectMiscValue`.

### 2.1. Champs essentiels

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97000` | ID unique du sort. |
| 5 | **Attributes** | `0x00000010` | `SPELL_ATTR0_IS_ABILITY` (compétence). |
| 18-19 | **TargetType** | `0` | Aucune cible (sort sur soi-même). |
| 72 | **Effect_1** | `6` | `SPELL_EFFECT_APPLY_AURA` (appliquer une aura). |
| 108 | **EffectApplyAuraName** | `36` | `SPELL_AURA_MOD_SHAPESHIFT` (modifier la forme). |
| 113 | **EffectMiscValue** | `50` | **ID de la forme** (référence `SpellShapeshiftForm.dbc`). |
| 128 | **SpellIconID** | `1` | Icône du sort. |
| 131-162 | **Name** | `Forme d'Araignée` | Nom du sort. |
| 171-178 | **Description** | `Vous transforme en araignée.` | Description. |

### 2.2. Exemple concret

- **ID** : `97000`
- **Effect_1** : `6` (APPLY_AURA)
- **EffectApplyAuraName** : `36` (MOD_SHAPESHIFT)
- **EffectMiscValue** : `50` (ID de la forme)
- **Name** : `Forme d'Araignée`

---

## Étape 3 : Lier le sort à la compétence de druide (SkillLineAbility.dbc)

Ce fichier lie le sort de transformation à la compétence de classe du druide (SkillLine `573`).

### 3.1. Structure

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97000` | ID unique de l'entrée. |
| 2 | **SkillLine** | `573` | Compétence de druide (référence `SkillLine.dbc`). |
| 3 | **Spell** | `97000` | ID du sort de transformation. |
| 4 | **RaceMask** | `0` | Toutes races. |
| 5 | **ClassMask** | `1024` | **Druide** (`1 << (11 - 1) = 1024`). |
| 8 | **MinSkillLineRank** | `1` | Rang minimum. |
| 10 | **AcquireMethod** | `0` | Méthode d'acquisition. |

> **📝 Note** : Le bitmask de classe pour le druide est `1024` (ID de classe 11 dans `ChrClasses.dbc`).

---

## Étape 4 : Définir le modèle côté serveur (player_shapeshift_model)

La table `player_shapeshift_model` définit quel modèle est utilisé pour chaque forme, selon la race, la personnalisation et le genre du personnage.

### 4.1. Structure de la table

| Champ | Type | Description |
|-------|------|-------------|
| **ShapeshiftID** | TINYINT | ID de la forme (référence `SpellShapeshiftForm.dbc`). |
| **RaceID** | TINYINT | ID de la race (référence `ChrRaces.dbc`). |
| **CustomizationID** | TINYINT | ID de personnalisation (couleur de peau/cheveux). |
| **GenderID** | TINYINT | `0` = Masculin, `1` = Féminin, `2` = Tous. |
| **ModelID** | INT | ID du modèle (référence `CreatureDisplayInfo.dbc`). |

### 4.2. Exemple concret

**Forme « Forme d'Araignée » (ID 50) pour un Elfe de la Nuit (Race 4) masculin**

```sql
INSERT INTO `player_shapeshift_model` (
    `ShapeshiftID`, `RaceID`, `CustomizationID`, `GenderID`, `ModelID`
) VALUES (
    50, 4, 0, 0, 12345
);
```

> **📝 Note** : Pour les formes principales (Ours, Loup), le noyau AzerothCore utilise la fonction `GetModelForForm()` qui ignore cette table et applique des règles spécifiques. Pour les formes secondaires (Voyage, Aquatique, Arbre de vie, etc.), cette table est utilisée.

---

## Étape 5 : Modifier le noyau C++ (pour les formes principales)

Pour les formes principales (Ours, Loup, Ours redoutable), le noyau contrôle directement les modèles via la fonction `GetModelForForm()`. Vous devez modifier cette fonction dans le code source pour utiliser vos modèles personnalisés.

### 5.1. Localiser la fonction

La fonction se trouve dans `src/server/game/Entities/Unit/Unit.cpp` (ou un fichier similaire selon votre version).

### 5.2. Exemple de modification

```cpp
uint32 Unit::GetModelForForm(ShapeshiftForm form) const
{
    // ... code existant ...
    
    switch (form)
    {
        case FORM_CAT:
            // Modèle personnalisé pour la forme de loup
            return 12345; // Votre ModelID personnalisé
        case FORM_BEAR:
            // Modèle personnalisé pour la forme d'ours
            return 12346; // Votre ModelID personnalisé
        case FORM_DIREBEAR:
            return 12347;
        // ...
    }
    
    // ... code existant ...
}
```

> **⚠️ Important** : Cette modification nécessite une **recompilation complète du noyau**. Elle est uniquement nécessaire si vous souhaitez modifier les formes principales. Pour les formes secondaires, la table `player_shapeshift_model` suffit.

---

## Étape 6 : Créer le patch MPQ côté client

1. Créez la structure :
   ```
   patch-4/
   └── DBFilesClient/
       ├── SpellShapeshiftForm.dbc
       ├── Spell.dbc
       └── SkillLineAbility.dbc
   ```
2. Placez les fichiers modifiés dans `DBFilesClient`.
3. Créez `patch-4.MPQ` avec Ladik's MPQ Editor.
4. Placez-le dans le dossier `Data` du client.

---

## Étape 7 : Redémarrer et tester

1. **Recompilez le noyau** (si vous avez modifié le C++).
2. **Redémarrez le serveur** pour appliquer les modifications de la base de données.
3. **Reconnectez-vous** avec le client patché.
4. **Apprenez le sort** via la commande `.learn 97000` ou via un PNJ entraîneur.
5. **Activez la forme** et vérifiez que le modèle, les sorts et les statistiques sont corrects.

---

## Dépannage

| Problème | Cause possible | Solution |
|----------|----------------|----------|
| La forme ne s'active pas | `Spell.dbc` mal configuré | Vérifiez `EffectApplyAuraName` (36) et `EffectMiscValue`. |
| Le modèle ne change pas | `player_shapeshift_model` manquant ou `GetModelForForm()` non modifié | Vérifiez la table et le code C++. |
| Les sorts de la forme ne s'affichent pas | `presetSpellID` non configuré dans `SpellShapeshiftForm.dbc` | Remplissez les colonnes 28-35. |
| Le personnage reste coincé dans la forme | `Flags` incorrects | Vérifiez les drapeaux (Not Toggleable, Don't Auto-Unshift). |
| Erreur de compilation | Syntaxe C++ incorrecte | Vérifiez le code et recompilez. |

---

## Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| Forme (SpellShapeshiftForm) | `50` | `SpellShapeshiftForm.dbc` |
| Sort de transformation | `97000` | `Spell.dbc` |
| Compétence de druide | `573` | `SkillLine.dbc` |
| Bitmask de classe (Druide) | `1024` | `1 << (11 - 1)` |
| Modèle personnalisé | `12345` | `CreatureDisplayInfo.dbc` |

---

## Conclusion

Vous savez maintenant créer une forme de druide personnalisée dans AzerothCore 3.3.5. Le processus implique la modification de fichiers DBC (`SpellShapeshiftForm.dbc`, `Spell.dbc`, `SkillLineAbility.dbc`), de la table `player_shapeshift_model`, et pour les formes principales, une modification du noyau C++ (`GetModelForForm()`). Les formes secondaires (Voyage, Aquatique, etc.) sont plus simples à personnaliser car elles ne dépendent pas du code C++. Testez toujours progressivement et consultez les logs serveur en cas d'erreur.

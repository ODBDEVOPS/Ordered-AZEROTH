# Création d'une nouvelle forme de druide : **Forme de Dryade** (orientée Soin)

Voici un exemple complet de création d'une forme de druide dédiée au **soin**, la **Forme de Dryade**. Elle offre des bonus passifs de soins et de régénération de mana, ainsi que des sorts de soin spécifiques. Ce guide suit la même structure que le précédent, avec les adaptations nécessaires pour une orientation heal.

---

## 1. Concept de la forme

| Propriété | Valeur |
|-----------|--------|
| **Nom** | Forme de Dryade |
| **Type** | Forme de soin (alternative à l'Arbre de Vie) |
| **Spécialité** | Augmentation des soins, régénération de mana, sorts de soin dédiés |
| **ID de forme** | `51` |
| **ID du sort de transformation** | `97010` |
| **Compétence** | Druide (573) |
| **Classes** | Druide uniquement (ClassMask `1024`) |
| **Races** | Toutes les races druidiques (Elfe de la Nuit, Tauren, Troll, Worgen) |

---

## 2. Étape 1 : SpellShapeshiftForm.dbc

### 2.1. Définition de la forme

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `51` | ID unique de la forme |
| 2 | **ActionBar** | `1` | Barre d'action bonus activée |
| 3-19 | **Name** | `Forme de Dryade` | Nom affiché |
| 20 | **Flags** | `0x000001C8` | `Can Interact NPC (0x08)` + `Can Use Equipped Items (0x40)` + `Can Use Items (0x80)` + `Don't Auto-Unshift (0x100)` |
| 21 | **CreatureType** | `7` | Humanoid (Dryade) |
| 22 | **SpellIcon** | `1` | Icône de forme |
| 23 | **combatRoundTime** | `{2500, 1000}` | Temps de round de combat |
| 24 | **DisplayID_A** | `17090` | Modèle Alliance (Dryade) — ignoré par le core, mais utile pour le client |
| 25 | **DisplayID_H** | `17090` | Modèle Horde (Dryade) |
| 28-35 | **presetSpellID[8]** | `97011`, `97012`, `97013`, `97014` | Sorts de soin prédéfinis |

### 2.2. Sorts prédéfinis (presetSpellID)

| ID | Nom | Description |
|----|-----|-------------|
| `97011` | Toucher de Dryade | Soin direct puissant |
| `97012` | Bénédiction de la Nature | Soin sur la durée (HoT) |
| `97013` | Éclat de Vie | Soin de groupe (AoE) |
| `97014` | Lien Naturel | Buff de régénération de mana (passif via transformation) |

> **Note** : Le sort `97014` peut être un buff actif, mais la régénération de mana passive est plutôt appliquée directement par le sort de transformation (voir Étape 2).

---

## 3. Étape 2 : Spell.dbc — Sort de transformation et sorts de soin

### 3.1. Sort de transformation (97010)

Ce sort applique la forme et des bonus passifs de soin et de mana.

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97010` | ID unique du sort |
| 5 | **Attributes** | `0x00000010` | `SPELL_ATTR0_IS_ABILITY` |
| 18-19 | **TargetType** | `0` | Sort sur soi-même |
| 72 | **Effect_1** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 108 | **EffectApplyAuraName_1** | `36` | `SPELL_AURA_MOD_SHAPESHIFT` |
| 113 | **EffectMiscValue_1** | `51` | ID de la forme (SpellShapeshiftForm.dbc) |
| 73 | **Effect_2** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 109 | **EffectApplyAuraName_2** | `118` | `SPELL_AURA_MOD_HEALING_DONE_PERCENT` |
| 114 | **EffectMiscValue_2** | `0` | - |
| 81 | **EffectBasePoints_2** | `20` | +20% de soins |
| 74 | **Effect_3** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 110 | **EffectApplyAuraName_3** | `96` | `SPELL_AURA_MOD_MANA_REGEN_PCT` |
| 115 | **EffectMiscValue_3** | `0` | - |
| 82 | **EffectBasePoints_3** | `50` | +50% de régénération de mana |
| 128 | **SpellIconID** | `1` | Icône |
| 131-162 | **Name** | `Forme de Dryade` | Nom du sort |
| 171-178 | **Description** | `Vous transforme en dryade, augmentant vos soins de 20% et votre régénération de mana de 50%.` | Description |
| 40 | **Duration** | `-1` | Permanent tant que la forme est active |

### 3.2. Sorts de soin prédéfinis

#### 97011 — Toucher de Dryade (Soin direct)
| Champ | Valeur |
|-------|--------|
| ID | `97011` |
| Effect_1 | `10` (SPELL_EFFECT_HEAL) |
| EffectBasePoints | `800` |
| SpellIconID | `1` |
| Name | `Toucher de Dryade` |
| Description | `Soigne la cible de X points de vie.` |

#### 97012 — Bénédiction de la Nature (HoT)
| Champ | Valeur |
|-------|--------|
| ID | `97012` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `8` (SPELL_AURA_PERIODIC_HEAL) |
| EffectBasePoints | `250` |
| EffectAmplitude | `2000` (2 sec) |
| Duration | `12000` (12 sec) |
| Name | `Bénédiction de la Nature` |

#### 97013 — Éclat de Vie (Soin de groupe)
| Champ | Valeur |
|-------|--------|
| ID | `97013` |
| Effect_1 | `10` (SPELL_EFFECT_HEAL) |
| EffectBasePoints | `400` |
| TargetType | `21` (TARGET_UNIT_CASTER_AREA_PARTY) ou `22` (TARGET_UNIT_CASTER_AREA_RAID) |
| SpellIconID | `1` |
| Name | `Éclat de Vie` |

#### 97014 — Lien Naturel (Buff actif de mana)
| Champ | Valeur |
|-------|--------|
| ID | `97014` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `96` (SPELL_AURA_MOD_MANA_REGEN_PCT) |
| EffectBasePoints | `100` |
| Duration | `1800000` (30 min) |
| Name | `Lien Naturel` |

> **Note** : La régénération de mana passive est déjà appliquée par le sort de transformation (97010). Ce sort peut servir de buff supplémentaire ou être remplacé par un autre soin.

---

## 4. Étape 3 : SkillLineAbility.dbc

Liez le sort de transformation et les sorts de soin à la compétence de druide.

| ID | SkillLine | Spell | RaceMask | ClassMask | MinSkillLineRank | AcquireMethod |
|----|-----------|-------|----------|-----------|------------------|---------------|
| 97010 | 573 | 97010 | 0 | 1024 | 1 | 0 |
| 97011 | 573 | 97011 | 0 | 1024 | 1 | 0 |
| 97012 | 573 | 97012 | 0 | 1024 | 1 | 0 |
| 97013 | 573 | 97013 | 0 | 1024 | 1 | 0 |
| 97014 | 573 | 97014 | 0 | 1024 | 1 | 0 |

---

## 5. Étape 4 : player_shapeshift_model (SQL)

Bien que le core utilise `GetModelForForm()` pour les formes principales, vous pouvez insérer des entrées pour la cohérence ou si vous utilisez une version modifiée.

```sql
-- Forme de Dryade (ID 51)
-- Utilisation d'un modèle unique de Dryade pour toutes les races/genres
INSERT INTO `player_shapeshift_model` 
(`ShapeshiftID`, `RaceID`, `CustomizationID`, `GenderID`, `ModelID`) 
VALUES 
(51, 4, 0, 2, 17090),  -- Elfe de la Nuit (tous genres)
(51, 6, 0, 2, 17090),  -- Tauren
(51, 8, 0, 2, 17090),  -- Troll
(51, 22, 0, 2, 17090); -- Worgen
```

> **Note** : `GenderID = 2` signifie "tous genres". Le modèle `17090` est un exemple de Dryade existante dans le client 3.3.5.

---

## 6. Étape 5 : Modification du noyau C++

### 6.1. Définition de FORM_DRYAD

Dans `src/server/game/Miscellaneous/SharedDefines.h` :

```cpp
enum ShapeshiftForm
{
    // ... existant ...
    FORM_RAPTOR                                 = 50,
    FORM_DRYAD                                  = 51,  // ← AJOUT
};
```

### 6.2. Modification de GetModelForForm()

Dans `src/server/game/Entities/Unit/Unit.cpp` :

```cpp
uint32 Unit::GetModelForForm(ShapeshiftForm form) const
{
    // ... code existant ...

    switch (form)
    {
        // ... autres cas ...

        // ============================================
        // AJOUT : Forme de Dryade (ID 51)
        // ============================================
        case FORM_DRYAD:
        {
            // Modèle unique de Dryade pour toutes les races
            return 17090;
        }

        default:
            break;
    }

    // ... code existant ...
}
```

> **Important** : Si vous souhaitez des modèles différents selon la race, vous pouvez ajouter un `switch (getRace())` à l'intérieur du case.

---

## 7. Étape 6 : Patch MPQ client

### 7.1. Structure du patch

```
patch-4/
└── DBFilesClient/
    ├── SpellShapeshiftForm.dbc
    ├── Spell.dbc
    └── SkillLineAbility.dbc
```

### 7.2. Création du MPQ

1. Ouvrez **Ladik's MPQ Editor**
2. Créez un nouveau MPQ : `patch-4.MPQ`
3. Ajoutez le dossier `DBFilesClient` avec les 3 fichiers DBC modifiés
4. Placez `patch-4.MPQ` dans le dossier `Data` du client WoW 3.3.5

---

## 8. Étape 7 : Compilation et test

### 8.1. Compilation

```bash
cmake --build . --config Release
```

### 8.2. Redémarrage

```bash
./worldserver
```

### 8.3. Apprentissage et test

```sql
-- Apprentissage du sort de transformation
.learn 97010

-- Apprentissage des sorts de soin (si nécessaire)
.learn 97011
.learn 97012
.learn 97013
.learn 97014
```

### 8.4. Vérifications

- ✅ La forme s'active via le sort `97010`
- ✅ Le modèle de Dryade s'affiche
- ✅ Les bonus passifs de soin (+20%) et de mana (+50%) sont appliqués
- ✅ La barre d'action bonus contient les sorts `97011` à `97014`
- ✅ Les sorts de soin fonctionnent correctement
- ✅ Le personnage peut sortir de la forme

---

## 9. Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| **Forme (SpellShapeshiftForm)** | `51` | `SpellShapeshiftForm.dbc` |
| **Sort de transformation** | `97010` | `Spell.dbc` |
| **Toucher de Dryade** | `97011` | `Spell.dbc` |
| **Bénédiction de la Nature** | `97012` | `Spell.dbc` |
| **Éclat de Vie** | `97013` | `Spell.dbc` |
| **Lien Naturel** | `97014` | `Spell.dbc` |
| **Compétence Druide** | `573` | `SkillLine.dbc` |
| **ClassMask Druide** | `1024` | `1 << (11 - 1)` |
| **Modèle Dryade** | `17090` | `CreatureDisplayInfo.dbc` |

---

## 10. Dépannage spécifique

| Problème | Cause | Solution |
|----------|-------|----------|
| Les bonus de soin ne s'appliquent pas | Effets mal configurés dans `Spell.dbc` | Vérifier `EffectApplyAuraName_2` (118) et `EffectBasePoints_2` |
| La régénération de mana ne fonctionne pas | Aura incorrecte | Vérifier `EffectApplyAuraName_3` (96) |
| Le modèle de Dryade est invisible | ModelID invalide | Vérifier `CreatureDisplayInfo.dbc` pour un ID de Dryade valide |
| Les sorts de soin ne s'affichent pas | `presetSpellID` non configuré | Remplir les colonnes 28-35 de `SpellShapeshiftForm.dbc` |
| La forme ne s'apprend pas | `SkillLineAbility.dbc` manquant | Vérifier ClassMask 1024 |
| Erreur de compilation `FORM_DRYAD` | Enum non définie | Ajouter dans `SharedDefines.h` |

---

## 11. Variantes possibles

Vous pouvez décliner cette forme de Dryade en plusieurs versions :

| Variante | ID Forme | ID Sort | Spécialité |
|----------|----------|---------|------------|
| **Dryade de Vie** | 51 | 97010 | Soins purs (+20% soins) |
| **Dryade de Mana** | 52 | 97020 | Régénération de mana accrue |
| **Dryade Ancienne** | 53 | 97030 | Soins de groupe améliorés |
| **Dryade Sylvestre** | 54 | 97040 | Furtivité + soins |

---

Ce guide vous permet de créer une **forme de druide orientée soin** complète et fonctionnelle. La **Forme de Dryade** utilise un modèle existant dans le jeu, ce qui facilite son intégration. Vous pouvez ajuster les bonus, les sorts et les flags selon vos besoins pour équilibrer la forme dans votre serveur.

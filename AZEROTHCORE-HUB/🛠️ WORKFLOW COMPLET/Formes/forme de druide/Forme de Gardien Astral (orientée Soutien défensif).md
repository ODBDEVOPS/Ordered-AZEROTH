# Création d'une nouvelle forme de druide : **Forme de Gardien Astral** (orientée Soutien défensif)

Après la **Forme de Fée** (Soutien régénération/hâte), voici une nouvelle forme de soutien plus défensive : la **Forme de Gardien Astral**. Elle apporte des boucliers, de la réduction de dégâts, de la dissipation et des buffs de groupe, tout en conservant une régénération de mana confortable. C’est une alternative au rôle de soin/support, idéale pour les druides qui veulent protéger leur groupe sans être exclusivement heal.

---

## 1. Concept de la forme

| Propriété | Valeur |
|-----------|--------|
| **Nom** | Forme de Gardien Astral |
| **Type** | Forme de Soutien défensif |
| **Spécialité** | Boucliers, réduction de dégâts, dissipation, buffs de groupe |
| **ID de forme** | `60` |
| **ID du sort de transformation** | `97100` |
| **Compétence** | Druide (573) |
| **Classes** | Druide uniquement (ClassMask `1024`) |
| **Races** | Toutes les races druidiques (Elfe de la Nuit, Tauren, Troll, Worgen) |

---

## 2. Étape 1 : SpellShapeshiftForm.dbc

### 2.1. Définition de la forme

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `60` | ID unique de la forme |
| 2 | **ActionBar** | `1` | Barre d’action bonus activée |
| 3-19 | **Name** | `Forme de Gardien Astral` | Nom affiché |
| 20 | **Flags** | `0x000001C8` | `Can Interact NPC (0x08)` + `Can Use Equipped Items (0x40)` + `Can Use Items (0x80)` + `Don't Auto-Unshift (0x100)` |
| 21 | **CreatureType** | `7` | Humanoid |
| 22 | **SpellIcon** | `1` | Icône de forme |
| 23 | **combatRoundTime** | `{2500, 1000}` | Temps de round de combat |
| 24 | **DisplayID_A** | `20251` | Modèle Alliance (Gardien Astral) — exemple |
| 25 | **DisplayID_H** | `20251` | Modèle Horde (Gardien Astral) |
| 28-35 | **presetSpellID[8]** | `97101`, `97102`, `97103`, `97104` | Sorts de soutien prédéfinis |

### 2.2. Sorts prédéfinis (presetSpellID)

| ID | Nom | Description |
|----|-----|-------------|
| `97101` | Bénédiction Astrale | Buff de groupe : +10% armure et +10% régénération de mana |
| `97102` | Bouclier Astral | Absorbe les dégâts subis par un allié |
| `97103` | Purification | Dissipe un effet de magie et un effet de poison |
| `97104` | Éveil de l’Ancien | Buff temporaire : +20% hâte et +100% régénération de mana |

---

## 3. Étape 2 : Spell.dbc — Sort de transformation et sorts de soutien

### 3.1. Sort de transformation (97100)

Ce sort applique la forme et des bonus passifs de soutien défensif.

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97100` | ID unique du sort |
| 5 | **Attributes** | `0x00000010` | `SPELL_ATTR0_IS_ABILITY` |
| 18-19 | **TargetType** | `0` | Sort sur soi-même |
| 72 | **Effect_1** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 108 | **EffectApplyAuraName_1** | `36` | `SPELL_AURA_MOD_SHAPESHIFT` |
| 113 | **EffectMiscValue_1** | `60` | ID de la forme (SpellShapeshiftForm.dbc) |
| 73 | **Effect_2** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 109 | **EffectApplyAuraName_2** | `87` | `SPELL_AURA_MOD_DAMAGE_PERCENT_TAKEN` |
| 81 | **EffectBasePoints_2** | `-10` | -10% de dégâts subis |
| 74 | **Effect_3** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 110 | **EffectApplyAuraName_3** | `96` | `SPELL_AURA_MOD_MANA_REGEN_PCT` |
| 82 | **EffectBasePoints_3** | `100` | +100% de régénération de mana |
| 75 | **Effect_4** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 111 | **EffectApplyAuraName_4** | `118` | `SPELL_AURA_MOD_HEALING_DONE_PERCENT` |
| 83 | **EffectBasePoints_4** | `10` | +10% de soins prodigués |
| 128 | **SpellIconID** | `1` | Icône |
| 131-162 | **Name** | `Forme de Gardien Astral` | Nom du sort |
| 171-178 | **Description** | `Vous transforme en gardien astral, réduisant les dégâts subis de 10%, augmentant votre régénération de mana de 100% et vos soins de 10%.` | Description |
| 40 | **Duration** | `-1` | Permanent tant que la forme est active |

### 3.2. Sorts de soutien prédéfinis

#### 97101 — Bénédiction Astrale (Buff de groupe)
| Champ | Valeur |
|-------|--------|
| ID | `97101` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `52` (SPELL_AURA_MOD_RESISTANCE_PCT) |
| EffectBasePoints | `10` | +10% armure |
| EffectMiscValue | `1` (Armure) |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `96` (SPELL_AURA_MOD_MANA_REGEN_PCT) |
| EffectBasePoints_2 | `10` | +10% régénération de mana |
| TargetType | `21` (TARGET_UNIT_CASTER_AREA_PARTY) ou `22` (RAID) |
| Duration | `1800000` (30 min) |
| Name | `Bénédiction Astrale` |
| Description | `Augmente l’armure et la régénération de mana de tous les membres du groupe de 10% pendant 30 minutes.` |

#### 97102 — Bouclier Astral (Absorption)
| Champ | Valeur |
|-------|--------|
| ID | `97102` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `69` (SPELL_AURA_SCHOOL_ABSORB) |
| EffectBasePoints | `1500` | Absorbe 1500 dégâts |
| EffectMiscValue | `127` (toutes les écoles) |
| Duration | `15000` (15 sec) |
| Cooldown | `30000` (30 sec) |
| Name | `Bouclier Astral` |
| Description | `Protège la cible avec un bouclier qui absorbe jusqu’à X points de dégâts pendant 15 secondes.` |

#### 97103 — Purification (Dissipation)
| Champ | Valeur |
|-------|--------|
| ID | `97103` |
| Effect_1 | `38` (SPELL_EFFECT_DISPEL) |
| EffectMiscValue | `1` (Magie) |
| Effect_2 | `38` (SPELL_EFFECT_DISPEL) |
| EffectMiscValue_2 | `4` (Poison) |
| Cooldown | `8000` (8 sec) |
| Name | `Purification` |
| Description | `Dissipe un effet de magie et un effet de poison sur la cible.` |

#### 97104 — Éveil de l’Ancien (Buff temporaire)
| Champ | Valeur |
|-------|--------|
| ID | `97104` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `216` (SPELL_AURA_MOD_HASTE) |
| EffectBasePoints | `20` | +20% hâte |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `96` (SPELL_AURA_MOD_MANA_REGEN_PCT) |
| EffectBasePoints_2 | `100` | +100% régénération de mana |
| Duration | `20000` (20 sec) |
| Cooldown | `120000` (2 min) |
| Name | `Éveil de l’Ancien` |
| Description | `Augmente votre hâte de 20% et votre régénération de mana de 100% pendant 20 secondes.` |

---

## 4. Étape 3 : SkillLineAbility.dbc

Liez le sort de transformation et les sorts de soutien à la compétence de druide.

| ID | SkillLine | Spell | RaceMask | ClassMask | MinSkillLineRank | AcquireMethod |
|----|-----------|-------|----------|-----------|------------------|---------------|
| 97100 | 573 | 97100 | 0 | 1024 | 1 | 0 |
| 97101 | 573 | 97101 | 0 | 1024 | 1 | 0 |
| 97102 | 573 | 97102 | 0 | 1024 | 1 | 0 |
| 97103 | 573 | 97103 | 0 | 1024 | 1 | 0 |
| 97104 | 573 | 97104 | 0 | 1024 | 1 | 0 |

---

## 5. Étape 4 : player_shapeshift_model (SQL)

```sql
-- Forme de Gardien Astral (ID 60)
-- Utilisation d'un modèle unique de Gardien Astral pour toutes les races/genres
INSERT INTO `player_shapeshift_model` 
(`ShapeshiftID`, `RaceID`, `CustomizationID`, `GenderID`, `ModelID`) 
VALUES 
(60, 4, 0, 2, 20251),  -- Elfe de la Nuit (tous genres)
(60, 6, 0, 2, 20251),  -- Tauren
(60, 8, 0, 2, 20251),  -- Troll
(60, 22, 0, 2, 20251); -- Worgen
```

> **Note** : `GenderID = 2` signifie « tous genres ». Le modèle `20251` est un exemple (Ancien Protecteur) à vérifier dans `CreatureDisplayInfo.dbc`. Remplacez-le par un ID valide.

---

## 6. Étape 5 : Modification du noyau C++

### 6.1. Définition de FORM_ASTRAL_GUARDIAN

Dans `src/server/game/Miscellaneous/SharedDefines.h` :

```cpp
enum ShapeshiftForm
{
    // ... existant ...
    FORM_RAPTOR                                 = 50,
    FORM_DRYAD                                  = 51,
    FORM_SENTINEL                               = 52,
    FORM_TURTLE                                 = 53,
    FORM_FAERIE                                 = 54,
    FORM_SERPENT                                = 55,
    FORM_SCORPION                               = 56,
    FORM_MANTICORE                              = 57,
    FORM_WYVERN                                 = 58,
    FORM_NIGHTSABER                             = 59,
    FORM_ASTRAL_GUARDIAN                        = 60,  // ← AJOUT
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
        // AJOUT : Forme de Gardien Astral (ID 60)
        // ============================================
        case FORM_ASTRAL_GUARDIAN:
        {
            // Modèle unique de Gardien Astral pour toutes les races
            return 20251;
        }

        default:
            break;
    }

    // ... code existant ...
}
```

> **Important** : Si vous souhaitez des modèles différents selon la race, vous pouvez ajouter un `switch (getRace())` à l’intérieur du case.

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
.learn 97100

-- Apprentissage des sorts de soutien (si nécessaire)
.learn 97101
.learn 97102
.learn 97103
.learn 97104
```

### 8.4. Vérifications

- ✅ La forme s’active via le sort `97100`
- ✅ Le modèle de Gardien Astral s’affiche
- ✅ Les bonus passifs (-10% dégâts subis, +100% régénération mana, +10% soins) sont appliqués
- ✅ La barre d’action bonus contient les sorts `97101` à `97104`
- ✅ Les sorts de soutien fonctionnent correctement
- ✅ Le personnage peut sortir de la forme

---

## 9. Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| **Forme (SpellShapeshiftForm)** | `60` | `SpellShapeshiftForm.dbc` |
| **Sort de transformation** | `97100` | `Spell.dbc` |
| **Bénédiction Astrale** | `97101` | `Spell.dbc` |
| **Bouclier Astral** | `97102` | `Spell.dbc` |
| **Purification** | `97103` | `Spell.dbc` |
| **Éveil de l’Ancien** | `97104` | `Spell.dbc` |
| **Compétence Druide** | `573` | `SkillLine.dbc` |
| **ClassMask Druide** | `1024` | `1 << (11 - 1)` |
| **Modèle Gardien Astral** | `20251` | `CreatureDisplayInfo.dbc` |

---

## 10. Dépannage spécifique

| Problème | Cause | Solution |
|----------|-------|----------|
| La réduction de dégâts ne s’applique pas | Effet mal configuré dans `Spell.dbc` | Vérifier `EffectApplyAuraName_2` (87) et `EffectBasePoints_2` (-10) |
| La régénération de mana ne fonctionne pas | Aura incorrecte | Vérifier `EffectApplyAuraName_3` (96) |
| Les soins ne sont pas augmentés | Aura incorrecte | Vérifier `EffectApplyAuraName_4` (118) |
| Le bouclier n’absorbe pas | Aura incorrecte | Vérifier `EffectApplyAuraName` (69) et `EffectMiscValue` (127) |
| La dissipation ne fonctionne pas | Effet incorrect | Vérifier `Effect_1` (38) avec `EffectMiscValue` (1 pour magie, 4 pour poison) |
| Le modèle est invisible | ModelID invalide | Vérifier `CreatureDisplayInfo.dbc` pour un ID valide |
| Les sorts ne s’affichent pas | `presetSpellID` non configuré | Remplir les colonnes 28-35 de `SpellShapeshiftForm.dbc` |
| La forme ne s’apprend pas | `SkillLineAbility.dbc` manquant | Vérifier ClassMask 1024 |
| Erreur de compilation `FORM_ASTRAL_GUARDIAN` | Enum non définie | Ajouter dans `SharedDefines.h` |

---

## 11. Variantes possibles

Vous pouvez décliner cette forme de Gardien Astral en plusieurs versions :

| Variante | ID Forme | ID Sort | Spécialité |
|----------|----------|---------|------------|
| **Gardien Astral** | 60 | 97100 | Soutien défensif équilibré |
| **Gardien Lunaire** | 61 | 97110 | Boucliers renforcés + soins |
| **Gardien Solaire** | 62 | 97120 | Buffs de hâte + critique |
| **Gardien Tellurique** | 63 | 97130 | Réduction de dégâts + armure |

---

Ce guide vous permet de créer une **forme de druide orientée Soutien défensif** complète et fonctionnelle. La **Forme de Gardien Astral** utilise un modèle existant dans le jeu, ce qui facilite son intégration. Vous pouvez ajuster les bonus, les sorts et les flags selon vos besoins pour équilibrer la forme dans votre serveur.

# Création d'une nouvelle forme de druide : **Forme de Gardien des Esprits** (orientée Soutien / Buff)

Après la **Forme de Fée** (Soutien régénération/hâte) et la **Forme de Gardien Astral** (Soutien défensif), voici une nouvelle forme dédiée au **soutien par les buffs** : la **Forme de Gardien des Esprits**. Elle se concentre sur l’amélioration des statistiques du groupe, la régénération de vie et de mana, les boucliers préventifs et les buffs temporaires de hâte et de critique. C’est une forme idéale pour un druide qui veut renforcer son groupe sans être exclusivement soigneur.

---

## 1. Concept de la forme

| Propriété | Valeur |
|-----------|--------|
| **Nom** | Forme de Gardien des Esprits |
| **Type** | Forme de Soutien / Buff |
| **Spécialité** | Buffs de groupe, régénération, boucliers, hâte, critique |
| **ID de forme** | `61` |
| **ID du sort de transformation** | `97120` |
| **Compétence** | Druide (573) |
| **Classes** | Druide uniquement (ClassMask `1024`) |
| **Races** | Toutes les races druidiques (Elfe de la Nuit, Tauren, Troll, Worgen) |

---

## 2. Étape 1 : SpellShapeshiftForm.dbc

### 2.1. Définition de la forme

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `61` | ID unique de la forme |
| 2 | **ActionBar** | `1` | Barre d’action bonus activée |
| 3-19 | **Name** | `Forme de Gardien des Esprits` | Nom affiché |
| 20 | **Flags** | `0x000001C8` | `Can Interact NPC (0x08)` + `Can Use Equipped Items (0x40)` + `Can Use Items (0x80)` + `Don't Auto-Unshift (0x100)` |
| 21 | **CreatureType** | `7` | Humanoid |
| 22 | **SpellIcon** | `1` | Icône de forme |
| 23 | **combatRoundTime** | `{2500, 1000}` | Temps de round de combat |
| 24 | **DisplayID_A** | `19706` | Modèle Alliance (Gardien des Esprits) — exemple |
| 25 | **DisplayID_H** | `19706` | Modèle Horde (Gardien des Esprits) |
| 28-35 | **presetSpellID[8]** | `97121`, `97122`, `97123`, `97124` | Sorts de buff prédéfinis |

### 2.2. Sorts prédéfinis (presetSpellID)

| ID | Nom | Description |
|----|-----|-------------|
| `97121` | Bénédiction des Esprits | Buff de groupe : +10% à toutes les statistiques |
| `97122` | Aura de Régénération | Buff de groupe : +20% régénération de mana et +20% régénération de vie |
| `97123` | Garde Spirituelle | Bouclier absorbant les dégâts sur un allié |
| `97124` | Éveil Spirituel | Buff temporaire : +15% hâte et +10% critique des sorts |

---

## 3. Étape 2 : Spell.dbc — Sort de transformation et sorts de buff

### 3.1. Sort de transformation (97120)

Ce sort applique la forme et des bonus passifs de soutien.

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97120` | ID unique du sort |
| 5 | **Attributes** | `0x00000010` | `SPELL_ATTR0_IS_ABILITY` |
| 18-19 | **TargetType** | `0` | Sort sur soi-même |
| 72 | **Effect_1** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 108 | **EffectApplyAuraName_1** | `36` | `SPELL_AURA_MOD_SHAPESHIFT` |
| 113 | **EffectMiscValue_1** | `61` | ID de la forme (SpellShapeshiftForm.dbc) |
| 73 | **Effect_2** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 109 | **EffectApplyAuraName_2** | `29` | `SPELL_AURA_MOD_STAT_PERCENT` |
| 114 | **EffectMiscValue_2** | `-1` | Toutes les statistiques |
| 81 | **EffectBasePoints_2** | `10` | +10% à toutes les stats |
| 74 | **Effect_3** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 110 | **EffectApplyAuraName_3** | `96` | `SPELL_AURA_MOD_MANA_REGEN_PCT` |
| 82 | **EffectBasePoints_3** | `50` | +50% régénération de mana |
| 75 | **Effect_4** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 111 | **EffectApplyAuraName_4** | `118` | `SPELL_AURA_MOD_SPELL_CRIT_CHANCE` |
| 83 | **EffectBasePoints_4** | `5` | +5% critique des sorts |
| 128 | **SpellIconID** | `1` | Icône |
| 131-162 | **Name** | `Forme de Gardien des Esprits` | Nom du sort |
| 171-178 | **Description** | `Vous transforme en gardien des esprits, augmentant toutes vos statistiques de 10%, votre régénération de mana de 50% et votre critique des sorts de 5%.` | Description |
| 40 | **Duration** | `-1` | Permanent tant que la forme est active |

### 3.2. Sorts de buff prédéfinis

#### 97121 — Bénédiction des Esprits (Buff de groupe)
| Champ | Valeur |
|-------|--------|
| ID | `97121` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `29` (SPELL_AURA_MOD_STAT_PERCENT) |
| EffectMiscValue | `-1` (Toutes les stats) |
| EffectBasePoints | `10` | +10% |
| TargetType | `21` (TARGET_UNIT_CASTER_AREA_PARTY) ou `22` (RAID) |
| Duration | `1800000` (30 min) |
| Name | `Bénédiction des Esprits` |
| Description | `Augmente toutes les statistiques des membres du groupe de 10% pendant 30 minutes.` |

#### 97122 — Aura de Régénération (Buff de groupe)
| Champ | Valeur |
|-------|--------|
| ID | `97122` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `96` (SPELL_AURA_MOD_MANA_REGEN_PCT) |
| EffectBasePoints | `20` | +20% mana |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `119` (SPELL_AURA_MOD_HEALTH_REGEN_PERCENT) |
| EffectBasePoints_2 | `20` | +20% vie |
| TargetType | `21` (PARTY) ou `22` (RAID) |
| Duration | `1800000` (30 min) |
| Name | `Aura de Régénération` |
| Description | `Augmente la régénération de mana et de vie des membres du groupe de 20% pendant 30 minutes.` |

#### 97123 — Garde Spirituelle (Bouclier)
| Champ | Valeur |
|-------|--------|
| ID | `97123` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `69` (SPELL_AURA_SCHOOL_ABSORB) |
| EffectBasePoints | `2000` | Absorbe 2000 dégâts |
| EffectMiscValue | `127` (Toutes les écoles) |
| Duration | `20000` (20 sec) |
| Cooldown | `45000` (45 sec) |
| Name | `Garde Spirituelle` |
| Description | `Protège un allié avec un bouclier absorbant jusqu’à X points de dégâts pendant 20 secondes.` |

#### 97124 — Éveil Spirituel (Buff temporaire de groupe)
| Champ | Valeur |
|-------|--------|
| ID | `97124` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `216` (SPELL_AURA_MOD_HASTE) |
| EffectBasePoints | `15` | +15% hâte |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `118` (SPELL_AURA_MOD_SPELL_CRIT_CHANCE) |
| EffectBasePoints_2 | `10` | +10% critique des sorts |
| TargetType | `21` (PARTY) ou `22` (RAID) |
| Duration | `20000` (20 sec) |
| Cooldown | `120000` (2 min) |
| Name | `Éveil Spirituel` |
| Description | `Augmente la hâte de 15% et le critique des sorts de 10% des membres du groupe pendant 20 secondes.` |

---

## 4. Étape 3 : SkillLineAbility.dbc

Liez le sort de transformation et les sorts de buff à la compétence de druide.

| ID | SkillLine | Spell | RaceMask | ClassMask | MinSkillLineRank | AcquireMethod |
|----|-----------|-------|----------|-----------|------------------|---------------|
| 97120 | 573 | 97120 | 0 | 1024 | 1 | 0 |
| 97121 | 573 | 97121 | 0 | 1024 | 1 | 0 |
| 97122 | 573 | 97122 | 0 | 1024 | 1 | 0 |
| 97123 | 573 | 97123 | 0 | 1024 | 1 | 0 |
| 97124 | 573 | 97124 | 0 | 1024 | 1 | 0 |

---

## 5. Étape 4 : player_shapeshift_model (SQL)

```sql
-- Forme de Gardien des Esprits (ID 61)
-- Utilisation d'un modèle unique de Gardien des Esprits pour toutes les races/genres
INSERT INTO `player_shapeshift_model` 
(`ShapeshiftID`, `RaceID`, `CustomizationID`, `GenderID`, `ModelID`) 
VALUES 
(61, 4, 0, 2, 19706),  -- Elfe de la Nuit (tous genres)
(61, 6, 0, 2, 19706),  -- Tauren
(61, 8, 0, 2, 19706),  -- Troll
(61, 22, 0, 2, 19706); -- Worgen
```

> **Note** : `GenderID = 2` signifie « tous genres ». Le modèle `19706` est un exemple (Gardien des Esprits). Remplacez-le par un ID valide de `CreatureDisplayInfo.dbc`.

---

## 6. Étape 5 : Modification du noyau C++

### 6.1. Définition de FORM_SPIRIT_GUARDIAN

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
    FORM_ASTRAL_GUARDIAN                        = 60,
    FORM_SPIRIT_GUARDIAN                        = 61,  // ← AJOUT
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
        // AJOUT : Forme de Gardien des Esprits (ID 61)
        // ============================================
        case FORM_SPIRIT_GUARDIAN:
        {
            // Modèle unique de Gardien des Esprits pour toutes les races
            return 19706;
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
.learn 97120

-- Apprentissage des sorts de buff (si nécessaire)
.learn 97121
.learn 97122
.learn 97123
.learn 97124
```

### 8.4. Vérifications

- ✅ La forme s’active via le sort `97120`
- ✅ Le modèle de Gardien des Esprits s’affiche
- ✅ Les bonus passifs (+10% stats, +50% mana, +5% critique) sont appliqués
- ✅ La barre d’action bonus contient les sorts `97121` à `97124`
- ✅ Les sorts de buff fonctionnent correctement
- ✅ Le personnage peut sortir de la forme

---

## 9. Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| **Forme (SpellShapeshiftForm)** | `61` | `SpellShapeshiftForm.dbc` |
| **Sort de transformation** | `97120` | `Spell.dbc` |
| **Bénédiction des Esprits** | `97121` | `Spell.dbc` |
| **Aura de Régénération** | `97122` | `Spell.dbc` |
| **Garde Spirituelle** | `97123` | `Spell.dbc` |
| **Éveil Spirituel** | `97124` | `Spell.dbc` |
| **Compétence Druide** | `573` | `SkillLine.dbc` |
| **ClassMask Druide** | `1024` | `1 << (11 - 1)` |
| **Modèle Gardien des Esprits** | `19706` | `CreatureDisplayInfo.dbc` |

---

## 10. Dépannage spécifique

| Problème | Cause | Solution |
|----------|-------|----------|
| Les bonus de stats ne s’appliquent pas | Effet mal configuré dans `Spell.dbc` | Vérifier `EffectApplyAuraName_2` (29) et `EffectMiscValue_2` (-1) |
| La régénération de mana ne fonctionne pas | Aura incorrecte | Vérifier `EffectApplyAuraName_3` (96) |
| Le critique ne s’applique pas | Aura incorrecte | Vérifier `EffectApplyAuraName_4` (118) |
| Le bouclier n’absorbe pas | Aura incorrecte | Vérifier `EffectApplyAuraName` (69) et `EffectMiscValue` (127) |
| Le buff de groupe ne touche pas les alliés | TargetType incorrect | Utiliser `21` (PARTY) ou `22` (RAID) |
| Le modèle est invisible | ModelID invalide | Vérifier `CreatureDisplayInfo.dbc` pour un ID valide |
| Les sorts ne s’affichent pas | `presetSpellID` non configuré | Remplir les colonnes 28-35 de `SpellShapeshiftForm.dbc` |
| La forme ne s’apprend pas | `SkillLineAbility.dbc` manquant | Vérifier ClassMask 1024 |
| Erreur de compilation `FORM_SPIRIT_GUARDIAN` | Enum non définie | Ajouter dans `SharedDefines.h` |

---

## 11. Variantes possibles

Vous pouvez décliner cette forme de Gardien des Esprits en plusieurs versions :

| Variante | ID Forme | ID Sort | Spécialité |
|----------|----------|---------|------------|
| **Gardien des Esprits** | 61 | 97120 | Buffs équilibrés (+10% stats) |
| **Gardien Lunaire** | 62 | 97130 | Buffs de critique et de soins |
| **Gardien Solaire** | 63 | 97140 | Buffs de hâte et de dégâts |
| **Gardien Tellurique** | 64 | 97150 | Buffs d’armure et de régénération |

---

Ce guide vous permet de créer une **forme de druide orientée Soutien / Buff** complète et fonctionnelle. La **Forme de Gardien des Esprits** utilise un modèle existant dans le jeu, ce qui facilite son intégration. Vous pouvez ajuster les bonus, les sorts et les flags selon vos besoins pour équilibrer la forme dans votre serveur.

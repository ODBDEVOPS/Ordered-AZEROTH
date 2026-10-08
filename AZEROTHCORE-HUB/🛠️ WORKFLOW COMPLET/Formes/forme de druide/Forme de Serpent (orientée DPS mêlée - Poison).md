# Création d'une nouvelle forme de druide : **Forme de Serpent** (orientée DPS mêlée / Poison)

Après la **Forme de Raptor** (DPS mêlée), la **Forme de Dryade** (Soin), la **Forme de Sentinelle** (DPS caster), la **Forme de Tortue** (Tank) et la **Forme de Fée** (Soutien), voici une nouvelle forme dédiée au **DPS mêlée avec orientation poison et évasion** : la **Forme de Serpent**. Elle constitue une alternative à la Forme de Loup, avec des dégâts de poison sur la durée, une vitesse d’attaque élevée et une capacité d’esquive accrue.

---

## 1. Concept de la forme

| Propriété | Valeur |
|-----------|--------|
| **Nom** | Forme de Serpent |
| **Type** | Forme de DPS mêlée (alternative à la Forme de Loup) |
| **Spécialité** | Poison, vitesse d’attaque, esquive, saignement |
| **ID de forme** | `55` |
| **ID du sort de transformation** | `97050` |
| **Compétence** | Druide (573) |
| **Classes** | Druide uniquement (ClassMask `1024`) |
| **Races** | Toutes les races druidiques (Elfe de la Nuit, Tauren, Troll, Worgen) |

---

## 2. Étape 1 : SpellShapeshiftForm.dbc

### 2.1. Définition de la forme

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `55` | ID unique de la forme |
| 2 | **ActionBar** | `1` | Barre d’action bonus activée |
| 3-19 | **Name** | `Forme de Serpent` | Nom affiché |
| 20 | **Flags** | `0x00000030` | `Don't Use Weapon (0x10)` + `Agility Attack Bonus (0x20)` |
| 21 | **CreatureType** | `1` | Bête |
| 22 | **SpellIcon** | `1` | Icône de forme |
| 23 | **combatRoundTime** | `{2500, 1000}` | Temps de round de combat |
| 24 | **DisplayID_A** | `7550` | Modèle Alliance (Serpent) — exemple |
| 25 | **DisplayID_H** | `7550` | Modèle Horde (Serpent) |
| 28-35 | **presetSpellID[8]** | `97051`, `97052`, `97053`, `97054` | Sorts de serpent prédéfinis |

### 2.2. Sorts prédéfinis (presetSpellID)

| ID | Nom | Description |
|----|-----|-------------|
| `97051` | Morsure venimeuse | Attaque de mêlée appliquant un poison |
| `97052` | Étreinte constrictrice | Saignement et réduction de vitesse |
| `97053` | Nuage de poison | Dégâts de poison en zone |
| `97054` | Esquive serpentine | Cooldown défensif : augmente l’esquive |

---

## 3. Étape 2 : Spell.dbc — Sort de transformation et sorts offensifs

### 3.1. Sort de transformation (97050)

Ce sort applique la forme et des bonus passifs de DPS mêlée.

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97050` | ID unique du sort |
| 5 | **Attributes** | `0x00000010` | `SPELL_ATTR0_IS_ABILITY` |
| 18-19 | **TargetType** | `0` | Sort sur soi-même |
| 72 | **Effect_1** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 108 | **EffectApplyAuraName_1** | `36` | `SPELL_AURA_MOD_SHAPESHIFT` |
| 113 | **EffectMiscValue_1** | `55` | ID de la forme (SpellShapeshiftForm.dbc) |
| 73 | **Effect_2** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 109 | **EffectApplyAuraName_2** | `13` | `SPELL_AURA_MOD_DAMAGE_DONE` |
| 114 | **EffectMiscValue_2** | `8` | École : Nature (poison) |
| 81 | **EffectBasePoints_2** | `10` | +10% de dégâts nature |
| 74 | **Effect_3** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 110 | **EffectApplyAuraName_3** | `216` | `SPELL_AURA_MOD_HASTE` (hâte de mêlée) |
| 82 | **EffectBasePoints_3** | `15` | +15% de hâte |
| 75 | **Effect_4** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 111 | **EffectApplyAuraName_4** | `31` | `SPELL_AURA_MOD_DODGE_PERCENT` |
| 83 | **EffectBasePoints_4** | `5` | +5% d’esquive |
| 128 | **SpellIconID** | `1` | Icône |
| 131-162 | **Name** | `Forme de Serpent` | Nom du sort |
| 171-178 | **Description** | `Vous transforme en serpent, augmentant vos dégâts nature de 10%, votre hâte de 15% et votre esquive de 5%.` | Description |
| 40 | **Duration** | `-1` | Permanent tant que la forme est active |

### 3.2. Sorts offensifs prédéfinis

#### 97051 — Morsure venimeuse (Poison)
| Champ | Valeur |
|-------|--------|
| ID | `97051` |
| Effect_1 | `2` (SPELL_EFFECT_SCHOOL_DAMAGE) |
| EffectBasePoints | `250` |
| EffectMiscValue | `8` (Nature) |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `3` (SPELL_AURA_PERIODIC_DAMAGE) |
| EffectBasePoints_2 | `80` |
| EffectAmplitude_2 | `3000` (3 sec) |
| Duration_2 | `12000` (12 sec) |
| Name | `Morsure venimeuse` |
| Description | `Inflige des dégâts nature et un poison qui inflige X dégâts toutes les 3 secondes pendant 12 secondes.` |

#### 97052 — Étreinte constrictrice (Saignement + Ralentissement)
| Champ | Valeur |
|-------|--------|
| ID | `97052` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `3` (SPELL_AURA_PERIODIC_DAMAGE) |
| EffectBasePoints | `100` |
| EffectAmplitude | `2000` (2 sec) |
| Duration | `10000` (10 sec) |
| EffectMiscValue | `1` (Physique) |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `33` (SPELL_AURA_MOD_DECREASE_SPEED) |
| EffectBasePoints_2 | `-30` | -30% vitesse |
| Duration_2 | `10000` (10 sec) |
| Name | `Étreinte constrictrice` |
| Description | `Inflige un saignement et réduit la vitesse de déplacement de la cible de 30% pendant 10 secondes.` |

#### 97053 — Nuage de poison (AoE)
| Champ | Valeur |
|-------|--------|
| ID | `97053` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `3` (SPELL_AURA_PERIODIC_DAMAGE) |
| EffectBasePoints | `60` |
| EffectAmplitude | `2000` (2 sec) |
| Duration | `8000` (8 sec) |
| EffectMiscValue | `8` (Nature) |
| TargetType | `22` (TARGET_UNIT_CASTER_AREA_RAID) ou `21` (PARTY) |
| Name | `Nuage de poison` |
| Description | `Inflige des dégâts nature toutes les 2 secondes aux ennemis proches pendant 8 secondes.` |

#### 97054 — Esquive serpentine (Cooldown défensif)
| Champ | Valeur |
|-------|--------|
| ID | `97054` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `31` (SPELL_AURA_MOD_DODGE_PERCENT) |
| EffectBasePoints | `30` | +30% esquive |
| Duration | `10000` (10 sec) |
| Cooldown | `120000` (2 min) |
| Name | `Esquive serpentine` |
| Description | `Augmente votre esquive de 30% pendant 10 secondes.` |

---

## 4. Étape 3 : SkillLineAbility.dbc

Liez le sort de transformation et les sorts offensifs à la compétence de druide.

| ID | SkillLine | Spell | RaceMask | ClassMask | MinSkillLineRank | AcquireMethod |
|----|-----------|-------|----------|-----------|------------------|---------------|
| 97050 | 573 | 97050 | 0 | 1024 | 1 | 0 |
| 97051 | 573 | 97051 | 0 | 1024 | 1 | 0 |
| 97052 | 573 | 97052 | 0 | 1024 | 1 | 0 |
| 97053 | 573 | 97053 | 0 | 1024 | 1 | 0 |
| 97054 | 573 | 97054 | 0 | 1024 | 1 | 0 |

---

## 5. Étape 4 : player_shapeshift_model (SQL)

```sql
-- Forme de Serpent (ID 55)
-- Utilisation d'un modèle unique de Serpent pour toutes les races/genres
INSERT INTO `player_shapeshift_model` 
(`ShapeshiftID`, `RaceID`, `CustomizationID`, `GenderID`, `ModelID`) 
VALUES 
(55, 4, 0, 2, 7550),  -- Elfe de la Nuit (tous genres)
(55, 6, 0, 2, 7550),  -- Tauren
(55, 8, 0, 2, 7550),  -- Troll
(55, 22, 0, 2, 7550); -- Worgen
```

> **Note** : `GenderID = 2` signifie « tous genres ». Le modèle `7550` est un exemple de serpent existant dans le client 3.3.5. Remplacez-le par un ID valide de `CreatureDisplayInfo.dbc`.

---

## 6. Étape 5 : Modification du noyau C++

### 6.1. Définition de FORM_SERPENT

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
    FORM_SERPENT                                = 55,  // ← AJOUT
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
        // AJOUT : Forme de Serpent (ID 55)
        // ============================================
        case FORM_SERPENT:
        {
            // Modèle unique de Serpent pour toutes les races
            return 7550;
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
.learn 97050

-- Apprentissage des sorts offensifs (si nécessaire)
.learn 97051
.learn 97052
.learn 97053
.learn 97054
```

### 8.4. Vérifications

- ✅ La forme s’active via le sort `97050`
- ✅ Le modèle de Serpent s’affiche
- ✅ Les bonus passifs (+10% dégâts nature, +15% hâte, +5% esquive) sont appliqués
- ✅ La barre d’action bonus contient les sorts `97051` à `97054`
- ✅ Les sorts offensifs fonctionnent correctement
- ✅ Le personnage peut sortir de la forme

---

## 9. Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| **Forme (SpellShapeshiftForm)** | `55` | `SpellShapeshiftForm.dbc` |
| **Sort de transformation** | `97050` | `Spell.dbc` |
| **Morsure venimeuse** | `97051` | `Spell.dbc` |
| **Étreinte constrictrice** | `97052` | `Spell.dbc` |
| **Nuage de poison** | `97053` | `Spell.dbc` |
| **Esquive serpentine** | `97054` | `Spell.dbc` |
| **Compétence Druide** | `573` | `SkillLine.dbc` |
| **ClassMask Druide** | `1024` | `1 << (11 - 1)` |
| **Modèle Serpent** | `7550` | `CreatureDisplayInfo.dbc` |

---

## 10. Dépannage spécifique

| Problème | Cause | Solution |
|----------|-------|----------|
| Les bonus de dégâts nature ne s’appliquent pas | Effets mal configurés dans `Spell.dbc` | Vérifier `EffectApplyAuraName_2` (13) et `EffectMiscValue_2` (8) |
| La hâte ne fonctionne pas | Aura incorrecte | Vérifier `EffectApplyAuraName_3` (216) |
| L’esquive ne s’applique pas | Aura incorrecte | Vérifier `EffectApplyAuraName_4` (31) |
| Le modèle de Serpent est invisible | ModelID invalide | Vérifier `CreatureDisplayInfo.dbc` pour un ID de serpent valide |
| Les sorts offensifs ne s’affichent pas | `presetSpellID` non configuré | Remplir les colonnes 28-35 de `SpellShapeshiftForm.dbc` |
| La forme ne s’apprend pas | `SkillLineAbility.dbc` manquant | Vérifier ClassMask 1024 |
| Erreur de compilation `FORM_SERPENT` | Enum non définie | Ajouter dans `SharedDefines.h` |

---

## 11. Variantes possibles

Vous pouvez décliner cette forme de Serpent en plusieurs versions :

| Variante | ID Forme | ID Sort | Spécialité |
|----------|----------|---------|------------|
| **Serpent venimeux** | 55 | 97050 | Poison (+10% dégâts nature) |
| **Serpent constricteur** | 56 | 97060 | Saignement + ralentissement |
| **Serpent des sables** | 57 | 97070 | Esquive + vitesse |
| **Serpent de mana** | 58 | 97080 | Dégâts arcane + régénération mana |

---

Ce guide vous permet de créer une **forme de druide orientée DPS mêlée / Poison** complète et fonctionnelle. La **Forme de Serpent** utilise un modèle existant dans le jeu, ce qui facilite son intégration. Vous pouvez ajuster les bonus, les sorts et les flags selon vos besoins pour équilibrer la forme dans votre serveur.

# Création d'une nouvelle forme de druide : **Forme de Sabre de Nuit** (orientée DPS mêlée / Furtivité)

Après la **Forme de Raptor** (DPS mêlée), la **Forme de Dryade** (Soin), la **Forme de Sentinelle** (DPS caster), la **Forme de Tortue** (Tank), la **Forme de Fée** (Soutien), la **Forme de Serpent** (DPS mêlée / Poison), la **Forme de Scorpion** (DPS mêlée / Contrôle), la **Forme de Manticore** (DPS mêlée / Contrôle) et la **Forme de Wyverne** (DPS caster / Contrôle), voici une nouvelle forme dédiée au **DPS mêlée furtif** : la **Forme de Sabre de Nuit**. Elle mise sur la furtivité, les attaques d’ouverture, les saignements et l’esquive, à la manière d’un voleur mais avec la puissance brute d’un félin.

---

## 1. Concept de la forme

| Propriété | Valeur |
|-----------|--------|
| **Nom** | Forme de Sabre de Nuit |
| **Type** | Forme de DPS mêlée / Furtivité |
| **Spécialité** | Furtivité, burst, saignement, esquive |
| **ID de forme** | `59` |
| **ID du sort de transformation** | `97090` |
| **Compétence** | Druide (573) |
| **Classes** | Druide uniquement (ClassMask `1024`) |
| **Races** | Toutes les races druidiques (Elfe de la Nuit, Tauren, Troll, Worgen) |

---

## 2. Étape 1 : SpellShapeshiftForm.dbc

### 2.1. Définition de la forme

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `59` | ID unique de la forme |
| 2 | **ActionBar** | `1` | Barre d’action bonus activée |
| 3-19 | **Name** | `Forme de Sabre de Nuit` | Nom affiché |
| 20 | **Flags** | `0x00000030` | `Don't Use Weapon (0x10)` + `Agility Attack Bonus (0x20)` |
| 21 | **CreatureType** | `1` | Bête |
| 22 | **SpellIcon** | `1` | Icône de forme |
| 23 | **combatRoundTime** | `{2500, 1000}` | Temps de round de combat |
| 24 | **DisplayID_A** | `9994` | Modèle Alliance (Sabre de Nuit) — exemple |
| 25 | **DisplayID_H** | `9994` | Modèle Horde (Sabre de Nuit) |
| 28-35 | **presetSpellID[8]** | `97091`, `97092`, `97093`, `97094` | Sorts de sabre prédéfinis |

### 2.2. Sorts prédéfinis (presetSpellID)

| ID | Nom | Description |
|----|-----|-------------|
| `97091` | Griffe d’ombre | Attaque d’ouverture (dégâts accrus si furtif) |
| `97092` | Morsure du Sabre | Saignement sur la durée |
| `97093` | Bond de l’Ombre | Charge furtive vers la cible |
| `97094` | Voile de Nuit | Cooldown : furtivité + esquive |

---

## 3. Étape 2 : Spell.dbc — Sort de transformation et sorts offensifs

### 3.1. Sort de transformation (97090)

Ce sort applique la forme et des bonus passifs de DPS mêlée furtif.

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97090` | ID unique du sort |
| 5 | **Attributes** | `0x00000010` | `SPELL_ATTR0_IS_ABILITY` |
| 18-19 | **TargetType** | `0` | Sort sur soi-même |
| 72 | **Effect_1** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 108 | **EffectApplyAuraName_1** | `36` | `SPELL_AURA_MOD_SHAPESHIFT` |
| 113 | **EffectMiscValue_1** | `59` | ID de la forme (SpellShapeshiftForm.dbc) |
| 73 | **Effect_2** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 109 | **EffectApplyAuraName_2** | `13` | `SPELL_AURA_MOD_DAMAGE_DONE` |
| 114 | **EffectMiscValue_2** | `1` | École : Physique |
| 81 | **EffectBasePoints_2** | `10` | +10% de dégâts physiques |
| 74 | **Effect_3** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 110 | **EffectApplyAuraName_3** | `216` | `SPELL_AURA_MOD_HASTE` (hâte de mêlée) |
| 82 | **EffectBasePoints_3** | `15` | +15% de hâte |
| 75 | **Effect_4** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 111 | **EffectApplyAuraName_4** | `31` | `SPELL_AURA_MOD_DODGE_PERCENT` |
| 83 | **EffectBasePoints_4** | `5` | +5% d’esquive |
| 128 | **SpellIconID** | `1` | Icône |
| 131-162 | **Name** | `Forme de Sabre de Nuit` | Nom du sort |
| 171-178 | **Description** | `Vous transforme en sabre de nuit, augmentant vos dégâts physiques de 10%, votre hâte de 15% et votre esquive de 5%.` | Description |
| 40 | **Duration** | `-1` | Permanent tant que la forme est active |

> **Note** : La furtivité n’est pas innée à la forme ; elle est activée par le sort `97094` (Voile de Nuit) ou par un sort de furtivité séparé.

### 3.2. Sorts offensifs prédéfinis

#### 97091 — Griffe d’Ombre (Attaque d’ouverture)
| Champ | Valeur |
|-------|--------|
| ID | `97091` |
| Effect_1 | `2` (SPELL_EFFECT_SCHOOL_DAMAGE) |
| EffectBasePoints | `350` |
| EffectMiscValue | `1` (Physique) |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `3` (SPELL_AURA_PERIODIC_DAMAGE) |
| EffectBasePoints_2 | `100` |
| EffectAmplitude_2 | `2000` (2 sec) |
| Duration_2 | `10000` (10 sec) |
| Name | `Griffe d’Ombre` |
| Description | `Inflige des dégâts physiques et un saignement. Dégâts accrus si utilisé en furtivité.` |

#### 97092 — Morsure du Sabre (Saignement)
| Champ | Valeur |
|-------|--------|
| ID | `97092` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `3` (SPELL_AURA_PERIODIC_DAMAGE) |
| EffectBasePoints | `120` |
| EffectAmplitude | `2000` (2 sec) |
| Duration | `12000` (12 sec) |
| EffectMiscValue | `1` (Physique) |
| Name | `Morsure du Sabre` |
| Description | `Inflige X points de dégâts physiques toutes les 2 secondes pendant 12 secondes.` |

#### 97093 — Bond de l’Ombre (Charge furtive)
| Champ | Valeur |
|-------|--------|
| ID | `97093` |
| Effect_1 | `96` (SPELL_EFFECT_CHARGE) |
| EffectBasePoints | `10` (portée) |
| Cooldown | `15000` (15 sec) |
| Name | `Bond de l’Ombre` |
| Description | `Charge vers la cible. Ne peut être utilisé qu’en furtivité.` |

#### 97094 — Voile de Nuit (Furtivité + Esquive)
| Champ | Valeur |
|-------|--------|
| ID | `97094` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `16` (SPELL_AURA_MOD_INVISIBILITY) |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `31` (SPELL_AURA_MOD_DODGE_PERCENT) |
| EffectBasePoints_2 | `20` | +20% esquive |
| Duration | `10000` (10 sec) |
| Cooldown | `60000` (1 min) |
| Name | `Voile de Nuit` |
| Description | `Vous rend furtif et augmente votre esquive de 20% pendant 10 secondes.` |

---

## 4. Étape 3 : SkillLineAbility.dbc

Liez le sort de transformation et les sorts offensifs à la compétence de druide.

| ID | SkillLine | Spell | RaceMask | ClassMask | MinSkillLineRank | AcquireMethod |
|----|-----------|-------|----------|-----------|------------------|---------------|
| 97090 | 573 | 97090 | 0 | 1024 | 1 | 0 |
| 97091 | 573 | 97091 | 0 | 1024 | 1 | 0 |
| 97092 | 573 | 97092 | 0 | 1024 | 1 | 0 |
| 97093 | 573 | 97093 | 0 | 1024 | 1 | 0 |
| 97094 | 573 | 97094 | 0 | 1024 | 1 | 0 |

---

## 5. Étape 4 : player_shapeshift_model (SQL)

```sql
-- Forme de Sabre de Nuit (ID 59)
-- Utilisation d'un modèle unique de Sabre de Nuit pour toutes les races/genres
INSERT INTO `player_shapeshift_model` 
(`ShapeshiftID`, `RaceID`, `CustomizationID`, `GenderID`, `ModelID`) 
VALUES 
(59, 4, 0, 2, 9994),  -- Elfe de la Nuit (tous genres)
(59, 6, 0, 2, 9994),  -- Tauren
(59, 8, 0, 2, 9994),  -- Troll
(59, 22, 0, 2, 9994); -- Worgen
```

> **Note** : `GenderID = 2` signifie « tous genres ». Le modèle `9994` est un exemple de sabre de nuit existant dans le client 3.3.5. Remplacez-le par un ID valide de `CreatureDisplayInfo.dbc`.

---

## 6. Étape 5 : Modification du noyau C++

### 6.1. Définition de FORM_NIGHTSABER

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
    FORM_NIGHTSABER                             = 59,  // ← AJOUT
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
        // AJOUT : Forme de Sabre de Nuit (ID 59)
        // ============================================
        case FORM_NIGHTSABER:
        {
            // Modèle unique de Sabre de Nuit pour toutes les races
            return 9994;
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
.learn 97090

-- Apprentissage des sorts offensifs (si nécessaire)
.learn 97091
.learn 97092
.learn 97093
.learn 97094
```

### 8.4. Vérifications

- ✅ La forme s’active via le sort `97090`
- ✅ Le modèle de Sabre de Nuit s’affiche
- ✅ Les bonus passifs (+10% dégâts physiques, +15% hâte, +5% esquive) sont appliqués
- ✅ La barre d’action bonus contient les sorts `97091` à `97094`
- ✅ Les sorts offensifs fonctionnent correctement
- ✅ La furtivité (Voile de Nuit) fonctionne
- ✅ Le personnage peut sortir de la forme

---

## 9. Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| **Forme (SpellShapeshiftForm)** | `59` | `SpellShapeshiftForm.dbc` |
| **Sort de transformation** | `97090` | `Spell.dbc` |
| **Griffe d’Ombre** | `97091` | `Spell.dbc` |
| **Morsure du Sabre** | `97092` | `Spell.dbc` |
| **Bond de l’Ombre** | `97093` | `Spell.dbc` |
| **Voile de Nuit** | `97094` | `Spell.dbc` |
| **Compétence Druide** | `573` | `SkillLine.dbc` |
| **ClassMask Druide** | `1024` | `1 << (11 - 1)` |
| **Modèle Sabre de Nuit** | `9994` | `CreatureDisplayInfo.dbc` |

---

## 10. Dépannage spécifique

| Problème | Cause | Solution |
|----------|-------|----------|
| Les bonus de dégâts physiques ne s’appliquent pas | Effets mal configurés dans `Spell.dbc` | Vérifier `EffectApplyAuraName_2` (13) et `EffectMiscValue_2` (1) |
| La hâte ne fonctionne pas | Aura incorrecte | Vérifier `EffectApplyAuraName_3` (216) |
| L’esquive ne s’applique pas | Aura incorrecte | Vérifier `EffectApplyAuraName_4` (31) |
| La furtivité ne fonctionne pas | Aura incorrecte | Vérifier `EffectApplyAuraName` (16) pour le sort 97094 |
| Le modèle de Sabre de Nuit est invisible | ModelID invalide | Vérifier `CreatureDisplayInfo.dbc` pour un ID de sabre valide |
| Les sorts offensifs ne s’affichent pas | `presetSpellID` non configuré | Remplir les colonnes 28-35 de `SpellShapeshiftForm.dbc` |
| La forme ne s’apprend pas | `SkillLineAbility.dbc` manquant | Vérifier ClassMask 1024 |
| Erreur de compilation `FORM_NIGHTSABER` | Enum non définie | Ajouter dans `SharedDefines.h` |

---

## 11. Variantes possibles

Vous pouvez décliner cette forme de Sabre de Nuit en plusieurs versions :

| Variante | ID Forme | ID Sort | Spécialité |
|----------|----------|---------|------------|
| **Sabre de l’Ombre** | 59 | 97090 | Furtivité + burst |
| **Sabre Sanglant** | 60 | 97100 | Saignement + hâte |
| **Sabre des Sables** | 61 | 97110 | Esquive + vitesse |
| **Sabre Lunaire** | 62 | 97120 | Dégâts arcane + furtivité |

---

Ce guide vous permet de créer une **forme de druide orientée DPS mêlée / Furtivité** complète et fonctionnelle. La **Forme de Sabre de Nuit** utilise un modèle existant dans le jeu, ce qui facilite son intégration. Vous pouvez ajuster les bonus, les sorts et les flags selon vos besoins pour équilibrer la forme dans votre serveur.

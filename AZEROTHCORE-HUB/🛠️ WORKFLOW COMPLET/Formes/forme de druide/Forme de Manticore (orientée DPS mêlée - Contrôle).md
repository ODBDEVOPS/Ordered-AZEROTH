# Création d'une nouvelle forme de druide : **Forme de Manticore** (orientée DPS mêlée / Contrôle)

Après la **Forme de Raptor** (DPS mêlée), la **Forme de Dryade** (Soin), la **Forme de Sentinelle** (DPS caster), la **Forme de Tortue** (Tank), la **Forme de Fée** (Soutien), la **Forme de Serpent** (DPS mêlée / Poison) et la **Forme de Scorpion** (DPS mêlée / Contrôle), voici une nouvelle forme dédiée au **DPS mêlée avec contrôle de foule** : la **Forme de Manticore**. Elle combine saignement, poison et peur, avec une bonne mobilité grâce à ses ailes.

---

## 1. Concept de la forme

| Propriété | Valeur |
|-----------|--------|
| **Nom** | Forme de Manticore |
| **Type** | Forme de DPS mêlée / Contrôle |
| **Spécialité** | Saignement, poison, peur, esquive, vitesse |
| **ID de forme** | `57` |
| **ID du sort de transformation** | `97070` |
| **Compétence** | Druide (573) |
| **Classes** | Druide uniquement (ClassMask `1024`) |
| **Races** | Toutes les races druidiques (Elfe de la Nuit, Tauren, Troll, Worgen) |

---

## 2. Étape 1 : SpellShapeshiftForm.dbc

### 2.1. Définition de la forme

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `57` | ID unique de la forme |
| 2 | **ActionBar** | `1` | Barre d’action bonus activée |
| 3-19 | **Name** | `Forme de Manticore` | Nom affiché |
| 20 | **Flags** | `0x00000030` | `Don't Use Weapon (0x10)` + `Agility Attack Bonus (0x20)` |
| 21 | **CreatureType** | `1` | Bête |
| 22 | **SpellIcon** | `1` | Icône de forme |
| 23 | **combatRoundTime** | `{2500, 1000}` | Temps de round de combat |
| 24 | **DisplayID_A** | `20578` | Modèle Alliance (Manticore) — exemple |
| 25 | **DisplayID_H** | `20578` | Modèle Horde (Manticore) |
| 28-35 | **presetSpellID[8]** | `97071`, `97072`, `97073`, `97074` | Sorts de manticore prédéfinis |

### 2.2. Sorts prédéfinis (presetSpellID)

| ID | Nom | Description |
|----|-----|-------------|
| `97071` | Morsure de Manticore | Attaque directe + saignement |
| `97072` | Dard de Manticore | Poison sur la durée |
| `97073` | Rugissement effrayant | Peur de zone |
| `97074` | Ailes de Manticore | Cooldown : esquive + vitesse |

---

## 3. Étape 2 : Spell.dbc — Sort de transformation et sorts offensifs

### 3.1. Sort de transformation (97070)

Ce sort applique la forme et des bonus passifs de DPS mêlée.

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97070` | ID unique du sort |
| 5 | **Attributes** | `0x00000010` | `SPELL_ATTR0_IS_ABILITY` |
| 18-19 | **TargetType** | `0` | Sort sur soi-même |
| 72 | **Effect_1** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 108 | **EffectApplyAuraName_1** | `36` | `SPELL_AURA_MOD_SHAPESHIFT` |
| 113 | **EffectMiscValue_1** | `57` | ID de la forme (SpellShapeshiftForm.dbc) |
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
| 131-162 | **Name** | `Forme de Manticore` | Nom du sort |
| 171-178 | **Description** | `Vous transforme en manticore, augmentant vos dégâts physiques de 10%, votre hâte de 15% et votre esquive de 5%.` | Description |
| 40 | **Duration** | `-1` | Permanent tant que la forme est active |

### 3.2. Sorts offensifs prédéfinis

#### 97071 — Morsure de Manticore (Dégâts + Saignement)
| Champ | Valeur |
|-------|--------|
| ID | `97071` |
| Effect_1 | `2` (SPELL_EFFECT_SCHOOL_DAMAGE) |
| EffectBasePoints | `300` |
| EffectMiscValue | `1` (Physique) |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `3` (SPELL_AURA_PERIODIC_DAMAGE) |
| EffectBasePoints_2 | `80` |
| EffectAmplitude_2 | `2000` (2 sec) |
| Duration_2 | `12000` (12 sec) |
| EffectMiscValue_2 | `1` (Physique) |
| Name | `Morsure de Manticore` |
| Description | `Inflige des dégâts physiques et un saignement qui inflige X dégâts toutes les 2 secondes pendant 12 secondes.` |

#### 97072 — Dard de Manticore (Poison)
| Champ | Valeur |
|-------|--------|
| ID | `97072` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `3` (SPELL_AURA_PERIODIC_DAMAGE) |
| EffectBasePoints | `100` |
| EffectAmplitude | `3000` (3 sec) |
| Duration | `15000` (15 sec) |
| EffectMiscValue | `8` (Nature) |
| Name | `Dard de Manticore` |
| Description | `Inflige X points de dégâts nature toutes les 3 secondes pendant 15 secondes.` |

#### 97073 — Rugissement effrayant (Peur de zone)
| Champ | Valeur |
|-------|--------|
| ID | `97073` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `7` (SPELL_AURA_MOD_FEAR) |
| Duration | `4000` (4 sec) |
| TargetType | `22` (TARGET_UNIT_CASTER_AREA_RAID) ou `21` (PARTY) |
| Cooldown | `60000` (1 min) |
| Name | `Rugissement effrayant` |
| Description | `Effraie les ennemis proches pendant 4 secondes.` |

#### 97074 — Ailes de Manticore (Cooldown utilitaire)
| Champ | Valeur |
|-------|--------|
| ID | `97074` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `31` (SPELL_AURA_MOD_DODGE_PERCENT) |
| EffectBasePoints | `25` | +25% esquive |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `103` (SPELL_AURA_MOD_SPEED_ALWAYS) |
| EffectBasePoints_2 | `30` | +30% vitesse |
| Duration | `8000` (8 sec) |
| Cooldown | `90000` (1 min 30) |
| Name | `Ailes de Manticore` |
| Description | `Augmente votre esquive de 25% et votre vitesse de déplacement de 30% pendant 8 secondes.` |

---

## 4. Étape 3 : SkillLineAbility.dbc

Liez le sort de transformation et les sorts offensifs à la compétence de druide.

| ID | SkillLine | Spell | RaceMask | ClassMask | MinSkillLineRank | AcquireMethod |
|----|-----------|-------|----------|-----------|------------------|---------------|
| 97070 | 573 | 97070 | 0 | 1024 | 1 | 0 |
| 97071 | 573 | 97071 | 0 | 1024 | 1 | 0 |
| 97072 | 573 | 97072 | 0 | 1024 | 1 | 0 |
| 97073 | 573 | 97073 | 0 | 1024 | 1 | 0 |
| 97074 | 573 | 97074 | 0 | 1024 | 1 | 0 |

---

## 5. Étape 4 : player_shapeshift_model (SQL)

```sql
-- Forme de Manticore (ID 57)
-- Utilisation d'un modèle unique de Manticore pour toutes les races/genres
INSERT INTO `player_shapeshift_model` 
(`ShapeshiftID`, `RaceID`, `CustomizationID`, `GenderID`, `ModelID`) 
VALUES 
(57, 4, 0, 2, 20578),  -- Elfe de la Nuit (tous genres)
(57, 6, 0, 2, 20578),  -- Tauren
(57, 8, 0, 2, 20578),  -- Troll
(57, 22, 0, 2, 20578); -- Worgen
```

> **Note** : `GenderID = 2` signifie « tous genres ». Le modèle `20578` est un exemple de manticore existante dans le client 3.3.5 (Outland). Remplacez-le par un ID valide de `CreatureDisplayInfo.dbc`.

---

## 6. Étape 5 : Modification du noyau C++

### 6.1. Définition de FORM_MANTICORE

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
    FORM_MANTICORE                              = 57,  // ← AJOUT
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
        // AJOUT : Forme de Manticore (ID 57)
        // ============================================
        case FORM_MANTICORE:
        {
            // Modèle unique de Manticore pour toutes les races
            return 20578;
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
.learn 97070

-- Apprentissage des sorts offensifs (si nécessaire)
.learn 97071
.learn 97072
.learn 97073
.learn 97074
```

### 8.4. Vérifications

- ✅ La forme s’active via le sort `97070`
- ✅ Le modèle de Manticore s’affiche
- ✅ Les bonus passifs (+10% dégâts physiques, +15% hâte, +5% esquive) sont appliqués
- ✅ La barre d’action bonus contient les sorts `97071` à `97074`
- ✅ Les sorts offensifs fonctionnent correctement
- ✅ Le personnage peut sortir de la forme

---

## 9. Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| **Forme (SpellShapeshiftForm)** | `57` | `SpellShapeshiftForm.dbc` |
| **Sort de transformation** | `97070` | `Spell.dbc` |
| **Morsure de Manticore** | `97071` | `Spell.dbc` |
| **Dard de Manticore** | `97072` | `Spell.dbc` |
| **Rugissement effrayant** | `97073` | `Spell.dbc` |
| **Ailes de Manticore** | `97074` | `Spell.dbc` |
| **Compétence Druide** | `573` | `SkillLine.dbc` |
| **ClassMask Druide** | `1024` | `1 << (11 - 1)` |
| **Modèle Manticore** | `20578` | `CreatureDisplayInfo.dbc` |

---

## 10. Dépannage spécifique

| Problème | Cause | Solution |
|----------|-------|----------|
| Les bonus de dégâts physiques ne s’appliquent pas | Effets mal configurés dans `Spell.dbc` | Vérifier `EffectApplyAuraName_2` (13) et `EffectMiscValue_2` (1) |
| La hâte ne fonctionne pas | Aura incorrecte | Vérifier `EffectApplyAuraName_3` (216) |
| L’esquive ne s’applique pas | Aura incorrecte | Vérifier `EffectApplyAuraName_4` (31) |
| Le modèle de Manticore est invisible | ModelID invalide | Vérifier `CreatureDisplayInfo.dbc` pour un ID de manticore valide |
| Les sorts offensifs ne s’affichent pas | `presetSpellID` non configuré | Remplir les colonnes 28-35 de `SpellShapeshiftForm.dbc` |
| La forme ne s’apprend pas | `SkillLineAbility.dbc` manquant | Vérifier ClassMask 1024 |
| Erreur de compilation `FORM_MANTICORE` | Enum non définie | Ajouter dans `SharedDefines.h` |

---

## 11. Variantes possibles

Vous pouvez décliner cette forme de Manticore en plusieurs versions :

| Variante | ID Forme | ID Sort | Spécialité |
|----------|----------|---------|------------|
| **Manticore venimeuse** | 57 | 97070 | Poison + saignement |
| **Manticore rugissante** | 58 | 97080 | Peur + contrôle |
| **Manticore des sables** | 59 | 97090 | Esquive + vitesse |
| **Manticore de magma** | 60 | 97100 | Dégâts de feu + armure |

---

Ce guide vous permet de créer une **forme de druide orientée DPS mêlée / Contrôle** complète et fonctionnelle. La **Forme de Manticore** utilise un modèle existant dans le jeu, ce qui facilite son intégration. Vous pouvez ajuster les bonus, les sorts et les flags selon vos besoins pour équilibrer la forme dans votre serveur.

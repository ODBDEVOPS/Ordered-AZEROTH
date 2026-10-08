# Création d'une nouvelle forme de druide : **Forme de Tortue** (orientée Tank)

Après la **Forme de Raptor** (DPS mêlée), la **Forme de Dryade** (Soin) et la **Forme de Sentinelle** (DPS caster), voici une nouvelle forme dédiée au **rôle de tank** : la **Forme de Tortue**. Elle constitue une alternative à la Forme d’Ours, avec une orientation défensive basée sur l’armure, les points de vie et la génération de menace.

---

## 1. Concept de la forme

| Propriété | Valeur |
|-----------|--------|
| **Nom** | Forme de Tortue |
| **Type** | Forme de Tank (alternative à la Forme d’Ours) |
| **Spécialité** | Armure, points de vie, menace, défense |
| **ID de forme** | `53` |
| **ID du sort de transformation** | `97030` |
| **Compétence** | Druide (573) |
| **Classes** | Druide uniquement (ClassMask `1024`) |
| **Races** | Toutes les races druidiques (Elfe de la Nuit, Tauren, Troll, Worgen) |

---

## 2. Étape 1 : SpellShapeshiftForm.dbc

### 2.1. Définition de la forme

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `53` | ID unique de la forme |
| 2 | **ActionBar** | `1` | Barre d’action bonus activée |
| 3-19 | **Name** | `Forme de Tortue` | Nom affiché |
| 20 | **Flags** | `0x000000C8` | `Can Interact NPC (0x08)` + `Can Use Equipped Items (0x40)` + `Can Use Items (0x80)` |
| 21 | **CreatureType** | `1` | Bête |
| 22 | **SpellIcon** | `1` | Icône de forme |
| 23 | **combatRoundTime** | `{2500, 1000}` | Temps de round de combat |
| 24 | **DisplayID_A** | `7370` | Modèle Alliance (Tortue) — exemple |
| 25 | **DisplayID_H** | `7370` | Modèle Horde (Tortue) |
| 28-35 | **presetSpellID[8]** | `97031`, `97032`, `97033`, `97034` | Sorts de tank prédéfinis |

### 2.2. Sorts prédéfinis (presetSpellID)

| ID | Nom | Description |
|----|-----|-------------|
| `97031` | Morsure de Tortue | Attaque de mêlée générant de la menace |
| `97032` | Carapace | Cooldown défensif : réduit les dégâts subis |
| `97033` | Rugissement | Génère de la menace sur les cibles autour de vous |
| `97034` | Écrasement | Attaque lourde avec étourdissement court |

---

## 3. Étape 2 : Spell.dbc — Sort de transformation et sorts de tank

### 3.1. Sort de transformation (97030)

Ce sort applique la forme et des bonus passifs de tank.

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97030` | ID unique du sort |
| 5 | **Attributes** | `0x00000010` | `SPELL_ATTR0_IS_ABILITY` |
| 18-19 | **TargetType** | `0` | Sort sur soi-même |
| 72 | **Effect_1** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 108 | **EffectApplyAuraName_1** | `36` | `SPELL_AURA_MOD_SHAPESHIFT` |
| 113 | **EffectMiscValue_1** | `53` | ID de la forme (SpellShapeshiftForm.dbc) |
| 73 | **Effect_2** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 109 | **EffectApplyAuraName_2** | `49` | `SPELL_AURA_MOD_INCREASE_HEALTH_PERCENT` |
| 81 | **EffectBasePoints_2** | `25` | +25% de points de vie |
| 74 | **Effect_3** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 110 | **EffectApplyAuraName_3** | `52` | `SPELL_AURA_MOD_RESISTANCE_PCT` |
| 115 | **EffectMiscValue_3** | `1` | Type de résistance : Armure (Physique) |
| 82 | **EffectBasePoints_3** | `50` | +50% d’armure |
| 75 | **Effect_4** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 111 | **EffectApplyAuraName_4** | `77` | `SPELL_AURA_MOD_THREAT` |
| 83 | **EffectBasePoints_4** | `30` | +30% de menace générée |
| 128 | **SpellIconID** | `1` | Icône |
| 131-162 | **Name** | `Forme de Tortue` | Nom du sort |
| 171-178 | **Description** | `Vous transforme en tortue, augmentant vos points de vie de 25%, votre armure de 50% et votre menace de 30%.` | Description |
| 40 | **Duration** | `-1` | Permanent tant que la forme est active |

### 3.2. Sorts de tank prédéfinis

#### 97031 — Morsure de Tortue (Menace)
| Champ | Valeur |
|-------|--------|
| ID | `97031` |
| Effect_1 | `2` (SPELL_EFFECT_SCHOOL_DAMAGE) |
| EffectBasePoints | `300` |
| EffectMiscValue | `1` (Physique) |
| SpellIconID | `1` |
| Name | `Morsure de Tortue` |
| Description | `Inflige X points de dégâts et génère une menace élevée.` |

#### 97032 — Carapace (Défensif)
| Champ | Valeur |
|-------|--------|
| ID | `97032` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `87` (SPELL_AURA_MOD_DAMAGE_PERCENT_TAKEN) |
| EffectBasePoints | `-40` | Réduit les dégâts subis de 40% |
| Duration | `12000` (12 sec) |
| Cooldown | `180000` (3 min) |
| Name | `Carapace` |
| Description | `Réduit les dégâts subis de 40% pendant 12 secondes.` |

#### 97033 — Rugissement (Menace de zone)
| Champ | Valeur |
|-------|--------|
| ID | `97033` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `77` (SPELL_AURA_MOD_THREAT) |
| EffectBasePoints | `100` |
| TargetType | `22` (TARGET_UNIT_CASTER_AREA_RAID) ou `21` (PARTY) |
| Duration | `6000` (6 sec) |
| Cooldown | `8000` (8 sec) |
| Name | `Rugissement` |
| Description | `Génère de la menace sur tous les ennemis proches.` |

#### 97034 — Écrasement (Dégâts + étourdissement)
| Champ | Valeur |
|-------|--------|
| ID | `97034` |
| Effect_1 | `2` (SPELL_EFFECT_SCHOOL_DAMAGE) |
| EffectBasePoints | `450` |
| EffectMiscValue | `1` (Physique) |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `12` (SPELL_AURA_MOD_STUN) |
| Duration | `2000` (2 sec) |
| Cooldown | `30000` (30 sec) |
| Name | `Écrasement` |
| Description | `Inflige des dégâts et étourdit la cible pendant 2 secondes.` |

---

## 4. Étape 3 : SkillLineAbility.dbc

Liez le sort de transformation et les sorts de tank à la compétence de druide.

| ID | SkillLine | Spell | RaceMask | ClassMask | MinSkillLineRank | AcquireMethod |
|----|-----------|-------|----------|-----------|------------------|---------------|
| 97030 | 573 | 97030 | 0 | 1024 | 1 | 0 |
| 97031 | 573 | 97031 | 0 | 1024 | 1 | 0 |
| 97032 | 573 | 97032 | 0 | 1024 | 1 | 0 |
| 97033 | 573 | 97033 | 0 | 1024 | 1 | 0 |
| 97034 | 573 | 97034 | 0 | 1024 | 1 | 0 |

---

## 5. Étape 4 : player_shapeshift_model (SQL)

```sql
-- Forme de Tortue (ID 53)
-- Utilisation d'un modèle unique de Tortue pour toutes les races/genres
INSERT INTO `player_shapeshift_model` 
(`ShapeshiftID`, `RaceID`, `CustomizationID`, `GenderID`, `ModelID`) 
VALUES 
(53, 4, 0, 2, 7370),  -- Elfe de la Nuit (tous genres)
(53, 6, 0, 2, 7370),  -- Tauren
(53, 8, 0, 2, 7370),  -- Troll
(53, 22, 0, 2, 7370); -- Worgen
```

> **Note** : `GenderID = 2` signifie « tous genres ». Le modèle `7370` est un exemple de tortue existante dans le client 3.3.5. Remplacez-le par un ID valide de `CreatureDisplayInfo.dbc`.

---

## 6. Étape 5 : Modification du noyau C++

### 6.1. Définition de FORM_TURTLE

Dans `src/server/game/Miscellaneous/SharedDefines.h` :

```cpp
enum ShapeshiftForm
{
    // ... existant ...
    FORM_RAPTOR                                 = 50,
    FORM_DRYAD                                  = 51,
    FORM_SENTINEL                               = 52,
    FORM_TURTLE                                 = 53,  // ← AJOUT
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
        // AJOUT : Forme de Tortue (ID 53)
        // ============================================
        case FORM_TURTLE:
        {
            // Modèle unique de Tortue pour toutes les races
            return 7370;
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
.learn 97030

-- Apprentissage des sorts de tank (si nécessaire)
.learn 97031
.learn 97032
.learn 97033
.learn 97034
```

### 8.4. Vérifications

- ✅ La forme s’active via le sort `97030`
- ✅ Le modèle de Tortue s’affiche
- ✅ Les bonus passifs (+25% PV, +50% armure, +30% menace) sont appliqués
- ✅ La barre d’action bonus contient les sorts `97031` à `97034`
- ✅ Les sorts de tank fonctionnent correctement
- ✅ Le personnage peut sortir de la forme

---

## 9. Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| **Forme (SpellShapeshiftForm)** | `53` | `SpellShapeshiftForm.dbc` |
| **Sort de transformation** | `97030` | `Spell.dbc` |
| **Morsure de Tortue** | `97031` | `Spell.dbc` |
| **Carapace** | `97032` | `Spell.dbc` |
| **Rugissement** | `97033` | `Spell.dbc` |
| **Écrasement** | `97034` | `Spell.dbc` |
| **Compétence Druide** | `573` | `SkillLine.dbc` |
| **ClassMask Druide** | `1024` | `1 << (11 - 1)` |
| **Modèle Tortue** | `7370` | `CreatureDisplayInfo.dbc` |

---

## 10. Dépannage spécifique

| Problème | Cause | Solution |
|----------|-------|----------|
| Les bonus de PV/armure ne s’appliquent pas | Effets mal configurés dans `Spell.dbc` | Vérifier `EffectApplyAuraName_2` (49), `EffectApplyAuraName_3` (52) et `EffectMiscValue_3` (1) |
| La menace n’augmente pas | Aura incorrecte | Vérifier `EffectApplyAuraName_4` (77) |
| Le modèle de Tortue est invisible | ModelID invalide | Vérifier `CreatureDisplayInfo.dbc` pour un ID de tortue valide |
| Les sorts de tank ne s’affichent pas | `presetSpellID` non configuré | Remplir les colonnes 28-35 de `SpellShapeshiftForm.dbc` |
| La forme ne s’apprend pas | `SkillLineAbility.dbc` manquant | Vérifier ClassMask 1024 |
| Erreur de compilation `FORM_TURTLE` | Enum non définie | Ajouter dans `SharedDefines.h` |

---

## 11. Variantes possibles

Vous pouvez décliner cette forme de Tortue en plusieurs versions :

| Variante | ID Forme | ID Sort | Spécialité |
|----------|----------|---------|------------|
| **Tortue Ancienne** | 53 | 97030 | Tank équilibré (+25% PV, +50% armure) |
| **Tortue de Magma** | 54 | 97040 | Tank résistant au Feu |
| **Tortue de Glace** | 55 | 97050 | Tank résistant au Givre |
| **Tortue de l’Ombre** | 56 | 97060 | Tank résistant à l’Ombre |

---

Ce guide vous permet de créer une **forme de druide orientée Tank** complète et fonctionnelle. La **Forme de Tortue** utilise un modèle existant dans le jeu, ce qui facilite son intégration. Vous pouvez ajuster les bonus, les sorts et les flags selon vos besoins pour équilibrer la forme dans votre serveur.

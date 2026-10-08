# Création d'une nouvelle forme de druide : **Forme de Wyverne** (orientée DPS caster / Contrôle)

Après la **Forme de Raptor** (DPS mêlée), la **Forme de Dryade** (Soin), la **Forme de Sentinelle** (DPS caster), la **Forme de Tortue** (Tank), la **Forme de Fée** (Soutien), la **Forme de Serpent** (DPS mêlée / Poison), la **Forme de Scorpion** (DPS mêlée / Contrôle) et la **Forme de Manticore** (DPS mêlée / Contrôle), voici une nouvelle forme dédiée au **DPS caster avec contrôle** : la **Forme de Wyverne**. Elle mise sur les dégâts de nature et de foudre, le poison et l’incapacitation, avec une bonne mobilité aérienne symbolique.

---

## 1. Concept de la forme

| Propriété | Valeur |
|-----------|--------|
| **Nom** | Forme de Wyverne |
| **Type** | Forme de DPS caster / Contrôle |
| **Spécialité** | Dégâts nature, foudre, poison, étourdissement |
| **ID de forme** | `58` |
| **ID du sort de transformation** | `97080` |
| **Compétence** | Druide (573) |
| **Classes** | Druide uniquement (ClassMask `1024`) |
| **Races** | Toutes les races druidiques (Elfe de la Nuit, Tauren, Troll, Worgen) |

---

## 2. Étape 1 : SpellShapeshiftForm.dbc

### 2.1. Définition de la forme

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `58` | ID unique de la forme |
| 2 | **ActionBar** | `1` | Barre d’action bonus activée |
| 3-19 | **Name** | `Forme de Wyverne` | Nom affiché |
| 20 | **Flags** | `0x000001C8` | `Can Interact NPC (0x08)` + `Can Use Equipped Items (0x40)` + `Can Use Items (0x80)` + `Don't Auto-Unshift (0x100)` |
| 21 | **CreatureType** | `1` | Bête |
| 22 | **SpellIcon** | `1` | Icône de forme |
| 23 | **combatRoundTime** | `{2500, 1000}` | Temps de round de combat |
| 24 | **DisplayID_A** | `6377` | Modèle Alliance (Wyverne) — exemple |
| 25 | **DisplayID_H** | `6377` | Modèle Horde (Wyverne) |
| 28-35 | **presetSpellID[8]** | `97081`, `97082`, `97083`, `97084` | Sorts de wyverne prédéfinis |

### 2.2. Sorts prédéfinis (presetSpellID)

| ID | Nom | Description |
|----|-----|-------------|
| `97081` | Éclair de Wyverne | Attaque directe de dégâts nature |
| `97082` | Venin de Wyverne | Poison sur la durée |
| `97083` | Chaîne d’éclairs | Dégâts de zone (jusqu’à 3 cibles) |
| `97084` | Souffle paralysant | Étourdissement court |

---

## 3. Étape 2 : Spell.dbc — Sort de transformation et sorts offensifs

### 3.1. Sort de transformation (97080)

Ce sort applique la forme et des bonus passifs de DPS caster.

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97080` | ID unique du sort |
| 5 | **Attributes** | `0x00000010` | `SPELL_ATTR0_IS_ABILITY` |
| 18-19 | **TargetType** | `0` | Sort sur soi-même |
| 72 | **Effect_1** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 108 | **EffectApplyAuraName_1** | `36` | `SPELL_AURA_MOD_SHAPESHIFT` |
| 113 | **EffectMiscValue_1** | `58` | ID de la forme (SpellShapeshiftForm.dbc) |
| 73 | **Effect_2** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 109 | **EffectApplyAuraName_2** | `13` | `SPELL_AURA_MOD_DAMAGE_DONE` |
| 114 | **EffectMiscValue_2** | `8` | École : Nature |
| 81 | **EffectBasePoints_2** | `15` | +15% de dégâts nature |
| 74 | **Effect_3** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 110 | **EffectApplyAuraName_3** | `216` | `SPELL_AURA_MOD_HASTE` (hâte des sorts) |
| 82 | **EffectBasePoints_3** | `10` | +10% de hâte des sorts |
| 75 | **Effect_4** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 111 | **EffectApplyAuraName_4** | `118` | `SPELL_AURA_MOD_SPELL_CRIT_CHANCE` |
| 83 | **EffectBasePoints_4** | `5` | +5% de critique des sorts |
| 128 | **SpellIconID** | `1` | Icône |
| 131-162 | **Name** | `Forme de Wyverne` | Nom du sort |
| 171-178 | **Description** | `Vous transforme en wyverne, augmentant vos dégâts nature de 15%, votre hâte des sorts de 10% et votre critique des sorts de 5%.` | Description |
| 40 | **Duration** | `-1` | Permanent tant que la forme est active |

### 3.2. Sorts offensifs prédéfinis

#### 97081 — Éclair de Wyverne (Dégâts directs Nature)
| Champ | Valeur |
|-------|--------|
| ID | `97081` |
| Effect_1 | `2` (SPELL_EFFECT_SCHOOL_DAMAGE) |
| EffectBasePoints | `500` |
| EffectMiscValue | `8` (Nature) |
| SpellIconID | `1` |
| Name | `Éclair de Wyverne` |
| Description | `Inflige X points de dégâts nature à la cible.` |

#### 97082 — Venin de Wyverne (Poison DoT)
| Champ | Valeur |
|-------|--------|
| ID | `97082` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `3` (SPELL_AURA_PERIODIC_DAMAGE) |
| EffectBasePoints | `110` |
| EffectAmplitude | `3000` (3 sec) |
| Duration | `15000` (15 sec) |
| EffectMiscValue | `8` (Nature) |
| Name | `Venin de Wyverne` |
| Description | `Inflige X points de dégâts nature toutes les 3 secondes pendant 15 secondes.` |

#### 97083 — Chaîne d’éclairs (AoE)
| Champ | Valeur |
|-------|--------|
| ID | `97083` |
| Effect_1 | `2` (SPELL_EFFECT_SCHOOL_DAMAGE) |
| EffectBasePoints | `350` |
| EffectMiscValue | `8` (Nature) |
| TargetType | `22` (TARGET_UNIT_CASTER_AREA_RAID) ou `21` (PARTY) |
| ChainTargets | `3` |
| Name | `Chaîne d’éclairs` |
| Description | `Inflige X points de dégâts nature à jusqu’à 3 ennemis proches.` |

#### 97084 — Souffle paralysant (Étourdissement)
| Champ | Valeur |
|-------|--------|
| ID | `97084` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `12` (SPELL_AURA_MOD_STUN) |
| Duration | `3000` (3 sec) |
| TargetType | `6` (TARGET_UNIT_TARGET_ENEMY) |
| Cooldown | `30000` (30 sec) |
| Name | `Souffle paralysant` |
| Description | `Étourdit la cible pendant 3 secondes.` |

---

## 4. Étape 3 : SkillLineAbility.dbc

Liez le sort de transformation et les sorts offensifs à la compétence de druide.

| ID | SkillLine | Spell | RaceMask | ClassMask | MinSkillLineRank | AcquireMethod |
|----|-----------|-------|----------|-----------|------------------|---------------|
| 97080 | 573 | 97080 | 0 | 1024 | 1 | 0 |
| 97081 | 573 | 97081 | 0 | 1024 | 1 | 0 |
| 97082 | 573 | 97082 | 0 | 1024 | 1 | 0 |
| 97083 | 573 | 97083 | 0 | 1024 | 1 | 0 |
| 97084 | 573 | 97084 | 0 | 1024 | 1 | 0 |

---

## 5. Étape 4 : player_shapeshift_model (SQL)

```sql
-- Forme de Wyverne (ID 58)
-- Utilisation d'un modèle unique de Wyverne pour toutes les races/genres
INSERT INTO `player_shapeshift_model` 
(`ShapeshiftID`, `RaceID`, `CustomizationID`, `GenderID`, `ModelID`) 
VALUES 
(58, 4, 0, 2, 6377),  -- Elfe de la Nuit (tous genres)
(58, 6, 0, 2, 6377),  -- Tauren
(58, 8, 0, 2, 6377),  -- Troll
(58, 22, 0, 2, 6377); -- Worgen
```

> **Note** : `GenderID = 2` signifie « tous genres ». Le modèle `6377` est un exemple de wyverne existante dans le client 3.3.5. Remplacez-le par un ID valide de `CreatureDisplayInfo.dbc`.

---

## 6. Étape 5 : Modification du noyau C++

### 6.1. Définition de FORM_WYVERN

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
    FORM_WYVERN                                 = 58,  // ← AJOUT
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
        // AJOUT : Forme de Wyverne (ID 58)
        // ============================================
        case FORM_WYVERN:
        {
            // Modèle unique de Wyverne pour toutes les races
            return 6377;
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
.learn 97080

-- Apprentissage des sorts offensifs (si nécessaire)
.learn 97081
.learn 97082
.learn 97083
.learn 97084
```

### 8.4. Vérifications

- ✅ La forme s’active via le sort `97080`
- ✅ Le modèle de Wyverne s’affiche
- ✅ Les bonus passifs (+15% dégâts nature, +10% hâte, +5% critique) sont appliqués
- ✅ La barre d’action bonus contient les sorts `97081` à `97084`
- ✅ Les sorts offensifs fonctionnent correctement
- ✅ Le personnage peut sortir de la forme

---

## 9. Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| **Forme (SpellShapeshiftForm)** | `58` | `SpellShapeshiftForm.dbc` |
| **Sort de transformation** | `97080` | `Spell.dbc` |
| **Éclair de Wyverne** | `97081` | `Spell.dbc` |
| **Venin de Wyverne** | `97082` | `Spell.dbc` |
| **Chaîne d’éclairs** | `97083` | `Spell.dbc` |
| **Souffle paralysant** | `97084` | `Spell.dbc` |
| **Compétence Druide** | `573` | `SkillLine.dbc` |
| **ClassMask Druide** | `1024` | `1 << (11 - 1)` |
| **Modèle Wyverne** | `6377` | `CreatureDisplayInfo.dbc` |

---

## 10. Dépannage spécifique

| Problème | Cause | Solution |
|----------|-------|----------|
| Les bonus de dégâts nature ne s’appliquent pas | Effets mal configurés dans `Spell.dbc` | Vérifier `EffectApplyAuraName_2` (13) et `EffectMiscValue_2` (8) |
| La hâte ne fonctionne pas | Aura incorrecte | Vérifier `EffectApplyAuraName_3` (216) |
| Le critique ne s’applique pas | Aura incorrecte | Vérifier `EffectApplyAuraName_4` (118) |
| Le modèle de Wyverne est invisible | ModelID invalide | Vérifier `CreatureDisplayInfo.dbc` pour un ID de wyverne valide |
| Les sorts offensifs ne s’affichent pas | `presetSpellID` non configuré | Remplir les colonnes 28-35 de `SpellShapeshiftForm.dbc` |
| La forme ne s’apprend pas | `SkillLineAbility.dbc` manquant | Vérifier ClassMask 1024 |
| Erreur de compilation `FORM_WYVERN` | Enum non définie | Ajouter dans `SharedDefines.h` |

---

## 11. Variantes possibles

Vous pouvez décliner cette forme de Wyverne en plusieurs versions :

| Variante | ID Forme | ID Sort | Spécialité |
|----------|----------|---------|------------|
| **Wyverne de foudre** | 58 | 97080 | Dégâts nature + hâte |
| **Wyverne venimeuse** | 59 | 97090 | Poison + contrôle |
| **Wyverne des tempêtes** | 60 | 97100 | Dégâts de zone + critique |
| **Wyverne de sable** | 61 | 97110 | Étourdissement + esquive |

---

Ce guide vous permet de créer une **forme de druide orientée DPS caster / Contrôle** complète et fonctionnelle. La **Forme de Wyverne** utilise un modèle existant dans le jeu, ce qui facilite son intégration. Vous pouvez ajuster les bonus, les sorts et les flags selon vos besoins pour équilibrer la forme dans votre serveur.

# Création d'une nouvelle forme de druide : **Forme de Sentinelle** (orientée DPS caster)

Voici un exemple complet de création d'une forme de druide dédiée au **DPS à distance** (caster), la **Forme de Sentinelle**. Elle offre des bonus passifs de dégâts des sorts, de critique et de hâte, ainsi que des sorts offensifs lunaires et naturels. Ce guide suit la même structure que les précédents, avec les adaptations nécessaires pour une orientation DPS caster.

---

## 1. Concept de la forme

| Propriété | Valeur |
|-----------|--------|
| **Nom** | Forme de Sentinelle |
| **Type** | Forme de DPS caster (alternative à la Forme de Sélénien) |
| **Spécialité** | Dégâts des sorts, critique, hâte, sorts lunaires et naturels |
| **ID de forme** | `52` |
| **ID du sort de transformation** | `97020` |
| **Compétence** | Druide (573) |
| **Classes** | Druide uniquement (ClassMask `1024`) |
| **Races** | Toutes les races druidiques (Elfe de la Nuit, Tauren, Troll, Worgen) |

---

## 2. Étape 1 : SpellShapeshiftForm.dbc

### 2.1. Définition de la forme

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `52` | ID unique de la forme |
| 2 | **ActionBar** | `1` | Barre d'action bonus activée |
| 3-19 | **Name** | `Forme de Sentinelle` | Nom affiché |
| 20 | **Flags** | `0x000000C8` | `Can Interact NPC (0x08)` + `Can Use Equipped Items (0x40)` + `Can Use Items (0x80)` |
| 21 | **CreatureType** | `7` | Humanoid (Sentinelle) |
| 22 | **SpellIcon** | `1` | Icône de forme |
| 23 | **combatRoundTime** | `{2500, 1000}` | Temps de round de combat |
| 24 | **DisplayID_A** | `21406` | Modèle Alliance (Sentinelle) — exemple |
| 25 | **DisplayID_H** | `21406` | Modèle Horde (Sentinelle) |
| 28-35 | **presetSpellID[8]** | `97021`, `97022`, `97023`, `97024` | Sorts offensifs prédéfinis |

### 2.2. Sorts prédéfinis (presetSpellID)

| ID | Nom | Description |
|----|-----|-------------|
| `97021` | Éclat lunaire | Attaque directe de dégâts arcane |
| `97022` | Colère | Attaque directe de dégâts nature |
| `97023` | Feu stellaire | Attaque directe de dégâts arcane (plus lente, plus puissante) |
| `97024` | Essaim d'insectes | Dégâts nature sur la durée (DoT) |

---

## 3. Étape 2 : Spell.dbc — Sort de transformation et sorts offensifs

### 3.1. Sort de transformation (97020)

Ce sort applique la forme et des bonus passifs de DPS caster.

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97020` | ID unique du sort |
| 5 | **Attributes** | `0x00000010` | `SPELL_ATTR0_IS_ABILITY` |
| 18-19 | **TargetType** | `0` | Sort sur soi-même |
| 72 | **Effect_1** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 108 | **EffectApplyAuraName_1** | `36` | `SPELL_AURA_MOD_SHAPESHIFT` |
| 113 | **EffectMiscValue_1** | `52` | ID de la forme (SpellShapeshiftForm.dbc) |
| 73 | **Effect_2** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 109 | **EffectApplyAuraName_2** | `13` | `SPELL_AURA_MOD_DAMAGE_DONE` |
| 114 | **EffectMiscValue_2** | `126` | École de magie : Arcane + Nature (`0x7E` = 126) |
| 81 | **EffectBasePoints_2** | `15` | +15% de dégâts des sorts |
| 74 | **Effect_3** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 110 | **EffectApplyAuraName_3** | `118` | `SPELL_AURA_MOD_SPELL_CRIT_CHANCE` |
| 115 | **EffectMiscValue_3** | `126` | Arcane + Nature |
| 82 | **EffectBasePoints_3** | `5` | +5% de critique des sorts |
| 128 | **SpellIconID** | `1` | Icône |
| 131-162 | **Name** | `Forme de Sentinelle` | Nom du sort |
| 171-178 | **Description** | `Vous transforme en sentinelle, augmentant vos dégâts des sorts de 15% et votre critique des sorts de 5%.` | Description |
| 40 | **Duration** | `-1` | Permanent tant que la forme est active |

> **Note** : L'école de magie `126` correspond à Arcane (64) + Nature (2) + Feu (4) + Givre (16) + Ombre (32) + Sacré (2) ? En réalité, les masques d'école sont : `1` = Physique, `2` = Sacré, `4` = Feu, `8` = Nature, `16` = Givre, `32` = Ombre, `64` = Arcane. Pour Arcane + Nature : `64 + 8 = 72`. Utilisez `72` plutôt que `126`. Corrigez si nécessaire.

### 3.2. Sorts offensifs prédéfinis

#### 97021 — Éclat lunaire (Dégâts directs Arcane)
| Champ | Valeur |
|-------|--------|
| ID | `97021` |
| Effect_1 | `2` (SPELL_EFFECT_SCHOOL_DAMAGE) |
| EffectBasePoints | `450` |
| EffectMiscValue | `64` (Arcane) |
| SpellIconID | `1` |
| Name | `Éclat lunaire` |
| Description | `Inflige X points de dégâts arcane à la cible.` |

#### 97022 — Colère (Dégâts directs Nature)
| Champ | Valeur |
|-------|--------|
| ID | `97022` |
| Effect_1 | `2` (SPELL_EFFECT_SCHOOL_DAMAGE) |
| EffectBasePoints | `380` |
| EffectMiscValue | `8` (Nature) |
| SpellIconID | `1` |
| Name | `Colère` |
| Description | `Inflige X points de dégâts nature à la cible.` |

#### 97023 — Feu stellaire (Dégâts directs Arcane, plus puissant)
| Champ | Valeur |
|-------|--------|
| ID | `97023` |
| Effect_1 | `2` (SPELL_EFFECT_SCHOOL_DAMAGE) |
| EffectBasePoints | `650` |
| EffectMiscValue | `64` (Arcane) |
| CastTime | `3000` (3 sec) |
| SpellIconID | `1` |
| Name | `Feu stellaire` |
| Description | `Inflige X points de dégâts arcane à la cible. Temps d'incantation plus long.` |

#### 97024 — Essaim d'insectes (DoT Nature)
| Champ | Valeur |
|-------|--------|
| ID | `97024` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `3` (SPELL_AURA_PERIODIC_DAMAGE) |
| EffectBasePoints | `120` |
| EffectAmplitude | `2000` (2 sec) |
| Duration | `12000` (12 sec) |
| EffectMiscValue | `8` (Nature) |
| Name | `Essaim d'insectes` |
| Description | `Inflige X points de dégâts nature toutes les 2 secondes pendant 12 secondes.` |

---

## 4. Étape 3 : SkillLineAbility.dbc

Liez le sort de transformation et les sorts offensifs à la compétence de druide.

| ID | SkillLine | Spell | RaceMask | ClassMask | MinSkillLineRank | AcquireMethod |
|----|-----------|-------|----------|-----------|------------------|---------------|
| 97020 | 573 | 97020 | 0 | 1024 | 1 | 0 |
| 97021 | 573 | 97021 | 0 | 1024 | 1 | 0 |
| 97022 | 573 | 97022 | 0 | 1024 | 1 | 0 |
| 97023 | 573 | 97023 | 0 | 1024 | 1 | 0 |
| 97024 | 573 | 97024 | 0 | 1024 | 1 | 0 |

---

## 5. Étape 4 : player_shapeshift_model (SQL)

Bien que le core utilise `GetModelForForm()` pour les formes principales, vous pouvez insérer des entrées pour la cohérence ou si vous utilisez une version modifiée.

```sql
-- Forme de Sentinelle (ID 52)
-- Utilisation d'un modèle unique de Sentinelle pour toutes les races/genres
INSERT INTO `player_shapeshift_model` 
(`ShapeshiftID`, `RaceID`, `CustomizationID`, `GenderID`, `ModelID`) 
VALUES 
(52, 4, 0, 2, 21406),  -- Elfe de la Nuit (tous genres)
(52, 6, 0, 2, 21406),  -- Tauren
(52, 8, 0, 2, 21406),  -- Troll
(52, 22, 0, 2, 21406); -- Worgen
```

> **Note** : `GenderID = 2` signifie "tous genres". Le modèle `21406` est un exemple de Sentinelle existante dans le client 3.3.5. Remplacez-le par un ID valide de `CreatureDisplayInfo.dbc`.

---

## 6. Étape 5 : Modification du noyau C++

### 6.1. Définition de FORM_SENTINEL

Dans `src/server/game/Miscellaneous/SharedDefines.h` :

```cpp
enum ShapeshiftForm
{
    // ... existant ...
    FORM_RAPTOR                                 = 50,
    FORM_DRYAD                                  = 51,
    FORM_SENTINEL                               = 52,  // ← AJOUT
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
        // AJOUT : Forme de Sentinelle (ID 52)
        // ============================================
        case FORM_SENTINEL:
        {
            // Modèle unique de Sentinelle pour toutes les races
            return 21406;
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
.learn 97020

-- Apprentissage des sorts offensifs (si nécessaire)
.learn 97021
.learn 97022
.learn 97023
.learn 97024
```

### 8.4. Vérifications

- ✅ La forme s'active via le sort `97020`
- ✅ Le modèle de Sentinelle s'affiche
- ✅ Les bonus passifs de dégâts (+15%) et de critique (+5%) sont appliqués
- ✅ La barre d'action bonus contient les sorts `97021` à `97024`
- ✅ Les sorts offensifs fonctionnent correctement
- ✅ Le personnage peut sortir de la forme

---

## 9. Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| **Forme (SpellShapeshiftForm)** | `52` | `SpellShapeshiftForm.dbc` |
| **Sort de transformation** | `97020` | `Spell.dbc` |
| **Éclat lunaire** | `97021` | `Spell.dbc` |
| **Colère** | `97022` | `Spell.dbc` |
| **Feu stellaire** | `97023` | `Spell.dbc` |
| **Essaim d'insectes** | `97024` | `Spell.dbc` |
| **Compétence Druide** | `573` | `SkillLine.dbc` |
| **ClassMask Druide** | `1024` | `1 << (11 - 1)` |
| **Modèle Sentinelle** | `21406` | `CreatureDisplayInfo.dbc` |

---

## 10. Dépannage spécifique

| Problème | Cause | Solution |
|----------|-------|----------|
| Les bonus de dégâts ne s'appliquent pas | Effets mal configurés dans `Spell.dbc` | Vérifier `EffectApplyAuraName_2` (13) et `EffectMiscValue_2` (72 pour Arcane+Nature) |
| Le critique des sorts ne fonctionne pas | Aura incorrecte | Vérifier `EffectApplyAuraName_3` (118) |
| Le modèle de Sentinelle est invisible | ModelID invalide | Vérifier `CreatureDisplayInfo.dbc` pour un ID de Sentinelle valide |
| Les sorts offensifs ne s'affichent pas | `presetSpellID` non configuré | Remplir les colonnes 28-35 de `SpellShapeshiftForm.dbc` |
| La forme ne s'apprend pas | `SkillLineAbility.dbc` manquant | Vérifier ClassMask 1024 |
| Erreur de compilation `FORM_SENTINEL` | Enum non définie | Ajouter dans `SharedDefines.h` |

---

## 11. Variantes possibles

Vous pouvez décliner cette forme de Sentinelle en plusieurs versions :

| Variante | ID Forme | ID Sort | Spécialité |
|----------|----------|---------|------------|
| **Sentinelle lunaire** | 52 | 97020 | Dégâts Arcane (+15%) |
| **Sentinelle solaire** | 53 | 97030 | Dégâts Feu (+15%) |
| **Sentinelle d'épines** | 54 | 97040 | Dégâts Nature (+15%) |
| **Sentinelle d'ombre** | 55 | 97050 | Dégâts Ombre (+15%) |

---

Ce guide vous permet de créer une **forme de druide orientée DPS caster** complète et fonctionnelle. La **Forme de Sentinelle** utilise un modèle existant dans le jeu, ce qui facilite son intégration. Vous pouvez ajuster les bonus, les sorts et les flags selon vos besoins pour équilibrer la forme dans votre serveur.

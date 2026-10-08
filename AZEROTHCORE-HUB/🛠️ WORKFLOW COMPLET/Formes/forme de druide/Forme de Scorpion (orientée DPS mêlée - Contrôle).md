# Création d'une nouvelle forme de druide : **Forme de Scorpion** (orientée DPS mêlée / Contrôle)

Après la **Forme de Raptor** (DPS mêlée), la **Forme de Dryade** (Soin), la **Forme de Sentinelle** (DPS caster), la **Forme de Tortue** (Tank), la **Forme de Fée** (Soutien) et la **Forme de Serpent** (DPS mêlée / Poison), voici une nouvelle forme dédiée au **DPS mêlée avec contrôle** : la **Forme de Scorpion**. Elle mise sur le poison, l’étourdissement et la réduction d’armure, avec une attaque rapide et une bonne survie grâce à sa carapace.

---

## 1. Concept de la forme

| Propriété | Valeur |
|-----------|--------|
| **Nom** | Forme de Scorpion |
| **Type** | Forme de DPS mêlée / Contrôle |
| **Spécialité** | Poison, étourdissement, réduction d’armure, hâte |
| **ID de forme** | `56` |
| **ID du sort de transformation** | `97060` |
| **Compétence** | Druide (573) |
| **Classes** | Druide uniquement (ClassMask `1024`) |
| **Races** | Toutes les races druidiques (Elfe de la Nuit, Tauren, Troll, Worgen) |

---

## 2. Étape 1 : SpellShapeshiftForm.dbc

### 2.1. Définition de la forme

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `56` | ID unique de la forme |
| 2 | **ActionBar** | `1` | Barre d’action bonus activée |
| 3-19 | **Name** | `Forme de Scorpion` | Nom affiché |
| 20 | **Flags** | `0x00000030` | `Don't Use Weapon (0x10)` + `Agility Attack Bonus (0x20)` |
| 21 | **CreatureType** | `1` | Bête |
| 22 | **SpellIcon** | `1` | Icône de forme |
| 23 | **combatRoundTime** | `{2500, 1000}` | Temps de round de combat |
| 24 | **DisplayID_A** | `3679` | Modèle Alliance (Scorpion) — exemple |
| 25 | **DisplayID_H** | `3679` | Modèle Horde (Scorpion) |
| 28-35 | **presetSpellID[8]** | `97061`, `97062`, `97063`, `97064` | Sorts de scorpion prédéfinis |

### 2.2. Sorts prédéfinis (presetSpellID)

| ID | Nom | Description |
|----|-----|-------------|
| `97061` | Morsure de Scorpion | Attaque directe + réduction d’armure |
| `97062` | Dard empoisonné | Poison sur la durée |
| `97063` | Pincement | Étourdissement court |
| `97064` | Carapace de Chitine | Cooldown défensif : armure et réduction des dégâts |

---

## 3. Étape 2 : Spell.dbc — Sort de transformation et sorts offensifs

### 3.1. Sort de transformation (97060)

Ce sort applique la forme et des bonus passifs de DPS mêlée.

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97060` | ID unique du sort |
| 5 | **Attributes** | `0x00000010` | `SPELL_ATTR0_IS_ABILITY` |
| 18-19 | **TargetType** | `0` | Sort sur soi-même |
| 72 | **Effect_1** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 108 | **EffectApplyAuraName_1** | `36` | `SPELL_AURA_MOD_SHAPESHIFT` |
| 113 | **EffectMiscValue_1** | `56` | ID de la forme (SpellShapeshiftForm.dbc) |
| 73 | **Effect_2** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 109 | **EffectApplyAuraName_2** | `13` | `SPELL_AURA_MOD_DAMAGE_DONE` |
| 114 | **EffectMiscValue_2** | `8` | École : Nature (poison) |
| 81 | **EffectBasePoints_2** | `10` | +10% de dégâts nature |
| 74 | **Effect_3** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 110 | **EffectApplyAuraName_3** | `216` | `SPELL_AURA_MOD_HASTE` (hâte de mêlée) |
| 82 | **EffectBasePoints_3** | `15` | +15% de hâte |
| 75 | **Effect_4** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 111 | **EffectApplyAuraName_4** | `118` | `SPELL_AURA_MOD_SPELL_CRIT_CHANCE` (ou `SPELL_AURA_MOD_MELEE_CRIT_CHANCE` selon DBC) |
| 83 | **EffectBasePoints_4** | `5` | +5% de critique |
| 128 | **SpellIconID** | `1` | Icône |
| 131-162 | **Name** | `Forme de Scorpion` | Nom du sort |
| 171-178 | **Description** | `Vous transforme en scorpion, augmentant vos dégâts nature de 10%, votre hâte de 15% et votre critique de 5%.` | Description |
| 40 | **Duration** | `-1` | Permanent tant que la forme est active |

> **Note** : Pour le critique de mêlée, l’aura exacte peut être `SPELL_AURA_MOD_MELEE_CRIT_CHANCE` (ID 52 ou autre selon la version). Vérifiez dans `SpellAuraDefines.h`.

### 3.2. Sorts offensifs prédéfinis

#### 97061 — Morsure de Scorpion (Dégâts + Réduction d’armure)
| Champ | Valeur |
|-------|--------|
| ID | `97061` |
| Effect_1 | `2` (SPELL_EFFECT_SCHOOL_DAMAGE) |
| EffectBasePoints | `280` |
| EffectMiscValue | `1` (Physique) |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `101` (SPELL_AURA_MOD_RESISTANCE_PCT) |
| EffectBasePoints_2 | `-10` | -10% armure |
| EffectMiscValue_2 | `1` (Armure) |
| Duration_2 | `15000` (15 sec) |
| Name | `Morsure de Scorpion` |
| Description | `Inflige des dégâts physiques et réduit l’armure de la cible de 10% pendant 15 secondes.` |

#### 97062 — Dard empoisonné (Poison DoT)
| Champ | Valeur |
|-------|--------|
| ID | `97062` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `3` (SPELL_AURA_PERIODIC_DAMAGE) |
| EffectBasePoints | `90` |
| EffectAmplitude | `3000` (3 sec) |
| Duration | `15000` (15 sec) |
| EffectMiscValue | `8` (Nature) |
| Name | `Dard empoisonné` |
| Description | `Inflige X points de dégâts nature toutes les 3 secondes pendant 15 secondes.` |

#### 97063 — Pincement (Étourdissement)
| Champ | Valeur |
|-------|--------|
| ID | `97063` |
| Effect_1 | `2` (SPELL_EFFECT_SCHOOL_DAMAGE) |
| EffectBasePoints | `200` |
| EffectMiscValue | `1` (Physique) |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `12` (SPELL_AURA_MOD_STUN) |
| Duration_2 | `2000` (2 sec) |
| Cooldown | `20000` (20 sec) |
| Name | `Pincement` |
| Description | `Inflige des dégâts et étourdit la cible pendant 2 secondes.` |

#### 97064 — Carapace de Chitine (Cooldown défensif)
| Champ | Valeur |
|-------|--------|
| ID | `97064` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `52` (SPELL_AURA_MOD_RESISTANCE_PCT) |
| EffectBasePoints | `50` | +50% armure |
| EffectMiscValue | `1` (Armure) |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `87` (SPELL_AURA_MOD_DAMAGE_PERCENT_TAKEN) |
| EffectBasePoints_2 | `-20` | -20% dégâts subis |
| Duration | `12000` (12 sec) |
| Cooldown | `120000` (2 min) |
| Name | `Carapace de Chitine` |
| Description | `Augmente votre armure de 50% et réduit les dégâts subis de 20% pendant 12 secondes.` |

---

## 4. Étape 3 : SkillLineAbility.dbc

Liez le sort de transformation et les sorts offensifs à la compétence de druide.

| ID | SkillLine | Spell | RaceMask | ClassMask | MinSkillLineRank | AcquireMethod |
|----|-----------|-------|----------|-----------|------------------|---------------|
| 97060 | 573 | 97060 | 0 | 1024 | 1 | 0 |
| 97061 | 573 | 97061 | 0 | 1024 | 1 | 0 |
| 97062 | 573 | 97062 | 0 | 1024 | 1 | 0 |
| 97063 | 573 | 97063 | 0 | 1024 | 1 | 0 |
| 97064 | 573 | 97064 | 0 | 1024 | 1 | 0 |

---

## 5. Étape 4 : player_shapeshift_model (SQL)

```sql
-- Forme de Scorpion (ID 56)
-- Utilisation d'un modèle unique de Scorpion pour toutes les races/genres
INSERT INTO `player_shapeshift_model` 
(`ShapeshiftID`, `RaceID`, `CustomizationID`, `GenderID`, `ModelID`) 
VALUES 
(56, 4, 0, 2, 3679),  -- Elfe de la Nuit (tous genres)
(56, 6, 0, 2, 3679),  -- Tauren
(56, 8, 0, 2, 3679),  -- Troll
(56, 22, 0, 2, 3679); -- Worgen
```

> **Note** : `GenderID = 2` signifie « tous genres ». Le modèle `3679` est un exemple de scorpion existant dans le client 3.3.5. Remplacez-le par un ID valide de `CreatureDisplayInfo.dbc`.

---

## 6. Étape 5 : Modification du noyau C++

### 6.1. Définition de FORM_SCORPION

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
    FORM_SCORPION                               = 56,  // ← AJOUT
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
        // AJOUT : Forme de Scorpion (ID 56)
        // ============================================
        case FORM_SCORPION:
        {
            // Modèle unique de Scorpion pour toutes les races
            return 3679;
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
.learn 97060

-- Apprentissage des sorts offensifs (si nécessaire)
.learn 97061
.learn 97062
.learn 97063
.learn 97064
```

### 8.4. Vérifications

- ✅ La forme s’active via le sort `97060`
- ✅ Le modèle de Scorpion s’affiche
- ✅ Les bonus passifs (+10% dégâts nature, +15% hâte, +5% critique) sont appliqués
- ✅ La barre d’action bonus contient les sorts `97061` à `97064`
- ✅ Les sorts offensifs fonctionnent correctement
- ✅ Le personnage peut sortir de la forme

---

## 9. Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| **Forme (SpellShapeshiftForm)** | `56` | `SpellShapeshiftForm.dbc` |
| **Sort de transformation** | `97060` | `Spell.dbc` |
| **Morsure de Scorpion** | `97061` | `Spell.dbc` |
| **Dard empoisonné** | `97062` | `Spell.dbc` |
| **Pincement** | `97063` | `Spell.dbc` |
| **Carapace de Chitine** | `97064` | `Spell.dbc` |
| **Compétence Druide** | `573` | `SkillLine.dbc` |
| **ClassMask Druide** | `1024` | `1 << (11 - 1)` |
| **Modèle Scorpion** | `3679` | `CreatureDisplayInfo.dbc` |

---

## 10. Dépannage spécifique

| Problème | Cause | Solution |
|----------|-------|----------|
| Les bonus de dégâts nature ne s’appliquent pas | Effets mal configurés dans `Spell.dbc` | Vérifier `EffectApplyAuraName_2` (13) et `EffectMiscValue_2` (8) |
| La hâte ne fonctionne pas | Aura incorrecte | Vérifier `EffectApplyAuraName_3` (216) |
| Le critique ne s’applique pas | Aura incorrecte | Vérifier `EffectApplyAuraName_4` (118 ou 52 selon DBC) |
| Le modèle de Scorpion est invisible | ModelID invalide | Vérifier `CreatureDisplayInfo.dbc` pour un ID de scorpion valide |
| Les sorts offensifs ne s’affichent pas | `presetSpellID` non configuré | Remplir les colonnes 28-35 de `SpellShapeshiftForm.dbc` |
| La forme ne s’apprend pas | `SkillLineAbility.dbc` manquant | Vérifier ClassMask 1024 |
| Erreur de compilation `FORM_SCORPION` | Enum non définie | Ajouter dans `SharedDefines.h` |

---

## 11. Variantes possibles

Vous pouvez décliner cette forme de Scorpion en plusieurs versions :

| Variante | ID Forme | ID Sort | Spécialité |
|----------|----------|---------|------------|
| **Scorpion venimeux** | 56 | 97060 | Poison (+10% dégâts nature) |
| **Scorpion des sables** | 57 | 97070 | Réduction d’armure + hâte |
| **Scorpion noir** | 58 | 97080 | Étourdissement + saignement |
| **Scorpion de magma** | 59 | 97090 | Dégâts de feu + armure |

---

Ce guide vous permet de créer une **forme de druide orientée DPS mêlée / Contrôle** complète et fonctionnelle. La **Forme de Scorpion** utilise un modèle existant dans le jeu, ce qui facilite son intégration. Vous pouvez ajuster les bonus, les sorts et les flags selon vos besoins pour équilibrer la forme dans votre serveur.

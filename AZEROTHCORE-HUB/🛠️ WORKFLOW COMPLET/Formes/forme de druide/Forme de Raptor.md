# Création d'une nouvelle forme de druide : **Forme de Raptor**

Voici un exemple complet et concret de création d'une nouvelle forme de druide, la **Forme de Raptor**, basée sur le guide fourni. Cette forme est orientée DPS mêlée rapide, avec des attaques empoisonnées et une mobilité accrue.

---

## 1. Concept de la forme

| Propriété | Valeur |
|-----------|--------|
| **Nom** | Forme de Raptor |
| **Type** | Forme de DPS mêlée (alternative à la Forme de Loup) |
| **Spécialité** | Vitesse d'attaque élevée, dégâts de poison, saignement |
| **ID de forme** | `50` |
| **ID du sort** | `97000` |
| **Compétence** | Druide (573) |
| **Classes** | Druide uniquement (ClassMask `1024`) |
| **Races** | Toutes les races druidiques (Elfe de la Nuit, Tauren, Troll, Worgen) |

---

## 2. Étape 1 : SpellShapeshiftForm.dbc

### 2.1. Définition de la forme

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `50` | ID unique de la forme |
| 2 | **ActionBar** | `1` | Barre d'action bonus activée |
| 3-19 | **Name** | `Forme de Raptor` | Nom affiché |
| 20 | **Flags** | `0x00000030` | `Don't Use Weapon (0x10)` + `Agility Attack Bonus (0x20)` |
| 21 | **CreatureType** | `1` | Bête |
| 22 | **SpellIcon** | `1` | Icône de forme |
| 23 | **combatRoundTime** | `{2500, 1000}` | Temps de round de combat (comme les autres formes de druide) |
| 24 | **DisplayID_A** | `14334` | Modèle Alliance (Raptor) — ignoré par le core pour les formes principales |
| 25 | **DisplayID_H** | `14335` | Modèle Horde (Raptor) — ignoré par le core pour les formes principales |
| 28-35 | **presetSpellID[8]** | `97001`, `97002`, `97003`, `97004` | Sorts prédéfinis de la forme |

### 2.2. Sorts prédéfinis (presetSpellID)

| ID | Nom | Description |
|----|-----|-------------|
| `97001` | Morsure de Raptor | Attaque de mêlée principale |
| `97002` | Griffe Déchirante | Saignement sur la cible |
| `97003` | Crachat Venimeux | Poison (dégâts nature sur la durée) |
| `97004` | Bond de Raptor | Charge rapide vers la cible |

---

## 3. Étape 2 : Spell.dbc — Sort de transformation

### 3.1. Configuration du sort de transformation

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97000` | ID unique du sort |
| 5 | **Attributes** | `0x00000010` | `SPELL_ATTR0_IS_ABILITY` |
| 18-19 | **TargetType** | `0` | Sort sur soi-même |
| 72 | **Effect_1** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 108 | **EffectApplyAuraName** | `36` | `SPELL_AURA_MOD_SHAPESHIFT` |
| 113 | **EffectMiscValue** | `50` | ID de la forme (SpellShapeshiftForm.dbc) |
| 128 | **SpellIconID** | `1` | Icône |
| 131-162 | **Name** | `Forme de Raptor` | Nom du sort |
| 171-178 | **Description** | `Vous transforme en raptor, augmentant votre vitesse d'attaque et vous donnant accès aux attaques de raptor.` | Description |

### 3.2. Sorts prédéfinis (Spell.dbc)

#### 97001 — Morsure de Raptor
| Champ | Valeur |
|-------|--------|
| ID | `97001` |
| Effect_1 | `2` (SPELL_EFFECT_SCHOOL_DAMAGE) |
| EffectBasePoints | `40` |
| EffectMiscValue | `0` |
| SpellIconID | `1` |
| Name | `Morsure de Raptor` |

#### 97002 — Griffe Déchirante
| Champ | Valeur |
|-------|--------|
| ID | `97002` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `3` (SPELL_AURA_PERIODIC_DAMAGE) |
| EffectBasePoints | `15` |
| EffectAmplitude | `2000` (2 sec) |
| Duration | `12000` (12 sec) |
| Name | `Griffe Déchirante` |

#### 97003 — Crachat Venimeux
| Champ | Valeur |
|-------|--------|
| ID | `97003` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `3` (PERIODIC_DAMAGE) |
| EffectBasePoints | `20` |
| EffectAmplitude | `3000` (3 sec) |
| Duration | `15000` (15 sec) |
| SchoolMask | `8` (Nature) |
| Name | `Crachat Venimeux` |

#### 97004 — Bond de Raptor
| Champ | Valeur |
|-------|--------|
| ID | `97004` |
| Effect_1 | `96` (SPELL_EFFECT_CHARGE) |
| EffectBasePoints | `8` (portée) |
| EffectMiscValue | `0` |
| Name | `Bond de Raptor` |

---

## 4. Étape 3 : SkillLineAbility.dbc

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97000` | ID unique |
| 2 | **SkillLine** | `573` | Compétence de druide |
| 3 | **Spell** | `97000` | Sort de transformation |
| 4 | **RaceMask** | `0` | Toutes races |
| 5 | **ClassMask** | `1024` | Druide |
| 8 | **MinSkillLineRank** | `1` | Rang minimum |
| 10 | **AcquireMethod** | `0` | Apprentissage standard |

### 4.1. Entrées pour les sorts prédéfinis

| ID | SkillLine | Spell | ClassMask | AcquireMethod |
|----|-----------|-------|-----------|---------------|
| 97001 | 573 | 97001 | 1024 | 0 |
| 97002 | 573 | 97002 | 1024 | 0 |
| 97003 | 573 | 97003 | 1024 | 0 |
| 97004 | 573 | 97004 | 1024 | 0 |

---

## 5. Étape 4 : player_shapeshift_model (SQL)

```sql
-- Forme de Raptor (ID 50)
-- Elfe de la Nuit (Race 4)
INSERT INTO `player_shapeshift_model` 
(`ShapeshiftID`, `RaceID`, `CustomizationID`, `GenderID`, `ModelID`) 
VALUES 
(50, 4, 0, 0, 14334),  -- Elfe de la Nuit Masculin
(50, 4, 0, 1, 14335),  -- Elfe de la Nuit Féminin
(50, 4, 1, 0, 14336),  -- Variante de peau 1
(50, 4, 1, 1, 14337),  -- Variante de peau 1 féminin
-- Tauren (Race 6)
(50, 6, 0, 0, 14338),  -- Tauren Masculin
(50, 6, 0, 1, 14339),  -- Tauren Féminin
-- Troll (Race 8)
(50, 8, 0, 0, 14340),  -- Troll Masculin
(50, 8, 0, 1, 14341),  -- Troll Féminin
-- Worgen (Race 22)
(50, 22, 0, 0, 14342), -- Worgen Masculin
(50, 22, 0, 1, 14343); -- Worgen Féminin
```

> **📝 Note** : Les ModelID doivent correspondre à des entrées valides dans `CreatureDisplayInfo.dbc`. Pour un raptor, les modèles existants dans le jeu peuvent être utilisés (par exemple, les raptors de Stranglethorn Vale ou d'Un'Goro).

---

## 6. Étape 5 : Modification du noyau C++ (GetModelForForm)

Pour que la forme de Raptor utilise un modèle spécifique (et non le modèle par défaut de la race), il faut modifier la fonction `GetModelForForm()` dans `src/server/game/Entities/Unit/Unit.cpp`.

### 6.1. Localisation

```cpp
uint32 Unit::GetModelForForm(ShapeshiftForm form) const
{
    // ...
}
```

### 6.2. Modification

```cpp
uint32 Unit::GetModelForForm(ShapeshiftForm form) const
{
    // ... code existant ...

    switch (form)
    {
        case FORM_CAT:
            // ... code existant pour le loup ...
            break;
        case FORM_BEAR:
            // ... code existant pour l'ours ...
            break;
        case FORM_DIREBEAR:
            // ... code existant pour l'ours redoutable ...
            break;

        // ============================================
        // AJOUT : Forme de Raptor (ID 50)
        // ============================================
        case FORM_RAPTOR:  // À définir dans SharedDefines.h
        {
            uint32 modelId = 0;
            
            // Sélection du modèle selon la race et le genre
            switch (getRace())
            {
                case RACE_NIGHTELF:
                    modelId = (getGender() == GENDER_FEMALE) ? 14335 : 14334;
                    break;
                case RACE_TAUREN:
                    modelId = (getGender() == GENDER_FEMALE) ? 14339 : 14338;
                    break;
                case RACE_TROLL:
                    modelId = (getGender() == GENDER_FEMALE) ? 14341 : 14340;
                    break;
                case RACE_WORGEN:
                    modelId = (getGender() == GENDER_FEMALE) ? 14343 : 14342;
                    break;
                default:
                    modelId = 14334; // Modèle par défaut (Elfe de la Nuit masculin)
                    break;
            }
            
            return modelId;
        }

        default:
            break;
    }

    // ... code existant ...
}
```

### 6.3. Définition de FORM_RAPTOR

Dans `src/server/game/Miscellaneous/SharedDefines.h` :

```cpp
enum ShapeshiftForm
{
    FORM_NONE                                   = 0,
    FORM_CAT                                    = 1,
    FORM_TREE                                   = 2,
    FORM_TRAVEL                                 = 3,
    FORM_AQUA                                   = 4,
    FORM_BEAR                                   = 5,
    FORM_DIREBEAR                               = 6,
    FORM_GHOSTWOLF                              = 16,
    FORM_BATTLESTANCE                           = 17,
    FORM_DEFENSIVESTANCE                        = 18,
    FORM_BERSERKERSTANCE                        = 19,
    FORM_SHADOW                                 = 28,
    FORM_STEALTH                                = 30,
    FORM_MOONKIN                               = 31,
    FORM_SPIRITOFREDEMPTION                     = 32,
    // ...
    FORM_RAPTOR                                 = 50,  // ← AJOUT
};
```

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
# Dans le dossier de build d'AzerothCore
cmake --build . --config Release
```

### 8.2. Redémarrage

```bash
# Redémarrer le serveur world
./worldserver
```

### 8.3. Test en jeu

```sql
-- Apprentissage du sort via commande MJ
.learn 97000

-- Ou via un PNJ entraîneur (ajouter dans npc_trainer)
INSERT INTO `npc_trainer` (`entry`, `spell`, `spellcost`, `reqskill`, `reqskillvalue`, `reqlevel`)
VALUES (100003, 97000, 5000, 0, 0, 20);
```

### 8.4. Vérifications

- ✅ La forme s'active via le sort 97000
- ✅ Le modèle de raptor s'affiche correctement
- ✅ La barre d'action bonus apparaît avec les 4 sorts prédéfinis
- ✅ Les attaques de raptor fonctionnent (Morsure, Griffe, Crachat, Bond)
- ✅ Le personnage peut sortir de la forme (sauf si `Not Toggleable` activé)

---

## 9. Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| **Forme (SpellShapeshiftForm)** | `50` | `SpellShapeshiftForm.dbc` |
| **Sort de transformation** | `97000` | `Spell.dbc` |
| **Morsure de Raptor** | `97001` | `Spell.dbc` |
| **Griffe Déchirante** | `97002` | `Spell.dbc` |
| **Crachat Venimeux** | `97003` | `Spell.dbc` |
| **Bond de Raptor** | `97004` | `Spell.dbc` |
| **Compétence Druide** | `573` | `SkillLine.dbc` |
| **ClassMask Druide** | `1024` | `1 << (11 - 1)` |
| **Modèle Elfe Nuit M** | `14334` | `CreatureDisplayInfo.dbc` |
| **Modèle Elfe Nuit F** | `14335` | `CreatureDisplayInfo.dbc` |
| **Modèle Tauren M** | `14338` | `CreatureDisplayInfo.dbc` |
| **Modèle Tauren F** | `14339` | `CreatureDisplayInfo.dbc` |

---

## 10. Dépannage spécifique

| Problème | Cause | Solution |
|----------|-------|----------|
| La forme s'active mais modèle invisible | ModelID invalide | Vérifier `CreatureDisplayInfo.dbc` |
| Les sorts ne s'affichent pas | `presetSpellID` mal configuré | Vérifier colonnes 28-35 de `SpellShapeshiftForm.dbc` |
| Le modèle est celui d'un loup | `GetModelForForm()` non modifié | Ajouter le case `FORM_RAPTOR` |
| Erreur de compilation `FORM_RAPTOR` | Enum non définie | Ajouter dans `SharedDefines.h` |
| La forme ne s'apprend pas | `SkillLineAbility.dbc` manquant | Vérifier ClassMask 1024 |
| Le personnage reste coincé | Flags incorrects | Retirer `Not Toggleable` (0x02) |

---

## 11. Variantes possibles

Vous pouvez décliner cette forme de Raptor en plusieurs versions :

| Variante | ID Forme | ID Sort | Spécialité |
|----------|----------|---------|------------|
| **Raptor de Venin** | 50 | 97000 | DPS poison |
| **Raptor des Sables** | 51 | 97010 | DPS saignement |
| **Raptor Ancien** | 52 | 97020 | Tank (haute armure) |
| **Raptor Spectral** | 53 | 97030 | Furtivité améliorée |

---

Ce guide vous permet de créer une forme de druide complète et fonctionnelle. La **Forme de Raptor** est un excellent exemple car elle utilise des modèles existants dans le jeu (raptors), ce qui évite d'avoir à importer de nouveaux modèles 3D. Vous pouvez ensuite personnaliser les sorts, les flags et les modèles selon vos besoins.

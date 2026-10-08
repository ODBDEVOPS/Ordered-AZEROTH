# Création d'une nouvelle forme de druide : **Forme de Fée** (orientée Soutien)

Après la **Forme de Raptor** (DPS mêlée), la **Forme de Dryade** (Soin), la **Forme de Sentinelle** (DPS caster) et la **Forme de Tortue** (Tank), voici une nouvelle forme dédiée au **soutien** : la **Forme de Fée**. Elle offre des bonus passifs de régénération de mana et de hâte des sorts, ainsi que des sorts utilitaires et de buff de groupe. C’est une forme hybride entre le soin et le support, idéale pour les druides qui souhaitent aider leur groupe sans être exclusively heal.

---

## 1. Concept de la forme

| Propriété | Valeur |
|-----------|--------|
| **Nom** | Forme de Fée |
| **Type** | Forme de Soutien (buff, régénération, utilité) |
| **Spécialité** | Régénération de mana, hâte des sorts, buffs de groupe, utilité |
| **ID de forme** | `54` |
| **ID du sort de transformation** | `97040` |
| **Compétence** | Druide (573) |
| **Classes** | Druide uniquement (ClassMask `1024`) |
| **Races** | Toutes les races druidiques (Elfe de la Nuit, Tauren, Troll, Worgen) |

---

## 2. Étape 1 : SpellShapeshiftForm.dbc

### 2.1. Définition de la forme

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `54` | ID unique de la forme |
| 2 | **ActionBar** | `1` | Barre d’action bonus activée |
| 3-19 | **Name** | `Forme de Fée` | Nom affiché |
| 20 | **Flags** | `0x000001C8` | `Can Interact NPC (0x08)` + `Can Use Equipped Items (0x40)` + `Can Use Items (0x80)` + `Don't Auto-Unshift (0x100)` |
| 21 | **CreatureType** | `7` | Humanoid (Fée) |
| 22 | **SpellIcon** | `1` | Icône de forme |
| 23 | **combatRoundTime** | `{2500, 1000}` | Temps de round de combat |
| 24 | **DisplayID_A** | `10044` | Modèle Alliance (Fée) — exemple |
| 25 | **DisplayID_H** | `10044` | Modèle Horde (Fée) |
| 28-35 | **presetSpellID[8]** | `97041`, `97042`, `97043`, `97044` | Sorts de soutien prédéfinis |

### 2.2. Sorts prédéfinis (presetSpellID)

| ID | Nom | Description |
|----|-----|-------------|
| `97041` | Feu féerique | Réduit l’armure de la cible et l’empêche de se camoufler |
| `97042` | Innervation | Rend de la mana à un allié |
| `97043` | Bénédiction de la Fée | Buff de groupe : +10% hâte des sorts, +10% régénération de mana |
| `97044` | Poussière de Fée | Utilitaire : réduit la menace et augmente la vitesse de déplacement |

---

## 3. Étape 2 : Spell.dbc — Sort de transformation et sorts de soutien

### 3.1. Sort de transformation (97040)

Ce sort applique la forme et des bonus passifs de soutien.

| Colonne | Champ | Valeur | Description |
|---------|-------|--------|-------------|
| 1 | **ID** | `97040` | ID unique du sort |
| 5 | **Attributes** | `0x00000010` | `SPELL_ATTR0_IS_ABILITY` |
| 18-19 | **TargetType** | `0` | Sort sur soi-même |
| 72 | **Effect_1** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 108 | **EffectApplyAuraName_1** | `36` | `SPELL_AURA_MOD_SHAPESHIFT` |
| 113 | **EffectMiscValue_1** | `54` | ID de la forme (SpellShapeshiftForm.dbc) |
| 73 | **Effect_2** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 109 | **EffectApplyAuraName_2** | `96` | `SPELL_AURA_MOD_MANA_REGEN_PCT` |
| 81 | **EffectBasePoints_2** | `100` | +100% de régénération de mana |
| 74 | **Effect_3** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 110 | **EffectApplyAuraName_3** | `216` | `SPELL_AURA_MOD_HASTE` (hâte des sorts) |
| 82 | **EffectBasePoints_3** | `10` | +10% de hâte des sorts |
| 75 | **Effect_4** | `6` | `SPELL_EFFECT_APPLY_AURA` |
| 111 | **EffectApplyAuraName_4** | `118` | `SPELL_AURA_MOD_SPELL_CRIT_CHANCE` |
| 83 | **EffectBasePoints_4** | `5` | +5% de critique des sorts |
| 128 | **SpellIconID** | `1` | Icône |
| 131-162 | **Name** | `Forme de Fée` | Nom du sort |
| 171-178 | **Description** | `Vous transforme en fée, augmentant votre régénération de mana de 100%, votre hâte des sorts de 10% et votre critique des sorts de 5%.` | Description |
| 40 | **Duration** | `-1` | Permanent tant que la forme est active |

### 3.2. Sorts de soutien prédéfinis

#### 97041 — Feu féerique (Débuff)
| Champ | Valeur |
|-------|--------|
| ID | `97041` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `101` (SPELL_AURA_MOD_RESISTANCE_PCT) |
| EffectBasePoints | `-5` | Réduit l’armure de 5% |
| EffectMiscValue | `1` (Armure) |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `16` (SPELL_AURA_MOD_INVISIBILITY) |
| Duration | `40000` (40 sec) |
| Name | `Feu féerique` |
| Description | `Réduit l’armure de la cible de 5% et l’empêche de se camoufler pendant 40 secondes.` |

#### 97042 — Innervation (Rendu de mana)
| Champ | Valeur |
|-------|--------|
| ID | `97042` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `24` (SPELL_AURA_PERIODIC_ENERGIZE) |
| EffectBasePoints | `200` |
| EffectAmplitude | `1000` (1 sec) |
| Duration | `10000` (10 sec) |
| Name | `Innervation` |
| Description | `Rend X points de mana à la cible toutes les secondes pendant 10 secondes.` |

#### 97043 — Bénédiction de la Fée (Buff de groupe)
| Champ | Valeur |
|-------|--------|
| ID | `97043` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `216` (SPELL_AURA_MOD_HASTE) |
| EffectBasePoints | `10` |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `96` (SPELL_AURA_MOD_MANA_REGEN_PCT) |
| EffectBasePoints_2 | `10` |
| TargetType | `21` (TARGET_UNIT_CASTER_AREA_PARTY) ou `22` (RAID) |
| Duration | `1800000` (30 min) |
| Name | `Bénédiction de la Fée` |
| Description | `Augmente la hâte des sorts et la régénération de mana de tous les membres du groupe de 10% pendant 30 minutes.` |

#### 97044 — Poussière de Fée (Utilitaire)
| Champ | Valeur |
|-------|--------|
| ID | `97044` |
| Effect_1 | `6` (APPLY_AURA) |
| EffectApplyAuraName | `103` (SPELL_AURA_MOD_SPEED_ALWAYS) |
| EffectBasePoints | `40` | +40% vitesse |
| Effect_2 | `6` (APPLY_AURA) |
| EffectApplyAuraName_2 | `77` (SPELL_AURA_MOD_THREAT) |
| EffectBasePoints_2 | `-50` | -50% menace |
| Duration | `10000` (10 sec) |
| Cooldown | `60000` (1 min) |
| Name | `Poussière de Fée` |
| Description | `Augmente votre vitesse de déplacement de 40% et réduit votre menace de 50% pendant 10 secondes.` |

---

## 4. Étape 3 : SkillLineAbility.dbc

Liez le sort de transformation et les sorts de soutien à la compétence de druide.

| ID | SkillLine | Spell | RaceMask | ClassMask | MinSkillLineRank | AcquireMethod |
|----|-----------|-------|----------|-----------|------------------|---------------|
| 97040 | 573 | 97040 | 0 | 1024 | 1 | 0 |
| 97041 | 573 | 97041 | 0 | 1024 | 1 | 0 |
| 97042 | 573 | 97042 | 0 | 1024 | 1 | 0 |
| 97043 | 573 | 97043 | 0 | 1024 | 1 | 0 |
| 97044 | 573 | 97044 | 0 | 1024 | 1 | 0 |

---

## 5. Étape 4 : player_shapeshift_model (SQL)

```sql
-- Forme de Fée (ID 54)
-- Utilisation d'un modèle unique de Fée pour toutes les races/genres
INSERT INTO `player_shapeshift_model` 
(`ShapeshiftID`, `RaceID`, `CustomizationID`, `GenderID`, `ModelID`) 
VALUES 
(54, 4, 0, 2, 10044),  -- Elfe de la Nuit (tous genres)
(54, 6, 0, 2, 10044),  -- Tauren
(54, 8, 0, 2, 10044),  -- Troll
(54, 22, 0, 2, 10044); -- Worgen
```

> **Note** : `GenderID = 2` signifie « tous genres ». Le modèle `10044` est un exemple de fée existante dans le client 3.3.5 (par exemple, un dragon féerique). Remplacez-le par un ID valide de `CreatureDisplayInfo.dbc`.

---

## 6. Étape 5 : Modification du noyau C++

### 6.1. Définition de FORM_FAERIE

Dans `src/server/game/Miscellaneous/SharedDefines.h` :

```cpp
enum ShapeshiftForm
{
    // ... existant ...
    FORM_RAPTOR                                 = 50,
    FORM_DRYAD                                  = 51,
    FORM_SENTINEL                               = 52,
    FORM_TURTLE                                 = 53,
    FORM_FAERIE                                 = 54,  // ← AJOUT
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
        // AJOUT : Forme de Fée (ID 54)
        // ============================================
        case FORM_FAERIE:
        {
            // Modèle unique de Fée pour toutes les races
            return 10044;
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
.learn 97040

-- Apprentissage des sorts de soutien (si nécessaire)
.learn 97041
.learn 97042
.learn 97043
.learn 97044
```

### 8.4. Vérifications

- ✅ La forme s’active via le sort `97040`
- ✅ Le modèle de Fée s’affiche
- ✅ Les bonus passifs (+100% mana, +10% hâte, +5% critique) sont appliqués
- ✅ La barre d’action bonus contient les sorts `97041` à `97044`
- ✅ Les sorts de soutien fonctionnent correctement
- ✅ Le personnage peut sortir de la forme

---

## 9. Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| **Forme (SpellShapeshiftForm)** | `54` | `SpellShapeshiftForm.dbc` |
| **Sort de transformation** | `97040` | `Spell.dbc` |
| **Feu féerique** | `97041` | `Spell.dbc` |
| **Innervation** | `97042` | `Spell.dbc` |
| **Bénédiction de la Fée** | `97043` | `Spell.dbc` |
| **Poussière de Fée** | `97044` | `Spell.dbc` |
| **Compétence Druide** | `573` | `SkillLine.dbc` |
| **ClassMask Druide** | `1024` | `1 << (11 - 1)` |
| **Modèle Fée** | `10044` | `CreatureDisplayInfo.dbc` |

---

## 10. Dépannage spécifique

| Problème | Cause | Solution |
|----------|-------|----------|
| Les bonus de mana/hâte ne s’appliquent pas | Effets mal configurés dans `Spell.dbc` | Vérifier `EffectApplyAuraName_2` (96), `EffectApplyAuraName_3` (216) |
| Le buff de groupe ne fonctionne pas | TargetType incorrect | Utiliser `21` (PARTY) ou `22` (RAID) pour `97043` |
| Le modèle de Fée est invisible | ModelID invalide | Vérifier `CreatureDisplayInfo.dbc` pour un ID de fée valide |
| Les sorts de soutien ne s’affichent pas | `presetSpellID` non configuré | Remplir les colonnes 28-35 de `SpellShapeshiftForm.dbc` |
| La forme ne s’apprend pas | `SkillLineAbility.dbc` manquant | Vérifier ClassMask 1024 |
| Erreur de compilation `FORM_FAERIE` | Enum non définie | Ajouter dans `SharedDefines.h` |

---

## 11. Variantes possibles

Vous pouvez décliner cette forme de Fée en plusieurs versions :

| Variante | ID Forme | ID Sort | Spécialité |
|----------|----------|---------|------------|
| **Fée de Mana** | 54 | 97040 | Régénération de mana (+100%) |
| **Fée de Hâte** | 55 | 97050 | Hâte des sorts (+15%) |
| **Fée de Critique** | 56 | 97060 | Critique des sorts (+10%) |
| **Fée de Protection** | 57 | 97070 | Bouclier absorbant les dégâts |

---

Ce guide vous permet de créer une **forme de druide orientée Soutien** complète et fonctionnelle. La **Forme de Fée** utilise un modèle existant dans le jeu, ce qui facilite son intégration. Vous pouvez ajuster les bonus, les sorts et les flags selon vos besoins pour équilibrer la forme dans votre serveur.

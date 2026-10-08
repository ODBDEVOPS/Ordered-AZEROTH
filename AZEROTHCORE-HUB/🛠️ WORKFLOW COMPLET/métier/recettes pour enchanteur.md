# Guide complet : Créer une recette d'enchantement dans AzerothCore 3.3.5

## Introduction

Ce guide détaille étape par étape la création d'une recette d'enchantement personnalisée pour AzerothCore 3.3.5 (Wrath of the Lich King). Il couvre la modification des fichiers DBC côté client et des tables de base de données côté serveur. À la fin, vous aurez un parchemin d'enchantement fonctionnel, utilisable par un joueur pour appliquer un enchantement sur une pièce d'équipement.

> **⚠️ Avertissement** : Toute modification des fichiers DBC nécessite la création d'un patch MPQ côté client. Les modifications de base de données, elles, sont appliquées côté serveur et ne nécessitent pas de patch client. Assurez-vous de toujours sauvegarder vos fichiers avant modification.

---

## Prérequis

Avant de commencer, assurez-vous de disposer des outils suivants :

- **Un éditeur DBC** : MyDBCEditor, DBC Editor, ou tout autre outil capable d'ouvrir et modifier les fichiers `.dbc`.
- **Un client de base de données** : HeidiSQL, MySQL Workbench, ou tout autre client pour interagir avec la base de données `world`.
- **Les fichiers DBC de référence** : `SpellItemEnchantment.dbc`, `Spell.dbc`, `Item.dbc`, extraits de votre client 3.3.5a.
- **Un outil de création de MPQ** : MPQ Editor (Ladik's MPQ Editor) pour packager les fichiers modifiés.
- **Accès au serveur** : Base de données `world` et possibilité de redémarrer le serveur.

---

## Vue d'ensemble du processus

La création d'une recette d'enchantement implique la modification de **cinq éléments** interconnectés :

1. **SpellItemEnchantment.dbc** — Définit l'effet de l'enchantement (statistiques, proc, etc.).
2. **Spell.dbc** — Définit le sort d'enchantement qui sera lancé par le parchemin.
3. **Item.dbc** — Référence l'objet parchemin côté client.
4. **item_template** (base de données `world`) — Définit l'objet parchemin côté serveur.
5. **spell_enchant_proc_data** (base de données `world`, optionnel) — Définit le comportement de proc de l'enchantement.

Voici un schéma illustrant les relations entre ces éléments :

```
┌─────────────────────────┐
│ SpellItemEnchantment.dbc │  ← Définit l'effet (ex: +10 Force)
│  ID: 3000               │
└───────────┬─────────────┘
            │  référencé par
            ▼
┌─────────────────────────┐
│       Spell.dbc          │  ← Sort d'enchantement (effet 53)
│  ID: 90000              │
│  MiscValue: 3000        │  ← ID de SpellItemEnchantment
│  ItemType: 90001        │  ← ID de l'objet parchemin
└───────────┬─────────────┘
            │  référencé par
            ▼
┌─────────────────────────┐
│      Item.dbc           │  ← Référence client de l'objet
│  ID: 90001              │
└───────────┬─────────────┘
            │  correspond à
            ▼
┌─────────────────────────┐
│    item_template        │  ← Définition serveur de l'objet
│  entry: 90001           │
│  spellid_1: 90000       │  ← ID du sort d'enchantement
└─────────────────────────┘
```

---

## Étape 1 : Définir l'effet d'enchantement dans SpellItemEnchantment.dbc

Le fichier `SpellItemEnchantment.dbc` contient toutes les définitions d'enchantements existants dans le jeu. C'est ici que vous définissez **ce que fait** votre enchantement.

### 1.1. Ouvrir et dupliquer une ligne

Ouvrez `SpellItemEnchantment.dbc` avec votre éditeur DBC. La méthode la plus simple consiste à **dupliquer une ligne existante** qui correspond au type d'enchantement souhaité, puis à modifier ses valeurs. Par exemple, pour un enchantement de statistiques (comme +10 Force), dupliquez une ligne existante d'enchantement de type « équipement » avec un effet de statistique.

### 1.2. Configurer les champs principaux

Voici les champs essentiels à configurer dans `SpellItemEnchantment.dbc` :

| Colonne | Champ | Description | Exemple |
|---------|-------|-------------|---------|
| 1 | **ID** | Identifiant unique de l'enchantement. Doit être unique et non utilisé. | `3000` |
| 2 | **Charges** | Nombre de charges (généralement 0, inutilisé en 3.3.5). | `0` |
| 3-5 | **Effect Type** | Type d'effet pour chaque slot (jusqu'à 3 effets). `3` = équipement, `5` = statistiques, `1` = proc au coup, `2` = dégâts directs. | `5, 0, 0` |
| 6-8 | **Effect Points Min** | Valeur minimale de l'effet (ex: 10 pour +10 Force). | `10, 0, 0` |
| 9-11 | **Effect Points Max** | Valeur maximale de l'effet (généralement identique à Min pour un enchantement fixe). | `10, 0, 0` |
| 12-14 | **Effect Arg** | Argument de l'effet (pour les statistiques : 0=Force, 1=Agilité, 3=Endurance, 7=Intelligence, etc.). Pour un type `5` (statistiques), cette colonne définit quelle statistique est augmentée. | `0, 0, 0` |
| 15-31 | **Name** | Nom de l'enchantement affiché sur le parchemin. | `+10 Force` |
| 32 | **ItemVisual** | Référence à `ItemVisuals.dbc` pour l'effet visuel (glow). Laisser `0` pour aucun effet visuel. | `0` |
| 33 | **Flags** | Drapeaux divers. Laisser `0` par défaut. | `0` |
| 34 | **SrcItemID** | Référence à `Item.dbc` (objet source). Laisser `0`. | `0` |
| 35 | **ConditionID** | Référence à `SpellItemEnchantmentCondition.dbc` pour les conditions. Laisser `0`. | `0` |
| 36 | **RequiredSkillID** | Compétence requise (référence `SkillLine.dbc`). Pour l'enchantement : `333`. | `333` |
| 37 | **RequiredSkillRank** | Niveau de compétence requis pour utiliser l'enchantement. | `300` |
| 38 | **MinLevel** | Niveau minimum du joueur pour utiliser l'enchantement. | `0` |

> **📝 Note sur les valeurs de statistiques** : Pour la colonne `Effect Arg` (12-14), les valeurs courantes sont : `0` = Force, `1` = Agilité, `3` = Endurance, `4` = Intelligence, `5` = Esprit, `6` = Puissance d'attaque, `7` = Puissance des sorts, `8` = Score de hâte, `9` = Score de critique, etc.

### 1.3. Exemple concret

**Enchantement « Puissance du Berserker » : +10 Force, +5 Agilité**

- **ID** : `3000`
- **Effect Type** : colonne 3 = `5` (statistiques), colonne 4 = `5` (statistiques), colonne 5 = `0`
- **Effect Points Min** : colonne 6 = `10`, colonne 7 = `5`, colonne 8 = `0`
- **Effect Points Max** : colonne 9 = `10`, colonne 10 = `5`, colonne 11 = `0`
- **Effect Arg** : colonne 12 = `0` (Force), colonne 13 = `1` (Agilité), colonne 14 = `0`
- **Name** : `Puissance du Berserker`
- **RequiredSkillID** : `333`
- **RequiredSkillRank** : `300`

---

## Étape 2 : Créer le sort d'enchantement dans Spell.dbc

Le fichier `Spell.dbc` définit le sort qui sera lancé lorsque le joueur utilisera le parchemin. C'est ce sort qui appliquera l'enchantement défini à l'étape 1.

### 2.1. Champs essentiels

| Colonne | Champ | Description | Valeur à utiliser |
|---------|-------|-------------|-------------------|
| 1 | **ID** | Identifiant unique du sort. | `90000` |
| 2 | **Category** | Catégorie du sort. Laisser `0`. | `0` |
| 3 | **DispelType** | Type de dissipation. Laisser `0`. | `0` |
| 4 | **Mechanic** | Mécanique. Laisser `0`. | `0` |
| 5 | **Attributes** | Attributs du sort. Doit inclure le flag `0x2000` (sauvegarde de l'enchantement). | `0x00002000` |
| 6 | **AttributesEx** | Attributs étendus. Laisser `0`. | `0` |
| 7 | **AttributesEx2** | Attributs étendus 2. Laisser `0`. | `0` |
| 8 | **AttributesEx3** | Attributs étendus 3. Laisser `0`. | `0` |
| 9 | **AttributesEx4** | Attributs étendus 4. Laisser `0`. | `0` |
| 10-17 | **CastingTime** | Temps d'incantation. Laisser `0`. | `0` |
| 18-19 | **TargetType** | Type de cible. Utilisez `16` pour « objet équipé » (permettra de cibler l'équipement à enchanter). | `16` |
| 20-21 | **TargetCreatureType** | Type de créature cible. Laisser `0`. | `0` |
| 39 | **BaseLevel** | Niveau de base. Pour un enchantement, ce champ définit le **niveau d'objet minimum requis** pour que l'équipement puisse recevoir l'enchantement. Par exemple, `60` signifie que l'équipement doit être de niveau 60 ou plus. | `60` |
| 72 | **Effect_1** | Effet du sort. Doit être `53` (SPELL_EFFECT_ENCHANT_ITEM). | `53` |
| 73-74 | **Effect_2, Effect_3** | Effets supplémentaires. Laisser `0`. | `0` |
| 75-77 | **EffectDieSides** | Dés de côté pour chaque effet. Laisser `0`. | `0` |
| 78-80 | **EffectBaseDice** | Dés de base. Laisser `0`. | `0` |
| 81-83 | **EffectBasePoints** | Points de base pour chaque effet. Pour l'effet 53, ce champ peut rester `0`. | `0` |
| 84-86 | **EffectRealPointsPerLevel** | Points réels par niveau. Laisser `0`. | `0` |
| 87-89 | **EffectBasePointsPerLevel** | Points de base par niveau. Laisser `0`. | `0` |
| 90-92 | **EffectMechanic** | Mécanique de l'effet. Laisser `0`. | `0` |
| 93-95 | **EffectImplicitTargetA** | Cible implicite. Laisser `0`. | `0` |
| 96-98 | **EffectImplicitTargetB** | Cible implicite B. Laisser `0`. | `0` |
| 99-101 | **EffectRadiusIndex** | Index de rayon. Laisser `0`. | `0` |
| 102-104 | **EffectApplyAuraName** | Nom de l'aura appliquée. Laisser `0`. | `0` |
| 105-107 | **EffectAmplitude** | Amplitude. Laisser `0`. | `0` |
| 108-110 | **EffectMultipleValue** | Valeur multiple. Laisser `0`. | `0` |
| 111 | **EffectChainTarget** | Cibles en chaîne. Laisser `0`. | `0` |
| 112 | **EffectItemType** | **ID de l'objet parchemin** (référence `Item.dbc` et `item_template`). | `90001` |
| 113 | **EffectMiscValue** | **ID de l'enchantement** (référence `SpellItemEnchantment.dbc`). | `3000` |
| 114 | **EffectMiscValueB** | Valeur diverse B. Laisser `0`. | `0` |
| 115 | **EffectTriggerSpell** | Sort déclenché. Laisser `0`. | `0` |
| 116-118 | **EffectPointsPerComboPoint** | Points par point de combo. Laisser `0`. | `0` |
| 119-121 | **EffectSpellClassMask** | Masque de classe de sort. Laisser `0`. | `0` |
| 122-124 | **EffectSpellClassMask** | Masque de classe de sort 2. Laisser `0`. | `0` |
| 125-127 | **EffectSpellClassMask** | Masque de classe de sort 3. Laisser `0`. | `0` |
| 128 | **SpellIconID** | ID de l'icône. Référence `SpellIcon.dbc`. | `1` |
| 129 | **ActiveIconID** | ID d'icône active. Laisser `0`. | `0` |
| 130 | **SpellPriority** | Priorité du sort. Laisser `0`. | `0` |
| 131-162 | **Name** | Nom du sort (affiché dans le livre de sorts). | `Enchantement : Puissance du Berserker` |
| 163-170 | **NameSubtext** | Sous-titre du nom. Laisser vide. | `` |
| 171-178 | **Description** | Description du sort. | `Enchante une arme avec +10 Force et +5 Agilité.` |
| 179-186 | **AuraDescription** | Description de l'aura. Laisser vide. | `` |
| 187 | **ManaCost** | Coût en mana. Laisser `0`. | `0` |
| 188 | **ManaCostPerLevel** | Coût en mana par niveau. Laisser `0`. | `0` |
| 189 | **ManaPerSecond** | Mana par seconde. Laisser `0`. | `0` |
| 190 | **ManaPerSecondPerLevel** | Mana par seconde par niveau. Laisser `0`. | `0` |
| 191 | **ManaCostPercentage** | Pourcentage de mana. Laisser `0`. | `0` |
| 192 | **ManaCostPercent** | Pourcentage de mana. Laisser `0`. | `0` |
| 193 | **RangeIndex** | Index de portée. Laisser `0`. | `0` |
| 194 | **Speed** | Vitesse. Laisser `0`. | `0` |
| 195 | **ModalNextSpell** | Sort suivant modal. Laisser `0`. | `0` |
| 196 | **StackAmount** | Quantité empilable. Laisser `0`. | `0` |
| 197-198 | **Totem_1, Totem_2** | Totems. Laisser `0`. | `0` |
| 199-202 | **Reagent** | Réactifs. Laisser `0`. | `0` |
| 203-206 | **ReagentCount** | Nombre de réactifs. Laisser `0`. | `0` |
| 207 | **EquippedItemClass** | Classe d'objet équipé. `-1` pour tous types. | `-1` |
| 208 | **EquippedItemSubClass** | Sous-classe d'objet équipé. `-1` pour tous. | `-1` |
| 209 | **EquippedItemInvTypes** | Types d'inventaire d'objet équipé. Peut être utilisé pour restreindre l'enchantement à certains types d'armes/armures. | `0` |
| 210 | **EffectItemType** | (Doublon, voir colonne 112) | `90001` |
| 211 | **EffectMiscValue** | (Doublon, voir colonne 113) | `3000` |
| 212 | **SpellVisual** | Visuel du sort. Laisser `0`. | `0` |
| 213 | **SpellVisual2** | Visuel du sort 2. Laisser `0`. | `0` |
| 214 | **SpellCategory** | Catégorie de sort. Laisser `0`. | `0` |
| 215 | **SpellCategoryCooldown** | Cooldown de catégorie. Laisser `0`. | `0` |
| 216 | **SpellCooldown** | Cooldown du sort. Laisser `0`. | `0` |
| 217 | **SpellCooldownPerLevel** | Cooldown par niveau. Laisser `0`. | `0` |
| 218 | **SpellCooldownTime** | Temps de cooldown. Laisser `0`. | `0` |
| 219 | **SpellCooldownTimePerLevel** | Temps de cooldown par niveau. Laisser `0`. | `0` |
| 220 | **InterruptFlags** | Drapeaux d'interruption. Laisser `0`. | `0` |
| 221 | **AuraInterruptFlags** | Drapeaux d'interruption d'aura. Laisser `0`. | `0` |
| 222 | **ChannelInterruptFlags** | Drapeaux d'interruption de canalisation. Laisser `0`. | `0` |
| 223 | **CasterAuraState** | État d'aura du lanceur. Laisser `0`. | `0` |
| 224 | **TargetAuraState** | État d'aura de la cible. Laisser `0`. | `0` |
| 225 | **CasterAuraStateNot** | État d'aura négatif du lanceur. Laisser `0`. | `0` |
| 226 | **TargetAuraStateNot** | État d'aura négatif de la cible. Laisser `0`. | `0` |
| 227 | **CasterAuraSpell** | Sort d'aura du lanceur. Laisser `0`. | `0` |
| 228 | **TargetAuraSpell** | Sort d'aura de la cible. Laisser `0`. | `0` |
| 229 | **ExcludeCasterAuraSpell** | Exclure le sort d'aura du lanceur. Laisser `0`. | `0` |
| 230 | **ExcludeTargetAuraSpell** | Exclure le sort d'aura de la cible. Laisser `0`. | `0` |
| 231 | **CastingTimeIndex** | Index de temps d'incantation. Laisser `0`. | `0` |
| 232 | **RecoveryTime** | Temps de récupération. Laisser `0`. | `0` |
| 233 | **CategoryRecoveryTime** | Temps de récupération de catégorie. Laisser `0`. | `0` |
| 234 | **StartRecoveryTime** | Temps de récupération de départ. Laisser `0`. | `0` |
| 235 | **StartRecoveryCategory** | Catégorie de récupération de départ. Laisser `0`. | `0` |
| 236 | **MaxTargetLevel** | Niveau de cible maximum. Laisser `0`. | `0` |
| 237 | **SpellFamilyName** | Nom de famille de sort. Laisser `0`. | `0` |
| 238-240 | **SpellFamilyFlags** | Drapeaux de famille de sort. Laisser `0`. | `0` |
| 241 | **MaxAffectedTargets** | Cibles affectées maximum. Laisser `0`. | `0` |
| 242 | **DmgClass** | Classe de dégâts. Laisser `0`. | `0` |
| 243 | **PreventionType** | Type de prévention. Laisser `0`. | `0` |
| 244 | **StanceBarOrder** | Ordre de barre de stance. Laisser `0`. | `0` |
| 245-247 | **DmgMultiplier** | Multiplicateur de dégâts. Laisser `0`. | `0` |
| 248 | **MinFactionID** | ID de faction minimum. Laisser `0`. | `0` |
| 249 | **MinReputation** | Réputation minimum. Laisser `0`. | `0` |
| 250 | **RequiredAuraVision** | Vision d'aura requise. Laisser `0`. | `0` |
| 251-252 | **TotemCategory** | Catégorie de totem. Laisser `0`. | `0` |
| 253 | **AreaGroupID** | ID de groupe de zones. Laisser `0`. | `0` |
| 254 | **SchoolMask** | Masque d'école de magie. Laisser `0`. | `0` |
| 255 | **RuneCostID** | ID de coût runique. Laisser `0`. | `0` |
| 256 | **SpellMissileID** | ID de missile de sort. Laisser `0`. | `0` |
| 257 | **PowerDisplayID** | ID d'affichage de puissance. Laisser `0`. | `0` |
| 258-260 | **EffectBonusMultiplier** | Multiplicateur de bonus d'effet. Laisser `0`. | `0` |
| 261 | **SpellDescriptionVariableID** | ID de variable de description. Laisser `0`. | `0` |
| 262 | **SpellDifficultyID** | ID de difficulté de sort. Laisser `0`. | `0` |

> **📝 Note importante** : Les numéros de colonnes peuvent varier légèrement selon la version exacte du fichier DBC. Vérifiez toujours la structure de votre fichier avec votre éditeur DBC. Les champs critiques sont l'**ID** (colonne 1), l'**Effect 53** (colonne 72), l'**EffectItemType** (colonne 112, ID du parchemin) et l'**EffectMiscValue** (colonne 113, ID de l'enchantement).

### 2.2. Exemple concret

**Sort d'enchantement « Enchantement : Puissance du Berserker »**

- **ID** : `90000`
- **Attributes** : `0x00002000` (flag de sauvegarde d'enchantement)
- **TargetType** : `16` (objet équipé)
- **BaseLevel** : `60` (niveau d'objet minimum requis)
- **Effect_1** : `53` (SPELL_EFFECT_ENCHANT_ITEM)
- **EffectItemType** : `90001` (ID du parchemin)
- **EffectMiscValue** : `3000` (ID de l'enchantement défini à l'étape 1)
- **SpellIconID** : `1`
- **Name** : `Enchantement : Puissance du Berserker`
- **Description** : `Enchante une arme avec +10 Force et +5 Agilité.`

---

## Étape 3 : Créer l'objet parchemin dans Item.dbc

Le fichier `Item.dbc` est la référence côté client pour tous les objets du jeu. Vous devez y ajouter une entrée pour votre parchemin.

### 3.1. Procédure

1. Ouvrez `Item.dbc`.
2. Dupliquez la ligne d'un parchemin d'enchantement existant (par exemple, un « Parchemin d'enchantement » standard).
3. Modifiez le champ **ID** (colonne 1) pour qu'il corresponde à l'ID de votre objet parchemin (ex: `90001`).
4. Modifiez le champ **Class** (colonne 2) : `0` (Consommable).
5. Modifiez le champ **SubClass** (colonne 3) : `6` (Objet d'enchantement).
6. Modifiez le champ **DisplayInfoID** (colonne 6) : L'ID d'affichage de l'icône. Référence `ItemDisplayInfo.dbc`. Utilisez une icône de parchemin existante.
7. Le champ **InventoryType** (colonne 7) doit être `0`.
8. Le champ **SheatheType** (colonne 8) doit être `0`.

> **📝 Note** : Les autres champs (statistiques, niveaux, etc.) dans `Item.dbc` sont principalement utilisés par le client pour l'affichage. Les valeurs réelles sont définies dans la base de données côté serveur (`item_template`).

---

## Étape 4 : Créer l'objet parchemin dans la table `item_template`

C'est ici que l'objet parchemin est réellement défini pour le serveur. Connectez-vous à votre base de données `world` et insérez une nouvelle ligne dans la table `item_template`.

### 4.1. Champs essentiels

| Champ | Valeur | Description |
|-------|--------|-------------|
| **entry** | `90001` | ID unique de l'objet (doit correspondre à l'ID dans `Item.dbc`). |
| **class** | `0` | Classe : Consommable. |
| **subclass** | `6` | Sous-classe : Objet d'enchantement. |
| **name** | `Parchemin : Puissance du Berserker` | Nom de l'objet. |
| **displayid** | `12345` | ID d'affichage (référence `ItemDisplayInfo.dbc`). |
| **Quality** | `3` | Qualité : Rare (bleu). |
| **BuyCount** | `1` | Quantité d'achat. |
| **BuyPrice** | `10000` | Prix d'achat. |
| **SellPrice** | `2500` | Prix de vente. |
| **InventoryType** | `0` | Type d'inventaire : non équipable. |
| **ItemLevel** | `1` | Niveau d'objet. |
| **RequiredLevel** | `60` | Niveau minimum du joueur pour utiliser le parchemin. |
| **RequiredSkill** | `333` | Compétence requise : Enchantement. |
| **RequiredSkillRank** | `300` | Niveau de compétence requis. |
| **spellid_1** | `90000` | ID du sort d'enchantement (étape 2). |
| **spelltrigger_1** | `0` | Déclencheur : `0` = Utilisation. |
| **spellcharges_1** | `-1` | Charges : `-1` = Usage unique (consommé à l'utilisation). |
| **spellcooldown_1** | `-1` | Cooldown : `-1` = Aucun. |
| **spellcategory_1** | `0` | Catégorie de sort : `0`. |
| **spellcategorycooldown_1** | `-1` | Cooldown de catégorie : `-1`. |
| **bonding** | `1` | Liaison : `1` = Lié quand ramassé (Bind on Pickup). |
| **description** | `Enchante une arme avec +10 Force et +5 Agilité.` | Description de l'objet. |
| **flags** | `0` | Drapeaux : `0`. |

### 4.2. Requête SQL d'exemple

```sql
INSERT INTO `item_template` (
    `entry`, `class`, `subclass`, `name`, `displayid`, `Quality`,
    `BuyCount`, `BuyPrice`, `SellPrice`, `InventoryType`, `ItemLevel`,
    `RequiredLevel`, `RequiredSkill`, `RequiredSkillRank`,
    `spellid_1`, `spelltrigger_1`, `spellcharges_1`,
    `spellcooldown_1`, `spellcategory_1`, `spellcategorycooldown_1`,
    `bonding`, `description`, `flags`
) VALUES (
    90001, 0, 6, 'Parchemin : Puissance du Berserker', 12345, 3,
    1, 10000, 2500, 0, 1,
    60, 333, 300,
    90000, 0, -1,
    -1, 0, -1,
    1, 'Enchante une arme avec +10 Force et +5 Agilité.', 0
);
```

> **📝 Note** : La colonne `spellid_1` fait référence à l'ID du sort créé à l'étape 2. Le `spelltrigger_1` à `0` signifie que l'objet est utilisé (clic droit). Le `spellcharges_1` à `-1` signifie que l'objet est consommé à l'utilisation.

---

## Étape 5 : (Optionnel) Configurer `spell_enchant_proc_data`

Si votre enchantement doit avoir un **effet de proc** (par exemple, un enchantement d'arme qui a une chance de déclencher un effet au combat), vous devez configurer la table `spell_enchant_proc_data` dans la base de données `world`.

### 5.1. Structure de la table

| Champ | Type | Description |
|-------|------|-------------|
| **entry** | INT UNSIGNED | ID de l'enchantement (référence `SpellItemEnchantment.dbc`). |
| **customChance** | INT UNSIGNED | Chance personnalisée de proc (en pourcentage). |
| **PPMChance** | FLOAT UNSIGNED | Chance de proc par minute (Procs Per Minute). |
| **procEx** | INT UNSIGNED | Conditions de proc (masque). |
| **attributeMask** | INT UNSIGNED | Masque d'attributs. |

### 5.2. Exemple

```sql
INSERT INTO `spell_enchant_proc_data` (
    `entry`, `customChance`, `PPMChance`, `procEx`, `attributeMask`
) VALUES (
    3000, 0, 1.0, 0, 0
);
```

Dans cet exemple, l'enchantement avec l'ID `3000` a une chance de proc de 1.0 PPM (Procs Per Minute).

> **📝 Note** : Si votre enchantement n'a pas d'effet de proc (par exemple, un simple bonus de statistiques), vous pouvez ignorer cette étape.

---

## Étape 6 : Créer le patch MPQ côté client

Les modifications apportées aux fichiers DBC (`SpellItemEnchantment.dbc`, `Spell.dbc`, `Item.dbc`) doivent être packagées dans un fichier MPQ pour être prises en compte par le client.

### 6.1. Procédure

1. Créez une structure de dossiers similaire à celle de votre client :
   ```
   patch-4/
   └── DBFilesClient/
       ├── SpellItemEnchantment.dbc
       ├── Spell.dbc
       └── Item.dbc
   ```
2. Placez vos fichiers DBC modifiés dans le dossier `DBFilesClient`.
3. Ouvrez votre outil de création MPQ (par exemple, Ladik's MPQ Editor).
4. Créez un nouveau MPQ nommé `patch-4.MPQ` (ou tout autre nom de patch supérieur à vos patchs existants).
5. Ajoutez le dossier `DBFilesClient` avec les fichiers DBC modifiés.
6. Placez le fichier `patch-4.MPQ` dans le dossier `Data` de votre client World of Warcraft.

> **⚠️ Important** : Le nom du patch doit être supérieur au dernier patch existant dans votre dossier `Data`. Par exemple, si vous avez `patch-3.MPQ`, nommez le vôtre `patch-4.MPQ` ou `patch-Z.MPQ`. Si vous avez déjà un `patch-4.MPQ`, utilisez un nom comme `patch-5.MPQ` ou `patch-A.MPQ`.

---

## Étape 7 : Redémarrer le serveur et tester

1. **Redémarrez votre serveur AzerothCore** pour que les modifications de la base de données soient prises en compte.
2. **Reconnectez-vous** au jeu avec le client patché.
3. **Créez ou obtenez** le parchemin d'enchantement. Vous pouvez utiliser la commande `.additem 90001` en jeu (si vous avez les droits admin) ou l'ajouter via un marchand.
4. **Testez l'utilisation** du parchemin :
   - Clic droit sur le parchemin.
   - Le curseur devrait se transformer en cible.
   - Cliquez sur une pièce d'équipement équipée ou dans votre sac.
   - L'enchantement devrait s'appliquer, et le parchemin devrait être consommé.

---

## Dépannage

### Le parchemin n'apparaît pas ou est invisible

- Vérifiez que l'ID dans `Item.dbc` correspond exactement à l'`entry` dans `item_template`.
- Vérifiez que le fichier `Item.dbc` modifié est bien dans le patch MPQ.
- Vérifiez que le `displayid` est valide.

### L'enchantement ne s'applique pas

- Vérifiez que l'ID de l'enchantement dans `SpellItemEnchantment.dbc` correspond au `EffectMiscValue` dans `Spell.dbc`.
- Vérifiez que l'ID du sort dans `Spell.dbc` correspond au `spellid_1` dans `item_template`.
- Vérifiez que l'ID du parchemin dans `Spell.dbc` (`EffectItemType`) correspond à l'`entry` dans `item_template` et à l'ID dans `Item.dbc`.
- Vérifiez que le niveau d'objet minimum (`BaseLevel` dans `Spell.dbc`) n'est pas trop élevé pour l'équipement testé.

### Le parchemin n'est pas consommé

- Vérifiez que `spellcharges_1` est bien `-1` dans `item_template`.
- Vérifiez que `spelltrigger_1` est bien `0`.

### Erreur « Vous ne possédez pas la compétence requise »

- Vérifiez que `RequiredSkill` est bien `333` (Enchantement) dans `item_template`.
- Vérifiez que `RequiredSkillRank` correspond au niveau de compétence réel du personnage.

### Le serveur plante ou des erreurs apparaissent dans les logs

- Vérifiez que les ID utilisés sont uniques et ne sont pas déjà utilisés par d'autres sorts, objets ou enchantements.
- Vérifiez que les fichiers DBC modifiés sont correctement formatés (pas de décalage de colonnes).
- Consultez les logs du serveur (`Server.log` ou `DBErrors.log`) pour plus de détails.

---

## Résumé des IDs utilisés dans l'exemple

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| Enchantement (effet) | `3000` | `SpellItemEnchantment.dbc` |
| Sort d'enchantement | `90000` | `Spell.dbc` |
| Objet parchemin | `90001` | `Item.dbc` + `item_template` |
| Entrée de proc (optionnel) | `3000` | `spell_enchant_proc_data` |

---

## Conclusion

Vous avez maintenant toutes les clés pour créer des recettes d'enchantement personnalisées dans AzerothCore 3.3.5. Le processus implique la modification de fichiers DBC côté client et de tables de base de données côté serveur. N'oubliez pas de toujours sauvegarder vos fichiers avant modification et de tester progressivement.

Pour des enchantements plus complexes (proc au combat, effets multiples, conditions), vous devrez approfondir la configuration de `SpellItemEnchantment.dbc` et de `spell_enchant_proc_data`. N'hésitez pas à consulter la documentation officielle d'AzerothCore et les forums communautaires pour des cas d'usage avancés.

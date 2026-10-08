# Guide complet : Créer une classe de personnage personnalisée dans AzerothCore 3.3.5

## Introduction

Ce guide détaille étape par étape la création d'une **classe de personnage entièrement personnalisée** pour AzerothCore 3.3.5. Contrairement aux guides précédents (enchantements, recettes, réputations, métiers), la création d'une classe est l'opération la plus complexe, car elle touche à la fois aux fichiers DBC côté client, aux tables de base de données côté serveur, et nécessite une compréhension approfondie du système de compétences (SkillLine), de talents et de sorts.

> **⚠️ Avertissement** : La création d'une classe personnalisée est une opération avancée. Elle nécessite des modifications importantes des fichiers DBC et de la base de données. Sauvegardez toujours vos fichiers avant modification. Le support officiel d'AzerothCore ne couvre pas les modifications client.

---

## Prérequis

- **Éditeur DBC** : MyDBCEditor, DBC Editor, ou équivalent.
- **Client de base de données** : HeidiSQL, MySQL Workbench, etc.
- **Fichiers DBC de référence** : `ChrClasses.dbc`, `SkillLine.dbc`, `SkillLineAbility.dbc`, `SkillRaceClassInfo.dbc`, `Spell.dbc`, `Talent.dbc`, `TalentTab.dbc`.
- **Outil MPQ** : Ladik's MPQ Editor.
- **Accès serveur** : Base de données `world` et possibilité de compiler le core si nécessaire.

---

## Vue d'ensemble du processus

La création d'une classe personnalisée implique **sept éléments interconnectés** :

1. **ChrClasses.dbc** (client) — Définit la classe (nom, type de ressource, icône).
2. **SkillLine.dbc** (client) — Définit les compétences de classe (onglets du livre de sorts, arbres de talents).
3. **SkillLineAbility.dbc** (client) — Lie les sorts aux compétences de classe.
4. **SkillRaceClassInfo.dbc** (client) — Définit quelles races peuvent jouer cette classe.
5. **Talent.dbc** et **TalentTab.dbc** (client) — Définissent les arbres de talents (optionnel).
6. **playercreateinfo_*** (serveur) — Configure les paramètres de création (position, sorts, actions, compétences).
7. **Tables de base de données** — Configurent les sorts de classe, les talents et les restrictions.

### Schéma des relations

```
┌─────────────────────────┐
│    ChrClasses.dbc        │  ← Définit la classe
│  ID: 12                 │
│  Name: "Nécromancien"   │
│  PowerType: 0 (Mana)    │
└───────────┬─────────────┘
            │  référencé par
            ▼
┌─────────────────────────┐
│ SkillRaceClassInfo.dbc   │  ← Quelles races peuvent jouer
│  SkillLine: 2000        │
│  ChrRaces: 0 (toutes)   │
│  ChrClasses: 2048       │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│     SkillLine.dbc        │  ← Compétences de classe
│  ID: 2000               │
│  Name: "Nécromancie"    │
└───────────┬─────────────┘
            │  lié à
            ▼
┌─────────────────────────┐
│  SkillLineAbility.dbc    │  ← Lie sorts ↔ compétence
│  SkillLine: 2000        │
│  Spell: 96000           │
│  ClassMask: 2048        │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│       Spell.dbc          │  ← Sorts de la classe
│  ID: 96000              │
│  Name: "Éclair noir"    │
└─────────────────────────┘
```

---

## Étape 1 : Définir la classe dans ChrClasses.dbc

Le fichier `ChrClasses.dbc` contient toutes les classes jouables du jeu. C'est ici que vous définissez le nom, le type de ressource et l'icône de votre classe.

### 1.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique de la classe. |
| 2 | **Unknown** | Integer | `1` pour Chasseur, Voleur, Chaman ; `9` pour Chevalier de la mort ; `0` pour les autres. |
| 3 | **PowerType** | Integer | `0` = Mana, `1` = Rage, `2` = Focus, `3` = Énergie, `6` = Runes. |
| 4 | **m_petNameToken** | String | Type de familier. `101` pour les démons du Démoniste, `1` pour les autres. |
| 5-20 | **Name** | Loc | Nom de la classe. |
| 21 | **NameLangMask** | Integer | Drapeaux de langue (inutilisé). |
| 22-37 | **Name_female** | Loc | Nom féminin (si différent). |
| 38 | **NameFemaleLangMask** | Integer | Drapeaux de langue (inutilisé). |
| 39-54 | **Name_male** | Loc | Nom masculin (si différent). |
| 55 | **NameMaleLangMask** | Integer | Drapeaux de langue (inutilisé). |
| 56 | **fileName** | String | Nom anglais capitalisé (ex: « NECROMANCER »). |
| 57 | **spellClassSet** | Integer | Famille de sorts (référence `SpellClassSet.dbc`). |
| 58 | **Flags** | Integer | Drapeaux (voir ci-dessous). |
| 59 | **Camera** | iRefID | Caméra de la cinématique d'ouverture. |
| 60 | **required_expansion** | Integer | `0` = Classic, `1` = BC, `3` = Wrath. |

### 1.2. Drapeaux (Flags)

| Drapeau | Valeur | Description |
|---------|--------|-------------|
| Use loincloth | `1` | Utilise un pagne. |
| Player class | `2` | Classe jouable. |
| Display pet | `4` | Affiche le familier. |
| Can wear mail | `16` | Peut porter la maille. |
| Can wear scaling-stat plate | `32` | Peut porter le plate à statistiques évolutives. |
| Bind starting area | `64` | Lie la zone de départ. |

### 1.3. Exemple concret

**Classe « Nécromancien »**

- **ID** : `12` (doit être unique et non utilisé ; les IDs existants sont 1 à 11)
- **PowerType** : `0` (Mana)
- **m_petNameToken** : `1` (familier standard)
- **Name** : `Nécromancien`
- **fileName** : `NECROMANCER`
- **spellClassSet** : `16` (ou une nouvelle famille)
- **Flags** : `2` (Player class)
- **required_expansion** : `3` (Wrath)

> **📝 Note** : Le champ **ID** est utilisé dans de nombreux bitmasks. La formule est `Value = 1 << (ID - 1)`. Pour l'ID `12`, la valeur du bitmask est `1 << 11 = 2048`.

---

## Étape 2 : Définir les compétences de classe dans SkillLine.dbc

Chaque classe possède des compétences (SkillLine) qui apparaissent comme des onglets dans le livre de sorts. Vous devez créer au moins une compétence pour votre classe.

### 2.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique de la compétence. |
| 2 | **CategoryId** | iRefID | Référence à `SkillLineCategory.dbc`. Utilisez `7` (Class skills). |
| 3 | **SkillCostId** | Integer | Référence à `SkillCostsData.dbc`. Utilisez `0`. |
| 4 | **Name** | Loc | Nom de la compétence (ex: « Nécromancie »). |
| 5 | **Description** | Loc | Description de la compétence. |
| 6 | **SpellIcon** | iRefID | Référence à `SpellIcon.dbc`. |
| 7 | **Verb** | Loc | Verbe utilisé dans l'interface. |
| 8 | **CanLink** | Integer | `1` si la compétence a des sorts liés. |

### 2.2. Exemple concret

**Compétence « Nécromancie »**

- **ID** : `2000` (doit être unique)
- **CategoryId** : `7` (Class skills)
- **Name** : `Nécromancie`
- **Description** : `La maîtrise des arts sombres de la mort.`
- **SpellIcon** : `1`
- **CanLink** : `1`

---

## Étape 3 : Lier les sorts à la compétence dans SkillLineAbility.dbc

Ce fichier est **essentiel** : il lie chaque sort de votre classe à la compétence créée à l'étape 2.

### 3.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique de l'entrée. |
| 2 | **SkillLine** | iRefID | Référence à l'ID dans `SkillLine.dbc`. |
| 3 | **Spell** | iRefID | Référence à l'ID du sort dans `Spell.dbc`. |
| 4 | **RaceMask** | BitMask | Races autorisées (`0` = toutes). |
| 5 | **ClassMask** | BitMask | **Classes autorisées** (bitmask de votre classe). |
| 6 | **ExcludeRace** | BitMask | Races exclues. |
| 7 | **ExcludeClass** | BitMask | Classes exclues. |
| 8 | **MinSkillLineRank** | Integer | Rang minimum de compétence. |
| 9 | **SupercededBySpell** | iRefID | Sort qui remplace (pour les rangs). |
| 10 | **AcquireMethod** | Integer | Méthode d'acquisition. |
| 11 | **TrivialSkillLineRankHigh** | Integer | Rang où la recette devient grise. |
| 12 | **TrivialSkillLineRankLow** | Integer | Rang où la recette devient jaune. |

### 3.2. Exemple concret

**Liaison du sort « Éclair noir » (ID 96000) à la compétence « Nécromancie » (ID 2000) pour la classe « Nécromancien » (bitmask 2048)**

| ID | SkillLine | Spell | RaceMask | ClassMask | MinSkillLineRank |
|----|-----------|-------|----------|-----------|------------------|
| 2000 | 2000 | 96000 | 0 | 2048 | 1 |

> **📝 Note** : Le champ **ClassMask** doit correspondre au bitmask de votre classe (`2048` pour l'ID 12). Le champ **SkillLine** doit correspondre à l'ID de la compétence créée à l'étape 2.

---

## Étape 4 : Définir les races autorisées dans SkillRaceClassInfo.dbc

Ce fichier détermine **quelles races** peuvent jouer votre classe.

### 4.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique de l'entrée. |
| 2 | **SkillLine** | iRefID | Référence à l'ID dans `SkillLine.dbc`. |
| 3 | **ChrRaces** | BitMask | Races autorisées (`0` = toutes). |
| 4 | **ChrClasses** | BitMask | Classes autorisées (`0` = toutes). |
| 5 | **Flags** | Integer | Drapeaux. |
| 6 | **RequLvl** | Integer | Niveau minimum. |
| 7 | **SkillTierId** | iRefID | Référence à `SkillTiers.dbc`. |

### 4.2. Exemple concret

**Classe « Nécromancien » accessible à toutes les races**

- **ID** : `2000`
- **SkillLine** : `2000`
- **ChrRaces** : `0` (toutes les races)
- **ChrClasses** : `2048` (Nécromancien)
- **Flags** : `0`
- **RequLvl** : `0`

---

## Étape 5 : Créer les sorts de classe dans Spell.dbc

Vous devez créer les sorts de votre classe (sorts de dégâts, soins, utilitaires, etc.). Chaque sort doit être lié à la compétence de classe via `SkillLineAbility.dbc`.

### 5.1. Champs essentiels pour un sort de classe

| Colonne | Champ | Description |
|---------|-------|-------------|
| 1 | **ID** | ID unique du sort. |
| 5 | **Attributes** | Attributs du sort (ex: `0x00010000` pour une capacité). |
| 72 | **Effect_1** | Effet du sort (ex: `2` pour dégâts, `6` pour appliquer une aura). |
| 75 | **EffectBasePoints** | Points de base de l'effet. |
| 108 | **EffectMiscValue** | Valeur diverse (ex: école de magie). |
| 128 | **SpellIconID** | Icône du sort. |
| 131-162 | **Name** | Nom du sort. |
| 171-178 | **Description** | Description du sort. |
| 237 | **SpellFamilyName** | Famille de sorts (référence `SpellClassSet.dbc`). |

### 5.2. Exemple concret

**Sort « Éclair noir » (dégâts de l'ombre)**

- **ID** : `96000`
- **Effect_1** : `2` (SPELL_EFFECT_SCHOOL_DAMAGE)
- **EffectBasePoints** : `100` (dégâts de base)
- **SchoolMask** : `32` (Ombre)
- **Name** : `Éclair noir`
- **Description** : `Inflige 100 points de dégâts d'ombre.`

---

## Étape 6 : (Optionnel) Créer les arbres de talents

Si votre classe doit avoir des arbres de talents, vous devez configurer `TalentTab.dbc` et `Talent.dbc`. Cette étape est complexe et optionnelle.

### 6.1. TalentTab.dbc

Définit les onglets d'arbres de talents.

| Colonne | Champ | Description |
|---------|-------|-------------|
| 1 | **ID** | ID unique de l'onglet. |
| 2 | **Name** | Nom de l'onglet (ex: « Sombre »). |
| 3 | **SpellIcon** | Icône de l'onglet. |
| 4 | **RaceMask** | Races autorisées. |
| 5 | **ClassMask** | Classes autorisées (votre classe). |
| 6 | **OrderIndex** | Ordre de l'onglet. |
| 7 | **BackgroundFile** | Fichier d'arrière-plan. |

### 6.2. Talent.dbc

Définit chaque talent individuel.

---

## Étape 7 : Configurer la base de données (playercreateinfo_*)

Plusieurs tables de base de données doivent être configurées pour que votre classe soit fonctionnelle.

### 7.1. playercreateinfo

Définit la position de départ pour chaque combinaison race-classe.

```sql
INSERT INTO `playercreateinfo` (
    `race`, `class`, `map`, `zone`, `position_x`, `position_y`, `position_z`, `orientation`
) VALUES (
    1, 12, 0, 12, -8949.95, -132.493, 83.5312, 0
);
```

| Champ | Description |
|-------|-------------|
| `race` | ID de la race (référence `ChrRaces.dbc`). |
| `class` | ID de la classe (référence `ChrClasses.dbc`). |
| `map` | ID de la carte. |
| `zone` | ID de la zone. |
| `position_x/y/z` | Coordonnées de départ. |
| `orientation` | Orientation. |

### 7.2. playercreateinfo_action

Définit les actions par défaut sur la barre d'action.

```sql
INSERT INTO `playercreateinfo_action` (
    `race`, `class`, `button`, `action`, `type`
) VALUES (
    1, 12, 0, 6603, 0
);
```

### 7.3. playercreateinfo_spell_custom

Définit les sorts de départ si `PlayerStart.CustomSpells` est activé.

```sql
INSERT INTO `playercreateinfo_spell_custom` (
    `racemask`, `classmask`, `Spell`, `Note`
) VALUES (
    0, 2048, 96000, 'Éclair noir'
);
```

> **📝 Note** : Vous devez activer `PlayerStart.CustomSpells = 1` dans `worldserver.conf` pour que cette table soit prise en compte.

### 7.4. playercreateinfo_skills

Définit les compétences de départ.

```sql
INSERT INTO `playercreateinfo_skills` (
    `raceMask`, `classMask`, `skill`, `rank`, `comment`
) VALUES (
    0, 2048, 2000, 1, 'Nécromancie'
);
```

---

## Étape 8 : Créer le patch MPQ côté client

1. Créez la structure :
   ```
   patch-4/
   └── DBFilesClient/
       ├── ChrClasses.dbc
       ├── SkillLine.dbc
       ├── SkillLineAbility.dbc
       ├── SkillRaceClassInfo.dbc
       ├── Spell.dbc
       ├── Talent.dbc
       └── TalentTab.dbc
   ```
2. Placez les fichiers modifiés dans `DBFilesClient`.
3. Créez `patch-4.MPQ` avec Ladik's MPQ Editor.
4. Placez-le dans le dossier `Data` du client.

---

## Étape 9 : Redémarrer et tester

1. **Redémarrez votre serveur** pour que les modifications de la base de données soient prises en compte.
2. **Reconnectez-vous** au jeu avec le client patché.
3. **Créez un nouveau personnage** et sélectionnez votre classe personnalisée.
4. **Vérifiez** que les sorts, compétences et talents apparaissent correctement.

---

## Dépannage

| Problème | Cause possible | Solution |
|----------|----------------|----------|
| La classe n'apparaît pas à la création | `ChrClasses.dbc` non patché ou ID invalide | Vérifiez le MPQ et l'ID. |
| Les sorts n'apparaissent pas | `SkillLineAbility.dbc` mal configuré | Vérifiez `SkillLine` et `ClassMask`. |
| La classe n'a pas de compétences | `SkillLine.dbc` ou `SkillRaceClassInfo.dbc` manquant | Vérifiez les entrées. |
| Le serveur plante | Conflit d'ID avec une classe existante | Utilisez des IDs uniques (12+). |
| Les talents ne s'affichent pas | `TalentTab.dbc` mal configuré | Vérifiez `ClassMask` et `OrderIndex`. |

---

## Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| Classe | `12` | `ChrClasses.dbc` |
| Compétence | `2000` | `SkillLine.dbc` |
| Sort | `96000` | `Spell.dbc` |
| Bitmask de classe | `2048` | `1 << (12 - 1)` |

---

## Conclusion

Vous savez maintenant créer une classe de personnage entièrement personnalisée dans AzerothCore 3.3.5. Le processus implique la modification de plusieurs fichiers DBC (`ChrClasses.dbc`, `SkillLine.dbc`, `SkillLineAbility.dbc`, `SkillRaceClassInfo.dbc`, `Spell.dbc`) et de tables de base de données (`playercreateinfo`, `playercreateinfo_action`, `playercreateinfo_spell_custom`, `playercreateinfo_skills`). Les arbres de talents sont optionnels mais ajoutent une profondeur de jeu considérable. Testez toujours progressivement et consultez les logs serveur en cas d'erreur.

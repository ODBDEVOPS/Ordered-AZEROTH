# Guide complet : Créer une réputation (Faction) personnalisée dans AzerothCore 3.3.5

## Introduction

Ce guide détaille étape par étape la création d'une **faction/réputation personnalisée** pour AzerothCore 3.3.5. Contrairement aux enchantements et aux recettes de métier, la création d'une faction implique la modification de fichiers DBC côté client et de tables de base de données côté serveur, ainsi que la configuration des mécanismes de gain de réputation (quêtes, créatures tuées, sorts).

> **⚠️ Avertissement** : Toute modification des fichiers DBC nécessite la création d'un patch MPQ côté client. Les modifications de base de données sont appliquées côté serveur et ne nécessitent pas de patch client. Sauvegardez toujours vos fichiers avant modification.

---

## Prérequis

- **Éditeur DBC** : MyDBCEditor, DBC Editor, ou équivalent.
- **Client de base de données** : HeidiSQL, MySQL Workbench, etc.
- **Fichiers DBC de référence** : `Faction.dbc`, `FactionTemplate.dbc`.
- **Outil MPQ** : Ladik's MPQ Editor.
- **Accès serveur** : Base de données `world` et redémarrage possible.

---

## Vue d'ensemble du processus

La création d'une faction personnalisée implique **cinq éléments interconnectés** :

1. **Faction.dbc** — Définit la faction elle-même (nom, drapeaux, réputation de base).
2. **FactionTemplate.dbc** — Définit les relations d'hostilité/amitié avec les autres factions.
3. **Tables de base de données** — Configurent les sources de gain de réputation (quêtes, créatures, sorts).
4. **PNJ/Quêtes** — Fournissent des moyens d'interagir avec la faction.
5. **Récompenses** — Objets, sorts ou avantages débloqués par la réputation.

### Schéma des relations

```
┌─────────────────────────┐
│      Faction.dbc        │  ← Définit la faction (nom, drapeaux)
│  ID: 90000              │
│  reputationIndex: 200   │  ← Index unique de réputation
└───────────┬─────────────┘
            │  référencé par
            ▼
┌─────────────────────────┐
│   FactionTemplate.dbc   │  ← Définit les relations (hostile/amical)
│  ID: 90000              │
│  ourMask, friendlyMask  │
└───────────┬─────────────┘
            │  utilisé par
            ▼
┌─────────────────────────┐
│  creature_template      │  ← Les PNJ de la faction
│  faction: 90000         │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ creature_onkill_reputation│  ← Gain de réputation en tuant
│  RewOnKillRepFaction1   │
└─────────────────────────┘
```

---

## Étape 1 : Définir la faction dans Faction.dbc

Le fichier `Faction.dbc` contient toutes les factions de base du jeu. C'est ici que vous définissez **le nom** de votre faction et ses propriétés fondamentales.

### 1.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique de la faction. |
| 2 | **reputationIndex** | Integer | Index unique de réputation (max 127). Les factions sans gain de réputation ont `-1`. |
| 3-6 | **reputationRaceMask** | BitMask[4] | Masques de race pour la réputation de base. |
| 7-10 | **reputationClassMask** | BitMask[4] | Masques de classe pour la réputation de base. |
| 11-14 | **reputationBase** | Integer[4] | Valeur de réputation de base (0 = Neutre). |
| 15-18 | **reputationFlags** | Integer[4] | Drapeaux de réputation (voir ci-dessous). |
| 19 | **parentFactionID** | iRefID | Faction parente (récursif). |
| 20-21 | **parentFactionMod** | Float[2] | Modificateur de la faction parente. |
| 22-23 | **parentFactionCap** | Integer[2] | Plafond de la faction parente. |
| 24-40 | **Name** | Loc | Nom affiché de la faction. |
| 41-57 | **Description** | Loc | Description visible dans l'interface de réputation. |

### 1.2. Drapeaux de réputation

| Drapeau | Valeur | Description |
|---------|--------|-------------|
| `FACTION_FLAG_NONE` | `0x00` | Aucun drapeau. |
| `FACTION_FLAG_VISIBLE` | `0x01` | Visible dans l'interface de réputation. |
| `FACTION_FLAG_AT_WAR` | `0x02` | Active le bouton « En guerre ». |
| `FACTION_FLAG_HIDDEN` | `0x04` | Cachée (le joueur peut gagner de la réputation mais ne la voit pas). |
| `FACTION_FLAG_INVISIBLE_FORCED` | `0x08` | Toujours invisible. |
| `FACTION_FLAG_PEACE_FORCED` | `0x10` | Toujours en paix. |
| `FACTION_FLAG_INACTIVE` | `0x20` | Inactive (contrôlée par le joueur). |
| `FACTION_FLAG_RIVAL` | `0x40` | Faction rivale (Outreterre). |
| `FACTION_FLAG_SPECIAL` | `0x80` | Spéciale (villes principales). |



### 1.3. Exemple concret

**Faction « Gardiens d'Azjol-Nerub »**

- **ID** : `90000`
- **reputationIndex** : `200` (doit être unique et ≤ 127)
- **reputationBase** : `0` (Neutre)
- **reputationFlags** : `0x01` (Visible) + `0x02` (At War) = `0x03`
- **Name** : `Gardiens d'Azjol-Nerub`
- **Description** : `Une faction mystérieuse veillant sur les profondeurs du royaume nerubien.`

---

## Étape 2 : Définir les relations dans FactionTemplate.dbc

Le fichier `FactionTemplate.dbc` définit les relations d'hostilité et d'amitié entre les factions. C'est ce fichier qui détermine si les PNJ de votre faction attaqueront les joueurs ou seront amicaux.

### 2.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Int | Identifiant unique du template. |
| 2 | **Name (Ref to Faction.dbc)** | Int | Référence à l'ID dans `Faction.dbc`. |
| 3 | **ourMask** | Bitmask (4 bits) | Définit le type de faction. |
| 4 | **friendlyMask** | Bitmask (4 bits) | Groupes avec lesquels cette faction est amicale. |
| 5 | **hostileMask** | Bitmask (4 bits) | Groupes avec lesquels cette faction est hostile. |
| 6-9 | **enemyFactions** | Int | Liste des factions ennemies spécifiques. |
| 10-13 | **friendFactions** | Int | Liste des factions amies spécifiques. |



### 2.2. Masques de faction

| ID | Bit | Nom |
|----|-----|-----|
| 0 | 1 | Tous les joueurs (et familiers) |
| 1 | 2 | Joueurs de l'Alliance (et familiers) |
| 2 | 4 | Joueurs de la Horde (et familiers) |
| 3 | 8 | Monstres (ni joueur ni familier) |



### 2.3. Exemple concret

**Faction « Gardiens d'Azjol-Nerub » (neutre, amicale avec personne, hostile avec les monstres)**

- **ID** : `90000`
- **Name** : `90000` (référence à `Faction.dbc`)
- **ourMask** : `8` (Monstre)
- **friendlyMask** : `0` (Aucun groupe ami)
- **hostileMask** : `0` (Aucun groupe hostile automatique)

> **📝 Note** : Si votre faction doit être amicale avec les joueurs, définissez `friendlyMask` avec les bits appropriés. Par exemple, `friendlyMask = 3` (bits 1 et 2) signifie amical avec tous les joueurs.

---

## Étape 3 : Créer la faction en base de données (optionnel)

AzerothCore ne stocke pas les factions dans une table dédiée, mais vous pouvez avoir besoin de configurer des tables associées pour le gain de réputation.

### 3.1. Table `creature_onkill_reputation`

Cette table contrôle la réputation gagnée en tuant des créatures.

```sql
INSERT INTO `creature_onkill_reputation` (
    `creature_id`, `RewOnKillRepFaction1`, `MaxStanding1`,
    `IsTeamAward1`, `RewOnKillRepValue1`, `TeamDependent`
) VALUES (
    90001, 90000, 7, 0, 25, 0
);
```

| Champ | Valeur | Description |
|-------|--------|-------------|
| `creature_id` | `90001` | ID de la créature (référence `creature_template.entry`). |
| `RewOnKillRepFaction1` | `90000` | ID de la faction (référence `Faction.dbc`). |
| `MaxStanding1` | `7` | Rang maximum (0=Hated, 7=Exalted). |
| `IsTeamAward1` | `0` | `1` = réputation aussi pour l'équipe. |
| `RewOnKillRepValue1` | `25` | Points de réputation gagnés. |
| `TeamDependent` | `0` | `1` = réputation différente selon l'équipe. |



### 3.2. Table `reputation_reward_rate` (optionnel)

Permet de définir des multiplicateurs de réputation pour votre faction.

```sql
INSERT INTO `reputation_reward_rate` (
    `faction`, `quest_rate`, `creature_rate`, `spell_rate`
) VALUES (
    90000, 1.5, 2.0, 1.0
);
```

| Champ | Description |
|-------|-------------|
| `faction` | ID de la faction. |
| `quest_rate` | Multiplicateur pour les quêtes. |
| `creature_rate` | Multiplicateur pour les créatures tuées. |
| `spell_rate` | Multiplicateur pour les sorts. |



### 3.3. Table `reputation_spillover_template` (optionnel)

Permet de donner de la réputation à des factions alliées lorsqu'un joueur gagne de la réputation avec votre faction.

```sql
INSERT INTO `reputation_spillover_template` (
    `faction`, `faction1`, `rate_1`, `rank_1`
) VALUES (
    90000, 67, 0.5, 5
);
```

| Champ | Description |
|-------|-------------|
| `faction` | Faction où la réputation est gagnée. |
| `faction1` | Faction recevant le spillover. |
| `rate_1` | Multiplicateur (0.5 = 50% de la réputation). |
| `rank_1` | Rang maximum pour le spillover. |



---

## Étape 4 : Configurer les sources de gain de réputation

### 4.1. Via des quêtes

Ajoutez une récompense de réputation à une quête existante ou nouvelle dans `quest_template`.

```sql
UPDATE `quest_template`
SET `RewardFactionID1` = 90000,
    `RewardFactionValue1` = 250,
    `RewardFactionOverride1` = 0
WHERE `ID` = 12345;
```

| Champ | Description |
|-------|-------------|
| `RewardFactionID1` | ID de la faction (référence `Faction.dbc`). |
| `RewardFactionValue1` | Valeur de réputation (en points). |
| `RewardFactionOverride1` | Remplace la valeur standard (0 = utiliser la valeur standard). |

### 4.2. Via des sorts

Créez un sort qui donne de la réputation. Utilisez l'effet `SPELL_EFFECT_REPUTATION` (ID `103`) ou `SPELL_EFFECT_QUEST_COMPLETE` (ID `16`).

### 4.3. Via des objets

Créez un objet consommable qui donne de la réputation.

```sql
INSERT INTO `item_template` (
    `entry`, `class`, `subclass`, `name`, `displayid`, `Quality`,
    `spellid_1`, `spelltrigger_1`, `spellcharges_1`, `bonding`
) VALUES (
    90002, 0, 0, 'Insigne des Gardiens', 12345, 3,
    95000, 0, -1, 1
);
```

---

## Étape 5 : Créer le patch MPQ côté client

1. Créez la structure :
   ```
   patch-4/
   └── DBFilesClient/
       ├── Faction.dbc
       └── FactionTemplate.dbc
   ```
2. Placez les fichiers modifiés dans `DBFilesClient`.
3. Créez `patch-4.MPQ` avec Ladik's MPQ Editor.
4. Placez-le dans le dossier `Data` du client.

---

## Étape 6 : Redémarrer et tester

1. Redémarrez le serveur.
2. Reconnectez-vous avec le client patché.
3. Utilisez les commandes GM pour tester :
   - `.modify reputation 90000 1000` — Ajoute 1000 points de réputation.
   - `.lookup faction Gardiens` — Recherche l'ID de la faction.
   - `.pinfo rep` — Liste toutes les réputations connues.



---

## Dépannage

| Problème | Cause possible | Solution |
|----------|----------------|----------|
| La faction n'apparaît pas | `Faction.dbc` non patché ou `reputationIndex` invalide | Vérifiez le MPQ et l'index (≤ 127). |
| Les PNJ sont hostiles | `FactionTemplate.dbc` mal configuré | Vérifiez `friendlyMask` et `hostileMask`. |
| Aucun gain de réputation | Tables `creature_onkill_reputation` ou `quest_template` non configurées | Vérifiez les entrées. |
| Erreur « faction not found » | ID de faction invalide dans `Faction.dbc` | Vérifiez l'ID. |
| Le serveur plante | Conflit d'ID avec une faction existante | Utilisez des IDs uniques (90000+). |

---

## Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| Faction | `90000` | `Faction.dbc` |
| FactionTemplate | `90000` | `FactionTemplate.dbc` |
| Créature | `90001` | `creature_template` |
| Quête | `12345` | `quest_template` |
| Objet | `90002` | `item_template` |

---

## Conclusion

Vous savez maintenant créer une faction/réputation personnalisée complète dans AzerothCore 3.3.5. Le processus implique la modification de fichiers DBC (`Faction.dbc`, `FactionTemplate.dbc`) et de tables de base de données (`creature_onkill_reputation`, `reputation_reward_rate`, etc.). Les systèmes de spillover et de multiplicateurs vous permettent d'affiner le comportement de votre faction. Testez toujours progressivement et consultez les logs serveur en cas d'erreur.

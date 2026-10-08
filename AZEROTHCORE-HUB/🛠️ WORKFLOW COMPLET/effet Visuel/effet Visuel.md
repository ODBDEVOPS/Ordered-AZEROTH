# Guide complet : Créer un effet visuel personnalisé dans AzerothCore 3.3.5

## Introduction

Ce guide détaille étape par étape la création d'un **effet visuel personnalisé** (Spell Visual Effect) pour AzerothCore 3.3.5. Contrairement aux guides précédents (gemmes, enchantements, mascottes), la création d'un effet visuel est une opération **multi-niveaux** qui implique une **chaîne de quatre fichiers DBC** interconnectés côté client. Le serveur, quant à lui, ne stocke aucune donnée d'effet visuel : il ne fait que **référencer** le sort dont l'ID de visuel est défini dans `Spell.dbc`.

La difficulté principale réside dans le fait que les effets visuels dans WoW 3.3.5 sont gérés par une **hiérarchie de DBC** où chaque fichier référence le suivant. Une erreur dans un seul maillon de la chaîne rend l'effet invisible ou provoque un crash du client.

> **⚠️ Avertissement** : Toute modification des fichiers DBC nécessite la création d'un patch MPQ côté client. Sauvegardez toujours vos fichiers avant modification. Un mauvais ID dans la chaîne de DBC peut provoquer un crash client au chargement.

---

## Prérequis

- **Éditeur DBC** : MyDBCEditor, DBC Editor, ou équivalent.
- **Client WoW 3.3.5a** avec le dossier `Data`.
- **Fichiers DBC de référence** : `SpellVisualEffectName.dbc`, `SpellVisualKitModelAttach.dbc`, `SpellVisualKit.dbc`, `SpellVisual.dbc`, `Spell.dbc`.
- **Outil MPQ** : Ladik's MPQ Editor.
- **Modèle 3D** : Un fichier `.m2` (ou `.mdx`) à utiliser comme effet, **ou** l'ID d'un modèle existant à réutiliser.
- **Optionnel** : MPQ Editor pour extraire les DBC de votre client.

---

## Vue d'ensemble de la chaîne des DBC

La création d'un effet visuel suit une **chaîne de dépendances** précise. L'ordre de création recommandé est le suivant (du plus fondamental au plus global) :

1. **SpellVisualEffectName.dbc** — Définit le **modèle 3D** utilisé (chemin `.m2`).
2. **SpellVisualKitModelAttach.dbc** — Définit **où** et **comment** le modèle est attaché (point d'ancrage, position, rotation).
3. **SpellVisualKit.dbc** — Définit **quels effets** sont actifs pendant une phase du sort (incantation, buff, etc.).
4. **SpellVisual.dbc** — Définit **quel kit** est utilisé pour chaque phase du sort.
5. **Spell.dbc** — Référence l'ID de `SpellVisual.dbc` dans la colonne `SpellVisualID`.

### Schéma de la chaîne

```
┌──────────────────────────────┐
│  SpellVisualEffectName.dbc   │  ← Définit le modèle (chemin .m2)
│  ID: 5000                    │
│  FileName: Spells\MonEffet.m2│
└───────────┬──────────────────┘
            │  référencé par
            ▼
┌──────────────────────────────┐
│ SpellVisualKitModelAttach.dbc│  ← Attache le modèle à un point
│  ID: 6000                    │
│  SpellVisualEffectNameID: 5000│
│  ParentSpellVisualKitID: 7000│
│  AttachID: 0 (base)          │
└───────────┬──────────────────┘
            │  référencé par
            ▼
┌──────────────────────────────┐
│     SpellVisualKit.dbc        │  ← Définit les effets d'une phase
│  ID: 7000                    │
│  m_headEffect: 5000          │
│  m_chestEffect: 5000         │
│  m_baseEffect: 5000          │
└───────────┬──────────────────┘
            │  référencé par
            ▼
┌──────────────────────────────┐
│      SpellVisual.dbc          │  ← Définit les kits par phase
│  ID: 8000                    │
│  PrecastKit: 7000            │
│  CastKit: 7000               │
│  BuffKit: 7000               │
└───────────┬──────────────────┘
            │  référencé par
            ▼
┌──────────────────────────────┐
│        Spell.dbc              │  ← Sort qui utilise le visuel
│  ID: 97000                   │
│  SpellVisualID: 8000         │
└──────────────────────────────┘
```

> **📝 Note** : Cette chaîne est confirmée par la communauté de modding : « 视觉效果，我们通常用到4个DBC： SpellVisualEffectName.dbc, SpellVisualKitModelAttach.dbc, SpellVisualKit.dbc, SpellVisual.dbc. 请注意我的书写顺序，因为一般情况下，我们做一个视觉效果都是用按这个顺序来做DBC 的 ».

---

## Étape 1 : Définir le modèle dans SpellVisualEffectName.dbc

Ce fichier est la **base** de la chaîne. Il définit le chemin vers le fichier modèle 3D (`.m2` ou `.mdx`) qui sera affiché.

### 1.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique de l'effet. |
| 2 | **Name** | String | Nom/commentaire (non affiché en jeu). |
| 3 | **FileName** | String | **Chemin du modèle** (ex: `Spells\MonEffet.m2`). |
| 4 | **AreaEffectSize** | Float | Taille de l'effet (par défaut `1`). |
| 5 | **Scale** | Float | Échelle du modèle (par défaut `1`). |
| 6 | **MinScale** | Float | Échelle minimale (par défaut `0.1`). |
| 7 | **MaxScale** | Float | Échelle maximale (par défaut `100`). |

> **📝 Note** : Le chemin du fichier utilise des **antislashs** (`\`) et ne doit **pas** inclure l'extension `.m2` dans certains DBC. Vérifiez le format exact dans votre version du fichier. Comme l'explique un tutoriel chinois : « 第三列是模型路径，比如 spells\shaman_ascendance_glyph_air_base.mdx。（说句题外话，WOW里面的模型格式，MDX=M2） ».

### 1.2. Exemple concret

**Effet « Aura de flammes »**

- **ID** : `5000`
- **Name** : `Aura_Flammes`
- **FileName** : `Spells\Fire_Aura.m2`
- **AreaEffectSize** : `1`
- **Scale** : `1`
- **MinScale** : `0.1`
- **MaxScale** : `100`

### 1.3. Où trouver des modèles

Les modèles existants se trouvent dans les fichiers MPQ du client, dans les dossiers `Spells\`, `Particles\`, etc. Vous pouvez les extraire avec **MPQ Editor** ou **Ladik's MPQ Editor**. Pour un modèle personnalisé, vous devez créer votre propre fichier `.m2` (avec un logiciel comme Blender + un plugin M2) et le placer dans votre patch MPQ.

---

## Étape 2 : Attacher le modèle dans SpellVisualKitModelAttach.dbc

Ce fichier définit **comment** et **où** le modèle est attaché au personnage ou à la cible. Il lie un `SpellVisualEffectName` à un `SpellVisualKit` parent.

### 2.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique de l'attachement. |
| 2 | **ParentSpellVisualKitID** | iRefID | **ID du SpellVisualKit parent** (référence `SpellVisualKit.dbc`). |
| 3 | **SpellVisualEffectNameID** | iRefID | **ID du modèle** (référence `SpellVisualEffectName.dbc`). |
| 4 | **AttachmentID** | Integer | Point d'ancrage sur le modèle (`0` = base, `1` = main droite, `2` = main gauche, etc.). |
| 5-7 | **OffsetX/Y/Z** | Float | Décalage de position par rapport au point d'ancrage. |
| 8-10 | **RotationX/Y/Z** | Float | Rotation du modèle autour des axes. |

> **📝 Note** : Le champ **ParentSpellVisualKitID** doit référencer un ID qui **existe déjà** ou que vous allez créer à l'étape 3. L'ordre de création est donc : créer d'abord `SpellVisualKitModelAttach` avec un ID parent temporaire, puis créer le `SpellVisualKit` correspondant.

### 2.2. Points d'ancrage courants

| AttachmentID | Emplacement |
|--------------|-------------|
| `-1` | Non attaché (au sol) |
| `0` | Base du personnage |
| `1` | Main droite |
| `2` | Main gauche |
| `3` | Tête |
| `4` | Torse |
| `5` | Dos |

> **📝 Note** : La liste complète des points d'ancrage dépend du modèle du personnage. Il n'existe pas de documentation exhaustive, et le meilleur moyen est de tester empiriquement : « 需要自己去慢慢试试，总之这个我没找到定义 ».

### 2.3. Exemple concret

**Attacher l'effet « Aura de flammes » (ID 5000) au torse (ID 4) du personnage**

- **ID** : `6000`
- **ParentSpellVisualKitID** : `7000` (à créer à l'étape 3)
- **SpellVisualEffectNameID** : `5000`
- **AttachmentID** : `4` (torse)
- **OffsetX/Y/Z** : `0, 0, 0`
- **RotationX/Y/Z** : `0, 0, 0`

---

## Étape 3 : Définir le kit dans SpellVisualKit.dbc

Ce fichier regroupe **tous les effets** qui composent une phase visuelle du sort (incantation, buff, etc.). Chaque colonne correspond à un emplacement sur le modèle.

### 3.1. Structure du fichier (colonnes essentielles)

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique du kit. |
| 2 | **Unknown** | Integer | Type de sort (souvent `0`). |
| 3 | **AnimationID** | iRefID | Animation du lanceur (référence `AnimationData.dbc`). |
| 4-5 | **Unknown** | Integer | Inconnu. |
| 6 | **m_targetEffect** | iRefID | Effet sur la cible (référence `SpellVisualEffectName.dbc`). |
| 7 | **m_rightHandEffect** | iRefID | Effet sur la main droite. |
| 8 | **m_leftHandEffect** | iRefID | Effet sur la main gauche. |
| 9-14 | **Unknown** | Integer | Divers emplacements. |
| 15 | **CameraShakeID** | iRefID | Secousse de caméra (référence `SpellEffectCameraShakes.dbc`). |
| 22 | **Color / Effect** | Integer | **Couleur (décimal) ou ID d'effet** (voir note ci-dessous). |

> **📝 Note cruciale sur la colonne 22** : Cette colonne a un double comportement. Si la **colonne 18** est supérieure à 0, la colonne 22 est un **masque de couleur décimal**. Si la colonne 18 vaut 0, la colonne 22 référence un **autre ID d'effet**. Comme l'explique un moddeur sur Modcraft : « If Column18 > 0, then Column22 is the decimal colormask. If Column18 = 0, then Column22 refers to some kind of model located in some other tables ».

### 3.2. Exemple concret

**Kit « Aura de flammes » (buff)**

- **ID** : `7000`
- **AnimationID** : `0`
- **m_targetEffect** : `5000` (l'effet « Aura de flammes »)
- **m_rightHandEffect** : `5000`
- **m_leftHandEffect** : `5000`
- **Colonne 18** : `0`
- **Colonne 22** : `0` (pas de couleur personnalisée)

---

## Étape 4 : Définir le visuel dans SpellVisual.dbc

Ce fichier orchestre **quels kits** sont utilisés pour chaque phase du sort (pré-incantation, incantation, buff, canalisation, etc.).

### 4.1. Structure du fichier

| Colonne | Champ | Type | Description |
|---------|-------|------|-------------|
| 1 | **ID** | Integer | Identifiant unique du visuel. |
| 2 | **PrecastKit** | iRefID | Kit utilisé pendant la pré-incantation (référence `SpellVisualKit.dbc`). |
| 3 | **CastKit** | iRefID | Kit utilisé pendant l'incantation. |
| 4 | **TargetKit** | iRefID | Kit utilisé sur la cible. |
| 5 | **BuffKit** | iRefID | **Kit utilisé pour le buff/debuff** (le plus important pour un effet persistant). |
| 6 | **ChannelKit** | iRefID | Kit utilisé pendant la canalisation. |
| 7 | **HasMissile** | Boolean | `1` si le sort a un missile. |
| 8 | **MissileEffectID** | iRefID | Effet du missile (référence `SpellVisualEffectName.dbc`). |

> **📝 Note** : Pour un **effet visuel persistant** (comme une aura), c'est la colonne **5 (BuffKit)** qui est utilisée. C'est le kit qui sera affiché tant que le buff reste actif sur la cible. Comme l'explique wowdev : « 5 iRefID_BuffEffId uinteger The visual effect that can be seen while this buff/debuff remains on the target ».

### 4.2. Exemple concret

**Visuel « Aura de flammes »**

- **ID** : `8000`
- **PrecastKit** : `7000`
- **CastKit** : `7000`
- **TargetKit** : `7000`
- **BuffKit** : `7000`
- **ChannelKit** : `0`
- **HasMissile** : `0`
- **MissileEffectID** : `0`

---

## Étape 5 : Lier le visuel au sort dans Spell.dbc

C'est l'étape finale côté client : référencer l'ID de `SpellVisual.dbc` dans le sort qui déclenche l'effet.

### 5.1. Colonne à modifier

| Colonne | Champ | Description |
|---------|-------|-------------|
| **132** | **SpellVisualID** | **ID du visuel** (référence `SpellVisual.dbc`). |

> **📝 Note** : La colonne exacte peut varier légèrement selon la version de `Spell.dbc`. Dans le DBC 3.3.5 standard, c'est la colonne **132** qui contient `SpellVisualID`. Vérifiez la structure de votre fichier avec votre éditeur DBC.

### 5.2. Exemple concret

Pour un sort existant (par exemple, un buff de feu), remplacez la valeur de la colonne 132 par `8000` (l'ID de votre `SpellVisual`). Vous pouvez aussi créer un **nouveau sort** (voir le guide sur les recettes ou les mascottes pour la création de sorts).

---

## Étape 6 : Créer le patch MPQ côté client

1. Créez la structure de dossiers :
   ```
   patch-4/
   └── DBFilesClient/
       ├── SpellVisualEffectName.dbc
       ├── SpellVisualKitModelAttach.dbc
       ├── SpellVisualKit.dbc
       ├── SpellVisual.dbc
       └── Spell.dbc
   ```
2. Placez les fichiers DBC modifiés dans `DBFilesClient`.
3. **Placez également le modèle `.m2`** (et ses fichiers `.skin`, `.blp` associés) dans le dossier approprié du MPQ :
   ```
   patch-4/
   └── Spells/
       └── Fire_Aura.m2
       └── Fire_Aura00.skin
       └── Fire_Aura.blp
   ```
4. Créez `patch-4.MPQ` avec Ladik's MPQ Editor.
5. Placez-le dans le dossier `Data` du client.

> **⚠️ Important** : Le nom du patch doit être supérieur au dernier patch existant dans votre dossier `Data`. Si vous avez `patch-3.MPQ`, nommez le vôtre `patch-4.MPQ` ou `patch-Z.MPQ`. Le dossier `Spells` doit être à la **racine** du MPQ (pas dans `DBFilesClient`).

---

## Étape 7 : Redémarrer et tester

1. **Redémarrez votre serveur** (uniquement si vous avez modifié la base de données, ce qui n'est pas nécessaire pour un effet visuel pur).
2. **Reconnectez-vous** au jeu avec le client patché.
3. **Lancez le sort** qui utilise votre effet visuel.
4. **Vérifiez** que l'effet apparaît correctement sur le personnage, la cible, ou les deux.

---

## Alternatives côté serveur (sans modification DBC)

Si vous ne voulez pas modifier les DBC, vous pouvez utiliser des mécanismes **côté serveur** pour attacher un effet visuel à une créature ou un joueur.

### Via `creature_addon` ou `creature_template_addon`

Les tables `creature_addon` et `creature_template_addon` permettent d'appliquer une **aura visuelle** à une créature sans modifier les DBC. Comme l'explique la documentation AzerothCore : « This field controls any auras to be applied on the creature (both in effect and visually). To apply multiple auras, you can add more aura entries, separating each entry by a space ».

```sql
UPDATE `creature_template_addon`
SET `auras` = '12345'  -- ID du sort dont l'effet visuel est souhaité
WHERE `entry` = 90001;
```

> **📝 Note** : Cette méthode **ne crée pas** un nouvel effet visuel : elle applique un effet **existant** (référencé par un ID de sort) à une créature. C'est une solution simple pour ajouter une aura visuelle sans toucher aux DBC.

### Via `spell_area`

La table `spell_area` applique une aura à un joueur dans une zone donnée. Utile pour les effets environnementaux : « This table is used to apply a specific spell aura to the player within an area in the game ».

---

## Dépannage

| Problème | Cause possible | Solution |
|----------|----------------|----------|
| L'effet n'apparaît pas | `SpellVisualID` incorrect dans `Spell.dbc` | Vérifiez la colonne 132. |
| L'effet est invisible | Chemin du modèle incorrect dans `SpellVisualEffectName.dbc` | Vérifiez le chemin et la présence du `.m2` dans le MPQ. |
| Le client crash au chargement | Référence circulaire ou ID inexistant | Vérifiez que chaque ID existe bien dans le DBC suivant. |
| L'effet apparaît au mauvais endroit | `AttachmentID` incorrect | Testez différentes valeurs (`0`, `1`, `2`, `4`). |
| L'effet est trop petit/grand | `Scale` dans `SpellVisualEffectName.dbc` | Ajustez la colonne 5. |
| L'effet ne persiste pas | `BuffKit` vide dans `SpellVisual.dbc` | Remplissez la colonne 5. |
| Erreur « File not found » | Modèle absent du MPQ | Vérifiez que le dossier `Spells` est à la racine du MPQ. |

---

## Résumé des IDs utilisés

| Élément | ID | Fichier/Table |
|---------|-----|---------------|
| Modèle d'effet | `5000` | `SpellVisualEffectName.dbc` |
| Attachement | `6000` | `SpellVisualKitModelAttach.dbc` |
| Kit visuel | `7000` | `SpellVisualKit.dbc` |
| Visuel global | `8000` | `SpellVisual.dbc` |
| Sort | `97000` | `Spell.dbc` (colonne 132) |

---

## Conclusion

Vous savez maintenant créer un effet visuel personnalisé complet dans AzerothCore 3.3.5. Le processus repose sur une **chaîne de quatre fichiers DBC** (`SpellVisualEffectName.dbc` → `SpellVisualKitModelAttach.dbc` → `SpellVisualKit.dbc` → `SpellVisual.dbc`) qui doit être construite **dans l'ordre**. Le point critique est la **cohérence des IDs** : chaque maillon doit référencer un ID existant dans le fichier suivant. Pour les effets simples, l'alternative côté serveur via `creature_addon` permet d'appliquer des auras visuelles existantes sans modifier les DBC. Testez toujours progressivement et vérifiez que votre modèle `.m2` est bien présent dans le MPQ.


### 4.2 — `docs/world-merge/phases/phase-2-fusion.md`

```markdown
# Phase 2 — Fusion des ADT

> **Durée** : 1 à 2 semaines · **Difficulté** : ⭐⭐⭐⭐⭐

## 🎯 Objectifs

- Rassembler les ADT dans une nouvelle grille
- Corriger tous les offsets
- Générer le WDT

## 2.1 Création du projet Rius

1. Lance **Rius Zone Masher**.
2. `File → New Project` → nom : `AzerothOrdered`.
3. Grille : **64×64** ADT.
4. Emplacement : `work/mash/AzerothOrdered/`.

## 2.2 Import des ADT

1. `Import → ADT Folder` → sélectionne `work/adt/azeroth/`.
2. Répète pour `work/adt/kalimdor/`.

## 2.3 Positionnement des continents

### Décalage Azeroth (vers l'est)

Sélectionne tous les ADT provenant d'`azeroth/` et applique un offset :

- **Offset X** : `+18`
- **Offset Y** : `0`

### Décalage Kalimdor (vers l'ouest)

Sélectionne tous les ADT provenant de `kalimdor/` :

- **Offset X** : `-18`
- **Offset Y** : `0`

## 2.4 Correction des offsets internes ⚠️

**C'est l'étape la plus critique.** Sans elle, tous les PNJ, bâtiments et objets flotteront dans le vide.

Dans Rius :

1. `Tools → Fix Offsets` → sélectionne **"All Selected ADT"**.
2. Coche **"Recalculate Doodad Offsets"** et **"Recalculate WMO Offsets"**.
3. Lance l'opération (peut prendre plusieurs heures).

## 2.5 Génération du WDT

1. `File → Generate WDT`.
2. Nom : `AzerothOrdered`.
3. Options :
   - ✅ Include all ADT
   - ✅ Generate WDL
   - ✅ Preserve original filenames
4. Lance `Mash`.

## 2.6 Vérification avec Noggit Red

1. Lance **Noggit Red**.
2. `File → Open` → sélectionne `AzerothOrdered.wdt`.
3. Navigue dans le monde.

**Ce que tu dois voir** :
- ✅ Terrain continu
- ✅ Bâtiments à leur place
- ✅ Pas de trous

## 2.7 Ajustements manuels

- Lisse les côtes avec l'outil **"Flatten/Blur"**.
- Crée des **îles de liaison** entre les deux continents.
- Ajoute des **routes maritimes**.

## ✅ Checklist de fin de phase

- [ ] ADT importés dans Rius
- [ ] Continents décalés correctement
- [ ] Offsets corrigés
- [ ] WDT généré
- [ ] Vérification Noggit réussie

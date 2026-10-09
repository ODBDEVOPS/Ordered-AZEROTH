# Outils requis

## 🔧 Chaîne d'outils complète

| Outil | Usage | Lien |
|---|---|---|
| **MPQ Editor** | Extraction MPQ | [Zeox](http://www.zezula.net/en/mpq/download.html) |
| **Noggit Red** | Édition terrain 3D | [GitHub](https://github.com/Noggit-Red/noggit-red) |
| **Rius Zone Masher** | Fusion ADT | [GitHub](https://github.com/Rius/Zone-Masher) |
| **OffsetFix** | Correction offsets | [GitHub](https://github.com/MaNGOS/OffsetFix) |
| **ADTAdder** | Création ADT vides | [GitHub](https://github.com/MaNGOS/ADTAdder) |
| **MyDBCEditor** | Édition DBC | [GitHub](https://github.com/MaNGOS/MyDBCEditor) |
| **Extracteurs AC** | VMaps/MMaps | Compilés avec le serveur |

## 📋 Checklist d'installation

- [ ] .NET Framework 4.8
- [ ] Visual C++ Redistributable 2015-2022
- [ ] 50+ Go d'espace libre
- [ ] Client WoW 3.3.5a propre
- [ ] AzerothCore compilé

## 🛠️ Configuration initiale

### Noggit Red — `noggit.conf`

```ini
ClientPath = "C:/World of Warcraft/"
ProjectPath = "./work/"
ViewDistance = 2000
LoadThreads = 4
MaxFPS = 60
ShowWater = true
ShowDoodads = true
```
💡 Conseils
Toujours travailler sur des copies

Sauvegarder toutes les 30 minutes

Tester dans Noggit après chaque modification

Documenter chaque changement


---

## 📝 Étape 6 : Remplir les 2 fichiers de référence

### 6.1 — `docs/world-merge/reference/adt-coordinates.md`

```markdown
# Coordonnées ADT de référence

## 📐 Système de coordonnées WoW

Chaque ADT couvre **533.33 × 533.33 unités**. La grille va de `00_00` (nord-ouest) à `63_63` (sud-est).

Formule :
worldX = (32 - adtX) * 533.33
worldY = (32 - adtY) * 533.33
```


## 🗺️ Coordonnées Azeroth (extraits)

| Zone | ADT X_Y |
|---|---|
| Elwynn Forest | 32_48, 32_49, 33_48, 33_49 |
| Stormwind | 32_49, 33_49 |
| Westfall | 30_48, 31_48 |
| Redridge | 34_46, 34_47 |
| Duskwood | 32_46, 33_46 |
| Burning Steppes | 32_44, 33_44 |
| Tirisfal Glades | 31_41, 32_41 |
| Hillsbrad Foothills | 29_44, 30_44 |
| Stranglethorn Vale | 30_50, 31_50, 32_50 |

## 🗺️ Coordonnées Kalimdor (extraits)

| Zone | ADT X_Y |
|---|---|
| Durotar | 42_40, 43_40 |
| Orgrimmar | 42_40 |
| Mulgore | 40_42, 41_42 |
| Barrens | 42_38, 43_38, 44_38 |
| Ashenvale | 40_36, 41_36 |
| Darkshore | 38_35, 39_35 |
| Teldrassil | 38_33, 39_33 |
| Tanaris | 44_44, 45_44 |
| Silithus | 42_46, 43_46 |

## 🔧 Calculs d'offset

**Kalimdor décalé de -18 en X :**


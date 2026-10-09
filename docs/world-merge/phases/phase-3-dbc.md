# Phase 3 — Configuration DBC

> **Durée** : 1 à 2 jours · **Difficulté** : ⭐⭐⭐☆☆

## 🎯 Objectifs

- Déclarer la nouvelle carte dans Map.dbc
- Ajouter les entrées AreaTable.dbc
- Compiler un patch client

## 3.1 Map.dbc

1. Ouvre `DBFilesClient/Map.dbc` avec **MyDBCEditor**.
2. Duplique la ligne d'Azeroth (ID 0).
3. Modifie les champs :

| Champ | Valeur |
|---|---|
| ID | `1000` |
| Directory | `AzerothOrdered` |
| InstanceType | `0` |
| PVP | `0` |
| MapName | `Azeroth Ordered` |
| LoadingScreenID | `35` |

## 3.2 AreaTable.dbc

Pour chaque zone, modifie le champ `ContinentID` → `1000`.

⚠️ Sauvegarde une copie avant modification.

## 3.3 Autres DBC à vérifier

| DBC | Modification |
|---|---|
| `WorldMapArea.dbc` | Met à jour les coordonnées |
| `WorldMapContinent.dbc` | Ajoute la carte 1000 |
| `Light.dbc` | Vérifie les zones lumineuses |
| `SoundEmitters.dbc` | Recalcule les positions |

## 3.4 Création du patch client

Structure :

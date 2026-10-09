# Phase 4 — Configuration AzerothCore

> **Durée** : 3 à 5 jours · **Difficulté** : ⭐⭐⭐⭐☆

## 🎯 Objectifs

- Enregistrer la carte 1000 côté serveur
- Migrer les spawns
- Régénérer VMaps et MMaps

## 4.1 Enregistrement de la carte

Dans `acore_world` :

```sql
INSERT INTO `worldmap_info`
    (`entry`, `map_type`, `parent_map`, `max_players`, `reset_interval`,
     `ghost_entrance_map`, `ghost_entrance_x`, `ghost_entrance_y`,
     `time_of_day_override`, `expansion`, `difficulty`, `allowed_difficulty`)
VALUES
    (1000, 0, 0, 0, 0, 1000, 0, 0, -1, 0, 0, 0);

# Phase 1 — Préparation

> **Durée** : 2 à 3 jours · **Difficulté** : ⭐⭐☆☆☆

## 🎯 Objectifs

- Sauvegarder l'existant
- Extraire les ADT sources
- Cartographier les zones à déplacer

## 1.1 Sauvegarde intégrale

**Obligatoire.** Une erreur ici peut corrompre ton client.

```bash
# Client
cp -r "World of Warcraft/Data" "World of Warcraft/Data.backup"

# Serveur
mysqldump -u root -p acore_world > backup_world.sql
mysqldump -u root -p acore_characters > backup_characters.sql
mysqldump -u root -p acore_auth > backup_auth.sql

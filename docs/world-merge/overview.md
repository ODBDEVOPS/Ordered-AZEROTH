# Vue d'ensemble

## Pourquoi fusionner les continents ?

Azeroth Ordered repose sur un principe : **un monde continu, sans frontière arbitraire**. Dans WoW 3.3.5, Azeroth (ID 0) et Kalimdor (ID 1) sont deux cartes séparées. Passer de l'une à l'autre nécessite un chargement et un portail.

En les fusionnant :

- Les joueurs peuvent **voler d'un continent à l'autre** sans chargement
- Les **événements mondiaux** traversent toutes les zones
- Les **trajets maritimes** deviennent physiques
- L'identité **"Ordered"** prend tout son sens

## Défis techniques

| Défi | Description | Solution |
|---|---|---|
| **Coordonnées** | Chaque ADT a ses propres offsets | OffsetFix + Rius Zone Masher |
| **WDT** | Un seul WDT doit couvrir les deux continents | Rius Zone Masher |
| **DBC** | Map.dbc doit référencer la nouvelle carte | MyDBCEditor |
| **VMaps/MMaps** | À régénérer intégralement | Extracteurs AzerothCore |
| **Spawns** | Migrer créatures et objets vers map 1000 | SQL custom |
| **Transports** | Recalculer les trajets | SmartAI + waypoints |

## Prérequis

- Client WoW 3.3.5a (build 12340)
- Serveur AzerothCore Ordered compilé
- 50+ Go d'espace disque libre
- Patience (projet sur plusieurs semaines)

## Durée estimée

| Phase | Durée |
|---|---|
| Préparation | 2-3 jours |
| Fusion ADT | 1-2 semaines |
| DBC | 1-2 jours |
| Serveur | 3-5 jours |
| Tests | 2-4 semaines |

**Total : 6 à 10 semaines** pour un résultat jouable.

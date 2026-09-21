C'est un projet d'envergure : recréer une **"Azeroth Unifiée" (Ordered Azeroth)** sous la forme d'un supercontinent unique sur un client 3.3.5a. Cette carte fusionne des zones de Kalimdor (Hyjal, Un'Goro, Ahn'Qiraj, Uldum), de Northrend (Sholazar, Ulduar, Wyrmrest) et de Pandarie (Vallée des Fleurs Éternelles). 

Voici le **MODOP (Mode Opératoire)** technique pour tenter de reproduire cette carte dans AzerothCore 3.3.5a.

---

### ⚠️ Phase 0 : Analyse des limites techniques (À lire absolument)

Avant de commencer, vous devez connaître les limites du moteur 3.3.5a :
1.  **La limite des 64x64 ADT** : Un client 3.3.5a ne peut pas charger une carte plus grande qu'une grille de 64x64 tuiles ADT (environ la taille de Northrend). Fusionner Kalimdor, les Royaumes de l'Est et Northrend en une seule carte *jouable* dans le même monde physique est impossible sans dépasser cette limite. Vous devrez donc **réduire l'échelle** ou créer une carte "hub" sur mesure.
2.  **Les assets Pandarie** : La Vallée des Fleurs Éternelles n'existe pas dans le client 3.3.5a. Vous devrez extraire ces fichiers (M2/WMO/ADT) d'un client MoP et les convertir pour les rendre compatibles avec le client WotLK (via des outils comme **Blender** avec les add-ons de conversion ou **WoW Model Viewer**).
3.  **Le centre de la carte** : Le Puits d'Éternité au centre implique de modifier le terrain pour créer un cratère central entouré d'eau.

---

### 🗺️ Phase 1 : Création du terrain (Client-side)

Vous ne pouvez pas simplement copier-coller des morceaux de Kalimdor dans Northrend. Il faut créer une carte personnalisée.

1.  **Créer un MapID personnalisé** : Choisissez un ID libre (par exemple, `MapID 2000`).
2.  **Utiliser Noggit Red** : C'est l'outil indispensable pour sculpter le terrain.
    *   Ouvrez une carte existante vide (ou créez une grille 64x64 personnalisée).
    *   **Sculpter le relief** : Utilisez les outils de heightmap pour créer les montagnes de Hyjal à l'ouest, le bassin central, les plaines d'Uldum au sud, etc.
    *   **Texturer le sol** : Appliquez les textures de sol correspondantes (neige pour le nord, jungle pour Sholazar/Un'Goro, désert pour Uldum, etc.). *Note : Vous devrez peut-être importer des textures de Pandarie pour la Vallée.*
3.  **Importer les WMO/M2 majeurs** :
    *   Importez les bâtiments d'Ulduar, le Temple de Wyrmrest, les portes d'Ahn'Qiraj, les structures d'Uldaman et les tentacules de N'Zoth (probablement des WMO personnalisés à créer ou à adapter).
    *   Placez le Puits d'Éternité au centre exact de la carte.

### 💾 Phase 2 : Intégration Serveur & DBC

Une fois le terrain prêt et packagé dans un fichier MPQ (ou CASC) pour le client, il faut le déclarer au serveur.

1.  **Modifier `Map.dbc`** : Ajoutez une nouvelle ligne pour votre MapID 2000. Définissez le type de carte (0 = Monde normal).
2.  **Modifier `AreaTable.dbc`** :
    *   Créez de nouveaux `AreaID` pour chaque sous-zone (Sholazar, Uldum, Hyjal, etc.).
    *   Assignez le `MapID 2000` à ces zones.
    *   *Astuce AzerothCore* : Utilisez la table `areatable_dbc` dans la base de données `world` pour surcharger ces paramètres sans modifier le fichier DBC binaire.
3.  **Table `worldmap_info`** :
    *   Insérez une entrée pour le MapID 2000. Définissez les limites de la carte (Boundaries) et le type de vol autorisé.

### 🐺 Phase 3 : Peupler le supercontinent (SQL)

C'est ici que votre stratégie d'utiliser les données existantes est cruciale. Vous allez importer les zones d'origine et les **translater** vers leurs nouvelles coordonnées.

1.  **Extraire les données d'origine** :
    *   `creature` et `gameobject` pour Sholazar (ZoneID 3711), Un'Goro (ZoneID 490), Uldum (ZoneID 5034 - *attention, Uldum n'existe pas en 3.3.5a, il faudra utiliser une zone similaire ou créer les PNJ de zéro*), etc.
2.  **Créer une table de correspondance (Mapping)** :
    *   Établissez un tableau Excel avec les coordonnées actuelles (X, Y, Z) de chaque zone et leurs nouvelles coordonnées sur votre supercontinent.
    *   *Exemple* : Le centre de Sholazar (X=5500, Y=4500) doit être déplacé vers le nord-ouest de votre nouvelle carte (X=2000, Y=2000).
3.  **Mettre à jour les coordonnées en masse** :
    *   Utilisez des requêtes SQL pour mettre à jour les tables `creature` et `gameobject`.
    ```sql
    -- Exemple : Déplacer les créatures de Sholazar vers le Nord-Ouest
    UPDATE `creature` 
    SET `map` = 2000, 
        `zoneId` = [NOUVEAU_ZONE_ID_SHOLAZAR],
        `position_x` = `position_x` - 3500, -- Ajustez selon vos calculs
        `position_y` = `position_y` - 2500,
        `position_z` = `position_z` + 100
    WHERE `map` = 571 AND `zoneId` = 3711;
    ```
4.  **Réassigner les quêtes (`quest_template`)** :
    *   Mettez à jour le champ `QuestSortID` (ou `ZoneOrSort`) des quêtes pour qu'elles pointent vers les nouveaux `AreaID` de votre supercontinent.
5.  **Lier les PNJ aux quêtes** :
    *   Vérifiez les tables `creature_queststarter` et `creature_questender` pour vous assurer que les PNJ déplacés donnent toujours les bonnes quêtes.

### 🛣️ Phase 4 : Connectivité et Pathfinding (Vmaps/Mmaps)

1.  **Générer les Vmaps et Mmaps** :
    *   C'est l'étape la plus longue. Une fois le client à jour avec votre nouveau MPQ, relancez `vmap4extractor` et `mmaps_generator` sur votre nouveau MapID 2000.
    *   Sans cela, les PNJ traverseront les murs et tomberont à travers le sol.
2.  **Transports et Vols** :
    *   Créez des chemins de vol (`taxipath` et `taxinode`) reliant les différentes régions de votre supercontinent.
    *   Si vous voulez des bateaux ou des zeppelins, vous devrez scripter leurs trajets (généralement via des `gameobject` de type transport ou des scripts Eluna).

### 💡 Conseils pour la progression

*   **Commencez petit** : Ne créez pas tout le supercontinent d'un coup. Commencez par une seule zone (par exemple, fusionnez Sholazar et Un'Goro). Assurez-vous que le terrain, les PNJ et les quêtes fonctionnent.
*   **Utilisez Eluna** : Pour le Puits d'Éternité au centre, vous pouvez créer un script Eluna qui téléporte les joueurs ou déclenche des événements visuels (sorts, animations) lorsqu'ils s'en approchent.
*   **Cohérence visuelle** : Le défi principal de cette carte est le mélange des biomes. Utilisez des objets de transition (montagnes, rivières) pour masquer les coutures entre les zones importées (par exemple, entre le désert d'Uldum et la jungle d'Un'Goro).

Ce projet est un marathon, pas un sprint. En utilisant les données existantes comme base et en les déplaçant méthodiquement via SQL, vous pouvez donner vie à cette "Azeroth Ordonnée" étape par étape. Bon courage !

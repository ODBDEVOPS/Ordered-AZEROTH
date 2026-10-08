Voici le workflow et le code complet pour ajouter un PNJ Skip DK (*Death Knight Skip*) dans Acherus (la zone de départ des Chevaliers de la Mort). Il permet de valider automatiquement les quêtes de la zone de départ, de faire passer le joueur au niveau 58/80 (selon votre choix), de lui apprendre ses sorts et de le téléporter directement dans la capitale de sa faction (Hurlevent ou Orgrimmar).

---

## 🛠️ **WORKFLOW COMPLET : PNJ Skip Zone de Départ DK**

---

### **ÉTAPE 1 : Structure du Projet GitHub**

```text
mod-dk-skip/
├── lua_scripts/
│   └── dk_skip/
│       ├── dk_skip_config.lua
│       └── dk_skip_npc.lua
├── sql/
│   └── custom/
│       └── dk_skip_creature.sql
├── docs/
│   └── README.md
├── LICENSE
└── .gitignore

```

---

### **ÉTAPE 2 : Déclarer le PNJ en Base de Données**

```sql
-- sql/custom/dk_skip_creature.sql

-- 1. Création du Template PNJ (Passage d'Acherus)
DELETE FROM `creature_template` WHERE `entry` = 99100;
INSERT INTO `creature_template` (
    `entry`, `difficulty_entry_1`, `difficulty_entry_2`, `difficulty_entry_3`, 
    `KillCredit1`, `KillCredit2`, `modelid1`, `modelid2`, `modelid3`, `modelid4`, 
    `name`, `subname`, `IconName`, `gossip_menu_id`, `minlevel`, `maxlevel`, 
    `exp`, `faction`, `npcflag`, `scale`, `rank`, `dmgschool`, `BaseAttackTime`, 
    `RangeAttackTime`, `BaseVariance`, `RangeVariance`, `unit_class`, `unit_flags`, 
    `unit_flags2`, `dynamicflags`, `family`, `trainer_type`, `trainer_spell`, 
    `trainer_class`, `trainer_race`, `type`, `type_flags`, `lootid`, `pickpocketLootId`, 
    `SkinLootId`, `PetSpellDataId`, `VehicleId`, `mingold`, `maxgold`, `AIName`, 
    `MovementType`, `HoverHeight`, `HealthModifier`, `ManaModifier`, `ArmorModifier`, 
    `DamageModifier`, `ExperienceModifier`, `RacialLeader`, `movementId`, `RegenHealth`, 
    `mechanic_immune_mask`, `spell_school_immune_mask`, `flags_extra`, `ScriptName`, `VerifiedBuild`
) VALUES (
    99100, 0, 0, 0, 
    0, 0, 25277, 0, 0, 0, 
    'Messager d Acherus', 'Passeur d Âmes (Skip Zone DK)', '', 0, 80, 80, 
    2, 35, 1, 1, 1, 0, 2000, 
    2000, 1, 1, 1, 0, 
    2048, 0, 0, 0, 0, 
    0, 0, 7, 0, 0, 0, 
    0, 0, 0, 0, 0, '', 
    0, 1, 1, 1, 1, 
    1, 1, 0, 0, 1, 
    0, 0, 0, '', 12340
);

-- 2. FactionTemplate du PNJ (35 = Neutre envers tout le monde)[cite: 1]
-- Spawns du PNJ dans la zone de départ DK (Acherus : Le Fort d'Ébène)
DELETE FROM `creature` WHERE `id` = 99100;
INSERT INTO `creature` (`guid`, `id`, `map`, `spawnMask`, `phaseMask`, `position_x`, `position_y`, `position_z`, `orientation`, `spawntimesecs`, `spawndistance`, `currentwaypoint`, `curhealth`, `curmana`, `MovementType`, `npcflag`, `unit_flags`, `dynamicflags`) VALUES
(991000, 99100, 609, 1, 1, 2358.12, -5661.10, 382.25, 0.52, 300, 0, 0, 10000, 0, 0, 1, 0, 0);

```

---

### **ÉTAPE 3 : Fichier de Configuration Lua**

```lua
-- lua_scripts/dk_skip/dk_skip_config.lua
DKSkipConfig = {}

-- ID du PNJ
DKSkipConfig.NPCEntry = 99100

-- Niveau appliqué au joueur lors du skip
DKSkipConfig.TargetLevel = 58 -- Passez à 80 si vous voulez un skip total du leveling

-- Coordonnées des capitales de départ
DKSkipConfig.AllianceTeleport = { map = 0, x = -8833.38, y = 628.62, z = 94.01, o = 1.0 }  -- Hurlevent
DKSkipConfig.HordeTeleport    = { map = 1, x = 1601.32, y = -4378.81, z = 9.92, o = 0.0 }  -- Orgrimmar

-- Liste des quêtes principales DK à valider automatiquement pour débloquer les sorts / montures
DKSkipConfig.QuestsToComplete = {
    12619, -- In Service of the Lich King
    12687, -- The Scourge Challenge
    12698, -- Grand Theft Palomino
    12701, -- Into the Realm of Shadows
    12733, -- Death Comes From Above
    12801, -- The Light of Dawn
    13188, -- Where Kings Walk (Alliance)
    13189  -- Warchief's Blessing (Horde)
}

-- Sorts clés à enseigner (Talents, Monture, Portail)
DKSkipConfig.SpellsToLearn = {
    53428, -- Runeforging (Runeforge)
    50977, -- Gate of Acherus (Porte de la mort)
    48778, -- Acherus Deathcharger (Monture DK)
    54197  -- Cold Weather Flying (Vol par temps froid)
}

```

---

### **ÉTAPE 4 : Script Eluna Lua du PNJ**

```lua
-- lua_scripts/dk_skip/dk_skip_npc.lua

local function OnGossipHello(event, player, creature)
    -- Vérifier que la classe du joueur est bien un Chevalier de la Mort (Classe ID 6)[cite: 1]
    if player:GetClass() ~= 6 then
        player:GossipClearMenu()
        player:GossipMenuAddItem(0, "Seuls les Chevaliers de la Mort peuvent utiliser mes services.", 1, 99)
        player:GossipSendMenu(1, creature)
        return true
    end

    player:GossipClearMenu()
    player:GossipMenuAddItem(0, "⚡ Sauter l introduction DK et rejoindre ma capitale", 1, 1)
    player:GossipMenuAddItem(0, "❌ Non merci, je souhaite faire les quêtes normalement", 1, 2)
    player:GossipSendMenu(1, creature)
    return true
end

local function OnGossipSelect(event, player, creature, sender, action)
    if action == 1 then
        -- 1. Monter au niveau défini
        if player:GetLevel() < DKSkipConfig.TargetLevel then
            player:SetLevel(DKSkipConfig.TargetLevel)
        end

        -- 2. Valider la suite de quêtes du Fort d'Ébène
        for _, questId in ipairs(DKSkipConfig.QuestsToComplete) do
            if player:GetQuestStatus(questId) ~= 2 then -- Status 2 = Complétée
                player:CompleteQuest(questId)
            end
        end

        -- 3. Apprendre les compétences requises
        for _, spellId in ipairs(DKSkipConfig.SpellsToLearn) do
            if not player:HasSpell(spellId) then
                player:LearnSpell(spellId)
            end
        end

        -- 4. Réinitialiser et accorder les points de talents
        player:SendTalentWipeConfirm(creature)

        -- 5. Téléporter le joueur selon sa faction (0 = Alliance, 1 = Horde)[cite: 1]
        local faction = player:GetTeam()
        if faction == 0 then
            local loc = DKSkipConfig.AllianceTeleport
            player:Teleport(loc.map, loc.x, loc.y, loc.z, loc.o)
            player:SendBroadcastMessage("|cFF00FF00Vous avez passé la zone d introduction ! Bienvenue à Hurlevent.|r")
        else
            local loc = DKSkipConfig.HordeTeleport
            player:Teleport(loc.map, loc.x, loc.y, loc.z, loc.o)
            player:SendBroadcastMessage("|cFF00FF00Vous avez passé la zone d introduction ! Bienvenue à Orgrimmar.|r")
        end

        player:GossipComplete()
    elseif action == 2 or action == 99 then
        player:GossipComplete()
    end
end

RegisterCreatureGossipEvent(DKSkipConfig.NPCEntry, 1, OnGossipHello)
RegisterCreatureGossipEvent(DKSkipConfig.NPCEntry, 2, OnGossipSelect)

```

---

### **Résumé des Fonctionnalités :**

* **Validation Automatique** : Complète la chaîne de quêtes d'Acherus pour débloquer l'accès complet aux compétences et aux factions d'origine.


* **Apprentissage des Sorts** : Enseigne la monture de classe, le forgeage des runes et la porte de la mort.


* **Téléportation Intelligente** : Redirige vers Orgrimmar pour la Horde ou Hurlevent pour l'Alliance selon la race du joueur.


* **Sécurité** : Bloque l'interaction si le personnage n'est pas un Chevalier de la Mort.

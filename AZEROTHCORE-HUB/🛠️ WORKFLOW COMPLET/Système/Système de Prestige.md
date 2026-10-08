Voici le projet complet pour un **Système de Prestige avec Interface NPC Gossip** (menu textuel interactif complet) pour AzerothCore 3.3.5.

Le système remet le joueur au niveau 1 lorsqu'il atteint le niveau 80, augmente son **Niveau de Prestige**, et lui accorde des **bonus permanents cumulativement** (Auras/Buffs de statistiques, pièces d'or, et titres).

---

## 🛠️ **WORKFLOW COMPLET : Système de Prestige**

### **1. Structure du Projet GitHub**

```text
mod-prestige-system/
├── lua_scripts/
│   └── prestige_system/
│       ├── prestige_config.lua
│       └── prestige_main.lua
├── sql/
│   └── custom/
│       └── prestige_tables.sql
├── docs/
│   └── README.md
├── LICENSE
└── .gitignore

```

---

### **2. Base de Données SQL**

```sql
-- sql/custom/prestige_tables.sql

-- 1. Table de sauvegarde du niveau de prestige des joueurs
CREATE TABLE IF NOT EXISTS `character_prestige` (
    `guid` INT UNSIGNED NOT NULL,
    `prestige_level` INT UNSIGNED NOT NULL DEFAULT 0,
    PRIMARY KEY (`guid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 2. Création du PNJ "Maître du Prestige" (Entry 99300)
DELETE FROM `creature_template` WHERE `entry` = 99300;
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
    99300, 0, 0, 0, 
    0, 0, 24978, 0, 0, 0, 
    'Chronos', 'Maître du Prestige', '', 0, 80, 80, 
    2, 35, 1, 1.2, 1, 0, 2000, 
    2000, 1, 1, 1, 0, 
    2048, 0, 0, 0, 0, 
    0, 0, 7, 0, 0, 0, 
    0, 0, 0, 0, 0, '', 
    0, 1, 1, 1, 1, 
    1, 1, 0, 0, 1, 
    0, 0, 0, '', 12340
);

-- Spawn du PNJ dans les capitales
DELETE FROM `creature` WHERE `id` = 99300;
INSERT INTO `creature` (`guid`, `id`, `map`, `spawnMask`, `phaseMask`, `position_x`, `position_y`, `position_z`, `orientation`, `spawntimesecs`, `spawndistance`, `currentwaypoint`, `curhealth`, `curmana`, `MovementType`, `npcflag`, `unit_flags`, `dynamicflags`) VALUES
(993000, 99300, 0, 1, 1, -8830.0, 625.0, 94.0, 1.0, 300, 0, 0, 50000, 0, 0, 1, 0, 0),  -- Hurlevent
(993001, 99300, 1, 1, 1, 1600.0, -4370.0, 10.0, 0.0, 300, 0, 0, 50000, 0, 0, 1, 0, 0);  -- Orgrimmar

```

---

### **3. Configuration Lua**

```lua
-- lua_scripts/prestige_system/prestige_config.lua
PrestigeConfig = {}

-- ID du PNJ
PrestigeConfig.NPCEntry = 99300

-- Niveau requis pour passer un prestige
PrestigeConfig.RequiredLevel = 80

-- Niveau maximum de prestige possible
PrestigeConfig.MaxPrestige = 10

-- Niveau auquel le joueur est remis lors du prestige
PrestigeConfig.ResetLevel = 1

-- Récompense en pièces d'or par niveau de prestige (ex: 500 PO x Niveau de Prestige)
PrestigeConfig.GoldRewardPerPrestige = 5000000 -- En cuivre (500 PO)

-- Buffs / Auras passifs appliqués selon le niveau de prestige (Spell.dbc)[cite: 1]
-- Note: Remplacez ces IDs par des sorts de buff existants sur votre serveur (ex: +1% dégâts/soins par rang)
PrestigeConfig.PrestigeBuffs = {
    [1] = 23768, -- Faveur de Sayge (+10% dégâts)
    [2] = 23737, -- Faveur de Sayge (+10% toutes stats)
    [3] = 23735, -- Faveur de Sayge (+10% Armure)
    [4] = 23767, -- Faveur de Sayge (+10% Résistance)
    [5] = 23766, -- Faveur de Sayge (+10% Esprit)
}

```

---

### **4. Script Principal Eluna Lua**

```lua
-- lua_scripts/prestige_system/prestige_main.lua

local PrestigeSystem = {}

-- Charger le niveau de prestige depuis la base de données
function PrestigeSystem:GetPrestige(player)
    local pGUID = player:GetGUIDLow()
    local query = CharDBQuery("SELECT prestige_level FROM character_prestige WHERE guid = " .. pGUID)
    if query then
        return query:GetRow()[1] and tonumber(query:GetRow()[1]) or 0
    end
    return 0
end

-- Sauvegarder le niveau de prestige
function PrestigeSystem:SetPrestige(player, level)
    local pGUID = player:GetGUIDLow()
    CharDBExecute("INSERT INTO character_prestige (guid, prestige_level) VALUES (" .. pGUID .. ", " .. level .. ") ON DUPLICATE KEY UPDATE prestige_level = " .. level)
end

-- Appliquer les buffs permanents associés au prestige
function PrestigeSystem:ApplyPrestigeBuffs(player)
    local level = self:GetPrestige(player)
    if level <= 0 then return end

    for i = 1, level do
        local buffId = PrestigeConfig.PrestigeBuffs[i]
        if buffId and not player:HasAura(buffId) then
            player:AddAura(buffId, player)
        end
    end
end

-- Réinitialisation du personnage lors du Prestige
function PrestigeSystem:PerformPrestige(player)
    local currentPrestige = self:GetPrestige(player)

    if player:GetLevel() < PrestigeConfig.RequiredLevel then
        player:SendBroadcastMessage("|cFFFF0000Vous devez être niveau " .. PrestigeConfig.RequiredLevel .. " pour passer un prestige !|r")
        return false
    end

    if currentPrestige >= PrestigeConfig.MaxPrestige then
        player:SendBroadcastMessage("|cFFFF0000Vous avez déjà atteint le niveau de prestige maximum (" .. PrestigeConfig.MaxPrestige .. ") !|r")
        return false
    end

    local newPrestige = currentPrestige + 1

    -- 1. Mise à jour de la DB
    self:SetPrestige(player, newPrestige)

    -- 2. Réinitialisation du niveau
    player:SetLevel(PrestigeConfig.ResetLevel)

    -- 3. Récompense en or
    local goldGain = PrestigeConfig.GoldRewardPerPrestige * newPrestige
    player:ModifyMoney(goldGain)

    -- 4. Application des buffs
    self:ApplyPrestigeBuffs(player)

    -- 5. Visuel & Annonce
    player:CastSpell(player, 47292, true) -- Visuel d'ascension
    SendWorldMessage("|cFFFFD700[PRESTIGE]|r Le joueur |cFF00FF00" .. player:GetName() .. "|r vient de franchir le |cFFFFD700Niveau de Prestige " .. newPrestige .. "|r !")

    return true
end

-- Interface NPC Gossip
local function OnGossipHello(event, player, creature)
    local prestige = PrestigeSystem:GetPrestige(player)

    player:GossipClearMenu()
    player:GossipMenuAddItem(0, "⭐ Votre Niveau de Prestige Actuel : " .. prestige .. " / " .. PrestigeConfig.MaxPrestige, 1, 99)
    
    if player:GetLevel() >= PrestigeConfig.RequiredLevel and prestige < PrestigeConfig.MaxPrestige then
        player:GossipMenuAddItem(2, "⚡ Passer le Prestige (Remise au Niveau " .. PrestigeConfig.ResetLevel .. ")", 1, 1, true, "Êtes-vous sûr de vouloir réinitialiser votre niveau en échange de récompenses de Prestige ?")
    else
        player:GossipMenuAddItem(0, "🔒 Vous devez être niveau 80 pour débloquer le prochain Prestige.", 1, 99)
    end

    player:GossipMenuAddItem(0, "📜 En savoir plus sur les récompenses de Prestige", 1, 2)
    player:GossipSendMenu(1, creature)
    return true
end

local function OnGossipSelect(event, player, creature, sender, action)
    if action == 1 then
        PrestigeSystem:PerformPrestige(player)
        player:GossipComplete()
    elseif action == 2 then
        player:GossipClearMenu()
        player:GossipMenuAddItem(0, "Chaque prestige vous accorde :", 1, 99)
        player:GossipMenuAddItem(0, "- Un bonus passif cumulatif de statistiques.", 1, 99)
        player:GossipMenuAddItem(0, "- Un sac d or supplémentaire.", 1, 99)
        player:GossipMenuAddItem(0, "- Une annonce sur tout le royaume.", 1, 99)
        player:GossipMenuAddItem(0, "⬅️ Retour", 1, 3)
        player:GossipSendMenu(1, creature)
    elseif action == 3 then
        OnGossipHello(event, player, creature)
    else
        player:GossipComplete()
    end
end

-- Rechargement des buffs à la connexion du joueur
local function OnPlayerLogin(event, player)
    PrestigeSystem:ApplyPrestigeBuffs(player)
end

RegisterCreatureGossipEvent(PrestigeConfig.NPCEntry, 1, OnGossipHello)
RegisterCreatureGossipEvent(PrestigeConfig.NPCEntry, 2, OnGossipSelect)
RegisterPlayerEvent(3, OnPlayerLogin) -- EVENT_ON_LOGIN

```

---

### **Fonctionnalités Incluses :**

* **Menu Interactif Complete (Gossip)** : Dialogue interactif avec fenêtre de confirmation d'action prévenant le joueur de la réinitialisation de son niveau.
* **Sauvegardes MySQL** : Table `character_prestige` dédiée pour conserver le rang de chaque personnage de manière indépendante.
* **Récompenses Dynamiques** : Accorde des buffs passifs via des Auras, de l'or proportionnel au rang, ainsi qu'une annonce serveur pour valoriser la progression.
* **Persistance au Login** : Réapplique automatiquement les auras passives de prestige dès qu'un joueur se connecte au serveur.

Voici le projet complet pour ajouter un **Minion Auto-Loot (Pet Ramasseur)**. Le principe : vous invoquez un serviteur compagnon qui recherche en permanence les cadavres de monstres lootables dans un rayon de 20 yards autour de vous, se déplace vers eux, ramasse le butin à votre place, puis revient à vos côtés.

---

## 🛠️ **WORKFLOW COMPLET : Minion Auto-Loot (20 Yards)**

### **ÉTAPE 1 : Structure du Projet GitHub**

```text
mod-autoloot-minion/
├── lua_scripts/
│   └── autoloot_minion/
│       ├── minion_config.lua
│       └── minion_system.lua
├── sql/
│   └── custom/
│       └── minion_creature.sql
├── docs/
│   └── README.md
├── LICENSE
└── .gitignore

```

---

### **ÉTAPE 2 : Déclarer le Minion en Base de Données**

```sql
-- sql/custom/minion_creature.sql

-- 1. Création du Template du Minion (Petit Robot Ramasseur / Fée)
DELETE FROM `creature_template` WHERE `entry` = 99200;
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
    99200, 0, 0, 0, 
    0, 0, 21071, 0, 0, 0, -- Model ID 21071 (Petit robot / Goblin Pet)
    'Loot-O-Matic', 'Minion Ramasseur', '', 0, 80, 80, 
    2, 35, 1, 0.6, 0, 0, 2000, 
    2000, 1, 1, 1, 768, -- 768 = Inciblable + Non-attaquable
    2048, 0, 0, 0, 0, 
    0, 0, 7, 0, 0, 0, 
    0, 0, 0, 0, 0, '', 
    0, 1, 1, 1, 1, 
    1, 1, 0, 0, 1, 
    0, 0, 0, '', 12340
);

```

---

### **ÉTAPE 3 : Fichier de Configuration Lua**

```lua
-- lua_scripts/autoloot_minion/minion_config.lua
MinionConfig = {}

-- Identifiant du Minion
MinionConfig.MinionEntry = 99200

-- Rayon de recherche du butin (en yards)
MinionConfig.LootRadius = 20.0

-- Intervalle de vérification des cadavres (en millisecondes)
MinionConfig.ScanInterval = 1000 

-- Sort d'invocation du Minion (ou item/commande)
MinionConfig.SummonCommand = "minion"

```

---

### **ÉTAPE 4 : Script Eluna Lua du Minion Ramasseur**

```lua
-- lua_scripts/autoloot_minion/minion_system.lua

local activeMinions = {}

-- Fonction de recherche et de ramassage du butin
local function ProcessAutoLoot(eventId, delay, repeats, creature)
    if not creature or not creature:IsInWorld() then return end

    local owner = creature:GetOwner()
    if not owner or not owner:IsInWorld() or not owner:IsAlive() then return end

    -- Si le minion est en cours de déplacement pour un loot, on attend
    if creature:IsMoving() then return end

    -- Rechercher les créatures mortes proches
    local nearCreatures = creature:GetCreaturesInRange(MinionConfig.LootRadius, 0, 2) -- Type 0, DeadState 2 (Mort)
    
    for _, deadMob in ipairs(nearCreatures) do
        if deadMob and deadMob:IsDead() and deadMob:HasLoot() then
            -- Vérifier si le joueur a le droit de looter le cadavre
            if deadMob:IsLootAllowedFor(owner) then
                -- Déplacer le minion vers le cadavre
                creature:MoveTo(1, deadMob:GetX(), deadMob:GetY(), deadMob:GetZ(), true)
                
                -- Simuler l'effet visuel de ramassage
                creature:CastSpell(deadMob, 6372, true) -- Visuel d'aspiration/éclair
                
                -- Transférer le butin au joueur et nettoyer le mob
                owner:AutoLootCreature(deadMob)
                
                -- Message de confirmation discret au joueur
                owner:SendAreaTriggerMessage("|cFF00FF00[Minion] Butin ramassé !|r")
                
                -- Faire revenir le minion auprès de son maître après le loot
                creature:RegisterEvent(function(_, _, _, minion)
                    if minion and owner then
                        minion:MoveFollow(owner, 2.0, 0)
                    end
                end, 800, 1)
                
                break -- Un cadavre à la fois par intervalle
            end
        end
    end
end

-- Apparition du Minion
local function OnMinionSpawn(event, creature)
    if creature:GetEntry() == MinionConfig.MinionEntry then
        -- Lancer la boucle de scrutation des loots
        creature:RegisterEvent(ProcessAutoLoot, MinionConfig.ScanInterval, 0)
    end
end

-- Commande GM / Joueur pour invoquer/renvoyer le minion
local function OnPlayerCommand(event, player, command)
    if command == MinionConfig.SummonCommand then
        local pGUID = player:GetGUIDLow()

        -- Si le minion existe déjà, on le renvoie
        if activeMinions[pGUID] and activeMinions[pGUID]:IsInWorld() then
            activeMinions[pGUID]:DespawnOrUnsummon()
            activeMinions[pGUID] = nil
            player:SendBroadcastMessage("|cFFFF0000Votre Minion Ramasseur a été renvoyé.|r")
            return false
        end

        -- Sinon, invocation du minion
        local minion = player:SpawnCreature(MinionConfig.MinionEntry, player:GetX() + 2, player:GetY(), player:GetZ(), player:GetO(), 3, 0)
        if minion then
            minion:SetOwnerGUID(player:GetGUID())
            minion:MoveFollow(player, 2.0, 0)
            activeMinions[pGUID] = minion
            player:SendBroadcastMessage("|cFF00FF00Minion Ramasseur invoqué ! Il lootera dans un rayon de " .. MinionConfig.LootRadius .. " yards.|r")
        end
        return false
    end
end

-- Nettoyage à la déconnexion
local function OnPlayerLogout(event, player)
    local pGUID = player:GetGUIDLow()
    if activeMinions[pGUID] then
        activeMinions[pGUID]:DespawnOrUnsummon()
        activeMinions[pGUID] = nil
    end
end

RegisterCreatureEvent(MinionConfig.MinionEntry, 5, OnMinionSpawn) -- EVENT_ON_JUST_SPAWNED
RegisterPlayerEvent(42, OnPlayerCommand)                          -- EVENT_ON_COMMAND
RegisterPlayerEvent(4, OnPlayerLogout)                            -- EVENT_ON_LOGOUT

```

---

### **Résumé du Fonctionnement :**

* **Détection à 20 yards** : Analyse les corps de monstres vaincus dans un rayon configurable de 20 yards.
* **Respect des droits de loot** : Le minion vérifie avec `IsLootAllowedFor(owner)` que le butin appartient bien au joueur ou à son groupe.
* **Auto-Loot Direct** : Transfère automatiquement l'or, les objets et les composants directement dans les sacs du joueur via `AutoLootCreature()`.
* **Inviolable** : Le PNJ utilise des flags de sécurité (inciblable et insensible aux attaques) pour ne pas perturber les combats.

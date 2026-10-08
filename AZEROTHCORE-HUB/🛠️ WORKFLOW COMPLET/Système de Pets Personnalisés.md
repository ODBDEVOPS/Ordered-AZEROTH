Excellente idée ! Créons un système complet de pets personnalisés type démoniste/chasseur. Voici le workflow détaillé :

## 🐾 **WORKFLOW COMPLET : Système de Pets Personnalisés**

### **ÉTAPE 1 : Comprendre le Système de Pets**

Les pets dans WoW utilisent :
- **CreatureProto** : Définition de base de la créature
- **PetSpell** : Sorts et compétences du pet
- **PetLevelStats** : Statistiques par niveau
- **PetAura** : Auras passives
- **PetCreateInfo** : Configuration de création

### **ÉTAPE 2 : Créer les Tables de Base**

```sql
-- Table des types de pets personnalisés
CREATE TABLE IF NOT EXISTS `custom_pet_types` (
    `pet_type_id` INT AUTO_INCREMENT PRIMARY KEY,
    `pet_name` VARCHAR(100) NOT NULL,
    `creature_entry` INT NOT NULL,
    `display_id` INT NOT NULL,
    `family` INT DEFAULT 0,
    `type` INT DEFAULT 0, -- 0 = démon, 1 = bête, 2 = mort-vivant, 3 = élémentaire
    `scale` FLOAT DEFAULT 1.0,
    `mountable` TINYINT DEFAULT 0,
    `flying` TINYINT DEFAULT 0,
    `description` TEXT,
    UNIQUE KEY `idx_creature_entry` (`creature_entry`)
);

-- Table des sorts de pets
CREATE TABLE IF NOT EXISTS `custom_pet_spells` (
    `id` INT AUTO_INCREMENT PRIMARY KEY,
    `pet_type_id` INT NOT NULL,
    `spell_id` INT NOT NULL,
    `slot` INT DEFAULT 0,
    `level_required` INT DEFAULT 1,
    `autocast` TINYINT DEFAULT 0,
    `cooldown` INT DEFAULT 0,
    `mana_cost` INT DEFAULT 0,
    FOREIGN KEY (`pet_type_id`) REFERENCES `custom_pet_types`(`pet_type_id`)
);

-- Table des statistiques par niveau
CREATE TABLE IF NOT EXISTS `custom_pet_level_stats` (
    `pet_type_id` INT NOT NULL,
    `level` INT NOT NULL,
    `hp` INT NOT NULL,
    `mana` INT NOT NULL,
    `armor` INT NOT NULL,
    `strength` INT NOT NULL,
    `agility` INT NOT NULL,
    `stamina` INT NOT NULL,
    `intellect` INT NOT NULL,
    `spirit` INT NOT NULL,
    `min_damage` FLOAT NOT NULL,
    `max_damage` FLOAT NOT NULL,
    PRIMARY KEY (`pet_type_id`, `level`)
);

-- Table des pets des joueurs
CREATE TABLE IF NOT EXISTS `custom_player_pets` (
    `id` INT AUTO_INCREMENT PRIMARY KEY,
    `player_guid` INT NOT NULL,
    `pet_type_id` INT NOT NULL,
    `pet_guid` INT DEFAULT 0,
    `slot` INT DEFAULT 0,
    `name` VARCHAR(100),
    `level` INT DEFAULT 1,
    `experience` INT DEFAULT 0,
    `loyalty` INT DEFAULT 0,
    `happiness` INT DEFAULT 100,
    `is_active` TINYINT DEFAULT 0,
    `is_summoned` TINYINT DEFAULT 0,
    FOREIGN KEY (`pet_type_id`) REFERENCES `custom_pet_types`(`pet_type_id`)
);
```

### **ÉTAPE 3 : Insérer les Pets de Départ**

```sql
-- Pets de type démoniste
INSERT INTO `custom_pet_types` VALUES
(1, 'Diablotin', 416, 4449, 23, 0, 0.8, 0, 0, 'Pet de base démoniste'),
(2, 'Marcheur du Vide', 1860, 1132, 16, 0, 1.2, 0, 0, 'Tank démoniste'),
(3, 'Succube', 1863, 10923, 17, 0, 1.0, 0, 0, 'Contrôle démoniste'),
(4, 'Gangrchien', 417, 850, 15, 0, 1.0, 0, 0, 'DPS démoniste'),
(5, 'Gangregarde', 17252, 16631, 29, 0, 1.5, 0, 0, 'Pet ultime démoniste');

-- Pets de type chasseur
INSERT INTO `custom_pet_types` VALUES
(6, 'Loup Gris', 382, 780, 1, 1, 1.0, 1, 0, 'Pet de base chasseur'),
(7, 'Ours Brun', 1196, 982, 4, 1, 1.3, 1, 0, 'Tank chasseur'),
(8, 'Tigre Blanc', 3129, 3201, 2, 1, 1.0, 1, 0, 'DPS chasseur'),
(9, 'Aigle Royal', 1961, 1098, 26, 1, 1.0, 0, 1, 'Pet volant chasseur'),
(10, 'Raptor Rapide', 3254, 788, 11, 1, 1.1, 1, 0, 'Pet rapide chasseur');

-- Sorts des pets démonistes
INSERT INTO `custom_pet_spells` VALUES
(1, 1, 3110, 1, 1, 1, 0, 15),  -- Boule de feu
(2, 1, 3716, 2, 10, 1, 0, 20),  -- Bouclier de feu
(3, 2, 3716, 1, 1, 1, 0, 0),    -- Tourment
(4, 2, 17735, 2, 20, 1, 30, 0), -- Consommation
(5, 3, 7812, 1, 1, 1, 0, 25),   -- Fouet
(6, 3, 6358, 2, 15, 1, 0, 30),  -- Séduction
(7, 4, 17253, 1, 1, 1, 0, 20),  -- Morsure
(8, 4, 17255, 2, 10, 1, 15, 15),-- Grondement
(9, 5, 19483, 1, 1, 1, 0, 50),  -- Immolation
(10, 5, 19491, 2, 30, 1, 60, 0);-- Pacte noir

-- Sorts des pets chasseurs
INSERT INTO `custom_pet_spells` VALUES
(11, 6, 17253, 1, 1, 1, 0, 15),  -- Morsure
(12, 6, 17255, 2, 5, 1, 10, 10), -- Grondement
(13, 7, 17264, 1, 1, 1, 0, 20),  -- Coup de patte
(14, 7, 16827, 2, 10, 1, 20, 0), -- Charge
(15, 8, 17253, 1, 1, 1, 0, 15),  -- Morsure
(16, 8, 24450, 2, 15, 1, 0, 25), -- Grâce féline
(17, 9, 17253, 1, 1, 1, 0, 15),  -- Morsure
(18, 9, 19596, 2, 20, 1, 30, 0), -- Plongeon
(19, 10, 17253, 1, 1, 1, 0, 15), -- Morsure
(20, 10, 19596, 2, 10, 1, 0, 20); -- Course
```

### **ÉTAPE 4 : Script Lua Principal - Gestion des Pets**

```lua
-- pet_system.lua
local PetSystem = {}

-- Configuration
PetSystem.pets = {}
PetSystem.activePets = {}
PetSystem.petSpells = {}

-- Chargement des pets depuis la DB
function PetSystem:LoadPets()
    local query = WorldDBQuery("SELECT pet_type_id, pet_name, creature_entry, display_id, family, type, scale, mountable, flying FROM custom_pet_types")
    if query then
        self.pets = {}
        repeat
            local row = query:GetRow()
            self.pets[tonumber(row[1])] = {
                id = tonumber(row[1]),
                name = row[2],
                creatureEntry = tonumber(row[3]),
                displayId = tonumber(row[4]),
                family = tonumber(row[5]),
                type = tonumber(row[6]),
                scale = tonumber(row[7]) or 1.0,
                mountable = tonumber(row[8]) == 1,
                flying = tonumber(row[9]) == 1
            }
        until not query:NextRow()
        return true
    end
    return false
end

-- Chargement des sorts de pets
function PetSystem:LoadPetSpells()
    local query = WorldDBQuery("SELECT pet_type_id, spell_id, slot, level_required, autocast, cooldown, mana_cost FROM custom_pet_spells")
    if query then
        self.petSpells = {}
        repeat
            local row = query:GetRow()
            local petTypeId = tonumber(row[1])
            if not self.petSpells[petTypeId] then
                self.petSpells[petTypeId] = {}
            end
            table.insert(self.petSpells[petTypeId], {
                spellId = tonumber(row[2]),
                slot = tonumber(row[3]),
                levelRequired = tonumber(row[4]),
                autocast = tonumber(row[5]) == 1,
                cooldown = tonumber(row[6]),
                manaCost = tonumber(row[7])
            })
        until not query:NextRow()
        return true
    end
    return false
end

-- Créer un pet pour un joueur
function PetSystem:CreatePet(player, petTypeId, slot)
    local petType = self.pets[petTypeId]
    if not petType then
        player:SendBroadcastMessage("|cFFFF0000Type de pet invalide !|r")
        return nil
    end
    
    -- Vérifier si le joueur a déjà un pet actif
    if self.activePets[player:GetGUIDLow()] then
        player:SendBroadcastMessage("|cFFFF0000Vous avez déjà un pet actif !|r")
        return nil
    end
    
    -- Vérifier les restrictions de classe
    local playerClass = player:GetClass()
    if petType.type == 0 and playerClass ~= 9 then -- Démoniste
        player:SendBroadcastMessage("|cFFFF0000Seuls les démonistes peuvent invoquer ce pet !|r")
        return nil
    elseif petType.type == 1 and playerClass ~= 3 then -- Chasseur
        player:SendBroadcastMessage("|cFFFF0000Seuls les chasseurs peuvent apprivoiser ce pet !|r")
        return nil
    end
    
    -- Créer la créature
    local pet = player:SpawnCreature(
        petType.creatureEntry,
        player:GetX() + 2,
        player:GetY(),
        player:GetZ(),
        player:GetO(),
        1, -- TempSummonType
        0  -- Despawn time (0 = permanent)
    )
    
    if not pet then
        player:SendBroadcastMessage("|cFFFF0000Erreur lors de la création du pet !|r")
        return nil
    end
    
    -- Configurer le pet
    pet:SetDisplayId(petType.displayId)
    pet:SetObjectScale(petType.scale)
    pet:SetFaction(player:GetFaction())
    pet:SetLevel(player:GetLevel())
    pet:SetCreatorGUID(player:GetGUID())
    pet:SetOwnerGUID(player:GetGUID())
    pet:SetPetNumber(WorldDBQuery("SELECT MAX(id) FROM custom_player_pets"):GetUInt32(0) + 1)
    
    -- Définir les statistiques
    self:SetPetStats(pet, petType, player:GetLevel())
    
    -- Appliquer les sorts
    self:ApplyPetSpells(pet, petType.id, player:GetLevel())
    
    -- Sauvegarder dans la DB
    local petGUID = pet:GetGUIDLow()
    WorldDBExecute(string.format(
        "INSERT INTO custom_player_pets (player_guid, pet_type_id, pet_guid, slot, name, level, is_active, is_summoned) VALUES (%d, %d, %d, %d, '%s', %d, 1, 1)",
        player:GetGUIDLow(), petTypeId, petGUID, slot or 0,
        petType.name, player:GetLevel()
    ))
    
    -- Enregistrer en mémoire
    self.activePets[player:GetGUIDLow()] = {
        pet = pet,
        petType = petType,
        slot = slot or 0,
        petGUID = petGUID
    }
    
    -- Effets visuels
    player:SendPlaySpellVisual(6372)
    pet:SendPlaySpellVisual(6372)
    
    player:SendBroadcastMessage(string.format("|cFF00FF00%s vous accompagne maintenant !|r", petType.name))
    
    return pet
end

-- Définir les statistiques du pet
function PetSystem:SetPetStats(pet, petType, level)
    local query = WorldDBQuery(string.format(
        "SELECT hp, mana, armor, strength, agility, stamina, intellect, spirit, min_damage, max_damage FROM custom_pet_level_stats WHERE pet_type_id = %d AND level = %d",
        petType.id, level
    ))
    
    if query then
        local row = query:GetRow()
        pet:SetMaxHealth(tonumber(row[1]))
        pet:SetHealth(tonumber(row[1]))
        pet:SetMaxPower(POWER_MANA, tonumber(row[2]))
        pet:SetPower(POWER_MANA, tonumber(row[2]))
        pet:SetArmor(tonumber(row[3]))
        pet:SetStat(0, tonumber(row[4])) -- Strength
        pet:SetStat(1, tonumber(row[5])) -- Agility
        pet:SetStat(2, tonumber(row[6])) -- Stamina
        pet:SetStat(3, tonumber(row[7])) -- Intellect
        pet:SetStat(4, tonumber(row[8])) -- Spirit
        pet:SetMinDamage(tonumber(row[9]))
        pet:SetMaxDamage(tonumber(row[10]))
    else
        -- Statistiques par défaut
        pet:SetMaxHealth(level * 50)
        pet:SetHealth(level * 50)
        pet:SetMaxPower(POWER_MANA, level * 30)
        pet:SetPower(POWER_MANA, level * 30)
        pet:SetArmor(level * 20)
        pet:SetStat(0, level * 3)
        pet:SetStat(1, level * 2)
        pet:SetStat(2, level * 4)
        pet:SetStat(3, level * 2)
        pet:SetStat(4, level * 1)
        pet:SetMinDamage(level * 2)
        pet:SetMaxDamage(level * 3)
    end
end

-- Appliquer les sorts au pet
function PetSystem:ApplyPetSpells(pet, petTypeId, level)
    local spells = self.petSpells[petTypeId]
    if not spells then
        return
    end
    
    for _, spell in ipairs(spells) do
        if level >= spell.levelRequired then
            pet:AddSpell(spell.spellId, spell.slot, spell.autocast, spell.cooldown, spell.manaCost)
        end
    end
end

-- Retirer le pet
function PetSystem:RemovePet(player, saveToDB)
    local playerGUID = player:GetGUIDLow()
    local activePet = self.activePets[playerGUID]
    
    if not activePet then
        return false
    end
    
    -- Désinvoquer le pet
    if activePet.pet and not activePet.pet:IsDead() then
        activePet.pet:DespawnOrUnsummon(0)
    end
    
    -- Mettre à jour la DB
    if saveToDB ~= false then
        WorldDBExecute(string.format(
            "UPDATE custom_player_pets SET is_summoned = 0 WHERE player_guid = %d AND pet_guid = %d",
            playerGUID, activePet.petGUID
        ))
    end
    
    -- Nettoyer
    self.activePets[playerGUID] = nil
    
    player:SendBroadcastMessage("|cFFFFFF00Votre pet a été renvoyé.|r")
    
    return true
end
```

### **ÉTAPE 5 : Système de Contrôle des Pets**

```lua
-- pet_control.lua
local PetControl = {}

-- Commandes de contrôle
PetControl.commands = {
    attack = "Attaque",
    follow = "Suit",
    stay = "Reste",
    aggressive = "Agressif",
    defensive = "Défensif",
    passive = "Passif",
    dismiss = "Renvoyer"
}

function PetControl:HandleCommand(event, player, command)
    local args = {}
    for word in command:gmatch("%S+") do
        table.insert(args, word)
    end
    
    if args[1] == "pet" or args[1] == "familier" then
        local action = args[2]
        local target = args[3]
        
        local activePet = PetSystem.activePets[player:GetGUIDLow()]
        if not activePet or not activePet.pet then
            player:SendBroadcastMessage("|cFFFF0000Vous n'avez pas de pet actif !|r")
            return false
        end
        
        local pet = activePet.pet
        
        if action == "attack" or action == "attaque" then
            local targetUnit = target and player:GetSelection() or player:GetSelection()
            if targetUnit and targetUnit:IsHostileTo(player) then
                pet:Attack(targetUnit)
                player:SendBroadcastMessage("|cFF00FF00Votre pet attaque !|r")
            else
                player:SendBroadcastMessage("|cFFFF0000Sélectionnez une cible valide !|r")
            end
        elseif action == "follow" or action == "suit" then
            pet:SetFollow()
            player:SendBroadcastMessage("|cFF00FF00Votre pet vous suit.|r")
        elseif action == "stay" or action == "reste" then
            pet:SetStay(true)
            player:SendBroadcastMessage("|cFF00FF00Votre pet reste sur place.|r")
        elseif action == "aggressive" or action == "agressif" then
            pet:SetReactState(1) -- REACT_AGGRESSIVE
            player:SendBroadcastMessage("|cFF00FF00Votre pet est agressif.|r")
        elseif action == "defensive" or action == "défensif" then
            pet:SetReactState(2) -- REACT_DEFENSIVE
            player:SendBroadcastMessage("|cFF00FF00Votre pet est défensif.|r")
        elseif action == "passive" or action == "passif" then
            pet:SetReactState(0) -- REACT_PASSIVE
            player:SendBroadcastMessage("|cFF00FF00Votre pet est passif.|r")
        elseif action == "dismiss" or action == "renvoyer" then
            PetSystem:RemovePet(player, true)
        elseif action == "spells" or action == "sorts" then
            self:ListPetSpells(player, pet)
        elseif action == "cast" or action == "lance" then
            local spellId = tonumber(args[3])
            if spellId then
                pet:CastSpell(pet, spellId, true)
            end
        end
        
        return false
    end
end

function PetControl:ListPetSpells(player, pet)
    player:SendBroadcastMessage("|cFF00FF00=== Sorts du Pet ===|r")
    local spells = pet:GetSpellMap()
    for spellId, _ in pairs(spells) do
        local spellName = GetSpellInfo(spellId)
        player:SendBroadcastMessage(string.format("- [%d] %s", spellId, spellName or "Inconnu"))
    end
end

-- Enregistrement des commandes
RegisterPlayerEvent(42, function(event, player, command)
    return PetControl:HandleCommand(event, player, command)
end)
```

### **ÉTAPE 6 : Système de Capture pour Chasseur**

```lua
-- pet_taming.lua
local PetTaming = {}

PetTaming.tamableCreatures = {}
PetTaming.tamingSpells = {
    1515, -- Capture de bête
    6991, -- Nourrir le pet
    982,  -- Revivre le pet
}

function PetTaming:LoadTamableCreatures()
    local query = WorldDBQuery("SELECT entry, name, family, min_level, max_level FROM creature_template WHERE type = 1 AND family > 0 LIMIT 100")
    if query then
        self.tamableCreatures = {}
        repeat
            local row = query:GetRow()
            self.tamableCreatures[tonumber(row[1])] = {
                entry = tonumber(row[1]),
                name = row[2],
                family = tonumber(row[3]),
                minLevel = tonumber(row[4]),
                maxLevel = tonumber(row[5])
            }
        until not query:NextRow()
        return true
    end
    return false
end

function PetTaming:StartTaming(event, player, spell, skipCheck)
    if spell:GetEntry() == 1515 then -- Capture de bête
        local target = player:GetSelection()
        if not target or not target:IsCreature() then
            player:SendBroadcastMessage("|cFFFF0000Sélectionnez une bête à apprivoiser !|r")
            return
        end
        
        local creatureEntry = target:GetEntry()
        local tamableInfo = self.tamableCreatures[creatureEntry]
        
        if not tamableInfo then
            player:SendBroadcastMessage("|cFFFF0000Cette créature ne peut pas être apprivoisée !|r")
            return
        end
        
        if target:GetLevel() > player:GetLevel() then
            player:SendBroadcastMessage("|cFFFF0000Cette bête est trop puissante pour vous !|r")
            return
        end
        
        -- Vérifier si le joueur a déjà un pet
        if PetSystem.activePets[player:GetGUIDLow()] then
            player:SendBroadcastMessage("|cFFFF0000Vous avez déjà un pet actif !|r")
            return
        end
        
        -- Démarrer le processus d'apprivoisement
        player:SendBroadcastMessage("|cFFFFFF00Apprivoisement en cours...|r")
        player:SendPlaySpellVisual(6372)
        
        -- Simuler le temps d'apprivoisement
        player:RegisterEvent(function()
            if target and not target:IsDead() then
                -- Créer le pet basé sur la créature
                local petTypeId = self:CreatePetFromCreature(player, creatureEntry)
                if petTypeId then
                    PetSystem:CreatePet(player, petTypeId, 1)
                    target:DespawnOrUnsummon(0)
                end
            end
        end, 5000, 1) -- 5 secondes d'apprivoisement
    end
end

function PetTaming:CreatePetFromCreature(player, creatureEntry)
    local creatureInfo = self.tamableCreatures[creatureEntry]
    if not creatureInfo then
        return nil
    end
    
    -- Créer une entrée dans la table des pets
    local petName = creatureInfo.name
    local displayId = WorldDBQuery(string.format("SELECT modelid1 FROM creature_template WHERE entry = %d", creatureEntry)):GetUInt32(0)
    
    -- Insérer dans la DB
    WorldDBExecute(string.format(
        "INSERT INTO custom_pet_types (pet_name, creature_entry, display_id, family, type, scale, mountable, flying) VALUES ('%s', %d, %d, %d, 1, 1.0, 0, 0)",
        petName, creatureEntry, displayId, creatureInfo.family
    ))
    
    -- Récupérer l'ID
    local petTypeId = WorldDBQuery("SELECT LAST_INSERT_ID()"):GetUInt32(0)
    
    -- Ajouter les sorts de base
    WorldDBExecute(string.format(
        "INSERT INTO custom_pet_spells (pet_type_id, spell_id, slot, level_required, autocast, cooldown, mana_cost) VALUES (%d, 17253, 1, 1, 1, 0, 15), (%d, 17255, 2, 10, 1, 10, 10)",
        petTypeId, petTypeId
    ))
    
    -- Recharger les pets
    PetSystem:LoadPets()
    PetSystem:LoadPetSpells()
    
    return petTypeId
end

-- Enregistrement des événements
RegisterPlayerEvent(5, function(event, player, spell, skipCheck)
    PetTaming:StartTaming(event, player, spell, skipCheck)
end)

PetTaming:LoadTamableCreatures()
```

### **ÉTAPE 7 : Système de Nourriture et Loyauté**

```lua
-- pet_care.lua
local PetCare = {}

-- Nourriture pour les pets
PetCare.foods = {
    [1113] = { happiness = 35, name = "Pain croûté" },
    [4540] = { happiness = 25, name = "Pain de seigle" },
    [4604] = { happiness = 20, name = "Champignon" },
    [4605] = { happiness = 15, name = "Poisson cru" },
    [4606] = { happiness = 30, name = "Fruit" },
    [4607] = { happiness = 40, name = "Viande cuite" },
}

function PetCare:FeedPet(event, player, item, target)
    local activePet = PetSystem.activePets[player:GetGUIDLow()]
    if not activePet or not activePet.pet then
        return
    end
    
    local foodInfo = self.foods[item:GetEntry()]
    if not foodInfo then
        return
    end
    
    local pet = activePet.pet
    
    -- Vérifier si le pet est à proximité
    local distance = player:GetDistance(pet)
    if distance > 10 then
        player:SendBroadcastMessage("|cFFFF0000Votre pet est trop loin !|r")
        return
    end
    
    -- Améliorer le bonheur
    local currentHappiness = activePet.happiness or 100
    local newHappiness = math.min(100, currentHappiness + foodInfo.happiness)
    activePet.happiness = newHappiness
    
    -- Consommer l'item
    player:RemoveItem(item:GetEntry(), 1)
    
    -- Effets
    pet:SendPlaySpellVisual(6372)
    player:SendBroadcastMessage(string.format("|cFF00FF00%s mange %s. Bonheur : %d%%|r", pet:GetName(), foodInfo.name, newHappiness))
    
    -- Mettre à jour la loyauté
    self:UpdateLoyalty(player, activePet)
end

function PetCare:UpdateLoyalty(player, activePet)
    local happiness = activePet.happiness or 100
    local loyalty = activePet.loyalty or 0
    
    if happiness >= 90 and loyalty < 6 then
        loyalty = loyalty + 0.1
        activePet.loyalty = loyalty
    elseif happiness < 30 and loyalty > 0 then
        loyalty = loyalty - 0.1
        activePet.loyalty = loyalty
    end
    
    -- Mettre à jour dans la DB
    WorldDBExecute(string.format(
        "UPDATE custom_player_pets SET happiness = %d, loyalty = %d WHERE pet_guid = %d",
        happiness, loyalty, activePet.petGUID
    ))
end

-- Enregistrement de l'événement d'utilisation d'item
RegisterPlayerEvent(18, function(event, player, item, target)
    PetCare:FeedPet(event, player, item, target)
end)
```

### **ÉTAPE 8 : Interface et Commandes GM**

```lua
-- pet_commands.lua
local PetCommands = {}

function PetCommands:HandleGMCommand(event, player, command)
    local args = {}
    for word in command:gmatch("%S+") do
        table.insert(args, word)
    end
    
    if args[1] == "pets" or args[1] == "listepets" then
        player:SendBroadcastMessage("|cFF00FF00=== Pets Disponibles ===|r")
        for id, petType in pairs(PetSystem.pets) do
            player:SendBroadcastMessage(string.format("%d - %s (Type: %d, Famille: %d)", 
                id, petType.name, petType.type, petType.family))
        end
        return false
    elseif args[1] == "createpet" or args[1] == "creerpet" then
        local petTypeId = tonumber(args[2])
        if petTypeId then
            PetSystem:CreatePet(player, petTypeId, tonumber(args[3]) or 1)
        else
            player:SendBroadcastMessage("|cFFFF0000Usage : .createpet [type_id] [slot]|r")
        end
        return false
    elseif args[1] == "removepet" or args[1] == "retirerpet" then
        PetSystem:RemovePet(player, true)
        return false
    elseif args[1] == "petstats" or args[1] == "statspet" then
        local activePet = PetSystem.activePets[player:GetGUIDLow()]
        if activePet and activePet.pet then
            local pet = activePet.pet
            player:SendBroadcastMessage("|cFF00FF00=== Stats du Pet ===|r")
            player:SendBroadcastMessage(string.format("Nom : %s", pet:GetName()))
            player:SendBroadcastMessage(string.format("Niveau : %d", pet:GetLevel()))
            player:SendBroadcastMessage(string.format("HP : %d/%d", pet:GetHealth(), pet:GetMaxHealth()))
            player:SendBroadcastMessage(string.format("Mana : %d/%d", pet:GetPower(POWER_MANA), pet:GetMaxPower(POWER_MANA)))
            player:SendBroadcastMessage(string.format("Dégâts : %d-%d", pet:GetMinDamage(), pet:GetMaxDamage()))
        end
        return false
    end
end

-- Enregistrement des commandes
RegisterPlayerEvent(42, function(event, player, command)
    return PetCommands:HandleGMCommand(event, player, command)
end)
```

### **ÉTAPE 9 : Script d'Initialisation**

```lua
-- pet_init.lua
print("=== Initialisation du Système de Pets ===")

-- Charger les données
PetSystem:LoadPets()
PetSystem:LoadPetSpells()
PetTaming:LoadTamableCreatures()

-- Initialiser les événements globaux
RegisterPlayerEvent(3, function(event, player) -- EVENT_ON_LOGIN
    -- Charger les pets du joueur depuis la DB
    local query = WorldDBQuery(string.format(
        "SELECT pet_type_id, slot, name, level, is_active FROM custom_player_pets WHERE player_guid = %d AND is_active = 1",
        player:GetGUIDLow()
    ))
    
    if query then
        repeat
            local row = query:GetRow()
            local petTypeId = tonumber(row[1])
            local slot = tonumber(row[2])
            
            -- Invoquer le pet si nécessaire
            if PetSystem.pets[petTypeId] then
                PetSystem:CreatePet(player, petTypeId, slot)
            end
        until not query:NextRow()
    end
end)

RegisterPlayerEvent(4, function(event, player) -- EVENT_ON_LOGOUT
    -- Retirer les pets
    PetSystem:RemovePet(player, true)
end)

RegisterPlayerEvent(33, function(event, player) -- EVENT_ON_PLAYER_DEATH
    -- Retirer les pets à la mort
    PetSystem:RemovePet(player, false)
end)

print("=== Système de Pets Initialisé ===")
```

### **Résumé du Workflow :**

1. **Étape 1** : Comprendre le système de pets
2. **Étape 2** : Créer les tables de base
3. **Étape 3** : Insérer les pets de départ
4. **Étape 4** : Créer le script principal
5. **Étape 5** : Implémenter le contrôle des pets
6. **Étape 6** : Ajouter la capture pour chasseur
7. **Étape 7** : Gérer la nourriture et loyauté
8. **Étape 8** : Créer l'interface et commandes
9. **Étape 9** : Script d'initialisation

### **Fonctionnalités Supplémentaires :**

- **Montures Pets** : Certains pets peuvent être montés
- **Pets de Combat** : Avec IA personnalisée
- **Pets de Collection** : Système de collection
- **Évolution** : Les pets évoluent avec le niveau
- **Compétences** : Apprentissage de nouvelles compétences

Ce système complet vous permet de créer des pets type démoniste/chasseur pour toutes les classes ! Voulez-vous que je détaille un aspect particulier ?

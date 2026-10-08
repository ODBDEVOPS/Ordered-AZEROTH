Excellente idée ! Créons une classe personnalisée complète avec ses sorts. Voici le workflow détaillé :

## ⚔️ **WORKFLOW COMPLET : Création d'une Nouvelle Classe Personnalisée**

### **ÉTAPE 1 : Comprendre le Système de Classes**

Une classe dans WoW nécessite :
- **ChrClasses.dbc** : Définition de base de la classe
- **SkillLineAbility** : Compétences de la classe
- **Spells** : Sorts spécifiques
- **Talents** : Arbre de talents
- **Quêtes de classe** : Quêtes spécifiques

### **ÉTAPE 2 : Créer la Table de Classe Personnalisée**

```sql
-- Table de la classe personnalisée
CREATE TABLE IF NOT EXISTS `custom_classes` (
    `class_id` INT PRIMARY KEY,
    `class_name` VARCHAR(100) NOT NULL,
    `class_name_fr` VARCHAR(100) NOT NULL,
    `class_description` TEXT,
    `class_icon` INT DEFAULT 0,
    `class_color` VARCHAR(7) DEFAULT '#FFFFFF',
    `primary_stat` VARCHAR(20) DEFAULT 'strength',
    `armor_type` INT DEFAULT 1, -- 1 = Tissu, 2 = Cuir, 3 = Mailles, 4 = Plaques
    `weapon_skills` TEXT, -- Liste des compétences d'armes
    `starting_level` INT DEFAULT 1,
    `starting_zone` INT DEFAULT 0,
    `starting_gold` INT DEFAULT 0,
    `health_per_level` FLOAT DEFAULT 10.0,
    `mana_per_level` FLOAT DEFAULT 5.0,
    UNIQUE KEY `idx_class_id` (`class_id`)
);

-- Table des sorts de la classe
CREATE TABLE IF NOT EXISTS `custom_class_spells` (
    `id` INT AUTO_INCREMENT PRIMARY KEY,
    `class_id` INT NOT NULL,
    `spell_id` INT NOT NULL,
    `spell_level` INT NOT NULL,
    `spell_rank` INT DEFAULT 1,
    `spell_cost` INT DEFAULT 0,
    `spell_type` VARCHAR(50) DEFAULT 'active', -- active, passive, talent
    `spell_tree` VARCHAR(50) DEFAULT 'general', -- general, specialization_1, specialization_2, specialization_3
    `requires_specialization` INT DEFAULT 0,
    `parent_spell_id` INT DEFAULT 0,
    `description` TEXT,
    FOREIGN KEY (`class_id`) REFERENCES `custom_classes`(`class_id`)
);

-- Table des compétences de la classe
CREATE TABLE IF NOT EXISTS `custom_class_skills` (
    `id` INT AUTO_INCREMENT PRIMARY KEY,
    `class_id` INT NOT NULL,
    `skill_id` INT NOT NULL,
    `skill_name` VARCHAR(100) NOT NULL,
    `max_rank` INT DEFAULT 300,
    `start_rank` INT DEFAULT 1,
    FOREIGN KEY (`class_id`) REFERENCES `custom_classes`(`class_id`)
);

-- Table des talents de la classe
CREATE TABLE IF NOT EXISTS `custom_class_talents` (
    `id` INT AUTO_INCREMENT PRIMARY KEY,
    `class_id` INT NOT NULL,
    `talent_id` INT NOT NULL,
    `talent_tree` INT NOT NULL, -- 1, 2, 3 pour les trois spécialisations
    `talent_row` INT NOT NULL,
    `talent_column` INT NOT NULL,
    `spell_id` INT NOT NULL,
    `requires_points` INT DEFAULT 0,
    `requires_talent_id` INT DEFAULT 0,
    `max_rank` INT DEFAULT 5,
    FOREIGN KEY (`class_id`) REFERENCES `custom_classes`(`class_id`)
);
```

### **ÉTAPE 3 : Créer une Classe Exemple - "Lame Spectrale"**

```sql
-- Insertion de la classe "Lame Spectrale"
INSERT INTO `custom_classes` VALUES
(15, 'Spectral Blade', 'Lame Spectrale', 
 'Maître des lames spectrales, capable de manipuler les énergies spirituelles',
 132350, '#8B00FF', 'agility', 2, '1,2,3,4,5,6,7,8,9,10',
 1, 0, 100, 12.0, 8.0);

-- Compétences de base
INSERT INTO `custom_class_skills` VALUES
(1, 15, 162, 'Armes d''hast', 300, 1),
(2, 15, 44, 'Dagues', 300, 1),
(3, 15, 54, 'Masses', 300, 1),
(4, 15, 173, 'Armes de pugilat', 300, 1),
(5, 15, 43, 'Epées', 300, 1),
(6, 15, 55, 'Epées à deux mains', 300, 1),
(7, 15, 95, 'Armure de cuir', 300, 1),
(8, 15, 109, 'Armure de mailles', 300, 1);

-- Sorts de base de la Lame Spectrale
INSERT INTO `custom_class_spells` VALUES
-- Sorts généraux
(1, 15, 90030, 1, 1, 20, 'active', 'general', 0, 0, 'Frappe Spectrale - Attaque de base'),
(2, 15, 90031, 4, 1, 30, 'active', 'general', 0, 90030, 'Lame Fantôme - Attaque améliorée'),
(3, 15, 90032, 8, 1, 45, 'active', 'general', 0, 90031, 'Entaille Spirituelle - Attaque puissante'),
(4, 15, 90033, 10, 1, 50, 'active', 'general', 0, 0, 'Pas de l''Ombre - Téléportation courte'),
(5, 15, 90034, 12, 1, 60, 'active', 'general', 0, 0, 'Bouclier Spectral - Absorption'),
(6, 15, 90035, 15, 1, 0, 'passive', 'general', 0, 0, 'Maîtrise de la Lame - Passif'),
(7, 15, 90036, 20, 1, 80, 'active', 'general', 0, 0, 'Danse des Spectres - AoE'),
(8, 15, 90037, 25, 1, 100, 'active', 'general', 0, 0, 'Frappe du Néant - Dégâts massifs'),
(9, 15, 90038, 30, 1, 120, 'active', 'general', 0, 0, 'Invocation Spectrale - Pet'),
(10, 15, 90039, 40, 1, 150, 'active', 'general', 0, 0, 'Lame Temporelle - Ralentissement');

-- Spécialisation 1 : Assassinat Spectral
INSERT INTO `custom_class_spells` VALUES
(11, 15, 90040, 10, 1, 0, 'passive', 'specialization_1', 1, 0, 'Maîtrise de l''Assassinat'),
(12, 15, 90041, 20, 1, 70, 'active', 'specialization_1', 1, 0, 'Coup Mortel'),
(13, 15, 90042, 30, 1, 90, 'active', 'specialization_1', 1, 90041, 'Éviscération Spectrale'),
(14, 15, 90043, 40, 1, 110, 'active', 'specialization_1', 1, 90042, 'Assassinat Fantôme'),
(15, 15, 90044, 50, 1, 130, 'active', 'specialization_1', 1, 0, 'Marque de Mort'),
(16, 15, 90045, 60, 1, 0, 'passive', 'specialization_1', 1, 0, 'Maître Assassin');

-- Spécialisation 2 : Combat Spirituel
INSERT INTO `custom_class_spells` VALUES
(17, 15, 90050, 10, 1, 0, 'passive', 'specialization_2', 1, 0, 'Maîtrise du Combat'),
(18, 15, 90051, 20, 1, 75, 'active', 'specialization_2', 1, 0, 'Tourbillon Spectral'),
(19, 15, 90052, 30, 1, 95, 'active', 'specialization_2', 1, 90051, 'Danse des Lames'),
(20, 15, 90053, 40, 1, 115, 'active', 'specialization_2', 1, 90052, 'Tempête Spirituelle'),
(21, 15, 90054, 50, 1, 135, 'active', 'specialization_2', 1, 0, 'Avatar Spectral'),
(22, 15, 90055, 60, 1, 0, 'passive', 'specialization_2', 1, 0, 'Guerrier Éternel');

-- Spécialisation 3 : Manipulation Spirituelle
INSERT INTO `custom_class_spells` VALUES
(23, 15, 90060, 10, 1, 0, 'passive', 'specialization_3', 1, 0, 'Maîtrise Spirituelle'),
(24, 15, 90061, 20, 1, 80, 'active', 'specialization_3', 1, 0, 'Chaînes Spectrales'),
(25, 15, 90062, 30, 1, 100, 'active', 'specialization_3', 1, 90061, 'Domination Mentale'),
(26, 15, 90063, 40, 1, 120, 'active', 'specialization_3', 1, 90062, 'Contrôle Total'),
(27, 15, 90064, 50, 1, 140, 'active', 'specialization_3', 1, 0, 'Armée Spectrale'),
(28, 15, 90065, 60, 1, 0, 'passive', 'specialization_3', 1, 0, 'Seigneur Spirituel');
```

### **ÉTAPE 4 : Créer les Sorts Personnalisés**

```sql
-- Sort : Frappe Spectrale (90030)
INSERT INTO `spell_dbc` (
    `Id`, `Attributes`, `AttributesEx`, `AttributesEx2`,
    `CastingTimeIndex`, `DurationIndex`, `RangeIndex`,
    `SchoolMask`, `SpellIconID`, `ActiveIconID`,
    `SpellName`, `SpellNameFlag`,
    `Effect1`, `EffectDieSides1`, `EffectBasePoints1`,
    `EffectMiscValue1`, `EffectAura1`, `TargetA1`,
    `SpellFamilyName`, `SpellFamilyFlags1`,
    `MaxTargetLevel`, `DmgClass`, `PreventionType`
) VALUES (
    90030, -- ID
    0x00000100, -- Attributes (ABILITY)
    0x00000020, -- AttributesEx
    0x00000004, -- AttributesEx2
    1, -- Instantané
    0, -- Pas de durée
    2, -- Portée de mêlée
    1, -- Physique
    132350, 132350, -- Icônes
    'Frappe Spectrale',
    16712190,
    2, -- Effect : SCHOOL_DAMAGE
    1, 1, -- DieSides et BasePoints
    0, 0, 1 -- Pas d'aura, Target : ENNEMI
    , 0, 0, 80, 0, 0
);

-- Créer les sorts pour chaque niveau
DELIMITER $$
CREATE PROCEDURE CreateSpectralSpells()
BEGIN
    DECLARE i INT DEFAULT 1;
    DECLARE spell_id INT;
    DECLARE spell_name VARCHAR(100);
    
    -- Créer les sorts de niveau 1 à 80
    WHILE i <= 80 DO
        SET spell_id = 90030 + i;
        SET spell_name = CONCAT('Frappe Spectrale (Rang ', i, ')');
        
        INSERT IGNORE INTO `spell_dbc` (
            `Id`, `Attributes`, `CastingTimeIndex`, `DurationIndex`,
            `RangeIndex`, `SchoolMask`, `SpellIconID`, `ActiveIconID`,
            `SpellName`, `SpellNameFlag`,
            `Effect1`, `EffectDieSides1`, `EffectBasePoints1`,
            `EffectMiscValue1`, `EffectAura1`, `TargetA1`,
            `MaxTargetLevel`, `DmgClass`, `PreventionType`
        ) VALUES (
            spell_id, 0x00000100, 1, 0, 2, 1,
            132350, 132350, spell_name, 16712190,
            2, i * 2, i * 3,
            0, 0, 6,
            80, 0, 0
        );
        
        SET i = i + 1;
    END WHILE;
END$$
DELIMITER ;

CALL CreateSpectralSpells();
DROP PROCEDURE CreateSpectralSpells;
```

### **ÉTAPE 5 : Script Lua Principal - Gestion de la Classe**

```lua
-- custom_class.lua
local CustomClass = {}

-- Configuration
CustomClass.classId = 15
CustomClass.className = "Lame Spectrale"
CustomClass.classColor = "|cFF8B00FF" -- Violet

-- Table des joueurs de cette classe
CustomClass.players = {}

-- Initialisation de la classe
function CustomClass:Initialize()
    print("=== Initialisation de la classe Lame Spectrale ===")
    
    -- Charger les données
    self:LoadClassData()
    
    -- Enregistrer les événements
    RegisterPlayerEvent(3, function(event, player)
        self:OnPlayerLogin(event, player)
    end)
    
    RegisterPlayerEvent(4, function(event, player)
        self:OnPlayerLogout(event, player)
    end)
    
    RegisterPlayerEvent(5, function(event, player, spell, skipCheck)
        self:OnSpellCast(event, player, spell, skipCheck)
    end)
    
    RegisterPlayerEvent(33, function(event, player)
        self:OnPlayerDeath(event, player)
    end)
    
    RegisterPlayerEvent(42, function(event, player, command)
        return self:HandleCommand(event, player, command)
    end)
end

-- Charger les données de la classe
function CustomClass:LoadClassData()
    -- Charger les sorts
    self.spells = {}
    local query = WorldDBQuery("SELECT spell_id, spell_level, spell_type, spell_tree FROM custom_class_spells WHERE class_id = 15")
    if query then
        repeat
            local row = query:GetRow()
            table.insert(self.spells, {
                spellId = tonumber(row[1]),
                level = tonumber(row[2]),
                type = row[3],
                tree = row[4]
            })
        until not query:NextRow()
    end
    
    -- Charger les compétences
    self.skills = {}
    query = WorldDBQuery("SELECT skill_id, skill_name, max_rank FROM custom_class_skills WHERE class_id = 15")
    if query then
        repeat
            local row = query:GetRow()
            self.skills[tonumber(row[1])] = {
                name = row[2],
                maxRank = tonumber(row[3])
            }
        until not query:NextRow()
    end
end

-- Vérifier si un joueur est de cette classe
function CustomClass:IsSpectralBlade(player)
    return player:GetClass() == self.classId
end

-- Gestion à la connexion
function CustomClass:OnPlayerLogin(event, player)
    if not self:IsSpectralBlade(player) then
        return
    end
    
    self.players[player:GetGUIDLow()] = {
        player = player,
        spells = {},
        specialization = 0,
        soulEnergy = 100,
        lastSoulEnergyUpdate = os.time()
    }
    
    -- Apprendre les sorts de base
    self:LearnClassSpells(player)
    
    -- Apprendre les compétences
    self:LearnClassSkills(player)
    
    -- Message de bienvenue
    player:SendBroadcastMessage(string.format("%s=== %s ===|r", self.classColor, self.className))
    player:SendBroadcastMessage(string.format("%sBienvenue, %s !|r", self.classColor, self.className))
    player:SendBroadcastMessage(string.format("%sVos pouvoirs spectraux vous attendent.|r", self.classColor))
end

-- Apprendre les sorts de la classe
function CustomClass:LearnClassSpells(player)
    local playerLevel = player:GetLevel()
    
    for _, spell in ipairs(self.spells) do
        if spell.level <= playerLevel then
            local spellId = spell.spellId
            
            -- Vérifier si le sort est déjà appris
            if not player:HasSpell(spellId) then
                player:LearnSpell(spellId)
                
                if spell.type == "active" then
                    player:SendBroadcastMessage(string.format(
                        "%sNouveau sort appris : %s|r",
                        self.classColor,
                        GetSpellInfo(spellId) or "Sort " .. spellId
                    ))
                end
            end
        end
    end
end

-- Apprendre les compétences
function CustomClass:LearnClassSkills(player)
    for skillId, skillInfo in pairs(self.skills) do
        player:SetSkill(skillId, 1, skillInfo.maxRank, skillInfo.maxRank)
    end
end

-- Gestion des sorts
function CustomClass:OnSpellCast(event, player, spell, skipCheck)
    if not self:IsSpectralBlade(player) then
        return
    end
    
    local spellId = spell:GetEntry()
    local playerData = self.players[player:GetGUIDLow()]
    
    if not playerData then
        return
    end
    
    -- Gérer les sorts spéciaux
    if spellId == 90033 then -- Pas de l'Ombre
        self:ShadowStep(player)
    elseif spellId == 90034 then -- Bouclier Spectral
        self:SpectralShield(player)
    elseif spellId == 90038 then -- Invocation Spectrale
        self:SpectralInvocation(player)
    end
end

-- Pas de l'Ombre (téléportation)
function CustomClass:ShadowStep(player)
    local target = player:GetSelection()
    if target and not target:IsFriendlyTo(player) then
        player:NearTeleport(
            target:GetX() - math.cos(target:GetO()) * 2,
            target:GetY() - math.sin(target:GetO()) * 2,
            target:GetZ(),
            target:GetO()
        )
        player:SendPlaySpellVisual(6372)
        player:SendBroadcastMessage(string.format("%sVous vous téléportez derrière votre cible !|r", self.classColor))
    else
        player:SendBroadcastMessage(string.format("%sSélectionnez une cible ennemie !|r", self.classColor))
    end
end

-- Bouclier Spectral
function CustomClass:SpectralShield(player)
    local level = player:GetLevel()
    local absorbAmount = level * 15
    
    -- Créer un bouclier d'absorption
    local aura = player:AddAura(29528, player) -- Aura de bouclier générique
    if aura then
        aura:SetStackAmount(absorbAmount)
        player:SendBroadcastMessage(string.format(
            "%sBouclier Spectral actif : absorbe %d dégâts !|r",
            self.classColor, absorbAmount
        ))
    end
end

-- Invocation Spectrale
function CustomClass:SpectralInvocation(player)
    -- Créer un pet spectral
    local pet = player:SpawnCreature(
        10184, -- Onyxia comme base (à remplacer par un modèle spectral)
        player:GetX() + 2,
        player:GetY(),
        player:GetZ(),
        player:GetO(),
        2, -- TEMPSUMMON_TIMED_OR_DEAD_DESPAWN
        60000 -- 1 minute
    )
    
    if pet then
        pet:SetDisplayId(11686) -- Display ID spectral
        pet:SetFaction(player:GetFaction())
        pet:SetLevel(player:GetLevel())
        pet:SetCreatorGUID(player:GetGUID())
        pet:SetOwnerGUID(player:GetGUID())
        
        player:SendBroadcastMessage(string.format(
            "%sSpectre invoqué pour 60 secondes !|r",
            self.classColor
        ))
    end
end

-- Gestion de la mort
function CustomClass:OnPlayerDeath(event, player)
    if not self:IsSpectralBlade(player) then
        return
    end
    
    local playerData = self.players[player:GetGUIDLow()]
    if playerData then
        -- Effet spécial de mort
        player:SendBroadcastMessage(string.format(
            "%sVotre forme spectrale se dissipe...|r",
            self.classColor
        ))
    end
end

-- Gestion de la déconnexion
function CustomClass:OnPlayerLogout(event, player)
    if self:IsSpectralBlade(player) then
        self.players[player:GetGUIDLow()] = nil
    end
end

-- Commandes GM
function CustomClass:HandleCommand(event, player, command)
    local args = {}
    for word in command:gmatch("%S+") do
        table.insert(args, word)
    end
    
    if args[1] == "setclass" and args[2] then
        local targetPlayer = player
        if args[3] then
            targetPlayer = GetPlayerByName(args[3])
        end
        
        if not targetPlayer then
            player:SendBroadcastMessage("|cFFFF0000Joueur introuvable !|r")
            return false
        end
        
        local classId = tonumber(args[2])
        if classId == self.classId then
            targetPlayer:SetClass(self.classId, true)
            self:OnPlayerLogin(nil, targetPlayer)
            player:SendBroadcastMessage(string.format(
                "%s%s est maintenant un(e) %s !|r",
                self.classColor, targetPlayer:GetName(), self.className
            ))
        end
        
        return false
    elseif args[1] == "classinfo" then
        player:SendBroadcastMessage(string.format("%s=== %s ===|r", self.classColor, self.className))
        player:SendBroadcastMessage("Maître des lames spectrales")
        player:SendBroadcastMessage("Stat principale : Agilité")
        player:SendBroadcastMessage("Types d'armure : Cuir, Mailles")
        player:SendBroadcastMessage("Spécialisations : Assassinat, Combat, Manipulation")
        return false
    end
end

-- Initialiser la classe
CustomClass:Initialize()
```

### **ÉTAPE 6 : Système de Talents**

```lua
-- talent_system.lua
local TalentSystem = {}

TalentSystem.talents = {}
TalentSystem.playerTalents = {}

function TalentSystem:LoadTalents()
    local query = WorldDBQuery("SELECT talent_id, talent_tree, talent_row, talent_column, spell_id, requires_points, requires_talent_id, max_rank FROM custom_class_talents WHERE class_id = 15")
    if query then
        self.talents = {}
        repeat
            local row = query:GetRow()
            table.insert(self.talents, {
                id = tonumber(row[1]),
                tree = tonumber(row[2]),
                row = tonumber(row[3]),
                column = tonumber(row[4]),
                spellId = tonumber(row[5]),
                requiresPoints = tonumber(row[6]),
                requiresTalentId = tonumber(row[7]),
                maxRank = tonumber(row[8])
            })
        until not query:NextRow()
    end
end

function TalentSystem:GetTalentPoints(player)
    local level = player:GetLevel()
    return math.floor(level / 10) + 1 -- 1 point tous les 10 niveaux + 1
end

function TalentSystem:LearnTalent(player, talentId, rank)
    local playerGUID = player:GetGUIDLow()
    
    if not self.playerTalents[playerGUID] then
        self.playerTalents[playerGUID] = {
            pointsSpent = 0,
            talents = {}
        }
    end
    
    local playerData = self.playerTalents[playerGUID]
    local talent = nil
    
    -- Trouver le talent
    for _, t in ipairs(self.talents) do
        if t.id == talentId then
            talent = t
            break
        end
    end
    
    if not talent then
        return false
    end
    
    -- Vérifier les points disponibles
    local totalPoints = self:GetTalentPoints(player)
    if playerData.pointsSpent >= totalPoints then
        player:SendBroadcastMessage("|cFFFF0000Plus de points de talent disponibles !|r")
        return false
    end
    
    -- Vérifier les prérequis
    if talent.requiresPoints > 0 then
        local pointsInTree = self:GetPointsInTree(playerData, talent.tree)
        if pointsInTree < talent.requiresPoints then
            player:SendBroadcastMessage(string.format(
                "|cFFFF0000Nécessite %d points dans cette branche !|r",
                talent.requiresPoints
            ))
            return false
        end
    end
    
    -- Apprendre le sort du talent
    player:LearnSpell(talent.spellId)
    
    -- Enregistrer le talent
    if not playerData.talents[talent.id] then
        playerData.talents[talent.id] = 0
    end
    
    if playerData.talents[talent.id] < talent.maxRank then
        playerData.talents[talent.id] = playerData.talents[talent.id] + 1
        playerData.pointsSpent = playerData.pointsSpent + 1
        
        player:SendBroadcastMessage(string.format(
            "|cFF00FF00Talent appris : %s (Rang %d/%d)|r",
            GetSpellInfo(talent.spellId) or "Talent " .. talent.id,
            playerData.talents[talent.id],
            talent.maxRank
        ))
        
        return true
    end
    
    return false
end

function TalentSystem:GetPointsInTree(playerData, treeId)
    local points = 0
    for talentId, rank in pairs(playerData.talents) do
        for _, talent in ipairs(self.talents) do
            if talent.id == talentId and talent.tree == treeId then
                points = points + rank
                break
            end
        end
    end
    return points
end

-- Commandes de talents
RegisterPlayerEvent(42, function(event, player, command)
    local args = {}
    for word in command:gmatch("%S+") do
        table.insert(args, word)
    end
    
    if args[1] == "talents" then
        local points = TalentSystem:GetTalentPoints(player)
        local spent = 0
        
        if TalentSystem.playerTalents[player:GetGUIDLow()] then
            spent = TalentSystem.playerTalents[player:GetGUIDLow()].pointsSpent
        end
        
        player:SendBroadcastMessage(string.format("|cFF00FF00Points de talent : %d/%d|r", spent, points))
        return false
    elseif args[1] == "learntalent" and args[2] then
        TalentSystem:LearnTalent(player, tonumber(args[2]), 1)
        return false
    end
end)

TalentSystem:LoadTalents()
```

### **ÉTAPE 7 : Système de Ressources Personnalisées**

```lua
-- resource_system.lua
local ResourceSystem = {}

ResourceSystem.soulEnergy = {}

function ResourceSystem:Initialize()
    -- Mise à jour périodique de l'énergie spirituelle
    CreateLuaEvent(function()
        self:UpdateSoulEnergy()
    end, 1000, 0) -- Toutes les secondes
end

function ResourceSystem:UpdateSoulEnergy()
    for playerGUID, data in pairs(CustomClass.players) do
        local player = data.player
        if player and player:IsInWorld() then
            -- Régénérer l'énergie spirituelle
            if not player:IsInCombat() then
                data.soulEnergy = math.min(100, data.soulEnergy + 2)
            end
            
            -- Mettre à jour l'interface
            player:SetPower(POWER_MANA, data.soulEnergy * 10) -- Utiliser le mana comme indicateur
        end
    end
end

function ResourceSystem:ConsumeSoulEnergy(player, amount)
    local playerData = CustomClass.players[player:GetGUIDLow()]
    if not playerData then
        return false
    end
    
    if playerData.soulEnergy >= amount then
        playerData.soulEnergy = playerData.soulEnergy - amount
        return true
    end
    
    return false
end

ResourceSystem:Initialize()
```

### **ÉTAPE 8 : Intégration avec la Création de Personnage**

```sql
-- Ajouter la classe à la création de personnage
INSERT INTO `playercreateinfo` (`race`, `class`, `map`, `zone`, `position_x`, `position_y`, `position_z`, `orientation`)
SELECT `race`, 15, 0, 12, 0, 0, 0, 0
FROM `playercreateinfo`
WHERE `class` = 1; -- Copier depuis les guerriers

-- Ajouter les sorts de départ
INSERT INTO `playercreateinfo_spell` (`raceMask`, `classMask`, `Spell`, `Note`)
SELECT `raceMask`, 15, `Spell`, 'Spectral Blade Starting Spell'
FROM `playercreateinfo_spell`
WHERE `classMask` = 1 AND `Spell` IN (81, 196, 201, 204, 522, 668, 2382, 2479, 3050, 3365, 6233, 6246, 6247, 6477, 6478, 6603, 7266, 7267, 7355, 8386, 9077, 9078, 9125, 20597, 20598, 20599, 20864, 21651, 21652, 22027, 22810, 45927, 49410, 52665, 60091, 62600, 642, 21156);

-- Actions de départ
INSERT INTO `playercreateinfo_action` (`race`, `class`, `button`, `action`, `type`)
SELECT `race`, 15, `button`, `action`, `type`
FROM `playercreateinfo_action`
WHERE `class` = 1;
```

### **ÉTAPE 9 : Quêtes de Classe**

```sql
-- Créer des quêtes de classe
INSERT INTO `quest_template` (`ID`, `QuestType`, `QuestLevel`, `MinLevel`, `QuestSortID`, `QuestInfoID`, `SuggestedGroupNum`, `RequiredClasses`, `RewardXP`, `RewardMoney`, `RewardSpell`, `LogTitle`, `LogDescription`, `QuestDescription`, `QuestCompletionLog`)
VALUES
(90001, 0, 1, 1, -15, 0, 0, 15, 100, 100, 90030, 
 'La Voie de la Lame Spectrale',
 'Apprenez les bases de la Lame Spectrale auprès de votre maître.',
 'Bienvenue, jeune Lame Spectrale. Votre voyage commence ici. Apprenez à manier votre lame spectrale.',
 'Vous avez appris les bases de la Lame Spectrale !');
```

### **Résumé du Workflow :**

1. **Étape 1** : Comprendre le système de classes
2. **Étape 2** : Créer les tables de classe
3. **Étape 3** : Créer une classe exemple
4. **Étape 4** : Créer les sorts personnalisés
5. **Étape 5** : Script principal de la classe
6. **Étape 6** : Système de talents
7. **Étape 7** : Ressources personnalisées
8. **Étape 8** : Intégration création de personnage
9. **Étape 9** : Quêtes de classe

### **Fonctionnalités Supplémentaires :**

- **Formes spéciales** : Transformation en spectre
- **Pets de classe** : Invocations spectrales
- **Montures de classe** : Monture spectrale unique
- **Zones de classe** : Zone d'entraînement dédiée
- **PNJ de classe** : Maîtres et formateurs

Cette classe personnalisée "Lame Spectrale" est complète avec ses sorts, talents, ressources et quêtes. Vous pouvez la modifier selon vos besoins !

Voulez-vous que je détaille un aspect particulier ou que j'ajoute d'autres fonctionnalités ?

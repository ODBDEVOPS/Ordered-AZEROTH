Excellente question ! Les formes de druide (ours, chat, voyage, etc.) sont des transformations spéciales avec des mécaniques uniques. Voici le workflow complet pour créer des formes similaires pour d'autres classes :

## 🐻 **WORKFLOW COMPLET : Création de Formes Type Druide**

### **ÉTAPE 1 : Comprendre le Système des Formes de Druide**

Les formes de druide utilisent :
- **ShapeshiftForm** : Un système distinct des auras normales
- **DisplayID** : L'apparence de la forme
- **SpellShapeshiftFormId** : Le lien entre le sort et la forme
- **Flags** : Comportements spéciaux (combat, voyage, etc.)

### **ÉTAPE 2 : Créer les Formes dans la Base de Données**

```sql
-- Table des formes personnalisées
CREATE TABLE IF NOT EXISTS `custom_shapeshift_forms` (
    `form_id` INT AUTO_INCREMENT PRIMARY KEY,
    `form_name` VARCHAR(100) NOT NULL,
    `display_id` INT NOT NULL,
    `spell_id` INT NOT NULL,
    `aura_spell_id` INT NOT NULL,
    `flags` INT DEFAULT 0,
    `combat_state` TINYINT DEFAULT 1,
    `scale` FLOAT DEFAULT 1.0,
    `sound_id` INT DEFAULT 0,
    `description` TEXT,
    UNIQUE KEY `idx_form_id` (`form_id`),
    UNIQUE KEY `idx_spell_id` (`spell_id`)
);

-- Insertion des formes de base (copie des formes druide)
INSERT INTO `custom_shapeshift_forms` VALUES
-- Forme d'ours pour guerrier
(1, 'Forme d''Ours Guerrier', 29422, 90010, 90011, 1, 1, 1.2, 0, 'Forme de combat tank'),
-- Forme de félin pour voleur
(2, 'Forme de Félin Voleur', 892, 90012, 90013, 2, 1, 1.0, 0, 'Forme de combat furtif'),
-- Forme de voyage pour chasseur
(3, 'Forme de Voyage Chasseur', 918, 90014, 90015, 4, 0, 1.5, 0, 'Forme de déplacement rapide'),
-- Forme aquatique pour chaman
(4, 'Forme Aquatique Chaman', 2428, 90016, 90017, 8, 0, 1.0, 0, 'Forme de nage'),
-- Forme de vol pour démoniste
(5, 'Forme de Vol Démoniste', 20857, 90018, 90019, 16, 0, 1.3, 0, 'Forme de vol');
```

### **ÉTAPE 3 : Créer les Sorts de Transformation**

#### **3.1 Sorts de Formes (Sorts principaux)**

```sql
-- Sort : Forme d'Ours Guerrier (90010)
INSERT INTO `spell_dbc` (
    `Id`, `Attributes`, `AttributesEx`, `AttributesEx2`,
    `CastingTimeIndex`, `DurationIndex`, `RangeIndex`,
    `SchoolMask`, `SpellIconID`, `ActiveIconID`,
    `SpellName`, `SpellNameFlag`,
    `Effect1`, `EffectDieSides1`, `EffectBasePoints1`,
    `EffectMiscValue1`, `EffectAura1`, `TargetA1`,
    `SpellFamilyName`, `SpellFamilyFlags1`,
    `MaxTargetLevel`, `DmgClass`, `PreventionType`,
    `StanceBarOrder`, `RecoveryTime`, `CategoryRecoveryTime`
) VALUES (
    90010, -- ID
    0x00000100, -- Attributes (ABILITY)
    0x00000020, -- AttributesEx (SPECIAL)
    0x00000004, -- AttributesEx2 (AUTOREPEAT)
    1, -- CastingTimeIndex (instantané)
    21, -- DurationIndex (permanent)
    1, -- RangeIndex (self)
    1, -- SchoolMask (physique)
    132276, -- SpellIconID (icône d'ours)
    132276, -- ActiveIconID
    'Forme d''Ours Guerrier',
    16712190,
    6, -- Effect : APPLY_AURA
    0, 0,
    29422, -- EffectMiscValue1 : DisplayID de l'ours
    56, -- EffectAura : SPELL_AURA_TRANSFORM
    1, -- TargetA : SELF
    0, 0, -- Pas de famille de sort
    80, 0, 0, -- Max level, DmgClass, Prevention
    1, -- StanceBarOrder (position dans la barre de formes)
    0, 0 -- Pas de cooldown
);
```

#### **3.2 Sorts d'Aura (Effets passifs des formes)**

```sql
-- Aura : Bonus de la Forme d'Ours (90011)
INSERT INTO `spell_dbc` (
    `Id`, `Attributes`, `AttributesEx`,
    `CastingTimeIndex`, `DurationIndex`, `RangeIndex`,
    `SchoolMask`, `SpellIconID`, `ActiveIconID`,
    `SpellName`, `SpellNameFlag`,
    `Effect1`, `Effect2`, `Effect3`,
    `EffectDieSides1`, `EffectDieSides2`, `EffectDieSides3`,
    `EffectBasePoints1`, `EffectBasePoints2`, `EffectBasePoints3`,
    `EffectMiscValue1`, `EffectMiscValue2`, `EffectMiscValue3`,
    `EffectAura1`, `EffectAura2`, `EffectAura3`,
    `TargetA1`, `TargetA2`, `TargetA3`,
    `MaxTargetLevel`, `DmgClass`, `PreventionType`
) VALUES (
    90011,
    0x00000100, -- Attributes
    0x00000020, -- AttributesEx
    1, 21, 1, -- Casting, Duration, Range
    1, 132276, 132276, -- School, Icônes
    'Bénédiction de la Forme d''Ours',
    16712190,
    -- Effects
    6, 6, 6, -- APPLY_AURA
    0, 0, 0, -- DieSides
    10, 5, 0, -- BasePoints (+10% dégâts, +5% armure)
    0, 0, 0, -- MiscValues
    79, 22, 0, -- MOD_DAMAGE_PERCENT_DONE, MOD_RESISTANCE
    1, 1, 0, -- Targets SELF
    80, 0, 0
);
```

### **ÉTAPE 4 : Créer le Script Lua Principal**

```lua
-- shapeshift_system.lua
local ShapeshiftSystem = {}

-- Configuration
ShapeshiftSystem.forms = {}
ShapeshiftSystem.activeForms = {}

-- Chargement des formes depuis la DB
function ShapeshiftSystem:LoadForms()
    local query = WorldDBQuery("SELECT form_id, form_name, display_id, spell_id, aura_spell_id, flags, combat_state, scale, sound_id FROM custom_shapeshift_forms")
    if query then
        self.forms = {}
        repeat
            local row = query:GetRow()
            self.forms[tonumber(row[1])] = {
                id = tonumber(row[1]),
                name = row[2],
                displayId = tonumber(row[3]),
                spellId = tonumber(row[4]),
                auraSpellId = tonumber(row[5]),
                flags = tonumber(row[6]),
                combatState = tonumber(row[7]) == 1,
                scale = tonumber(row[8]) or 1.0,
                soundId = tonumber(row[9]) or 0
            }
        until not query:NextRow()
        return true
    end
    return false
end

-- Vérifier si un joueur a une forme active
function ShapeshiftSystem:GetActiveForm(player)
    return self.activeForms[player:GetGUIDLow()]
end

-- Appliquer une forme
function ShapeshiftSystem:ApplyForm(player, formId)
    local form = self.forms[formId]
    if not form then
        player:SendBroadcastMessage("|cFFFF0000Forme inconnue !|r")
        return false
    end
    
    -- Vérifier si le joueur a déjà une forme
    local currentForm = self:GetActiveForm(player)
    if currentForm then
        self:RemoveForm(player, false)
    end
    
    -- Vérifier l'état de combat
    if form.combatState and player:IsInCombat() then
        player:SendBroadcastMessage("|cFFFF0000Vous ne pouvez pas utiliser cette forme en combat !|r")
        return false
    end
    
    -- Sauvegarder l'état du joueur
    local playerGUID = player:GetGUIDLow()
    local originalState = {
        displayId = player:GetDisplayId(),
        scale = player:GetObjectScale(),
        mount = player:GetMountDisplayId(),
        powers = {}
    }
    
    -- Sauvegarder les pouvoirs actuels
    for spellId in pairs(player:GetSpellMap()) do
        if player:HasSpell(spellId) then
            originalState.powers[spellId] = true
        end
    end
    
    -- Appliquer la transformation
    player:SetDisplayId(form.displayId)
    player:SetObjectScale(form.scale)
    
    -- Démontage si nécessaire
    if player:IsMounted() then
        player:Dismount()
    end
    
    -- Appliquer l'aura de la forme
    if form.auraSpellId and form.auraSpellId > 0 then
        player:AddAura(form.auraSpellId, player)
    end
    
    -- Effets visuels
    player:SendPlaySpellVisual(6372)
    player:PlayDirectSound(form.soundId or 0)
    
    -- Message
    player:SendBroadcastMessage("|cFF00FF00Vous prenez la " .. form.name .. " !|r")
    player:SendAreaTriggerMessage("|cFF00FF00" .. form.name .. "|r")
    
    -- Sauvegarder l'état
    self.activeForms[playerGUID] = {
        form = form,
        originalState = originalState
    }
    
    -- Gérer la barre de formes
    self:UpdateShapeshiftBar(player)
    
    return true
end

-- Retirer la forme
function ShapeshiftSystem:RemoveForm(player, showMessage)
    local playerGUID = player:GetGUIDLow()
    local activeForm = self.activeForms[playerGUID]
    
    if not activeForm then
        return false
    end
    
    -- Restaurer l'état original
    player:SetDisplayId(activeForm.originalState.displayId)
    player:SetObjectScale(activeForm.originalState.scale)
    
    -- Retirer l'aura
    if activeForm.form.auraSpellId and activeForm.form.auraSpellId > 0 then
        player:RemoveAura(activeForm.form.auraSpellId)
    end
    
    -- Effets visuels
    player:SendPlaySpellVisual(6372)
    
    if showMessage ~= false then
        player:SendBroadcastMessage("|cFFFFFF00Vous reprenez votre forme normale.|r")
    end
    
    -- Nettoyer
    self.activeForms[playerGUID] = nil
    
    -- Mettre à jour la barre
    self:UpdateShapeshiftBar(player)
    
    return true
end

-- Mettre à jour la barre de formes (simulation)
function ShapeshiftSystem:UpdateShapeshiftBar(player)
    -- Cette fonction simule la barre de formes du druide
    local availableForms = {}
    
    for formId, form in pairs(self.forms) do
        if player:HasSpell(form.spellId) then
            table.insert(availableForms, form)
        end
    end
    
    -- Envoyer les formes disponibles au client
    player:SendShapeshiftFormBind(availableForms)
end

-- Gestionnaire de sorts
function ShapeshiftSystem:OnSpellCast(event, player, spell, skipCheck)
    local spellId = spell:GetEntry()
    
    -- Chercher si c'est un sort de forme
    for formId, form in pairs(self.forms) do
        if form.spellId == spellId then
            -- Si le joueur a déjà cette forme, la retirer
            local activeForm = self:GetActiveForm(player)
            if activeForm and activeForm.form.id == formId then
                self:RemoveForm(player, true)
            else
                self:ApplyForm(player, formId)
            end
            return
        end
    end
end

-- Gestionnaire de mort du joueur
function ShapeshiftSystem:OnPlayerDeath(event, player)
    self:RemoveForm(player, false)
end

-- Gestionnaire de déconnexion
function ShapeshiftSystem:OnPlayerLogout(event, player)
    self.activeForms[player:GetGUIDLow()] = nil
end

-- Commandes GM
function ShapeshiftSystem:HandleCommand(event, player, command)
    if command == "formes" then
        player:SendBroadcastMessage("|cFF00FF00=== Formes Disponibles ===|r")
        for formId, form in pairs(self.forms) do
            local status = player:HasSpell(form.spellId) and "|cFF00FF00[Apprise]|r" or "|cFFFF0000[Non apprise]|r"
            player:SendBroadcastMessage(string.format("%d - %s %s", form.id, form.name, status))
        end
        return false
    elseif command:match("^forme (%d+)$") then
        local formId = tonumber(command:match("(%d+)"))
        self:ApplyForm(player, formId)
        return false
    elseif command == "deforme" then
        self:RemoveForm(player, true)
        return false
    end
end

-- Initialisation
ShapeshiftSystem:LoadForms()

-- Enregistrement des événements
RegisterPlayerEvent(5, function(event, player, spell, skipCheck)
    ShapeshiftSystem:OnSpellCast(event, player, spell, skipCheck)
end)

RegisterPlayerEvent(33, function(event, player)
    ShapeshiftSystem:OnPlayerDeath(event, player)
end)

RegisterPlayerEvent(4, function(event, player)
    ShapeshiftSystem:OnPlayerLogout(event, player)
end)

RegisterPlayerEvent(42, function(event, player, command)
    return ShapeshiftSystem:HandleCommand(event, player, command)
end)
```

### **ÉTAPE 5 : Créer le Système de Barre de Formes**

```lua
-- shapeshift_bar.lua
local ShapeshiftBar = {}

-- Configuration de la barre
ShapeshiftBar.slots = 4 -- Nombre de formes maximum
ShapeshiftBar.forms = {}

function ShapeshiftBar:Initialize()
    -- Créer la barre de formes pour chaque joueur
    RegisterPlayerEvent(3, function(event, player) -- EVENT_ON_LOGIN
        self:SetupBar(player)
    end)
end

function ShapeshiftBar:SetupBar(player)
    -- Envoyer les formes disponibles
    local forms = self:GetPlayerForms(player)
    
    -- Créer la barre visuelle
    for i = 1, self.slots do
        if forms[i] then
            player:SendShapeshiftForm(forms[i].spellId, i)
        end
    end
end

function ShapeshiftBar:GetPlayerForms(player)
    local forms = {}
    local query = WorldDBQuery("SELECT form_id, form_name, spell_id FROM custom_shapeshift_forms")
    
    if query then
        repeat
            local row = query:GetRow()
            local formId = tonumber(row[1])
            local spellId = tonumber(row[3])
            
            if player:HasSpell(spellId) then
                table.insert(forms, {
                    id = formId,
                    name = row[2],
                    spellId = spellId
                })
            end
        until not query:NextRow()
    end
    
    return forms
end

-- Gestionnaire de clic sur la barre
function ShapeshiftBar:OnShapeshiftForm(event, player, formId)
    local forms = self:GetPlayerForms(player)
    
    for _, form in ipairs(forms) do
        if form.id == formId then
            player:CastSpell(player, form.spellId, true)
            return
        end
    end
end

-- Enregistrement des événements
RegisterPlayerEvent(34, function(event, player, formId) -- EVENT_ON_SHAPESHIFT_FORM
    ShapeshiftBar:OnShapeshiftForm(event, player, formId)
end)

ShapeshiftBar:Initialize()
```

### **ÉTAPE 6 : Ajouter les Restrictions de Classe**

```sql
-- Table des restrictions de formes par classe
CREATE TABLE IF NOT EXISTS `custom_form_class_restrictions` (
    `form_id` INT NOT NULL,
    `class_id` INT NOT NULL,
    `level_required` INT DEFAULT 1,
    `quest_required` INT DEFAULT 0,
    PRIMARY KEY (`form_id`, `class_id`)
);

-- Exemple : Forme d'ours pour guerriers niveau 40+
INSERT INTO `custom_form_class_restrictions` VALUES
(1, 1, 40, 0),  -- Guerrier
(2, 4, 30, 0),  -- Voleur
(3, 3, 20, 0),  -- Chasseur
(4, 7, 25, 0),  -- Chaman
(5, 9, 60, 0);  -- Démoniste
```

### **ÉTAPE 7 : Script de Vérification des Prérequis**

```lua
-- Ajouter dans ShapeshiftSystem
function ShapeshiftSystem:CanUseForm(player, form)
    -- Vérifier le niveau
    local playerLevel = player:GetLevel()
    local classId = player:GetClass()
    
    -- Vérifier les restrictions
    local query = WorldDBQuery(string.format(
        "SELECT level_required, quest_required FROM custom_form_class_restrictions WHERE form_id = %d AND class_id = %d",
        form.id, classId
    ))
    
    if query then
        local row = query:GetRow()
        local requiredLevel = tonumber(row[1])
        local requiredQuest = tonumber(row[2])
        
        if playerLevel < requiredLevel then
            player:SendBroadcastMessage(string.format(
                "|cFFFF0000Niveau %d requis pour cette forme !|r", requiredLevel
            ))
            return false
        end
        
        if requiredQuest > 0 and not player:HasQuest(requiredQuest) then
            player:SendBroadcastMessage("|cFFFF0000Quête requise non complétée !|r")
            return false
        end
    end
    
    -- Vérifier l'état de combat
    if not form.combatState and player:IsInCombat() then
        player:SendBroadcastMessage("|cFFFF0000Impossible en combat !|r")
        return false
    end
    
    return true
end
```

### **ÉTAPE 8 : Créer le Modpack/Module Complet**

**Structure du dossier :**
```
azerothcore/
├── lua_scripts/
│   ├── shapeshift_system.lua
│   ├── shapeshift_bar.lua
│   └── shapeshift_config.lua
├── sql/
│   └── custom/
│       ├── shapeshift_forms.sql
│       ├── shapeshift_spells.sql
│       └── shapeshift_restrictions.sql
└── docs/
    └── README.md
```

**README.md :**
```markdown
# Modpack : Système de Formes Personnalisées

## Installation
1. Importez les fichiers SQL dans votre base de données
2. Copiez les scripts Lua dans `lua_scripts/`
3. Redémarrez AzerothCore

## Utilisation
- Commande GM : `.formes` pour voir les formes disponibles
- Commande GM : `.forme [id]` pour tester une forme
- Commande GM : `.deforme` pour retirer la forme

## Personnalisation
- Ajoutez des formes dans `custom_shapeshift_forms`
- Modifiez les restrictions dans `custom_form_class_restrictions`
- Ajustez les bonus dans les sorts d'aura
```

### **ÉTAPE 9 : Tests et Debug**

```lua
-- shapeshift_debug.lua
local DebugSystem = {}

function DebugSystem:EnableDebug(player)
    player:SendBroadcastMessage("|cFFFFFF00=== Debug Formes Activé ===|r")
    
    -- Vérifier les formes
    local forms = ShapeshiftSystem.forms
    player:SendBroadcastMessage(string.format("Formes chargées : %d", #forms))
    
    -- Vérifier l'état du joueur
    local displayId = player:GetDisplayId()
    local scale = player:GetObjectScale()
    player:SendBroadcastMessage(string.format("DisplayID actuel : %d, Scale : %.2f", displayId, scale))
end

RegisterPlayerEvent(42, function(event, player, command)
    if command == "debugformes" then
        DebugSystem:EnableDebug(player)
        return false
    end
end)
```

### **Résumé du Workflow :**

1. **Étape 1** : Comprendre le système des formes
2. **Étape 2** : Créer les tables SQL pour les formes
3. **Étape 3** : Créer les sorts de transformation et d'aura
4. **Étape 4** : Créer le script Lua principal
5. **Étape 5** : Implémenter la barre de formes
6. **Étape 6** : Ajouter les restrictions de classe
7. **Étape 7** : Vérifier les prérequis
8. **Étape 8** : Organiser le modpack
9. **Étape 9** : Tester et déboguer

Ce système complet vous permet de créer des formes type druide pour toutes les classes, avec les fonctionnalités complètes de gestion, de restrictions et d'effets. Voulez-vous que je détaille un aspect particulier ou que j'ajoute des fonctionnalités spécifiques ?

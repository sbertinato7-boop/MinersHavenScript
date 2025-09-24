-- =============================
-- VENYX UI ORE TRACKER FOR MINER'S HAVEN
-- =============================
local library = loadstring(game:HttpGet("https://raw.githubusercontent.com/GreenDeno/Venyx-UI-Library/main/source.lua"))()
local venyx = library.new("Prestige Hub", 5013109572)

local Suffixes = { 
    "k", "M", "B", "T", "qd", "Qn", "sx", "Sp", "O", "N", "de", "Ud", "DD", "tdD", "qdD", "QnD", "sxD", "SpD", "OcD", "NvD", 
    "Vgn", "UVg", "DVg", "TVg", "qtV", "QnV", "SeV", "SPG", "OVG", "NVG", "TGN", "UTG", "DTG", "tsTG", "qtTG", "QnTG", "ssTG", 
    "SpTG", "OcTG", "NoTG", "QdDR", "uQDR", "dQDR", "tQDR", "qdQDR", "QnQDR", "sxQDR", "SpQDR", "OQDDr", "NQDDr", "qQGNT", 
    "uQGNT", "dQGNT", "tQGNT", "qdQGNT", "QnQGNT", "sxQGNT", "SpQGNT", "OQQGNT", "NQQGNT", "SXGNTL", "USXGNTL", "DSXGNTL", 
    "TSXGNTL", "QTSXGNTL", "QNSXGNTL", "SXSXGNTL", "SPSXGNTL", "OSXGNTL", "NVSXGNTL", "SPTGNTL", "USPTGNTL", "DSPTGNTL", 
    "TSPTGNTL", "QTSPTGNTL", "QNSPTGNTL", "SXSPTGNTL", "SPSPTGNTL", "OSPTGNTL", "NVSPTGNTL", "OTGNTL", "UOTGNTL", "DOTGNTL", 
    "TOTGNTL", "QTOTGNTL","QNOTGNTL", "SXOTGNTL", "SPOTGNTL", "OTOTGNTL", "NVOTGNTL", "NONGNTL", "UNONGNTL", "DNONGNTL", 
    "TNONGNTL", "QTNONGNTL", "QNNONGNTL", "SXNONGNTL", "SPNONGNTL", "OTNONGNTL", "NONONGNTL", "CENT", "UNCENT","inf"
}

local function formatNumber(num)
    if not num or typeof(num) ~= "number" then 
        print("[DEBUG formatNumber] Invalid input:", tostring(num), typeof(num))
        return tostring(num) 
    end
    if num < 1000 then return string.format("%.0f", num) end
    local index = 0
    while num >= 1000 and index < #Suffixes do
        num /= 1000
        index += 1
    end
    local formatted = string.format("%.1f", num)
    formatted = formatted:gsub("%.0$", "")
    return formatted .. (Suffixes[index] or "")
end

-- =============================
-- THEME CONFIGURATION
-- =============================
local themes = {
    Background = Color3.fromRGB(24, 24, 24),
    Glow = Color3.fromRGB(0, 0, 0),
    Accent = Color3.fromRGB(10, 10, 10),
    LightContrast = Color3.fromRGB(20, 20, 20),
    DarkContrast = Color3.fromRGB(14, 14, 14),
    TextColor = Color3.fromRGB(255, 255, 255)
}

-- =============================
-- GLOBAL VARIABLES
-- =============================

-- Box Management Variables
local boxNames = {
    "Regular","Unreal","Inferno","Luxury","Red-Banded","Spectral",
    "Magnificent","Heavenly","Pumpkin","Festive","Birthday","Twitch",
    "Easter","Cake Raffle"
}
local selectedBox = "Regular"
local autoOpenSelected = false
local autoOpenAll = false
local autoBoxLoop = nil
local autoBoxesEnabled = false
local boxesLoop = nil

-- Ore Tracking Variables
local diagnosticMode = true
local labelCount = 0
local scanCount = 0
local errorCount = 0
local oreTrackerEnabled = false
local trackerLoop = nil
local labelSize = 120

-- Ore Booster Variables
local AutoUpgradeEnabled = false
local boosterEnabled = false
local factoryConnections = {}
local processed = setmetatable({}, { __mode = "k" })

-- Industrial Mine Variables
local usingIndustrialMine = false
local industrialMineItem = nil

-- Player Variables
local currentConveyorSpeed = 1
local factoryWatchConn = nil
local conveyorWarned = false

-- =============================
-- FURNACE AND UPGRADER LISTS
-- =============================

local furnaceNames = {
    "Advanced Furnace","Aether Refinery","Aethereal Synthesizer","All Star Behemoth Cake","Ambrosia Forest",
    "Ambrosia Fountain","Ambrosia Garden","Ancient Magic","Angelic Grace","Anguished Garden",
    "Anguished Guardian of the Gate","Assembly Unit","Aurora Borealis","Australis Fornacem","Awakened Jack O Lantern",
    "B","Bag of Stars","Basic Furnace","Basic Research Center","Belated 4th Birthday Cake",
    "Birthday Cake","Blood Magic","Book of Knowledge","Breached Motherboard","C",
    "Candy Bag","Candy Center","Candy Factory","Candy Metropolis","Candy Research Center",
    "Canis Processus","Carat Carrot Factory","Cell Collider","Cell Furnace","Cell Incinerator",
    "Cell Particalizer","Cell Processor","Cell Singularity","Coconut Drencher","Coliseum Catharsis",
    "Conquered Wasteland","Crystal Altar","Crystal Shrine","D","Dark Illuminator",
    "Dark Magic","DECADE HAVEN CAKE","Devourer of Nightmares","Diamond Falls","Dracula's Coffin",
    "Dreamcatcher","Dreamer's Anguish","Dreamer's Blight","Dreamer's Devastation","Dreamer's Fright",
    "Dreamer's Might","Dreamer's Nightmare","Dreamer's Sight","Dreamer's Terror","Dreamer's Valor",
    "E","Electronic Furnace","Elevated Furnace","Elysium Solemnity","Equinox",
    "Eternal Fracture","Eternal Journey","Eternal Limbo","Ethereal Sanctuary","Excalibur",
    "F","Flying Dutchman","Forbidden Magic","Forsaken Cake","Frozen Justice",
    "Frozen Peaks","Funky Town","Fusion Reactor","G","Gaia's Grasp",
    "Gargantium Core","Gladiator's Fury","Grand Birthday Cake","Grand Christmas Tree","Grand Willow",
    "Gravitational Gyrocopter","Guardian of the Gate","Guardian of the Portal","H","Haunted Carnival",
    "Heart of Dusk","Heavenly Forge","Heavenly Wisp","Hungry Spirit","I",
    "Igneous Forge","Industrial Scarab","Innovator's Research Center","Interstellar Conqueror","Invasive Cyberlord",
    "K","Kappa Investor","L","Lemon Land","Leprechaun's Hat",
    "Lucky Research Center","Lunar Bombardment","N","Natural Spring","Nature's Enchantment",
    "Nature's Grip","Nature's Temple","New Year's City","Northern Lights","Nuclear Castle",
    "Nuclear Stronghold","O","Oasis","Oblivion Emission","Octopi Council",
    "Ore Encapsulator","Ore Incinerator","P","Pandora's Box","Phase Processor",
    "Phasecursor Furnace","Pirate's Loot Collector","Pizza Furnace","Pizza Research Center","Plagued Forest",
    "Pot of Gold","Precursor Furnace","Present Center","Proficient Research Center","Pumpkin Possession",
    "Q","Quantum Processor","Quantum Research Center","Queen Birthday Cake","R",
    "Raised Furnace","Redwood Compass","Reinforced Furnace","Research Center","Category:Research Center",
    "Rock Candy Center","S","Sacrificial Altar","Sacrificial Cell","Sage Justice",
    "Sage King","Sage Redeemer","Sakura Falls","Sakura Garden","Santa's Chimney",
    "Shrine of Penitence","Solar Eruption","Solar Flare","Soul Blossom","Spectral Research Center",
    "Spooky Fruit Punch Processor","Spooky Research Center","Supreme Birthday Cake","Supreme Christmas Tree","Sweet Tooth",
    "Swift Justice","Sword Master's Spirit","T","Temporal Enchantment","The Black Hole",
    "The Dream-Maker","The Easter Shrine","The Fissure","The Fracture","The Great Parasite",
    "The Heart of Void","The Sunken Past","THE UNDERTAKER","The Vault","Thingamajig",
    "Timeless Enhancement","Tyrant's Forge","Tyrant's Throne","V","Volcango",
    "Y","Yuge Furnace"
}

-- Create furnace lookup table for performance
local furnaceLookup = {}
for _,name in ipairs(furnaceNames) do 
    furnaceLookup[name] = true 
end

-- =============================
-- AUTO REMOTE CLICKER
-- =============================
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RemoteDrop = ReplicatedStorage:WaitForChild("RemoteDrop")

local RemoteClickerEnabled = false
local RemoteClickerLoop = nil
local clickerDelay = 0.1 -- delay between clicks (seconds)

local function startRemoteClicker()
    if RemoteClickerLoop then return end
    RemoteClickerLoop = task.spawn(function()
        while RemoteClickerEnabled do
            pcall(function()
                RemoteDrop:FireServer()
            end)
            task.wait(clickerDelay)
        end
    end)
    print("[Remote Clicker] Enabled")
end

local function stopRemoteClicker()
    if RemoteClickerLoop then
        pcall(task.cancel, RemoteClickerLoop)
        RemoteClickerLoop = nil
    end
    print("[Remote Clicker] Disabled")
end

-- =============================
-- BOX MANAGEMENT FUNCTIONS
-- =============================
local function openBox(name)
    pcall(function()
        local remote = game:GetService("ReplicatedStorage"):WaitForChild("MysteryBox")
        if remote then remote:InvokeServer(name) print("Opened box:", name) end
    end)
end

local function startAutoBoxLoop()
    if autoBoxLoop then return end
    autoBoxLoop = task.spawn(function()
        while autoOpenSelected or autoOpenAll do
            if autoOpenSelected then openBox(selectedBox)
            elseif autoOpenAll then for _, name in ipairs(boxNames) do openBox(name) task.wait(0.2) end end
            task.wait(0.5)
        end
    end)
end

local function stopAutoBoxLoop()
    if autoBoxLoop then pcall(task.cancel, autoBoxLoop) autoBoxLoop = nil end
end

local function collectBox(box)
    local hrp = game:GetService("Players").LocalPlayer.Character and game:GetService("Players").LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return false end
    local primaryPart = box:IsA("BasePart") and box or box.PrimaryPart or box:FindFirstChildWhichIsA("BasePart")
    if not primaryPart then return false end
    local originalCFrame = hrp.CFrame
    pcall(function() hrp.CFrame = primaryPart.CFrame * CFrame.new(0, -(primaryPart.Size.Y / 2) + 1, 0) end)
    task.wait(0.15)
    pcall(function() hrp.CFrame = originalCFrame end)
    print("[AutoBox] Collected (touched):", box.Name)
    return true
end

local function scanAndCollectBoxes()
    local Boxes = workspace:FindFirstChild("Boxes")
    if not Boxes then return end
    local collectedCount = 0
    for _, box in pairs(Boxes:GetChildren()) do
        if collectBox(box) then collectedCount = collectedCount + 1 end
        if box.Name:match("Present") or box.Name:match("Gift") then
            local crate = box:FindFirstChild("crate")
            if crate and collectBox(crate) then collectedCount = collectedCount + 1 end
        end
        for _, descendant in pairs(box:GetDescendants()) do
            if (descendant.Name:lower():match("crate") or descendant.Name:lower():match("box")) and collectBox(descendant) then
                collectedCount = collectedCount + 1
            end
        end
    end
    if collectedCount > 0 then print("Total boxes collected this scan:", collectedCount) end
end

local function startAutoBoxes()
    if boxesLoop then return end
    boxesLoop = task.spawn(function() while autoBoxesEnabled do scanAndCollectBoxes() task.wait(1) end end)
    print("Auto Collect Boxes enabled")
end

local function stopAutoBoxes()
    if boxesLoop then pcall(task.cancel, boxesLoop) boxesLoop = nil end
    print("Auto Collect Boxes disabled")
end

-- =============================
-- AUTO REBIRTH CORE LOGIC
-- =============================
local autoRebirthEnabled = false
local rebirthLoop
local minSkips = 0
local rebirthDelay = 1

local suffixToIndex = {}
for i, suffix in ipairs(Suffixes) do
    suffixToIndex[suffix] = i
    suffixToIndex[suffix:lower()] = i
end

local function compareSuffixValues(value1, suffix1, value2, suffix2)
    local index1 = suffixToIndex[suffix1] or 0
    local index2 = suffixToIndex[suffix2] or 0
    
    if index1 > index2 then
        return true
    elseif index1 < index2 then
        return false
    else
        return value1 >= value2
    end
end

local function getPlayerCashComponents()
    local player = game.Players.LocalPlayer
    if not player then return 0, "" end

    local leaderstats = player:FindFirstChild("leaderstats")
    if not leaderstats then return 0, "" end

    local cashValue = leaderstats:FindFirstChild("Cash")
    if not cashValue then return 0, "" end

    local rawCash = tostring(cashValue.Value)
    local cleanCash = rawCash:gsub("%$", "")
    local num, suffix = cleanCash:match("([%d%.]+)([%a]*)")
    
    if not num then return 0, "" end
    local value = tonumber(num)
    if not value then return 0, "" end

    return value, suffix or ""
end

local function getRebirthPriceComponents()
    local playerGui = game.Players.LocalPlayer:FindFirstChild("PlayerGui")
    if not playerGui then return nil, "" end

    local rebirthGui = playerGui:FindFirstChild("Rebirth")
    if not rebirthGui then return nil, "" end

    local rebornButton = rebirthGui:FindFirstChild("Frame")
        and rebirthGui.Frame:FindFirstChild("Rebirth_Content")
        and rebirthGui.Frame.Rebirth_Content:FindFirstChild("Content")
        and rebirthGui.Frame.Rebirth_Content.Content:FindFirstChild("Rebirth")
        and rebirthGui.Frame.Rebirth_Content.Content.Rebirth:FindFirstChild("Frame")
        and rebirthGui.Frame.Rebirth_Content.Content.Rebirth.Frame:FindFirstChild("Bottom")
        and rebirthGui.Frame.Rebirth_Content.Content.Rebirth.Frame.Bottom:FindFirstChild("Reborn")

    if not rebornButton then return nil, "" end

    local rawText = rebornButton.Text
    local number, suffix = rawText:match("[Bb][Ee]%s*[Rr][Ee][Bb][Oo][Rr][Nn]:%s*%$?([%d%.]+)([%a]+)")
    if not number or not suffix then return nil, "" end

    local base = tonumber(number)
    if not base then return nil, "" end

    return base, suffix
end

local function getPlayerCash()
    local num, suffix = getPlayerCashComponents()
    if suffix == "" then return num end
    
    local index = suffixToIndex[suffix]
    if not index then return 0 end
    
    if index <= 15 then
        return num * (1000 ^ index)
    else
        return math.huge
    end
end

local function startAutoRebirth()
    if rebirthLoop then return end
    autoRebirthEnabled = true
    rebirthLoop = task.spawn(function()
        while autoRebirthEnabled do
            local priceNum, priceSuffix = getRebirthPriceComponents()
            local skips = tonumber(minSkips) or 0
            
            if priceNum and priceSuffix then
                local cashNum, cashSuffix = getPlayerCashComponents()
                
                local basePriceIndex = suffixToIndex[priceSuffix] or 0
                local finalSkipIndex = basePriceIndex + skips
                local finalSkipSuffix = Suffixes[finalSkipIndex] or priceSuffix
                local finalSkipPrice = priceNum
                
                local canAfford = false
                if skips == 0 then
                    canAfford = compareSuffixValues(cashNum, cashSuffix, priceNum, priceSuffix)
                else
                    canAfford = compareSuffixValues(cashNum, cashSuffix, finalSkipPrice, finalSkipSuffix)
                end
                
                if canAfford then
                    pcall(function()
                        game.ReplicatedStorage.Rebirth:InvokeServer()
                    end)
                    task.wait(rebirthDelay)
                else
                    task.wait(0.5)
                end
            else
                task.wait(1)
            end
        end
    end)
end

local function stopAutoRebirth()
    autoRebirthEnabled = false
    if rebirthLoop then
        task.cancel(rebirthLoop)
        rebirthLoop = nil
    end
    print("[AUTO REBIRTH] Stopped")
end

-- =============================
-- ORE TRACKING FUNCTIONS
-- =============================

local function debugPrint(message)
    if diagnosticMode then
        print("[ORE TRACKER DEBUG]", message)
    end
end

local function createLabel(part)
    if not part or not part.Parent or part:FindFirstChild("_OreLabel") or tostring(part.Name):match("Factory") then return end
    
    local valueObject = part:FindFirstChildOfClass("NumberValue") or part:FindFirstChildOfClass("IntValue") or 
                       part:FindFirstChild("Cash") or part:FindFirstChildWhichIsA("NumberValue", true) or 
                       part:FindFirstChildWhichIsA("IntValue", true)
    if not valueObject then return end
    
    local gui = Instance.new("BillboardGui")
    gui.Name = "_OreLabel"
    gui.Size = UDim2.new(0, labelSize, 0, math.floor(labelSize/3))
    gui.StudsOffset = Vector3.new(0, 4, 0)
    gui.AlwaysOnTop = true
    gui.MaxDistance = 300
    gui.Adornee = part
    gui.Parent = part
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1,0,1,0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(0,255,0)
    label.TextScaled = true
    label.Font = Enum.Font.SourceSansBold
    label.TextStrokeTransparency = 0
    label.TextStrokeColor3 = Color3.fromRGB(0,0,0)
    label.Text = "$" .. formatNumber(valueObject.Value)
    label.Parent = gui
    
    valueObject.Changed:Connect(function()
        if label.Parent then
            label.Text = "$" .. formatNumber(valueObject.Value)
        end
    end)
end

local function updateAllLabelSizes()
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("BillboardGui") and obj.Name == "_OreLabel" then
            obj.Size = UDim2.new(0, labelSize, 0, math.floor(labelSize/3))
        end
    end
end

local function scanForOres()
    local DroppedParts = workspace:FindFirstChild("DroppedParts")
    if not DroppedParts then return end
    local function scanEverything(parent)
        for _, child in pairs(parent:GetChildren()) do
            if child:IsA("BasePart") then createLabel(child)
            elseif child:IsA("Model") then
                local firstPart = child:FindFirstChildWhichIsA("BasePart")
                if firstPart then createLabel(child) end
                scanEverything(child)
            elseif #child:GetChildren() > 0 then scanEverything(child) end
        end
    end
    scanEverything(DroppedParts)
end

local function startOreTracker()
    if trackerLoop then return end
    trackerLoop = task.spawn(function() 
        while oreTrackerEnabled do 
            scanForOres() 
            task.wait(0.05) 
        end 
    end)
    print("Ore Tracker enabled")
end

local function stopOreTracker()
    if trackerLoop then 
        pcall(task.cancel, trackerLoop) 
        trackerLoop = nil 
    end
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("BillboardGui") and obj.Name == "_OreLabel" then 
            pcall(obj.Destroy, obj) 
        end
    end
    print("Ore Tracker disabled")
end

-- =============================
-- ORE PROCESSING SYSTEM (Based on second script method)
-- =============================

local furnaceRefreshConnection = nil

-- Configuration
local MoneyLoopables = {
    ["Large Ore Upgrader"] = {Cap = 50e+3, Effect = nil, MinVal = nil},
    ["Solar Large Upgrader"] = {Cap = 50e+3, Effect = nil, MinVal = nil},
    ["Precision Refiner"] = {Cap = 1e+8, Effect = "Fire", MinVal = nil},
    ["Rainbow Upgrader"] = {Cap = 1e+8, Effect = nil, MinVal = nil},
    ["Way-Up-High Upgrader"] = {Cap = 1e+9, Effect = nil, MinVal = nil},
    ["Digital Ore Cleaner"] = {Cap = 10e+9, Effect = nil, MinVal = nil},
    ["Freon-Blast Upgrader"] = {Cap = 125e+9, Effect = nil, MinVal = nil},
    ["Radioactive Refiner"] = {Cap = 500e+9, Effect = nil, MinVal = nil},
    ["Fire-Blast Upgrader"] = {Cap = 64e+9, Effect = "Fire", MinVal = nil},
    ["Serpentine Upgrader"] = {Cap = 1e+12, Effect = nil, MinVal = nil},
    ["Suspended Refiner"] = {Cap = 1e+18, Effect = nil, MinVal = nil},
    ["Molten Upgrader"] = {Cap = 50e+18, Effect = nil, MinVal = 1e+12},
    ["Advanced Ore Atomizer"] = {Cap = 1e+21, Effect = nil, MinVal = nil},
    ["Freon Suppressor"] = {Cap = 100e+21, Effect = nil, MinVal = nil},
    ["Horizon Centrifuge"] = {Cap = 1e+24, Effect = nil, MinVal = 1e+21},
    ["Ore Thermocrusher"] = {Cap = 100e+24, Effect = nil, MinVal = nil},
    ["Suspended Lava Refiner"] = {Cap = 1e+27, Effect = nil, MinVal = nil},
    ["Ore Transistor"] = {Cap = 1e+30, Effect = nil, MinVal = 1e+24},
    ["Ore Transponder"] = {Cap = 1e+33, Effect = nil, MinVal = nil},
    ["Morning Star"] = {Cap = 1e+30, Effect = "Fire", MinVal = nil, MinWait = 1.5},
    ["⭐ Celestial Morning Star ⭐"] = {Cap = 1e+30, Effect = "Fire", MinVal = nil, MinWait = 1.5},
    ["Red Giant"] = {Cap = 1e+60, Effect = "Fire", MinVal = nil, MinWait = 1.5},
    ["⭐ Wholesome Red Giant ⭐"] = {Cap = 1e+60, Effect = "Fire", MinVal = nil, MinWait = 1.5},
    ["Catalyzed Star"] = {Cap = 1e+60, Effect = "Fire", MinVal = nil, MinWait = 1.5},
    ["Neutron Star"] = {Cap = 1e+72, Effect = "Fire", MinVal = nil, MinWait = 1.5},
    ["⭐ Wholesome Neutron Star ⭐"] = {Cap = 1e+72, Effect = "Fire", MinVal = nil, MinWait = 1.5},
    ["Blue Supergiant"] = {Cap = 1e+90, Effect = "Fire", MinVal = nil, MinWait = 1.5},
    ["⭐ Hypergiant Blue Supergiant ⭐"] = {Cap = 1e+90, Effect = "Fire", MinVal = nil, MinWait = 1.5},
}

local EffectRemovers = {"Wild Spore", "Deadly Spore", "Azure Spore", "The Death Cap"}
local ResettersNames = {"Tesla Resetter","Tesla Refuter","Black Dwarf","Void Star","The Ultimate Sacrifice","The Final Upgrader","Daestrophe"}

-- Variables
local tycoonFolder = nil
local furnaceItem = nil
local indMineItem = nil

-- Helper Functions
local function findPlayerTycoon()
    local player = game.Players.LocalPlayer
    if not player then return nil end
    
    -- Method 1: Try using PlayerTycoon.Value (from second script)
    if player:FindFirstChild("PlayerTycoon") and player.PlayerTycoon.Value then
        return player.PlayerTycoon.Value
    end
    
    -- Method 2: Fallback to searching workspace.Tycoons (original method)
    local tycoons = workspace:FindFirstChild("Tycoons")
    if not tycoons then return nil end
    
    for _, factory in ipairs(tycoons:GetChildren()) do
        local ownerVal = factory:FindFirstChild("Owner")
        if ownerVal and ownerVal.Value == player then
            return factory
        end
    end
    
    return nil
end

local function getFurnaceItems()
    if not tycoonFolder then return end
    
    local oldFurnace = furnaceItem
    local oldIndMine = indMineItem
    local oldIndustrialMine = industrialMineItem
    furnaceItem = nil
    indMineItem = nil
    industrialMineItem = nil
    
    for _, item in ipairs(tycoonFolder:GetChildren()) do
        if item:FindFirstChild("Model") and item.Model:FindFirstChild("Lava") and not item.Model.Lava:FindFirstChild("TeleportSend") then
            if not item.Model:FindFirstChild("Drop") and not furnaceItem then 
                furnaceItem = item
                if oldFurnace ~= furnaceItem then
                    print("[OreBooster] New furnace detected:", furnaceItem.Name)
                end
            end
            if item.Model:FindFirstChild("Drop") and item.Model:FindFirstChild("Lava") and not indMineItem then
                indMineItem = item
                if oldIndMine ~= indMineItem then
                    print("[OreBooster] New industrial mine detected:", indMineItem.Name)
                end
            end
        end
        
        -- Look for Industrial Mines (improved detection - check for "Industrial" OR "Mine" in name)
        if usingIndustrialMine and item:FindFirstChild("Model") then
            local itemName = item.Name:lower()
            if (itemName:find("industrial") or itemName:find("mine")) and item.Model:FindFirstChild("Drop") and item.Model:FindFirstChild("Lava") then
                industrialMineItem = item
                if oldIndustrialMine ~= industrialMineItem then
                    print("[OreBooster] New industrial mine detected for ore processing:", industrialMineItem.Name)
                    print("[OreBooster] Industrial mine path:", item:GetFullName())
                end
            end
        end
    end
    
    if not furnaceItem then
        print("[OreBooster] Warning: No furnace found!")
    end
    
    if usingIndustrialMine and not industrialMineItem then
        print("[OreBooster] Warning: Industrial Mine mode enabled but no Industrial Mine found!")
        print("[OreBooster] Available items in tycoon:")
        for _, item in ipairs(tycoonFolder:GetChildren()) do
            if item:FindFirstChild("Model") then
                print(" -", item.Name, "| Has Drop:", tostring(item.Model:FindFirstChild("Drop") ~= nil), "| Has Lava:", tostring(item.Model:FindFirstChild("Lava") ~= nil))
            end
        end
    end
end

local function startFurnaceAutoDetection()
    if furnaceRefreshConnection then return end
    
    furnaceRefreshConnection = task.spawn(function()
        while boosterEnabled do
            if tycoonFolder then
                getFurnaceItems()
                
                -- Also listen for new items being placed
                for _, item in ipairs(tycoonFolder:GetChildren()) do
                    if not item:GetAttribute("_OreBoosterWatched") and item:FindFirstChild("Model") then
                        item:SetAttribute("_OreBoosterWatched", true)
                        
                        -- Watch for model changes that might indicate a furnace
                        item.Model.ChildAdded:Connect(function(child)
                            if child.Name == "Lava" and not child:FindFirstChild("TeleportSend") then
                                task.wait(0.1) -- Small delay to ensure everything is loaded
                                getFurnaceItems()
                            end
                        end)
                    end
                end
            end
            task.wait(2) -- Check every 2 seconds
        end
    end)
    
    print("[OreBooster] Furnace auto-detection started")
end

local function stopFurnaceAutoDetection()
    if furnaceRefreshConnection then
        pcall(task.cancel, furnaceRefreshConnection)
        furnaceRefreshConnection = nil
        print("[OreBooster] Furnace auto-detection stopped")
    end
end

local function sendOreToIndustrialMine(ore)
    if not ore or not ore.Parent or not industrialMineItem then return false end
    
    local success = false
    pcall(function()
        -- Stop ore movement
        ore.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
        ore.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
        ore.Velocity = Vector3.new(0, 0, 0)
        
        -- Mark this ore as already processed by Industrial Mine
        ore:SetAttribute("ProcessedByIndustrialMine", true)
        
        if industrialMineItem:FindFirstChild("Model") and industrialMineItem.Model:FindFirstChild("Lava") then
            local lava = industrialMineItem.Model.Lava
            
            -- Try multiple teleportation methods for reliability
            for i = 1, 5 do
                if ore and ore.Parent then
                    ore.CFrame = lava.CFrame + Vector3.new(math.random(-2,2), 3, math.random(-2,2))
                    task.wait(0.05)
                end
            end
            
            print("[Industrial Mine] Sent ore to industrial mine:", industrialMineItem.Name)
            success = true
        else
            print("[Industrial Mine] ERROR: Could not find Lava component in:", industrialMineItem.Name)
        end
    end)
    
    return success
end

local function isCoalMineOre(ore)
    -- Check if the ore came from a coal mine by looking at its properties or parent folder
    if not ore or not ore.Parent then return false end
    
    -- Skip ores that have already been processed by Industrial Mine
    if ore:GetAttribute("ProcessedByIndustrialMine") then
        print("[Industrial Mine] Ore already processed by Industrial Mine, skipping:", ore.Name)
        return false
    end
    
    -- Method 1: Check if ore has properties typical of coal mine ores (Industrial Coal Mine produces $0 ores)
    if ore:FindFirstChild("Cash") then
        local cashValue = ore.Cash.Value
        if cashValue == 0 then
            print("[Industrial Mine] Detected coal mine ore (Cash = $0):", ore.Name)
            return true
        end
    end
    
    -- Method 2: Check if ore name suggests it's from coal mine
    local oreName = ore.Name:lower()
    if oreName:find("coal") then
        print("[Industrial Mine] Detected coal ore by name:", ore.Name)
        return true
    end
    
    -- Method 3: Check if it's a very low value ore that might benefit from industrial processing
    -- But exclude ores that might be processed industrial mine outputs
    if ore:FindFirstChild("Cash") and ore.Cash.Value > 0 and ore.Cash.Value < 1000 then
        print("[Industrial Mine] Detected low-value ore for industrial processing:", ore.Name, "Value:", ore.Cash.Value)
        return true
    end
    
    return false
end

local function boostOre(ore) 
    if not ore or not ore.Parent or not boosterEnabled then return end
    
    for _, item in ipairs(tycoonFolder:GetChildren()) do
        if not boosterEnabled then break end
        if not ore or not ore.Parent then break end
        
        -- Skip money loopables and resetters for regular boost
        if MoneyLoopables[item.Name] or table.find(ResettersNames, item.Name) then continue end
        
        if item:FindFirstChild("ItemId") and item:FindFirstChild("Model") and item.Model:FindFirstChild("Upgrade") then
            for i = 1, 3 do
                if ore and ore.Parent then
                    pcall(function()
                        ore.CFrame = item.Model.Upgrade.CFrame 
                    end)
                    task.wait(0.01)
                end
            end
        end
    end
end

local function resetOre(ore)
    if not ore or not ore.Parent or not tycoonFolder then return end
    
    -- Find available resetters - FIXED LOGIC
    local resetters = {}
    
    -- Check each resetter individually and add to list if found
    local dae = tycoonFolder:FindFirstChild("Daestrophe")
    local sac = tycoonFolder:FindFirstChild("The Final Upgrader") or tycoonFolder:FindFirstChild("The Ultimate Sacrifice")
    local star = tycoonFolder:FindFirstChild("Void Star") or tycoonFolder:FindFirstChild("Black Dwarf")
    local tes = tycoonFolder:FindFirstChild("Tesla Resetter") or tycoonFolder:FindFirstChild("Tesla Refuter")
    
    -- Add resetters to list in priority order
    if dae then table.insert(resetters, dae) end
    if sac then table.insert(resetters, sac) end
    if star then table.insert(resetters, star) end
    if tes then table.insert(resetters, tes) end
    
    print("[DEBUG] Found resetters:", #resetters)
    for i, resetter in ipairs(resetters) do
        print(" - " .. resetter.Name)
    end
    
    -- Boost ore before resetters
    boostOre(ore)
    
    -- Process each resetter
    for _, resetter in ipairs(resetters) do
        if not ore or not ore.Parent or not boosterEnabled then break end
        
        print("[DEBUG] Processing with:", resetter.Name)
        
        -- Check if resetter has proper model structure
        if resetter:FindFirstChild("Model") and resetter.Model:FindFirstChild("Upgrade") then
            for i = 1, 5 do -- Increased attempts for better reliability
                if ore and ore.Parent then
                    pcall(function()
                        -- Stop ore movement first
                        ore.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
                        ore.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                        ore.Velocity = Vector3.new(0, 0, 0)
                        
                        -- Teleport to resetter
                        ore.CFrame = resetter.Model.Upgrade.CFrame
                        print("[DEBUG] Teleported ore to", resetter.Name, "attempt", i)
                    end)
                    task.wait(0.02) -- Slightly longer wait for processing
                else
                    break
                end
            end
            
            -- Boost ore again after each resetter
            task.wait(0.1) -- Wait for resetter to process
            if ore and ore.Parent then
                boostOre(ore)
            end
        else
            warn("[DEBUG] Resetter missing Model/Upgrade:", resetter.Name)
        end
    end
    
    print("[DEBUG] Finished processing ore through all resetters")
end

local function processMoneyLoopable(ore)
    if not ore or not ore.Parent or not tycoonFolder then return end
    
    local moneyLoopItem = nil
    local protectItem = nil
    
    -- Find money loopable
    for itemName, _ in pairs(MoneyLoopables) do
        local item = tycoonFolder:FindFirstChild(itemName)
        if item then 
            moneyLoopItem = item
            break
        end
    end
    
    -- Find effect remover
    for _, removerName in ipairs(EffectRemovers) do
        local item = tycoonFolder:FindFirstChild(removerName)
        if item then 
            protectItem = item
            break
        end
    end
    
    if moneyLoopItem and ore:FindFirstChild("Cash") then
        local info = MoneyLoopables[moneyLoopItem.Name]
        
        repeat 
            if not ore or not ore.Parent or not boosterEnabled then break end
            
            for i = 1, 3 do
                if ore and ore.Parent then
                    pcall(function()
                        ore.CFrame = moneyLoopItem.Model.Upgrade.CFrame
                    end)
                    task.wait(info.MinWait or 0.01)
                    
                    if info.Effect and protectItem then
                        pcall(function()
                            ore.CFrame = protectItem.Model.Upgrade.CFrame
                        end)
                    end
                end
            end
            task.wait(0.1)
        until not ore or not ore.Parent or not ore:FindFirstChild("Cash") or ore.Cash.Value >= info.Cap
    end
end

local function sellOre(ore)
    if not ore or not ore.Parent then return end
    
    pcall(function()
        ore.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
        ore.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
        
        if furnaceItem and furnaceItem:FindFirstChild("Model") and furnaceItem.Model:FindFirstChild("Lava") then
            ore.CFrame = furnaceItem.Model.Lava.CFrame + Vector3.new(0, 2, 0)
        end
    end)
end

local function startOreBoost(ore)
    if not ore or not ore.Parent then return end
    
    task.spawn(function()
        pcall(function()
            print("[DEBUG] Starting ore boost for:", ore.Name)
            
            -- Anchor ore initially
            ore.Anchored = true
            
            -- Wait for Cash value to load
            repeat 
                task.wait(0.01) 
            until ore:FindFirstChild("Cash") or not ore.Parent
            
            if not ore or not ore.Parent then return end
            
            ore.Anchored = false
            
            print("[DEBUG] Ore has Cash value:", ore.Cash.Value)
            
            -- Check if using Industrial Mine mode and if this is a coal mine ore
            if usingIndustrialMine and isCoalMineOre(ore) and industrialMineItem then
                print("[DEBUG] Sending to Industrial Mine")
                if sendOreToIndustrialMine(ore) then
                    return -- Exit here, industrial mine will create new ore
                end
            end
            
            -- Process money loopable if enabled
            if AutoUpgradeEnabled then
                print("[DEBUG] Processing money loopables")
                processMoneyLoopable(ore)
            end
            
            -- Process resetters if enabled
            if boosterEnabled then
                print("[DEBUG] Processing resetters")
                resetOre(ore)
            end
            
            -- Final boost before selling
            task.wait(0.1)
            if ore and ore.Parent then
                boostOre(ore)
            end
            
            -- Sell the ore
            task.wait(0.2) -- Give time for final processing
            if ore and ore.Parent then
                print("[DEBUG] Selling ore with final value:", ore.Cash and ore.Cash.Value or "NO CASH")
                sellOre(ore)
            end
        end)
    end)
end

local function detachAllFactoryConnections()
    for folder, conn in pairs(factoryConnections) do
        pcall(function() conn:Disconnect() end)
    end
    factoryConnections = {}
end

function StartOreBooster()
    if boosterEnabled then return end
    boosterEnabled = true
    
    tycoonFolder = findPlayerTycoon()
    if not tycoonFolder then
        warn("[OreBooster] Player tycoon not found.")
        return
    end
    
    getFurnaceItems()
    startFurnaceAutoDetection() -- Start auto-detection
    
    local droppedParts = workspace:FindFirstChild("DroppedParts")
    if not droppedParts then
        warn("[OreBooster] DroppedParts not found.")
        return
    end
    
    local playerFolder = droppedParts:FindFirstChild(tycoonFolder.Name)
    if playerFolder then
        factoryConnections["main"] = playerFolder.ChildAdded:Connect(function(ore)
            if boosterEnabled then
                startOreBoost(ore)
            end
        end)
        print("[OreBooster] Connected to player folder:", tycoonFolder.Name)
    else
        warn("[OreBooster] Player ore folder not found in DroppedParts")
    end
    
    -- Also watch for new items being added to tycoon (like placing new furnaces or industrial mines)
    factoryConnections["tycoonWatcher"] = tycoonFolder.ChildAdded:Connect(function(newItem)
        if newItem:FindFirstChild("Model") then
            task.wait(0.1) -- Wait for model to fully load
            newItem:SetAttribute("_OreBoosterWatched", true)
            
            -- Check if it's a furnace or industrial mine and update immediately
            if newItem.Model:FindFirstChild("Lava") and not newItem.Model.Lava:FindFirstChild("TeleportSend") then
                getFurnaceItems()
            end
            if usingIndustrialMine and newItem.Name:find("Industrial") and newItem.Model:FindFirstChild("Drop") then
                getFurnaceItems()
            end
            
            -- Watch for future changes to this item's model
            newItem.Model.ChildAdded:Connect(function(child)
                if child.Name == "Lava" and not child:FindFirstChild("TeleportSend") then
                    task.wait(0.1)
                    getFurnaceItems()
                end
            end)
        end
    end)
    
    print("[OreBooster] Started with automatic furnace detection" .. (usingIndustrialMine and " and Industrial Mine mode" or "") .. ".")
end

function StopOreBooster()
    if not boosterEnabled then return end
    boosterEnabled = false
    stopFurnaceAutoDetection() -- Stop auto-detection
    detachAllFactoryConnections()
    
    -- Clear attributes we set
    if tycoonFolder then
        for _, item in ipairs(tycoonFolder:GetChildren()) do
            if item:GetAttribute("_OreBoosterWatched") then
                item:SetAttribute("_OreBoosterWatched", nil)
            end
        end
    end
    
    print("[OreBooster] Stopped.")
end

-- =============================
-- PLAYER MANAGEMENT FUNCTIONS
-- =============================
local function findPlayerFactory()
    local tycoons = workspace:FindFirstChild("Tycoons")
    if not tycoons then return nil end
    local player = game.Players.LocalPlayer
    for _, f in ipairs(tycoons:GetChildren()) do
        local ownerVal = f:FindFirstChild("Owner")
        if ownerVal then
            local owner = ownerVal.Value
            if owner == player or tostring(owner) == player.Name then
                return f
            end
        end
    end
    return nil
end

local function applyAdjustSpeed(factoryFolder, speed)
    local adjustVal = factoryFolder:FindFirstChild("AdjustSpeed")
    if adjustVal and (adjustVal:IsA("NumberValue") or adjustVal:IsA("IntValue")) then
        adjustVal.Value = speed
        print("[CONVEYOR] AdjustSpeed set to", speed)
    else
        warn("[CONVEYOR] AdjustSpeed value not found inside factory!")
    end
end

local function scheduleFactoryWatch()
    if factoryWatchConn and factoryWatchConn.Connected then return end
    local tycoons = workspace:FindFirstChild("Tycoons")
    if not tycoons then
        factoryWatchConn = workspace.ChildAdded:Connect(function(child)
            if child.Name == "Tycoons" then
                pcall(function() UpdateConveyorSpeed(currentConveyorSpeed) end)
            end
        end)
        return
    end

    factoryWatchConn = tycoons.ChildAdded:Connect(function(folder)
        local ownerVal = folder:FindFirstChild("Owner")
        if ownerVal then
            local function tryApply()
                if ownerVal.Value == game.Players.LocalPlayer or tostring(ownerVal.Value) == game.Players.LocalPlayer.Name then
                    applyAdjustSpeed(folder, currentConveyorSpeed)
                    if factoryWatchConn then
                        factoryWatchConn:Disconnect()
                        factoryWatchConn = nil
                    end
                end
            end
            tryApply()
            ownerVal.Changed:Connect(tryApply)
        end
    end)
end

function UpdateConveyorSpeed(speed)
    currentConveyorSpeed = speed or currentConveyorSpeed
    local factory = findPlayerFactory()
    if factory then
        conveyorWarned = false
        applyAdjustSpeed(factory, currentConveyorSpeed)
    else
        if not conveyorWarned then
            warn("[CONVEYOR] Player factory not found; will watch for it and apply when available.")
            conveyorWarned = true
        end
        scheduleFactoryWatch()
    end
end

-- =============================
-- UNLOAD FUNCTION
-- =============================
local function unloadEverything()
    -- Disable all toggles
    oreTrackerEnabled, autoBoxesEnabled, autoOpenSelected, autoOpenAll, AutoUpgradeEnabled, boosterEnabled = false, false, false, false, false, false
    
    -- Cancel loops
    task.wait(0.15)
    for _, loop in pairs({trackerLoop, boxesLoop, autoBoxLoop}) do pcall(task.cancel, loop) end
    trackerLoop, boxesLoop, autoBoxLoop = nil, nil, nil
    
    -- Stop booster
    pcall(StopOreBooster)
    
    -- Clear cache and remove labels
    processed = setmetatable({}, {__mode = "k"})
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("BillboardGui") and obj.Name == "_OreLabel" then pcall(obj.Destroy, obj) end
    end
    
    -- Destroy UI
    if venyx then
        for _, method in pairs({"Destroy", "DestroyGui"}) do pcall(venyx[method], venyx) end
        for _, component in pairs({venyx.gui, venyx.window, venyx.main}) do 
            if typeof(component) == "Instance" then pcall(component.Destroy, component) end 
        end
    end
    
    -- Remove orphaned GUIs
    for _, gui in ipairs(game:GetService("CoreGui"):GetChildren()) do
        if gui:IsA("ScreenGui") then
            for _, d in ipairs(gui:GetDescendants()) do
                if d:IsA("TextLabel") and d.Text and d.Text:match("Prestige Hub") then
                    pcall(gui.Destroy, gui) break
                end
            end
        end
    end
    
    venyx, library = nil, nil
    print("[Prestige Hub] ✅ Fully unloaded and all connections disconnected.")
end

-- =============================
-- UI CREATION
-- =============================
local mainPage = venyx:addPage("Main", 5012544693)
local oreSection = mainPage:addSection("Ore Tracking")
local boosterSection = mainPage:addSection("Ore Booster")
local rebirthSection = mainPage:addSection("Auto Rebirth")
local RemoteSection = mainPage:addSection("Remote Clicker")
local conveyorSection = mainPage:addSection("Conveyors")

local layoutPage = venyx:addPage("Layouts", 5012544693)
local layoutManagerSection = layoutPage:addSection("Layout Management")
local secondLayoutSection = layoutPage:addSection("2nd Layout Setup")
local layoutStealSection = layoutPage:addSection("Layout Stealing")

local boxPage = venyx:addPage("Boxes", 5012544693)
local boxSection = boxPage:addSection("Mystery Boxes")

local playerTab = venyx:addPage("Player", 5012544693)
local playerSection = playerTab:addSection("Player Controls | Reapplies on character reset")

local ShopTab = venyx:addPage("Shop", 5012544693)
local TeleportTab = venyx:addPage("Teleport", 5012544693)

-- =============================
-- UI CONTROLS - AUTO REBIRTH
-- =============================

rebirthSection:addToggle("Enable Auto Rebirth", false, function(state)
    if state then
        startAutoRebirth()
    else
        stopAutoRebirth()
    end
end)

rebirthSection:addSlider("Wait for Skips", 0, 0, 20, function(value)
    minSkips = math.floor(value)
    print("[AUTO REBIRTH] Min skips set to:", minSkips)
end)

rebirthSection:addSlider("Rebirth Delay", 1, 0, 10, function(value)
    rebirthDelay = value
    print("[AUTO REBIRTH] Delay set to:", rebirthDelay)
end)

RemoteSection:addToggle("Auto Remote Clicker", false, function(value)
    RemoteClickerEnabled = value
    if value then
        startRemoteClicker()
    else
        stopRemoteClicker()
    end
end)

RemoteSection:addSlider("Remote Click Delay", 0, 0, 2, function(value)
    clickerDelay = value
    print("[Remote Clicker] Delay set to:", value)
end)

-- =============================
-- UI CONTROLS - ORE TRACKING
-- =============================
oreSection:addToggle("Ore Tracker", nil, function(value)
    oreTrackerEnabled = value
    if value then startOreTracker() else stopOreTracker() end
end)

oreSection:addSlider("Label Size", 50, 50, 300, function(value)
    labelSize = value
    updateAllLabelSizes()
    print("Label size changed to:", value)
end)

oreSection:addButton("Refresh Labels", function()
    if oreTrackerEnabled then
        stopOreTracker()
        oreTrackerEnabled = true
        startOreTracker()
        venyx:Notify("Ore Tracker", "Labels refreshed!")
    else
        venyx:Notify("Ore Tracker", "Enable Ore Tracker first!")
    end
end)

oreSection:addButton("Unload Tracker", function()
    oreTrackerEnabled = false
    stopOreTracker()
    venyx:Notify("Ore Tracker", "Completely unloaded!")
end)

-- =============================
-- UI CONTROLS - ORE BOOSTER
-- =============================
boosterSection:addToggle("Ore Booster", false, function(value)
    if value then
        StartOreBooster()
        venyx:Notify("Ore Booster", "Ores will now instantly teleport to your furnace!")
    else
        StopOreBooster()
        venyx:Notify("Ore Booster", "Ore Booster disabled.")
    end
end)

boosterSection:addToggle("Auto-Upgrade", false, function(value)
    AutoUpgradeEnabled = value
    if value then
        venyx:Notify("Auto-Upgrade", "Ores will now pass through money loopables first!")
    else
        venyx:Notify("Auto-Upgrade", "Skipping money loopables.")
    end
end)

boosterSection:addToggle("Using Industrial Mine", false, function(value)
    usingIndustrialMine = value
    if value then
        getFurnaceItems() -- Refresh to detect industrial mines
        venyx:Notify("Industrial Mine", "Coal mine ores will now be sent to Industrial Mine first!")
    else
        industrialMineItem = nil -- Clear the industrial mine reference
        venyx:Notify("Industrial Mine", "Industrial Mine mode disabled.")
    end
end)

boosterSection:addButton("Debug Industrial Mine", function()
    print("[Industrial Mine Debug] Scanning for Industrial Mines...")
    local tycoon = findPlayerTycoon()
    if not tycoon then
        print("[Industrial Mine Debug] No tycoon found!")
        return
    end
    
    print("[Industrial Mine Debug] Tycoon found:", tycoon.Name)
    print("[Industrial Mine Debug] Items in tycoon:")
    
    for _, item in ipairs(tycoon:GetChildren()) do
        local hasModel = item:FindFirstChild("Model") ~= nil
        local hasLava = hasModel and item.Model:FindFirstChild("Lava") ~= nil
        local hasDrop = hasModel and item.Model:FindFirstChild("Drop") ~= nil
        local nameCheck = item.Name:lower():find("industrial") or item.Name:lower():find("mine")
        
        print(string.format(" - %s | Model:%s | Lava:%s | Drop:%s | NameMatch:%s", 
            item.Name, tostring(hasModel), tostring(hasLava), tostring(hasDrop), tostring(nameCheck)))
        
        if hasModel and hasLava and hasDrop and nameCheck then
            print("   ^ THIS WOULD BE DETECTED AS INDUSTRIAL MINE!")
            if item.Model.Lava then
                print("   ^ Lava CFrame:", tostring(item.Model.Lava.CFrame))
            end
        end
    end
    
    if industrialMineItem then
        print("[Industrial Mine Debug] Current industrial mine:", industrialMineItem.Name)
    else
        print("[Industrial Mine Debug] No industrial mine currently detected")
    end
end)

-- =============================
-- UI CONTROLS - CONVEYORS
-- =============================
conveyorSection:addSlider("Conveyor Speed", currentConveyorSpeed, 1, 20, function(value)
    UpdateConveyorSpeed(value)
end)

-- =============================
-- AUTO LAYOUT SYSTEM (WITH LIFE VALUE DETECTION)
-- =============================

-- Variables
local selectedLayout1 = nil
local selectedLayout2 = nil
local layoutDelay = 0 -- delay before first layout loads
local layout2Delay = 5 -- delay before second layout loads
local autoLayoutEnabled = false
local lifeValue = nil

-- Services
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Layouts = ReplicatedStorage:WaitForChild("Layouts")
local DestroyAll = ReplicatedStorage:WaitForChild("DestroyAll")
local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- ========== CORE FUNCTIONS ==========
local function destroyAllItems()
    DestroyAll:InvokeServer()
    print("[AUTO LAYOUT] Destroyed all items.")
end

local function loadLayout(layoutName)
    if not layoutName then return end
    local args = {"Load", layoutName}
    Layouts:InvokeServer(unpack(args))
    print("[AUTO LAYOUT] Loaded layout:", layoutName)
end

local function runAutoLayoutLoop()
    if not autoLayoutEnabled or not selectedLayout1 then return end

    task.spawn(function()
        -- Step 1: Withdraw everything
        destroyAllItems()

        -- Step 2: Wait for the configured delay BEFORE loading the first layout
        task.wait(layoutDelay)

        -- Step 3: Load the first layout
        loadLayout(selectedLayout1)

        -- Step 4: If a second layout is selected, wait then load it
        if selectedLayout2 then
            task.wait(layout2Delay)
            destroyAllItems()
            loadLayout(selectedLayout2)
            print("[AUTO LAYOUT] Switched to second layout.")
        end
    end)
end

-- ========== LIFE VALUE REBIRTH DETECTION ==========
local function setupLifeListener()
    local leaderstats = player:WaitForChild("leaderstats")
    lifeValue = leaderstats:WaitForChild("Life")

    lifeValue:GetPropertyChangedSignal("Value"):Connect(function()
        if autoLayoutEnabled then
            print("[AUTO LAYOUT] Detected life change (rebirth), running auto-layout loop.")
            runAutoLayoutLoop()
        end
    end)
end

-- ========== INITIAL SETUP ==========
setupLifeListener()

-- ========== UI HOOKS ==========
layoutManagerSection:addButton("Withdraw All", function()
    destroyAllItems()
end)

layoutManagerSection:addDropdown("Auto Layout", {"Layout 1", "Layout 2", "Layout 3"}, function(selected)
    selectedLayout1 = selected:gsub("Layout ", "Layout")
    print("[AUTO LAYOUT] Selected Layout 1:", selectedLayout1)

    if autoLayoutEnabled then
        runAutoLayoutLoop() -- Immediately trigger auto-layout on selecting Layout 1
    end
end)

layoutManagerSection:addToggle("Enable Auto Layout", false, function(state)
    autoLayoutEnabled = state
    print("[AUTO LAYOUT] Auto layout enabled:", autoLayoutEnabled)

    if autoLayoutEnabled then
        runAutoLayoutLoop() -- Run once on enabling
    end
end)

layoutManagerSection:addSlider("Layout Delay", 0, 0, 30, function(value)
    layoutDelay = value
    print("[AUTO LAYOUT] Layout 1 delay set to:", layoutDelay)
end)

secondLayoutSection:addDropdown("Auto 2nd Layout", {"Layout 1", "Layout 2", "Layout 3"}, function(selected)
    selectedLayout2 = selected:gsub("Layout ", "Layout")
    print("[AUTO LAYOUT] Selected Layout 2:", selectedLayout2)

    if autoLayoutEnabled and selectedLayout1 then
        -- If we already have a first layout selected and auto layout is on, run loop immediately
        runAutoLayoutLoop()
    end
end)

secondLayoutSection:addSlider("2nd Layout Delay", 5, 0, 30, function(value)
    layout2Delay = value
    print("[AUTO LAYOUT] Layout 2 delay set to:", layout2Delay)
end)

-- === Layout Steal / Load integration (venyx) - Updated Version ===
local playerDropdown

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HTTP = game:GetService("HttpService")

-- safe remotes (may be nil if names changed)
local HasItem = ReplicatedStorage:FindFirstChild("HasItem")
local DestroyAll = ReplicatedStorage:FindFirstChild("DestroyAll") or ReplicatedStorage:FindFirstChild("Withdraw") -- fallback names
local PlaceItem = ReplicatedStorage:FindFirstChild("PlaceItem")
local buyItem = ReplicatedStorage:FindFirstChild("BuyItem") or ReplicatedStorage:FindFirstChild("BuyItemFromShop") -- fallback
local ItemsFolder = ReplicatedStorage:FindFirstChild("Items")

local Player = Players.LocalPlayer

local function safeNotify(title, text)
    if venyx and type(venyx.Notify) == "function" then
        pcall(function() venyx:Notify(title, text) end)
    else
        print("["..tostring(title).."]", text)
    end
end

-- Updated item finding function based on working code
local function findItemByName(name)
    if not ItemsFolder then return nil end
    -- try direct child by name first
    local child = ItemsFolder:FindFirstChild(name)
    if child then return child end
    -- fallback: search by ItemId property if name doesn't match
    for _, v in ipairs(ItemsFolder:GetChildren()) do
        if v:FindFirstChild("ItemId") then
            if tostring(v.Name) == tostring(name) then return v end
            -- sometimes stolen layout has names/IDs mismatch: check numeric id match
            if tonumber(name) and tonumber(v.ItemId.Value) == tonumber(name) then
                return v
            end
        end
    end
    return nil
end

-- New function to find item by ID (from working code)
local function findItemById(id)
    if not ItemsFolder then return nil end
    for _, v in ipairs(ItemsFolder:GetChildren()) do
        if v:FindFirstChild("ItemId") and v.ItemId.Value == tonumber(id) then
            return v
        end 
    end
    return nil
end

-- build player list for dropdown
local playerNames = {}
for _, plr in ipairs(Players:GetPlayers()) do
    table.insert(playerNames, plr.Name)
end

local selectedPlayer = nil
local stolenLayout = nil -- table of layout data

-- create dropdown with your venyx section variable (layoutStealSection must exist)
local playerDropdown = layoutStealSection:addDropdown("Pick a player", playerNames, function(selected)
    selectedPlayer = selected
    print("[LAYOUT STEAL] Selected player:", selected)
end)

-- update dropdown as players join/leave
Players.PlayerAdded:Connect(function(plr)
    table.insert(playerNames, plr.Name)
    if playerDropdown and playerDropdown.update then
        pcall(function() playerDropdown:update(playerNames) end)
    end
end)

Players.PlayerRemoving:Connect(function(plr)
    for i, name in ipairs(playerNames) do
        if name == plr.Name then
            table.remove(playerNames, i)
            break
        end
    end
    if playerDropdown and playerDropdown.update then
        pcall(function() playerDropdown:update(playerNames) end)
    end
end)

-- Updated capture function based on working code
local function captureLayoutFromBase(tycoon)
    local data = {}
    local basePart = tycoon:FindFirstChild("Base")

    if not basePart or not basePart:IsA("BasePart") then
        warn("[LAYOUT STEAL] Base part not found in tycoon:", tycoon.Name)
        return data
    end

    for _, item in ipairs(tycoon:GetChildren()) do
        if item:FindFirstChild("ItemId") and item:FindFirstChild("Hitbox") then
            local hitbox = item.Hitbox

            -- relative to the base part's position
            local relPos = hitbox.CFrame.Position - basePart.Position
            local lookVector = hitbox.CFrame.LookVector

            table.insert(data, {
                ItemId = item.ItemId.Value,
                Position = {
                    relPos.X,
                    relPos.Y, 
                    relPos.Z,
                    lookVector.X,
                    lookVector.Y,
                    lookVector.Z
                }
            })
        end
    end
    return data
end

layoutStealSection:addButton("Steal Layout", function()
    if not selectedPlayer then
        safeNotify("Layout Steal", "No player selected.")
        return
    end

    local plr = Players:FindFirstChild(selectedPlayer)
    if not plr then
        safeNotify("Layout Steal", "Selected player not found on server.")
        return
    end

    if not plr:FindFirstChild("PlayerTycoon") or not plr.PlayerTycoon.Value then
        safeNotify("Layout Steal", "Player has no loaded tycoon.")
        return
    end

    local tycoon = plr.PlayerTycoon.Value
    local base = tycoon:FindFirstChild("Base")
    if not base then
        safeNotify("Layout Steal", "Player tycoon has no Base object.")
        return
    end

    -- Check if player has BaseDataLoaded (from working code)
    if not plr:FindFirstChild("BaseDataLoaded") then
        safeNotify("Layout Steal", "Player is not fully loaded in. Please wait or select another player.")
        return
    end

    local data = captureLayoutFromBase(tycoon)

    if #data == 0 then
        safeNotify("Layout Steal", "No placeable items found in player's base.")
        return
    end

    stolenLayout = data
    local encoded = nil
    pcall(function() encoded = HTTP:JSONEncode(data) end)
    if encoded then
        pcall(function()
            if setclipboard then setclipboard(encoded) end
        end)
    end

    safeNotify("Layout Steal", "Stole layout of "..selectedPlayer.." ("..tostring(#data).." items). Copied to clipboard (if available).")
    print("[LAYOUT STEAL] Stolen layout for "..selectedPlayer..": items="..tostring(#data))
end)

-- Updated load function based on working code
local function loadStolenLayout(layoutData)
    local me = Players.LocalPlayer
    if not me then
        safeNotify("Layout Load", "Local player not found.")
        return
    end

    if not me:FindFirstChild("PlayerTycoon") or not me.PlayerTycoon.Value then
        safeNotify("Layout Load", "You don't have a tycoon loaded. Join the world / load base first.")
        return
    end

    if not me:FindFirstChild("BaseDataLoaded") then
        safeNotify("Layout Load", "You need to be fully loaded in first.")
        return
    end

    local myTycoon = me.PlayerTycoon.Value
    local myBase = myTycoon:FindFirstChild("Base")
    if not myBase then
        safeNotify("Layout Load", "Your tycoon base not found.")
        return
    end

    -- withdraw everything first
    if DestroyAll and DestroyAll:IsA("RemoteFunction") then
        pcall(function() DestroyAll:InvokeServer() end)
    elseif DestroyAll and DestroyAll:IsA("RemoteEvent") then
        pcall(function() DestroyAll:FireServer() end)
    else
        warn("[Layout Load] DestroyAll remote not found or unknown type.")
    end

    wait(0.5) -- Give time for withdrawal

    local failed = {}
    local placedCount = 0

    for _, itemData in ipairs(layoutData) do
        spawn(function()
            pcall(function()
                local realItem = findItemById(itemData.ItemId)
                if not realItem then
                    table.insert(failed, "Item ID "..tostring(itemData.ItemId).." (Unknown item)")
                    return
                end

                -- Calculate position exactly like working code
                local tycoonBase = myBase
                local relativePos = Vector3.new(itemData.Position[1], itemData.Position[2], itemData.Position[3])
                local position = tycoonBase.Position + relativePos
                local lookVector = Vector3.new(itemData.Position[4], itemData.Position[5], itemData.Position[6])
                local coordinateFrame = CFrame.new(position, position + (lookVector * 5))

                -- Check item type and handle accordingly (from working code)
                if realItem.ItemType.Value >= 1 and realItem.ItemType.Value < 5 then
                    -- Shop item - needs to be bought
                    local moneyVal = me.PlayerGui and me.PlayerGui:FindFirstChild("GUI") and me.PlayerGui.GUI:FindFirstChild("Money")
                    local currentMoney = (moneyVal and tonumber(moneyVal.Value)) or 0
                    local cost = (realItem:FindFirstChild("Cost") and realItem.Cost.Value) or 0
                    
                    if currentMoney >= cost then
                        if buyItem and buyItem:IsA("RemoteFunction") then
                            pcall(function() buyItem:InvokeServer(realItem.Name, 1) end)
                        elseif buyItem and buyItem:IsA("RemoteEvent") then
                            pcall(function() buyItem:FireServer(realItem.Name, 1) end)
                        end
                        
                        if PlaceItem and PlaceItem:IsA("RemoteFunction") then
                            pcall(function() PlaceItem:InvokeServer(realItem.Name, coordinateFrame, {myBase}) end)
                            placedCount = placedCount + 1
                        else
                            table.insert(failed, realItem.Name.." (PlaceItem missing)")
                        end
                    else
                        table.insert(failed, realItem.Name.." (Insufficient money)")
                    end
                else
                    -- Regular item - check if owned
                    local hasCount = 0
                    if HasItem and HasItem:IsA("RemoteFunction") then
                        pcall(function() hasCount = HasItem:InvokeServer(realItem.ItemId.Value) end)
                    end
                    
                    if hasCount and tonumber(hasCount) > 0 then
                        if PlaceItem and PlaceItem:IsA("RemoteFunction") then
                            pcall(function() PlaceItem:InvokeServer(realItem.Name, coordinateFrame, {myBase}) end)
                            placedCount = placedCount + 1
                        else
                            table.insert(failed, realItem.Name.." (PlaceItem missing)")
                        end
                    else
                        table.insert(failed, realItem.Name.." (Don't own item)")
                    end
                end
                
                wait(0.1) -- Small delay between placements
            end)
        end)
    end

    wait(3) -- Give time for all spawned functions to complete

    if #failed > 0 then
        safeNotify("Layout Load", "Loaded with "..tostring(#failed).." issues. Check console for details.")
        warn("[LAYOUT LOAD] Failed items:")
        for _, failedItem in ipairs(failed) do
            warn(" - " .. failedItem)
        end
    else
        safeNotify("Layout Load", "Layout loaded successfully: placed "..tostring(placedCount).." items.")
    end
end

-- load stolen layout button - updated
layoutStealSection:addButton("Load Stolen Layout", function()
    if not stolenLayout then
        safeNotify("Layout Load", "No stolen layout stored. Use 'Steal Layout' first.")
        return
    end

    loadStolenLayout(stolenLayout)
end)

-- Add button to load layout from clipboard
layoutStealSection:addButton("Load Layout from Clipboard", function()
    local success, clipboardData = pcall(function()
        if getclipboard then
            return getclipboard()
        else
            error("Clipboard not available")
        end
    end)
    
    if not success or not clipboardData then
        safeNotify("Layout Load", "Could not get clipboard data or clipboard not available.")
        return
    end
    
    local layoutData = nil
    pcall(function()
        layoutData = HTTP:JSONDecode(clipboardData)
    end)
    
    if not layoutData or type(layoutData) ~= "table" then
        safeNotify("Layout Load", "Invalid layout data in clipboard.")
        return
    end
    
    stolenLayout = layoutData
    safeNotify("Layout Load", "Layout loaded from clipboard ("..tostring(#layoutData).." items). Use 'Load Stolen Layout' to place.")
    loadStolenLayout(layoutData)
end)

-- =============================
-- UI CONTROLS - TELEPORTS
-- =============================

local TeleportSection = TeleportTab:addSection("Teleport To NPC'S")

local npcLocations = {
    ["Wizard"] = CFrame.new(-146.7, 219.5, 209.64), -- placeholder coords
    ["Spook McDooks"] = CFrame.new(-356.009765625, 58.21588897705078, 1329.711669921875),
    ["Fargield"] = CFrame.new(-325.82, 102.571, 533.412),
    ["John Doe"] = CFrame.new(-723.939453125, 40.55, -34.37),
    ["Zalgo"] = CFrame.new(-5341.048828125, 179.54486083984375, 14295.0166015625),
    ["a nice view (not a npc)"] = CFrame.new(-318.112, 604.63, -3202.3142),
}

local npcNames = {}
for name, _ in pairs(npcLocations) do
    table.insert(npcNames, name)
end

local selectedNPC = nil

local npcDropdown = TeleportSection:addDropdown("Choose NPC", npcNames, function(selected)
    selectedNPC = selected
    print("[TELEPORT] Selected NPC:", selectedNPC)
end)

TeleportSection:addButton("Teleport", function()
    if selectedNPC and npcLocations[selectedNPC] then
        local player = game.Players.LocalPlayer
        local char = player.Character or player.CharacterAdded:Wait()
        local hrp = char:WaitForChild("HumanoidRootPart")

        hrp.CFrame = npcLocations[selectedNPC]
        print("[TELEPORT] Teleported to:", selectedNPC)
    else
        warn("[TELEPORT] No NPC selected or no location set.")
    end
end)

TeleportSection:addButton("Teleport to My Factory", function()
    local factory = findPlayerFactory()
    if factory and factory:FindFirstChild("Base") then
        local base = factory:FindFirstChild("Base")
        local player = game.Players.LocalPlayer
        local char = player.Character or player.CharacterAdded:Wait()
        local hrp = char:WaitForChild("HumanoidRootPart")

        hrp.CFrame = base.CFrame + Vector3.new(0, 10, 0) -- spawn slightly above base
        print("[TELEPORT] Teleported to your factory.")
    else
        warn("[TELEPORT] Could not find your factory.")
    end
end)

-- =============================
-- UI CONTROLS - SHOP
-- =============================

local automationSection = ShopTab:addSection("WILL SPEND RP (I didn't add EVERY blueprint, but there are 60+)") 

automationSection:addButton("Buy All Blueprints", function()
    local blueprintIDs = {
        667, 494, 873, 877, 876, 872, 477, 1777, 436, 881, 600, 587, 588, 880,
        693, 472, 533, 736, 878, 525, 670, 437, 517, 556, 1442, 874, 674, 439,
        534, 473, 433, 438, 875, 527, 440, 435, 706, 643, 668, 671, 845, 879,
        1136, 665, 596, 1411, 871, 1062, 442, 1141, 1172, 1170, 1179, 1219,
        1217, 1216, 1214, 1212, 1225, 1224, 1140, 1708, 1783, 1781, 1181
    }
    local remote = game:GetService("ReplicatedStorage"):WaitForChild("CraftsmanEvents")
    for _, id in ipairs(blueprintIDs) do
        local args = {"type:buyblueprint", id}
        remote:InvokeServer(unpack(args))
        task.wait(0.1) -- small delay to avoid flooding
    end
    venyx:Notify("Blueprint AutoBuyer", "Attempted to buy all listed blueprints.")
end)

local shopMenus = ShopTab:addSection("Open Interaction Menus")

local shopGUIs = {
    ["Craftsman (Click Again to Close)"] = game.Players.LocalPlayer.PlayerGui.GUI:WaitForChild("Craftsman"),
    ["Fleabag (Click Again to Close)"] = game.Players.LocalPlayer.PlayerGui.GUI:WaitForChild("Fleabag"),
    ["Superstitious Crafting (Click Again to Close)"] = game.Players.LocalPlayer.PlayerGui.GUI:WaitForChild("SuperstitiousCrafting"),
    ["Spook McDook"] = game.Players.LocalPlayer.PlayerGui.GUI:WaitForChild("SpookMcDookShop"),
    ["Event Shop"] = game.Players.LocalPlayer.PlayerGui.GUI:WaitForChild("EventShop"),
}

local function enableCloseButton(gui)
    local topFrame = gui:FindFirstChild("Top")
    if topFrame then
        local pc = topFrame:FindFirstChild("PC")
        local mobile = topFrame:FindFirstChild("Mobile")
        if pc then pc.Value = true end
        if mobile then mobile.Value = true end

        local closeButton = topFrame:FindFirstChild("Close")
        if closeButton then
            closeButton.Visible = true
            closeButton.Active = true
        end
    end
end

for name, gui in pairs(shopGUIs) do
    shopMenus:addButton("Open " .. name, function()
        if name:find("Click Again to Close") then
            -- Toggle visibility manually for these menus
            gui.Visible = not gui.Visible
            print("[SHOP] Toggled:", name, "->", gui.Visible and "OPEN" or "CLOSED")
        else
            -- Normal open (X button should work)
            enableCloseButton(gui)
            gui.Visible = true
            print("[SHOP] Opened:", name)
        end
    end)
end

-- =============================
-- UI CONTROLS - PLAYER
-- =============================

local player = game.Players.LocalPlayer
local playerSpeed = 16
local playerJump = 50

local function applyPlayerStats()
    local char = player.Character
    if char and char:FindFirstChildOfClass("Humanoid") then
        char:FindFirstChildOfClass("Humanoid").WalkSpeed = playerSpeed
        char:FindFirstChildOfClass("Humanoid").JumpPower = playerJump
    end
end

player.CharacterAdded:Connect(function()
    task.wait(0.1)
    applyPlayerStats()
end)

playerSection:addSlider("WalkSpeed", 16, 16, 200, function(value)
    playerSpeed = value
    applyPlayerStats()
end)

playerSection:addSlider("JumpPower", 50, 50, 300, function(value)
    playerJump = value
    applyPlayerStats()
end)

local antiAFK = false
local afkConnection
playerSection:addToggle("Anti-AFK", false, function(state)
    antiAFK = state
    if antiAFK then
        print("[PLAYER] Anti-AFK enabled")
        afkConnection = game:GetService("Players").LocalPlayer.Idled:Connect(function()
            game.VirtualUser:Button2Down(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
            task.wait(1)
            game.VirtualUser:Button2Up(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
        end)
    else
        print("[PLAYER] Anti-AFK disabled")
        if afkConnection then afkConnection:Disconnect() end
    end
end)

-- =============================
-- UI CONTROLS - BOXES
-- =============================
boxSection:addToggle("Auto Collect Boxes", false, function(value)
    autoBoxesEnabled = value
    if value then
        startAutoBoxes()
        venyx:Notify("Auto Boxes", "Auto Collect Boxes Enabled!")
    else
        stopAutoBoxes()
        venyx:Notify("Auto Boxes", "Auto Collect Boxes Disabled!")
    end
end)

boxSection:addDropdown("Select Box", boxNames, function(value)
    selectedBox = value
    print("Selected box:", selectedBox)
end)

boxSection:addButton("Open Selected Box", function()
    openBox(selectedBox)
end)

boxSection:addToggle("Auto Open Selected Box", false, function(v)
    autoOpenSelected = v
    if v then startAutoBoxLoop() else stopAutoBoxLoop() end
end)

boxSection:addToggle("Auto Open All Boxes", false, function(v)
    autoOpenAll = v
    if v then startAutoBoxLoop() else stopAutoBoxLoop() end
end)

-- =============================
-- UI CONTROLS - THEME
-- =============================
local theme = venyx:addPage("Theme", 5012544693)
local colors = theme:addSection("Colors")
for themeName, color in pairs(themes) do
    colors:addColorPicker(themeName, color, function(color3)
        venyx:setTheme(themeName, color3)
    end)
end

-- =============================
-- UI CONTROLS - SETTINGS
-- =============================
local settings = venyx:addPage("Settings", 5012544693)
local controlSection = settings:addSection("Script Controls")

controlSection:addButton("Unload Script", function()
    unloadEverything()
end)

-- =============================
-- FINALIZATION
-- =============================

venyx:SelectPage(venyx.pages[1], true)
print("[Prestige Hub] Loaded with Industrial Mine support.")
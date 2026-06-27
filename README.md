-- ============================================================
-- MINI-WAR HUB | Batman Script
-- ============================================================
-- CONFIG: แก้ตรงนี้ได้เลย
-- ============================================================
getgenv().MiniWarConfig = getgenv().MiniWarConfig or {
    -- =================== [ Auto Buy ] ===================
    AutoBuy      = true, -- true = เปิด auto buy
    BuyInterval  = 15,     -- วินาทีระหว่างซื้อแต่ละรอบ

    BuyItems = {          -- ชื่อ item ที่ต้องการซื้อ
        "Data Center",
        "Blackhole Generator",
        "Area51Lab",
        -- "AntimatterReactor",
        -- "QuantumCoreGenerator",
        -- "SupernovaAccelerator",
        -- "Artillery Depot",
        "Rocket Bunker",
        "MechStation",
        -- "SpiderBase",
        -- "AirFortress",
        "GemMine",
        -- "TransportPad",
        -- "ConstructionSpecial",
        -- "CloneFacility",
        -- "CloneFacilityV2",
    },

    -- =================== [ Auto Attack ] ===================
    AutoAttack   = true, -- true = เปิด auto attack
    AttackDelay  = 120,    -- วินาทีก่อนไป target ถัดไป
    ArmyIndex    = 1,     -- army index ที่ใช้โจมตี

    PriorityQueue = {     -- target ตามลำดับที่ต้องการโจมตี
		"Special",
        "MilitaryTown/Town",
        "Laboratory/Laboratory",
        "Laboratory/Laboratory1",
        "Laboratory/Laboratory2",
        "MilitaryBase/MilitaryBase1",
        "MilitaryBase/MilitaryBase2",
        "MilitaryBase/MilitaryBase3",
        "MilitaryBase/MilitaryBase4",
        "WaterRigs/WaterRig1",
        -- "Bandit/Garnison1",
        -- "Bandit/Garnison2",
        -- "MilitaryBase/MilitaryBase1",
    },

    -- =================== [ Webhook ] ===================
    WebhookUrl     = "https://discord.com/api/webhooks/1520139298605891765/eGYUxALsSWLiKmKkZZOnmxEjY58SWNq36NwZ_QMEHAHCH6rZ4J6Iwj4R9aGgX7EweKi0", -- Discord webhook URL
    WebhookEnabled = true,
}

-- ============================================================
local HttpService    = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players        = game:GetService("Players")
local RunService     = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer  = Players.LocalPlayer
local HttpRequest  = request or http_request or (syn and syn.request)
assert(HttpRequest, "No HTTP support")

local Config = getgenv().MiniWarConfig

-- convert BuyItems array -> set
local buyItemsSet = {}
for _, name in ipairs(Config.BuyItems or {}) do
    buyItemsSet[name] = true
end

local function fetch(url)
    local ok, res = pcall(function()
        return HttpRequest({ Url = url, Method = "GET" })
    end)
    assert(ok and res and res.Body, "Fetch failed: " .. url)
    return res.Body
end

local Fluent           = loadstring(fetch("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager      = loadstring(fetch("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(fetch("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

local Remote = ReplicatedStorage
    :WaitForChild("ncxyzero_bridgenet2-fork@1.1.5")
    :WaitForChild("dataRemoteEvent")

local unpackArgs = table.unpack or unpack
local function fireBridge(data, code)
    Remote:FireServer(unpackArgs({ { data, code } }))
end

-- ============================================================
-- WINDOW
-- ============================================================
local Window = Fluent:CreateWindow({
    Title       = "Mini-War Hub",
    SubTitle    = "Wave Script",
    TabWidth    = 160,
    Size        = UDim2.fromOffset(630, 540),
    Acrylic     = false,
    Theme       = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    Main   = Window:AddTab({ Title = "Main",   Icon = "swords"   }),
    Player = Window:AddTab({ Title = "Player", Icon = "user"     }),
    Log    = Window:AddTab({ Title = "Log",    Icon = "scroll"   }),
    Config = Window:AddTab({ Title = "Config", Icon = "settings" }),
}

local Options = Fluent.Options

-- ============================================================
-- LOG
-- ============================================================
local logLines     = {}
local LogParagraph = Tabs.Log:AddParagraph({ Title = "Activity Log", Content = "No activity yet." })

local function addLog(msg)
    local time = os.date("%H:%M:%S")
    table.insert(logLines, 1, "[" .. time .. "] " .. msg)
    if #logLines > 40 then table.remove(logLines) end
    LogParagraph:SetDesc(table.concat(logLines, "\n"))
end

Tabs.Log:AddButton({ Title = "Clear Log", Callback = function()
    table.clear(logLines)
    LogParagraph:SetDesc("Cleared.")
end })

-- ============================================================
-- BUY DATA
-- ============================================================
local BuyItems = {
    ["Data Center"]          = { shop = "Farm",        code = "\031" },
    ["Blackhole Generator"]  = { shop = "Farm",        code = "\031" },
    ["Area51Lab"]            = { shop = "Farm",        code = "\031" },
    ["AntimatterReactor"]    = { shop = "Farm",        code = "\031" },
    ["QuantumCoreGenerator"] = { shop = "Farm",        code = "\031" },
    ["SupernovaAccelerator"] = { shop = "Farm",        code = "\031" },
    ["Artillery Depot"]      = { shop = "Military",    code = "\031" },
    ["Rocket Bunker"]        = { shop = "Military",    code = "\031" },
    ["MechStation"]          = { shop = "Military",    code = "\031" },
    ["SpiderBase"]           = { shop = "Military",    code = "\031" },
    ["AirFortress"]          = { shop = "Military",    code = "\031" },
    ["GemMine"]              = { shop = "BlackMarket", code = "M"    },
    ["TransportPad"]         = { shop = "BlackMarket", code = "M"    },
    ["ConstructionSpecial"]  = { shop = "BlackMarket", code = "M"    },
    ["CloneFacility"]        = { shop = "BlackMarket", code = "M"    },
    ["CloneFacilityV2"]      = { shop = "BlackMarket", code = "M"    },
}

local BuyItemNames = {
    "Data Center","Blackhole Generator","Area51Lab","AntimatterReactor",
    "QuantumCoreGenerator","SupernovaAccelerator","Artillery Depot",
    "Rocket Bunker","MechStation","SpiderBase","AirFortress",
    "GemMine","TransportPad","ConstructionSpecial","CloneFacility","CloneFacilityV2"
}

-- ============================================================
-- WEBHOOK
-- ============================================================
local function sendWebhook(itemName, shop)
    if Config.WebhookUrl == "" then return end
    if not Config.WebhookEnabled then return end
    task.spawn(function()
        pcall(function()
            HttpRequest({
                Url     = Config.WebhookUrl,
                Method  = "POST",
                Headers = { ["Content-Type"] = "application/json" },
                Body    = HttpService:JSONEncode({
                    username = "Mini-War Hub",
                    embeds   = {{
                        title  = "Auto Buy",
                        fields = {
                            { name = "Item", value = itemName, inline = true },
                            { name = "Shop", value = shop,     inline = true },
                        },
                        timestamp = DateTime.now():ToIsoDate()
                    }}
                })
            })
        end)
    end)
end

-- ============================================================
-- BUY FUNCTIONS
-- ============================================================
local buyLoopRunning = false

local function buyItem(itemName)
    local data = BuyItems[itemName]
    if not data then addLog("❌ Unknown item: " .. itemName) return end
    local ok, err = pcall(fireBridge, { item = itemName, shop = data.shop }, data.code)
    if ok then
        addLog("🛒 Bought: " .. itemName .. " [" .. data.shop .. "]")
        sendWebhook(itemName, data.shop)
    else
        addLog("❌ Buy error: " .. tostring(err))
    end
end

local function buyAllSelected()
    local count = 0
    -- ซื้อจาก buyItemsSet (config) + UI toggle
    local merged = {}
    for name in pairs(buyItemsSet) do merged[name] = true end
    if Options.BuyItem and Options.BuyItem.Value then
        for name, state in pairs(Options.BuyItem.Value) do
            if state then merged[name] = true end
        end
    end
    for name in pairs(merged) do
        buyItem(name)
        count += 1
        task.wait(0.3)
    end
    if count == 0 then addLog("⚠️ No items selected") end
end

-- ============================================================
-- MAIN — AUTO BUY UI
-- ============================================================
Tabs.Main:AddSection("Auto Buy")

Tabs.Main:AddDropdown("BuyItem", {
    Title       = "Select Items",
    Description = "เลือกได้หลายชิ้น",
    Values      = BuyItemNames,
    Multi       = true,
    Default     = {},
}):OnChanged(function(value)
    -- merge กับ config
    for name, state in pairs(value) do
        buyItemsSet[name] = state
    end
    local list = {}
    for name, state in pairs(buyItemsSet) do
        if state then table.insert(list, name) end
    end
    addLog("📋 Selected: " .. (#list > 0 and table.concat(list, ", ") or "none"))
end)

Tabs.Main:AddSlider("BuyInterval", {
    Title    = "Buy Interval (s)",
    Default  = Config.BuyInterval,
    Min = 1, Max = 60, Rounding = 0,
    Callback = function(v) Config.BuyInterval = v end
})

Tabs.Main:AddButton({
    Title       = "Buy Once",
    Description = "ซื้อทุกชิ้นที่เลือกทันที",
    Callback    = function() buyAllSelected() end
})

local AutoBuyToggle = Tabs.Main:AddToggle("AutoBuy", {
    Title   = "Auto Buy Loop",
    Default = Config.AutoBuy,
})

AutoBuyToggle:OnChanged(function()
    local val = Options.AutoBuy.Value
    Config.AutoBuy = val
    if val then
        if buyLoopRunning then return end
        buyLoopRunning = true
        addLog("▶️ Auto Buy started (every " .. Config.BuyInterval .. "s)")
        task.spawn(function()
            while Options.AutoBuy.Value and not Fluent.Unloaded do
                buyAllSelected()
                task.wait(Config.BuyInterval)
            end
            buyLoopRunning = false
            addLog("⏹️ Auto Buy stopped")
        end)
    else
        addLog("⏹️ Auto Buy stopping...")
    end
end)

-- ============================================================
-- ATTACK DATA
-- ============================================================
local TargetNames = {}
local TargetPaths = {}

local function addTarget(display, path)
    table.insert(TargetNames, display)
    TargetPaths[display] = path
end

for i = 1, 6 do addTarget("Bandit/Garnison"..i,          { "Bandit",       "Garnison"..i      }) end
addTarget("Laboratory/Laboratory",                         { "Laboratory",   "Laboratory"       })
for i = 1, 5 do addTarget("Laboratory/Laboratory"..i,     { "Laboratory",   "Laboratory"..i    }) end
for i = 1, 6 do addTarget("MilitaryBase/MilitaryBase"..i, { "MilitaryBase", "MilitaryBase"..i  }) end
addTarget("MilitaryTown/Town",                             { "MilitaryTown", "Town"             })
addTarget("Special",                                       { "Special"                          })
for i = 1, 6 do addTarget("WaterRigs/WaterRig"..i,        { "WaterRigs",    "WaterRig"..i      }) end
for i = 1, 6 do addTarget("Wooden/Garnison"..i,           { "Wooden",       "Garnison"..i      }) end

-- ============================================================
-- ATTACK FUNCTIONS
-- ============================================================
local selectedTarget    = TargetNames[1]
local attackLoopRunning = false
local priorityQueue     = {}

-- โหลด queue จาก config
for _, name in ipairs(Config.PriorityQueue or {}) do
    table.insert(priorityQueue, name)
end

local function getObjectRoot()
    local map = workspace:FindFirstChild("MilitaryMap")
    return map and map:FindFirstChild("Object")
end

local function resolveTargets(targetName)
    local root = getObjectRoot()
    if not root then addLog("❌ MilitaryMap not found") return {} end
    if targetName == "Special" then
        local folder = root:FindFirstChild("Special")
        if not folder then return {} end
        local t = {}
        for _, c in ipairs(folder:GetChildren()) do table.insert(t, c) end
        return t
    end
    local path = TargetPaths[targetName]
    if not path then return {} end
    local cur = root
    for _, name in ipairs(path) do
        cur = cur:FindFirstChild(name)
        if not cur then addLog("❌ Not found: " .. name) return {} end
    end
    return { cur }
end

local function attackTarget(targetName)
    local targets = resolveTargets(targetName)
    if #targets == 0 then addLog("⚠️ No target: " .. targetName) return end
    for _, target in ipairs(targets) do
        local ok, err = pcall(fireBridge, { armyIndex = Config.ArmyIndex, capturePoint = target }, "V")
        if ok then
            addLog("⚔️ Attack: " .. targetName .. " [army " .. Config.ArmyIndex .. "]")
        else
            addLog("❌ Attack error: " .. tostring(err))
        end
        task.wait(0.2)
    end
end

-- ============================================================
-- MAIN — AUTO ATTACK UI
-- ============================================================
Tabs.Main:AddSection("Auto Attack")

local PriorityText = Tabs.Main:AddParagraph({ Title = "Priority Queue", Content = "Empty" })

local function refreshQueue()
    if #priorityQueue == 0 then
        PriorityText:SetDesc("Empty")
    else
        local lines = {}
        for i, v in ipairs(priorityQueue) do lines[i] = i .. ". " .. v end
        PriorityText:SetDesc(table.concat(lines, "\n"))
    end
end

refreshQueue()

Tabs.Main:AddDropdown("AttackTarget", {
    Title  = "Target",
    Values = TargetNames,
    Multi  = false,
    Default = 1,
}):OnChanged(function(v) selectedTarget = v end)

Tabs.Main:AddButton({ Title = "Add To Queue", Callback = function()
    for _, v in ipairs(priorityQueue) do
        if v == selectedTarget then
            Fluent:Notify({ Title = "Queue", Content = "Already in queue", Duration = 2 })
            return
        end
    end
    table.insert(priorityQueue, selectedTarget)
    addLog("➕ Queue: " .. selectedTarget)
    refreshQueue()
end })

Tabs.Main:AddButton({ Title = "Remove Last", Callback = function()
    local r = table.remove(priorityQueue)
    if r then addLog("➖ Removed: " .. r) end
    refreshQueue()
end })

Tabs.Main:AddButton({ Title = "Clear Queue", Callback = function()
    table.clear(priorityQueue)
    addLog("🗑️ Queue cleared")
    refreshQueue()
end })

Tabs.Main:AddSlider("AttackDelay", {
    Title    = "Delay Per Target (s)",
    Default  = Config.AttackDelay,
    Min = 1, Max = 180, Rounding = 0,
    Callback = function(v) Config.AttackDelay = v end
})

Tabs.Main:AddSlider("ArmyIndex", {
    Title    = "Army Index",
    Default  = Config.ArmyIndex,
    Min = 1, Max = 10, Rounding = 0,
    Callback = function(v) Config.ArmyIndex = v end
})

Tabs.Main:AddButton({
    Title    = "Attack Selected Once",
    Callback = function() attackTarget(selectedTarget) end
})

local AutoAttackToggle = Tabs.Main:AddToggle("AutoAttack", {
    Title   = "Auto Attack Loop",
    Default = Config.AutoAttack,
})

AutoAttackToggle:OnChanged(function()
    local val = Options.AutoAttack.Value
    Config.AutoAttack = val
    if val then
        if attackLoopRunning then return end
        attackLoopRunning = true
        addLog("▶️ Auto Attack started")
        task.spawn(function()
            while Options.AutoAttack.Value and not Fluent.Unloaded do
                if #priorityQueue == 0 then
                    addLog("⚠️ Queue empty, waiting...")
                    task.wait(2)
                else
                    for _, targetName in ipairs(priorityQueue) do
                        if not Options.AutoAttack.Value then break end
                        attackTarget(targetName)
                        local elapsed = 0
                        while elapsed < Config.AttackDelay and Options.AutoAttack.Value do
                            elapsed += task.wait(0.25)
                        end
                    end
                end
            end
            attackLoopRunning = false
            addLog("⏹️ Auto Attack stopped")
        end)
    else
        addLog("⏹️ Auto Attack stopping...")
    end
end)

-- ============================================================
-- PLAYER TAB
-- ============================================================
Tabs.Player:AddSection("Movement")

local noclipOriginal = setmetatable({}, { __mode = "k" })
local flyObjects     = {}
local flyKeys        = { W=false, A=false, S=false, D=false, Space=false, Ctrl=false }

local function getChar()  return LocalPlayer.Character end
local function getHuman() local c = getChar() return c and c:FindFirstChildOfClass("Humanoid") end
local function getRoot()  local c = getChar() return c and c:FindFirstChild("HumanoidRootPart") end

local walkSpeed = 16
local flySpeed  = 60

Tabs.Player:AddToggle("WalkSpeedEnabled", { Title = "Custom WalkSpeed", Default = false })
Tabs.Player:AddSlider("WalkSpeedValue", {
    Title = "Speed", Default = 16, Min = 16, Max = 500, Rounding = 0,
    Callback = function(v) walkSpeed = v end
})
Tabs.Player:AddToggle("InfiniteJump", { Title = "Infinite Jump", Default = false })

local NoclipToggle = Tabs.Player:AddToggle("Noclip", { Title = "Noclip", Default = false })
NoclipToggle:OnChanged(function(v)
    if not v then
        for part, orig in pairs(noclipOriginal) do
            if part and part.Parent then part.CanCollide = orig end
            noclipOriginal[part] = nil
        end
    end
end)

Tabs.Player:AddToggle("Fly", { Title = "Fly", Default = false })
Tabs.Player:AddSlider("FlySpeed", {
    Title = "Fly Speed", Default = 60, Min = 10, Max = 500, Rounding = 0,
    Callback = function(v) flySpeed = v end
})

local function destroyFly()
    for _, o in ipairs(flyObjects) do if o and o.Parent then o:Destroy() end end
    table.clear(flyObjects)
    local h = getHuman()
    if h then h.PlatformStand = false end
end

local function ensureFly()
    local root = getRoot()
    local h    = getHuman()
    if not root or not h then return nil, nil end
    local vel = root:FindFirstChild("WaveFlyVel") or (function()
        local v = Instance.new("BodyVelocity")
        v.Name = "WaveFlyVel"
        v.MaxForce = Vector3.new(1e9, 1e9, 1e9)
        v.Velocity = Vector3.zero
        v.Parent = root
        table.insert(flyObjects, v)
        return v
    end)()
    local gyro = root:FindFirstChild("WaveFlyGyro") or (function()
        local g = Instance.new("BodyGyro")
        g.Name = "WaveFlyGyro"
        g.MaxTorque = Vector3.new(1e9, 1e9, 1e9)
        g.P = 90000
        g.CFrame = root.CFrame
        g.Parent = root
        table.insert(flyObjects, g)
        return g
    end)()
    h.PlatformStand = true
    return vel, gyro
end

local keyMap = {
    [Enum.KeyCode.W]           = "W",
    [Enum.KeyCode.A]           = "A",
    [Enum.KeyCode.S]           = "S",
    [Enum.KeyCode.D]           = "D",
    [Enum.KeyCode.Space]       = "Space",
    [Enum.KeyCode.LeftControl] = "Ctrl",
}

local c1 = UserInputService.InputBegan:Connect(function(inp, gp)
    if gp then return end
    local k = keyMap[inp.KeyCode]
    if k then flyKeys[k] = true end
end)
local c2 = UserInputService.InputEnded:Connect(function(inp)
    local k = keyMap[inp.KeyCode]
    if k then flyKeys[k] = false end
end)
local c3 = UserInputService.JumpRequest:Connect(function()
    if Options.InfiniteJump and Options.InfiniteJump.Value then
        local h = getHuman()
        if h then h:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)
local c4 = RunService.Heartbeat:Connect(function()
    if Options.WalkSpeedEnabled and Options.WalkSpeedEnabled.Value then
        local h = getHuman()
        if h then h.WalkSpeed = walkSpeed end
    end
end)
local c5 = RunService.Stepped:Connect(function()
    if not Options.Noclip or not Options.Noclip.Value then return end
    local char = getChar()
    if not char then return end
    for _, d in ipairs(char:GetDescendants()) do
        if d:IsA("BasePart") then
            if noclipOriginal[d] == nil then noclipOriginal[d] = d.CanCollide end
            d.CanCollide = false
        end
    end
end)
local c6 = RunService.RenderStepped:Connect(function()
    if not Options.Fly or not Options.Fly.Value then
        if #flyObjects > 0 then destroyFly() end
        return
    end
    local vel, gyro = ensureFly()
    local root = getRoot()
    local cam  = workspace.CurrentCamera
    local h    = getHuman()
    if not vel or not gyro or not root or not cam or not h then return end
    local dir = Vector3.zero
    if flyKeys.W     then dir += cam.CFrame.LookVector  end
    if flyKeys.S     then dir -= cam.CFrame.LookVector  end
    if flyKeys.D     then dir += cam.CFrame.RightVector end
    if flyKeys.A     then dir -= cam.CFrame.RightVector end
    if flyKeys.Space then dir += Vector3.yAxis           end
    if flyKeys.Ctrl  then dir -= Vector3.yAxis           end
    if dir.Magnitude == 0 and h.MoveDirection.Magnitude > 0 then dir = h.MoveDirection end
    vel.Velocity = dir.Magnitude > 0 and dir.Unit * flySpeed or Vector3.zero
    gyro.CFrame  = CFrame.lookAt(root.Position, root.Position + cam.CFrame.LookVector)
end)

-- ============================================================
-- CONFIG TAB
-- ============================================================
Tabs.Config:AddSection("Webhook")

Tabs.Config:AddInput("WebhookURL", {
    Title       = "Discord Webhook URL",
    Default     = Config.WebhookUrl,
    Placeholder = "https://discord.com/api/webhooks/...",
    Numeric     = false,
    Finished    = true,
    Callback    = function(v) Config.WebhookUrl = v end
})

Tabs.Config:AddToggle("WebhookEnabled", {
    Title   = "Enable Webhook Logs",
    Default = Config.WebhookEnabled,
}):OnChanged(function(v) Config.WebhookEnabled = v end)

Tabs.Config:AddSection("Config Info")

Tabs.Config:AddParagraph({
    Title   = "วิธีใช้ getgenv() config",
    Content = "วางก่อนรัน script:\n\ngetgenv().MiniWarConfig = {\n  AutoBuy = true,\n  BuyInterval = 3,\n  BuyItems = {\n    \"GemMine\",\n    \"Data Center\",\n  },\n  AutoAttack = true,\n  AttackDelay = 20,\n  ArmyIndex = 2,\n  PriorityQueue = {\n    \"Bandit/Garnison1\",\n    \"WaterRigs/WaterRig1\",\n  },\n}"
})

Tabs.Config:AddButton({
    Title    = "Print Config",
    Callback = function()
        print("=== MiniWarConfig ===")
        for k, v in pairs(Config) do
            if type(v) == "table" then
                local items = {}
                for _, vi in ipairs(v) do table.insert(items, tostring(vi)) end
                print(k .. " = { " .. table.concat(items, ", ") .. " }")
            else
                print(k .. " = " .. tostring(v))
            end
        end
        addLog("📄 Config printed to console")
    end
})

Tabs.Config:AddButton({
    Title    = "Reset Config",
    Callback = function()
        getgenv().MiniWarConfig = nil
        addLog("🔄 Config reset, re-run script")
        Fluent:Notify({ Title = "Config", Content = "Reset done, re-run script", Duration = 4 })
    end
})

-- ============================================================
-- INIT
-- ============================================================
SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)
SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({ "WebhookURL", "BuyItem" })
InterfaceManager:SetFolder("MiniWarHub")
SaveManager:SetFolder("MiniWarHub/configs")
InterfaceManager:BuildInterfaceSection(Tabs.Config)
SaveManager:BuildConfigSection(Tabs.Config)

Window:SelectTab(1)

Fluent:Notify({ Title = "Mini-War Hub", Content = "✅ Loaded", Duration = 4 })
addLog("✅ Script loaded")
addLog("📦 Queue loaded: " .. #priorityQueue .. " targets")

local buyList = {}
for name in pairs(buyItemsSet) do table.insert(buyList, name) end
addLog("🛒 Buy items: " .. (#buyList > 0 and table.concat(buyList, ", ") or "none"))

-- auto start จาก config
if Config.AutoBuy and not buyLoopRunning then
    Options.AutoBuy:SetValue(true)
end
if Config.AutoAttack and not attackLoopRunning then
    Options.AutoAttack:SetValue(true)
end

-- ============================================================
-- CLEANUP
-- ============================================================
task.spawn(function()
    while not Fluent.Unloaded do task.wait(1) end
    destroyFly()
    for part, orig in pairs(noclipOriginal) do
        if part and part.Parent then part.CanCollide = orig end
    end
    c1:Disconnect() c2:Disconnect() c3:Disconnect()
    c4:Disconnect() c5:Disconnect() c6:Disconnect()
    addLog("🔴 Unloaded")
end)

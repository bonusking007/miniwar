local PLACE_ID = 131346454575416

if game.PlaceId ~= PLACE_ID then
    print("[Wave] Wrong game! Expected: " .. PLACE_ID .. " | Got: " .. game.PlaceId)
	wait(1)
	local Players = game:GetService("Players")
local lp = Players.LocalPlayer

for _, v in ipairs(lp.PlayerGui:GetDescendants()) do
    if v:IsA("TextButton") and string.find(string.upper(v.Text), "AFK") then
        firesignal(v.MouseButton1Click)
        break
    end
end
wait(3)
	local TeleportService = game:GetService("TeleportService")

local targetPlaceId = 5411459567

if game.PlaceId ~= targetPlaceId then
    TeleportService:Teleport(targetPlaceId)
else
    warn("Already in AFK World")
end
wait(0.1)
	warn("Anti afk running")
game:GetService("Players").LocalPlayer.Idled:connect(function()
warn("Anti afk ran")
game:GetService("VirtualUser"):CaptureController()
game:GetService("VirtualUser"):ClickButton2(Vector2.new())
end)

    return
end

print("[Wave] Correct game, loading...")

-- ============================================================
-- MINI-WAR HUB | Wave Script
-- ============================================================
getgenv().MiniWarConfig = getgenv().MiniWarConfig or {
    -- =================== [ Auto Buy ] ===================
    AutoBuy     = true,
    BuyInterval = 5,

    BuyItems = {
        -- Farm
		"Wheat Farm",
        -- "Corn Farm",
        -- "Data Center",
		"Blackhole Generator",
		"Area51Lab",
        "AntimatterReactor",
        -- "QuantumCoreGenerator",
        -- "SupernovaAccelerator",
        -- Military
        -- "Artillery Depot",
        "Rocket Bunker",
		"MechStation",
        -- "SpiderBase",
        -- "AirFortress",
        -- Black Market
        "GemMine",
        -- "TransportPad",
        -- "ConstructionSpecial",
        -- "CloneFacility",
        "CloneFacilityV2",
    },

    -- =================== [ Auto Attack ] ===================
    AutoAttack  = true,
    AttackDelay = 120,
    ArmyIndex   = 1,

    PriorityQueue = {
        "Special",
        "MilitaryTown/Town",
        "Laboratory/Laboratory",
		"Laboratory/Laboratory1",
		"Laboratory/Laboratory2",
        "MilitaryBase/MilitaryBase1",
        "MilitaryBase/MilitaryBase2",
		"MilitaryBase/MilitaryBase3",
		"MilitaryBase/MilitaryBase4",
    },

    -- =================== [ Auto Quest ] ===================
    AutoQuest        = true,
    QuestInterval    = 10, -- วินาทีระหว่าง claim แต่ละรอบ

    -- =================== [ Webhook ] ===================
    WebhookUrl     = "https://discord.com/api/webhooks/1520139298605891765/eGYUxALsSWLiKmKkZZOnmxEjY58SWNq36NwZ_QMEHAHCH6rZ4J6Iwj4R9aGgX7EweKi0",
    WebhookEnabled = true,
}

-- ============================================================
local HttpService       = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players           = game:GetService("Players")
local RunService        = game:GetService("RunService")
local UserInputService  = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local HttpRequest = request or http_request or (syn and syn.request)
assert(HttpRequest, "No HTTP support")

local Config = getgenv().MiniWarConfig

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

local Remote     = ReplicatedStorage:WaitForChild("ncxyzero_bridgenet2-fork@1.1.5"):WaitForChild("dataRemoteEvent")
local BridgeFunc = ReplicatedStorage:WaitForChild("_GetBridgeFunction")

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
    Size        = UDim2.fromOffset(630, 560),
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
-- INVENTORY CHECKER
-- ============================================================
local InventoryNames = {
    ["Wheat Farm"]           = "Wheat Farm",
    ["Corn Farm"]            = "Corn Farm",
    ["Data Center"]          = "Data Center",
    ["Blackhole Generator"]  = "Blackhole Generator",
    ["Area51Lab"]            = "Area51Lab",
    ["AntimatterReactor"]    = "AntimatterReactor",
    ["QuantumCoreGenerator"] = "QuantumCoreGenerator",
    ["SupernovaAccelerator"] = "SupernovaAccelerator",
    ["Artillery Depot"]      = "Artillery Depot",
    ["Rocket Bunker"]        = "Rocket Bunker",
    ["MechStation"]          = "MechStation",
    ["SpiderBase"]           = "SpiderBase",
    ["AirFortress"]          = "Air Base",
    ["GemMine"]              = "GemMine",
    ["TransportPad"]         = "TransportPad",
    ["ConstructionSpecial"]  = "ConstructionSpecial",
    ["CloneFacility"]        = "CloneFacility",
    ["CloneFacilityV2"]      = "CloneFacilityV2",
}

local function getInventoryQty(displayName)
    local invName = InventoryNames[displayName]
    if not invName then return 0 end
    local ok, result = pcall(function()
        local grid = LocalPlayer.PlayerGui.MainUI.Fullscreen.ItemsUI.ItemsGrid.ItemContent
        local item = grid:FindFirstChild(invName)
        if not item then return 0 end
        local qty = item:FindFirstChild("Quantity")
        if not qty then return 0 end
        local num = (qty.Text or ""):match("x(%d+)")
        return num and tonumber(num) or 0
    end)
    return ok and result or 0
end

-- ============================================================
-- BUY DATA
-- ============================================================
local BuyItems = {
    ["Wheat Farm"]           = { item = "FarmWheat",            shop = "Farm",        code = "\031" },
    ["Corn Farm"]            = { item = "FarmCorn",             shop = "Farm",        code = "\031" },
    ["Data Center"]          = { item = "Data Center",          shop = "Farm",        code = "\031" },
    ["Blackhole Generator"]  = { item = "Blackhole Generator",  shop = "Farm",        code = "\031" },
    ["Area51Lab"]            = { item = "Area51Lab",            shop = "Farm",        code = "\031" },
    ["AntimatterReactor"]    = { item = "AntimatterReactor",    shop = "Farm",        code = "\031" },
    ["QuantumCoreGenerator"] = { item = "QuantumCoreGenerator", shop = "Farm",        code = "\031" },
    ["SupernovaAccelerator"] = { item = "SupernovaAccelerator", shop = "Farm",        code = "\031" },
    ["Artillery Depot"]      = { item = "Artillery Depot",      shop = "Military",    code = "\031" },
    ["Rocket Bunker"]        = { item = "Rocket Bunker",        shop = "Military",    code = "\031" },
    ["MechStation"]          = { item = "MechStation",          shop = "Military",    code = "\031" },
    ["SpiderBase"]           = { item = "SpiderBase",           shop = "Military",    code = "\031" },
    ["AirFortress"]          = { item = "AirFortress",          shop = "Military",    code = "\031" },
    ["GemMine"]              = { item = "GemMine",              shop = "BlackMarket", code = "\031" },
    ["TransportPad"]         = { item = "TransportPad",         shop = "BlackMarket", code = "\031" },
    ["ConstructionSpecial"]  = { item = "ConstructionSpecial",  shop = "BlackMarket", code = "\031" },
    ["CloneFacility"]        = { item = "CloneFacility",        shop = "BlackMarket", code = "\031" },
    ["CloneFacilityV2"]      = { item = "CloneFacilityV2",      shop = "BlackMarket", code = "\031" },
}

local BuyItemNames = {
    "Wheat Farm","Corn Farm",
    "Data Center","Blackhole Generator","Area51Lab","AntimatterReactor",
    "QuantumCoreGenerator","SupernovaAccelerator",
    "Artillery Depot","Rocket Bunker","MechStation","SpiderBase","AirFortress",
    "GemMine","TransportPad","ConstructionSpecial","CloneFacility","CloneFacilityV2"
}

local buyItemsSet = {}
for _, name in ipairs(Config.BuyItems or {}) do
    buyItemsSet[name] = true
end

-- ============================================================
-- WEBHOOK
-- ============================================================
local function sendWebhook(itemName, shop, qtyBefore, qtyAfter)
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
                        title  = "✅ Item Added",
                        color  = 3066993,
                        fields = {
                            { name = "Item",     value = itemName,                        inline = true },
                            { name = "Shop",     value = shop,                            inline = true },
                            { name = "Quantity", value = qtyBefore .. " → " .. qtyAfter, inline = true },
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

local function buyItem(displayName)
    local data = BuyItems[displayName]
    if not data then addLog("❌ Unknown: " .. displayName) return false end

    local qtyBefore = getInventoryQty(displayName)

    local ok, err = pcall(function()
        BridgeFunc:InvokeServer("BuyFromShop")
        task.wait(0.3)
        fireBridge({ item = data.item, shop = data.shop }, data.code)
    end)

    if not ok then
        addLog("❌ Buy error: " .. tostring(err))
        return false
    end

    -- รอให้ inventory อัปเดต
    task.wait(0.8)

    local qtyAfter = getInventoryQty(displayName)

    if qtyAfter > qtyBefore then
        addLog("🛒 " .. displayName .. " (" .. qtyBefore .. " → " .. qtyAfter .. ")")
        sendWebhook(displayName, data.shop, qtyBefore, qtyAfter)
        return true
    else
        addLog("⚠️ No change: " .. displayName)
        return false
    end
end

local function buyAllSelected()
    local merged = {}
    for name, state in pairs(buyItemsSet) do
        if state then merged[name] = true end
    end
    if Options.BuyItem and Options.BuyItem.Value then
        for name, state in pairs(Options.BuyItem.Value) do
            if state then merged[name] = true end
        end
    end
    local count = 0
    for name in pairs(merged) do
        buyItem(name)
        count += 1
        task.wait(0.5)
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
    for name, state in pairs(value) do buyItemsSet[name] = state end
    local list = {}
    for name, state in pairs(buyItemsSet) do if state then table.insert(list, name) end end
    addLog("📋 Selected: " .. (#list > 0 and table.concat(list, ", ") or "none"))
end)

Tabs.Main:AddSlider("BuyInterval", {
    Title    = "Buy Interval (s)",
    Default  = Config.BuyInterval,
    Min = 1, Max = 120, Rounding = 0,
    Callback = function(v) Config.BuyInterval = v end
})

Tabs.Main:AddButton({
    Title    = "Buy Once",
    Callback = function() buyAllSelected() end
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
        addLog("▶️ Auto Buy started")
        task.spawn(function()
            while Options.AutoBuy.Value and not Fluent.Unloaded do
                local ok, err = pcall(buyAllSelected)
                if not ok then addLog("❌ Loop error: " .. tostring(err)) end
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
-- AUTO QUEST
-- ============================================================
Tabs.Main:AddSection("Auto Quest")

local questLoopRunning = false

local function claimAllQuests()
    for i = 1, 4 do
        local ok, err = pcall(fireBridge, tostring(i), "Z")
        if ok then
            addLog("🎯 Claimed quest " .. i)
        else
            addLog("⚠️ Quest " .. i .. " error: " .. tostring(err))
        end
        task.wait(0.3)
    end
end

Tabs.Main:AddButton({
    Title    = "Claim All Quests Once",
    Callback = function() claimAllQuests() end
})

Tabs.Main:AddSlider("QuestInterval", {
    Title    = "Quest Interval (s)",
    Default  = Config.QuestInterval,
    Min = 10, Max = 300, Rounding = 0,
    Callback = function(v) Config.QuestInterval = v end
})

local AutoQuestToggle = Tabs.Main:AddToggle("AutoQuest", {
    Title   = "Auto Claim Quest",
    Default = Config.AutoQuest,
})

AutoQuestToggle:OnChanged(function()
    local val = Options.AutoQuest.Value
    Config.AutoQuest = val
    if val then
        if questLoopRunning then return end
        questLoopRunning = true
        addLog("▶️ Auto Quest started")
        task.spawn(function()
            while Options.AutoQuest.Value and not Fluent.Unloaded do
                local ok, err = pcall(claimAllQuests)
                if not ok then addLog("❌ Quest error: " .. tostring(err)) end
                task.wait(Config.QuestInterval)
            end
            questLoopRunning = false
            addLog("⏹️ Auto Quest stopped")
        end)
    else
        addLog("⏹️ Auto Quest stopping...")
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

for i=1,6 do addTarget("Bandit/Garnison"..i,          {"Bandit",       "Garnison"..i     }) end
addTarget("Laboratory/Laboratory",                      {"Laboratory",   "Laboratory"      })
for i=1,5 do addTarget("Laboratory/Laboratory"..i,     {"Laboratory",   "Laboratory"..i   }) end
for i=1,6 do addTarget("MilitaryBase/MilitaryBase"..i, {"MilitaryBase", "MilitaryBase"..i }) end
addTarget("MilitaryTown/Town",                          {"MilitaryTown", "Town"            })
addTarget("Special",                                    {"Special"                         })
for i=1,6 do addTarget("WaterRigs/WaterRig"..i,        {"WaterRigs",    "WaterRig"..i     }) end
for i=1,6 do addTarget("Wooden/Garnison"..i,           {"Wooden",       "Garnison"..i     }) end

local selectedTarget    = TargetNames[1]
local attackLoopRunning = false
local priorityQueue     = {}

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
        if not folder then addLog("❌ Special not found") return {} end
        local t = {}
        for _, c in ipairs(folder:GetChildren()) do table.insert(t, c) end
        return t
    end
    local path = TargetPaths[targetName]
    if not path then return {} end
    local cur = root
    for _, name in ipairs(path) do
        cur = cur:FindFirstChild(name)
        if not cur then addLog("❌ Not found: "..name) return {} end
    end
    return { cur }
end

local function attackTarget(targetName)
    local targets = resolveTargets(targetName)
    if #targets == 0 then addLog("⚠️ No target: "..targetName) return end
    local code = targetName == "Special" and "^" or "V"
    for _, target in ipairs(targets) do
        local ok, err = pcall(fireBridge, { armyIndex = Config.ArmyIndex, capturePoint = target }, code)
        if ok then
            addLog("⚔️ Attack: "..target.Name.." [army "..Config.ArmyIndex.."]")
        else
            addLog("❌ Attack error: "..tostring(err))
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
        for i, v in ipairs(priorityQueue) do lines[i] = i..". "..v end
        PriorityText:SetDesc(table.concat(lines, "\n"))
    end
end

refreshQueue()

Tabs.Main:AddDropdown("AttackTarget", {
    Title = "Target", Values = TargetNames, Multi = false, Default = 1,
}):OnChanged(function(v) selectedTarget = v end)

Tabs.Main:AddButton({ Title = "Add To Queue", Callback = function()
    for _, v in ipairs(priorityQueue) do
        if v == selectedTarget then
            Fluent:Notify({ Title="Queue", Content="Already in queue", Duration=2 })
            return
        end
    end
    table.insert(priorityQueue, selectedTarget)
    addLog("➕ Queue: "..selectedTarget)
    refreshQueue()
end })

Tabs.Main:AddButton({ Title = "Remove Last", Callback = function()
    local r = table.remove(priorityQueue)
    if r then addLog("➖ Removed: "..r) end
    refreshQueue()
end })

Tabs.Main:AddButton({ Title = "Clear Queue", Callback = function()
    table.clear(priorityQueue)
    addLog("🗑️ Queue cleared")
    refreshQueue()
end })

Tabs.Main:AddSlider("AttackDelay", {
    Title = "Delay Per Target (s)", Default = Config.AttackDelay,
    Min=1, Max=180, Rounding=0,
    Callback = function(v) Config.AttackDelay = v end
})

Tabs.Main:AddSlider("ArmyIndex", {
    Title = "Army Index", Default = Config.ArmyIndex,
    Min=1, Max=10, Rounding=0,
    Callback = function(v) Config.ArmyIndex = v end
})

Tabs.Main:AddButton({
    Title = "Attack Selected Once",
    Callback = function() attackTarget(selectedTarget) end
})

local AutoAttackToggle = Tabs.Main:AddToggle("AutoAttack", {
    Title = "Auto Attack Loop", Default = Config.AutoAttack,
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
                        local ok, err = pcall(attackTarget, targetName)
                        if not ok then addLog("❌ Attack error: "..tostring(err)) end
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

local walkSpeed      = 16
local flySpeed       = 60
local noclipOriginal = setmetatable({}, { __mode = "k" })
local flyObjects     = {}
local flyKeys        = { W=false, A=false, S=false, D=false, Space=false, Ctrl=false }

local function getChar()  return LocalPlayer.Character end
local function getHuman() local c=getChar() return c and c:FindFirstChildOfClass("Humanoid") end
local function getRoot()  local c=getChar() return c and c:FindFirstChild("HumanoidRootPart") end

Tabs.Player:AddToggle("WalkSpeedEnabled", { Title="Custom WalkSpeed", Default=false })
Tabs.Player:AddSlider("WalkSpeedValue", {
    Title="Speed", Default=16, Min=16, Max=500, Rounding=0,
    Callback=function(v) walkSpeed=v end
})
Tabs.Player:AddToggle("InfiniteJump", { Title="Infinite Jump", Default=false })

local NoclipToggle = Tabs.Player:AddToggle("Noclip", { Title="Noclip", Default=false })
NoclipToggle:OnChanged(function(v)
    if not v then
        for part, orig in pairs(noclipOriginal) do
            if part and part.Parent then part.CanCollide=orig end
            noclipOriginal[part]=nil
        end
    end
end)

Tabs.Player:AddToggle("Fly", { Title="Fly", Default=false })
Tabs.Player:AddSlider("FlySpeed", {
    Title="Fly Speed", Default=60, Min=10, Max=500, Rounding=0,
    Callback=function(v) flySpeed=v end
})

local function destroyFly()
    for _,o in ipairs(flyObjects) do if o and o.Parent then o:Destroy() end end
    table.clear(flyObjects)
    local h=getHuman() if h then h.PlatformStand=false end
end

local function ensureFly()
    local root=getRoot() local h=getHuman()
    if not root or not h then return nil,nil end
    local vel=root:FindFirstChild("WaveFlyVel") or (function()
        local v=Instance.new("BodyVelocity")
        v.Name="WaveFlyVel" v.MaxForce=Vector3.new(1e9,1e9,1e9) v.Velocity=Vector3.zero v.Parent=root
        table.insert(flyObjects,v) return v
    end)()
    local gyro=root:FindFirstChild("WaveFlyGyro") or (function()
        local g=Instance.new("BodyGyro")
        g.Name="WaveFlyGyro" g.MaxTorque=Vector3.new(1e9,1e9,1e9) g.P=90000 g.CFrame=root.CFrame g.Parent=root
        table.insert(flyObjects,g) return g
    end)()
    h.PlatformStand=true
    return vel,gyro
end

local keyMap={
    [Enum.KeyCode.W]="W",[Enum.KeyCode.A]="A",[Enum.KeyCode.S]="S",
    [Enum.KeyCode.D]="D",[Enum.KeyCode.Space]="Space",[Enum.KeyCode.LeftControl]="Ctrl"
}

local c1=UserInputService.InputBegan:Connect(function(inp,gp)
    if gp then return end local k=keyMap[inp.KeyCode] if k then flyKeys[k]=true end
end)
local c2=UserInputService.InputEnded:Connect(function(inp)
    local k=keyMap[inp.KeyCode] if k then flyKeys[k]=false end
end)
local c3=UserInputService.JumpRequest:Connect(function()
    if Options.InfiniteJump and Options.InfiniteJump.Value then
        local h=getHuman() if h then h:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)
local c4=RunService.Heartbeat:Connect(function()
    if Options.WalkSpeedEnabled and Options.WalkSpeedEnabled.Value then
        local h=getHuman() if h then h.WalkSpeed=walkSpeed end
    end
end)
local c5=RunService.Stepped:Connect(function()
    if not Options.Noclip or not Options.Noclip.Value then return end
    local char=getChar() if not char then return end
    for _,d in ipairs(char:GetDescendants()) do
        if d:IsA("BasePart") then
            if noclipOriginal[d]==nil then noclipOriginal[d]=d.CanCollide end
            d.CanCollide=false
        end
    end
end)
local c6=RunService.RenderStepped:Connect(function()
    if not Options.Fly or not Options.Fly.Value then
        if #flyObjects>0 then destroyFly() end return
    end
    local vel,gyro=ensureFly()
    local root=getRoot() local cam=workspace.CurrentCamera local h=getHuman()
    if not vel or not gyro or not root or not cam or not h then return end
    local dir=Vector3.zero
    if flyKeys.W     then dir+=cam.CFrame.LookVector  end
    if flyKeys.S     then dir-=cam.CFrame.LookVector  end
    if flyKeys.D     then dir+=cam.CFrame.RightVector end
    if flyKeys.A     then dir-=cam.CFrame.RightVector end
    if flyKeys.Space then dir+=Vector3.yAxis           end
    if flyKeys.Ctrl  then dir-=Vector3.yAxis           end
    if dir.Magnitude==0 and h.MoveDirection.Magnitude>0 then dir=h.MoveDirection end
    vel.Velocity=dir.Magnitude>0 and dir.Unit*flySpeed or Vector3.zero
    gyro.CFrame=CFrame.lookAt(root.Position,root.Position+cam.CFrame.LookVector)
end)

-- ============================================================
-- CONFIG TAB
-- ============================================================
Tabs.Config:AddSection("Webhook")

Tabs.Config:AddInput("WebhookURL", {
    Title="Discord Webhook URL", Default=Config.WebhookUrl,
    Placeholder="https://discord.com/api/webhooks/...",
    Numeric=false, Finished=true,
    Callback=function(v) Config.WebhookUrl=v end
})

Tabs.Config:AddToggle("WebhookEnabled", {
    Title="Enable Webhook Logs", Default=Config.WebhookEnabled,
}):OnChanged(function(v) Config.WebhookEnabled=v end)

Tabs.Config:AddSection("Info")

Tabs.Config:AddParagraph({
    Title   = "getgenv() Config Example",
    Content = "getgenv().MiniWarConfig = {\n  AutoBuy=true, BuyInterval=5,\n  BuyItems={\"Wheat Farm\",\"GemMine\"},\n  AutoAttack=true, AttackDelay=25,\n  ArmyIndex=1,\n  AutoQuest=true, QuestInterval=60,\n  PriorityQueue={\n    \"Special\",\n    \"MilitaryTown/Town\",\n  },\n}"
})

Tabs.Config:AddButton({
    Title="Print Config", Callback=function()
        print("=== MiniWarConfig ===")
        for k,v in pairs(Config) do
            if type(v)=="table" then
                local items={} for _,vi in ipairs(v) do table.insert(items,tostring(vi)) end
                print(k.." = { "..table.concat(items,", ").." }")
            else print(k.." = "..tostring(v)) end
        end
        addLog("📄 Config printed")
    end
})

Tabs.Config:AddButton({
    Title="Reset Config", Callback=function()
        getgenv().MiniWarConfig=nil
        addLog("🔄 Reset, re-run script")
        Fluent:Notify({ Title="Config", Content="Reset done, re-run script", Duration=4 })
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

Fluent:Notify({ Title="Mini-War Hub", Content="✅ Loaded", Duration=4 })
addLog("✅ Loaded")
addLog("📦 Queue: "..#priorityQueue.." targets")

local buyList={}
for name in pairs(buyItemsSet) do table.insert(buyList,name) end
addLog("🛒 Buy: "..(#buyList>0 and table.concat(buyList,", ") or "none"))

task.wait(0.5)
if Config.AutoBuy    then Options.AutoBuy:SetValue(true)    end
if Config.AutoAttack then Options.AutoAttack:SetValue(true) end
if Config.AutoQuest  then Options.AutoQuest:SetValue(true)  end

-- ============================================================
-- CLEANUP
-- ============================================================
task.spawn(function()
    while not Fluent.Unloaded do task.wait(1) end
    destroyFly()
    for part,orig in pairs(noclipOriginal) do
        if part and part.Parent then part.CanCollide=orig end
    end
    c1:Disconnect() c2:Disconnect() c3:Disconnect()
    c4:Disconnect() c5:Disconnect() c6:Disconnect()
    addLog("🔴 Unloaded")
end)

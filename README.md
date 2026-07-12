getgenv().MiniWarConfig = getgenv().MiniWarConfig or {
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
}

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
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

local Fluent = loadstring(fetch("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Remote = ReplicatedStorage:WaitForChild("ncxyzero_bridgenet2-fork@1.1.5"):WaitForChild("dataRemoteEvent")

local unpackArgs = table.unpack or unpack
local function fireBridge(data, code)
    Remote:FireServer(unpackArgs({ { data, code } }))
end

local Window = Fluent:CreateWindow({
    Title = "Mini-War Hub",
    SubTitle = "Auto Attack",
    TabWidth = 160,
    Size = UDim2.fromOffset(500, 460),
    Acrylic = false,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    Main = Window:AddTab({ Title = "Main", Icon = "swords" }),
    Log  = Window:AddTab({ Title = "Log", Icon = "scroll" }),
}

local Options = Fluent.Options

local logLines = {}
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

local TargetNames = {}
local TargetPaths = {}

local function addTarget(display, path)
    table.insert(TargetNames, display)
    TargetPaths[display] = path
end

for i=1,6 do addTarget("Bandit/Garnison"..i, {"Bandit","Garnison"..i}) end
addTarget("Laboratory/Laboratory", {"Laboratory","Laboratory"})
for i=1,5 do addTarget("Laboratory/Laboratory"..i, {"Laboratory","Laboratory"..i}) end
for i=1,6 do addTarget("MilitaryBase/MilitaryBase"..i, {"MilitaryBase","MilitaryBase"..i}) end
addTarget("MilitaryTown/Town", {"MilitaryTown","Town"})
addTarget("Special", {"Special"})
for i=1,6 do addTarget("WaterRigs/WaterRig"..i, {"WaterRigs","WaterRig"..i}) end
for i=1,6 do addTarget("Wooden/Garnison"..i, {"Wooden","Garnison"..i}) end

local selectedTarget = TargetNames[1]
local attackLoopRunning = false
local priorityQueue = {}

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

Window:SelectTab(1)

Fluent:Notify({ Title="Mini-War Hub", Content="✅ Loaded", Duration=4 })
addLog("✅ Loaded")
addLog("📦 Queue: "..#priorityQueue.." targets")

task.wait(0.5)
if Config.AutoAttack then Options.AutoAttack:SetValue(true) end

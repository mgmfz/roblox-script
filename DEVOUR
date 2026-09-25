local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local GuiService = game:GetService("GuiService")
local VIM = game:GetService("VirtualInputManager")
local VirtualUser = game:GetService("VirtualUser")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local MAX_DISTANCE = math.huge
local INTERVAL = 0.1
local MAX_PER_CYCLE = 6
local TELEPORT = true
local HOTSPOT_RADIUS = 40
local FALLBACK_F = 2
local ESP_RANGE = 2000
local PLAYER_ESP_RANGE = 8000
local ESP_MAX = 80
local SAFE_HEIGHT = 1500
local DIE_TIMEOUT = 25
local BOMB_COOLDOWN = 6
local HOT = 6
local PLAYERS_ESP = 7

local AFK_PRESETS = {
    {name = "Fast", per = 6, hop = 0.12, pause = 0.4},
    {name = "Medium", per = 4, hop = 0.25, pause = 0.8},
    {name = "Slow", per = 2, hop = 0.5, pause = 2.0},
}
local afkPreset = 2

local CATEGORIES = {
    {label = "Starfish", key = "starfish"},
    {label = "Coral (all)", key = "coral"},
    {label = "Seasponge", key = "sponge"},
    {label = "Seaweed", key = "seaweed"},
    {label = "Oyster", key = "oyster"},
    {label = "Auto Spot (TP + F)", key = false},
}
local selected = {true, false, false, false, false, false}
local allSel = {true, true, true, true, true, true}

local C = {
    bg = Color3.fromRGB(16, 18, 26),
    card = Color3.fromRGB(28, 31, 43),
    accent = Color3.fromRGB(64, 156, 255),
    on = Color3.fromRGB(60, 200, 120),
    off = Color3.fromRGB(230, 70, 80),
    gold = Color3.fromRGB(240, 170, 40),
    text = Color3.fromRGB(236, 239, 246),
    sub = Color3.fromRGB(140, 148, 170),
    track = Color3.fromRGB(58, 62, 80),
}

local COLORS = {
    Color3.fromRGB(255, 170, 60),
    Color3.fromRGB(255, 90, 140),
    Color3.fromRGB(210, 210, 120),
    Color3.fromRGB(80, 220, 120),
    Color3.fromRGB(170, 130, 255),
    Color3.fromRGB(240, 200, 40),
    Color3.fromRGB(255, 80, 80),
}

local function getGuiParent()
    if typeof(gethui) == "function" then
        local ok, h = pcall(gethui)
        if ok and h then return h end
    end
    local ok, cg = pcall(function() return game:GetService("CoreGui") end)
    if ok and cg then
        local t = Instance.new("Folder")
        local okp = pcall(function() t.Parent = cg end)
        t:Destroy()
        if okp then return cg end
    end
    return playerGui
end

local guiParent = getGuiParent()
local containers = {guiParent, playerGui}
pcall(function() table.insert(containers, game:GetService("CoreGui")) end)
for _, container in ipairs(containers) do
    for _, n in ipairs({"StarfishGrabGUI", "TPMenuGUI"}) do
        local o = container:FindFirstChild(n)
        if o then o:Destroy() end
    end
end

local gui = Instance.new("ScreenGui")
gui.Name = "TPMenuGUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.DisplayOrder = 2147483647
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
if syn and syn.protect_gui then pcall(syn.protect_gui, gui) end
gui.Parent = guiParent

local function corner(o, r)
    Instance.new("UICorner", o).CornerRadius = UDim.new(0, r)
end

local function label(parent, txt, size, color, font)
    local l = Instance.new("TextLabel")
    l.BackgroundTransparency = 1
    l.Text = txt
    l.TextSize = size
    l.TextColor3 = color
    l.Font = font or Enum.Font.GothamMedium
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = parent
    return l
end

local function button(parent, txt, size, color, textColor)
    local b = Instance.new("TextButton")
    b.Size = size
    b.BackgroundColor3 = color
    b.Text = txt
    b.TextSize = 14
    b.TextColor3 = textColor or C.text
    b.Font = Enum.Font.GothamBold
    b.BorderSizePixel = 0
    b.AutoButtonColor = false
    b.Parent = parent
    corner(b, 8)
    return b
end

local frame = Instance.new("Frame")
frame.Size = UDim2.fromOffset(300, 0)
frame.AutomaticSize = Enum.AutomaticSize.Y
frame.Position = UDim2.new(0, 20, 0.2, 0)
frame.BackgroundColor3 = C.bg
frame.BorderSizePixel = 0
frame.Parent = gui
corner(frame, 10)
local stroke = Instance.new("UIStroke", frame)
stroke.Color = C.accent
stroke.Thickness = 1.5
stroke.Transparency = 0.35
Instance.new("UIListLayout", frame).SortOrder = Enum.SortOrder.LayoutOrder

local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 36)
header.BackgroundColor3 = C.card
header.BorderSizePixel = 0
header.LayoutOrder = 0
header.Parent = frame
corner(header, 10)
local grad = Instance.new("UIGradient", header)
grad.Color = ColorSequence.new(Color3.fromRGB(42, 50, 76), Color3.fromRGB(24, 27, 40))

local title = label(header, "FISH HELPER", 15, C.text, Enum.Font.GothamBold)
title.Size = UDim2.new(1, -50, 1, 0)
title.Position = UDim2.fromOffset(12, 0)

local minBtn = button(header, "-", UDim2.fromOffset(26, 22), C.track)
minBtn.Position = UDim2.new(1, -34, 0.5, -11)
minBtn.TextSize = 18

local content = Instance.new("Frame")
content.Size = UDim2.new(1, 0, 0, 0)
content.AutomaticSize = Enum.AutomaticSize.Y
content.BackgroundTransparency = 1
content.LayoutOrder = 1
content.Parent = frame
local cl = Instance.new("UIListLayout", content)
cl.Padding = UDim.new(0, 8)
cl.SortOrder = Enum.SortOrder.LayoutOrder
local cp = Instance.new("UIPadding", content)
cp.PaddingTop = UDim.new(0, 10)
cp.PaddingBottom = UDim.new(0, 10)
cp.PaddingLeft = UDim.new(0, 10)
cp.PaddingRight = UDim.new(0, 10)

local function toggleMin()
    content.Visible = not content.Visible
    minBtn.Text = content.Visible and "-" or "+"
end
minBtn.Activated:Connect(toggleMin)

local tabbar = Instance.new("Frame")
tabbar.Size = UDim2.new(1, 0, 0, 34)
tabbar.BackgroundColor3 = C.card
tabbar.BorderSizePixel = 0
tabbar.LayoutOrder = 1
tabbar.Parent = content
corner(tabbar, 8)
local tl = Instance.new("UIListLayout", tabbar)
tl.FillDirection = Enum.FillDirection.Horizontal
tl.Padding = UDim.new(0, 4)
tl.VerticalAlignment = Enum.VerticalAlignment.Center
local tp = Instance.new("UIPadding", tabbar)
tp.PaddingLeft = UDim.new(0, 4)
tp.PaddingRight = UDim.new(0, 4)

local pages, tabBtns = {}, {}
local function newPage(name, order)
    local p = Instance.new("Frame")
    p.Size = UDim2.new(1, 0, 0, 0)
    p.AutomaticSize = Enum.AutomaticSize.Y
    p.BackgroundTransparency = 1
    p.LayoutOrder = order
    p.Visible = false
    p.Parent = content
    local l = Instance.new("UIListLayout", p)
    l.Padding = UDim.new(0, 6)
    l.SortOrder = Enum.SortOrder.LayoutOrder
    pages[name] = p

    local b = button(tabbar, name, UDim2.new(1 / 5, -5, 0, 26), C.card, C.sub)
    b.TextSize = 11
    tabBtns[name] = b
    return p
end

local function showTab(name)
    for k, p in pairs(pages) do p.Visible = (k == name) end
    for k, b in pairs(tabBtns) do
        b.BackgroundColor3 = (k == name) and C.accent or C.card
        b.TextColor3 = (k == name) and Color3.new(1, 1, 1) or C.sub
    end
end

local pageCollect = newPage("Collect", 2)
local pageSpeed = newPage("Speed", 3)
local pageEsp = newPage("ESP", 4)
local pageAfk = newPage("AFK", 5)
local pageExtra = newPage("Extra", 6)
for name, b in pairs(tabBtns) do
    b.Activated:Connect(function() showTab(name) end)
end
showTab("Collect")

local function section(page, txt)
    local s = label(page, txt, 11, C.sub, Enum.Font.GothamBold)
    s.Size = UDim2.new(1, 0, 0, 16)
    s.LayoutOrder = #page:GetChildren()
end

local function addToggle(page, text, color, initial, onChange)
    local state = initial
    local row = Instance.new("TextButton")
    row.Size = UDim2.new(1, 0, 0, 32)
    row.BackgroundColor3 = C.card
    row.Text = ""
    row.AutoButtonColor = false
    row.BorderSizePixel = 0
    row.LayoutOrder = #page:GetChildren()
    row.Parent = page
    corner(row, 8)

    local l = label(row, text, 14, C.text)
    l.Size = UDim2.new(1, -60, 1, 0)
    l.Position = UDim2.fromOffset(12, 0)

    local track = Instance.new("Frame")
    track.Size = UDim2.fromOffset(34, 18)
    track.Position = UDim2.new(1, -44, 0.5, -9)
    track.BackgroundColor3 = C.track
    track.BorderSizePixel = 0
    track.Parent = row
    corner(track, 9)

    local knob = Instance.new("Frame")
    knob.Size = UDim2.fromOffset(14, 14)
    knob.Position = UDim2.new(0, 2, 0.5, -7)
    knob.BackgroundColor3 = Color3.new(1, 1, 1)
    knob.BorderSizePixel = 0
    knob.Parent = track
    corner(knob, 7)

    local function apply()
        TweenService:Create(track, TweenInfo.new(0.15), {BackgroundColor3 = state and color or C.track}):Play()
        TweenService:Create(knob, TweenInfo.new(0.15), {
            Position = state and UDim2.new(1, -16, 0.5, -7) or UDim2.new(0, 2, 0.5, -7)
        }):Play()
    end
    apply()

    row.Activated:Connect(function()
        state = not state
        apply()
        onChange(state)
    end)

    return function(v)
        state = v
        apply()
    end
end

local function goHotspot() end
local setSafe = function() end
local safeOn, dying, collecting = false, false, false

-- ===== COLLECT TAB =====
local master = button(pageCollect, "TP: OFF", UDim2.new(1, 0, 0, 38), C.off, Color3.new(1, 1, 1))
master.TextSize = 16
master.LayoutOrder = 0

for i, cat in ipairs(CATEGORIES) do
    if i == 1 then section(pageCollect, "COLLECT (TP)") end
    if i == HOT then section(pageCollect, "SPOT") end
    addToggle(pageCollect, cat.label, i == HOT and C.gold or C.on, selected[i], function(v)
        selected[i] = v
        if i == HOT and v then goHotspot() end
    end)
end

local status = label(pageCollect, "", 12, C.sub)
status.Size = UDim2.new(1, 0, 0, 30)
status.TextWrapped = true
status.TextYAlignment = Enum.TextYAlignment.Top
status.LayoutOrder = #pageCollect:GetChildren()

local scan = button(pageCollect, "Scan hotspot (F9)", UDim2.new(1, 0, 0, 24), C.card, C.sub)
scan.TextSize = 12
scan.Font = Enum.Font.GothamMedium
scan.LayoutOrder = #pageCollect:GetChildren()

-- ===== SPEED TAB =====
local speedOn, mult, method = false, 2, "CFrame"
local origWalk = nil

section(pageSpeed, "SPEED")
addToggle(pageSpeed, "Speed", C.on, false, function(v) speedOn = v end)

local multRow = Instance.new("Frame")
multRow.Size = UDim2.new(1, 0, 0, 36)
multRow.BackgroundColor3 = C.card
multRow.BorderSizePixel = 0
multRow.LayoutOrder = #pageSpeed:GetChildren()
multRow.Parent = pageSpeed
corner(multRow, 8)

local multTitle = label(multRow, "Multiplier", 14, C.text)
multTitle.Size = UDim2.new(0, 100, 1, 0)
multTitle.Position = UDim2.fromOffset(12, 0)

local plusBtn = button(multRow, "+", UDim2.fromOffset(28, 24), C.track)
plusBtn.Position = UDim2.new(1, -36, 0.5, -12)
local multVal = label(multRow, "x2.0", 14, C.accent, Enum.Font.GothamBold)
multVal.Size = UDim2.fromOffset(50, 24)
multVal.Position = UDim2.new(1, -90, 0.5, -12)
multVal.TextXAlignment = Enum.TextXAlignment.Center
local minusBtn = button(multRow, "-", UDim2.fromOffset(28, 24), C.track)
minusBtn.Position = UDim2.new(1, -124, 0.5, -12)

local function setMult(v)
    mult = math.clamp(v, 1, 10)
    multVal.Text = string.format("x%.1f", mult)
end
plusBtn.Activated:Connect(function() setMult(mult + 0.5) end)
minusBtn.Activated:Connect(function() setMult(mult - 0.5) end)

local methodBtn = button(pageSpeed, "Method: CFrame", UDim2.new(1, 0, 0, 32), C.card)
methodBtn.LayoutOrder = #pageSpeed:GetChildren()
methodBtn.Activated:Connect(function()
    method = (method == "CFrame") and "WalkSpeed" or "CFrame"
    methodBtn.Text = "Method: " .. method
end)

local note = label(pageSpeed, "CFrame: multiplies the speed you're already swimming at. WalkSpeed: only works if the fish uses Humanoid.", 11, C.sub)
note.Size = UDim2.new(1, 0, 0, 44)
note.TextWrapped = true
note.TextYAlignment = Enum.TextYAlignment.Top
note.LayoutOrder = #pageSpeed:GetChildren()

-- ===== ESP TAB =====
local espMaster = false
local espSel = {true, true, true, true, true, true, false}

section(pageEsp, "ESP")
addToggle(pageEsp, "ESP on", C.on, false, function(v) espMaster = v end)
section(pageEsp, "SHOW")
for i, cat in ipairs(CATEGORIES) do
    addToggle(pageEsp, i == HOT and "Hotspot" or cat.label, COLORS[i], espSel[i], function(v)
        espSel[i] = v
    end)
end
addToggle(pageEsp, "Players (long range)", COLORS[PLAYERS_ESP], false, function(v)
    espSel[PLAYERS_ESP] = v
end)

local espNote = label(pageEsp, "Players are tracked at a much longer range than other categories, so you can spot them from far away.", 11, C.sub)
espNote.Size = UDim2.new(1, 0, 0, 32)
espNote.TextWrapped = true
espNote.TextYAlignment = Enum.TextYAlignment.Top
espNote.LayoutOrder = #pageEsp:GetChildren()

-- ===== AFK TAB =====
local afkOn, afkHot, antiKick = false, true, true
local afkCount = 0

local afkBtn = button(pageAfk, "PUFFER AFK: OFF", UDim2.new(1, 0, 0, 38), C.off, Color3.new(1, 1, 1))
afkBtn.TextSize = 16
afkBtn.LayoutOrder = 0

section(pageAfk, "OPTIONS")

local paceBtn = button(pageAfk, "Pace: " .. AFK_PRESETS[afkPreset].name, UDim2.new(1, 0, 0, 32), C.card)
paceBtn.LayoutOrder = #pageAfk:GetChildren()
paceBtn.Activated:Connect(function()
    afkPreset = afkPreset % #AFK_PRESETS + 1
    paceBtn.Text = "Pace: " .. AFK_PRESETS[afkPreset].name
end)

addToggle(pageAfk, "Stay at hotspot", C.gold, afkHot, function(v) afkHot = v end)
addToggle(pageAfk, "Anti-kick (idle)", C.on, antiKick, function(v) antiKick = v end)

local afkStatus = label(pageAfk, "AFK: off", 12, C.sub)
afkStatus.Size = UDim2.new(1, 0, 0, 30)
afkStatus.TextWrapped = true
afkStatus.TextYAlignment = Enum.TextYAlignment.Top
afkStatus.LayoutOrder = #pageAfk:GetChildren()

local afkNote = label(pageAfk, "Teleports across all categories and uses Puff Up (F) whenever it's ready. Slow pace is the most discreet. While AFK is on, the Collect tab is paused.", 11, C.sub)
afkNote.Size = UDim2.new(1, 0, 0, 56)
afkNote.TextWrapped = true
afkNote.TextYAlignment = Enum.TextYAlignment.Top
afkNote.LayoutOrder = #pageAfk:GetChildren()

-- ===== EXTRA TAB =====
section(pageExtra, "SAFESPOT")
local setSafeToggle = addToggle(pageExtra, "SafeSpot (sky)", C.on, false, function(v) setSafe(v) end)

local safeNote = label(pageExtra, "Takes you to a platform in the sky. Turning it off returns you to where you left. Collect and AFK are paused.", 11, C.sub)
safeNote.Size = UDim2.new(1, 0, 0, 44)
safeNote.TextWrapped = true
safeNote.TextYAlignment = Enum.TextYAlignment.Top
safeNote.LayoutOrder = #pageExtra:GetChildren()

section(pageExtra, "DIE")
local dieBtn = button(pageExtra, "💀 DIE", UDim2.new(1, 0, 0, 38), C.off, Color3.new(1, 1, 1))
dieBtn.TextSize = 16
dieBtn.LayoutOrder = #pageExtra:GetChildren()

local dieStatus = label(pageExtra, "Uses the map's bombs to kill the fish.", 12, C.sub)
dieStatus.Size = UDim2.new(1, 0, 0, 30)
dieStatus.TextWrapped = true
dieStatus.TextYAlignment = Enum.TextYAlignment.Top
dieStatus.LayoutOrder = #pageExtra:GetChildren()

local bombScan = button(pageExtra, "Scan bombs (F9)", UDim2.new(1, 0, 0, 24), C.card, C.sub)
bombScan.TextSize = 12
bombScan.Font = Enum.Font.GothamMedium
bombScan.LayoutOrder = #pageExtra:GetChildren()

-- ===== DRAG / SHORTCUT =====
local dragging, dragStart, startPos = false, nil, nil
header.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = i.Position
        startPos = frame.Position
        i.Changed:Connect(function()
            if i.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
local inputConn = UIS.InputChanged:Connect(function(i)
    if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
        local d = i.Position - dragStart
        frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
    end
end)
local keyConn = UIS.InputBegan:Connect(function(i, gp)
    if not gp and i.KeyCode == Enum.KeyCode.RightShift then toggleMin() end
end)

local idleConn = player.Idled:Connect(function()
    if antiKick then
        pcall(function()
            VirtualUser:CaptureController()
            VirtualUser:ClickButton2(Vector2.new())
        end)
    end
end)

-- ===== TARGETS =====
local targets, hotspots, bombs, bombPos = {}, {}, {}, {}
local BOMB_KEYS = {"bomb", "explos", "seamine"}

local function addBomb(part)
    local p = part.Position
    for _, e in ipairs(bombPos) do
        if (e.pos - p).Magnitude < 8 then return end
    end
    table.insert(bombPos, {pos = p, lastHit = -math.huge, id = #bombPos + 1})
end

local function hotspotPos()
    for h in pairs(hotspots) do
        if h.Parent then return h:GetPivot().Position end
        hotspots[h] = nil
    end
end

local function register(inst)
    if inst:IsA("BasePart") then
        local cur, depth = inst, 0
        while cur and cur ~= workspace and depth < 4 do
            local bn = (cur.Name:lower():gsub("%s", ""))
            for _, k in ipairs(BOMB_KEYS) do
                if bn:find(k, 1, true) then
                    bombs[inst] = true
                    addBomb(inst)
                    return
                end
            end
            cur = cur.Parent
            depth += 1
        end
    end

    if not (inst:IsA("BasePart") or inst:IsA("Model") or inst:IsA("BillboardGui")) then return end
    local n = (inst.Name:lower():gsub("%s", ""))

    if n:find("hotspot", 1, true) then
        if inst:IsA("BasePart") or inst:IsA("Model") then
            hotspots[inst] = true
        elseif inst:IsA("BillboardGui") then
            local a = inst.Adornee or inst.Parent
            if a and a:IsA("BasePart") then hotspots[a] = true end
        end
        return
    end
    if not (inst:IsA("BasePart") or inst:IsA("Model")) then return end
    if inst:IsA("Model") and inst:FindFirstChildOfClass("Humanoid") then return end
    for i, c in ipairs(CATEGORIES) do
        if c.key and n:find(c.key, 1, true) then
            targets[inst] = i
            return
        end
    end
end
for _, inst in ipairs(workspace:GetDescendants()) do register(inst) end
local addedConn = workspace.DescendantAdded:Connect(register)
local removingConn = workspace.DescendantRemoving:Connect(function(inst)
    targets[inst] = nil
    hotspots[inst] = nil
    bombs[inst] = nil
end)

local function getRoot()
    local char = player.Character
    return char and (char.PrimaryPart or char:FindFirstChild("HumanoidRootPart")), char
end

goHotspot = function()
    local hp = hotspotPos()
    local root = getRoot()
    if hp and root then
        root.CFrame = CFrame.new(hp + Vector3.new(0, 4, 0))
        root.AssemblyLinearVelocity = Vector3.zero
    end
end

scan.Activated:Connect(function()
    print("[TP Menu] --- Hotspot scan ---")
    local words = {"hotspot", "dome", "zone", "spot"}
    local shown = 0
    for _, inst in ipairs(workspace:GetDescendants()) do
        if inst:IsA("BasePart") or inst:IsA("Model") or inst:IsA("BillboardGui") then
            local n = inst.Name:lower()
            for _, w in ipairs(words) do
                if n:find(w, 1, true) then
                    shown += 1
                    if shown <= 25 then print(inst.ClassName, inst:GetFullName()) end
                    break
                end
            end
        end
    end
    print("[TP Menu] Total:", shown, "| Hotspot registered:", hotspotPos() ~= nil)
end)

local function getPart(inst)
    if inst:IsA("BasePart") then return inst end
    return inst.PrimaryPart or inst:FindFirstChildWhichIsA("BasePart", true)
end

local hasTouch = typeof(firetouchinterest) == "function"
local hasClick = typeof(fireclickdetector) == "function"
local hasPrompt = typeof(fireproximityprompt) == "function"

local function interact(char, part)
    if hasTouch then
        for _, p in ipairs(char:GetChildren()) do
            if p:IsA("BasePart") then
                pcall(function()
                    firetouchinterest(p, part, 0)
                    firetouchinterest(p, part, 1)
                end)
            end
        end
    end
    local cd = part:FindFirstChildWhichIsA("ClickDetector", true)
    if cd and hasClick then pcall(fireclickdetector, cd) end
    local pp = part:FindFirstChildWhichIsA("ProximityPrompt", true)
    if pp and hasPrompt then pcall(fireproximityprompt, pp) end
end

-- ===== ABILITY (PUFF UP / F) =====
local abilityLabel, lastScan = nil, 0
local function findAbility()
    if abilityLabel and abilityLabel.Parent then return abilityLabel end
    if os.clock() - lastScan < 2 then return nil end
    lastScan = os.clock()
    for _, d in ipairs(playerGui:GetDescendants()) do
        if d:IsA("TextLabel") and (d.Text:lower():gsub("%s", "")) == "puffup" then
            abilityLabel = d
            return d
        end
    end
end

local function abilityState()
    local lbl = findAbility()
    if not lbl then return false, false end
    local scope = lbl.Parent
    for _ = 1, 2 do
        if not scope then break end
        for _, d in ipairs(scope:GetDescendants()) do
            if d:IsA("TextLabel") and d ~= lbl and d.Visible and d.TextTransparency < 1 then
                local t = (d.Text:gsub("%s", ""))
                if t:upper() == "F" then return true, true end
                if tonumber(t) then return false, true end
            end
        end
        scope = scope.Parent
    end
    return false, false
end

local function pressF()
    if UIS:GetFocusedTextBox() then return end
    local ok = pcall(function()
        VIM:SendKeyEvent(true, Enum.KeyCode.F, false, game)
        task.wait(0.05)
        VIM:SendKeyEvent(false, Enum.KeyCode.F, false, game)
    end)
    if not ok and typeof(keypress) == "function" then
        pcall(keypress, 0x46)
        task.wait(0.05)
        pcall(keyrelease, 0x46)
    end
end

-- ===== UI HELPERS (used by DIE / respawn detection) =====
local function guiVisible(o)
    local cur = o
    while cur and cur ~= playerGui do
        if cur:IsA("GuiObject") and not cur.Visible then return false end
        if cur:IsA("ScreenGui") and not cur.Enabled then return false end
        cur = cur.Parent
    end
    return cur == playerGui
end

local function findText(want, exact)
    for _, d in ipairs(playerGui:GetDescendants()) do
        if (d:IsA("TextLabel") or d:IsA("TextButton")) and not d:IsDescendantOf(gui) then
            local t = d.Text:lower():gsub("^%s+", ""):gsub("%s+$", "")
            local hit
            if exact then
                hit = (t == want)
            else
                hit = t:find(want, 1, true) ~= nil
            end
            if hit and guiVisible(d) then return d end
        end
    end
end

local function isDead(startChar)
    local c = player.Character
    if not startChar or not startChar.Parent or c ~= startChar then return true end
    local hum = c:FindFirstChildOfClass("Humanoid")
    if hum and hum.Health <= 0 then return true end
    if findText("you are dead", false) then return true end
    return false
end

-- ===== SAFESPOT =====
local safePart, safeReturn = nil, nil

setSafe = function(v)
    safeOn = v
    local root = getRoot()
    if v then
        if root then
            safeReturn = root.CFrame
            local p = root.Position
            local pos = Vector3.new(p.X, SAFE_HEIGHT, p.Z)
            if not safePart or not safePart.Parent then
                safePart = Instance.new("Part")
                safePart.Name = "SafeSpotPlatform"
                safePart.Anchored = true
                safePart.Size = Vector3.new(80, 2, 80)
                safePart.Material = Enum.Material.Neon
                safePart.Color = Color3.fromRGB(64, 156, 255)
                safePart.Transparency = 0.4
                safePart.Parent = workspace
            end
            safePart.Position = pos
            root.CFrame = CFrame.new(pos + Vector3.new(0, 4, 0))
            root.AssemblyLinearVelocity = Vector3.zero
        end
    else
        if root and safeReturn then
            root.CFrame = safeReturn
            root.AssemblyLinearVelocity = Vector3.zero
        end
        if safePart then
            safePart:Destroy()
            safePart = nil
        end
        safeReturn = nil
    end
end

task.spawn(function()
    while gui.Parent do
        if safeOn and not dying and safePart and safePart.Parent then
            local root = getRoot()
            if root then
                local pp = safePart.Position
                local off = root.Position - pp
                if off.Y < -10 or Vector3.new(off.X, 0, off.Z).Magnitude > 60 then
                    root.CFrame = CFrame.new(pp + Vector3.new(0, 4, 0))
                    root.AssemblyLinearVelocity = Vector3.zero
                end
            end
        end
        task.wait(0.2)
    end
end)

-- ===== DIE (BOMBS) =====
local function bombsNear(pos)
    local list = {}
    for b in pairs(bombs) do
        if b.Parent and (b.Position - pos).Magnitude < 10 then
            table.insert(list, b)
        end
    end
    table.sort(list, function(a, c)
        return (a.Position - pos).Magnitude < (c.Position - pos).Magnitude
    end)
    return list
end

local function bombDeath(timeout, say)
    local startChar = player.Character
    local start = os.clock()
    local lastId

    while os.clock() - start < timeout and gui.Parent do
        if isDead(startChar) then return "died" end

        local root, char = getRoot()
        if not root then
            task.wait(0.2)
        elseif #bombPos == 0 then
            return "nobombs"
        else
            local now = os.clock()
            local cands = {}
            for _, e in ipairs(bombPos) do
                if now - e.lastHit >= BOMB_COOLDOWN then table.insert(cands, e) end
            end

            if #cands == 0 then
                say("Waiting for bombs to reset...")
                task.wait(0.3)
            else
                table.sort(cands, function(a, b)
                    if a.lastHit ~= b.lastHit then return a.lastHit < b.lastHit end
                    return (a.pos - root.Position).Magnitude < (b.pos - root.Position).Magnitude
                end)
                local e = cands[1]
                if e.id == lastId and #cands > 1 then e = cands[2] end
                lastId = e.id

                say(string.format("Bomb %d/%d", e.id, #bombPos))
                root.CFrame = CFrame.new(e.pos + Vector3.new(0, 15, 0))
                root.AssemblyLinearVelocity = Vector3.zero
                pcall(function() player:RequestStreamAroundAsync(e.pos, 1.5) end)
                task.wait(0.15)

                local parts = bombsNear(e.pos)
                if #parts > 0 and root.Parent then
                    root.CFrame = parts[1].CFrame
                    root.AssemblyLinearVelocity = Vector3.zero
                    for _, p in ipairs(parts) do interact(char, p) end
                    task.wait(0.6)
                    e.lastHit = os.clock()
                else
                    e.lastHit = os.clock() - (BOMB_COOLDOWN - 2)
                    task.wait(0.2)
                end
            end
        end
    end
    return "timeout"
end

local function doDie()
    if dying then return end
    dying = true
    if safeOn then
        setSafe(false)
        setSafeToggle(false)
    end
    task.spawn(function()
        local r = bombDeath(DIE_TIMEOUT, function(s) dieStatus.Text = s end)
        dying = false
        if r == "died" then
            dieStatus.Text = "Fish died."
        elseif r == "nobombs" then
            dieStatus.Text = "no bombs found (use Scan bombs)"
        else
            dieStatus.Text = "timed out"
        end
    end)
end

local diePending = 0
dieBtn.Activated:Connect(function()
    if dying then return end
    if os.clock() - diePending > 3 then
        diePending = os.clock()
        dieBtn.Text = "Confirm? click again"
        task.delay(3, function()
            if not dying then dieBtn.Text = "💀 DIE" end
        end)
        return
    end
    diePending = 0
    dieBtn.Text = "💀 DIE"
    doDie()
end)

bombScan.Activated:Connect(function()
    print("[TP Menu] --- Bomb scan ---")
    for _, e in ipairs(bombPos) do
        print(string.format("Position %d: %.0f, %.0f, %.0f", e.id, e.pos.X, e.pos.Y, e.pos.Z))
    end
    local loaded = 0
    for b in pairs(bombs) do
        if b.Parent then loaded += 1 end
    end
    print("[TP Menu] Known positions:", #bombPos, "| Parts currently loaded:", loaded)
    local words = {"bomb", "mine", "explos", "hazard", "trap", "spike", "danger"}
    local shown = 0
    for _, inst in ipairs(workspace:GetDescendants()) do
        if inst:IsA("BasePart") or inst:IsA("Model") then
            local n = inst.Name:lower()
            for _, w in ipairs(words) do
                if n:find(w, 1, true) then
                    shown += 1
                    if shown <= 25 then print("Candidate:", inst.ClassName, inst:GetFullName()) end
                    break
                end
            end
        end
    end
    print("[TP Menu] Candidates by name:", shown)
end)

-- ===== COLLECT (TP) =====
local function collectCycleRaw(sel, useHot, perCycle, hop)
    local root, char = getRoot()
    if not root then return 0 end

    local home
    if useHot then
        local hp = hotspotPos()
        if hp then home = CFrame.new(hp + Vector3.new(0, 4, 0)) end
    end

    local list = {}
    for inst, cat in pairs(targets) do
        if inst.Parent then
            if sel[cat] then
                local part = getPart(inst)
                if part then
                    local d = (part.Position - root.Position).Magnitude
                    if d <= MAX_DISTANCE then table.insert(list, {part = part, dist = d}) end
                end
            end
        else
            targets[inst] = nil
        end
    end
    table.sort(list, function(a, b) return a.dist < b.dist end)

    local count = #list
    local n = math.min(count, perCycle)
    if n == 0 then return 0 end

    local origin = home or root.CFrame
    for i = 1, n do
        local part = list[i].part
        if part.Parent and root.Parent then
            if TELEPORT then
                root.CFrame = part.CFrame
                root.AssemblyLinearVelocity = Vector3.zero
            end
            interact(char, part)
            if hop then
                task.wait(hop * (0.75 + math.random() * 0.5))
            else
                RunService.Heartbeat:Wait()
                RunService.Heartbeat:Wait()
            end
        end
    end
    if TELEPORT and root.Parent then
        root.CFrame = origin
        root.AssemblyLinearVelocity = Vector3.zero
    end
    return count
end

local function collectCycle(sel, useHot, perCycle, hop)
    collecting = true
    local ok, r = pcall(collectCycleRaw, sel, useHot, perCycle, hop)
    collecting = false
    if ok then return r end
    error(r)
end

-- ===== LOOPS =====
local enabled = false

task.spawn(function()
    while gui.Parent do
        if enabled and not afkOn and not safeOn and not dying then
            local ok, count = pcall(collectCycle, selected, selected[HOT], MAX_PER_CYCLE, nil)
            if ok then master.Text = "TP: ON (" .. tostring(count) .. ")" end
        end
        task.wait(INTERVAL)
    end
end)

task.spawn(function()
    while gui.Parent do
        if afkOn and not safeOn and not dying then
            local p = AFK_PRESETS[afkPreset]
            local ok, count = pcall(collectCycle, allSel, afkHot, p.per, p.hop)
            if ok then
                afkCount = count
                afkBtn.Text = "PUFFER AFK: ON (" .. tostring(count) .. ")"
            end
            task.wait(p.pause * (0.8 + math.random() * 0.4))
        else
            task.wait(INTERVAL)
        end
    end
end)

task.spawn(function()
    local lastF = 0
    while gui.Parent do
        local hp = hotspotPos()
        local ready, known = abilityState()
        local puff = known and (ready and "ready" or "cooling down") or "not detected"

        local txt = "Spot: " .. (hp and "found" or "not found") .. "  |  Puff Up: " .. puff
        if status.Text ~= txt then status.Text = txt end

        local afkTxt
        if afkOn then
            afkTxt = "AFK active (" .. AFK_PRESETS[afkPreset].name .. ") | targets: " .. afkCount .. "\nSpot: " .. (hp and "found" or "not found") .. " | Puff Up: " .. puff
            if safeOn then afkTxt = afkTxt .. "\n(paused: SafeSpot on)" end
        else
            afkTxt = "AFK: off"
        end
        if afkStatus.Text ~= afkTxt then afkStatus.Text = afkTxt end

        local paused = safeOn or dying
        local holdHot = ((selected[HOT]) or (afkOn and afkHot)) and hp
        if holdHot and not afkOn and not paused then
            local root = getRoot()
            if root and (root.Position - hp).Magnitude > HOTSPOT_RADIUS then
                root.CFrame = CFrame.new(hp + Vector3.new(0, 4, 0))
                root.AssemblyLinearVelocity = Vector3.zero
            end
        end

        if (holdHot or afkOn) and not paused then
            local now = os.clock()
            if known then
                if ready and now - lastF > 0.4 then
                    pressF()
                    lastF = now
                end
            elseif now - lastF >= FALLBACK_F then
                pressF()
                lastF = now
            end
        end
        task.wait(0.1)
    end
end)

master.Activated:Connect(function()
    enabled = not enabled
    master.Text = enabled and "TP: ON" or "TP: OFF"
    TweenService:Create(master, TweenInfo.new(0.15), {BackgroundColor3 = enabled and C.on or C.off}):Play()
end)

afkBtn.Activated:Connect(function()
    afkOn = not afkOn
    afkBtn.Text = afkOn and "PUFFER AFK: ON" or "PUFFER AFK: OFF"
    TweenService:Create(afkBtn, TweenInfo.new(0.15), {BackgroundColor3 = afkOn and C.on or C.off}):Play()
    if afkOn and afkHot and not safeOn then goHotspot() end
end)

-- ===== ESP =====
local espFolder = Instance.new("Folder")
espFolder.Name = "ESP"
espFolder.Parent = gui
local espObjs = {}

local function makeTag(adornee, color)
    local bb = Instance.new("BillboardGui")
    bb.Adornee = adornee
    bb.AlwaysOnTop = true
    bb.Size = UDim2.fromOffset(120, 24)
    bb.StudsOffset = Vector3.new(0, 2, 0)
    bb.Parent = espFolder
    local l = Instance.new("TextLabel")
    l.Size = UDim2.fromScale(1, 1)
    l.BackgroundTransparency = 1
    l.TextColor3 = color
    l.TextStrokeTransparency = 0.3
    l.Font = Enum.Font.GothamBold
    l.TextSize = 12
    l.Text = ""
    l.Parent = bb
    return bb, l
end

local function removeEsp(key)
    local e = espObjs[key]
    if e then
        e.tag:Destroy()
        if e.hl then e.hl:Destroy() end
        espObjs[key] = nil
    end
end

task.spawn(function()
    while gui.Parent do
        local seen, count = {}, 0
        local root = getRoot()

        if espMaster and root then
            local function show(key, part, name, color, hlTarget, range)
                local d = (part.Position - root.Position).Magnitude
                if d > range then return end
                local e = espObjs[key]
                if not e then
                    if count >= ESP_MAX then return end
                    local bb, l = makeTag(part, color)
                    e = {tag = bb, label = l}
                    if hlTarget then
                        local h = Instance.new("Highlight")
                        h.Adornee = hlTarget
                        h.FillColor = color
                        h.OutlineColor = color
                        h.FillTransparency = 0.75
                        h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                        h.Parent = espFolder
                        e.hl = h
                    end
                    espObjs[key] = e
                end
                seen[key] = true
                count += 1
                e.label.Text = string.format("%s [%d]", name, math.floor(d))
            end

            for inst, cat in pairs(targets) do
                if espSel[cat] and inst.Parent then
                    local part = getPart(inst)
                    if part then show(inst, part, inst.Name, COLORS[cat], nil, ESP_RANGE) end
                end
            end

            if espSel[HOT] then
                for h in pairs(hotspots) do
                    if h.Parent then
                        local part = getPart(h)
                        if part then show(h, part, "HOTSPOT", COLORS[HOT], h, ESP_RANGE) end
                    end
                end
            end

            if espSel[PLAYERS_ESP] then
                for _, p in ipairs(Players:GetPlayers()) do
                    local ch = p ~= player and p.Character
                    local pr = ch and (ch:FindFirstChild("HumanoidRootPart") or ch.PrimaryPart)
                    if pr then show(ch, pr, p.Name, COLORS[PLAYERS_ESP], ch, PLAYER_ESP_RANGE) end
                end
            end
        end

        for key in pairs(espObjs) do
            if not seen[key] then removeEsp(key) end
        end
        task.wait(0.2)
    end
end)

-- ===== SPEED =====
local speedConn = RunService.Heartbeat:Connect(function(dt)
    local root, char = getRoot()
    if not root then return end
    local hum = char:FindFirstChildOfClass("Humanoid")

    if not speedOn then
        if origWalk and hum then
            hum.WalkSpeed = origWalk
            origWalk = nil
        end
        return
    end

    if method == "WalkSpeed" then
        if hum then
            if not origWalk then origWalk = hum.WalkSpeed end
            hum.WalkSpeed = origWalk * mult
        end
        return
    end

    if origWalk and hum then
        hum.WalkSpeed = origWalk
        origWalk = nil
    end

    local vel = root.AssemblyLinearVelocity
    local speed = vel.Magnitude
    local dir = hum and hum.MoveDirection or Vector3.zero
    if dir.Magnitude < 0.1 then
        if speed < 2 then return end
        dir = vel.Unit
    else
        dir = dir.Unit
        speed = math.max(speed, hum.WalkSpeed)
    end
    root.CFrame = root.CFrame + dir * (mult - 1) * speed * dt
end)

gui.Destroying:Connect(function()
    enabled = false
    afkOn = false
    speedOn = false
    espMaster = false
    dying = false
    if safeOn then pcall(setSafe, false) end
    if safePart then safePart:Destroy() end
    local _, char = getRoot()
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hum and origWalk then hum.WalkSpeed = origWalk end
    addedConn:Disconnect()
    removingConn:Disconnect()
    inputConn:Disconnect()
    keyConn:Disconnect()
    idleConn:Disconnect()
    speedConn:Disconnect()
end)

print("[TP Menu] Loaded. touch:", hasTouch, "| parent:", guiParent.Name)

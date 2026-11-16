# Myluascripts
My personal Lua scripts and Roblox Studio project

-- CART3R.JAILBREAK
-- PLAYER DEBUG ESP v2.0

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local MAX_DISTANCE = 500
local UPDATE_INTERVAL = 0.1

local enabled = false
local drawings = {}
local lastUpdate = 0

local gui = Instance.new("ScreenGui")
gui.Name = "CART3R_ESP"
gui.ResetOnSpawn = false
gui.Parent = PlayerGui

local toggle = Instance.new("TextButton")
toggle.Size = UDim2.fromOffset(180, 45)
toggle.Position = UDim2.fromOffset(20, 100)
toggle.BackgroundColor3 = Color3.fromRGB(35, 10, 15)
toggle.TextColor3 = Color3.new(1, 1, 1)
toggle.Text = "ESP: OFF"
toggle.TextSize = 18
toggle.Parent = gui

local function getTeamColor(player)
    if player.Team then
        return player.Team.TeamColor.Color
    end

    return Color3.fromRGB(255, 70, 70)
end

local function removeESP(player)
    if drawings[player] then
        drawings[player]:Destroy()
        drawings[player] = nil
    end
end

local function createESP(player)
    removeESP(player)

    if player == LocalPlayer then
        return
    end

    if player:GetAttribute("CanUsePlayerESP") ~= true then
        return
    end

    local character = player.Character
    if not character then return end

    local head = character:FindFirstChild("Head")
    local humanoid = character:FindFirstChildOfClass("Humanoid")

    if not head or not humanoid then return end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "PlayerDebugInfo"
    billboard.Adornee = head
    billboard.Size = UDim2.fromOffset(180, 65)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true
    billboard.Enabled = false
    billboard.Parent = gui

    local name = Instance.new("TextLabel")
    name.Size = UDim2.new(1, 0, 0.35, 0)
    name.BackgroundTransparency = 1
    name.TextScaled = true
    name.Font = Enum.Font.GothamBold
    name.Text = player.DisplayName
    name.TextColor3 = getTeamColor(player)
    name.Parent = billboard

    local distance = Instance.new("TextLabel")
    distance.Position = UDim2.new(0, 0, 0.35, 0)
    distance.Size = UDim2.new(1, 0, 0.3, 0)
    distance.BackgroundTransparency = 1
    distance.TextColor3 = Color3.new(1, 1, 1)
    distance.TextScaled = true
    distance.Parent = billboard

    local background = Instance.new("Frame")
    background.Position = UDim2.new(0.1, 0, 0.75, 0)
    background.Size = UDim2.new(0.8, 0, 0.15, 0)
    background.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    background.BorderSizePixel = 0
    background.Parent = billboard

    local health = Instance.new("Frame")
    health.Size = UDim2.fromScale(1, 1)
    health.BackgroundColor3 = Color3.fromRGB(50, 220, 90)
    health.BorderSizePixel = 0
    health.Parent = background

    drawings[player] = billboard

    return {
        gui = billboard,
        name = name,
        distance = distance,
        health = health,
        humanoid = humanoid,
        character = character,
    }
end

local function refreshPlayer(player)
    if player:GetAttribute("CanUsePlayerESP") ~= true then
        removeESP(player)
        return
    end

    createESP(player)
end

local function setupPlayer(player)
    player:GetAttributeChangedSignal("CanUsePlayerESP"):Connect(function()
        refreshPlayer(player)
    end)

    player.CharacterAdded:Connect(function()
        task.wait(1)
        refreshPlayer(player)
    end)

    player.CharacterRemoving:Connect(function()
        removeESP(player)
    end)

    if player.Character then
        refreshPlayer(player)
    end
end

for _, player in Players:GetPlayers() do
    setupPlayer(player)
end

Players.PlayerAdded:Connect(setupPlayer)
Players.PlayerRemoving:Connect(removeESP)

toggle.MouseButton1Click:Connect(function()
    enabled = not enabled
    toggle.Text = enabled and "ESP: ON" or "ESP: OFF"
    toggle.BackgroundColor3 = enabled
        and Color3.fromRGB(120, 20, 35)
        or Color3.fromRGB(35, 10, 15)
end)

RunService.RenderStepped:Connect(function()
    if not enabled then
        for _, billboard in pairs(drawings) do
            billboard.Enabled = false
        end
        return
    end

    if os.clock() - lastUpdate < UPDATE_INTERVAL then
        return
    end

    lastUpdate = os.clock()

    local myCharacter = LocalPlayer.Character
    local myRoot = myCharacter
        and myCharacter:FindFirstChild("HumanoidRootPart")

    if not myRoot then return end

    for player, billboard in pairs(drawings) do
        local character = player.Character
        local root = character
            and character:FindFirstChild("HumanoidRootPart")

        local humanoid = character
            and character:FindFirstChildOfClass("Humanoid")

        if not root or not humanoid or humanoid.Health <= 0 then
            billboard.Enabled = false
            continue
        end

        local distance = (myRoot.Position - root.Position).Magnitude

        billboard.Enabled = distance <= MAX_DISTANCE

        if billboard.Enabled then
            local info = billboard

            local name = info:FindFirstChildOfClass("TextLabel")
            if name then
                name.Text = player.DisplayName
                name.TextColor3 = getTeamColor(player)
            end

            local healthBackground = info:FindFirstChildOfClass("Frame")
            local healthBar = healthBackground
                and healthBackground:FindFirstChildOfClass("Frame")

            local labels = {}
            for _, item in info:GetChildren() do
                if item:IsA("TextLabel") then
                    table.insert(labels, item)
                end
            end

            if #labels >= 2 then
                labels[2].Text = math.floor(distance) .. " studs"
            end

            if healthBar then
                healthBar.Size = UDim2.new(
                    math.clamp(humanoid.Health / humanoid.MaxHealth, 0, 1),
                    0, 1, 0
                )
            end
        end
    end
end)

print("[CART3R] Player ESP v2.0 loaded")

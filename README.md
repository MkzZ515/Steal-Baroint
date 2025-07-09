--[[
Hub ESP & Speed (ESP NAME 100% corrigida)
- Speed hack (ajustável)
- ESP Box e ESP Name (todas em branco)
- ESP Cooldown das bases (ajustável)
- Usar LOCAL SCRIPT em StarterGui para testar em ambiente local!
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- CONFIG
local config = {
    espName = true,
    espBox = true,
    espNameColor = Color3.new(1,1,1),
    espBoxColor = Color3.new(1,1,1),
    espBaseColor = Color3.new(1,1,1),
    speedEnabled = true,
    speedValue = 200,
}

-- ESP MANAGEMENT
local function removeOldESP(char)
    if char and char:FindFirstChild("Head") then
        for _,v in pairs(char.Head:GetChildren()) do
            if v:IsA("BillboardGui") and v.Name == "ESP_Name" then
                v:Destroy()
            end
        end
    end
    if char and char:FindFirstChild("HumanoidRootPart") then
        for _,v in pairs(char.HumanoidRootPart:GetChildren()) do
            if v:IsA("BoxHandleAdornment") and v.Name == "ESP_Box" then
                v:Destroy()
            end
        end
    end
end

local function createNameESP(char, plr)
    removeOldESP(char)
    if not char or not char:FindFirstChild("Head") then return end
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP_Name"
    billboard.Adornee = char.Head
    billboard.Size = UDim2.new(0, 120, 0, 24)
    billboard.StudsOffset = Vector3.new(0, 2.5, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = char.Head

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 1, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextStrokeTransparency = 0.5
    nameLabel.Text = plr.DisplayName or plr.Name
    nameLabel.TextColor3 = config.espNameColor
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextSize = 16
    nameLabel.Parent = billboard
end

local function createBoxESP(char)
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local adornment = Instance.new("BoxHandleAdornment")
    adornment.Name = "ESP_Box"
    adornment.Adornee = char.HumanoidRootPart
    adornment.Size = Vector3.new(4,7,2)
    adornment.Color3 = config.espBoxColor
    adornment.Transparency = 0.4
    adornment.ZIndex = 10
    adornment.AlwaysOnTop = true
    adornment.Parent = char.HumanoidRootPart
end

local function updateESPs()
    for _,plr in ipairs(Players:GetPlayers()) do
        if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChild("Head") then
            if config.espName then
                createNameESP(plr.Character, plr)
            end
            if config.espBox then
                createBoxESP(plr.Character)
            end
        end
    end
end

-- SPEED
local function updateSpeed()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if config.speedEnabled then
            hum.WalkSpeed = config.speedValue
        else
            hum.WalkSpeed = 16
        end
    end
end

-- ESP BASES (ajuste para seu jogo se necessário)
local function getBases()
    local bases = {}
    for _,obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj.Name:lower():find("base") and obj.PrimaryPart then
            table.insert(bases, obj)
        end
    end
    return bases
end

local function getBaseCooldown(base)
    if base:FindFirstChild("Cooldown") and base.Cooldown:IsA("NumberValue") then
        return base.Cooldown.Value
    elseif base:GetAttribute("Cooldown") then
        return base:GetAttribute("Cooldown")
    end
    return nil
end

local function addBaseESP(base, txt)
    if not base.PrimaryPart then return end
    for _,v in pairs(base.PrimaryPart:GetChildren()) do
        if v:IsA("BillboardGui") and v.Name == "ESP_BASE" then v:Destroy() end
    end
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP_BASE"
    billboard.Adornee = base.PrimaryPart
    billboard.Size = UDim2.new(0, 200, 0, 30)
    billboard.StudsOffset = Vector3.new(0, 5, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = base.PrimaryPart

    local label = Instance.new("TextLabel", billboard)
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = txt
    label.TextColor3 = config.espBaseColor
    label.Font = Enum.Font.GothamBold
    label.TextSize = 18
    label.TextStrokeTransparency = 0.5
end

local function updateBaseESP()
    for _,base in ipairs(getBases()) do
        local cd = getBaseCooldown(base)
        local txt = base.Name .. " | CD: "
        if cd then
            txt = txt .. tostring(cd)
        else
            txt = txt .. "N/A"
        end
        addBaseESP(base, txt)
    end
end

-- CONEXÕES
Players.PlayerAdded:Connect(function(p)
    p.CharacterAdded:Connect(function(char)
        RunService.RenderStepped:Wait()
        updateESPs()
    end)
end)
Players.PlayerRemoving:Connect(function()
    updateESPs()
end)
RunService.RenderStepped:Connect(function()
    updateESPs()
    updateBaseESP()
    updateSpeed()
end)
LocalPlayer.CharacterAdded:Connect(function()
    wait(0.2)
    updateSpeed()
end)

print("ESP NAME agora deve aparecer corretamente acima de todos os players!")

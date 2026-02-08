if game.PlaceId ~= 131623223084840 then return end

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local VirtualUser = game:GetService("VirtualUser")

local player = Players.LocalPlayer

-- Anti AFK
player.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new())
end)

-- Character setup
local hrp, humanoid
local function setupChar(char)
    hrp = char:WaitForChild("HumanoidRootPart")
    humanoid = char:WaitForChild("Humanoid")
end

if player.Character then
    setupChar(player.Character)
end

player.CharacterAdded:Connect(function(c)
    task.wait(1)
    setupChar(c)
end)

-- UI Init
local WindUI = loadstring(game:HttpGet("https://raw.githubusercontent.com/Footagesus/WindUI/main/dist/main.lua"))()
local Window = WindUI:CreateWindow({
    Title = "Tsunami Brainrot Hub",
    Icon = "monitor",
    Author = "By ErrorServer",
    Folder = "TsunamiBrainrotHub",
    Size = UDim2.fromOffset(560, 430),
    Theme = "Crimson",
    Transparent = true,
    Resizable = true,
    User = { Enabled = true }
})

local FarmTab = Window:Tab({ Title = "Farm", Icon = "zap" })
local UpgradeTab = Window:Tab({ Title = "Upgrade", Icon = "trending-up" })
local ThemeTab = Window:Tab({ Title = "Theme", Icon = "palette" })

-- Positions
local BASE_POS = Vector3.new(130, 3, 0)
local SECRET_POS = Vector3.new(2440, 3, -4)

-- =========================
-- MAIN ACTIONS
-- =========================
FarmTab:Section({ Title = "Main Actions" })

FarmTab:Button({
    Title = "Teleport to Spawn",
    Callback = function()
        if hrp then hrp.CFrame = CFrame.new(BASE_POS) end
    end
})

FarmTab:Button({
    Title = "Find Best Brainrot",
    Callback = function()
        if not hrp then return end
        local secretFolder = workspace:FindFirstChild("ActiveBrainrots") and workspace.ActiveBrainrots:FindFirstChild("Secret")
        if secretFolder then
            local items = secretFolder:GetChildren()
            if #items > 0 then
                local target = items[1]:FindFirstChild("Handle")
                if target then
                    hrp.CFrame = target.CFrame + Vector3.new(0, 3, 0)
                else
                    hrp.CFrame = CFrame.new(SECRET_POS)
                end
            else
                hrp.CFrame = CFrame.new(SECRET_POS)
            end
        else
            hrp.CFrame = CFrame.new(SECRET_POS)
        end
    end
})

FarmTab:Button({
    Title = "Remove Tsunami",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/slowzzxX/Remove-wallters/refs/heads/main/Ff"))()
    end
})

-- =========================
-- GOD MODE
-- =========================
FarmTab:Section({ Title = "Protections" })

local godMode = false

FarmTab:Toggle({
    Title = "God Mode (Infinite)",
    Default = false,
    Callback = function(v)
        godMode = v
        if humanoid then
            if v then
                humanoid.MaxHealth = math.huge
                humanoid.Health = math.huge
            else
                humanoid.MaxHealth = 100
                humanoid.Health = humanoid.MaxHealth
            end
        end
    end
})

RunService.Stepped:Connect(function()
    if godMode and humanoid then
        if humanoid.Health < humanoid.MaxHealth then
            humanoid.Health = humanoid.MaxHealth
        end
    end
end)

-- =========================
-- AUTO FARM OP
-- =========================
FarmTab:Section({ Title = "Auto Farm OP" })

local autoFarm = false
local farmSpeed = 0.15

FarmTab:Toggle({
    Title = "Enable Auto Farm OP",
    Default = false,
    Callback = function(v)
        autoFarm = v
    end
})

FarmTab:Slider({
    Title = "Farm Speed",
    Value = { Min = 0.05, Max = 1, Default = 0.15 },
    Step = 0.05,
    Callback = function(v)
        farmSpeed = v
    end
})

task.spawn(function()
    while task.wait(farmSpeed) do
        if autoFarm and hrp then
            local folder = workspace:FindFirstChild("ActiveBrainrots")
            if folder then
                for _, v in pairs(folder:GetDescendants()) do
                    if v:IsA("BasePart") then
                        hrp.CFrame = v.CFrame + Vector3.new(0, 3, 0)
                        task.wait(0.05)
                    end
                end
            end
        end
    end
end)

-- =========================
-- MOVEMENT
-- =========================
FarmTab:Section({ Title = "Movement" })

local flying = false
local noclip = false
local flySpeed = 2

RunService.Stepped:Connect(function()
    if noclip and player.Character then
        for _, v in pairs(player.Character:GetDescendants()) do
            if v:IsA("BasePart") then
                v.CanCollide = false
            end
        end
    end
end)

FarmTab:Toggle({
    Title = "Noclip",
    Default = false,
    Callback = function(v) noclip = v end
})

FarmTab:Toggle({
    Title = "Fly Mode",
    Default = false,
    Callback = function(v)
        flying = v
        if humanoid then
            humanoid.PlatformStand = v
        end
    end
})

FarmTab:Slider({
    Title = "Fly Speed",
    Value = { Min = 1, Max = 10, Default = 2 },
    Step = 0.5,
    Callback = function(v) flySpeed = v end
})

RunService.RenderStepped:Connect(function()
    if flying and hrp and humanoid then
        local cam = workspace.CurrentCamera
        local md = humanoid.MoveDirection
        if md.Magnitude > 0 then
            local direction = (cam.CFrame.LookVector * -md.Z) + (cam.CFrame.RightVector * md.X)
            hrp.CFrame = hrp.CFrame + direction.Unit * flySpeed
        end
    end
end)

-- =========================
-- AUTO UPGRADE (EXPERIMENTAL)
-- =========================
UpgradeTab:Section({ Title = "Auto Upgrade" })

local autoUpgrade = false

UpgradeTab:Toggle({
    Title = "Enable Auto Upgrade",
    Default = false,
    Callback = function(v)
        autoUpgrade = v
    end
})

task.spawn(function()
    while task.wait(1) do
        if autoUpgrade then
            local rs = game:GetService("ReplicatedStorage")
            local upgradeEvent = rs:FindFirstChild("UpgradeEvent") or rs:FindFirstChild("RemoteEvents") and rs.RemoteEvents:FindFirstChild("Upgrade")
            if upgradeEvent then
                pcall(function()
                    upgradeEvent:FireServer()
                end)
            end
        end
    end
end)

-- Zotty's Hub - By Blessed
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local UserInputService = game:GetService("UserInputService")

-- UI Library
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window = Library.CreateLib("Zotty's Hub", "Ocean")

-- GUI toggle system
local GUIVisible = true
local GUIFrame = game:GetService("CoreGui"):FindFirstChild("Zotty's Hub")
if not GUIFrame then
    GUIFrame = Library
end

-- SETTINGS
local Settings = {
    WalkSpeed = 16,
    AimbotKey = Enum.KeyCode.E,
    AimbotEnabled = false,
    FlyEnabled = false,
    FOV = 100,
}

-- Player Tab
local PlayerTab = Window:NewTab("Player")
local PlayerSection = PlayerTab:NewSection("Movement")

PlayerSection:NewSlider("WalkSpeed", "Velocidade do personagem", 100, 16, function(v)
    Settings.WalkSpeed = v
    LocalPlayer.Character.Humanoid.WalkSpeed = v
end)

-- Fly
local flying = false
local function Fly()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    flying = true
    local bv = Instance.new("BodyVelocity")
    bv.Velocity = Vector3.zero
    bv.MaxForce = Vector3.new(1, 1, 1) * math.huge
    bv.Parent = char.HumanoidRootPart

    local conn = game:GetService("RunService").RenderStepped:Connect(function()
        local move = Vector3.new()
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then move += char.HumanoidRootPart.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then move -= char.HumanoidRootPart.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then move -= char.HumanoidRootPart.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then move += char.HumanoidRootPart.CFrame.RightVector end
        bv.Velocity = move * Settings.WalkSpeed
    end)

    repeat task.wait() until not flying
    conn:Disconnect()
    bv:Destroy()
end

PlayerSection:NewButton("Fly (WASD)", "Ativa voo com WASD", function()
    flying = not flying
    if flying then Fly() end
end)

-- ESP
local EspTab = Window:NewTab("ESP")
local EspSection = EspTab:NewSection("Visual")

local function CreateESP(plr)
    local box = Drawing.new("Square")
    box.Color = Color3.fromRGB(0, 255, 0)
    box.Thickness = 2
    box.Transparency = 1
    box.Filled = false

    local run
    run = game:GetService("RunService").RenderStepped:Connect(function()
        if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local pos, visible = workspace.CurrentCamera:WorldToViewportPoint(plr.Character.HumanoidRootPart.Position)
            if visible then
                box.Size = Vector2.new(60, 100)
                box.Position = Vector2.new(pos.X - 30, pos.Y - 50)
                box.Visible = true
            else
                box.Visible = false
            end
        else
            box.Visible = false
        end
    end)

    plr.CharacterRemoving:Connect(function()
        box:Remove()
        run:Disconnect()
    end)
end

EspSection:NewButton("Ativar ESP", "Caixas nos players", function()
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer then
            CreateESP(v)
        end
    end
    Players.PlayerAdded:Connect(function(plr)
        CreateESP(plr)
    end)
end)

-- Aimbot
local AimbotTab = Window:NewTab("Aimbot")
local AimbotSection = AimbotTab:NewSection("Configurações")

AimbotSection:NewKeybind("Bind Aimbot", "Tecla para mirar", Enum.KeyCode.E, function(key)
    Settings.AimbotKey = key
end)

AimbotSection:NewSlider("FOV", "Campo de visão", 500, 50, function(val)
    Settings.FOV = val
end)

local function GetClosest()
    local closest = nil
    local shortest = Settings.FOV
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local pos, visible = workspace.CurrentCamera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
            local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
            if dist < shortest and visible then
                shortest = dist
                closest = player
            end
        end
    end
    return closest
end

game:GetService("UserInputService").InputBegan:Connect(function(input, gpe)
    if input.KeyCode == Settings.AimbotKey and not gpe then
        local target = GetClosest()
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local pos = workspace.CurrentCamera:WorldToViewportPoint(target.Character.HumanoidRootPart.Position)
            mousemoverel((pos.X - Mouse.X), (pos.Y - Mouse.Y))
        end
    end
end)

-- TP com clique
local MiscTab = Window:NewTab("TP")
local MiscSection = MiscTab:NewSection("Teleport")

MiscSection:NewButton("Clique para teleportar", "TP onde clicar", function()
    Mouse.Button1Down:Connect(function()
        local target = Mouse.Hit
        if target then
            LocalPlayer.Character:MoveTo(target.Position)
        end
    end)
end)

-- BOTÕES GUI
local GUIControlTab = Window:NewTab("GUI")
local GUISection = GUIControlTab:NewSection("Controle da Interface")

GUISection:NewButton("🔴 Fechar GUI", "Oculta toda a interface", function()
    if GUIFrame then
        GUIFrame.Enabled = false
    end
end)

GUISection:NewButton("🟡 Minimizar Abas", "Oculta menus, mantém GUI", function()
    for _, v in pairs(GUIFrame:GetDescendants()) do
        if v:IsA("Frame") and v.Name == "Main" then
            v.Visible = false
        end
    end
end)

GUISection:NewButton("🟢 Restaurar GUI", "Mostra tudo novamente", function()
    if GUIFrame then
        GUIFrame.Enabled = true
        for _, v in pairs(GUIFrame:GetDescendants()) do
            if v:IsA("Frame") and v.Name == "Main" then
                v.Visible = true
            end
        end
    end
end)

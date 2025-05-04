-- Services
local P = game:GetService("Players")
local W = workspace.CurrentCamera
local L = P.LocalPlayer
local M = L:GetMouse()
local RS = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Variables
local S = {
    Aimbot = false,  -- Ativar/Desativar o Aimbot
    ESP = false,     -- Ativar/Desativar o ESP
    FOV = 100,       -- Raio do FOV
    AimPart = "Head", -- Parte do corpo a ser mirado
    TeamCheck = true, -- Verificar se o jogador é do mesmo time
    ESPBoxColor = Color3.fromRGB(255, 0, 0), -- Cor do quadrado do ESP
}

-- UI Hub - Dragon Hub
local Pg = L.PlayerGui
local SG = Instance.new("ScreenGui", Pg)
SG.Name = "DragonHub"
SG.ResetOnSpawn = false  -- Garante que o Hub persista
local Frame = Instance.new("Frame", SG)
Frame.Size = UDim2.new(0, 280, 0, 350)
Frame.Position = UDim2.new(0.05, 0, 0.1, 0)
Frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = true

-- Title
local Title = Instance.new("TextLabel", Frame)
Title.Text = "Dragon Hub"
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 20
Title.TextAlign = Enum.TextXAlignment.Center

-- Create Buttons
local function CreateButton(text, position, callback)
    local Button = Instance.new("TextButton", Frame)
    Button.Size = UDim2.new(0.9, 0, 0, 40)
    Button.Position = UDim2.new(0.05, 0, 0, position)
    Button.Text = text
    Button.TextColor3 = Color3.fromRGB(255, 255, 255)
    Button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    Button.Font = Enum.Font.Gotham
    Button.TextSize = 16
    Button.MouseButton1Click:Connect(callback)
    return Button
end

-- Aimbot Toggle Button
CreateButton("Toggle Aimbot", 50, function()
    S.Aimbot = not S.Aimbot
end)

-- ESP Toggle Button
CreateButton("Toggle ESP", 100, function()
    S.ESP = not S.ESP
end)

-- FOV Textbox
local FovInput = Instance.new("TextBox", Frame)
FovInput.Size = UDim2.new(0.9, 0, 0, 30)
FovInput.Position = UDim2.new(0.05, 0, 0, 150)
FovInput.Text = "FOV: " .. S.FOV
FovInput.TextColor3 = Color3.fromRGB(255, 255, 255)
FovInput.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
FovInput.Font = Enum.Font.Gotham
FovInput.TextSize = 16
FovInput.ClearTextOnFocus = false
FovInput.FocusLost:Connect(function()
    local inputFOV = tonumber(FovInput.Text:match("%d+"))
    if inputFOV and inputFOV > 0 then
        S.FOV = inputFOV
    end
end)

-- Minimize Button
CreateButton("Minimizar", 270, function()
    Frame.Visible = false
end)

-- Close Button
CreateButton("Fechar", 320, function()
    SG:Destroy()
end)

-- Functions for Aimbot and ESP
local fovCircle = Drawing.new("Circle")
fovCircle.Radius = S.FOV
fovCircle.Thickness = 2
fovCircle.Color = Color3.fromRGB(255, 255, 0)
fovCircle.Filled = false

-- Function to find the closest player within FOV
local function GetClosestPlayer()
    local closestPlayer = nil
    local closestDistance = S.FOV
    for _, player in ipairs(P:GetPlayers()) do
        if player ~= L and player.Character and player.Character:FindFirstChild(S.AimPart) then
            local target = player.Character[S.AimPart]
            local screenPos, onScreen = W:WorldToViewportPoint(target.Position)
            if onScreen then
                local distance = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(M.X, M.Y)).Magnitude
                if distance < closestDistance then
                    closestDistance = distance
                    closestPlayer = player
                end
            end
        end
    end
    return closestPlayer
end

-- Aimbot Function
local function Aimbot()
    if S.Aimbot then
        local targetPlayer = GetClosestPlayer()
        if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild(S.AimPart) then
            local aimPosition = targetPlayer.Character[S.AimPart].Position
            local targetCFrame = CFrame.new(W.CFrame.Position, aimPosition)
            W.CFrame = targetCFrame
        end
    end
end

-- ESP Function
local espBoxes = {}

local function CreateESPBox(player)
    local box = Drawing.new("Square")
    box.Thickness = 2
    box.Color = S.ESPBoxColor
    box.Filled = false
    box.Visible = false
    return box
end

-- Update ESP and Aimbot every frame
RS.RenderStepped:Connect(function()
    -- Update FOV Circle
    fovCircle.Position = Vector2.new(W.ViewportSize.X / 2, W.ViewportSize.Y / 2)

    -- Aimbot
    Aimbot()

    -- Update ESP
    if not S.ESP then
        for _, box in pairs(espBoxes) do
            box.Visible = false
        end
        return
    end

    for _, player in ipairs(P:GetPlayers()) do
        if player ~= L and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if not S.TeamCheck or player.Team ~= L.Team then
                local humanoidRootPart = player.Character.HumanoidRootPart
                local pos, onScreen = W:WorldToViewportPoint(humanoidRootPart.Position)

                if onScreen then
                    local sizeX = (W:WorldToViewportPoint(humanoidRootPart.Position + Vector3.new(2, 3, 0)).X
                                  - W:WorldToViewportPoint(humanoidRootPart.Position - Vector3.new(2, 3, 0)).X)

                    local box = espBoxes[player] or CreateESPBox(player)
                    espBoxes[player] = box

                    box.Size = Vector2.new(sizeX, sizeX * 1.5)
                    box.Position = Vector2.new(pos.X - box.Size.X / 2, pos.Y - box.Size.Y / 2)
                    box.Visible = true
                end
            end
        elseif espBoxes[player] then
            espBoxes[player].Visible = false
        end
    end
end)

-- Finalizing
fovCircle.Visible = true

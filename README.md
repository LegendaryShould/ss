-- MONOCROMA MENU COMPLETO
-- LocalScript en StarterPlayerScripts

local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CollectionService = game:GetService("CollectionService")
local TeleportService = game:GetService("TeleportService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- INTERFAZ
local gui = Instance.new("ScreenGui")
gui.Name = "MonocromaMenu"
gui.ResetOnSpawn = false
gui.Parent = playerGui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 250, 0, 355)
frame.Position = UDim2.new(0.05, 0, 0.25, 0)
frame.BackgroundColor3 = Color3.fromRGB(15, 20, 15)
frame.Active = true
frame.Parent = gui

Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)

-- TITULO
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 45)
title.BackgroundColor3 = Color3.fromRGB(20, 35, 20)
title.Text = "MONOCROMA MENU"
title.TextColor3 = Color3.fromRGB(0, 255, 80)
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.Active = true
title.Parent = frame

-- BOTONES
local function makeButton(text, y)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.85, 0, 0, 40)
    button.Position = UDim2.new(0.075, 0, 0, y)
    button.BackgroundColor3 = Color3.fromRGB(30, 50, 30)
    button.TextColor3 = Color3.new(1, 1, 1)
    button.TextSize = 14
    button.Font = Enum.Font.GothamBold
    button.Text = text
    button.Parent = frame

    Instance.new("UICorner", button).CornerRadius = UDim.new(0, 8)
    return button
end

local nightButton = makeButton("Vision nocturna: OFF", 50)
local wallsButton = makeButton("Ver paredes: OFF", 95)
local keyButton = makeButton("ESP llaves: OFF", 140)
local noclipButton = makeButton("No Clip: OFF", 185)
local returnButton = makeButton("Return to Lobby", 230)
local closeButton = makeButton("Cerrar menú", 275)

-- MENU MOVIBLE
local dragging = false
local dragStart
local startPos

title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (
        input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch
    ) then
        local delta = input.Position - dragStart

        frame.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- VISION NOCTURNA VERDE
local night = false
local effect
local oldClock
local oldBrightness

nightButton.MouseButton1Click:Connect(function()
    night = not night

    if night then
        oldClock = Lighting.ClockTime
        oldBrightness = Lighting.Brightness

        effect = Instance.new("ColorCorrectionEffect")
        effect.Name = "MonocromaNight"
        effect.TintColor = Color3.fromRGB(80, 255, 80)
        effect.Brightness = 0.15
        effect.Parent = Lighting

        Lighting.ClockTime = 0
        Lighting.Brightness = 3
    else
        if effect then
            effect:Destroy()
            effect = nil
        end

        Lighting.ClockTime = oldClock or 14
        Lighting.Brightness = oldBrightness or 1
    end

    nightButton.Text = "Vision nocturna: " ..
        (night and "ON" or "OFF")
end)

-- VER PAREDES
local wallsEnabled = false
local originalTransparency = {}

local function setWalls(enabled)
    wallsEnabled = enabled

    for _, part in ipairs(CollectionService:GetTagged("SeeThroughWall")) do
        if part:IsA("BasePart") then
            if enabled then
                if originalTransparency[part] == nil then
                    originalTransparency[part] =
                        part.LocalTransparencyModifier
                end

                part.LocalTransparencyModifier = 0.85
            else
                if originalTransparency[part] ~= nil then
                    part.LocalTransparencyModifier =
                        originalTransparency[part]
                    originalTransparency[part] = nil
                end
            end
        end
    end

    wallsButton.Text = "Ver paredes: " ..
        (enabled and "ON" or "OFF")
end

wallsButton.MouseButton1Click:Connect(function()
    setWalls(not wallsEnabled)
end)

-- ESP DE LLAVES
local keyESP = false
local highlights = {}

local function clearKeys()
    for obj, highlight in pairs(highlights) do
        if highlight then
            highlight:Destroy()
        end
        highlights[obj] = nil
    end
end

local function updateKeys()
    clearKeys()

    if not keyESP then return end

    for _, obj in ipairs(workspace:GetDescendants()) do
        if string.lower(obj.Name) == "key"
            and (obj:IsA("Model") or obj:IsA("BasePart")) then

            local h = Instance.new("Highlight")
            h.Name = "MonocromaKeyESP"
            h.Adornee = obj
            h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            h.FillColor = Color3.fromRGB(255, 220, 0)
            h.OutlineColor = Color3.fromRGB(255, 220, 0)
            h.FillTransparency = 0.5
            h.Parent = gui

            highlights[obj] = h
        end
    end
end

keyButton.MouseButton1Click:Connect(function()
    keyESP = not keyESP
    keyButton.Text = "ESP llaves: " ..
        (keyESP and "ON" or "OFF")
    updateKeys()
end)

task.spawn(function()
    while gui.Parent do
        if keyESP then
            updateKeys()
        end
        task.wait(2)
    end
end)

-- NO CLIP
local noclip = false
local originalCollision = {}

local function restoreCollision()
    for part, value in pairs(originalCollision) do
        if part and part.Parent then
            part.CanCollide = value
        end
    end
    table.clear(originalCollision)
end

noclipButton.MouseButton1Click:Connect(function()
    noclip = not noclip

    if not noclip then
        restoreCollision()
    end

    noclipButton.Text = "No Clip: " ..
        (noclip and "ON" or "OFF")
end)

RunService.Stepped:Connect(function()
    if not noclip then return end

    local character = player.Character
    if not character then return end

    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            if originalCollision[part] == nil then
                originalCollision[part] = part.CanCollide
            end
            part.CanCollide = false
        end
    end
end)

-- RETURN TO LOBBY
returnButton.MouseButton1Click:Connect(function()
    TeleportService:Teleport(
        134208374070897,
        Players.LocalPlayer
    )
end)

-- CERRAR MENU
closeButton.MouseButton1Click:Connect(function()
    gui.Enabled = false
end)

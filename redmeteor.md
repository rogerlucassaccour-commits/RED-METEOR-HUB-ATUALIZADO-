-- RED METEOR HUB!* 

local Players = game:GetService("Players")

local RunService = game:GetService("RunService")

local UserInputService = game:GetService("UserInputService")

local Workspace = game:GetService("Workspace")

local TweenService = game:GetService("TweenService")

-- Jogador local e câmera*

local LocalPlayer = Players.LocalPlayer

local Camera = Workspace.CurrentCamera

-- Variáveis de controle*

local espEnabled = false

local aimbotEnabled = false

local teamCheck = false

local wallCheck = false

local glowEnabled = false     -- Novo: controle para o efeito glow*

local targetPart = "Head"

local fovRadius = 150

local showFov = false

local fovRainbow = false

local espObjects = {}

local glowObjects = {}        -- Novo: objetos para o efeito glow*

local hue = 0

local smoothingFactor = 0.5   -- Novo: suavização do aimbot (0-1)*

-- Visual / UI
local COLORS = {
    BG = Color3.fromRGB(5, 5, 8),
    PANEL = Color3.fromRGB(12, 12, 16),
    CARD = Color3.fromRGB(19, 19, 24),
    CARD_HOVER = Color3.fromRGB(27, 27, 34),
    RED = Color3.fromRGB(220, 24, 38),
    RED_BRIGHT = Color3.fromRGB(255, 55, 65),
    RED_DARK = Color3.fromRGB(105, 8, 16),
    WHITE = Color3.fromRGB(245, 245, 248),
    MUTED = Color3.fromRGB(145, 145, 155),
    GREEN = Color3.fromRGB(45, 220, 115),
    OFF = Color3.fromRGB(70, 70, 78)
}

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RedMeteorHubFPS"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.DisplayOrder = 999

pcall(function()
    if syn and syn.protect_gui then
        syn.protect_gui(ScreenGui)
    end
    ScreenGui.Parent = game.CoreGui
end)

if not ScreenGui.Parent then
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

local function corner(parent, radius)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, radius or 10)
    c.Parent = parent
    return c
end

local function stroke(parent, color, transparency, thickness)
    local s = Instance.new("UIStroke")
    s.Color = color or COLORS.RED
    s.Transparency = transparency or 0
    s.Thickness = thickness or 1
    s.Parent = parent
    return s
end

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.fromOffset(430, 520)
MainFrame.Position = UDim2.new(0.5, -215, 0.5, -260)
MainFrame.BackgroundColor3 = COLORS.PANEL
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
corner(MainFrame, 16)
stroke(MainFrame, COLORS.RED, 0.35, 1.5)

local Shadow = Instance.new("ImageLabel", MainFrame)
Shadow.Name = "Shadow"
Shadow.BackgroundTransparency = 1
Shadow.Position = UDim2.fromOffset(-25, -25)
Shadow.Size = UDim2.new(1, 50, 1, 50)
Shadow.Image = "rbxassetid://5028857084"
Shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
Shadow.ImageTransparency = 0.35
Shadow.ScaleType = Enum.ScaleType.Slice
Shadow.SliceCenter = Rect.new(24, 24, 276, 276)
Shadow.ZIndex = 0

local Top = Instance.new("Frame", MainFrame)
Top.Name = "TopBar"
Top.Size = UDim2.new(1, 0, 0, 76)
Top.BackgroundColor3 = Color3.fromRGB(9, 9, 13)
Top.BorderSizePixel = 0
Top.ZIndex = 2
corner(Top, 16)

local TopMask = Instance.new("Frame", Top)
TopMask.Size = UDim2.new(1, 0, 0, 20)
TopMask.Position = UDim2.new(0, 0, 1, -20)
TopMask.BackgroundColor3 = Top.BackgroundColor3
TopMask.BorderSizePixel = 0
TopMask.ZIndex = 2

local Accent = Instance.new("Frame", Top)
Accent.Size = UDim2.new(1, -36, 0, 3)
Accent.Position = UDim2.fromOffset(18, 72)
Accent.BackgroundColor3 = COLORS.RED
Accent.BorderSizePixel = 0
Accent.ZIndex = 4
corner(Accent, 3)

local Title = Instance.new("TextLabel", Top)
Title.BackgroundTransparency = 1
Title.Position = UDim2.fromOffset(20, 10)
Title.Size = UDim2.new(1, -90, 0, 30)
Title.Text = "RED METEOR HUB FPS"
Title.TextColor3 = COLORS.WHITE
Title.Font = Enum.Font.GothamBold
Title.TextSize = 21
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.ZIndex = 3

-- Efeito neon/animação no título
local TitleStroke = Instance.new("UIStroke")
TitleStroke.Color = COLORS.RED_BRIGHT
TitleStroke.Thickness = 0.8
TitleStroke.Transparency = 0.35
TitleStroke.Parent = Title

local TitleGradient = Instance.new("UIGradient")
TitleGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, COLORS.WHITE),
    ColorSequenceKeypoint.new(0.45, COLORS.RED_BRIGHT),
    ColorSequenceKeypoint.new(0.7, COLORS.WHITE),
    ColorSequenceKeypoint.new(1, COLORS.RED)
})
TitleGradient.Rotation = 0
TitleGradient.Parent = Title

-- Brilho passando pelo título
local titleGradientRunning = true
task.spawn(function()
    while titleGradientRunning and Title.Parent do
        for rotation = -25, 25, 2 do
            if not titleGradientRunning or not Title.Parent then break end
            TitleGradient.Rotation = rotation
            TitleStroke.Transparency = 0.2 + math.abs(rotation) / 100
            task.wait(0.035)
        end
        for rotation = 25, -25, -2 do
            if not titleGradientRunning or not Title.Parent then break end
            TitleGradient.Rotation = rotation
            TitleStroke.Transparency = 0.2 + math.abs(rotation) / 100
            task.wait(0.035)
        end
    end
end)

local Subtitle = Instance.new("TextLabel", Top)
Subtitle.BackgroundTransparency = 1
Subtitle.Position = UDim2.fromOffset(21, 40)
Subtitle.Size = UDim2.new(1, -100, 0, 20)
Subtitle.Text = "FPS CONTROL  •  RED / BLACK / WHITE"
Subtitle.TextColor3 = COLORS.MUTED
Subtitle.Font = Enum.Font.GothamMedium
Subtitle.TextSize = 10
Subtitle.TextXAlignment = Enum.TextXAlignment.Left
Subtitle.ZIndex = 3

local Close = Instance.new("TextButton", Top)
Close.Name = "Close"
Close.Size = UDim2.fromOffset(34, 34)
Close.Position = UDim2.new(1, -48, 0, 15)
Close.BackgroundColor3 = Color3.fromRGB(32, 12, 16)
Close.Text = "×"
Close.TextColor3 = COLORS.RED_BRIGHT
Close.Font = Enum.Font.GothamBold
Close.TextSize = 24
Close.AutoButtonColor = false
Close.ZIndex = 5
corner(Close, 10)

Close.MouseEnter:Connect(function()
    TweenService:Create(Close, TweenInfo.new(0.12), {BackgroundColor3 = COLORS.RED, TextColor3 = COLORS.WHITE}):Play()
end)
Close.MouseLeave:Connect(function()
    TweenService:Create(Close, TweenInfo.new(0.12), {BackgroundColor3 = Color3.fromRGB(32, 12, 16), TextColor3 = COLORS.RED_BRIGHT}):Play()
end)
Close.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
end)

-- Função de notificação é definida mais abaixo.
local showShiftNotification

-- SHIFT DIREITO: mostra/esconde o hub
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.RightShift then
        MainFrame.Visible = not MainFrame.Visible
        if showShiftNotification then
            if MainFrame.Visible then
                showShiftNotification("🔴 RED METEOR HUB FPS\nPainel aberto • RightShift", 2.2)
            else
                showShiftNotification("⚫ RED METEOR HUB FPS\nPainel ocultado • RightShift", 2.2)
            end
        end
    end
end)

-- Arrastar a janela
local dragging = false
local dragStart, startPos
Top.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

local ContentFrame = Instance.new("ScrollingFrame", MainFrame)
ContentFrame.Name = "Content"
ContentFrame.Position = UDim2.fromOffset(14, 88)
ContentFrame.Size = UDim2.new(1, -28, 1, -102)
ContentFrame.BackgroundTransparency = 1
ContentFrame.BorderSizePixel = 0
ContentFrame.ScrollBarThickness = 3
ContentFrame.ScrollBarImageColor3 = COLORS.RED
ContentFrame.CanvasSize = UDim2.fromOffset(0, 0)
ContentFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
ContentFrame.ZIndex = 2

local ContentPadding = Instance.new("UIPadding", ContentFrame)
ContentPadding.PaddingTop = UDim.new(0, 3)
ContentPadding.PaddingBottom = UDim.new(0, 12)
ContentPadding.PaddingLeft = UDim.new(0, 3)
ContentPadding.PaddingRight = UDim.new(0, 7)

local UIListLayout = Instance.new("UIListLayout", ContentFrame)
UIListLayout.Padding = UDim.new(0, 8)
UIListLayout.FillDirection = Enum.FillDirection.Vertical
UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder

local function createSection(name)
    local section = Instance.new("Frame", ContentFrame)
    section.Size = UDim2.new(1, 0, 0, 28)
    section.BackgroundTransparency = 1

    local line = Instance.new("Frame", section)
    line.Size = UDim2.fromOffset(3, 15)
    line.Position = UDim2.fromOffset(2, 6)
    line.BackgroundColor3 = COLORS.RED
    line.BorderSizePixel = 0
    corner(line, 2)

    local label = Instance.new("TextLabel", section)
    label.BackgroundTransparency = 1
    label.Position = UDim2.fromOffset(13, 0)
    label.Size = UDim2.new(1, -13, 1, 0)
    label.Text = string.upper(name)
    label.TextColor3 = COLORS.WHITE
    label.Font = Enum.Font.GothamBold
    label.TextSize = 11
    label.TextXAlignment = Enum.TextXAlignment.Left

    return section
end

local function createToggleButton(name, getState, onToggle)
    local button = Instance.new("TextButton", ContentFrame)
    button.Name = name:gsub("%s+", "_")
    button.Size = UDim2.new(1, 0, 0, 48)
    button.BackgroundColor3 = COLORS.CARD
    button.BorderSizePixel = 0
    button.Text = ""
    button.AutoButtonColor = false
    corner(button, 10)
    local outline = stroke(button, Color3.fromRGB(42, 42, 50), 0.15, 1)

    local indicator = Instance.new("Frame", button)
    indicator.Size = UDim2.fromOffset(5, 28)
    indicator.Position = UDim2.fromOffset(0, 10)
    indicator.BorderSizePixel = 0
    corner(indicator, 3)

    local label = Instance.new("TextLabel", button)
    label.BackgroundTransparency = 1
    label.Position = UDim2.fromOffset(17, 0)
    label.Size = UDim2.new(1, -95, 1, 0)
    label.Text = name
    label.TextColor3 = COLORS.WHITE
    label.Font = Enum.Font.GothamMedium
    label.TextSize = 13
    label.TextXAlignment = Enum.TextXAlignment.Left

    local state = Instance.new("TextLabel", button)
    state.BackgroundTransparency = 1
    state.Position = UDim2.new(1, -73, 0, 0)
    state.Size = UDim2.fromOffset(60, 48)
    state.Font = Enum.Font.GothamBold
    state.TextSize = 10
    state.TextXAlignment = Enum.TextXAlignment.Right

    local function updateColor()
        local enabled = getState()
        indicator.BackgroundColor3 = enabled and COLORS.RED_BRIGHT or COLORS.OFF
        state.Text = enabled and "ON" or "OFF"
        state.TextColor3 = enabled and COLORS.RED_BRIGHT or COLORS.MUTED
        outline.Color = enabled and COLORS.RED_DARK or Color3.fromRGB(42, 42, 50)
    end

    updateColor()

    button.MouseButton1Click:Connect(function()
        onToggle()
        updateColor()
    end)

    button.MouseEnter:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.12), {BackgroundColor3 = COLORS.CARD_HOVER}):Play()
    end)
    button.MouseLeave:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.12), {BackgroundColor3 = COLORS.CARD}):Play()
    end)

    return button
end

local function createSlider(name, min, max, default, onChanged)
    local container = Instance.new("Frame", ContentFrame)
    container.Size = UDim2.new(1, 0, 0, 62)
    container.BackgroundColor3 = COLORS.CARD
    container.BorderSizePixel = 0
    corner(container, 10)
    stroke(container, Color3.fromRGB(42, 42, 50), 0.15, 1)

    local label = Instance.new("TextLabel", container)
    label.BackgroundTransparency = 1
    label.Position = UDim2.fromOffset(15, 7)
    label.Size = UDim2.new(1, -85, 0, 18)
    label.Text = name
    label.TextColor3 = COLORS.WHITE
    label.Font = Enum.Font.GothamMedium
    label.TextSize = 12
    label.TextXAlignment = Enum.TextXAlignment.Left

    local valueLabel = Instance.new("TextLabel", container)
    valueLabel.BackgroundTransparency = 1
    valueLabel.Position = UDim2.new(1, -70, 0, 7)
    valueLabel.Size = UDim2.fromOffset(55, 18)
    valueLabel.TextColor3 = COLORS.RED_BRIGHT
    valueLabel.Font = Enum.Font.GothamBold
    valueLabel.TextSize = 11
    valueLabel.TextXAlignment = Enum.TextXAlignment.Right

    local track = Instance.new("Frame", container)
    track.Position = UDim2.fromOffset(15, 39)
    track.Size = UDim2.new(1, -30, 0, 5)
    track.BackgroundColor3 = Color3.fromRGB(42, 42, 48)
    track.BorderSizePixel = 0
    corner(track, 3)

    local fill = Instance.new("Frame", track)
    fill.BackgroundColor3 = COLORS.RED
    fill.BorderSizePixel = 0
    corner(fill, 3)

    local hit = Instance.new("TextButton", track)
    hit.Size = UDim2.new(1, 16, 1, 24)
    hit.Position = UDim2.fromOffset(-8, -12)
    hit.BackgroundTransparency = 1
    hit.Text = ""
    hit.AutoButtonColor = false

    local currentValue = default
    local draggingSlider = false

    local function updateSlider(inputX)
        local percent = math.clamp((inputX - track.AbsolutePosition.X) / track.AbsoluteSize.X, 0, 1)
        currentValue = min + (max - min) * percent
        if max - min <= 20 then
            currentValue = math.floor(currentValue * 10 + 0.5) / 10
        else
            currentValue = math.floor(currentValue + 0.5)
        end
        fill.Size = UDim2.new(percent, 0, 1, 0)
        valueLabel.Text = tostring(currentValue)
        onChanged(currentValue)
    end

    local initialPercent = (default - min) / (max - min)
    fill.Size = UDim2.new(math.clamp(initialPercent, 0, 1), 0, 1, 0)
    valueLabel.Text = tostring(default)

    hit.MouseButton1Down:Connect(function()
        draggingSlider = true
        updateSlider(UserInputService:GetMouseLocation().X)
    end)
    UserInputService.InputChanged:Connect(function(input)
        if draggingSlider and input.UserInputType == Enum.UserInputType.MouseMovement then
            updateSlider(input.Position.X)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            draggingSlider = false
        end
    end)

    return container, function() return currentValue end
end

local function createSelector(name, options, default, onChanged)
    local container = Instance.new("Frame", ContentFrame)
    container.Size = UDim2.new(1, 0, 0, 48)
    container.BackgroundColor3 = COLORS.CARD
    container.BorderSizePixel = 0
    corner(container, 10)
    stroke(container, Color3.fromRGB(42, 42, 50), 0.15, 1)

    local label = Instance.new("TextLabel", container)
    label.BackgroundTransparency = 1
    label.Position = UDim2.fromOffset(15, 0)
    label.Size = UDim2.new(1, -125, 1, 0)
    label.Text = name
    label.TextColor3 = COLORS.WHITE
    label.Font = Enum.Font.GothamMedium
    label.TextSize = 13
    label.TextXAlignment = Enum.TextXAlignment.Left

    local current = default
    local value = Instance.new("TextLabel", container)
    value.BackgroundTransparency = 1
    value.Position = UDim2.new(1, -105, 0, 0)
    value.Size = UDim2.fromOffset(70, 48)
    value.Text = tostring(options[current])
    value.TextColor3 = COLORS.RED_BRIGHT
    value.Font = Enum.Font.GothamBold
    value.TextSize = 11
    value.TextXAlignment = Enum.TextXAlignment.Right

    local nextButton = Instance.new("TextButton", container)
    nextButton.BackgroundColor3 = Color3.fromRGB(38, 14, 18)
    nextButton.Position = UDim2.new(1, -43, 0, 8)
    nextButton.Size = UDim2.fromOffset(32, 32)
    nextButton.Text = "›"
    nextButton.TextColor3 = COLORS.RED_BRIGHT
    nextButton.Font = Enum.Font.GothamBold
    nextButton.TextSize = 20
    nextButton.AutoButtonColor = false
    corner(nextButton, 9)

    nextButton.MouseButton1Click:Connect(function()
        current = current % #options + 1
        value.Text = tostring(options[current])
        onChanged(options[current])
    end)

    nextButton.MouseEnter:Connect(function()
        TweenService:Create(nextButton, TweenInfo.new(0.12), {BackgroundColor3 = COLORS.RED, TextColor3 = COLORS.WHITE}):Play()
    end)
    nextButton.MouseLeave:Connect(function()
        TweenService:Create(nextButton, TweenInfo.new(0.12), {BackgroundColor3 = Color3.fromRGB(38, 14, 18), TextColor3 = COLORS.RED_BRIGHT}):Play()
    end)

    return container
end

-- Seções da UI
createSection("Visual")
createToggleButton("ESP", function() return espEnabled end, function() espEnabled = not espEnabled end)
createToggleButton("ESP Glow", function() return glowEnabled end, function() glowEnabled = not glowEnabled end)
createToggleButton("Verificar Time", function() return teamCheck end, function() teamCheck = not teamCheck end)
createToggleButton("Verificar Parede", function() return wallCheck end, function() wallCheck = not wallCheck end)

createSection("Aimbot")
createToggleButton("Aimbot", function() return aimbotEnabled end, function() aimbotEnabled = not aimbotEnabled end)
createToggleButton("Exibir FOV", function() return showFov end, function() showFov = not showFov end)
createToggleButton("FOV RGB", function() return fovRainbow end, function() fovRainbow = not fovRainbow end)

createSlider("FOV", 50, 500, fovRadius, function(value)
    fovRadius = value
end)

createSlider("Suavização Aimbot", 0, 10, smoothingFactor * 10, function(value)
    smoothingFactor = value / 10
end)

createSelector("Parte Alvo", {"Head", "HumanoidRootPart", "Torso"}, 1, function(value)
    targetPart = value
end)

-- Efeito Glow

local function updateGlowForPlayer(player)

    if not glowEnabled or not espEnabled then

        if glowObjects[player] then

            for _, highlighter in pairs(glowObjects[player]) do

                highlighter:Destroy()

            end

            glowObjects[player] = nil

        end

        return

    end

    if player == LocalPlayer then return end

    if teamCheck and player.Team == LocalPlayer.Team then return end

    local character = player.Character

    if not character then return end

    if not glowObjects[player] then

        glowObjects[player] = {}

    end

    -- Limpa highlighters antigos*

    for _, highlighter in pairs(glowObjects[player]) do

        highlighter:Destroy()

    end

    glowObjects[player] = {}

    -- Adiciona novos highlighters*

    for _, part in pairs(character:GetChildren()) do

        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then

            -- Determina a cor do glow baseado no time*

            local glowColor = player.Team == LocalPlayer.Team and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)

            -- Cria o highlighter*

            local highlighter = Instance.new("Highlight")

            highlighter.Name = "ESPHighlight"

            highlighter.FillColor = glowColor

            highlighter.OutlineColor = glowColor

            highlighter.FillTransparency = 0.7

            highlighter.OutlineTransparency = 0

            highlighter.Adornee = part

            highlighter.Parent = game.CoreGui

            table.insert(glowObjects[player], highlighter)

        end

    end

end

-- FOV Círculo*

local fovCircle = Drawing.new("Circle")

fovCircle.Visible = showFov

fovCircle.Thickness = 2

fovCircle.Radius = fovRadius

fovCircle.Transparency = 1

fovCircle.Color = Color3.new(1, 1, 1)

fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

-- ESP*

local function createESP(player)

    if player == LocalPlayer or espObjects[player] then return end

    -- Box do ESP*

    local box = Drawing.new("Square")

    box.Visible = false

    box.Color = Color3.new(1, 1, 1)

    box.Thickness = 1

    box.Transparency = 1

    box.Filled = false

    -- Nome do jogador*

    local nameTag = Drawing.new("Text")

    nameTag.Visible = false

    nameTag.Center = true

    nameTag.Outline = true

    nameTag.Font = 2

    nameTag.Size = 14

    nameTag.Color = Color3.new(1, 1, 1)

    -- Distância*

    local distanceText = Drawing.new("Text")

    distanceText.Visible = false

    distanceText.Center = true

    distanceText.Outline = true

    distanceText.Font = 2

    distanceText.Size = 12

    distanceText.Color = Color3.new(1, 1, 1)

    espObjects[player] = {

        box = box,

        nameTag = nameTag,

        distanceText = distanceText

    }

end

local function removeESP(player)

    if espObjects[player] then

        for _, drawing in pairs(espObjects[player]) do

            drawing:Remove()

        end

        espObjects[player] = nil

    end

    if glowObjects[player] then

        for _, highlighter in pairs(glowObjects[player]) do

            highlighter:Destroy()

        end

        glowObjects[player] = nil

    end

end

-- Setup inicial*

for _, player in ipairs(Players:GetPlayers()) do

    createESP(player)

end

Players.PlayerAdded:Connect(createESP)

Players.PlayerRemoving:Connect(removeESP)

-- Verificação de parede*

local function isVisible(part)

    if not wallCheck then return true end

    local origin = Camera.CFrame.Position

    local direction = (part.Position - origin)

    local ray = Ray.new(origin, direction.Unit * math.min(direction.Magnitude, 1000))

    local hitPart, hitPosition = Workspace:FindPartOnRayWithIgnoreList(ray, {LocalPlayer.Character}, false, true)

    if hitPart then

        return hitPart:IsDescendantOf(part.Parent)

    end

    return false

end

-- Aimbot alvo*

local function getClosestPart()

    local closestPart = nil

    local shortestDist = fovRadius

    for _, player in ipairs(Players:GetPlayers()) do

        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(targetPart) then

            if teamCheck and player.Team == LocalPlayer.Team then continue end

            local part = player.Character[targetPart]

            local pos, onScreen = Camera:WorldToViewportPoint(part.Position)

            local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude

            if onScreen and dist < shortestDist and isVisible(part) then

                shortestDist = dist

                closestPart = part

            end

        end

    end

    return closestPart

end

-- Suavização do Aimbot*

local function smoothLookAt(target)

    local startCFrame = Camera.CFrame

    local endCFrame = CFrame.new(Camera.CFrame.Position, target.Position)

    local newCFrame = startCFrame:Lerp(endCFrame, 1 - smoothingFactor)

    Camera.CFrame = newCFrame

end

-- Loop principal*

RunService.RenderStepped:Connect(function()

    -- Atualiza FOV*

    fovCircle.Visible = showFov and aimbotEnabled

    fovCircle.Radius = fovRadius

    fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    -- Efeito de arco-íris para o FOV*

    if fovRainbow then

        hue = (hue + 0.005) % 1

        fovCircle.Color = Color3.fromHSV(hue, 1, 1)

    else

        fovCircle.Color = COLORS.RED

    end

    -- Atualiza ESP e Glow*

    for _, player in ipairs(Players:GetPlayers()) do

        if player ~= LocalPlayer then

            -- Update ESP*

            local esp = espObjects[player]

            if not esp then

                createESP(player)

                esp = espObjects[player]

            end

            local char = player.Character

            if char and char:FindFirstChild("HumanoidRootPart") and espEnabled then

                if teamCheck and player.Team == LocalPlayer.Team then

                    esp.nameTag.Visible = false

                    esp.box.Visible = false

                    esp.distanceText.Visible = false

                else

                    local humanoid = char:FindFirstChildOfClass("Humanoid")

                    local hrp = char.HumanoidRootPart

                    local head = char:FindFirstChild("Head")

                    if hrp and head and humanoid then

                        -- Cálculo da caixa do ESP*

                        local rootPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

                        if onScreen and isVisible(hrp) then

                            -- Cálculo de distância*

                            local distance = (Camera.CFrame.Position - hrp.Position).Magnitude

                            local displayDistance = math.floor(distance)

                            -- Atualizando nome*

                            esp.nameTag.Position = Vector2.new(rootPos.X, rootPos.Y - 40)

                            esp.nameTag.Text = player.Name

                            esp.nameTag.Visible = true

                            -- Atualizando distância*

                            esp.distanceText.Position = Vector2.new(rootPos.X, rootPos.Y + 20)

                            esp.distanceText.Text = displayDistance .. "m"

                            esp.distanceText.Visible = true

                            -- Determina cor baseada na saúde*

                            local healthPercent = humanoid.Health / humanoid.MaxHealth

                            local healthColor = Color3.fromRGB(

                                255 * (1 - healthPercent),

                                255 * healthPercent,

                                0

                            )

                            esp.nameTag.Color = healthColor

                            esp.box.Color = healthColor

                            -- Atualizando box*

                            local size = Vector3.new(4, 6, 0) * (1 / (rootPos.Z * 0.001 + 1))

                            esp.box.Size = Vector2.new(size.X, size.Y)

                            esp.box.Position = Vector2.new(rootPos.X - size.X/2, rootPos.Y - size.Y/2)

                            esp.box.Visible = true

                        else

                            esp.nameTag.Visible = false

                            esp.box.Visible = false

                            esp.distanceText.Visible = false

                        end

                    else

                        esp.nameTag.Visible = false

                        esp.box.Visible = false

                        esp.distanceText.Visible = false

                    end

                end

            else

                esp.nameTag.Visible = false

                esp.box.Visible = false

                esp.distanceText.Visible = false

            end

            -- Update Glow*

            updateGlowForPlayer(player)

        end

    end

    -- Aimbot*

    if aimbotEnabled and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then

        local target = getClosestPart()

        if target then

            smoothLookAt(target)

        end

    end

end)

-- Notificação de inicialização*

local function createNotification(text, duration)

    local notif = Instance.new("Frame", ScreenGui)

    notif.Size = UDim2.new(0, 250, 0, 60)

    notif.Position = UDim2.new(1, -260, 0, 20)

    notif.BackgroundColor3 = COLORS.PANEL

    notif.BorderSizePixel = 0

    notif.BackgroundTransparency = 0.1

    local UICorner = Instance.new("UICorner", notif)

    UICorner.CornerRadius = UDim.new(0, 8)

    local notifBar = Instance.new("Frame", notif)

    notifBar.Size = UDim2.new(0.02, 0, 1, 0)

    notifBar.BackgroundColor3 = COLORS.RED

    notifBar.BorderSizePixel = 0

    local barCorner = Instance.new("UICorner", notifBar)

    barCorner.CornerRadius = UDim.new(0, 8)

    local textLabel = Instance.new("TextLabel", notif)

    textLabel.Size = UDim2.new(0.98, -10, 1, 0)

    textLabel.Position = UDim2.new(0.02, 5, 0, 0)

    textLabel.BackgroundTransparency = 1

    textLabel.Text = text

    textLabel.TextColor3 = COLORS.WHITE

    textLabel.Font = Enum.Font.GothamSemibold

    textLabel.TextSize = 14

    textLabel.TextWrapped = true

    textLabel.TextXAlignment = Enum.TextXAlignment.Left

    -- Animação*

    notif.Position = UDim2.new(1, 50, 0, 20)

    local tweenInfo = TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)

    local tween = TweenService:Create(notif, tweenInfo, {Position = UDim2.new(1, -260, 0, 20)})

    tween:Play()

    task.delay(duration, function()

        local fadeOut = TweenService:Create(notif, TweenInfo.new(0.5), {Position = UDim2.new(1, 50, 0, 20)})

        fadeOut:Play()

        fadeOut.Completed:Connect(function()

            notif:Destroy()

        end)

    end)

end

showShiftNotification = createNotification
createNotification("🔴 RED METEOR HUB FPS • Carregado com sucesso!", 3)
task.delay(3.4, function()
    createNotification("⌨️ RightShift • Abrir / Fechar o Hub", 3)
end)

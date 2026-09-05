--[[
    FRUIT BATTLEGROUNDS - PLÍNIO HUB
    FOV estilo Blox Fruits | Painel arrastável + Teleporte
    Discord: https://discord.gg/VNjfq35gPQ
    ⚠️ Todas as funções DESATIVADAS por padrão
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- ========== CONFIGURAÇÕES (TUDO DESATIVADO POR PADRÃO) ==========
local Settings = {
    AutoAim = false,
    AutoHeal = false,
    ESP = false,
    FOVRadius = 150,
    ShowFOV = false,
    ShowLine = false,
    TeleportLock = false,
    TargetPlayerName = ""
}

_G.PlinioHub = Settings

-- ========== VARIAVEIS ==========
local CurrentTarget = nil
local ESPObjects = {}
local FOVCircle = nil
local AimLine = nil
local isRunning = true
local GuiMain = nil
local isMinimized = false
local Dragging = false
local DragStart = nil
local StartPos = nil
local TeleportTarget = nil

-- ========== FUNÇÃO: Ver se o alvo está visível ==========
local function CanSeeTarget(targetPart)
    if not targetPart then return false end
    local origin = Camera.CFrame.Position
    local direction = (targetPart.Position - origin).Unit
    local distance = (targetPart.Position - origin).Magnitude
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Blacklist
    params.FilterDescendantsInstances = {LocalPlayer.Character}
    params.IgnoreWater = true
    local result = workspace:Raycast(origin, direction * distance, params)
    if result then
        local hit = result.Instance
        local targetParent = targetPart.Parent
        if targetParent and hit:IsDescendantOf(targetParent) then return true end
        return false
    end
    return true
end

-- ========== PEGAR ALVOS NO FOV ==========
local function GetTargetsInFOV()
    local targets = {}
    local center = Camera.ViewportSize / 2
    local radius = Settings.FOVRadius
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") then
            local humanoid = player.Character.Humanoid
            if humanoid.Health > 0 then
                local part = player.Character:FindFirstChild("Head") or player.Character:FindFirstChild("HumanoidRootPart")
                if part then
                    local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
                    if onScreen then
                        local dist = (Vector2.new(pos.X, pos.Y) - center).Magnitude
                        if dist <= radius and CanSeeTarget(part) then
                            table.insert(targets, {Player=player, Part=part, Distance=dist, Position=part.Position})
                        end
                    end
                end
            end
        end
    end
    table.sort(targets, function(a,b) return a.Distance < b.Distance end)
    return targets
end

-- ========== MIRA AUTOMÁTICA ==========
local function UpdateAim()
    if not Settings.AutoAim then CurrentTarget=nil return end
    local targets = GetTargetsInFOV()
    if #targets > 0 then
        local target = targets[1]
        CurrentTarget = target.Player
        local current = Camera.CFrame
        local targetCF = CFrame.lookAt(Camera.CFrame.Position, target.Position)
        Camera.CFrame = current:Lerp(targetCF, 0.25)
    else
        CurrentTarget = nil
    end
end

-- ========== AUTO HEAL ==========
local function AutoHeal()
    if not Settings.AutoHeal then return end
    local char = LocalPlayer.Character
    if not char then return end
    local humanoid = char:FindFirstChild("Humanoid")
    if not humanoid then return end
    if humanoid.Health < 50 then
        pcall(function()
            local inv = LocalPlayer:FindFirstChild("Inventory")
            if inv then
                for _, item in pairs(inv:GetChildren()) do
                    if item.Name:lower():find("heal") or item.Name:lower():find("cura") or item.Name:lower():find("potion") then
                        local remote = game:GetService("ReplicatedStorage"):FindFirstChild("Remotes")
                        if remote then
                            local useItem = remote:FindFirstChild("UseItem")
                            if useItem then useItem:FireServer(item) break end
                        end
                    end
                end
            end
        end)
    end
end

-- ========== ESP ==========
local function UpdateESP()
    for _, obj in pairs(ESPObjects) do pcall(function() obj:Destroy() end) end
    ESPObjects = {}
    if not Settings.ESP then return end
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local highlight = Instance.new("Highlight")
            highlight.FillColor = Color3.fromRGB(255,0,0)
            highlight.FillTransparency = 0.4
            highlight.OutlineColor = Color3.fromRGB(255,255,255)
            highlight.OutlineTransparency = 0
            highlight.Adornee = player.Character
            highlight.Parent = player.Character
            table.insert(ESPObjects, highlight)
        end
    end
end

-- ========== TELEPORTE + TRAVAR ==========
local function UpdateTeleportLock()
    if not Settings.TeleportLock or Settings.TargetPlayerName == "" then
        TeleportTarget = nil
        return
    end
    local targetPlayer = nil
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and string.find(string.lower(player.Name), string.lower(Settings.TargetPlayerName), 1, true) then
            targetPlayer = player break
        end
    end
    if not targetPlayer or not targetPlayer.Character then
        TeleportTarget = nil
        return
    end
    TeleportTarget = targetPlayer
    local localChar = LocalPlayer.Character
    local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
    local localRoot = localChar and localChar:FindFirstChild("HumanoidRootPart")
    if localRoot and targetRoot then
        pcall(function() localChar.HumanoidRootPart.CFrame = targetRoot.CFrame * CFrame.new(0,3,0) end)
    end
end

-- ========== CÍRCULO FOV ==========
local function CreateFOVCircle()
    if FOVCircle then pcall(function() FOVCircle:Destroy() end) FOVCircle=nil end
    if AimLine then pcall(function() AimLine:Destroy() end) AimLine=nil end
    FOVCircle = Drawing.new("Circle")
    FOVCircle.Visible = false
    FOVCircle.Color = Color3.fromRGB(255,0,0)
    FOVCircle.Radius = Settings.FOVRadius
    FOVCircle.Thickness = 2
    FOVCircle.Filled = false
    FOVCircle.Transparency = 1
    AimLine = Drawing.new("Line")
    AimLine.Thickness = 2
    AimLine.Color = Color3.fromRGB(255,0,0)
    AimLine.Transparency = 1
    AimLine.Visible = false
end

-- ========== ATUALIZAR FOV ==========
RunService.RenderStepped:Connect(function()
    local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    if Settings.ShowFOV and Settings.AutoAim and FOVCircle then
        FOVCircle.Visible = true
        FOVCircle.Position = center
        FOVCircle.Radius = Settings.FOVRadius
    elseif FOVCircle then
        FOVCircle.Visible = false
    end
    if Settings.ShowLine and Settings.AutoAim and CurrentTarget and CurrentTarget.Character and AimLine then
        local part = CurrentTarget.Character:FindFirstChild("Head") or CurrentTarget.Character:FindFirstChild("HumanoidRootPart")
        if part then
            local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
            if onScreen then
                AimLine.Visible = true
                AimLine.From = center
                AimLine.To = Vector2.new(pos.X, pos.Y)
            else AimLine.Visible = false end
        end
    elseif AimLine then AimLine.Visible = false end
end)

-- ========== LOOP PRINCIPAL ==========
local function MainLoop()
    if not isRunning then return end
    pcall(function()
        UpdateAim()
        AutoHeal()
        UpdateESP()
        UpdateTeleportLock()
    end)
end
RunService.RenderStepped:Connect(MainLoop)
-- ========== GUI — BOTÕES IGUAIS DA IMAGEM: □ Amarelo | ✕ Vermelho ==========
local function CreateGUI()
    if GuiMain then pcall(function() GuiMain:Destroy() end) GuiMain=nil end
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "PlinioHub"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    GuiMain = screenGui

    -- FRAME PRINCIPAL
    local main = Instance.new("Frame")
    main.Size = UDim2.new(0, 180, 0, 310)
    main.Position = UDim2.new(0.02,0,0.2,0)
    main.BackgroundColor3 = Color3.fromRGB(22, 22, 33)
    main.BackgroundTransparency = 0.1
    main.BorderSizePixel = 0
    main.ClipsDescendants = true
    main.Parent = screenGui
    main.Active = true
    main.Draggable = false

    Instance.new("UICorner", main).CornerRadius = UDim.new(0,6)
    local stroke = Instance.new("UIStroke", main)
    stroke.Color = Color3.fromRGB(255,0,0)
    stroke.Thickness = 1.5
    stroke.Transparency = 0.4

    -- 📌 CABEÇALHO — Fundo vermelho igual da imagem
    local header = Instance.new("Frame")
    header.Size = UDim2.new(1,0,0,32)
    header.BackgroundColor3 = Color3.fromRGB(200, 25, 25)
    header.BorderSizePixel = 0
    header.Parent = main
    header.Active = true
    Instance.new("UICorner", header).CornerRadius = UDim.new(0,6)

    -- ✅ Ícone de espadas antes do nome
    local icon = Instance.new("TextLabel")
    icon.Size = UDim2.new(0,16,1,0)
    icon.Position = UDim2.new(0,6,0,0)
    icon.BackgroundTransparency = 1
    icon.Text = "⚔️"
    icon.Font = Enum.Font.GothamBold
    icon.TextColor3 = Color3.fromRGB(255,255,255)
    icon.TextSize = 12
    icon.Parent = header

    -- ✅ NOME DO PAINEL — Branco no cabeçalho vermelho
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(0.45,0,1,0)
    title.Position = UDim2.new(0,26,0,0)
    title.BackgroundTransparency = 1
    title.Text = "Plínio Hub"
    title.Font = Enum.Font.GothamBold
    title.TextColor3 = Color3.fromRGB(255,255,255)
    title.TextSize = 13
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = header

    -- 🟨 BOTÃO MINIMIZAR — Amarelo com □ (sem quadrado, só o símbolo)
    local minBtn = Instance.new("TextButton")
    minBtn.Size = UDim2.new(0,26,0,22)
    minBtn.Position = UDim2.new(0.72,0,0.5,-11)
    minBtn.BackgroundColor3 = Color3.fromRGB(255, 200, 0)
    minBtn.BackgroundTransparency = 0
    minBtn.Text = "□"
    minBtn.TextColor3 = Color3.fromRGB(0,0,0)
    minBtn.TextSize = 14
    minBtn.Font = Enum.Font.GothamBold
    minBtn.BorderSizePixel = 0
    minBtn.Parent = header
    Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0,4)

    -- 🟥 BOTÃO FECHAR — Vermelho com ✕ (sem quadrado, só o X)
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0,26,0,22)
    closeBtn.Position = UDim2.new(0.88,0,0.5,-11)
    closeBtn.BackgroundColor3 = Color3.fromRGB(220, 30, 30)
    closeBtn.BackgroundTransparency = 0
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(255,255,255)
    closeBtn.TextSize = 13
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.BorderSizePixel = 0
    closeBtn.Parent = header
    Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0,4)

    -- Efeitos hover
    minBtn.MouseEnter:Connect(function() minBtn.BackgroundTransparency = 0.15 end)
    minBtn.MouseLeave:Connect(function() minBtn.BackgroundTransparency = 0 end)
    closeBtn.MouseEnter:Connect(function() closeBtn.BackgroundTransparency = 0.15 end)
    closeBtn.MouseLeave:Connect(function() closeBtn.BackgroundTransparency = 0 end)

    -- CONTEÚDO
    local content = Instance.new("Frame")
    content.Size = UDim2.new(1,-10,1,-42)
    content.Position = UDim2.new(0,5,0,35)
    content.BackgroundTransparency = 1
    content.Parent = main

    -- TOGGLE
    local function CreateToggle(text, yPos, default, callback)
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1,0,0,22)
        frame.Position = UDim2.new(0,0,0,yPos)
        frame.BackgroundTransparency = 1
        frame.Parent = content
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(0.6,0,1,0)
        label.BackgroundTransparency = 1
        label.Text = text
        label.Font = Enum.Font.Gotham
        label.TextColor3 = Color3.fromRGB(210,210,210)
        label.TextSize = 10
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.Parent = frame
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0,35,0,18)
        btn.Position = UDim2.new(0.7,0,0.5,-9)
        btn.BackgroundColor3 = default and Color3.fromRGB(0,190,70) or Color3.fromRGB(70,70,85)
        btn.Text = default and "ON" or "OFF"
        btn.TextColor3 = Color3.fromRGB(255,255,255)
        btn.TextSize = 9
        btn.Font = Enum.Font.GothamBold
        btn.BorderSizePixel = 0
        btn.Parent = frame
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0,4)
        local state = default
        btn.MouseButton1Click:Connect(function()
            state = not state
            btn.BackgroundColor3 = state and Color3.fromRGB(0,190,70) or Color3.fromRGB(70,70,85)
            btn.Text = state and "ON" or "OFF"
            callback(state)
        end)
    end

    -- CAIXA DE TEXTO TELEPORTE
    local function CreateTextBox(yPos, placeholder, callback)
        local box = Instance.new("TextBox")
        box.Size = UDim2.new(1,0,0,24)
        box.Position = UDim2.new(0,0,0,yPos)
        box.BackgroundColor3 = Color3.fromRGB(35,35,50)
        box.Text = ""
        box.PlaceholderText = placeholder
        box.TextColor3 = Color3.fromRGB(230,230,230)
        box.Font = Enum.Font.Gotham
        box.TextSize = 10
        box.ClearTextOnFocus = false
        box.Parent = content
        Instance.new("UICorner", box).CornerRadius = UDim.new(0,4)
        box.FocusLost:Connect(function() callback(box.Text) end)
    end

    local y = 2
    CreateToggle("🎯 Auto Aim", y, Settings.AutoAim, function(v) Settings.AutoAim=v if not v then CurrentTarget=nil end end)
    y += 25
    CreateToggle("💚 Auto Heal", y, Settings.AutoHeal, function(v) Settings.AutoHeal=v end)
    y += 25
    CreateToggle("👁️ ESP", y, Settings.ESP, function(v) Settings.ESP=v if not v then for _,o in pairs(ESPObjects) do pcall(function() o:Destroy() end) end ESPObjects={} end end)
    y += 25
    CreateToggle("🔴 Mostrar FOV", y, Settings.ShowFOV, function(v) Settings.ShowFOV=v if not v and FOVCircle then FOVCircle.Visible=false end end)
    y += 25

    -- TELEPORTE
    local teleLabel = Instance.new("TextLabel")
    teleLabel.Size = UDim2.new(1,0,0,16)
    teleLabel.Position = UDim2.new(0,0,0,y)
    teleLabel.BackgroundTransparency = 1
    teleLabel.Text = "🚀 Teleportar e Travar"
    teleLabel.Font = Enum.Font.GothamBold
    teleLabel.TextColor3 = Color3.fromRGB(255,200,0)
    teleLabel.TextSize = 11
    teleLabel.TextXAlignment = Enum.TextXAlignment.Left
    teleLabel.Parent = content
    y += 18
    CreateTextBox(y, "Nome do jogador...", function(text) Settings.TargetPlayerName = text end)
    y += 28
    CreateToggle("🔒 Seguir/Travar", y, Settings.TeleportLock, function(v) Settings.TeleportLock=v if not v then TeleportTarget=nil end end)
    y += 30

    -- STATUS
    local statusFrame = Instance.new("Frame")
    statusFrame.Size = UDim2.new(1,0,0,20)
    statusFrame.Position = UDim2.new(0,0,0,y)
    statusFrame.BackgroundColor3 = Color3.fromRGB(70,70,85)
    statusFrame.BackgroundTransparency = 0
    statusFrame.Parent = content
    Instance.new("UICorner", statusFrame).CornerRadius = UDim.new(0,4)
    local statusText = Instance.new("TextLabel")
    statusText.Size = UDim2.new(1,0,1,0)
    statusText.BackgroundTransparency = 1
    statusText.Text = "✅ INATIVO"
    statusText.Font = Enum.Font.GothamBold
    statusText.TextColor3 = Color3.fromRGB(255,255,255)
    statusText.TextSize = 10
    statusText.Parent = statusFrame

    RunService.Heartbeat:Connect(function()
        pcall(function()
            if TeleportTarget then
                statusFrame.BackgroundColor3 = Color3.fromRGB(255,180,0)
                statusText.Text = "🚀 TRAVADO: "..TeleportTarget.Name
            elseif CurrentTarget then
                statusFrame.BackgroundColor3 = Color3.fromRGB(220,30,30)
                statusText.Text = "⚔️ EM COMBATE"
            else
                statusFrame.BackgroundColor3 = Color3.fromRGB(70,70,85)
                statusText.Text = "✅ INATIVO"
            end
        end)
    end)

    -- 🎯 SISTEMA DE ARRASTE — Cabeçalho inteiro arrasta
    local function StartDrag(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            Dragging = true
            DragStart = input.Position
            StartPos = main.Position
        end
    end
    local function UpdateDrag(input)
        if Dragging then
            local delta = input.Position - DragStart
            main.Position = UDim2.new(StartPos.X.Scale, StartPos.X.Offset+delta.X, StartPos.Y.Scale, StartPos.Y.Offset+delta.Y)
        end
    end
    local function StopDrag() Dragging = false end

    header.InputBegan:Connect(StartDrag)
    header.InputChanged:Connect(UpdateDrag)
    header.InputEnded:Connect(StopDrag)
    title.InputBegan:Connect(StartDrag)
    title.InputChanged:Connect(UpdateDrag)
    title.InputEnded:Connect(StopDrag)

    -- MINIMIZAR — Muda texto para □ / ─
    minBtn.MouseButton1Click:Connect(function()
        isMinimized = not isMinimized
        if isMinimized then
            main.Size = UDim2.new(0,180,0,32)
            content.Visible = false
            minBtn.Text = "─"
        else
            main.Size = UDim2.new(0,180,0,310)
            content.Visible = true
            minBtn.Text = "□"
        end
    end)

    -- FECHAR — ✕
    closeBtn.MouseButton1Click:Connect(function()
        isRunning = false
        screenGui:Destroy()
        if FOVCircle then pcall(function() FOVCircle:Destroy() end) end
        if AimLine then pcall(function() AimLine:Destroy() end) end
    end)
end

-- ========== INICIALIZAR ==========
print("⚔️ Plínio Hub — Carregado!")
print("✅ Cabeçalho vermelho + Nome branco")
print("✅ Botão Amarelo □ = Minimizar")
print("✅ Botão Vermelho ✕ = Fechar")
print("✅ Arraste pelo cabeçalho")

CreateFOVCircle()
task.wait(0.1)
CreateGUI()

_G.Plinio = {
    ToggleAim = function() Settings.AutoAim = not Settings.AutoAim end,
    ToggleHeal = function() Settings.AutoHeal = not Settings.AutoHeal end,
    ToggleESP = function() Settings.ESP = not Settings.ESP end,
    SetFOV = function(r) Settings.FOVRadius = r if FOVCircle then FOVCircle.Radius = r end end,
    SetTeleportTarget = function(n) Settings.TargetPlayerName = n end,
    ToggleTeleport = function() Settings.TeleportLock = not Settings.TeleportLock end
}

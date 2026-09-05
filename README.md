--[[
    ⚔️ PLÍNIO HUB — PARTE 1/3: ANIMAÇÃO
    ✅ Continua perfeita e sincronizada
]]

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local FIRST_SOUND = "rbxassetid://116866873116581"
local SECOND_SOUND = "rbxassetid://95724647330605"
local IMAGE_ID = "rbxassetid://113064380426857"

local CORES = {
	PRETO = Color3.fromRGB(0, 0, 0),
	VINHO_ESCURO = Color3.fromRGB(80, 10, 20),
	VINHO_MEDIO = Color3.fromRGB(130, 20, 35),
	VINHO_CLARO = Color3.fromRGB(180, 40, 60),
	VINHO_GLOW = Color3.fromRGB(200, 50, 75),
}

local CENTER = UDim2.fromScale(0.5, 0.5)

for _, name in ipairs({"RinneganEffectV7"}) do
	local old = playerGui:FindFirstChild(name)
	if old then old:Destroy() end
end

local gui = Instance.new("ScreenGui")
gui.Name = "RinneganEffectV7"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.DisplayOrder = 999999
gui.Parent = playerGui

local function tween(obj, dur, props, style, dir)
	local info = TweenInfo.new(dur, style or Enum.EasingStyle.Quad, dir or Enum.EasingDirection.Out)
	local t = TweenService:Create(obj, info, props)
	t:Play()
	return t
end

local function circle(obj)
	local c = Instance.new("UICorner")
	c.CornerRadius = UDim.new(1, 0)
	c.Parent = obj
end

local function makeFrame(name, zIndex)
	local f = Instance.new("Frame")
	f.Name = name
	f.AnchorPoint = Vector2.new(0.5, 0.5)
	f.Position = CENTER
	f.Size = UDim2.fromScale(1, 1)
	f.BackgroundTransparency = 1
	f.BorderSizePixel = 0
	f.ZIndex = zIndex
	f.Parent = gui
	return f
end

local darkness = makeFrame("Darkness", 1)
darkness.BackgroundColor3 = CORES.PRETO

local topBar = Instance.new("Frame")
topBar.Name = "TopBar"
topBar.Size = UDim2.new(1,0,0,0)
topBar.Position = UDim2.fromOffset(0,0)
topBar.BackgroundColor3 = CORES.PRETO
topBar.ZIndex = 55
topBar.Parent = gui

local bottomBar = Instance.new("Frame")
bottomBar.Name = "BottomBar"
bottomBar.AnchorPoint = Vector2.new(0,1)
bottomBar.Size = UDim2.new(1,0,0,0)
bottomBar.Position = UDim2.fromScale(0,1)
bottomBar.BackgroundColor3 = CORES.PRETO
bottomBar.ZIndex = 55
bottomBar.Parent = gui

local purpleFlash = makeFrame("RedFlash", 60)
purpleFlash.BackgroundColor3 = CORES.VINHO_MEDIO

local whiteFlash = makeFrame("WhiteFlash", 61)
whiteFlash.BackgroundColor3 = Color3.fromRGB(255,245,240)

local rinnegan = Instance.new("ImageLabel")
rinnegan.Name = "Rinnegan"
rinnegan.AnchorPoint = Vector2.new(0.5, 0.5)
rinnegan.Position = CENTER
rinnegan.Size = UDim2.fromOffset(1, 1)
rinnegan.BackgroundTransparency = 1
rinnegan.Image = IMAGE_ID
rinnegan.ImageTransparency = 1
rinnegan.ZIndex = 20
rinnegan.Parent = gui
circle(rinnegan)

local sound1 = Instance.new("Sound")
sound1.SoundId = FIRST_SOUND
sound1.Volume = 1
sound1.Parent = gui

local sound2 = Instance.new("Sound")
sound2.SoundId = SECOND_SOUND
sound2.Volume = 1
sound2.Parent = gui

local function shockwave(finalSize, duration, thickness)
	local ring = Instance.new("Frame")
	ring.AnchorPoint = Vector2.new(0.5,0.5)
	ring.Position = CENTER
	ring.Size = UDim2.fromOffset(5,5)
	ring.BackgroundTransparency = 1
	ring.Parent = gui
	circle(ring)
	local stroke = Instance.new("UIStroke")
	stroke.Thickness = thickness
	stroke.Color = CORES.VINHO_GLOW
	stroke.Parent = ring
	tween(ring, duration, {Size = UDim2.fromOffset(finalSize,finalSize)}, Enum.EasingStyle.Quart)
	tween(stroke, duration, {Transparency = 1, Thickness = 1}, Enum.EasingStyle.Quad)
	task.delay(duration + 0.1, function() ring:Destroy() end)
end

local function particleBurst(amount)
	for i = 1, amount do
		local p = Instance.new("Frame")
		p.AnchorPoint = Vector2.new(0.5,0.5)
		p.Position = CENTER
		p.Size = UDim2.fromOffset(math.random(2,6), math.random(2,6))
		p.BackgroundColor3 = Color3.fromRGB(math.random(120,200), math.random(15,50), math.random(20,60))
		p.Parent = gui
		circle(p)
		local ang = math.rad(math.random(0,359))
		local dist = math.random(150,400)
		tween(p, math.random(8,15)/10, {
			Position = UDim2.new(0.5, math.cos(ang)*dist, 0.5, math.sin(ang)*dist),
			Size = UDim2.fromOffset(0,0),
			BackgroundTransparency = 1
		}, Enum.EasingStyle.Quart)
	end
end

local function PlayIntro()
	rinnegan.Size = UDim2.fromOffset(1,1)
	rinnegan.ImageTransparency = 1
	darkness.BackgroundTransparency = 1
	purpleFlash.BackgroundTransparency = 1
	whiteFlash.BackgroundTransparency = 1
	topBar.Size = UDim2.new(1,0,0,0)
	bottomBar.Size = UDim2.new(1,0,0,0)

	tween(darkness, 0.7, {BackgroundTransparency = 0.35}, Enum.EasingStyle.Sine)
	tween(topBar, 0.6, {Size = UDim2.new(1,0,0,50)}, Enum.EasingStyle.Quart)
	tween(bottomBar, 0.6, {Size = UDim2.new(1,0,0,50)}, Enum.EasingStyle.Quart)
	sound1:Play()
	task.wait(1.7)

	whiteFlash.BackgroundTransparency = 0.8
	tween(whiteFlash, 0.15, {BackgroundTransparency = 1}, Enum.EasingStyle.Quad)
	purpleFlash.BackgroundTransparency = 0.5
	tween(purpleFlash, 0.45, {BackgroundTransparency = 1}, Enum.EasingStyle.Quart)
	sound2:Play()

	rinnegan.ImageTransparency = 0
	tween(rinnegan, 0.3, {Size = UDim2.fromOffset(220,220)}, Enum.EasingStyle.Exponential)
	task.wait(0.25)
	tween(rinnegan, 0.35, {Size = UDim2.fromOffset(280,280)}, Enum.EasingStyle.Back)
	task.wait(0.25)
	tween(rinnegan, 0.2, {Size = UDim2.fromOffset(270,270)}, Enum.EasingStyle.Quart)

	task.wait(0.3)
	shockwave(300, 0.7, 5)
	task.delay(0.15, function() shockwave(450, 0.9, 4) end)
	task.delay(0.3, function() shockwave(600, 1.1, 3) end)
	particleBurst(50)

	task.wait(2.0)
	tween(rinnegan, 0.6, {Size = UDim2.fromOffset(350,350), ImageTransparency = 1}, Enum.EasingStyle.Quart, Enum.EasingDirection.In)
	tween(darkness, 0.6, {BackgroundTransparency = 1}, Enum.EasingStyle.Sine)
	tween(topBar, 0.5, {Size = UDim2.new(1,0,0,0)}, Enum.EasingStyle.Quart)
	tween(bottomBar, 0.5, {Size = UDim2.new(1,0,0,0)}, Enum.EasingStyle.Quart)
	task.wait(0.6)

	gui:Destroy()
	player:SetAttribute("PlinioReady", true)
end

PlayIntro()
--[[
    ⚔️ PLÍNIO HUB — PARTE 2/3: BASE
    ✅ Speed + Super Jump + ESP + AutoAim + Teleport
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local Settings = {
    AutoAim = false,
    Speed = false,
    SuperJump = false,
    ESP = false,
    FOVRadius = 150,
    ShowFOV = false,
    TeleportLock = false,
    TargetPlayer = nil
}
_G.PlinioHub = Settings

local CurrentTarget = nil
local ESPObjects = {}
local FOVCircle = nil
local OriginalSpeed = 16
local OriginalJump = 50
_G.FOVUI = nil

local function CanSeeTarget(targetPart)
    if not targetPart then return false end
    local origin = Camera.CFrame.Position
    local direction = (targetPart.Position - origin).Unit
    local distance = (targetPart.Position - origin).Magnitude
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {LocalPlayer.Character}
    local result = workspace:Raycast(origin, direction * distance, params)
    if not result then return true end
    local hitModel = result.Instance:FindFirstAncestorWhichIsA("Model")
    local targetModel = targetPart:FindFirstAncestorWhichIsA("Model")
    return hitModel == targetModel
end

local function GetTargetsInFOV()
    local targets = {}
    local center = Camera.ViewportSize / 2
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local hum = player.Character:FindFirstChild("Humanoid")
            local head = player.Character:FindFirstChild("Head")
            if hum and head and hum.Health > 0 then
                local pos, onScreen = Camera:WorldToViewportPoint(head.Position)
                if onScreen then
                    local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(center.X, center.Y)).Magnitude
                    if dist <= Settings.FOVRadius and CanSeeTarget(head) then
                        table.insert(targets, {
                            Player = player,
                            Part = head,
                            Distance = dist,
                            Position = head.Position
                        })
                    end
                end
            end
        end
    end
    table.sort(targets, function(a, b) return a.Distance < b.Distance end)
    return targets
end

local function CreateFOV()
    if _G.FOVUI then return end
    local fovGui = Instance.new("ScreenGui")
    fovGui.Name = "PlinioFOV"
    fovGui.ResetOnSpawn = false
    fovGui.IgnoreGuiInset = true
    fovGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    _G.FOVUI = fovGui

    local circle = Instance.new("Frame")
    circle.Name = "FOVCircle"
    circle.Size = UDim2.new(0, Settings.FOVRadius*2, 0, Settings.FOVRadius*2)
    circle.Position = UDim2.new(0.5, -Settings.FOVRadius, 0.5, -Settings.FOVRadius)
    circle.BackgroundTransparency = 1
    circle.BorderSizePixel = 0
    circle.Visible = Settings.ShowFOV
    circle.Parent = fovGui
    Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)
    local stroke = Instance.new("UIStroke")
    stroke.Thickness = 2
    stroke.Color = Color3.fromRGB(255, 0, 0)
    stroke.Transparency = 0.1
    stroke.Parent = circle

    FOVCircle = circle
end

RunService.RenderStepped:Connect(function()
    if not FOVCircle then CreateFOV() return end
    FOVCircle.Visible = Settings.ShowFOV
    FOVCircle.Size = UDim2.new(0, Settings.FOVRadius*2, 0, Settings.FOVRadius*2)
    FOVCircle.Position = UDim2.new(0.5, -Settings.FOVRadius, 0.5, -Settings.FOVRadius)
end)

RunService.RenderStepped:Connect(function()
    if not Settings.AutoAim then
        CurrentTarget = nil
        return
    end
    local targets = GetTargetsInFOV()
    if #targets > 0 then
        CurrentTarget = targets[1].Player
        local targetPos = targets[1].Position
        local newCF = CFrame.lookAt(Camera.CFrame.Position, targetPos)
        Camera.CFrame = Camera.CFrame:Lerp(newCF, 0.2)
    else
        CurrentTarget = nil
    end
end)

RunService.Heartbeat:Connect(function()
    for _, h in pairs(ESPObjects) do pcall(function() h:Destroy() end) end
    ESPObjects = {}
    if not Settings.ESP then return end
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local hum = player.Character:FindFirstChild("Humanoid")
            if hum and hum.Health > 0 then
                local hl = Instance.new("Highlight")
                hl.Name = "ESP"
                hl.FillColor = Color3.fromRGB(255, 0, 0)
                hl.OutlineColor = Color3.fromRGB(255, 255, 255)
                hl.FillTransparency = 0.5
                hl.OutlineTransparency = 0
                hl.Adornee = player.Character
                hl.Parent = player.Character
                table.insert(ESPObjects, hl)
            end
        end
    end
end)

RunService.Heartbeat:Connect(function()
    if not Settings.TeleportLock or not Settings.TargetPlayer then return end
    local myChar = LocalPlayer.Character
    local tChar = Settings.TargetPlayer.Character
    if not myChar or not tChar then return end
    local myRoot = myChar:FindFirstChild("HumanoidRootPart")
    local tRoot = tChar:FindFirstChild("HumanoidRootPart")
    if myRoot and tRoot then
        myRoot.CFrame = tRoot.CFrame * CFrame.new(0, 2.5, 0)
    end
end)

game:GetService("RunService").Heartbeat:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end
    local hum = char:FindFirstChild("Humanoid")
    if not hum then return end

    if Settings.Speed then
        hum.WalkSpeed = 55
    else
        hum.WalkSpeed = OriginalSpeed
    end

    if Settings.SuperJump then
        hum.JumpPower = 120
    else
        hum.JumpPower = OriginalJump
    end
end)

task.spawn(CreateFOV)
--[[
    ⚔️ PLÍNIO HUB — PARTE 3/3: FECHAR DIRETO
    ✅ Botão X → fecha direto, sem confirmação
    ✅ Ao fechar → desativa TODAS as funções automaticamente
    ✅ Painel largo e arrumado como combinado
]]

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Settings = _G.PlinioHub

-- ✅ DESATIVA TUDO AO FECHAR
local function DesativarTudo()
    Settings.AutoAim = false
    Settings.Speed = false
    Settings.SuperJump = false
    Settings.ESP = false
    Settings.ShowFOV = false
    Settings.TeleportLock = false
    Settings.TargetPlayer = nil
end

local function OpenPanel()
    local old = PlayerGui:FindFirstChild("PlinioHubUI")
    if old then old:Destroy() end

    local Gui = Instance.new("ScreenGui")
    Gui.Name = "PlinioHubUI"
    Gui.ResetOnSpawn = false
    Gui.Parent = PlayerGui

    -- 📦 PAINEL PRINCIPAL
    local Main = Instance.new("Frame")
    Main.Size = UDim2.new(0, 380, 0, 320)
    Main.Position = UDim2.new(0.02, 0, 0.12, 0)
    Main.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
    Main.Active = true
    Main.ClipsDescendants = true
    Main.Parent = Gui
    Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
    local Stroke = Instance.new("UIStroke", Main)
    Stroke.Color = Color3.fromRGB(190, 0, 0)
    Stroke.Thickness = 2

    -- 📌 CABEÇALHO
    local Header = Instance.new("Frame")
    Header.Size = UDim2.new(1, 0, 0, 44)
    Header.BackgroundColor3 = Color3.fromRGB(180, 20, 40)
    Header.Parent = Main
    Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 12)
    Header.Active = true

    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(0.55, 0, 1, 0)
    Title.Position = UDim2.new(0, 14, 0, 0)
    Title.BackgroundTransparency = 1
    Title.Text = "⚔️ Plínio Hub"
    Title.Font = Enum.Font.GothamBold
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.TextSize = 15
    Title.TextXAlignment = Enum.TextXAlignment.Left
    Title.Parent = Header

    local MinBtn = Instance.new("TextButton")
    MinBtn.Size = UDim2.new(0, 32, 0, 32)
    MinBtn.Position = UDim2.new(0.86, 0, 0.5, -16)
    MinBtn.BackgroundColor3 = Color3.fromRGB(255, 200, 0)
    MinBtn.Text = "─"
    MinBtn.TextColor3 = Color3.fromRGB(0, 0, 0)
    MinBtn.TextSize = 20
    MinBtn.Font = Enum.Font.GothamBold
    MinBtn.Parent = Header
    Instance.new("UICorner", MinBtn).CornerRadius = UDim.new(0, 8)

    -- ❌ BOTÃO FECHAR — FECHA DIRETO
    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Size = UDim2.new(0, 32, 0, 32)
    CloseBtn.Position = UDim2.new(0.94, 0, 0.5, -16)
    CloseBtn.BackgroundColor3 = Color3.fromRGB(220, 30, 30)
    CloseBtn.Text = "X"
    CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseBtn.TextSize = 18
    CloseBtn.Font = Enum.Font.GothamBold
    CloseBtn.Parent = Header
    Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 8)

    -- 📄 CONTEÚDO
    local Content = Instance.new("Frame")
    Content.Size = UDim2.new(1, -16, 1, -60)
    Content.Position = UDim2.new(0, 8, 0, 48)
    Content.BackgroundTransparency = 1
    Content.ClipsDescendants = true
    Content.Parent = Main

    -- 🔘 BOTÃO TOGGLE
    local function CreateToggle(texto, x, y, padrao, acao)
        local Cont = Instance.new("Frame")
        Cont.Size = UDim2.new(0, 170, 0, 28)
        Cont.Position = UDim2.new(0, x, 0, y)
        Cont.BackgroundTransparency = 1
        Cont.Parent = Content

        local Lbl = Instance.new("TextLabel")
        Lbl.Size = UDim2.new(0, 110, 1, 0)
        Lbl.BackgroundTransparency = 1
        Lbl.Text = texto
        Lbl.Font = Enum.Font.Gotham
        Lbl.TextColor3 = Color3.fromRGB(230, 230, 230)
        Lbl.TextSize = 12
        Lbl.TextXAlignment = Enum.TextXAlignment.Left
        Lbl.Parent = Cont

        local Btn = Instance.new("TextButton")
        Btn.Size = UDim2.new(0, 52, 0, 24)
        Btn.Position = UDim2.new(1, -54, 0.5, -12)
        Btn.BackgroundColor3 = padrao and Color3.fromRGB(0, 180, 60) or Color3.fromRGB(55, 55, 75)
        Btn.Text = padrao and "ON" or "OFF"
        Btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        Btn.TextSize = 11
        Btn.Font = Enum.Font.GothamBold
        Btn.Parent = Cont
        Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 6)

        local ligado = padrao
        Btn.MouseButton1Click:Connect(function()
            ligado = not ligado
            Btn.BackgroundColor3 = ligado and Color3.fromRGB(0, 180, 60) or Color3.fromRGB(55, 55, 75)
            Btn.Text = ligado and "ON" or "OFF"
            acao(ligado)
        end)
    end

    -- 🎚️ SLIDER FOV
    local function CreateSlider(texto, y, valorMin, valorMax, valorInicial, acao)
        local Cont = Instance.new("Frame")
        Cont.Size = UDim2.new(1, 0, 0, 36)
        Cont.Position = UDim2.new(0, 0, 0, y)
        Cont.BackgroundTransparency = 1
        Cont.Parent = Content

        local Lbl = Instance.new("TextLabel")
        Lbl.Size = UDim2.new(1, 0, 0, 16)
        Lbl.BackgroundTransparency = 1
        Lbl.Text = texto .. ": " .. valorInicial
        Lbl.Font = Enum.Font.Gotham
        Lbl.TextColor3 = Color3.fromRGB(230, 230, 230)
        Lbl.TextSize = 11
        Lbl.TextXAlignment = Enum.TextXAlignment.Left
        Lbl.Parent = Cont

        local BarBg = Instance.new("Frame")
        BarBg.Size = UDim2.new(1, 0, 0, 12)
        BarBg.Position = UDim2.new(0, 0, 0, 20)
        BarBg.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
        BarBg.Parent = Cont
        Instance.new("UICorner", BarBg).CornerRadius = UDim.new(0, 6)

        local BarFill = Instance.new("Frame")
        BarFill.Size = UDim2.new(0, (valorInicial-valorMin)/(valorMax-valorMin), 1, 0)
        BarFill.BackgroundColor3 = Color3.fromRGB(220, 30, 60)
        BarFill.Parent = BarBg
        Instance.new("UICorner", BarFill).CornerRadius = UDim.new(0, 6)

        local atualValor = valorInicial
        local function Atualizar(input)
            local pos = input.Position.X - BarBg.AbsolutePosition.X
            local porcent = math.clamp(pos / BarBg.AbsoluteSize.X, 0, 1)
            atualValor = math.floor(valorMin + (valorMax - valorMin) * porcent)
            BarFill.Size = UDim2.new(porcent, 0, 1, 0)
            Lbl.Text = texto .. ": " .. atualValor
            acao(atualValor)
        end
        BarBg.InputBegan:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then Atualizar(i) end
        end)
        BarBg.InputChanged:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then Atualizar(i) end
        end)
    end

    -- 📋 LISTA DE JOGADORES
    local ListaAberta = false
    local ListaFrame = Instance.new("ScrollingFrame")
    ListaFrame.Size = UDim2.new(1, 0, 0, 85)
    ListaFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
    ListaFrame.ScrollBarThickness = 4
    ListaFrame.Visible = false
    ListaFrame.Parent = Content
    Instance.new("UICorner", ListaFrame).CornerRadius = UDim.new(0, 8)
    Instance.new("UIListLayout", ListaFrame).Padding = UDim.new(0, 4)

    local BtnLista = Instance.new("TextButton")
    BtnLista.Size = UDim2.new(1, 0, 0, 30)
    BtnLista.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
    BtnLista.Text = "📋 Clique para selecionar jogador"
    BtnLista.TextColor3 = Color3.fromRGB(220, 220, 220)
    BtnLista.TextSize = 11
    BtnLista.Font = Enum.Font.Gotham
    BtnLista.Parent = Content
    Instance.new("UICorner", BtnLista).CornerRadius = UDim.new(0, 8)

    local function AtualizarLista()
        for _, c in pairs(ListaFrame:GetChildren()) do if c:IsA("TextButton") then c:Destroy() end end
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer then
                local B = Instance.new("TextButton")
                B.Size = UDim2.new(1, 0, 0, 28)
                B.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
                B.Text = "👤 " .. p.Name
                B.TextColor3 = Color3.fromRGB(240, 240, 240)
                B.TextSize = 11
                B.Font = Enum.Font.Gotham
                B.Parent = ListaFrame
                Instance.new("UICorner", B).CornerRadius = UDim.new(0, 6)
                B.MouseButton1Click:Connect(function()
                    Settings.TargetPlayer = p
                    BtnLista.Text = "✅ Selecionado: " .. p.Name
                    ListaAberta = false
                    ListaFrame.Visible = false
                end)
            end
        end
        ListaFrame.CanvasSize = UDim2.new(0, 0, 0, #ListaFrame:GetChildren() * 32)
    end

    BtnLista.MouseButton1Click:Connect(function()
        ListaAberta = not ListaAberta
        if ListaAberta then
            AtualizarLista()
            ListaFrame.Visible = true
            BtnLista.Text = "📋 Fechar lista"
        else
            ListaFrame.Visible = false
            BtnLista.Text = "📋 Clique para selecionar jogador"
        end
    end)

    -- 📊 COLUNAS DE FUNÇÕES
    CreateToggle("🎯 Auto Aim", 0, 0, Settings.AutoAim, function(v) Settings.AutoAim = v end)
    CreateToggle("⚡ Speed", 190, 0, Settings.Speed, function(v) Settings.Speed = v end)
    CreateToggle("🦘 Super Jump", 0, 32, Settings.SuperJump, function(v) Settings.SuperJump = v end)
    CreateToggle("👁️ ESP", 190, 32, Settings.ESP, function(v) Settings.ESP = v end)
    CreateToggle("🔴 Mostrar FOV", 0, 64, Settings.ShowFOV, function(v) Settings.ShowFOV = v end)
    CreateToggle("🔒 Seguir/Travar", 190, 64, Settings.TeleportLock, function(v)
        Settings.TeleportLock = v
        if not v then Settings.TargetPlayer = nil; BtnLista.Text = "📋 Clique para selecionar jogador" end
    end)
    CreateSlider("🎚️ Tamanho FOV", 96, 50, 400, Settings.FOVRadius, function(v) Settings.FOVRadius = v end)

    local TeleLabel = Instance.new("TextLabel")
    TeleLabel.Size = UDim2.new(1, 0, 0, 20)
    TeleLabel.Position = UDim2.new(0, 0, 0, 150)
    TeleLabel.BackgroundTransparency = 1
    TeleLabel.Text = "🚀 Teleportar e Travar"
    TeleLabel.Font = Enum.Font.GothamBold
    TeleLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
    TeleLabel.TextSize = 12
    TeleLabel.Parent = Content

    BtnLista.Position = UDim2.new(0, 0, 0, 175)
    ListaFrame.Position = UDim2.new(0, 0, 0, 210)

    -- ✅ STATUS
    local Status = Instance.new("TextLabel")
    Status.Size = UDim2.new(1, 0, 0, 28)
    Status.Position = UDim2.new(0, 0, 0, 288)
    Status.BackgroundColor3 = Color3.fromRGB(55, 55, 75)
    Status.Text = "✅ INATIVO"
    Status.Font = Enum.Font.GothamBold
    Status.TextColor3 = Color3.fromRGB(255, 255, 255)
    Status.TextSize = 11
    Status.Parent = Main
    Instance.new("UICorner", Status).CornerRadius = UDim.new(0, 8)

    game:GetService("RunService").Heartbeat:Connect(function()
        pcall(function()
            if Settings.TeleportLock and Settings.TargetPlayer then
                Status.Text = "🚀 TRAVADO: " .. Settings.TargetPlayer.Name
                Status.BackgroundColor3 = Color3.fromRGB(255, 180, 0)
            elseif Settings.AutoAim then
                Status.Text = "⚔️ AUTO AIM ATIVO"
                Status.BackgroundColor3 = Color3.fromRGB(220, 30, 30)
            elseif Settings.Speed then
                Status.Text = "⚡ SPEED ATIVO"
                Status.BackgroundColor3 = Color3.fromRGB(0, 180, 60)
            elseif Settings.SuperJump then
                Status.Text = "🦘 SUPER JUMP ATIVO"
                Status.BackgroundColor3 = Color3.fromRGB(0, 180, 60)
            else
                Status.Text = "✅ INATIVO"
                Status.BackgroundColor3 = Color3.fromRGB(55, 55, 75)
            end
        end)
    end)

    -- 🖱️ ARRASTAR
    local Arrastando, Inicio, PosInicial
    local function Comecar(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            Arrastando = true; Inicio = input.Position; PosInicial = Main.Position
        end
    end
    local function Mover(input)
        if Arrastando then
            local d = input.Position - Inicio
            Main.Position = UDim2.new(PosInicial.X.Scale, PosInicial.X.Offset + d.X, PosInicial.Y.Scale, PosInicial.Y.Offset + d.Y)
        end
    end
    local function Parar() Arrastando = false end
    Header.InputBegan:Connect(Comecar); Header.InputChanged:Connect(Mover); Header.InputEnded:Connect(Parar)
    Title.InputBegan:Connect(Comecar); Title.InputChanged:Connect(Mover); Title.InputEnded:Connect(Parar)

    -- ✅ MINIMIZAR
    local Minimizado = false
    local TAMANHO_NORMAL = UDim2.new(0, 380, 0, 320)
    local TAMANHO_MINIMIZADO = UDim2.new(0, 380, 0, 44)
    MinBtn.MouseButton1Click:Connect(function()
        Minimizado = not Minimizado
        if Minimizado then
            Main.Size = TAMANHO_MINIMIZADO; Content.Visible = false; Status.Visible = false; MinBtn.Text = "□"
            ListaFrame.Visible = false; ListaAberta = false
        else
            Main.Size = TAMANHO_NORMAL; Content.Visible = true; Status.Visible = true; MinBtn.Text = "─"
        end
    end)

    -- ❌ FECHAR — DIRETO + DESATIVA TUDO ✅
    CloseBtn.MouseButton1Click:Connect(function()
        DesativarTudo() -- ⚡ Desativa TUDO antes de fechar!
        Gui:Destroy()
    end)

    -- ✨ ABERTURA ANIMADA
    Main.Size = UDim2.new(0, 0, 0, 0)
    Main.Position = UDim2.new(0.5, 0, 0.5, 0)
    Main.AnchorPoint = Vector2.new(0.5, 0.5)
    Main.BackgroundTransparency = 1
    Stroke.Transparency = 1
    Content.Visible = false
    Status.Visible = false

    task.wait(0.03)
    TweenService:Create(Main, TweenInfo.new(0.35, Enum.EasingStyle.Back), {
        Size = TAMANHO_NORMAL,
        Position = UDim2.new(0.02, 0, 0.12, 0),
        AnchorPoint = Vector2.new(0, 0),
        BackgroundTransparency = 0
    }):Play()
    TweenService:Create(Stroke, TweenInfo.new(0.35), {Transparency = 0}):Play()
    task.wait(0.25)
    Content.Visible = true
    Status.Visible = true
end

-- ⏱️ ABRIR AUTOMATICAMENTE
task.spawn(function()
    LocalPlayer:GetAttributeChangedSignal("PlinioReady"):Connect(function()
        if LocalPlayer:GetAttribute("PlinioReady") then OpenPanel() end
    end)
    task.wait(10)
    if not PlayerGui:FindFirstChild("PlinioHubUI") then OpenPanel() end
end)

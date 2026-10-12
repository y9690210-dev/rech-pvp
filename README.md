local Player = game.Players.LocalPlayer
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local Mouse = Player:GetMouse()

_G.Config = {
    SpeedActive = false,
    AutoFollowCamera = false,
    UltraAim = false,
    CustomSpeed = 165,
    AtaqueLonge = false,
    WaterWalk = false,
    FlashStepInf = false,
    AuraRange = math.huge,
    StretchedActive = false,
    FpsBoostActive = false,
    HitboxActive = false,
    HitboxShow = false,
    HitboxSize = 50
}

local CurrentTarget = nil

local function IsTargetValid(target)
    if not target or not target:FindFirstChild("HumanoidRootPart") or not target:FindFirstChild("Humanoid") then return false end
    if target.Humanoid.Health <= 0 then return false end
    local targetPlayer = game.Players:GetPlayerFromCharacter(target)
    if targetPlayer and targetPlayer.Team == Player.Team then return false end
    return true
end

local function GetClosestPlayer()
    local target = nil
    local dist = math.huge
    local myRoot = Player.Character and Player.Character:FindFirstChild("HumanoidRootPart")
    if not myRoot then return nil end

    for _, p in pairs(game.Players:GetPlayers()) do
        if p ~= Player and p.Team ~= Player.Team and p.Character and p.Character:FindFirstChild("HumanoidRootPart") and p.Character:FindFirstChild("Humanoid") and p.Character.Humanoid.Health > 0 then
            local d = (myRoot.Position - p.Character.HumanoidRootPart.Position).Magnitude
            if d < dist then
                dist = d
                target = p.Character
            end
        end
    end
    return target
end

local function GetOrUpdateTarget()
    if CurrentTarget and IsTargetValid(CurrentTarget) then
        return CurrentTarget
    else
        CurrentTarget = GetClosestPlayer()
        return CurrentTarget
    end
end

local ScreenGui = Instance.new("ScreenGui", Player.PlayerGui)
ScreenGui.Name = "RECH_V40_SIDE_TABS"
ScreenGui.ResetOnSpawn = false

-- ====== MENU PRINCIPAL (Largo, Arredondado e Compacto) ======
local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 390, 0, 250)
MainFrame.Position = UDim2.new(0.5, -195, 0.5, -125)
MainFrame.BackgroundColor3 = Color3.fromRGB(12, 12, 18)
MainFrame.BorderColor3 = Color3.fromRGB(0, 150, 255)
MainFrame.BorderSizePixel = 1
MainFrame.Active = true; MainFrame.Draggable = true
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 10)

-- Barra Superior (Título e Botões Minimizar / Fechar)
local TopBar = Instance.new("Frame", MainFrame)
TopBar.Size = UDim2.new(1, 0, 0, 32)
TopBar.BackgroundTransparency = 1

local TitleLabel = Instance.new("TextLabel", TopBar)
TitleLabel.Size = UDim2.new(1, -80, 1, 0)
TitleLabel.Position = UDim2.new(0, 10, 0, 0)
TitleLabel.Text = "⚡ RECH HUB V40"
TitleLabel.TextColor3 = Color3.fromRGB(0, 180, 255)
TitleLabel.TextSize = 15
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.BackgroundTransparency = 1

local MinimizeBtn = Instance.new("TextButton", TopBar)
MinimizeBtn.Size = UDim2.new(0, 28, 0, 28)
MinimizeBtn.Position = UDim2.new(1, -64, 0, 2)
MinimizeBtn.Text = "-"
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
MinimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
Instance.new("UICorner", MinimizeBtn).CornerRadius = UDim.new(0, 6)

local DeleteX = Instance.new("TextButton", TopBar)
DeleteX.Size = UDim2.new(0, 28, 0, 28)
DeleteX.Position = UDim2.new(1, -32, 0, 2)
DeleteX.Text = "X"
DeleteX.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
DeleteX.TextColor3 = Color3.fromRGB(255, 255, 255)
Instance.new("UICorner", DeleteX).CornerRadius = UDim.new(0, 6)

-- ====== ABAS NA LATERAL ESQUERDA ======
local Sidebar = Instance.new("Frame", MainFrame)
Sidebar.Size = UDim2.new(0, 105, 1, -40)
Sidebar.Position = UDim2.new(0, 5, 0, 35)
Sidebar.BackgroundTransparency = 1

local function CreateTabBtn(yPos, text)
    local btn = Instance.new("TextButton", Sidebar)
    btn.Size = UDim2.new(1, 0, 0, 34)
    btn.Position = UDim2.new(0, 0, 0, yPos)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(20, 20, 28)
    btn.TextColor3 = Color3.fromRGB(200, 200, 200)
    btn.TextSize = 12
    btn.Font = Enum.Font.GothamSemibold
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    return btn
end

local TabCombat = CreateTabBtn(0, "⚔️ Combate")
local TabMovement = CreateTabBtn(40, "🏃 Movimento")
local TabConfig = CreateTabBtn(80, "⚙️ Config")

-- ====== PAINÉIS DE CONTEÚDO (DIREITA) ======
local function CreatePanel()
    local p = Instance.new("Frame", MainFrame)
    p.Size = UDim2.new(1, -120, 1, -42)
    p.Position = UDim2.new(0, 115, 0, 38)
    p.BackgroundTransparency = 1
    p.Visible = false
    return p
end

local CombatPanel = CreatePanel()
local MovementPanel = CreatePanel()
ConfigPanel = CreatePanel()

CombatPanel.Visible = true
TabCombat.BackgroundColor3 = Color3.fromRGB(0, 120, 220)
TabCombat.TextColor3 = Color3.fromRGB(255, 255, 255)

local function SwitchTab(tab, panel)
    CombatPanel.Visible = false
    MovementPanel.Visible = false
    ConfigPanel.Visible = false
    
    TabCombat.BackgroundColor3 = Color3.fromRGB(20, 20, 28); TabCombat.TextColor3 = Color3.fromRGB(200, 200, 200)
    TabMovement.BackgroundColor3 = Color3.fromRGB(20, 20, 28); TabMovement.TextColor3 = Color3.fromRGB(200, 200, 200)
    TabConfig.BackgroundColor3 = Color3.fromRGB(20, 20, 28); TabConfig.TextColor3 = Color3.fromRGB(200, 200, 200)
    
    panel.Visible = true
    tab.BackgroundColor3 = Color3.fromRGB(0, 120, 220)
    tab.TextColor3 = Color3.fromRGB(255, 255, 255)
end

TabCombat.MouseButton1Click:Connect(function() SwitchTab(TabCombat, CombatPanel) end)
TabMovement.MouseButton1Click:Connect(function() SwitchTab(TabMovement, MovementPanel) end)
TabConfig.MouseButton1Click:Connect(function() SwitchTab(TabConfig, ConfigPanel) end)

local function StyleButton(btn)
    btn.BackgroundColor3 = Color3.fromRGB(22, 22, 30)
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 11
    btn.Font = Enum.Font.GothamSemibold
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
end

-- ====== ITENS: ABA COMBATE ======
local ataqueBtn = Instance.new("TextButton", CombatPanel); ataqueBtn.Size = UDim2.new(1, 0, 0, 30); ataqueBtn.Position = UDim2.new(0, 0, 0, 2); ataqueBtn.Text = "ATAQUE DE LONGE: OFF"; StyleButton(ataqueBtn)
ataqueBtn.MouseButton1Click:Connect(function()
    _G.Config.AtaqueLonge = not _G.Config.AtaqueLonge
    if not _G.Config.AtaqueLonge then CurrentTarget = nil end
    ataqueBtn.Text = _G.Config.AtaqueLonge and "ATAQUE DE LONGE: ON 🔥" or "ATAQUE DE LONGE: OFF"
    ataqueBtn.TextColor3 = _G.Config.AtaqueLonge and Color3.new(1, 0.5, 0) or Color3.new(1, 1, 1)
end)

local aimBtn = Instance.new("TextButton", CombatPanel); aimBtn.Size = UDim2.new(1, 0, 0, 30); aimBtn.Position = UDim2.new(0, 0, 0, 36); aimBtn.Text = "MIRA NA CÂMERA (SNIPER)"; StyleButton(aimBtn)
aimBtn.MouseButton1Click:Connect(function() _G.Config.UltraAim = not _G.Config.UltraAim end)

local hitboxToggleBtn = Instance.new("TextButton", CombatPanel); hitboxToggleBtn.Size = UDim2.new(1, 0, 0, 30); hitboxToggleBtn.Position = UDim2.new(0, 0, 0, 70); hitboxToggleBtn.Text = "HITBOX EXTENDER: OFF"; StyleButton(hitboxToggleBtn)
hitboxToggleBtn.MouseButton1Click:Connect(function()
    _G.Config.HitboxActive = not _G.Config.HitboxActive
    hitboxToggleBtn.Text = _G.Config.HitboxActive and "HITBOX EXTENDER: ON 🔥" or "HITBOX EXTENDER: OFF"
    hitboxToggleBtn.TextColor3 = _G.Config.HitboxActive and Color3.fromRGB(255, 128, 0) or Color3.fromRGB(255, 255, 255)
    if not _G.Config.HitboxActive then
        for _, p in pairs(game.Players:GetPlayers()) do
            if p ~= Player and p.Character then
                local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                local head = p.Character:FindFirstChild("Head")
                local torso = p.Character:FindFirstChild("Torso") or p.Character:FindFirstChild("UpperTorso")
                if hrp then hrp.Size = Vector3.new(2, 2, 1) hrp.Transparency = 1 end
                if head then head.Size = Vector3.new(1, 1, 1) head.Transparency = 0 end
                if torso then torso.Size = Vector3.new(2, 2, 1) torso.Transparency = 1 end
            end
        end
    end
end)

local hitboxShowBtn = Instance.new("TextButton", CombatPanel); hitboxShowBtn.Size = UDim2.new(1, 0, 0, 30); hitboxShowBtn.Position = UDim2.new(0, 0, 0, 104); hitboxShowBtn.Text = "VER HITBOX: OFF"; StyleButton(hitboxShowBtn)
hitboxShowBtn.MouseButton1Click:Connect(function()
    _G.Config.HitboxShow = not _G.Config.HitboxShow
    hitboxShowBtn.Text = _G.Config.HitboxShow and "VER HITBOX: ON 👁️" or "VER HITBOX: OFF"
    hitboxShowBtn.TextColor3 = _G.Config.HitboxShow and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 255, 255)
end)


-- ====== ITENS: ABA MOVIMENTO (Incluindo Seguir Player) ======
local speedToggleBtn = Instance.new("TextButton", MovementPanel); speedToggleBtn.Size = UDim2.new(1, 0, 0, 26); speedToggleBtn.Position = UDim2.new(0, 0, 0, 2); speedToggleBtn.Text = "VELOCIDADE FUGA: OFF"; StyleButton(speedToggleBtn)
speedToggleBtn.MouseButton1Click:Connect(function()
    _G.Config.SpeedActive = not _G.Config.SpeedActive
    speedToggleBtn.Text = _G.Config.SpeedActive and "VELOCIDADE: ON 🔥" or "VELOCIDADE FUGA: OFF"
end)

local SpeedTextBox = Instance.new("TextBox", MovementPanel)
SpeedTextBox.Size = UDim2.new(1, 0, 0, 24)
SpeedTextBox.Position = UDim2.new(0, 0, 0, 31)
SpeedTextBox.BackgroundColor3 = Color3.fromRGB(18, 18, 25)
SpeedTextBox.TextColor3 = Color3.fromRGB(255, 255, 255)
SpeedTextBox.PlaceholderText = "Velocidade (Ex: 165)"
SpeedTextBox.Text = "165"
SpeedTextBox.TextSize = 11
Instance.new("UICorner", SpeedTextBox).CornerRadius = UDim.new(0, 6)
SpeedTextBox.FocusLost:Connect(function()
    local val = tonumber(SpeedTextBox.Text)
    if val then _G.Config.CustomSpeed = val else SpeedTextBox.Text = tostring(_G.Config.CustomSpeed) end
end)

local followBtn = Instance.new("TextButton", MovementPanel); followBtn.Size = UDim2.new(1, 0, 0, 26); followBtn.Position = UDim2.new(0, 0, 0, 58); followBtn.Text = "SEGUIR PLAYER (∞)"; StyleButton(followBtn)

local waterBtn = Instance.new("TextButton", MovementPanel); waterBtn.Size = UDim2.new(1, 0, 0, 26); waterBtn.Position = UDim2.new(0, 0, 0, 87); waterBtn.Text = "ANDAR NA ÁGUA: OFF"; StyleButton(waterBtn)
waterBtn.MouseButton1Click:Connect(function()
    _G.Config.WaterWalk = not _G.Config.WaterWalk
    waterBtn.Text = _G.Config.WaterWalk and "ANDAR NA ÁGUA: ON 🔥" or "ANDAR NA ÁGUA: OFF"
end)

local flashMenuBtn = Instance.new("TextButton", MovementPanel); flashMenuBtn.Size = UDim2.new(1, 0, 0, 26); flashMenuBtn.Position = UDim2.new(0, 0, 0, 116); flashMenuBtn.Text = "FLASH STEP (INF)"; StyleButton(flashMenuBtn)
flashMenuBtn.MouseButton1Click:Connect(function()
    _G.Config.FlashStepInf = not _G.Config.FlashStepInf
    flashMenuBtn.Text = _G.Config.FlashStepInf and "FLASH STEP: ATIVO 🔥" or "FLASH STEP (INF)"
end)

local tpBtn = Instance.new("TextButton", MovementPanel); tpBtn.Size = UDim2.new(1, 0, 0, 26); tpBtn.Position = UDim2.new(0, 0, 0, 145); tpBtn.Text = "TP: CÉU (FUGA)"; StyleButton(tpBtn)


-- ====== ITENS: ABA CONFIGURAÇÕES ======
local stretchBtn = Instance.new("TextButton", ConfigPanel); stretchBtn.Size = UDim2.new(1, 0, 0, 36); stretchBtn.Position = UDim2.new(0, 0, 0, 15); stretchBtn.Text = "TELA ESTICADA: OFF"; StyleButton(stretchBtn)
stretchBtn.MouseButton1Click:Connect(function()
    _G.Config.StretchedActive = not _G.Config.StretchedActive
    if _G.Config.StretchedActive then
        stretchBtn.Text = "TELA ESTICADA: ON 🔥"
        stretchBtn.TextColor3 = Color3.fromRGB(255, 128, 0)
    else
        stretchBtn.Text = "TELA ESTICADA: OFF"
        stretchBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        Camera.FieldOfView = 70
    end
end)

local fpsBtn = Instance.new("TextButton", ConfigPanel); fpsBtn.Size = UDim2.new(1, 0, 0, 36); fpsBtn.Position = UDim2.new(0, 0, 0, 58); fpsBtn.Text = "FPS BOOST (LIMPEZA REAL)"; StyleButton(fpsBtn)
fpsBtn.MouseButton1Click:Connect(function()
    _G.Config.FpsBoostActive = true
    fpsBtn.Text = "FPS BOOST: ATIVO ✅"
    fpsBtn.TextColor3 = Color3.fromRGB(0, 255, 0)
    pcall(function()
        game:GetService("Lighting").GlobalShadows = false
        game:GetService("Lighting").FogEnd = 9e9
        for _, v in pairs(game:GetService("Lighting"):GetChildren()) do
            if v:IsA("PostEffect") or v:IsA("Sky") or v:IsA("Atmosphere") then v:Destroy() end
        end
    end)
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") then
            v.Material = Enum.Material.SmoothPlastic
            v.Reflectance = 0
        elseif v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") or v:IsA("Fire") or v:IsA("Smoke") or v:IsA("Sparkles") or v:IsA("Explosion") then
            v:Destroy() -- Deleta os efeitos pesados para sumirem de vez!
        elseif v:IsA("Decal") or v:IsA("Texture") then
            v:Destroy()
        end
    end
end)


-- ====== BOTÕES FLUTUANTES NA TELA ======
local FollowFloatBtn = Instance.new("TextButton", ScreenGui)
FollowFloatBtn.Size = UDim2.new(0, 110, 0, 32); FollowFloatBtn.Position = UDim2.new(0.05, 0, 0.30, 0)
FollowFloatBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 255); FollowFloatBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
FollowFloatBtn.Text = "🎯 SEGUIR: OFF"; FollowFloatBtn.Visible = true; FollowFloatBtn.Active = true; FollowFloatBtn.Draggable = true
Instance.new("UICorner", FollowFloatBtn).CornerRadius = UDim.new(0, 8)

local FlashFloatBtn = Instance.new("TextButton", ScreenGui)
FlashFloatBtn.Size = UDim2.new(0, 110, 0, 32); FlashFloatBtn.Position = UDim2.new(0.05, 0, 0.38, 0)
FlashFloatBtn.BackgroundColor3 = Color3.fromRGB(255, 100, 0); FlashFloatBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
FlashFloatBtn.Text = "⚡ FLASH STEP"; FlashFloatBtn.Visible = true; FlashFloatBtn.Active = true; FlashFloatBtn.Draggable = true
Instance.new("UICorner", FlashFloatBtn).CornerRadius = UDim.new(0, 8)

local TpFloatBtn = Instance.new("TextButton", ScreenGui)
TpFloatBtn.Size = UDim2.new(0, 110, 0, 32); TpFloatBtn.Position = UDim2.new(0.05, 0, 0.46, 0)
TpFloatBtn.BackgroundColor3 = Color3.fromRGB(255, 128, 0); TpFloatBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
TpFloatBtn.Text = "☁️ TP CÉU"; TpFloatBtn.Visible = true; TpFloatBtn.Active = true; TpFloatBtn.Draggable = true
Instance.new("UICorner", TpFloatBtn).CornerRadius = UDim.new(0, 8)

-- Bola minimizada
local RechBall = Instance.new("TextButton", ScreenGui)
RechBall.Size = UDim2.new(0, 50, 0, 50); RechBall.Position = UDim2.new(0.1, 0, 0.1, 0)
RechBall.BackgroundColor3 = Color3.fromRGB(0, 150, 255); RechBall.Text = "RECH"; RechBall.Visible = false
RechBall.TextColor3 = Color3.fromRGB(255, 255, 255); RechBall.Font = Enum.Font.GothamBold
Instance.new("UICorner", RechBall).CornerRadius = UDim.new(1, 0)

MinimizeBtn.MouseButton1Click:Connect(function() MainFrame.Visible = false; RechBall.Visible = true end)
DeleteX.MouseButton1Click:Connect(function() ScreenGui:Destroy() end)
RechBall.MouseButton1Click:Connect(function() MainFrame.Visible = true; RechBall.Visible = false end)

-- Conexão de Seguir Player
local function ToggleFollow()
    _G.Config.AutoFollowCamera = not _G.Config.AutoFollowCamera
    if not _G.Config.AutoFollowCamera then
        CurrentTarget = nil
        FollowFloatBtn.Text = "🎯 SEGUIR: OFF"
        FollowFloatBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
        followBtn.Text = "SEGUIR PLAYER (∞)"
    else
        FollowFloatBtn.Text = "🎯 SEGUIR: ON 🔥"
        FollowFloatBtn.BackgroundColor3 = Color3.fromRGB(255, 128, 0)
        followBtn.Text = "SEGUIR PLAYER: ON 🔥"
    end
end
followBtn.MouseButton1Click:Connect(ToggleFollow)
FollowFloatBtn.MouseButton1Click:Connect(ToggleFollow)

-- Flash Step Flutuante
FlashFloatBtn.MouseButton1Click:Connect(function()
    local char = Player.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        local target = GetOrUpdateTarget()
        if target and target:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = target.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)
        end
    end
end)

-- TP Céu
local function TpSky()
    if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
        Player.Character.HumanoidRootPart.CFrame = Player.Character.HumanoidRootPart.CFrame * CFrame.new(0, 500, 0)
    end
end
tpBtn.MouseButton1Click:Connect(TpSky)
TpFloatBtn.MouseButton1Click:Connect(TpSky)

local WaterPlatform = Instance.new("Part")
WaterPlatform.Name = "RechWaterPlatform"
WaterPlatform.Size = Vector3.new(12, 1, 12)
WaterPlatform.Anchored = true
WaterPlatform.Transparency = 1
WaterPlatform.CanCollide = false
WaterPlatform.Parent = workspace

-- LOOP PRINCIPAL
RunService.Heartbeat:Connect(function()
    local Char = Player.Character
    if not Char or not Char:FindFirstChild("HumanoidRootPart") then return end
    
    if _G.Config.StretchedActive then Camera.FieldOfView = 100 end

    if _G.Config.WaterWalk then
        local rootPos = Char.HumanoidRootPart.Position
        if rootPos.Y < 15 then
            WaterPlatform.Position = Vector3.new(rootPos.X, rootPos.Y - 3, rootPos.Z)
            WaterPlatform.CanCollide = true
        else
            WaterPlatform.CanCollide = false
        end
    else
        WaterPlatform.CanCollide = false
    end

    if _G.Config.HitboxActive then
        for _, p in pairs(game.Players:GetPlayers()) do
            if p ~= Player and p.Team ~= Player.Team and p.Character then
                local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                local head = p.Character:FindFirstChild("Head")
                local torso = p.Character:FindFirstChild("Torso") or p.Character:FindFirstChild("UpperTorso")
                
                if hrp then
                    hrp.Size = Vector3.new(_G.Config.HitboxSize, _G.Config.HitboxSize, _G.Config.HitboxSize)
                    hrp.CanCollide = false
                    hrp.Transparency = _G.Config.HitboxShow and 0.5 or 1
                    if _G.Config.HitboxShow then hrp.Color = Color3.fromRGB(255, 0, 0) end
                end
                if head then
                    head.Size = Vector3.new(_G.Config.HitboxSize, _G.Config.HitboxSize, _G.Config.HitboxSize)
                    head.CanCollide = false
                    head.Transparency = _G.Config.HitboxShow and 0.5 or 1
                end
                if torso then
                    torso.Size = Vector3.new(_G.Config.HitboxSize, _G.Config.HitboxSize, _G.Config.HitboxSize)
                    torso.CanCollide = false
                    torso.Transparency = _G.Config.HitboxShow and 0.5 or 1
                end
            end
        end
    end

    if _G.Config.AtaqueLonge then
        local target = GetOrUpdateTarget()
        if target and target:FindFirstChild("HumanoidRootPart") then
            local tool = Char:FindFirstChildOfClass("Tool")
            if tool then
                pcall(function()
                    tool:Activate()
                    for _, part in pairs(tool:GetDescendants()) do
                        if part:IsA("BasePart") then part.CFrame = target.HumanoidRootPart.CFrame end
                    end
                    if tool:FindFirstChild("ServerControl") then
                        tool.ServerControl:InvokeServer("Click", target.HumanoidRootPart.Position)
                    end
                end)
            end
        end
    end
    
    if _G.Config.UltraAim then 
        pcall(function() Mouse.Hit = CFrame.new(Camera.CFrame.Position + (Camera.CFrame.LookVector * 500)) end)
    end
    
    if _G.Config.SpeedActive and Char.Humanoid.MoveDirection.Magnitude > 0 then
        Char.HumanoidRootPart.CFrame = Char.HumanoidRootPart.CFrame + (Char.Humanoid.MoveDirection * 

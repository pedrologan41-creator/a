-- // SCRIPT FINAL - PAINEL RICKY VIP (ESP CORRIGIDO DEFINITIVO)
-- // Cálculo manual de projeção 3D->2D (Head para Feet)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local Config = {
    Aimbot_Enabled = false,
    Aimbot_TargetPart = "Head",
    Aimbot_MaxDistance = 250,
    Aimbot_WallCheck = false,
    Aimbot_TeamCheck = true,
    Aimbot_Smoothing = false,
    Aimbot_Smoothness = 5,
    
    FOV_Enabled = false,
    FOV_Radius = 150,
    FOV_Color = Color3.fromRGB(255, 255, 255),
    
    ESP_Enabled = false,
    ESP_Box = false,
    ESP_Name = false,
    ESP_Health = false,
    ESP_Line = false,
    ESP_HealthPos = "Top"
}

local Theme = {
    Background = Color3.fromRGB(15, 15, 15),
    Sidebar = Color3.fromRGB(20, 20, 20),
    ColumnBG = Color3.fromRGB(25, 25, 25),
    ElementBG = Color3.fromRGB(35, 35, 35),
    Text = Color3.fromRGB(255, 255, 255),
    Accent = Color3.fromRGB(255, 0, 0),
    Inactive = Color3.fromRGB(60, 60, 60)
}

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PainelRickyVip"
ScreenGui.Parent = game.CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local function CreateToggle(parent, text, defaultState, callback)
    local Container = Instance.new("Frame")
    Container.Size = UDim2.new(1, 0, 0, 30)
    Container.BackgroundTransparency = 1
    Container.Parent = parent

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.7, 0, 1, 0)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Theme.Text
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Font = Enum.Font.Gotham
    Label.TextSize = 13
    Label.Parent = Container

    local Button = Instance.new("TextButton")
    Button.Size = UDim2.new(0, 35, 0, 18)
    Button.Position = UDim2.new(1, -35, 0.5, -9)
    Button.BackgroundColor3 = defaultState and Theme.Accent or Theme.Inactive
    Button.Text = ""
    Button.Parent = Container
    
    local UICorner = Instance.new("UICorner", Button)
    UICorner.CornerRadius = UDim.new(1, 0)

    local state = defaultState
    local toggleRef = { Button = Button, State = state }
    
    Button.MouseButton1Click:Connect(function()
        state = not state
        toggleRef.State = state
        Button.BackgroundColor3 = state and Theme.Accent or Theme.Inactive
        callback(state)
    end)
    
    return toggleRef
end

local function CreateDropdown(parent, text, options, defaultOption, callback)
    local Container = Instance.new("Frame")
    Container.Size = UDim2.new(1, 0, 0, 45)
    Container.BackgroundTransparency = 1
    Container.Parent = parent

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(1, 0, 0, 18)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Theme.Text
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Font = Enum.Font.Gotham
    Label.TextSize = 13
    Label.Parent = Container

    local Button = Instance.new("TextButton")
    Button.Size = UDim2.new(1, 0, 0, 22)
    Button.Position = UDim2.new(0, 0, 0, 20)
    Button.BackgroundColor3 = Theme.ElementBG
    Button.Text = defaultOption
    Button.TextColor3 = Theme.Text
    Button.Font = Enum.Font.Gotham
    Button.TextSize = 13
    Button.Parent = Container
    
    local UICorner = Instance.new("UICorner", Button)
    UICorner.CornerRadius = UDim.new(0, 4)

    local currentIndex = 1
    for i, v in ipairs(options) do
        if v == defaultOption then currentIndex = i end
    end

    Button.MouseButton1Click:Connect(function()
        currentIndex = currentIndex + 1
        if currentIndex > #options then currentIndex = 1 end
        local newOption = options[currentIndex]
        Button.Text = newOption
        callback(newOption)
    end)
end

local function CreateSlider(parent, text, min, max, default, callback)
    local Container = Instance.new("Frame")
    Container.Size = UDim2.new(1, 0, 0, 40)
    Container.BackgroundTransparency = 1
    Container.Parent = parent

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.6, 0, 0, 18)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Theme.Text
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Font = Enum.Font.Gotham
    Label.TextSize = 13
    Label.Parent = Container

    local ValueLabel = Instance.new("TextLabel")
    ValueLabel.Size = UDim2.new(0.4, 0, 0, 18)
    ValueLabel.Position = UDim2.new(0.6, 0, 0, 0)
    ValueLabel.BackgroundTransparency = 1
    ValueLabel.Text = tostring(default)
    ValueLabel.TextColor3 = Theme.Text
    ValueLabel.TextXAlignment = Enum.TextXAlignment.Right
    ValueLabel.Font = Enum.Font.Gotham
    ValueLabel.TextSize = 13
    ValueLabel.Parent = Container

    local SliderFrame = Instance.new("Frame")
    SliderFrame.Size = UDim2.new(1, 0, 0, 6)
    SliderFrame.Position = UDim2.new(0, 0, 0, 25)
    SliderFrame.BackgroundColor3 = Theme.ElementBG
    SliderFrame.Parent = Container
    
    local UICorner = Instance.new("UICorner", SliderFrame)
    UICorner.CornerRadius = UDim.new(1, 0)

    local Fill = Instance.new("Frame")
    Fill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    Fill.BackgroundColor3 = Theme.Accent
    Fill.Parent = SliderFrame
    
    local FillCorner = Instance.new("UICorner", Fill)
    FillCorner.CornerRadius = UDim.new(1, 0)

    local dragging = false
    local function updateSlider(input)
        local mousePos = UserInputService:GetMouseLocation()
        local relativeX = math.clamp((mousePos.X - SliderFrame.AbsolutePosition.X) / SliderFrame.AbsoluteSize.X, 0, 1)
        local value = math.floor(min + (max - min) * relativeX)
        Fill.Size = UDim2.new(relativeX, 0, 1, 0)
        ValueLabel.Text = tostring(value)
        callback(value)
    end

    SliderFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true; updateSlider(input) end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then updateSlider(input) end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
end

local MainFrame = Instance.new("Frame")
MainFrame.Name = "Main"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.Position = UDim2.new(0.5, -350, 0.5, -200)
MainFrame.Size = UDim2.new(0, 700, 0, 400)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Visible = true
local MainCorner = Instance.new("UICorner", MainFrame)
MainCorner.CornerRadius = UDim.new(0, 6)

local TitleBar = Instance.new("Frame", MainFrame)
TitleBar.Size = UDim2.new(1, 0, 0, 30)
TitleBar.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
local TitleText = Instance.new("TextLabel", TitleBar)
TitleText.Size = UDim2.new(1, 0, 1, 0)
TitleText.Text = "PAINEL RICKY VIP | F = Aimbot | Insert = Menu"
TitleText.TextColor3 = Theme.Text
TitleText.BackgroundTransparency = 1
TitleText.Font = Enum.Font.GothamBold
TitleText.TextSize = 13

local SideBar = Instance.new("Frame", MainFrame)
SideBar.Size = UDim2.new(0, 120, 1, -30)
SideBar.Position = UDim2.new(0, 0, 0, 30)
SideBar.BackgroundColor3 = Theme.Sidebar

local ContentArea = Instance.new("Frame", MainFrame)
ContentArea.Size = UDim2.new(1, -120, 1, -30)
ContentArea.Position = UDim2.new(0, 120, 0, 30)
ContentArea.BackgroundColor3 = Theme.Background

local AimbotToggleRef = nil

local currentTab = nil
local function SwitchTab(tabName)
    if currentTab == tabName then return end
    currentTab = tabName
    for _, child in ipairs(ContentArea:GetChildren()) do
        if child:IsA("Frame") then child:Destroy() end
    end

    local Col2 = Instance.new("Frame", ContentArea)
    Col2.Size = UDim2.new(0, 280, 1, 0)
    Col2.BackgroundColor3 = Theme.ColumnBG
    local L2 = Instance.new("UIListLayout", Col2); L2.Padding = UDim.new(0, 5)
    local P2 = Instance.new("UIPadding", Col2); P2.PaddingTop = UDim.new(0, 10); P2.PaddingLeft = UDim.new(0, 10); P2.PaddingRight = UDim.new(0, 10)

    local Col3 = Instance.new("Frame", ContentArea)
    Col3.Size = UDim2.new(0, 280, 1, 0)
    Col3.Position = UDim2.new(0, 285, 0, 0)
    Col3.BackgroundColor3 = Theme.ColumnBG
    local L3 = Instance.new("UIListLayout", Col3); L3.Padding = UDim.new(0, 5)
    local P3 = Instance.new("UIPadding", Col3); P3.PaddingTop = UDim.new(0, 10); P3.PaddingLeft = UDim.new(0, 10); P3.PaddingRight = UDim.new(0, 10)

    if tabName == "Combat" then
        local T1 = Instance.new("TextLabel", Col2); T1.Size = UDim2.new(1,0,0,20); T1.Text = "Aimbot Settings"; T1.TextColor3 = Theme.Text; T1.BackgroundTransparency = 1; T1.Font = Enum.Font.GothamBold; T1.TextXAlignment = Enum.TextXAlignment.Left
        AimbotToggleRef = CreateToggle(Col2, "Enable Aimbot [F]", Config.Aimbot_Enabled, function(v) Config.Aimbot_Enabled = v end)
        CreateDropdown(Col2, "Target Part", {"Head", "HumanoidRootPart", "UpperTorso"}, Config.Aimbot_TargetPart, function(v) Config.Aimbot_TargetPart = v end)
        CreateSlider(Col2, "Max Distance", 0, 1000, Config.Aimbot_MaxDistance, function(v) Config.Aimbot_MaxDistance = v end)
        CreateToggle(Col2, "Wall Check", Config.Aimbot_WallCheck, function(v) Config.Aimbot_WallCheck = v end)
        CreateToggle(Col2, "Team Check", Config.Aimbot_TeamCheck, function(v) Config.Aimbot_TeamCheck = v end)
        CreateToggle(Col2, "Smoothing", Config.Aimbot_Smoothing, function(v) Config.Aimbot_Smoothing = v end)
        CreateSlider(Col2, "Smoothness", 1, 20, Config.Aimbot_Smoothness, function(v) Config.Aimbot_Smoothness = v end)
        
        local T2 = Instance.new("TextLabel", Col3); T2.Size = UDim2.new(1,0,0,20); T2.Text = "FOV Circle Settings"; T2.TextColor3 = Theme.Text; T2.BackgroundTransparency = 1; T2.Font = Enum.Font.GothamBold; T2.TextXAlignment = Enum.TextXAlignment.Left
        CreateToggle(Col3, "Show FOV Circle", Config.FOV_Enabled, function(v) Config.FOV_Enabled = v end)
        CreateSlider(Col3, "FOV Radius", 0, 500, Config.FOV_Radius, function(v) Config.FOV_Radius = v end)
        
    elseif tabName == "Visuals" then
        local T1 = Instance.new("TextLabel", Col2); T1.Size = UDim2.new(1,0,0,20); T1.Text = "ESP Settings"; T1.TextColor3 = Theme.Text; T1.BackgroundTransparency = 1; T1.Font = Enum.Font.GothamBold; T1.TextXAlignment = Enum.TextXAlignment.Left
        CreateToggle(Col2, "Enable ESP", Config.ESP_Enabled, function(v) Config.ESP_Enabled = v end)
        CreateToggle(Col2, "ESP Box", Config.ESP_Box, function(v) Config.ESP_Box = v end)
        CreateToggle(Col2, "ESP Name", Config.ESP_Name, function(v) Config.ESP_Name = v end)
        CreateToggle(Col2, "ESP Health", Config.ESP_Health, function(v) Config.ESP_Health = v end)
        CreateDropdown(Col2, "Health Bar Position", {"Top", "Side", "Bottom"}, Config.ESP_HealthPos, function(v) Config.ESP_HealthPos = v end)
        CreateToggle(Col2, "ESP Line", Config.ESP_Line, function(v) Config.ESP_Line = v end)
    end
end

local function CreateSidebarBtn(text, posY, tabName)
    local Btn = Instance.new("TextButton", SideBar)
    Btn.Size = UDim2.new(1, 0, 0, 35)
    Btn.Position = UDim2.new(0, 0, 0, posY)
    Btn.BackgroundColor3 = Theme.Sidebar
    Btn.Text = "  " .. text
    Btn.TextColor3 = Theme.Text
    Btn.Font = Enum.Font.Gotham
    Btn.TextSize = 13
    Btn.TextXAlignment = Enum.TextXAlignment.Left
    Btn.MouseButton1Click:Connect(function() SwitchTab(tabName) end)
end

CreateSidebarBtn("Combat", 0, "Combat")
CreateSidebarBtn("Visuals", 35, "Visuals")
CreateSidebarBtn("Logs", 70, "Logs")
CreateSidebarBtn("UI Settings", 105, "Settings")
SwitchTab("Combat")

local FOVCircle = Instance.new("Frame", ScreenGui)
FOVCircle.BackgroundTransparency = 1
FOVCircle.Visible = false
FOVCircle.ZIndex = 999
local FOVCorner = Instance.new("UICorner", FOVCircle); FOVCorner.CornerRadius = UDim.new(1, 0)
local FOVStroke = Instance.new("UIStroke", FOVCircle); FOVStroke.Thickness = 2; FOVStroke.Color = Config.FOV_Color

RunService.RenderStepped:Connect(function()
    if Config.FOV_Enabled then
        FOVCircle.Visible = true
        local screenCenter = Camera.ViewportSize / 2
        FOVCircle.Size = UDim2.new(0, Config.FOV_Radius * 2, 0, Config.FOV_Radius * 2)
        FOVCircle.Position = UDim2.new(0, screenCenter.X - Config.FOV_Radius, 0, screenCenter.Y - Config.FOV_Radius)
        FOVStroke.Color = Config.FOV_Color
    else
        FOVCircle.Visible = false
    end
end)

local AimbotIndicator = Instance.new("TextLabel", ScreenGui)
AimbotIndicator.Size = UDim2.new(0, 200, 0, 30)
AimbotIndicator.Position = UDim2.new(0.5, -100, 0, 10)
AimbotIndicator.BackgroundTransparency = 1
AimbotIndicator.Text = "AIMBOT: OFF [F]"
AimbotIndicator.TextColor3 = Color3.fromRGB(255, 0, 0)
AimbotIndicator.Font = Enum.Font.GothamBold
AimbotIndicator.TextSize = 16
AimbotIndicator.TextStrokeTransparency = 0

RunService.RenderStepped:Connect(function()
    if Config.Aimbot_Enabled then
        AimbotIndicator.Text = "AIMBOT: ON [F]"
        AimbotIndicator.TextColor3 = Color3.fromRGB(0, 255, 0)
    else
        AimbotIndicator.Text = "AIMBOT: OFF [F]"
        AimbotIndicator.TextColor3 = Color3.fromRGB(255, 0, 0)
    end
end)

-- // LÓGICA DO ESP (CÁLCULO MANUAL CORRIGIDO)
local ESPObjects = {}
local function CreateESP(player)
    if player == LocalPlayer then return end
    local espFolder = Instance.new("Folder", ScreenGui)
    espFolder.Name = player.Name .. "_ESP"
    
    local Box = Instance.new("Frame", espFolder)
    Box.BackgroundTransparency = 1; Box.BorderSizePixel = 0; Box.Visible = false
    local BoxOutline = Instance.new("UIStroke", Box); BoxOutline.Color = Theme.Accent; BoxOutline.Thickness = 1
    
    local NameTag = Instance.new("TextLabel", espFolder)
    NameTag.BackgroundTransparency = 1; NameTag.TextColor3 = Theme.Text; NameTag.TextStrokeTransparency = 0; NameTag.Font = Enum.Font.GothamBold; NameTag.TextSize = 14; NameTag.Visible = false; NameTag.ZIndex = 5
    
    local HealthBar = Instance.new("Frame", espFolder)
    HealthBar.BackgroundColor3 = Color3.fromRGB(0, 255, 0); HealthBar.BorderSizePixel = 0; HealthBar.Visible = false; HealthBar.ZIndex = 4
    
    local Line = Instance.new("Frame", espFolder)
    Line.BackgroundColor3 = Theme.Text; Line.BorderSizePixel = 0; Line.Visible = false; Line.AnchorPoint = Vector2.new(0.5, 1)
    
    ESPObjects[player] = {Box = Box, Name = NameTag, Health = HealthBar, Line = Line}
end

local function RemoveESP(player)
    if ESPObjects[player] then
        ESPObjects[player].Box:Destroy(); ESPObjects[player].Name:Destroy(); ESPObjects[player].Health:Destroy(); ESPObjects[player].Line:Destroy()
        ESPObjects[player] = nil
    end
end

Players.PlayerAdded:Connect(CreateESP)
Players.PlayerRemoving:Connect(RemoveESP)
for _, p in pairs(Players:GetPlayers()) do CreateESP(p) end

RunService.RenderStepped:Connect(function()
    if not Config.ESP_Enabled then
        for _, obj in pairs(ESPObjects) do obj.Box.Visible = false; obj.Name.Visible = false; obj.Health.Visible = false; obj.Line.Visible = false end
        return
    end
    
    for player, esp in pairs(ESPObjects) do
        local char = player.Character
        if char and char:FindFirstChild("Humanoid") and char:FindFirstChild("Head") and char:FindFirstChild("HumanoidRootPart") then
            local humanoid = char.Humanoid
            local head = char.Head
            local hrp = char.HumanoidRootPart
            
            -- PROJEÇÃO MANUAL: Topo da cabeça -> Base dos pés
            -- Pegamos o topo da cabeça (Head.Position + metade do tamanho da cabeça + um pequeno offset)
            local topWorldPos = head.Position + Vector3.new(0, head.Size.Y / 2 + 0.5, 0)
            -- Pegamos a base dos pés (HRP.Position - 3 studs, que é a altura média das pernas do Roblox)
            local bottomWorldPos = hrp.Position - Vector3.new(0, 3, 0)
            
            local topScreen, topOnScreen = Camera:WorldToViewportPoint(topWorldPos)
            local bottomScreen, bottomOnScreen = Camera:WorldToViewportPoint(bottomWorldPos)
            
            if topOnScreen and bottomOnScreen then
                -- Altura real em pixels do personagem inteiro
                local boxHeight = math.abs(bottomScreen.Y - topScreen.Y)
                -- Largura proporcional (Roblox é mais ou menos 50% da altura)
                local boxWidth = boxHeight * 0.5
                
                local boxX = topScreen.X - (boxWidth / 2)
                local boxY = topScreen.Y
                
                -- Desenhar a caixa
                if Config.ESP_Box then
                    esp.Box.Visible = true
                    esp.Box.Size = UDim2.new(0, boxWidth, 0, boxHeight)
                    esp.Box.Position = UDim2.new(0, boxX, 0, boxY)
                else esp.Box.Visible = false end
                
                -- Variável para controlar onde o nome vai ficar
                local nameYPosition = boxY - 5 -- Padrão: acima da box
                
                -- Desenhar a barra de vida
                if Config.ESP_Health then
                    esp.Health.Visible = true
                    local hpPerc = humanoid.Health / humanoid.MaxHealth
                    esp.Health.BackgroundColor3 = Color3.fromRGB(255 * (1-hpPerc), 255 * hpPerc, 0)
                    
                    if Config.ESP_HealthPos == "Top" then
                        esp.Health.Size = UDim2.new(0, boxWidth, 0, 3)
                        esp.Health.Position = UDim2.new(0, boxX, 0, boxY - 5)
                        nameYPosition = boxY - 22 -- Nome vai acima da vida
                    elseif Config.ESP_HealthPos == "Bottom" then
                        esp.Health.Size = UDim2.new(0, boxWidth, 0, 3)
                        esp.Health.Position = UDim2.new(0, boxX, 0, boxY + boxHeight + 3)
                        nameYPosition = boxY - 5 -- Nome fica no padrão
                    elseif Config.ESP_HealthPos == "Side" then
                        -- Barra do lado esquerdo, enchendo de baixo pra cima
                        esp.Health.Size = UDim2.new(0, 3, 0, boxHeight * hpPerc)
                        esp.Health.Position = UDim2.new(0, boxX - 6, 0, boxY + boxHeight - (boxHeight * hpPerc))
                        nameYPosition = boxY - 5 -- Nome fica no padrão
                    end
                else 
                    esp.Health.Visible = false 
                end
                
                -- Desenhar o nome (usa a variável nameYPosition)
                if Config.ESP_Name then
                    esp.Name.Visible = true
                    esp.Name.Text = player.Name
                    esp.Name.Position = UDim2.new(0, topScreen.X, 0, nameYPosition)
                else esp.Name.Visible = false end
                
                -- Desenhar a linha (Tracer)
                if Config.ESP_Line then
                    esp.Line.Visible = true
                    local centerX = Camera.ViewportSize.X / 2
                    local centerY = Camera.ViewportSize.Y
                    local lineLength = math.sqrt((topScreen.X - centerX)^2 + (topScreen.Y - centerY)^2)
                    local angle = math.atan2(topScreen.Y - centerY, topScreen.X - centerX)
                    esp.Line.Size = UDim2.new(0, lineLength, 0, 1)
                    esp.Line.Position = UDim2.new(0, centerX, 0, centerY)
                    esp.Line.Rotation = math.deg(angle)
                else esp.Line.Visible = false end
            end
        end
    end
end)

local function GetClosestPlayer()
    local closest = nil
    local shortestDist = Config.Aimbot_MaxDistance
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            if Config.Aimbot_TeamCheck and player.Team == LocalPlayer.Team then continue end
            local part = player.Character:FindFirstChild(Config.Aimbot_TargetPart)
            if part then
                local dist = (Camera.CFrame.Position - part.Position).Magnitude
                if dist < shortestDist then
                    local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)
                    if onScreen then
                        local screenCenter = Camera.ViewportSize / 2
                        local distToCenter = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
                        if distToCenter <= Config.FOV_Radius then
                            if Config.Aimbot_WallCheck then
                                local ray = Ray.new(Camera.CFrame.Position, part.Position - Camera.CFrame.Position)
                                local hit, pos = Workspace:FindPartOnRay(ray, LocalPlayer.Character)
                                if hit and hit:IsDescendantOf(player.Character) then
                                    shortestDist = dist; closest = player
                                end
                            else
                                shortestDist = dist; closest = player
                            end
                        end
                    end
                end
            end
        end
    end
    return closest
end

RunService.RenderStepped:Connect(function()
    if Config.Aimbot_Enabled == true then
        local target = GetClosestPlayer()
        if target then
            local part = target.Character:FindFirstChild(Config.Aimbot_TargetPart)
            if part then
                local targetPos = part.Position
                local aimCFrame = CFrame.new(Camera.CFrame.Position, targetPos)
                if Config.Aimbot_Smoothing then
                    Camera.CFrame = Camera.CFrame:Lerp(aimCFrame, 1 / Config.Aimbot_Smoothness)
                else
                    Camera.CFrame = aimCFrame
                end
            end
        end
    end
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.Insert then
        MainFrame.Visible = not MainFrame.Visible
    elseif input.KeyCode == Enum.KeyCode.F then
        Config.Aimbot_Enabled = not Config.Aimbot_Enabled
        if AimbotToggleRef and AimbotToggleRef.Button then
            AimbotToggleRef.Button.BackgroundColor3 = Config.Aimbot_Enabled and Theme.Accent or Theme.Inactive
        end
    end
end)

print("Painel Ricky Vip Carregado! Insert = Menu | F = Aimbot")

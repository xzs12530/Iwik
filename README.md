local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- // CONFIGURAÇÕES DE ATUALIZAÇÃO
local Version = "V1.1"
local LastUpdate = "30/01/24"

-- // CONFIGURAÇÕES (Lógica interna mantida)
local Settings = {
    Enabled = false,
    WallCheck = true,
    Smoothing = 0.2,
    FOV = 60,
    AimPart = "Head",
    Hitbox_Enabled = false,
    Hitbox_Size = 2,
    Hitbox_Transparency = 0.5,
    ESP_Master = false,
    ESP_Box = false,
    ESP_BoxTransparency = 0.6,
    ESP_Health = false,
    ESP_Name = false,
    RainbowMode = false
}

local ESP_Objects = {}

-- // TEMA NEVERLOSE
local Theme = {
    Background = Color3.fromRGB(8, 8, 10),
    Sidebar = Color3.fromRGB(10, 10, 12),
    Banner = Color3.fromRGB(12, 12, 15),
    Accent = Color3.fromRGB(0, 170, 255),
    TextMain = Color3.fromRGB(255, 255, 255),
    TextSecondary = Color3.fromRGB(120, 120, 125),
    Stroke = Color3.fromRGB(25, 25, 30),
    GroupBox = Color3.fromRGB(12, 12, 14)
}

-- // UI PRINCIPAL
local ScreenGui = Instance.new("ScreenGui", LocalPlayer.PlayerGui)
ScreenGui.Name = "NeverLose_Mobile"
ScreenGui.IgnoreGuiInset = true

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 550, 0, 400)
MainFrame.Position = UDim2.new(0.5, -275, 0.5, -200)
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.BorderSizePixel = 0
MainFrame.Visible = false

local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Color = Theme.Stroke
MainStroke.Thickness = 1
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 4)

-- // BARRA DE ANÚNCIO (TOP BANNER)
local TopBanner = Instance.new("Frame", MainFrame)
TopBanner.Size = UDim2.new(1, 0, 0, 25)
TopBanner.BackgroundColor3 = Theme.Banner
TopBanner.BorderSizePixel = 0

local BannerStroke = Instance.new("Frame", TopBanner)
BannerStroke.Size = UDim2.new(1, 0, 0, 1)
BannerStroke.BackgroundColor3 = Theme.Accent
BannerStroke.BorderSizePixel = 0

local BannerLeft = Instance.new("TextLabel", TopBanner)
BannerLeft.Size = UDim2.new(0.7, 0, 1, 0)
BannerLeft.Position = UDim2.new(0, 10, 0, 0)
BannerLeft.BackgroundTransparency = 1
BannerLeft.Text = "Neverlose.cc is Loaded anti cheat is broken"
BannerLeft.TextColor3 = Theme.TextSecondary
BannerLeft.Font = Enum.Font.GothamSemibold
BannerLeft.TextSize = 10
BannerLeft.TextXAlignment = Enum.TextXAlignment.Left

local BannerRight = Instance.new("TextLabel", TopBanner)
BannerRight.Size = UDim2.new(0.3, 0, 1, 0)
BannerRight.Position = UDim2.new(0.7, -10, 0, 0)
BannerRight.BackgroundTransparency = 1
BannerRight.Text = "| Better aesthetics"
BannerRight.TextColor3 = Theme.Accent
BannerRight.Font = Enum.Font.GothamBold
BannerRight.TextSize = 10
BannerRight.TextXAlignment = Enum.TextXAlignment.Right

-- // LINHA DE STATUS (FPS E VERSÃO)
local StatusLabel = Instance.new("TextLabel", MainFrame)
StatusLabel.Position = UDim2.new(0, 140, 0, 35)
StatusLabel.Size = UDim2.new(0, 400, 0, 20)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "FPS: 00 | Versão: " .. Version
StatusLabel.TextColor3 = Theme.TextSecondary
StatusLabel.Font = Enum.Font.GothamBold
StatusLabel.TextSize = 10
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left

-- // SIDEBAR
local Sidebar = Instance.new("Frame", MainFrame)
Sidebar.Size = UDim2.new(0, 130, 1, -25)
Sidebar.Position = UDim2.new(0, 0, 0, 25)
Sidebar.BackgroundColor3 = Theme.Sidebar
Sidebar.BorderSizePixel = 0

local SidebarList = Instance.new("UIListLayout", Sidebar)
SidebarList.Padding = UDim.new(0, 2)
SidebarList.HorizontalAlignment = Enum.HorizontalAlignment.Center

local LogoSpace = Instance.new("Frame", Sidebar)
LogoSpace.Size = UDim2.new(1, 0, 0, 45); LogoSpace.BackgroundTransparency = 1
local LogoIcon = Instance.new("TextLabel", LogoSpace)
LogoIcon.Size = UDim2.new(1, 0, 1, 0); LogoIcon.Text = "NEVERLOSE"; LogoIcon.Font = Enum.Font.GothamBlack; LogoIcon.TextSize = 14; LogoIcon.TextColor3 = Theme.TextMain; LogoIcon.BackgroundTransparency = 1

-- // CONTAINER DE CONTEÚDO
local Container = Instance.new("Frame", MainFrame)
Container.Position = UDim2.new(0, 140, 0, 60)
Container.Size = UDim2.new(1, -150, 1, -70)
Container.BackgroundTransparency = 1

-- // FUNÇÕES DE UTILITÁRIO
local function makeDrag(gui)
    local dragging, dragInput, dragStart, startPos
    gui.InputBegan:Connect(function(input)
        if (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then
            dragging = true; dragStart = input.Position; startPos = gui.Position
            input.Changed:Connect(function() if input.UserInputState == Enum.UserInputState.End then dragging = false end end)
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            gui.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

local function createGroupBox(parent, name)
    local box = Instance.new("Frame", parent)
    box.Size = UDim2.new(1, 0, 0, 20)
    box.AutomaticSize = Enum.AutomaticSize.Y
    box.BackgroundColor3 = Theme.GroupBox
    box.BorderSizePixel = 0
    box.ClipsDescendants = false
    
    local stroke = Instance.new("UIStroke", box)
    stroke.Color = Theme.Stroke
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    
    local title = Instance.new("TextLabel", box)
    title.Size = UDim2.new(0, 0, 0, 14)
    title.Position = UDim2.new(0, 10, 0, -8)
    title.BackgroundColor3 = Theme.Background
    title.Text = " " .. name:upper() .. " "
    title.AutomaticSize = Enum.AutomaticSize.X
    title.TextColor3 = Theme.TextMain
    title.Font = Enum.Font.GothamBold
    title.TextSize = 9
    title.ZIndex = 10
    
    local content = Instance.new("Frame", box)
    content.Size = UDim2.new(1, 0, 1, 0)
    content.BackgroundTransparency = 1
    local list = Instance.new("UIListLayout", content)
    list.Padding = UDim.new(0, 8)
    Instance.new("UIPadding", content).PaddingTop = UDim.new(0, 12)
    Instance.new("UIPadding", content).PaddingBottom = UDim.new(0, 10)
    Instance.new("UIPadding", content).PaddingLeft = UDim.new(0, 10)
    Instance.new("UIPadding", content).PaddingRight = UDim.new(0, 10)
    
    return content
end

local Pages = {}
local function createPage(name, icon)
    local pageFrame = Instance.new("Frame", Container)
    pageFrame.Size = UDim2.new(1, 0, 1, 0)
    pageFrame.BackgroundTransparency = 1
    pageFrame.Visible = false
    
    local leftCol = Instance.new("ScrollingFrame", pageFrame)
    leftCol.Size = UDim2.new(0.48, 0, 1, 0)
    leftCol.BackgroundTransparency = 1; leftCol.ScrollBarThickness = 0; leftCol.ClipsDescendants = false
    Instance.new("UIListLayout", leftCol).Padding = UDim.new(0, 25)
    Instance.new("UIPadding", leftCol).PaddingTop = UDim.new(0, 15)
    
    local rightCol = Instance.new("ScrollingFrame", pageFrame)
    rightCol.Position = UDim2.new(0.52, 0, 0, 0)
    rightCol.Size = UDim2.new(0.48, 0, 1, 0)
    rightCol.BackgroundTransparency = 1; rightCol.ScrollBarThickness = 0; rightCol.ClipsDescendants = false
    Instance.new("UIListLayout", rightCol).Padding = UDim.new(0, 25)
    Instance.new("UIPadding", rightCol).PaddingTop = UDim.new(0, 15)
    
    local tabBtn = Instance.new("TextButton", Sidebar)
    tabBtn.Size = UDim2.new(0.9, 0, 0, 30)
    tabBtn.BackgroundTransparency = 1
    tabBtn.Text = "  " .. icon .. "  " .. name
    tabBtn.TextColor3 = Theme.TextSecondary
    tabBtn.Font = Enum.Font.GothamBold
    tabBtn.TextSize = 10
    tabBtn.TextXAlignment = Enum.TextXAlignment.Left
    
    tabBtn.Activated:Connect(function()
        for _, p in pairs(Pages) do p.Visible = false end
        for _, b in pairs(Sidebar:GetChildren()) do if b:IsA("TextButton") then b.TextColor3 = Theme.TextSecondary end end
        pageFrame.Visible = true
        tabBtn.TextColor3 = Theme.Accent
    end)
    
    Pages[name] = pageFrame
    return leftCol, rightCol
end

local function createToggle(parent, text, key)
    local frame = Instance.new("Frame", parent); frame.Size = UDim2.new(1, 0, 0, 18); frame.BackgroundTransparency = 1
    local label = Instance.new("TextLabel", frame); label.Size = UDim2.new(1, -35, 1, 0); label.Text = text; label.TextColor3 = Theme.TextSecondary; label.Font = Enum.Font.Gotham; label.TextSize = 10; label.BackgroundTransparency = 1; label.TextXAlignment = Enum.TextXAlignment.Left
    local btn = Instance.new("TextButton", frame); btn.Size = UDim2.new(0, 24, 0, 12); btn.Position = UDim2.new(1, -25, 0.5, -6); btn.BackgroundColor3 = Settings[key] and Theme.Accent or Color3.fromRGB(30, 30, 35); btn.Text = ""
    Instance.new("UICorner", btn).CornerRadius = UDim.new(1, 0)
    local circle = Instance.new("Frame", btn); circle.Size = UDim2.new(0, 8, 0, 8); circle.Position = Settings[key] and UDim2.new(1, -10, 0.5, -4) or UDim2.new(0, 2, 0.5, -4); circle.BackgroundColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)
    btn.Activated:Connect(function()
        Settings[key] = not Settings[key]
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Settings[key] and Theme.Accent or Color3.fromRGB(30, 30, 35)}):Play()
        TweenService:Create(circle, TweenInfo.new(0.2), {Position = Settings[key] and UDim2.new(1, -10, 0.5, -4) or UDim2.new(0, 2, 0.5, -4)}):Play()
        label.TextColor3 = Settings[key] and Theme.TextMain or Theme.TextSecondary
    end)
end

local function createSlider(parent, text, min, max, key, dec)
    local frame = Instance.new("Frame", parent); frame.Size = UDim2.new(1, 0, 0, 28); frame.BackgroundTransparency = 1
    local label = Instance.new("TextLabel", frame); label.Size = UDim2.new(1, 0, 0, 12); label.Text = text .. ": " .. Settings[key]; label.TextColor3 = Theme.TextSecondary; label.Font = Enum.Font.Gotham; label.TextSize = 9; label.BackgroundTransparency = 1; label.TextXAlignment = Enum.TextXAlignment.Left
    local bar = Instance.new("Frame", frame); bar.Size = UDim2.new(1, 0, 0, 4); bar.Position = UDim2.new(0, 0, 1, -6); bar.BackgroundColor3 = Color3.fromRGB(30, 30, 35); bar.BorderSizePixel = 0; Instance.new("UICorner", bar)
    local fill = Instance.new("Frame", bar); fill.Size = UDim2.new((Settings[key]-min)/(max-min), 0, 1, 0); fill.BackgroundColor3 = Theme.Accent; fill.BorderSizePixel = 0; Instance.new("UICorner", fill)
    local function update(input)
        local pos = math.clamp((input.Position.X - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1)
        local val = min + (max - min) * pos
        val = dec and math.floor(val * 10) / 10 or math.floor(val)
        Settings[key] = val; label.Text = text .. ": " .. val; fill.Size = UDim2.new(pos, 0, 1, 0)
    end
    bar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            local conn; conn = UserInputService.InputChanged:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then update(i) end end)
            UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then conn:Disconnect() end end)
            update(input)
        end
    end)
end

-- // MONTAGEM DAS PÁGINAS
local AimLeft, AimRight = createPage("Aimbot", "🎯")
local GroupAim = createGroupBox(AimLeft, "Aim")
createToggle(GroupAim, "Enable Aimbot", "Enabled")
createToggle(GroupAim, "Wall Check", "WallCheck")
local GroupAimSettings = createGroupBox(AimRight, "Settings")
createSlider(GroupAimSettings, "Smoothness", 0.1, 1, "Smoothing", true)
createSlider(GroupAimSettings, "FOV Radius", 10, 300, "FOV", false)

local HitLeft, HitRight = createPage("Hitbox", "📦")
local GroupHit = createGroupBox(HitLeft, "Hitbox")
createToggle(GroupHit, "Enable Hitbox", "Hitbox_Enabled")
createSlider(GroupHit, "Head Size", 1, 35, "Hitbox_Size", false)
createSlider(GroupHit, "Transparency", 0, 1, "Hitbox_Transparency", true)

local VisLeft, VisRight = createPage("Visuals", "👁️")
local GroupEsp = createGroupBox(VisLeft, "ESP")
createToggle(GroupEsp, "Master ESP", "ESP_Master")
createToggle(GroupEsp, "Boxes", "ESP_Box")
createToggle(GroupEsp, "Player Names", "ESP_Name")
createToggle(GroupEsp, "Health Bar", "ESP_Health")
local GroupEspConfig = createGroupBox(VisRight, "Settings")
createSlider(GroupEspConfig, "Box Transparency", 0, 1, "ESP_BoxTransparency", true)

local SetLeft, SetRight = createPage("Settings", "⚙️")
local GroupMisc = createGroupBox(SetLeft, "Misc")
createToggle(GroupMisc, "Rainbow UI", "RainbowMode")

-- // BOTÃO OPEN (N)
local OpenBtn = Instance.new("TextButton", ScreenGui); OpenBtn.Size = UDim2.new(0, 35, 0, 35); OpenBtn.Position = UDim2.new(0.02, 0, 0.8, 0); OpenBtn.BackgroundColor3 = Theme.Sidebar; OpenBtn.Text = "N"; OpenBtn.TextColor3 = Theme.Accent; OpenBtn.Font = Enum.Font.GothamBlack; OpenBtn.BorderSizePixel = 0; Instance.new("UICorner", OpenBtn); Instance.new("UIStroke", OpenBtn).Color = Theme.Stroke
OpenBtn.Activated:Connect(function() MainFrame.Visible = not MainFrame.Visible end)
makeDrag(MainFrame); makeDrag(OpenBtn)

-- // LOOP CORE
local FOVCircle = Instance.new("Frame", ScreenGui); FOVCircle.AnchorPoint = Vector2.new(0.5, 0.5); FOVCircle.Position = UDim2.new(0.5, 0, 0.5, 0); FOVCircle.BackgroundTransparency = 1; local CircleStroke = Instance.new("UIStroke", FOVCircle); CircleStroke.Color = Theme.Accent; Instance.new("UICorner", FOVCircle).CornerRadius = UDim.new(1, 0)

RunService.RenderStepped:Connect(function()
    -- Atualizar FPS e Status
    StatusLabel.Text = string.format("FPS: %d | Versão: %s", 1/RunService.RenderStepped:Wait(), Version)

    local dynamicColor = Settings.RainbowMode and Color3.fromHSV(tick() % 4 / 4, 1, 1) or Theme.Accent
    BannerStroke.BackgroundColor3 = dynamicColor; OpenBtn.TextColor3 = dynamicColor; CircleStroke.Color = dynamicColor
    FOVCircle.Visible = Settings.Enabled; FOVCircle.Size = UDim2.new(0, Settings.FOV * 2, 0, Settings.FOV * 2)

    local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    local target = nil; local dist = Settings.FOV

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr == LocalPlayer then continue end
        local char = plr.Character; local obj = ESP_Objects[plr]
        if char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Head") and char.Humanoid.Health > 0 then
            local head = char.Head
            local hPos, onS = Camera:WorldToViewportPoint(head.Position)
            local isVisible = #Camera:GetPartsObscuringTarget({Camera.CFrame.Position, head.Position}, {LocalPlayer.Character, char}) == 0
            local espColor = isVisible and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)
            if Settings.Hitbox_Enabled then
                head.Size = Vector3.new(Settings.Hitbox_Size, Settings.Hitbox_Size, Settings.Hitbox_Size)
                head.Transparency = Settings.Hitbox_Transparency; head.CanCollide = false
            else
                head.Size = Vector3.new(1.1, 1.1, 1.1); head.Transparency = 0
            end
            if Settings.ESP_Master and onS then
                if not obj then
                    obj = {}
                    obj.Box = Instance.new("Frame", ScreenGui); obj.Box.BorderSizePixel = 0
                    obj.Stroke = Instance.new("UIStroke", obj.Box); obj.Stroke.Thickness = 1.5
                    obj.Name = Instance.new("TextLabel", ScreenGui); obj.Name.BackgroundTransparency = 1; obj.Name.Font = Enum.Font.GothamBold; obj.Name.TextSize = 10; obj.Name.TextColor3 = Color3.new(1,1,1)
                    obj.Health = Instance.new("Frame", ScreenGui); obj.Health.BorderSizePixel = 0
                    ESP_Objects[plr] = obj
                end
                local rootPos = Camera:WorldToViewportPoint(char.HumanoidRootPart.Position)
                local footPos = Camera:WorldToViewportPoint(char.HumanoidRootPart.Position - Vector3.new(0, 3.5, 0))
                local h = math.abs(rootPos.Y - footPos.Y) * 2.2; local w = h * 0.6
                obj.Box.Visible = Settings.ESP_Box; obj.Box.Size = UDim2.new(0, w, 0, h); obj.Box.Position = UDim2.new(0, rootPos.X - w/2, 0, rootPos.Y - h/2)
                obj.Box.BackgroundColor3 = espColor; obj.Box.BackgroundTransparency = Settings.ESP_BoxTransparency; obj.Stroke.Color = espColor
                obj.Name.Visible = Settings.ESP_Name; obj.Name.Text = plr.Name; obj.Name.Position = UDim2.new(0, rootPos.X - 50, 0, rootPos.Y - h/2 - 15); obj.Name.Size = UDim2.new(0, 100, 0, 15)
                local hp = char.Humanoid.Health / char.Humanoid.MaxHealth
                obj.Health.Visible = Settings.ESP_Health; obj.Health.Size = UDim2.new(0, 2, h * hp, 0); obj.Health.Position = UDim2.new(0, rootPos.X - w/2 - 5, 0, (rootPos.Y - h/2) + (h * (1-hp))); obj.Health.BackgroundColor3 = Color3.new(0,1,0)
            elseif obj then obj.Box.Visible = false; obj.Name.Visible = false; obj.Health.Visible = false end
            if Settings.Enabled and onS then
                local mag = (Vector2.new(hPos.X, hPos.Y) - center).Magnitude
                if mag < dist then
                    if not Settings.WallCheck or isVisible then
                        dist = mag; target = char:FindFirstChild(Settings.AimPart) or head
                    end
                end
            end
        elseif obj then obj.Box.Visible = false; obj.Name.Visible = false; obj.Health.Visible = false end
    end
    if target then Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, target.Position), Settings.Smoothing) end
end)

-- Ativar página inicial
Pages["Aimbot"].Visible = true
Sidebar:FindFirstChildWhichIsA("TextButton").TextColor3 = Theme.Accent

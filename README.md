-- // INICIALIZAÇÃO ESTÁVEL
repeat task.wait() until game:IsLoaded()
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Camera = workspace.CurrentCamera

-- Limpeza de duplicados
if PlayerGui:FindFirstChild("NeverLose_Official") then PlayerGui.NeverLose_Official:Destroy() end

-- // CONFIGURAÇÕES
local Version = "V1.4.2 (Multi-Skin & Inertia)"
local VioletPBR = {
    ColorMap = "rbxassetid://116943159478011",
    MetalnessMap = "rbxassetid://129989332071918",
    NormalMap = "rbxassetid://73004138504914",
    RoughnessMap = "rbxassetid://128918610096381"
}

local Settings = {
    Enabled = false,
    WallCheck = true,
    Smoothing = 0.5, 
    FOV = 80,
    AimPart = "Head",
    -- Hitbox
    Hitbox_Enabled = false,
    Hitbox_Size = 2,
    Hitbox_Transparency = 0.5,
    -- ESP
    ESP_Master = false,
    ESP_Box = false,
    ESP_BoxTransparency = 0.5,
    ESP_Health = false,
    ESP_Name = false,
    EnemyOnly = false,
    -- Weapon & Skin System
    NoRecoil = false,
    NoSpread = false,
    DefaultKnifeName = "Knife",
    ButterflyChanger = false,
    KarambitChanger = false,
    M9Changer = false,
    -- Misc
    RainbowMode = false
}

local ESP_Objects = {}
local WeaponsFolder = game:GetService("ReplicatedStorage"):FindFirstChild("Weapons", true) or workspace:FindFirstChild("Weapons", true)

-- // FUNÇÕES TÉCNICAS (CONSERVAÇÃO DA SKIN)
local function FinalizeVioletEclipse(obj)
    pcall(function()
        for _, part in pairs(obj:GetDescendants()) do
            if part:IsA("MeshPart") then
                local pbr = part:FindFirstChild("NewPBR") or Instance.new("SurfaceAppearance", part)
                pbr.Name = "NewPBR"; pbr.ColorMap = VioletPBR.ColorMap; pbr.MetalnessMap = VioletPBR.MetalnessMap; pbr.NormalMap = VioletPBR.NormalMap; pbr.RoughnessMap = VioletPBR.RoughnessMap
                part.Material = Enum.Material.Neon; part.TextureID = ""; part.Color = Color3.new(1, 1, 1); part.Transparency = 0
            end
        end
    end)
end

-- Função Mestre para aplicar qualquer faca com a skin Violet
local function ApplyUniversalSkin(sourceWeaponName, targetID)
    pcall(function()
        if not WeaponsFolder then WeaponsFolder = game:GetService("ReplicatedStorage"):FindFirstChild("Weapons", true) end
        
        local Places = {WeaponsFolder, LocalPlayer.Backpack, LocalPlayer.Character}
        local Def, Source
        
        for _, p in pairs(Places) do
            if not Def then Def = p:FindFirstChild(Settings.DefaultKnifeName) end
            if not Source then Source = p:FindFirstChild(sourceWeaponName) end
        end

        if Def and Source then
            if Def:FindFirstChild("Model") then Def.Model:Destroy() end
            if Def:FindFirstChild("Viewmodel") then Def.Viewmodel:Destroy() end
            
            local m, v = Source.Model:Clone(), Source.Viewmodel:Clone()
            FinalizeVioletEclipse(m); FinalizeVioletEclipse(v)
            
            m.Parent = Def; v.Parent = Def
            Def:SetAttribute("ID", targetID)
            print(sourceWeaponName .. " Violet Applied")
        end
    end)
end

-- // UI SETUP
local Theme = {
    Background = Color3.fromRGB(8, 8, 10), Sidebar = Color3.fromRGB(10, 10, 12), Accent = Color3.fromRGB(0, 170, 255),
    CT_Color = Color3.fromRGB(0, 120, 255), T_Color = Color3.fromRGB(255, 50, 50), Stroke = Color3.fromRGB(25, 25, 30), GroupBox = Color3.fromRGB(12, 12, 14)
}

local ScreenGui = Instance.new("ScreenGui", PlayerGui); ScreenGui.Name = "NeverLose_Official"; ScreenGui.IgnoreGuiInset = true; ScreenGui.ResetOnSpawn = false
local MainFrame = Instance.new("Frame", ScreenGui); MainFrame.Size = UDim2.new(0, 500, 0, 360); MainFrame.Position = UDim2.new(0.5, -250, 0.5, -180); MainFrame.BackgroundColor3 = Theme.Background; MainFrame.BorderSizePixel = 0; MainFrame.Visible = false
Instance.new("UIStroke", MainFrame).Color = Theme.Stroke; Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 4)

local TopBanner = Instance.new("Frame", MainFrame); TopBanner.Size = UDim2.new(1, 0, 0, 25); TopBanner.BackgroundColor3 = Color3.fromRGB(12, 12, 15); local BannerStroke = Instance.new("Frame", TopBanner); BannerStroke.Size = UDim2.new(1, 0, 0, 1); BannerStroke.BackgroundColor3 = Theme.Accent
local StatusLabel = Instance.new("TextLabel", MainFrame); StatusLabel.Position = UDim2.new(0, 160, 0, 35); StatusLabel.Size = UDim2.new(0, 300, 0, 20); StatusLabel.BackgroundTransparency = 1; StatusLabel.TextColor3 = Color3.fromRGB(120,120,125); StatusLabel.Font = Enum.Font.GothamBold; StatusLabel.TextSize = 10; StatusLabel.TextXAlignment = Enum.TextXAlignment.Left

local Sidebar = Instance.new("Frame", MainFrame); Sidebar.Size = UDim2.new(0, 145, 1, -25); Sidebar.Position = UDim2.new(0, 0, 0, 25); Sidebar.BackgroundColor3 = Theme.Sidebar; Sidebar.BorderSizePixel = 0
Instance.new("UIListLayout", Sidebar).HorizontalAlignment = Enum.HorizontalAlignment.Center; local LogoIcon = Instance.new("TextLabel", Sidebar); LogoIcon.Size = UDim2.new(1, 0, 0, 50); LogoIcon.Text = "NEVERLOSE"; LogoIcon.Font = Enum.Font.GothamBlack; LogoIcon.TextSize = 14; LogoIcon.TextColor3 = Color3.new(1,1,1); LogoIcon.BackgroundTransparency = 1
local Container = Instance.new("Frame", MainFrame); Container.Position = UDim2.new(0, 155, 0, 60); Container.Size = UDim2.new(1, -165, 1, -75); Container.BackgroundTransparency = 1
local OpenBtn = Instance.new("TextButton", ScreenGui); OpenBtn.Size = UDim2.new(0, 40, 0, 40); OpenBtn.Position = UDim2.new(0.02, 0, 0.8, 0); OpenBtn.BackgroundColor3 = Theme.Sidebar; OpenBtn.Text = "N"; OpenBtn.TextColor3 = Theme.Accent; OpenBtn.Font = Enum.Font.GothamBlack; Instance.new("UICorner", OpenBtn); Instance.new("UIStroke", OpenBtn).Color = Theme.Stroke; OpenBtn.Activated:Connect(function() MainFrame.Visible = not MainFrame.Visible end)

-- // BUILDERS
local function makeDrag(gui)
    local dragging, dragStart, startPos
    gui.InputBegan:Connect(function(input) if (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then dragging = true; dragStart = input.Position; startPos = gui.Position end end)
    UserInputService.InputChanged:Connect(function(input) if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local delta = input.Position - dragStart; gui.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end)
    UserInputService.InputEnded:Connect(function() dragging = false end)
end
makeDrag(MainFrame); makeDrag(OpenBtn)

local function createGroupBox(parent, name)
    local box = Instance.new("Frame", parent); box.Size = UDim2.new(1, 0, 0, 20); box.AutomaticSize = Enum.AutomaticSize.Y; box.BackgroundColor3 = Theme.GroupBox; box.BorderSizePixel = 0; box.ClipsDescendants = false; Instance.new("UIStroke", box).Color = Theme.Stroke; local title = Instance.new("TextLabel", box); title.Size = UDim2.new(0, 0, 0, 14); title.Position = UDim2.new(0, 10, 0, -8); title.BackgroundColor3 = Theme.Background; title.Text = " " .. name:upper() .. " "; title.AutomaticSize = Enum.AutomaticSize.X; title.TextColor3 = Color3.new(1,1,1); title.Font = Enum.Font.GothamBold; title.TextSize = 9; title.ZIndex = 11
    local content = Instance.new("Frame", box); content.Size = UDim2.new(1, 0, 1, 0); content.BackgroundTransparency = 1; Instance.new("UIListLayout", content).Padding = UDim.new(0, 8); Instance.new("UIPadding", content).PaddingTop = UDim.new(0, 12); Instance.new("UIPadding", content).PaddingBottom = UDim.new(0, 10); Instance.new("UIPadding", content).PaddingLeft = UDim.new(0, 10); Instance.new("UIPadding", content).PaddingRight = UDim.new(0, 10); return content
end

local Pages = {}
local function createPage(name, icon)
    local pf = Instance.new("Frame", Container); pf.Size = UDim2.new(1, 0, 1, 0); pf.BackgroundTransparency = 1; pf.Visible = false
    local L = Instance.new("ScrollingFrame", pf); L.Size = UDim2.new(0.48, 0, 1, 0); L.BackgroundTransparency = 1; L.ScrollBarThickness = 0; L.ClipsDescendants = false; Instance.new("UIListLayout", L).Padding = UDim.new(0, 25); Instance.new("UIPadding", L).PaddingTop = UDim.new(0, 15)
    local R = Instance.new("ScrollingFrame", pf); R.Position = UDim2.new(0.52, 0, 0, 0); R.Size = UDim2.new(0.48, 0, 1, 0); R.BackgroundTransparency = 1; R.ScrollBarThickness = 0; R.ClipsDescendants = false; Instance.new("UIListLayout", R).Padding = UDim.new(0, 25); Instance.new("UIPadding", R).PaddingTop = UDim.new(0, 15)
    local btn = Instance.new("TextButton", Sidebar); btn.Size = UDim2.new(0.9, 0, 0, 35); btn.BackgroundTransparency = 1; btn.Text = "  " .. icon .. "  " .. name; btn.TextColor3 = Color3.fromRGB(120,120,125); btn.Font = Enum.Font.GothamBold; btn.TextSize = 10; btn.TextXAlignment = Enum.TextXAlignment.Left; btn.Activated:Connect(function() for _, page in pairs(Pages) do page.Visible = false end pf.Visible = true; btn.TextColor3 = Theme.Accent end)
    Pages[name] = pf; return L, R
end

local function createToggle(parent, text, key, callback)
    local f = Instance.new("Frame", parent); f.Size = UDim2.new(1, 0, 0, 20); f.BackgroundTransparency = 1; local l = Instance.new("TextLabel", f); l.Size = UDim2.new(1, -40, 1, 0); l.Text = text; l.TextColor3 = Color3.fromRGB(120,120,125); l.Font = Enum.Font.Gotham; l.TextSize = 10; l.BackgroundTransparency = 1; l.TextXAlignment = Enum.TextXAlignment.Left
    local b = Instance.new("TextButton", f); b.Size = UDim2.new(0, 30, 0, 15); b.Position = UDim2.new(1, -30, 0.5, -7.5); b.BackgroundColor3 = Settings[key] and Theme.Accent or Color3.fromRGB(30, 30, 35); b.Text = ""; Instance.new("UICorner", b).CornerRadius = UDim.new(1, 0); local c = Instance.new("Frame", b); c.Size = UDim2.new(0, 10, 0, 10); c.Position = Settings[key] and UDim2.new(1, -12, 0.5, -5) or UDim2.new(0, 2, 0.5, -5); c.BackgroundColor3 = Color3.new(1,1,1); Instance.new("UICorner", c).CornerRadius = UDim.new(1, 0)
    b.Activated:Connect(function() Settings[key] = not Settings[key]; TweenService:Create(b, TweenInfo.new(0.15), {BackgroundColor3 = Settings[key] and Theme.Accent or Color3.fromRGB(30,30,35)}):Play(); TweenService:Create(c, TweenInfo.new(0.15), {Position = Settings[key] and UDim2.new(1, -12, 0.5, -5) or UDim2.new(0, 2, 0.5, -5)}):Play(); if callback then callback(Settings[key]) end end)
end

local function createSlider(parent, text, min, max, key, isFloat)
    local f = Instance.new("Frame", parent); f.Size = UDim2.new(1, 0, 0, 30); f.BackgroundTransparency = 1; local l = Instance.new("TextLabel", f); l.Size = UDim2.new(1, 0, 0, 14); l.Text = text .. ": " .. Settings[key]; l.TextColor3 = Color3.fromRGB(120,120,125); l.Font = Enum.Font.Gotham; l.TextSize = 10; l.BackgroundTransparency = 1; l.TextXAlignment = Enum.TextXAlignment.Left
    local b = Instance.new("Frame", f); b.Size = UDim2.new(1, 0, 0, 5); b.Position = UDim2.new(0, 0, 1, -7); b.BackgroundColor3 = Color3.fromRGB(30, 30, 35); Instance.new("UICorner", b); local fill = Instance.new("Frame", b); fill.Size = UDim2.new((Settings[key]-min)/(max-min), 0, 1, 0); fill.BackgroundColor3 = Theme.Accent; Instance.new("UICorner", fill)
    local function update(input) local pos = math.clamp((input.Position.X - b.AbsolutePosition.X) / b.AbsoluteSize.X, 0, 1); local val = min + (max - min) * pos; if isFloat then val = math.floor(val * 10) / 10 else val = math.floor(val) end Settings[key] = val; l.Text = text .. ": " .. val; fill.Size = UDim2.new(pos, 0, 1, 0) end
    b.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then local conn; conn = UserInputService.InputChanged:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then update(i) end end) UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then conn:Disconnect() end end) update(input) end end)
end

-- // MONTAGEM
local AL, AR = createPage("Aimbot", "🎯"); createToggle(createGroupBox(AL, "Aim"), "Enable Aimbot", "Enabled"); createToggle(createGroupBox(AL, "Aim"), "Wall Check", "WallCheck"); createSlider(createGroupBox(AR, "Settings"), "Smoothness", 0.1, 1, "Smoothing", true); createSlider(createGroupBox(AR, "Settings"), "FOV Radius", 10, 400, "FOV", false)
local HL, HR = createPage("Hitbox", "📦"); createToggle(createGroupBox(HL, "Main"), "Enable Hitbox", "Hitbox_Enabled"); createSlider(createGroupBox(HL, "Main"), "Size", 1, 40, "Hitbox_Size", false); createSlider(createGroupBox(HR, "Visuals"), "Transparency", 0.1, 1, "Hitbox_Transparency", true)
local VL, VR = createPage("Visuals", "👁️"); local G_ESP = createGroupBox(VL, "ESP Main"); createToggle(G_ESP, "Master ESP", "ESP_Master"); createToggle(G_ESP, "Enemy Only", "EnemyOnly"); createToggle(G_ESP, "Boxes", "ESP_Box"); createToggle(G_ESP, "Names", "ESP_Name"); createToggle(G_ESP, "Health", "ESP_Health"); createSlider(createGroupBox(VR, "Settings"), "Box Transparency", 0.1, 1, "ESP_BoxTransparency", true)

-- ABA WEAPON COM KARAMBIT E M9 BAYONET
local WL, WR = createPage("Weapon", "🔫")
local G_Weapon = createGroupBox(WL, "Mods")
createToggle(G_Weapon, "No Recoil", "NoRecoil")
createToggle(G_Weapon, "No Spread", "NoSpread")

local G_Skins = createGroupBox(WR, "Violet Skins")
createToggle(G_Skins, "Butterfly Knife", "ButterflyChanger", function(s) if s then ApplyUniversalSkin("Butterfly Knife", 120) end end)
createToggle(G_Skins, "Karambit", "KarambitChanger", function(s) if s then ApplyUniversalSkin("Karambit", 110) end end)
createToggle(G_Skins, "M9 Bayonet", "M9Changer", function(s) if s then ApplyUniversalSkin("M9 Bayonet", 130) end end)

createToggle(createGroupBox(createPage("Settings", "⚙️"), "Misc"), "Rainbow UI", "RainbowMode")

-- // FOV CIRCLE
local FOVCircle = Instance.new("Frame", ScreenGui); FOVCircle.AnchorPoint = Vector2.new(0.5, 0.5); FOVCircle.Position = UDim2.new(0.5, 0, 0.5, 0); FOVCircle.BackgroundTransparency = 1; local FOVSStr = Instance.new("UIStroke", FOVCircle); FOVSStr.Color = Theme.Accent; Instance.new("UICorner", FOVCircle).CornerRadius = UDim.new(1, 0)

-- // LOOP CORE (HITBOX & SKIN CONSISTENCY)
RunService.RenderStepped:Connect(function(delta)
    StatusLabel.Text = "FPS: " .. math.floor(1/delta) .. " | " .. Version
    local dc = Settings.RainbowMode and Color3.fromHSV(tick() % 4 / 4, 1, 1) or Theme.Accent
    BannerStroke.BackgroundColor3 = dc; OpenBtn.TextColor3 = dc; FOVSStr.Color = dc
    FOVCircle.Visible = Settings.Enabled; FOVCircle.Size = UDim2.new(0, Settings.FOV * 2, 0, Settings.FOV * 2)

    -- Weapon Mod Heartbeat
    pcall(function()
        if WeaponsFolder then
            for _, w in ipairs(WeaponsFolder:GetChildren()) do
                if Settings.NoRecoil then w:SetAttribute("RecoilX", 0) w:SetAttribute("RecoilY", 0) end
                if Settings.NoSpread then for _, s in pairs({"Spread", "StandSpread", "MoveSpread"}) do w:SetAttribute(s, 0) end end
            end
        end
    end)

    -- Skin Persistence (Garante que a cor Violet Eclipse não suma ao trocar de arma)
    if Settings.ButterflyChanger or Settings.KarambitChanger or Settings.M9Changer then
        for _, v in pairs(Camera:GetChildren()) do
            if v:IsA("Model") and (v.Name:lower():find("knife") or v.Name:lower():find("view")) then
                FinalizeVioletEclipse(v)
            end
        end
    end

    local myT = LocalPlayer:GetAttribute("Team"); local target, minDist = nil, Settings.FOV
    local screenCenter = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr == LocalPlayer then continue end
        local t = plr:GetAttribute("Team"); local isEnemy = (t ~= myT) or (t == nil)
        local char = plr.Character; local obj = ESP_Objects[plr]

        if char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Head") and char.Humanoid.Health > 0 then
            local head = char.Head; local root = char.HumanoidRootPart
            
            -- LÓGICA DE HITBOX PERSISTENTE
            if Settings.Hitbox_Enabled and isEnemy then
                head.Size = Vector3.new(Settings.Hitbox_Size, Settings.Hitbox_Size, Settings.Hitbox_Size)
                head.Transparency = Settings.Hitbox_Transparency
                head.CanCollide = false
            else
                head.Size = Vector3.new(1.1, 1.1, 1.1)
                head.Transparency = 0
            end

            if Settings.EnemyOnly and not isEnemy then if obj then obj.Box.Visible = false; obj.Name.Visible = false end continue end

            local headV, onS = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 1.5, 0))
            local footV = Camera:WorldToViewportPoint(root.Position - Vector3.new(0, 3.5, 0))
            
            if onS then
                if not obj then
                    obj = { Box = Instance.new("Frame", ScreenGui), Name = Instance.new("TextLabel", ScreenGui), Health = Instance.new("Frame", ScreenGui) }
                    obj.Box.BackgroundTransparency = 1; Instance.new("UIStroke", obj.Box).Thickness = 1.6
                    obj.Name.BackgroundTransparency = 1; obj.Name.Font = Enum.Font.GothamBold; obj.Name.TextSize = 10; obj.Health.BorderSizePixel = 0; ESP_Objects[plr] = obj
                end
                local boxH = math.abs(headV.Y - footV.Y); local boxW = boxH * 0.6
                local isVisible = #Camera:GetPartsObscuringTarget({Camera.CFrame.Position, head.Position}, {LocalPlayer.Character, char}) == 0
                local tc = (t == "CT") and Theme.CT_Color or Theme.T_Color
                
                obj.Box.Visible = Settings.ESP_Box and Settings.ESP_Master
                obj.Box.Size = UDim2.new(0, boxW, 0, boxH); obj.Box.Position = UDim2.new(0, headV.X - boxW/2, 0, headV.Y)
                obj.Box:FindFirstChildWhichIsA("UIStroke").Color = isVisible and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)
                obj.Box.BackgroundColor3 = tc; obj.Box.BackgroundTransparency = Settings.ESP_BoxTransparency
                obj.Name.Visible = Settings.ESP_Name and Settings.ESP_Master; obj.Name.Text = plr.Name:upper(); obj.Name.TextColor3 = tc; obj.Name.Position = UDim2.new(0, headV.X - 50, 0, headV.Y - 18); obj.Name.Size = UDim2.new(0, 100, 0, 15)
                obj.Health.Visible = Settings.ESP_Health and Settings.ESP_Master; local hp = char.Humanoid.Health/100; obj.Health.Size = UDim2.new(0, 2, boxH * hp, 0); obj.Health.Position = UDim2.new(0, headV.X - boxW/2 - 5, 0, headV.Y + (boxH * (1-hp))); obj.Health.BackgroundColor3 = Color3.new(1,0,0):Lerp(Color3.new(0,1,0), hp)

                if Settings.Enabled and isEnemy then
                    local mPos = Vector2.new(headV.X, headV.Y + (boxH/2))
                    local dist = (mPos - screenCenter).Magnitude
                    if dist < minDist and (not Settings.WallCheck or isVisible) then minDist = dist; target = head end
                end
            elseif obj then obj.Box.Visible = false; obj.Name.Visible = false end
        elseif obj then obj.Box.Visible = false; obj.Name.Visible = false end
    end
    if target and Settings.Enabled then
        Camera.CFrame = Camera.CFrame:Lerp(CFrame.lookAt(Camera.CFrame.Position, target.Position), Settings.Smoothing)
    end
end)

Pages["Aimbot"].Visible = true; MainFrame.Visible = true; Sidebar:FindFirstChildWhichIsA("TextButton").TextColor3 = Theme.Accent

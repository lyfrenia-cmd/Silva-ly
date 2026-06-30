--[[
    COMLOCK SILVA - ROXO TEMA (EXTREME UI OVERHAUL + INSTANT AIMBOT)
    - Tema: Cyberpunk Glassmorphism (Preto Fosco / Roxo Neon)
    - Animações Fluidas (Quint/Back)
    - Foco Total: Aimbot Instantâneo (0 Suavidade)
    - FOV Expandido: 400
    - Otimização: NPC Cache System (Sem Lag no RenderStepped)
]]

if not game:IsLoaded() then game.Loaded:Wait() end

task.spawn(function()
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local CoreGui = game:GetService("CoreGui")
    local UserInputService = game:GetService("UserInputService")
    local Workspace = game:GetService("Workspace")
    local Debris = game:GetService("Debris")
    local TweenService = game:GetService("TweenService")
    
    local LocalPlayer = Players.LocalPlayer
    while not LocalPlayer do Players.PlayerAdded:Wait(); LocalPlayer = Players.LocalPlayer end
    
    local Camera = Workspace.CurrentCamera
    local Mouse = LocalPlayer:GetMouse()

    -- 🎨 CORES DO TEMA AVANÇADO (GLASSMORPHISM ROXO)
    local THEME_PRIMARY = Color3.fromRGB(155, 89, 182) -- Roxo Claro/Ametista
    local THEME_PRIMARY_DARK = Color3.fromRGB(113, 54, 138) -- Roxo Escuro
    local THEME_BG = Color3.fromRGB(8, 8, 12)
    local THEME_TEXT = Color3.fromRGB(240, 240, 240)
    local THEME_TEXT_MUTED = Color3.fromRGB(150, 150, 160)
    local UI_PANEL = Color3.fromRGB(15, 15, 20)
    local UI_BUTTON = Color3.fromRGB(22, 22, 28)
    local UI_TOGGLE_OFF = Color3.fromRGB(40, 40, 50)
    local GLASS_TRANSPARENCY = 0.15

    -- Cores ESP
    local COLOR_LOCKED = Color3.fromRGB(190, 41, 236)
    local COLOR_VISIBLE = Color3.fromRGB(155, 89, 182)
    local COLOR_HIDDEN = Color3.fromRGB(80, 20, 100)

    -- CONFIGURAÇÕES
    local Settings = {
        Combat = {
            Enabled = false, CamLock = false, FreeCombat = false, Mode = "Dynamic", 
            AimPart = "HumanoidRootPart", AimAtBack = false, BackOffset = 3.5, 
            WallCheck = false, TargetNPCs = true, MaxDistance = 9e9, FOV = 400, 
            ShowFOV = false, Triggerbot = false, TriggerKey = Enum.UserInputType.MouseButton1, 
            TriggerDelay = 0.1, AntiAim = false, AntiAimAngle = 30, AutoDodge = false, 
            DodgeRange = 15, DodgeCooldown = 0.5, DodgeStyle = "Tween", DodgeDirection = "Back"
        },
        Movement = {
            SpiderEnabled = false, InfJump = false, Noclip = false, SpeedHackEnabled = false, 
            NormalSpeed = 50, ClimbSpeed = 40, SprintMult = 1.5, WallJumpForce = 110, 
            RayLength = 6.0, InfJumpForce = 50
        },
        Visuals = {ESPEnabled = true, ESPShowWeapon = true, FPSBoost = false, ESPBox = true, ESPTracer = true},
        Players = {SelectedPlayer = nil, Spectating = false}
    }

    local LockedTarget, isClimbing, isSprinting = nil, false, false
    local canDodge, lastJumpTime, lastShotTime = true, 0, 0
    local character, humanoid, rootPart, animator, bodyVel, bodyGyro, climbTrack
    local AnimIds = { R15 = "rbxassetid://507765644", R6 = "rbxassetid://180436334" }

    -- Limpeza de versões anteriores
    pcall(function()
        for _, v in pairs(CoreGui:GetChildren()) do
            if string.match(v.Name, "AdminNillo") or string.match(v.Name, "ComlockSilva") or v.Name == "Nillo_ESP" or v.Name == "Nillo_ESP2D" then v:Destroy() end
        end
        for _, v in pairs(LocalPlayer.PlayerGui:GetChildren()) do
             if string.match(v.Name, "AdminNillo") or string.match(v.Name, "ComlockSilva") or v.Name == "Nillo_ESP" or v.Name == "Nillo_ESP2D" then v:Destroy() end
        end
    end)

    local ESP_Folder = Instance.new("Folder", CoreGui); ESP_Folder.Name = "Nillo_ESP"
    local ESP2D_Gui = Instance.new("ScreenGui", CoreGui); ESP2D_Gui.Name = "Nillo_ESP2D"; ESP2D_Gui.IgnoreGuiInset = true

    -- ⚡ INTERFACE GRÁFICA EXTREMA
    local ScreenGui = Instance.new("ScreenGui", CoreGui)
    ScreenGui.Name = "ComlockSilva_UI"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Global

    local function Tween(obj, props, time, style, dir)
        time = time or 0.3
        style = style or Enum.EasingStyle.Quint
        dir = dir or Enum.EasingDirection.Out
        local tw = TweenService:Create(obj, TweenInfo.new(time, style, dir), props)
        tw:Play()
        return tw
    end

    -- Sistema de Notificações Moderno
    local NotifFrame = Instance.new("Frame", ScreenGui)
    NotifFrame.Size = UDim2.new(0, 280, 1, -20); NotifFrame.Position = UDim2.new(1, -300, 0, 10); NotifFrame.BackgroundTransparency = 1
    local NotifLayout = Instance.new("UIListLayout", NotifFrame)
    NotifLayout.SortOrder = Enum.SortOrder.LayoutOrder; NotifLayout.VerticalAlignment = Enum.VerticalAlignment.Bottom; NotifLayout.Padding = UDim.new(0, 12)

    local function SendNotification(title, text, duration)
        duration = duration or 3
        local notif = Instance.new("Frame", NotifFrame)
        notif.Size = UDim2.new(1, 0, 0, 70); notif.BackgroundColor3 = UI_PANEL; notif.Position = UDim2.new(1, 100, 0, 0); notif.BackgroundTransparency = GLASS_TRANSPARENCY
        Instance.new("UICorner", notif).CornerRadius = UDim.new(0, 10)
        
        local stroke = Instance.new("UIStroke", notif); stroke.Color = THEME_PRIMARY; stroke.Thickness = 1.5; stroke.Transparency = 0.5
        
        local lblTitle = Instance.new("TextLabel", notif)
        lblTitle.Size = UDim2.new(1, -20, 0, 20); lblTitle.Position = UDim2.new(0, 15, 0, 10); lblTitle.BackgroundTransparency = 1
        lblTitle.Text = string.upper(title); lblTitle.Font = Enum.Font.GothamBlack; lblTitle.TextColor3 = THEME_PRIMARY; lblTitle.TextSize = 13; lblTitle.TextXAlignment = Enum.TextXAlignment.Left

        local lblText = Instance.new("TextLabel", notif)
        lblText.Size = UDim2.new(1, -20, 0, 30); lblText.Position = UDim2.new(0, 15, 0, 30); lblText.BackgroundTransparency = 1
        lblText.Text = text; lblText.Font = Enum.Font.GothamMedium; lblText.TextColor3 = THEME_TEXT; lblText.TextSize = 12; lblText.TextXAlignment = Enum.TextXAlignment.Left; lblText.TextWrapped = true

        local barFill = Instance.new("Frame", notif)
        barFill.Size = UDim2.new(0, 0, 0, 3); barFill.Position = UDim2.new(0, 0, 1, -3); barFill.BackgroundColor3 = THEME_PRIMARY; barFill.BorderSizePixel = 0
        Instance.new("UICorner", barFill).CornerRadius = UDim.new(1, 0)

        Tween(notif, {Position = UDim2.new(0, 0, 0, 0)}, 0.5, Enum.EasingStyle.Back)
        Tween(barFill, {Size = UDim2.new(1, 0, 0, 3)}, duration, Enum.EasingStyle.Linear)

        task.delay(duration, function()
            local fadeOut = Tween(notif, {Position = UDim2.new(1, 100, 0, 0), BackgroundTransparency = 1}, 0.4, Enum.EasingStyle.Back, Enum.EasingDirection.In)
            fadeOut.Completed:Wait(); notif:Destroy()
        end)
    end

    local function Drag(obj, trigger)
        local drag, start, startPos
        trigger = trigger or obj
        trigger.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then drag = true; start = i.Position; startPos = obj.Position end end)
        UserInputService.InputChanged:Connect(function(i)
            if drag and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
                local d = i.Position - start
                Tween(obj, {Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)}, 0.1, Enum.EasingStyle.Linear)
            end
        end)
        trigger.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then drag = false end end)
    end

    -- Botão Flutuante
    local MainBtn = Instance.new("TextButton", ScreenGui)
    MainBtn.Size = UDim2.new(0, 50, 0, 50); MainBtn.Position = UDim2.new(0.02, 0, 0.2, 0); MainBtn.BackgroundColor3 = THEME_BG; MainBtn.BackgroundTransparency = GLASS_TRANSPARENCY
    MainBtn.Text = "C"; MainBtn.Font = Enum.Font.GothamBlack; MainBtn.TextSize = 25; MainBtn.TextColor3 = THEME_PRIMARY; MainBtn.AutoButtonColor = false
    Instance.new("UICorner", MainBtn).CornerRadius = UDim.new(1,0)
    local BtnStroke = Instance.new("UIStroke", MainBtn); BtnStroke.Color = THEME_PRIMARY; BtnStroke.Thickness = 2
    Drag(MainBtn)

    MainBtn.MouseEnter:Connect(function() Tween(MainBtn, {Size = UDim2.new(0, 56, 0, 56)}, 0.2, Enum.EasingStyle.Back) end)
    MainBtn.MouseLeave:Connect(function() Tween(MainBtn, {Size = UDim2.new(0, 50, 0, 50)}, 0.2, Enum.EasingStyle.Back) end)

    -- Painel Principal
    local Main = Instance.new("Frame", ScreenGui)
    Main.Size = UDim2.new(0, 0, 0, 0); Main.Position = UDim2.new(0.5, 0, 0.5, 0); Main.AnchorPoint = Vector2.new(0.5, 0.5)
    Main.BackgroundColor3 = THEME_BG; Main.BackgroundTransparency = GLASS_TRANSPARENCY; Main.ClipsDescendants = true; Main.Visible = false
    Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 14)
    local MainStroke = Instance.new("UIStroke", Main); MainStroke.Color = THEME_PRIMARY_DARK; MainStroke.Thickness = 1
    
    local TopBar = Instance.new("Frame", Main)
    TopBar.Size = UDim2.new(1, 0, 0, 45); TopBar.BackgroundTransparency = 1
    Drag(Main, TopBar)

    local Title = Instance.new("TextLabel", TopBar)
    Title.Size = UDim2.new(1, -20, 1, 0); Title.Position = UDim2.new(0, 20, 0, 0); Title.BackgroundTransparency = 1
    Title.Text = "COMLOCK SILVA <font color='rgb(155,89,182)'>ROXO TEMA</font>"; Title.RichText = true
    Title.TextColor3 = THEME_TEXT; Title.Font = Enum.Font.GothamBlack; Title.TextSize = 16; Title.TextXAlignment = Enum.TextXAlignment.Left
    
    local Divider = Instance.new("Frame", Main)
    Divider.Size = UDim2.new(1, -40, 0, 1); Divider.Position = UDim2.new(0, 20, 0, 45); Divider.BackgroundColor3 = THEME_TEXT_MUTED; Divider.BackgroundTransparency = 0.8; Divider.BorderSizePixel = 0

    local function ToggleMenu()
        if Main.Visible then
            local tw = Tween(Main, {Size = UDim2.new(0, 0, 0, 0), BackgroundTransparency = 1}, 0.3, Enum.EasingStyle.Back, Enum.EasingDirection.In)
            tw.Completed:Wait(); Main.Visible = false
        else
            Main.Visible = true
            Tween(Main, {Size = UDim2.new(0, 480, 0, 360), BackgroundTransparency = GLASS_TRANSPARENCY}, 0.4, Enum.EasingStyle.Back)
        end
    end
    MainBtn.MouseButton1Click:Connect(ToggleMenu)
    UserInputService.InputBegan:Connect(function(i, gpe) if not gpe and (i.KeyCode == Enum.KeyCode.Insert or i.KeyCode == Enum.KeyCode.RightControl) then ToggleMenu() end end)

    -- Sidebar
    local Sidebar = Instance.new("Frame", Main)
    Sidebar.Size = UDim2.new(0, 130, 1, -65); Sidebar.Position = UDim2.new(0, 15, 0, 55); Sidebar.BackgroundColor3 = UI_PANEL; Sidebar.BackgroundTransparency = 0.3
    Instance.new("UICorner", Sidebar).CornerRadius = UDim.new(0, 10)
    
    local SidebarLayout = Instance.new("UIListLayout", Sidebar)
    SidebarLayout.Padding = UDim.new(0, 6); SidebarLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    Instance.new("UIPadding", Sidebar).PaddingTop = UDim.new(0, 8)

    local Content = Instance.new("Frame", Main)
    Content.Size = UDim2.new(1, -170, 1, -65); Content.Position = UDim2.new(0, 155, 0, 55); Content.BackgroundTransparency = 1

    local Tabs = {}
    local ActiveTabLine = Instance.new("Frame", Sidebar)
    ActiveTabLine.Size = UDim2.new(0, 3, 0, 28); ActiveTabLine.BackgroundColor3 = THEME_PRIMARY; ActiveTabLine.BorderSizePixel = 0
    Instance.new("UICorner", ActiveTabLine).CornerRadius = UDim.new(1, 0)

    local function AddTab(name)
        local p = Instance.new("ScrollingFrame", Content)
        p.Size = UDim2.new(1, 0, 1, 0); p.BackgroundTransparency = 1; p.Visible = false; p.ScrollBarThickness = 2
        p.ScrollBarImageColor3 = THEME_PRIMARY; p.BorderSizePixel = 0; p.CanvasSize = UDim2.new(0, 0, 0, 0)
        
        local pLayout = Instance.new("UIListLayout", p)
        pLayout.Padding = UDim.new(0, 10); pLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
        pLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() p.CanvasSize = UDim2.new(0, 0, 0, pLayout.AbsoluteContentSize.Y + 15) end)
        Instance.new("UIPadding", p).PaddingTop = UDim.new(0, 2)

        local b = Instance.new("TextButton", Sidebar)
        b.Size = UDim2.new(0.9, 0, 0, 34); b.BackgroundColor3 = UI_BUTTON; b.BackgroundTransparency = 1
        b.Text = "  " .. name; b.TextColor3 = THEME_TEXT_MUTED; b.Font = Enum.Font.GothamBold; b.TextSize = 11; b.AutoButtonColor = false; b.TextXAlignment = Enum.TextXAlignment.Left
        Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)

        b.MouseEnter:Connect(function() if not p.Visible then Tween(b, {BackgroundTransparency = 0.5, TextColor3 = THEME_TEXT}, 0.2) end end)
        b.MouseLeave:Connect(function() if not p.Visible then Tween(b, {BackgroundTransparency = 1, TextColor3 = THEME_TEXT_MUTED}, 0.2) end end)

        b.MouseButton1Click:Connect(function()
            for _, t in pairs(Tabs) do t.P.Visible = false; Tween(t.B, {BackgroundTransparency = 1, TextColor3 = THEME_TEXT_MUTED}, 0.3) end
            p.Visible = true
            Tween(b, {BackgroundTransparency = 0, TextColor3 = THEME_PRIMARY}, 0.3)
            Tween(ActiveTabLine, {Position = UDim2.new(0, 2, 0, b.Position.Y.Offset + 3)}, 0.3, Enum.EasingStyle.Back)
        end)
        table.insert(Tabs, {P = p, B = b, Name = name})
        return p
    end

    local function AddToggle(parent, text, default, callback)
        local b = Instance.new("TextButton", parent)
        b.Size = UDim2.new(0.98, 0, 0, 45); b.BackgroundColor3 = UI_PANEL; b.BackgroundTransparency = 0.2
        b.Text = "   " .. text; b.TextColor3 = THEME_TEXT; b.Font = Enum.Font.GothamSemibold; b.TextSize = 12; b.TextXAlignment = Enum.TextXAlignment.Left; b.AutoButtonColor = false
        Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8); Instance.new("UIStroke", b).Color = UI_TOGGLE_OFF; Instance.new("UIStroke", b).Thickness = 1
        
        local ind = Instance.new("Frame", b)
        ind.Size = UDim2.new(0, 42, 0, 22); ind.Position = UDim2.new(1, -55, 0.5, -11); ind.BackgroundColor3 = default and THEME_PRIMARY or UI_TOGGLE_OFF
        Instance.new("UICorner", ind).CornerRadius = UDim.new(1,0)
        
        local circle = Instance.new("Frame", ind)
        circle.Size = UDim2.new(0, 16, 0, 16); circle.Position = default and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8); circle.BackgroundColor3 = Color3.fromRGB(255,255,255)
        Instance.new("UICorner", circle).CornerRadius = UDim.new(1,0)
        Instance.new("UIStroke", circle).Color = Color3.fromRGB(0,0,0); Instance.new("UIStroke", circle).Transparency = 0.8

        local s = default
        local func = function(state)
            if state ~= nil then s = state else s = not s end
            Tween(ind, {BackgroundColor3 = s and THEME_PRIMARY or UI_TOGGLE_OFF}, 0.3)
            Tween(circle, {Position = s and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8)}, 0.4, Enum.EasingStyle.Back)
            Tween(b:FindFirstChild("UIStroke"), {Color = s and THEME_PRIMARY_DARK or UI_TOGGLE_OFF}, 0.3)
            callback(s)
            if state == nil then SendNotification("Config Alterada", text .. " está " .. (s and "ATIVADO" or "DESATIVADO"), 1.5) end
        end
        b.MouseButton1Click:Connect(func)
        return {SetState = func}
    end

    local function AddSlider(parent, text, min, max, default, callback, suffix)
        local b = Instance.new("Frame", parent)
        b.Size = UDim2.new(0.98, 0, 0, 55); b.BackgroundColor3 = UI_PANEL; b.BackgroundTransparency = 0.2
        Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8); Instance.new("UIStroke", b).Color = UI_TOGGLE_OFF
        
        local lbl = Instance.new("TextLabel", b)
        lbl.Size = UDim2.new(1, -20, 0, 20); lbl.Position = UDim2.new(0, 15, 0, 8); lbl.BackgroundTransparency = 1
        lbl.TextColor3 = THEME_TEXT; lbl.Font = Enum.Font.GothamSemibold; lbl.TextSize = 11; lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.Text = text .. " <font color='rgb(150,150,150)'>|</font> " .. default .. (suffix or "")
        lbl.RichText = true
        
        local barBg = Instance.new("Frame", b)
        barBg.Size = UDim2.new(1, -30, 0, 6); barBg.Position = UDim2.new(0, 15, 0, 35); barBg.BackgroundColor3 = UI_TOGGLE_OFF; barBg.BorderSizePixel = 0
        Instance.new("UICorner", barBg).CornerRadius = UDim.new(1,0)
        
        local fill = Instance.new("Frame", barBg)
        fill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0); fill.BackgroundColor3 = THEME_PRIMARY; fill.BorderSizePixel = 0
        Instance.new("UICorner", fill).CornerRadius = UDim.new(1,0)

        local glow = Instance.new("UIStroke", fill); glow.Color = THEME_PRIMARY; glow.Transparency = 0.5; glow.Thickness = 2
        
        local btn = Instance.new("TextButton", b)
        btn.Size = UDim2.new(1, 0, 1, 0); btn.BackgroundTransparency = 1; btn.Text = ""
        
        local dragging = false
        btn.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then dragging = true; Tween(barBg, {Size = UDim2.new(1, -30, 0, 8)}, 0.2) end end)
        UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then dragging = false; Tween(barBg, {Size = UDim2.new(1, -30, 0, 6)}, 0.2) end end)
        
        local function update(input)
            local pos = math.clamp((input.Position.X - barBg.AbsolutePosition.X) / barBg.AbsoluteSize.X, 0, 1)
            local val = math.floor((min + ((max - min) * pos)) * 10) / 10
            Tween(fill, {Size = UDim2.new(pos, 0, 1, 0)}, 0.1, Enum.EasingStyle.Linear)
            lbl.Text = text .. " <font color='rgb(155,89,182)'>" .. val .. (suffix or "") .. "</font>"
            callback(val)
        end
        UserInputService.InputChanged:Connect(function(i) if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then update(i) end end)
        btn.MouseButton1Click:Connect(function() update(Mouse) end)
    end

    local function AddButton(parent, text, callback)
        local b = Instance.new("TextButton", parent)
        b.Size = UDim2.new(0.98, 0, 0, 40); b.BackgroundColor3 = UI_BUTTON; b.BackgroundTransparency = 0.1
        b.Text = text; b.TextColor3 = THEME_TEXT; b.Font = Enum.Font.GothamBold; b.TextSize = 12; b.AutoButtonColor = false
        Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)
        local stroke = Instance.new("UIStroke", b); stroke.Color = UI_TOGGLE_OFF
        
        b.MouseEnter:Connect(function() Tween(b, {BackgroundColor3 = THEME_PRIMARY_DARK, TextColor3 = THEME_BG}, 0.2); stroke.Color = THEME_PRIMARY end)
        b.MouseLeave:Connect(function() Tween(b, {BackgroundColor3 = UI_BUTTON, TextColor3 = THEME_TEXT}, 0.2); stroke.Color = UI_TOGGLE_OFF end)
        
        b.MouseButton1Click:Connect(function()
            Tween(b, {Size = UDim2.new(0.94, 0, 0, 36)}, 0.1, Enum.EasingStyle.Sine, Enum.EasingDirection.In)
            task.wait(0.1)
            Tween(b, {Size = UDim2.new(0.98, 0, 0, 40)}, 0.2, Enum.EasingStyle.Back)
            callback()
            SendNotification("Ação", text .. " Executado!", 2)
        end)
        return b
    end

    -- Criação das Abas
    local TabCombat = AddTab("COMBATE")
    local TabAimbot = AddTab("AIMBOT")
    local TabMovement = AddTab("MOVIMENTO")
    local TabPlayers = AddTab("JOGADORES")
    local TabVisual = AddTab("VISUAL")
    local TabMisc = AddTab("MISC")

    -- Populando Combate
    AddToggle(TabCombat, "Ligar Sistema Principal", false, function(v) Settings.Combat.Enabled = v; LockedTarget = nil end)
    AddToggle(TabCombat, "Travar Câmera (CamLock)", false, function(v) Settings.Combat.CamLock = v end)
    AddToggle(TabCombat, "Combate Livre (Follow)", false, function(v) Settings.Combat.FreeCombat = v end)
    
    local BtnDynamic, BtnSticky
    BtnDynamic = AddToggle(TabCombat, "Modo Dinâmico (Swap)", true, function(v) if v then Settings.Combat.Mode = "Dynamic"; if BtnSticky then BtnSticky.SetState(false) end end end)
    BtnSticky = AddToggle(TabCombat, "Modo Fixo (Sticky)", false, function(v) if v then Settings.Combat.Mode = "Sticky"; LockedTarget = nil; if BtnDynamic then BtnDynamic.SetState(false) end end end)
    
    AddToggle(TabCombat, "Mirar na Cabeça (Off = Corpo)", false, function(v) Settings.Combat.AimPart = v and "Head" or "HumanoidRootPart"; LockedTarget = nil end)
    AddToggle(TabCombat, "Mirar nas Costas (Bypass)", false, function(v) Settings.Combat.AimAtBack = v end)
    AddSlider(TabCombat, "Distância Costas", 1, 10, 3.5, function(v) Settings.Combat.BackOffset = v end, " stds")
    AddToggle(TabCombat, "Mirar em NPCs", true, function(v) Settings.Combat.TargetNPCs = v; LockedTarget = nil end)
    AddToggle(TabCombat, "Check de Parede", false, function(v) Settings.Combat.WallCheck = v end)

    -- Populando Aimbot
    AddSlider(TabAimbot, "Distância Máx", 100, 5000, 2500, function(v) Settings.Combat.MaxDistance = v end, " stds")
    AddSlider(TabAimbot, "FOV (Giro Total)", 1, 400, 400, function(v) Settings.Combat.FOV = v end, "°")
    AddToggle(TabAimbot, "Mostrar Círculo FOV", false, function(v) Settings.Combat.ShowFOV = v end)
    AddToggle(TabAimbot, "Triggerbot (Tiro Auto)", false, function(v) Settings.Combat.Triggerbot = v end)
    AddSlider(TabAimbot, "Delay Trigger", 0, 1000, 50, function(v) Settings.Combat.TriggerDelay = v / 1000 end, " ms")

    -- Populando Movimento
    AddToggle(TabMovement, "Modo Aranha (Spider)", false, function(v) Settings.Movement.SpiderEnabled = v; if not v then stopClimbing() end end)
    AddToggle(TabMovement, "Atravessar (Noclip)", false, function(v) Settings.Movement.Noclip = v end)
    AddToggle(TabMovement, "Pulo Infinito", false, function(v) Settings.Movement.InfJump = v end)
    AddToggle(TabMovement, "Speed Hack", false, function(v) Settings.Movement.SpeedHackEnabled = v; if not v and humanoid then humanoid.WalkSpeed = 16 end end)
    AddSlider(TabMovement, "Veloc. Speed Hack", 16, 350, 50, function(v) Settings.Movement.NormalSpeed = v end)

    -- Populando Jogadores
    local PlayerListIndex = 1
    local PlayerSelectFrame = Instance.new("Frame", TabPlayers)
    PlayerSelectFrame.Size = UDim2.new(0.98, 0, 0, 50); PlayerSelectFrame.BackgroundColor3 = UI_PANEL; PlayerSelectFrame.BackgroundTransparency = 0.2
    Instance.new("UICorner", PlayerSelectFrame).CornerRadius = UDim.new(0, 8); Instance.new("UIStroke", PlayerSelectFrame).Color = UI_TOGGLE_OFF
    
    local BtnPrev = Instance.new("TextButton", PlayerSelectFrame); BtnPrev.Size = UDim2.new(0, 40, 1, 0); BtnPrev.BackgroundColor3 = UI_BUTTON; BtnPrev.Text = "<"; BtnPrev.TextColor3 = THEME_PRIMARY; BtnPrev.Font = Enum.Font.GothamBold; Instance.new("UICorner", BtnPrev).CornerRadius = UDim.new(0, 8)
    local BtnNext = Instance.new("TextButton", PlayerSelectFrame); BtnNext.Size = UDim2.new(0, 40, 1, 0); BtnNext.Position = UDim2.new(1, -40, 0, 0); BtnNext.BackgroundColor3 = UI_BUTTON; BtnNext.Text = ">"; BtnNext.TextColor3 = THEME_PRIMARY; BtnNext.Font = Enum.Font.GothamBold; Instance.new("UICorner", BtnNext).CornerRadius = UDim.new(0, 8)
    
    local PlayerNameLabel = Instance.new("TextLabel", PlayerSelectFrame)
    PlayerNameLabel.Size = UDim2.new(1, -80, 1, 0); PlayerNameLabel.Position = UDim2.new(0, 40, 0, 0); PlayerNameLabel.BackgroundTransparency = 1
    PlayerNameLabel.TextColor3 = THEME_TEXT; PlayerNameLabel.Font = Enum.Font.GothamBold; PlayerNameLabel.TextSize = 13

    local function UpdatePlayerSelection()
        local allPlayers = Players:GetPlayers()
        local validPlayers = {}
        for _, p in pairs(allPlayers) do if p ~= LocalPlayer then table.insert(validPlayers, p) end end
        if #validPlayers == 0 then PlayerNameLabel.Text = "NENHUM JOGADOR"; Settings.Players.SelectedPlayer = nil; return end
        if PlayerListIndex > #validPlayers then PlayerListIndex = 1 end
        if PlayerListIndex < 1 then PlayerListIndex = #validPlayers end
        Settings.Players.SelectedPlayer = validPlayers[PlayerListIndex]
        PlayerNameLabel.Text = string.upper(Settings.Players.SelectedPlayer.Name)
        if Settings.Players.Spectating and Settings.Players.SelectedPlayer.Character then Camera.CameraSubject = Settings.Players.SelectedPlayer.Character:FindFirstChild("Humanoid") end
    end

    BtnNext.MouseButton1Click:Connect(function() PlayerListIndex = PlayerListIndex + 1; UpdatePlayerSelection() end)
    BtnPrev.MouseButton1Click:Connect(function() PlayerListIndex = PlayerListIndex - 1; UpdatePlayerSelection() end)
    Players.PlayerAdded:Connect(UpdatePlayerSelection); Players.PlayerRemoving:Connect(UpdatePlayerSelection)

    AddButton(TabPlayers, "Teleportar (TP) para Alvo", function()
        local target = Settings.Players.SelectedPlayer
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            LocalPlayer.Character.HumanoidRootPart.CFrame = target.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)
        end
    end)
    AddToggle(TabPlayers, "Espectar Alvo", false, function(v)
        Settings.Players.Spectating = v
        if v then UpdatePlayerSelection() else if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then Camera.CameraSubject = LocalPlayer.Character.Humanoid end end
    end)

    -- Populando Visuals
    AddToggle(TabVisual, "Ligar ESP Global", true, function(v) Settings.Visuals.ESPEnabled = v end)
    AddToggle(TabVisual, "Caixa (Box 2D)", true, function(v) Settings.Visuals.ESPBox = v end)
    AddToggle(TabVisual, "Linha (Tracer)", true, function(v) Settings.Visuals.ESPTracer = v end)
    AddToggle(TabVisual, "Mostrar Arma", true, function(v) Settings.Visuals.ESPShowWeapon = v end)
    AddToggle(TabVisual, "Ultra FPS Boost", false, function(v) Settings.Visuals.FPSBoost = v; if v then ApplyFPSBoost() end end)

    -- Populando Misc
    AddToggle(TabMisc, "Anti-Aim", false, function(v) Settings.Combat.AntiAim = v end)
    AddSlider(TabMisc, "Angulo Anti-Aim", 5, 180, 30, function(v) Settings.Combat.AntiAimAngle = v end, "°")
    AddToggle(TabMisc, "Esquiva Automatica", false, function(v) Settings.Combat.AutoDodge = v end)
    AddSlider(TabMisc, "Distancia Esquiva", 5, 50, 15, function(v) Settings.Combat.DodgeRange = v end, "s")
    AddSlider(TabMisc, "Cooldown Esquiva", 0.1, 2, 0.5, function(v) Settings.Combat.DodgeCooldown = v end, "s")

    -- INICIALIZAÇÃO DA INTERFACE
    Tabs[1].P.Visible = true
    Tabs[1].B.BackgroundTransparency = 0; Tabs[1].B.TextColor3 = THEME_PRIMARY
    Tween(ActiveTabLine, {Position = UDim2.new(0, 2, 0, Tabs[1].B.Position.Y.Offset + 3)}, 0)
    UpdatePlayerSelection()
    
    task.delay(1, function()
        SendNotification("Sistema Injetado", "Interface Comlock Silva Ativada.\nAperte INSERT ou R-CTRL para fechar.", 5)
    end)

    -- =========================================================================
    -- 🔥 CONTADOR DE FPS (CANTO INFERIOR DIREITO, ROXO)
    -- =========================================================================
    local fpsFrame = Instance.new("Frame", ScreenGui)
    fpsFrame.Size = UDim2.new(0, 80, 0, 25)
    fpsFrame.Position = UDim2.new(1, -90, 1, -35) -- canto inferior direito
    fpsFrame.BackgroundColor3 = UI_PANEL
    fpsFrame.BackgroundTransparency = GLASS_TRANSPARENCY
    fpsFrame.BorderSizePixel = 0
    Instance.new("UICorner", fpsFrame).CornerRadius = UDim.new(0, 6)
    local fpsStroke = Instance.new("UIStroke", fpsFrame); fpsStroke.Color = THEME_PRIMARY; fpsStroke.Thickness = 1

    local fpsLabel = Instance.new("TextLabel", fpsFrame)
    fpsLabel.Size = UDim2.new(1, 0, 1, 0)
    fpsLabel.BackgroundTransparency = 1
    fpsLabel.Text = "FPS: 0"
    fpsLabel.TextColor3 = THEME_PRIMARY
    fpsLabel.Font = Enum.Font.GothamBold
    fpsLabel.TextSize = 14
    fpsLabel.TextXAlignment = Enum.TextXAlignment.Center
    fpsLabel.TextYAlignment = Enum.TextYAlignment.Center

    local frameCount = 0
    local elapsedTime = 0
    RunService.Heartbeat:Connect(function(dt)
        frameCount = frameCount + 1
        elapsedTime = elapsedTime + dt
    end)

    task.spawn(function()
        while true do
            task.wait(0.5)
            local fps = frameCount / elapsedTime
            fpsLabel.Text = string.format("FPS: %d", math.floor(fps + 0.5))
            frameCount = 0
            elapsedTime = 0
        end
    end)

    -- =========================================================================
    -- LÓGICA CORE (AIMBOT INSTANTÂNEO & ESP RENDER) + OTIMIZAÇÃO CACHE
    -- =========================================================================

    -- 🌟 OTIMIZAÇÃO: Sistema de Cache de NPCs para zerar o LAG
    local NPCCache = {}
    task.spawn(function()
        while task.wait(3) do -- Só varre o mapa inteiro a cada 3 segundos em vez de 60x por segundo
            if Settings.Combat.TargetNPCs then
                pcall(function()
                    table.clear(NPCCache)
                    for _, v in pairs(Workspace:GetDescendants()) do
                        if v:IsA("Model") and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and not Players:GetPlayerFromCharacter(v) then
                            table.insert(NPCCache, v)
                        end
                    end
                end)
            end
        end
    end)

    local function GetIsVisible(targetChar, part)
        local origin = Camera.CFrame.Position
        local direction = part.Position - origin
        local params = RaycastParams.new()
        params.FilterType = Enum.RaycastFilterType.Exclude
        params.FilterDescendantsInstances = {LocalPlayer.Character, targetChar, Camera}
        return Workspace:Raycast(origin, direction, params) == nil
    end

    local function IsValidTarget(target)
        if not target then return false end
        local hum = target:FindFirstChild("Humanoid")
        local root = target:FindFirstChild(Settings.Combat.AimPart) or target:FindFirstChild("Head")
        if hum and hum.Health > 0 and root then
            if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return false end
            local dist = (LocalPlayer.Character.HumanoidRootPart.Position - root.Position).Magnitude
            if dist > Settings.Combat.MaxDistance then return false end
            if Settings.Combat.WallCheck and not GetIsVisible(target, root) then return false end
            return true
        end
        return false
    end

    local function GetAngleToTarget(targetPart)
        if not targetPart then return 180 end
        local cameraCF = Camera.CFrame
        local toTarget = (targetPart.Position - cameraCF.Position).Unit
        local cameraDirection = cameraCF.LookVector
        local dot = cameraDirection:Dot(toTarget)
        local angle = math.acos(math.clamp(dot, -1, 1)) * (180 / math.pi)
        return angle
    end

    local function GetClosestToPlayer()
        local shortestDist = Settings.Combat.MaxDistance
        local bestAngle = Settings.Combat.FOV
        local chosenTarget = nil
        if not LocalPlayer.Character then return nil end
        local myRoot = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if not myRoot then return nil end

        local potential = {}
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character then table.insert(potential, p.Character) end
        end
        -- OTIMIZADO: Lê os NPCs direto do cache ao invés de buscar no mapa inteiro
        if Settings.Combat.TargetNPCs then
            for _, npc in ipairs(NPCCache) do table.insert(potential, npc) end
        end

        for _, char in pairs(potential) do
            local root = char:FindFirstChild(Settings.Combat.AimPart) or char:FindFirstChild("Head")
            local hum = char:FindFirstChild("Humanoid")
            if root and hum and hum.Health > 0 then
                local dist = (myRoot.Position - root.Position).Magnitude
                local angle = GetAngleToTarget(root)
                if angle <= bestAngle and dist <= Settings.Combat.MaxDistance then
                    if not Settings.Combat.WallCheck or GetIsVisible(char, root) then
                        if angle < bestAngle or (angle == bestAngle and dist < shortestDist) then
                            bestAngle = angle
                            shortestDist = dist
                            chosenTarget = char
                        end
                    end
                end
            end
        end
        return chosenTarget
    end

    function ApplyFPSBoost()
        if not Settings.Visuals.FPSBoost then return end
        for _, v in pairs(Workspace:GetDescendants()) do
            if LocalPlayer.Character and v:IsDescendantOf(LocalPlayer.Character) then continue end
            if v:IsA("BasePart") then
                v.Material = Enum.Material.SmoothPlastic
                v.Reflectance = 0
                v.CastShadow = false
                if v:IsA("MeshPart") then v.TextureID = "" end
            elseif v:IsA("Decal") or v:IsA("Texture") then v.Transparency = 1
            elseif v:IsA("ParticleEmitter") or v:IsA("Trail") then v.Enabled = false end
        end
        game:GetService("Lighting").GlobalShadows = false
        game:GetService("Lighting").FogEnd = 9e9
    end
    task.spawn(function() while task.wait(5) do if Settings.Visuals.FPSBoost then pcall(ApplyFPSBoost) end end end)

    local function PerformDodge(target)
        if not canDodge or not rootPart then return end
        canDodge = false
        local dodgeDir = Vector3.new()
        local forward = rootPart.CFrame.LookVector; local right = rootPart.CFrame.RightVector
        if Settings.Combat.DodgeDirection == "Back" then dodgeDir = -forward * 8
        elseif Settings.Combat.DodgeDirection == "Side" then dodgeDir = right * (math.random() > 0.5 and 6 or -6)
        else dodgeDir = Vector3.new(math.random(-8,8), 0, math.random(-8,8)) end
        
        local targetPos = rootPart.Position + dodgeDir
        if Settings.Combat.DodgeStyle == "Tween" then
            TweenService:Create(rootPart, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {CFrame = CFrame.new(targetPos)}):Play()
        else
            rootPart.CFrame = CFrame.new(targetPos)
        end
        task.wait(Settings.Combat.DodgeCooldown)
        canDodge = true
    end

    local function CheckDodge(target)
        if not Settings.Combat.AutoDodge or not canDodge or not target or not target:FindFirstChild("Humanoid") or not rootPart then return end
        local dist = (target.HumanoidRootPart.Position - rootPart.Position).Magnitude
        if dist > Settings.Combat.DodgeRange then return end
        local targetRoot = target:FindFirstChild("HumanoidRootPart") or target:FindFirstChild("Head")
        if not targetRoot then return end
        
        local angle = math.acos(math.clamp(targetRoot.CFrame.LookVector:Dot((rootPart.Position - targetRoot.Position).Unit), -1, 1)) * (180 / math.pi)
        if angle < 45 then
            local enemyAnim = target.Humanoid:FindFirstChild("Animator")
            if enemyAnim then
                for _, track in pairs(enemyAnim:GetPlayingAnimationTracks()) do
                    if (track.Priority == Enum.AnimationPriority.Action or track.Priority == Enum.AnimationPriority.Movement) and track.Speed > 0 then
                        PerformDodge(target); break
                    end
                end
            end
        end
    end

    local function CheckTrigger()
        if not Settings.Combat.Triggerbot or not LockedTarget then return end
        local aimPart = LockedTarget:FindFirstChild(Settings.Combat.AimPart) or LockedTarget:FindFirstChild("Head")
        if aimPart and GetAngleToTarget(aimPart) <= 5 then
            local now = tick()
            if now - lastShotTime >= Settings.Combat.TriggerDelay then mouse1click(); lastShotTime = now end
        end
    end

    local function ApplyAntiAim()
        if not Settings.Combat.AntiAim or not rootPart then return end
        if humanoid and humanoid.MoveDirection.Magnitude < 0.1 then
            rootPart.CFrame = rootPart.CFrame * CFrame.Angles(0, math.rad(math.random(-Settings.Combat.AntiAimAngle, Settings.Combat.AntiAimAngle)), 0)
        end
    end

    -- Loop ESP Visual (Highlight/Billboard)
    task.spawn(function()
        while task.wait(0.1) do
            pcall(function()
                local targets = {}
                for _, p in pairs(Players:GetPlayers()) do if p ~= LocalPlayer and p.Character then table.insert(targets, p.Character) end end
                -- OTIMIZADO: Lê os NPCs direto do cache
                if Settings.Combat.TargetNPCs then
                    for _, npc in ipairs(NPCCache) do table.insert(targets, npc) end
                end

                for _, char in pairs(targets) do
                    local root = char:FindFirstChild("HumanoidRootPart")
                    local hum = char:FindFirstChild("Humanoid")
                    local id = char:GetDebugId()

                    if Settings.Combat.AutoDodge and root then CheckDodge(char) end

                    if Settings.Visuals.ESPEnabled and root and hum and hum.Health > 0 then
                        local isLocked = (char == LockedTarget)
                        local finalColor = isLocked and COLOR_LOCKED or (GetIsVisible(char, root) and COLOR_VISIBLE or COLOR_HIDDEN)

                        local hl = ESP_Folder:FindFirstChild(id.."_HL") or Instance.new("Highlight", ESP_Folder)
                        hl.Name = id.."_HL"; hl.Adornee = char; hl.FillColor = finalColor; hl.OutlineColor = finalColor; hl.FillTransparency = 0.6; hl.OutlineTransparency = 0

                        local bb = ESP_Folder:FindFirstChild(id.."_BB") or Instance.new("BillboardGui", ESP_Folder)
                        bb.Name = id.."_BB"; bb.Adornee = char:FindFirstChild("Head") or root; bb.Size = UDim2.new(0, 200, 0, 60); bb.AlwaysOnTop = true; bb.StudsOffset = Vector3.new(0, 3, 0)

                        local txt = bb:FindFirstChild("Text") or Instance.new("TextLabel", bb)
                        txt.Name = "Text"; txt.BackgroundTransparency = 1; txt.Size = UDim2.new(1,0,1,0); txt.Font = Enum.Font.GothamBold; txt.TextSize = 13; txt.TextColor3 = finalColor; txt.TextStrokeTransparency = 0.8

                        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                            local dist = math.floor((root.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude)
                            local name = char.Name
                            if not Players:GetPlayerFromCharacter(char) then name = "[NPC] " .. name end
                            local weapon = ""
                            if Settings.Visuals.ESPShowWeapon then local tool = char:FindFirstChildOfClass("Tool"); if tool then weapon = " [" .. tool.Name .. "]" end end
                            txt.Text = string.format("%s%s%s\n[%dm] HP: %d", (isLocked and "LOCKED | " or ""), name, weapon, dist, math.floor(hum.Health))
                        end
                    else
                        if ESP_Folder:FindFirstChild(id.."_HL") then ESP_Folder[id.."_HL"]:Destroy() end
                        if ESP_Folder:FindFirstChild(id.."_BB") then ESP_Folder[id.."_BB"]:Destroy() end
                    end
                end
                if not Settings.Visuals.ESPEnabled then ESP_Folder:ClearAllChildren() end
            end)
        end
    end)

    -- ESP 2D RENDER (Linhas/Boxes)
    RunService.RenderStepped:Connect(function()
        for _, v in pairs(ESP2D_Gui:GetChildren()) do v.Visible = false end
        if not Settings.Visuals.ESPEnabled then return end

        local targets = {}
        for _, p in pairs(Players:GetPlayers()) do if p ~= LocalPlayer and p.Character then table.insert(targets, p.Character) end end
        -- OTIMIZADO: Lê os NPCs direto do cache, salvando muito FPS aqui no RenderStepped
        if Settings.Combat.TargetNPCs then
            for _, npc in ipairs(NPCCache) do table.insert(targets, npc) end
        end

        for _, char in pairs(targets) do
            local root = char:FindFirstChild("HumanoidRootPart"); local head = char:FindFirstChild("Head"); local hum = char:FindFirstChild("Humanoid")
            if root and head and hum and hum.Health > 0 then
                local finalColor = (char == LockedTarget) and COLOR_LOCKED or (GetIsVisible(char, root) and COLOR_VISIBLE or COLOR_HIDDEN)
                local rootPos, onScreen = Camera:WorldToViewportPoint(root.Position)
                
                if onScreen then
                    local id = char:GetDebugId()
                    if Settings.Visuals.ESPTracer then
                        local line = ESP2D_Gui:FindFirstChild(id .. "_Tracer") or Instance.new("Frame", ESP2D_Gui)
                        line.Name = id .. "_Tracer"; line.AnchorPoint = Vector2.new(0.5, 0.5); line.BorderSizePixel = 0; line.BackgroundColor3 = finalColor; line.Visible = true
                        
                        local bottomCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y); local targetPos = Vector2.new(rootPos.X, rootPos.Y)
                        line.Size = UDim2.new(0, 1.5, 0, (bottomCenter - targetPos).Magnitude)
                        line.Position = UDim2.new(0, (bottomCenter.X + targetPos.X) / 2, 0, (bottomCenter.Y + targetPos.Y) / 2)
                        line.Rotation = math.deg(math.atan2(targetPos.Y - bottomCenter.Y, targetPos.X - bottomCenter.X)) + 90
                    end
                    if Settings.Visuals.ESPBox then
                        local headPos = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))
                        local legPos = Camera:WorldToViewportPoint(root.Position - Vector3.new(0, 3, 0))
                        local height = math.abs(headPos.Y - legPos.Y); local width = height / 1.5
                        
                        local box = ESP2D_Gui:FindFirstChild(id .. "_Box") or Instance.new("Frame", ESP2D_Gui)
                        if box.Name ~= id .. "_Box" then box.Name = id .. "_Box"; box.BackgroundTransparency = 1; Instance.new("UIStroke", box).Thickness = 1.5 end
                        box.Visible = true; box.Size = UDim2.new(0, width, 0, height); box.Position = UDim2.new(0, headPos.X - width/2, 0, headPos.Y)
                        box:FindFirstChild("UIStroke").Color = finalColor
                    end
                end
            end
        end
    end)

    -- Controles
    local function playSound(id, vol, pitch)
        if not rootPart and not LocalPlayer.PlayerGui then return end
        local s = Instance.new("Sound", rootPart or LocalPlayer.PlayerGui)
        s.SoundId = "rbxassetid://"..tostring(id)
        s.Volume = vol or 0.5
        s.Pitch = pitch or 1
        s:Play()
        Debris:AddItem(s, 2)
    end

    function stopClimbing()
        isClimbing = false
        if humanoid then humanoid.WalkSpeed = Settings.Movement.SpeedHackEnabled and Settings.Movement.NormalSpeed or 16; humanoid.AutoRotate = true end
        if bodyVel then bodyVel:Destroy() bodyVel = nil end; if bodyGyro then bodyGyro:Destroy() bodyGyro = nil end
        if climbTrack then climbTrack:Stop(0.1) end
    end

    local function onChar(char)
        character = char; humanoid = char:WaitForChild("Humanoid"); rootPart = char:WaitForChild("HumanoidRootPart"); animator = humanoid:WaitForChild("Animator")
        local animObj = Instance.new("Animation")
        animObj.AnimationId = (humanoid.RigType == Enum.HumanoidRigType.R15) and AnimIds.R15 or AnimIds.R6
        pcall(function() climbTrack = animator:LoadAnimation(animObj) end)
    end
    if LocalPlayer.Character then onChar(LocalPlayer.Character) end
    LocalPlayer.CharacterAdded:Connect(onChar)

    UserInputService.JumpRequest:Connect(function()
        if isClimbing and Settings.Movement.SpiderEnabled then
            stopClimbing(); lastJumpTime = tick()
            rootPart.Velocity = ((rootPart.CFrame.LookVector * -1.5) + Vector3.new(0, 1.2, 0)).Unit * Settings.Movement.WallJumpForce
            playSound(1428652081, 0.5, 1)
            return
        end
        if Settings.Movement.InfJump and rootPart then
            rootPart.Velocity = Vector3.new(rootPart.Velocity.X, Settings.Movement.InfJumpForce, rootPart.Velocity.Z)
            if humanoid then humanoid:ChangeState(Enum.HumanoidStateType.Jumping) end
        end
    end)
    UserInputService.InputBegan:Connect(function(i, gpe) if not gpe and i.KeyCode == Enum.KeyCode.LeftShift then isSprinting = true end end)
    UserInputService.InputEnded:Connect(function(i) if i.KeyCode == Enum.KeyCode.LeftShift then isSprinting = false end end)

    -- AIMBOT RENDER STEPPED (INSTANTÂNEO 0 LERP)
    RunService.RenderStepped:Connect(function()
        pcall(function()
            if Settings.Combat.Enabled then
                if Settings.Combat.Mode == "Dynamic" then
                    LockedTarget = GetClosestToPlayer()
                elseif Settings.Combat.Mode == "Sticky" then
                    if not IsValidTarget(LockedTarget) then
                        LockedTarget = GetClosestToPlayer()
                    end
                end

                if LockedTarget then
                    local aimRoot = LockedTarget:FindFirstChild(Settings.Combat.AimPart) or LockedTarget:FindFirstChild("Head")
                    if aimRoot then
                        local targetPosition = aimRoot.Position
                        
                        if Settings.Combat.AimAtBack then
                            targetPosition = (aimRoot.CFrame * CFrame.new(0, 0, Settings.Combat.BackOffset)).Position
                        end

                        if Settings.Combat.CamLock then
                            Camera.CFrame = CFrame.lookAt(Camera.CFrame.Position, targetPosition)
                        end
                        if Settings.Combat.FreeCombat and rootPart and humanoid then
                            humanoid.AutoRotate = false
                            rootPart.CFrame = CFrame.lookAt(rootPart.Position, Vector3.new(targetPosition.X, rootPart.Position.Y, targetPosition.Z))
                        end
                        CheckTrigger()
                    end
                else
                    if humanoid and not isClimbing then humanoid.AutoRotate = true end
                end
            else
                LockedTarget = nil
                if humanoid and not isClimbing then humanoid.AutoRotate = true end
            end

            ApplyAntiAim()
            if Settings.Movement.Noclip and character then
                for _, v in pairs(character:GetDescendants()) do if v:IsA("BasePart") and v.CanCollide then v.CanCollide = false end end
            end
        end)
    end)

    RunService.Heartbeat:Connect(function()
        if not rootPart or not humanoid then return end
        pcall(function()
            if Settings.Movement.SpeedHackEnabled and not isClimbing then humanoid.WalkSpeed = isSprinting and (Settings.Movement.NormalSpeed * Settings.Movement.SprintMult) or Settings.Movement.NormalSpeed end
            if not Settings.Movement.SpiderEnabled or tick() - lastJumpTime < 0.4 then return end

            local rayParams = RaycastParams.new(); rayParams.FilterDescendantsInstances = {character}
            local dirs = {rootPart.CFrame.LookVector, (rootPart.CFrame * CFrame.Angles(0, math.rad(30), 0)).LookVector, (rootPart.CFrame * CFrame.Angles(0, math.rad(-30), 0)).LookVector}
            local result = nil
            for _, dir in pairs(dirs) do
                local cast = Workspace:Raycast(rootPart.Position, dir * Settings.Movement.RayLength, rayParams)
                if cast then result = cast break end
            end

            if result then
                if not isClimbing then
                    isClimbing = true; humanoid.AutoRotate = false
                    bodyVel = Instance.new("BodyVelocity", rootPart); bodyVel.MaxForce = Vector3.new(9e9, 9e9, 9e9)
                    bodyGyro = Instance.new("BodyGyro", rootPart); bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9); bodyGyro.P = 15000
                    if climbTrack then climbTrack:Play() end
                    playSound(4806082497, 0.3, 1.1)
                end

                bodyGyro.CFrame = CFrame.lookAt(rootPart.Position, result.Position + result.Normal * -1)
                local move = humanoid.MoveDirection
                local speed = isSprinting and (Settings.Movement.ClimbSpeed * Settings.Movement.SprintMult) or Settings.Movement.ClimbSpeed

                if move.Magnitude > 0.1 then
                    local v = move:Dot(rootPart.CFrame.LookVector); local h = move:Dot(rootPart.CFrame.RightVector)
                    bodyVel.Velocity = (Vector3.new(0, 1, 0) * v * speed) + (rootPart.CFrame.RightVector * h * speed)
                    if climbTrack then climbTrack:AdjustSpeed(isSprinting and 2.2 or 1.4) end
                else
                    bodyVel.Velocity = Vector3.new(0, 0.1, 0)
                    if climbTrack then climbTrack:AdjustSpeed(0) end
                end
            else
                if isClimbing then
                    stopClimbing()
                    if humanoid.MoveDirection.Magnitude > 0 then rootPart.Velocity = Vector3.new(0, 45, 0) + (rootPart.CFrame.LookVector * 30) end
                end
            end
        end)
    end)
    print("COMLOCK SILVA (Glass UI + Roxo Otimizado) carregado!")
end)

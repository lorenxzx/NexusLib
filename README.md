--[[
    ███╗   ██╗███████╗██╗  ██╗██╗   ██╗███████╗    ██╗   ██╗██╗
    ████╗  ██║██╔════╝╚██╗██╔╝██║   ██║██╔════╝    ██║   ██║██║
    ██╔██╗ ██║█████╗   ╚███╔╝ ██║   ██║███████╗    ██║   ██║██║
    ██║╚██╗██║██╔══╝   ██╔██╗ ██║   ██║╚════██║    ██║   ██║██║
    ██║ ╚████║███████╗██╔╝ ██╗╚██████╔╝███████║    ╚██████╔╝██║
    ╚═╝  ╚═══╝╚══════╝╚═╝  ╚═╝ ╚═════╝ ╚══════╝     ╚═════╝ ╚═╝
    
    NexusUI — Modern Roblox UI Library
    Versão: 1.0.0
    Autor: NexusUI
    
    Funcionalidades:
      • Janelas arrastáveis com blur
      • Tabs / Abas
      • Botões e Toggles animados
      • Sliders customizados
      • Inputs de texto
      • Sistema de notificações
    
    USO BÁSICO:
    ─────────────────────────────────────────────────────────
    local NexusUI = loadstring(game:HttpGet("..."))()
    
    local Window = NexusUI:CreateWindow({
        Title = "Meu Script",
        SubTitle = "by você",
        Theme = "Dark",  -- "Dark" ou "Light"
    })
    
    local Tab = Window:AddTab("Principal", "rbxassetid://...")
    
    Tab:AddButton({ Label = "Clique aqui", Callback = function() print("Clicado!") end })
    Tab:AddToggle({ Label = "Ativar algo", Default = false, Callback = function(v) print(v) end })
    Tab:AddSlider({ Label = "Velocidade", Min = 1, Max = 100, Default = 16, Callback = function(v) print(v) end })
    Tab:AddInput({ Label = "Nome", Placeholder = "Digite aqui...", Callback = function(v) print(v) end })
    
    NexusUI:Notify({ Title = "Bem-vindo!", Message = "Script carregado com sucesso.", Duration = 4 })
    ─────────────────────────────────────────────────────────
--]]

local NexusUI = {}
NexusUI.__index = NexusUI

-- ══════════════════════════════════════════
--              SERVIÇOS
-- ══════════════════════════════════════════
local Players        = game:GetService("Players")
local TweenService   = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService     = game:GetService("RunService")
local CoreGui        = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- ══════════════════════════════════════════
--              TEMAS
-- ══════════════════════════════════════════
local Themes = {
    Dark = {
        Background      = Color3.fromRGB(13, 13, 18),
        Surface         = Color3.fromRGB(20, 20, 28),
        SurfaceHover    = Color3.fromRGB(26, 26, 36),
        Border          = Color3.fromRGB(40, 40, 58),
        Accent          = Color3.fromRGB(99, 102, 241),   -- índigo vibrante
        AccentHover     = Color3.fromRGB(129, 132, 255),
        AccentDim       = Color3.fromRGB(40, 42, 90),
        Text            = Color3.fromRGB(230, 230, 240),
        TextMuted       = Color3.fromRGB(110, 110, 140),
        TextDisabled    = Color3.fromRGB(60, 60, 80),
        Success         = Color3.fromRGB(52, 211, 153),
        Warning         = Color3.fromRGB(251, 191, 36),
        Error           = Color3.fromRGB(248, 113, 113),
        SliderFill      = Color3.fromRGB(99, 102, 241),
        ToggleOn        = Color3.fromRGB(99, 102, 241),
        ToggleOff       = Color3.fromRGB(45, 45, 60),
        TabActive       = Color3.fromRGB(99, 102, 241),
        TabInactive     = Color3.fromRGB(30, 30, 42),
        TitleBar        = Color3.fromRGB(16, 16, 22),
    },
    Light = {
        Background      = Color3.fromRGB(245, 245, 250),
        Surface         = Color3.fromRGB(255, 255, 255),
        SurfaceHover    = Color3.fromRGB(240, 240, 248),
        Border          = Color3.fromRGB(210, 210, 225),
        Accent          = Color3.fromRGB(79, 70, 229),
        AccentHover     = Color3.fromRGB(99, 90, 255),
        AccentDim       = Color3.fromRGB(220, 218, 255),
        Text            = Color3.fromRGB(20, 20, 35),
        TextMuted       = Color3.fromRGB(100, 100, 130),
        TextDisabled    = Color3.fromRGB(170, 170, 190),
        Success         = Color3.fromRGB(16, 185, 129),
        Warning         = Color3.fromRGB(217, 119, 6),
        Error           = Color3.fromRGB(220, 38, 38),
        SliderFill      = Color3.fromRGB(79, 70, 229),
        ToggleOn        = Color3.fromRGB(79, 70, 229),
        ToggleOff       = Color3.fromRGB(200, 200, 215),
        TabActive       = Color3.fromRGB(79, 70, 229),
        TabInactive     = Color3.fromRGB(235, 235, 245),
        TitleBar        = Color3.fromRGB(255, 255, 255),
    },
}

-- ══════════════════════════════════════════
--           UTILITÁRIOS INTERNOS
-- ══════════════════════════════════════════
local function Tween(obj, props, duration, style, dir)
    local info = TweenInfo.new(
        duration or 0.2,
        style or Enum.EasingStyle.Quart,
        dir or Enum.EasingDirection.Out
    )
    TweenService:Create(obj, info, props):Play()
end

local function MakeDraggable(frame, handle)
    local dragging, dragInput, dragStart, startPos = false, nil, nil, nil
    handle = handle or frame

    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    handle.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)
end

local function CreateInstance(class, props, children)
    local inst = Instance.new(class)
    for k, v in pairs(props or {}) do
        inst[k] = v
    end
    for _, child in pairs(children or {}) do
        child.Parent = inst
    end
    return inst
end

local function AddCorner(parent, radius)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, radius or 8)
    c.Parent = parent
    return c
end

local function AddStroke(parent, color, thickness)
    local s = Instance.new("UIStroke")
    s.Color = color
    s.Thickness = thickness or 1
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    s.Parent = parent
    return s
end

local function AddPadding(parent, top, bottom, left, right)
    local p = Instance.new("UIPadding")
    p.PaddingTop    = UDim.new(0, top or 8)
    p.PaddingBottom = UDim.new(0, bottom or 8)
    p.PaddingLeft   = UDim.new(0, left or 8)
    p.PaddingRight  = UDim.new(0, right or 8)
    p.Parent = parent
    return p
end

-- ══════════════════════════════════════════
--           SCREENGUI ROOT
-- ══════════════════════════════════════════
local ScreenGui = CreateInstance("ScreenGui", {
    Name = "NexusUI",
    ResetOnSpawn = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
})

-- Tenta colocar no CoreGui (executores), senão no PlayerGui
local ok = pcall(function() ScreenGui.Parent = CoreGui end)
if not ok then
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

-- Notificações container (canto inferior direito)
local NotifContainer = CreateInstance("Frame", {
    Name = "NotifContainer",
    BackgroundTransparency = 1,
    Position = UDim2.new(1, -20, 1, -20),
    AnchorPoint = Vector2.new(1, 1),
    Size = UDim2.new(0, 320, 1, -20),
    Parent = ScreenGui,
})
local NotifLayout = CreateInstance("UIListLayout", {
    SortOrder = Enum.SortOrder.LayoutOrder,
    VerticalAlignment = Enum.VerticalAlignment.Bottom,
    Padding = UDim.new(0, 8),
    Parent = NotifContainer,
})

-- ══════════════════════════════════════════
--         SISTEMA DE NOTIFICAÇÕES
-- ══════════════════════════════════════════
function NexusUI:Notify(config)
    local cfg = config or {}
    local title    = cfg.Title   or "Notificação"
    local message  = cfg.Message or ""
    local duration = cfg.Duration or 4
    local ntype    = cfg.Type or "Info" -- "Info", "Success", "Warning", "Error"

    local theme = Themes.Dark

    local typeColors = {
        Info    = theme.Accent,
        Success = theme.Success,
        Warning = theme.Warning,
        Error   = theme.Error,
    }
    local accentColor = typeColors[ntype] or theme.Accent

    -- Ícones simples por tipo
    local typeIcons = {
        Info    = "ℹ",
        Success = "✓",
        Warning = "⚠",
        Error   = "✕",
    }
    local icon = typeIcons[ntype] or "ℹ"

    local notif = CreateInstance("Frame", {
        BackgroundColor3 = theme.Surface,
        Size = UDim2.new(1, 0, 0, 0),
        AutomaticSize = Enum.AutomaticSize.Y,
        BackgroundTransparency = 1,
        Parent = NotifContainer,
    })
    AddCorner(notif, 10)
    AddStroke(notif, theme.Border, 1)

    -- Barra colorida lateral
    local bar = CreateInstance("Frame", {
        BackgroundColor3 = accentColor,
        Size = UDim2.new(0, 3, 1, 0),
        Position = UDim2.new(0, 0, 0, 0),
        Parent = notif,
    })
    AddCorner(bar, 3)

    local inner = CreateInstance("Frame", {
        BackgroundTransparency = 1,
        Size = UDim2.new(1, -15, 1, 0),
        Position = UDim2.new(0, 14, 0, 0),
        AutomaticSize = Enum.AutomaticSize.Y,
        Parent = notif,
    })
    AddPadding(inner, 10, 10, 6, 10)

    local iconLabel = CreateInstance("TextLabel", {
        BackgroundTransparency = 1,
        Size = UDim2.new(0, 22, 0, 22),
        Position = UDim2.new(0, 0, 0, 0),
        Text = icon,
        TextColor3 = accentColor,
        TextSize = 16,
        Font = Enum.Font.GothamBold,
        Parent = inner,
    })

    local titleLabel = CreateInstance("TextLabel", {
        BackgroundTransparency = 1,
        Size = UDim2.new(1, -28, 0, 20),
        Position = UDim2.new(0, 28, 0, 0),
        Text = title,
        TextColor3 = theme.Text,
        TextSize = 13,
        Font = Enum.Font.GothamBold,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = inner,
    })

    local msgLabel = CreateInstance("TextLabel", {
        BackgroundTransparency = 1,
        Size = UDim2.new(1, 0, 0, 0),
        Position = UDim2.new(0, 0, 0, 24),
        Text = message,
        TextColor3 = theme.TextMuted,
        TextSize = 12,
        Font = Enum.Font.Gotham,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextWrapped = true,
        AutomaticSize = Enum.AutomaticSize.Y,
        Parent = inner,
    })

    -- Barra de progresso
    local progressBg = CreateInstance("Frame", {
        BackgroundColor3 = theme.Border,
        Size = UDim2.new(1, 0, 0, 2),
        Position = UDim2.new(0, 0, 1, -2),
        Parent = notif,
    })
    local progressFill = CreateInstance("Frame", {
        BackgroundColor3 = accentColor,
        Size = UDim2.new(1, 0, 1, 0),
        Parent = progressBg,
    })

    -- Animação de entrada
    notif.BackgroundTransparency = 0
    notif.Position = UDim2.new(0.05, 0, 0, 0)
    Tween(notif, { Position = UDim2.new(0, 0, 0, 0) }, 0.3)

    -- Progresso
    Tween(progressFill, { Size = UDim2.new(0, 0, 1, 0) }, duration, Enum.EasingStyle.Linear)

    -- Remover após duração
    task.delay(duration, function()
        Tween(notif, { BackgroundTransparency = 1 }, 0.3)
        task.wait(0.35)
        notif:Destroy()
    end)

    return notif
end

-- ══════════════════════════════════════════
--           CRIAR JANELA
-- ══════════════════════════════════════════
function NexusUI:CreateWindow(config)
    local cfg = config or {}
    local title    = cfg.Title    or "NexusUI"
    local subtitle = cfg.SubTitle or "v1.0"
    local themeName = cfg.Theme   or "Dark"
    local size     = cfg.Size     or UDim2.new(0, 560, 0, 380)
    local position = cfg.Position or UDim2.new(0.5, -280, 0.5, -190)

    local T = Themes[themeName] or Themes.Dark

    -- ── Janela principal ──
    local Window = CreateInstance("Frame", {
        Name = "NexusWindow",
        BackgroundColor3 = T.Background,
        Size = size,
        Position = position,
        Parent = ScreenGui,
        ClipsDescendants = true,
    })
    AddCorner(Window, 12)
    AddStroke(Window, T.Border, 1)

    -- ── Barra de título ──
    local TitleBar = CreateInstance("Frame", {
        Name = "TitleBar",
        BackgroundColor3 = T.TitleBar,
        Size = UDim2.new(1, 0, 0, 46),
        Parent = Window,
    })
    AddStroke(TitleBar, T.Border, 1)

    -- Pill decorativo (accent)
    local pill = CreateInstance("Frame", {
        BackgroundColor3 = T.Accent,
        Size = UDim2.new(0, 4, 0, 24),
        Position = UDim2.new(0, 14, 0.5, -12),
        Parent = TitleBar,
    })
    AddCorner(pill, 4)

    local titleLabel = CreateInstance("TextLabel", {
        BackgroundTransparency = 1,
        Size = UDim2.new(0, 200, 1, 0),
        Position = UDim2.new(0, 26, 0, 0),
        Text = title,
        TextColor3 = T.Text,
        TextSize = 14,
        Font = Enum.Font.GothamBold,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = TitleBar,
    })
    local subtitleLabel = CreateInstance("TextLabel", {
        BackgroundTransparency = 1,
        Size = UDim2.new(0, 200, 1, 0),
        Position = UDim2.new(0, 26, 0, 16),
        Text = subtitle,
        TextColor3 = T.TextMuted,
        TextSize = 11,
        Font = Enum.Font.Gotham,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = TitleBar,
    })

    -- Botão fechar
    local closeBtn = CreateInstance("TextButton", {
        BackgroundColor3 = Color3.fromRGB(248, 113, 113),
        Size = UDim2.new(0, 14, 0, 14),
        Position = UDim2.new(1, -26, 0.5, -7),
        Text = "",
        Parent = TitleBar,
    })
    AddCorner(closeBtn, 7)
    closeBtn.MouseButton1Click:Connect(function()
        Tween(Window, { Size = UDim2.new(0, Window.AbsoluteSize.X, 0, 0) }, 0.25)
        task.wait(0.3)
        Window:Destroy()
    end)

    -- Botão minimizar
    local minimizeBtn = CreateInstance("TextButton", {
        BackgroundColor3 = Color3.fromRGB(251, 191, 36),
        Size = UDim2.new(0, 14, 0, 14),
        Position = UDim2.new(1, -46, 0.5, -7),
        Text = "",
        Parent = TitleBar,
    })
    AddCorner(minimizeBtn, 7)

    local minimized = false
    local normalSize = size
    minimizeBtn.MouseButton1Click:Connect(function()
        minimized = not minimized
        if minimized then
            normalSize = Window.Size
            Tween(Window, { Size = UDim2.new(normalSize.X.Scale, normalSize.X.Offset, 0, 46) }, 0.3)
        else
            Tween(Window, { Size = normalSize }, 0.3)
        end
    end)

    MakeDraggable(Window, TitleBar)

    -- ── Sidebar de Tabs ──
    local Sidebar = CreateInstance("Frame", {
        Name = "Sidebar",
        BackgroundColor3 = T.Surface,
        Size = UDim2.new(0, 130, 1, -46),
        Position = UDim2.new(0, 0, 0, 46),
        Parent = Window,
    })
    AddStroke(Sidebar, T.Border, 1)

    local TabList = CreateInstance("ScrollingFrame", {
        BackgroundTransparency = 1,
        Size = UDim2.new(1, 0, 1, 0),
        ScrollBarThickness = 0,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Parent = Sidebar,
    })
    AddPadding(TabList, 8, 8, 8, 8)
    local TabListLayout = CreateInstance("UIListLayout", {
        SortOrder = Enum.SortOrder.LayoutOrder,
        Padding = UDim.new(0, 4),
        Parent = TabList,
    })

    -- ── Área de conteúdo ──
    local ContentArea = CreateInstance("Frame", {
        Name = "ContentArea",
        BackgroundTransparency = 1,
        Size = UDim2.new(1, -130, 1, -46),
        Position = UDim2.new(0, 130, 0, 46),
        Parent = Window,
    })

    -- ══ OBJETO JANELA ══
    local WindowObj = {}
    WindowObj._theme = T
    WindowObj._tabs = {}
    WindowObj._activeTab = nil
    WindowObj._frame = Window

    -- ── ADICIONAR TAB ──
    function WindowObj:AddTab(label, icon)
        local tab = {}
        tab._elements = {}

        local isFirst = #self._tabs == 0

        -- Botão da sidebar
        local tabBtn = CreateInstance("TextButton", {
            BackgroundColor3 = isFirst and T.AccentDim or T.TabInactive,
            Size = UDim2.new(1, 0, 0, 34),
            Text = "",
            Parent = TabList,
        })
        AddCorner(tabBtn, 7)

        local tabLabel = CreateInstance("TextLabel", {
            BackgroundTransparency = 1,
            Size = UDim2.new(1, -10, 1, 0),
            Position = UDim2.new(0, 10, 0, 0),
            Text = label,
            TextColor3 = isFirst and T.Text or T.TextMuted,
            TextSize = 12,
            Font = isFirst and Enum.Font.GothamBold or Enum.Font.Gotham,
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = tabBtn,
        })

        -- Indicador lateral
        local indicator = CreateInstance("Frame", {
            BackgroundColor3 = T.Accent,
            Size = UDim2.new(0, 3, 0, 18),
            Position = UDim2.new(0, 0, 0.5, -9),
            BackgroundTransparency = isFirst and 0 or 1,
            Parent = tabBtn,
        })
        AddCorner(indicator, 3)

        -- Área de conteúdo da tab
        local tabContent = CreateInstance("ScrollingFrame", {
            BackgroundTransparency = 1,
            Size = UDim2.new(1, 0, 1, 0),
            ScrollBarThickness = 3,
            ScrollBarImageColor3 = T.Accent,
            CanvasSize = UDim2.new(0, 0, 0, 0),
            AutomaticCanvasSize = Enum.AutomaticSize.Y,
            Visible = isFirst,
            Parent = ContentArea,
        })
        AddPadding(tabContent, 10, 10, 14, 14)
        local contentLayout = CreateInstance("UIListLayout", {
            SortOrder = Enum.SortOrder.LayoutOrder,
            Padding = UDim.new(0, 7),
            Parent = tabContent,
        })

        tab._btn = tabBtn
        tab._label = tabLabel
        tab._indicator = indicator
        tab._content = tabContent
        tab._theme = T

        -- Lógica de troca de tab
        tabBtn.MouseButton1Click:Connect(function()
            if WindowObj._activeTab == tab then return end

            -- Desativa tab antiga
            if WindowObj._activeTab then
                local prev = WindowObj._activeTab
                Tween(prev._btn, { BackgroundColor3 = T.TabInactive }, 0.2)
                prev._label.TextColor3 = T.TextMuted
                prev._label.Font = Enum.Font.Gotham
                Tween(prev._indicator, { BackgroundTransparency = 1 }, 0.2)
                prev._content.Visible = false
            end

            -- Ativa tab nova
            Tween(tabBtn, { BackgroundColor3 = T.AccentDim }, 0.2)
            tabLabel.TextColor3 = T.Text
            tabLabel.Font = Enum.Font.GothamBold
            Tween(indicator, { BackgroundTransparency = 0 }, 0.2)
            tabContent.Visible = true
            WindowObj._activeTab = tab
        end)

        -- Hover
        tabBtn.MouseEnter:Connect(function()
            if WindowObj._activeTab ~= tab then
                Tween(tabBtn, { BackgroundColor3 = T.SurfaceHover }, 0.15)
            end
        end)
        tabBtn.MouseLeave:Connect(function()
            if WindowObj._activeTab ~= tab then
                Tween(tabBtn, { BackgroundColor3 = T.TabInactive }, 0.15)
            end
        end)

        if isFirst then WindowObj._activeTab = tab end
        table.insert(self._tabs, tab)

        -- ─────────────────────────────────
        --   ELEMENTOS DA TAB
        -- ─────────────────────────────────

        -- Seção / separador com título
        function tab:AddSection(label)
            local sectionFrame = CreateInstance("Frame", {
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 0, 24),
                Parent = tabContent,
            })
            local line = CreateInstance("Frame", {
                BackgroundColor3 = T.Border,
                Size = UDim2.new(1, 0, 0, 1),
                Position = UDim2.new(0, 0, 0.5, 0),
                Parent = sectionFrame,
            })
            local sectionLabel = CreateInstance("TextLabel", {
                BackgroundColor3 = T.Background,
                Size = UDim2.new(0, 0, 1, 0),
                AutomaticSize = Enum.AutomaticSize.X,
                Position = UDim2.new(0, 0, 0, 0),
                Text = "  " .. label .. "  ",
                TextColor3 = T.TextMuted,
                TextSize = 11,
                Font = Enum.Font.GothamBold,
                Parent = sectionFrame,
            })
        end

        -- ── BOTÃO ──
        function tab:AddButton(config)
            local cfg = config or {}
            local label = cfg.Label or "Botão"
            local cb = cfg.Callback or function() end

            local btn = CreateInstance("TextButton", {
                BackgroundColor3 = T.Surface,
                Size = UDim2.new(1, 0, 0, 38),
                Text = "",
                Parent = tabContent,
            })
            AddCorner(btn, 8)
            AddStroke(btn, T.Border, 1)

            local btnLabel = CreateInstance("TextLabel", {
                BackgroundTransparency = 1,
                Size = UDim2.new(1, -16, 1, 0),
                Position = UDim2.new(0, 14, 0, 0),
                Text = label,
                TextColor3 = T.Text,
                TextSize = 13,
                Font = Enum.Font.Gotham,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = btn,
            })

            -- Seta decorativa
            local arrow = CreateInstance("TextLabel", {
                BackgroundTransparency = 1,
                Size = UDim2.new(0, 20, 1, 0),
                Position = UDim2.new(1, -26, 0, 0),
                Text = "›",
                TextColor3 = T.Accent,
                TextSize = 18,
                Font = Enum.Font.GothamBold,
                Parent = btn,
            })

            btn.MouseEnter:Connect(function()
                Tween(btn, { BackgroundColor3 = T.SurfaceHover }, 0.15)
                Tween(arrow, { TextTransparency = 0 }, 0.15)
            end)
            btn.MouseLeave:Connect(function()
                Tween(btn, { BackgroundColor3 = T.Surface }, 0.15)
            end)
            btn.MouseButton1Down:Connect(function()
                Tween(btn, { BackgroundColor3 = T.AccentDim }, 0.1)
            end)
            btn.MouseButton1Up:Connect(function()
                Tween(btn, { BackgroundColor3 = T.SurfaceHover }, 0.1)
                cb()
            end)

            return btn
        end

        -- ── TOGGLE ──
        function tab:AddToggle(config)
            local cfg = config or {}
            local label = cfg.Label or "Toggle"
            local default = cfg.Default or false
            local cb = cfg.Callback or function() end

            local toggled = default

            local row = CreateInstance("Frame", {
                BackgroundColor3 = T.Surface,
                Size = UDim2.new(1, 0, 0, 38),
                Parent = tabContent,
            })
            AddCorner(row, 8)
            AddStroke(row, T.Border, 1)

            local rowLabel = CreateInstance("TextLabel", {
                BackgroundTransparency = 1,
                Size = UDim2.new(1, -60, 1, 0),
                Position = UDim2.new(0, 14, 0, 0),
                Text = label,
                TextColor3 = T.Text,
                TextSize = 13,
                Font = Enum.Font.Gotham,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = row,
            })

            -- Track do toggle
            local track = CreateInstance("Frame", {
                BackgroundColor3 = toggled and T.ToggleOn or T.ToggleOff,
                Size = UDim2.new(0, 40, 0, 22),
                Position = UDim2.new(1, -52, 0.5, -11),
                Parent = row,
            })
            AddCorner(track, 11)

            -- Bolinha
            local thumb = CreateInstance("Frame", {
                BackgroundColor3 = Color3.fromRGB(255,255,255),
                Size = UDim2.new(0, 16, 0, 16),
                Position = toggled
                    and UDim2.new(0, 21, 0.5, -8)
                    or  UDim2.new(0,  3, 0.5, -8),
                Parent = track,
            })
            AddCorner(thumb, 8)

            local function updateToggle()
                if toggled then
                    Tween(track, { BackgroundColor3 = T.ToggleOn }, 0.2)
                    Tween(thumb, { Position = UDim2.new(0, 21, 0.5, -8) }, 0.2)
                else
                    Tween(track, { BackgroundColor3 = T.ToggleOff }, 0.2)
                    Tween(thumb, { Position = UDim2.new(0, 3, 0.5, -8) }, 0.2)
                end
                cb(toggled)
            end

            local clickArea = CreateInstance("TextButton", {
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 1, 0),
                Text = "",
                Parent = row,
            })
            clickArea.MouseButton1Click:Connect(function()
                toggled = not toggled
                updateToggle()
            end)

            row.MouseEnter:Connect(function() Tween(row, { BackgroundColor3 = T.SurfaceHover }, 0.15) end)
            row.MouseLeave:Connect(function() Tween(row, { BackgroundColor3 = T.Surface }, 0.15) end)

            local obj = {}
            function obj:Set(val)
                toggled = val
                updateToggle()
            end
            function obj:Get() return toggled end
            return obj
        end

        -- ── SLIDER ──
        function tab:AddSlider(config)
            local cfg = config or {}
            local label = cfg.Label or "Slider"
            local minVal = cfg.Min or 0
            local maxVal = cfg.Max or 100
            local defaultVal = cfg.Default or minVal
            local decimals = cfg.Decimals or 0
            local cb = cfg.Callback or function() end

            local currentVal = defaultVal

            local container = CreateInstance("Frame", {
                BackgroundColor3 = T.Surface,
                Size = UDim2.new(1, 0, 0, 52),
                Parent = tabContent,
            })
            AddCorner(container, 8)
            AddStroke(container, T.Border, 1)
            AddPadding(container, 8, 8, 14, 14)

            local topRow = CreateInstance("Frame", {
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 0, 18),
                Parent = container,
            })

            local sliderLabel = CreateInstance("TextLabel", {
                BackgroundTransparency = 1,
                Size = UDim2.new(0.7, 0, 1, 0),
                Text = label,
                TextColor3 = T.Text,
                TextSize = 13,
                Font = Enum.Font.Gotham,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = topRow,
            })

            local valueLabel = CreateInstance("TextLabel", {
                BackgroundTransparency = 1,
                Size = UDim2.new(0.3, 0, 1, 0),
                Position = UDim2.new(0.7, 0, 0, 0),
                Text = tostring(currentVal),
                TextColor3 = T.Accent,
                TextSize = 13,
                Font = Enum.Font.GothamBold,
                TextXAlignment = Enum.TextXAlignment.Right,
                Parent = topRow,
            })

            -- Track
            local track = CreateInstance("Frame", {
                BackgroundColor3 = T.Border,
                Size = UDim2.new(1, 0, 0, 5),
                Position = UDim2.new(0, 0, 0, 26),
                Parent = container,
            })
            AddCorner(track, 3)

            local fill = CreateInstance("Frame", {
                BackgroundColor3 = T.SliderFill,
                Size = UDim2.new((currentVal - minVal) / (maxVal - minVal), 0, 1, 0),
                Parent = track,
            })
            AddCorner(fill, 3)

            -- Handle
            local handle = CreateInstance("Frame", {
                BackgroundColor3 = Color3.fromRGB(255,255,255),
                Size = UDim2.new(0, 14, 0, 14),
                AnchorPoint = Vector2.new(0.5, 0.5),
                Position = UDim2.new((currentVal - minVal)/(maxVal - minVal), 0, 0.5, 0),
                Parent = track,
            })
            AddCorner(handle, 7)

            local function updateSlider(val)
                val = math.clamp(val, minVal, maxVal)
                if decimals == 0 then
                    val = math.round(val)
                else
                    val = math.floor(val * (10^decimals) + 0.5) / (10^decimals)
                end
                currentVal = val
                local ratio = (val - minVal) / (maxVal - minVal)
                fill.Size = UDim2.new(ratio, 0, 1, 0)
                handle.Position = UDim2.new(ratio, 0, 0.5, 0)
                valueLabel.Text = tostring(val)
                cb(val)
            end

            local draggingSlider = false
            handle.InputBegan:Connect(function(inp)
                if inp.UserInputType == Enum.UserInputType.MouseButton1 then
                    draggingSlider = true
                    Tween(handle, { Size = UDim2.new(0, 18, 0, 18) }, 0.1)
                end
            end)

            track.InputBegan:Connect(function(inp)
                if inp.UserInputType == Enum.UserInputType.MouseButton1 then
                    draggingSlider = true
                    Tween(handle, { Size = UDim2.new(0, 18, 0, 18) }, 0.1)
                end
            end)

            UserInputService.InputEnded:Connect(function(inp)
                if inp.UserInputType == Enum.UserInputType.MouseButton1 then
                    draggingSlider = false
                    Tween(handle, { Size = UDim2.new(0, 14, 0, 14) }, 0.1)
                end
            end)

            RunService.RenderStepped:Connect(function()
                if draggingSlider then
                    local trackPos = track.AbsolutePosition.X
                    local trackSize = track.AbsoluteSize.X
                    local mouseX = Mouse.X
                    local ratio = math.clamp((mouseX - trackPos) / trackSize, 0, 1)
                    updateSlider(minVal + ratio * (maxVal - minVal))
                end
            end)

            local obj = {}
            function obj:Set(val) updateSlider(val) end
            function obj:Get() return currentVal end
            return obj
        end

        -- ── INPUT ──
        function tab:AddInput(config)
            local cfg = config or {}
            local label = cfg.Label or "Input"
            local placeholder = cfg.Placeholder or "Digite aqui..."
            local cb = cfg.Callback or function() end
            local cbChange = cfg.OnChange or nil

            local container = CreateInstance("Frame", {
                BackgroundColor3 = T.Surface,
                Size = UDim2.new(1, 0, 0, 58),
                Parent = tabContent,
            })
            AddCorner(container, 8)
            AddStroke(container, T.Border, 1)

            local inputLabel = CreateInstance("TextLabel", {
                BackgroundTransparency = 1,
                Size = UDim2.new(1, -16, 0, 20),
                Position = UDim2.new(0, 14, 0, 8),
                Text = label,
                TextColor3 = T.TextMuted,
                TextSize = 11,
                Font = Enum.Font.GothamBold,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = container,
            })

            local inputBg = CreateInstance("Frame", {
                BackgroundColor3 = T.Background,
                Size = UDim2.new(1, -16, 0, 24),
                Position = UDim2.new(0, 8, 0, 26),
                Parent = container,
            })
            AddCorner(inputBg, 6)
            AddStroke(inputBg, T.Border, 1)

            local textBox = CreateInstance("TextBox", {
                BackgroundTransparency = 1,
                Size = UDim2.new(1, -16, 1, 0),
                Position = UDim2.new(0, 8, 0, 0),
                PlaceholderText = placeholder,
                PlaceholderColor3 = T.TextDisabled,
                Text = "",
                TextColor3 = T.Text,
                TextSize = 12,
                Font = Enum.Font.Gotham,
                TextXAlignment = Enum.TextXAlignment.Left,
                ClearTextOnFocus = false,
                Parent = inputBg,
            })

            textBox.Focused:Connect(function()
                Tween(inputBg, { BackgroundColor3 = T.SurfaceHover }, 0.15)
                AddStroke(inputBg, T.Accent, 1)
            end)
            textBox.FocusLost:Connect(function(enter)
                Tween(inputBg, { BackgroundColor3 = T.Background }, 0.15)
                AddStroke(inputBg, T.Border, 1)
                if enter then cb(textBox.Text) end
            end)
            if cbChange then
                textBox:GetPropertyChangedSignal("Text"):Connect(function()
                    cbChange(textBox.Text)
                end)
            end

            local obj = {}
            function obj:Get() return textBox.Text end
            function obj:Set(val) textBox.Text = val end
            return obj
        end

        return tab
    end

    return WindowObj
end

-- ══════════════════════════════════════════
--       RETORNAR LIBRARY
-- ══════════════════════════════════════════
return NexusUI

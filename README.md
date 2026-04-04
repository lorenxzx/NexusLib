--[[
    ███╗   ██╗███████╗██╗  ██╗██╗   ██╗███████╗    ██╗   ██╗██╗
    ████╗  ██║██╔════╝╚██╗██╔╝██║   ██║██╔════╝    ██║   ██║██║
    ██╔██╗ ██║█████╗   ╚███╔╝ ██║   ██║███████╗    ██║   ██║██║
    ██║╚██╗██║██╔══╝   ██╔██╗ ██║   ██║╚════██║    ██║   ██║██║
    ██║ ╚████║███████╗██╔╝ ██╗╚██████╔╝███████║    ╚██████╔╝██║
    ╚═╝  ╚═══╝╚══════╝╚═╝  ╚═╝ ╚═════╝ ╚══════╝     ╚═════╝ ╚═╝

    nova 
]]

local NexusUI  = {}
NexusUI.__index = NexusUI

-- ═══════════════════════════════════════════════════
--  SERVIÇOS
-- ═══════════════════════════════════════════════════
local Players          = game:GetService("Players")
local TweenService     = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService       = game:GetService("RunService")
local CoreGui          = game:GetService("CoreGui")
local LP               = Players.LocalPlayer
local Mouse            = LP:GetMouse()

-- ═══════════════════════════════════════════════════
--  TEMAS
-- ═══════════════════════════════════════════════════
local Themes = {
    Dark = {
        WinBg         = Color3.fromRGB(28,  28,  30 ),
        TitleBg       = Color3.fromRGB(32,  32,  35 ),
        SidebarBg     = Color3.fromRGB(30,  30,  33 ),
        ContentBg     = Color3.fromRGB(24,  24,  26 ),
        Border        = Color3.fromRGB(52,  52,  58 ),
        BorderSubtle  = Color3.fromRGB(42,  42,  46 ),
        Card          = Color3.fromRGB(36,  36,  40 ),
        CardHover     = Color3.fromRGB(44,  44,  50 ),
        CardPress     = Color3.fromRGB(48,  46,  72 ),
        CardBorder    = Color3.fromRGB(50,  50,  56 ),
        Accent        = Color3.fromRGB(99,  102, 241),
        AccentText    = Color3.fromRGB(150, 152, 255),
        AccentDim     = Color3.fromRGB(38,  38,  76 ),
        Text          = Color3.fromRGB(218, 218, 222),
        TextMuted     = Color3.fromRGB(118, 118, 130),
        TextDisabled  = Color3.fromRGB(62,  62,  70 ),
        Success       = Color3.fromRGB(52,  211, 153),
        Warning       = Color3.fromRGB(251, 191, 36 ),
        Error         = Color3.fromRGB(248, 113, 113),
        Info          = Color3.fromRGB(99,  102, 241),
        ToggleOn      = Color3.fromRGB(99,  102, 241),
        ToggleOff     = Color3.fromRGB(52,  52,  60 ),
        SliderThumb   = Color3.fromRGB(255, 255, 255),  -- branco no dark
        TabActiveBg   = Color3.fromRGB(38,  38,  76 ),
        TabHoverBg    = Color3.fromRGB(40,  40,  46 ),
    },
    Light = {
        WinBg         = Color3.fromRGB(250, 250, 252),
        TitleBg       = Color3.fromRGB(244, 244, 248),
        SidebarBg     = Color3.fromRGB(246, 246, 250),
        ContentBg     = Color3.fromRGB(238, 238, 244),
        Border        = Color3.fromRGB(200, 200, 212),
        BorderSubtle  = Color3.fromRGB(216, 216, 224),
        Card          = Color3.fromRGB(255, 255, 255),
        CardHover     = Color3.fromRGB(244, 244, 252),
        CardPress     = Color3.fromRGB(230, 228, 255),
        CardBorder    = Color3.fromRGB(210, 210, 220),
        Accent        = Color3.fromRGB(79,  70,  229),
        AccentText    = Color3.fromRGB(79,  70,  229),
        AccentDim     = Color3.fromRGB(220, 218, 255),
        Text          = Color3.fromRGB(18,  18,  24 ),
        TextMuted     = Color3.fromRGB(100, 100, 118),
        TextDisabled  = Color3.fromRGB(170, 170, 185),
        Success       = Color3.fromRGB(16,  185, 129),
        Warning       = Color3.fromRGB(217, 119, 6  ),
        Error         = Color3.fromRGB(220, 38,  38 ),
        Info          = Color3.fromRGB(79,  70,  229),
        ToggleOn      = Color3.fromRGB(79,  70,  229),
        ToggleOff     = Color3.fromRGB(200, 200, 215),
        SliderThumb   = Color3.fromRGB(28,  28,  35 ),  -- escuro no light
        TabActiveBg   = Color3.fromRGB(220, 218, 255),
        TabHoverBg    = Color3.fromRGB(234, 234, 244),
    },
}

-- ═══════════════════════════════════════════════════
--  HELPERS
-- ═══════════════════════════════════════════════════
local function Tween(obj, props, t, style, dir)
    TweenService:Create(obj,
        TweenInfo.new(t or 0.18, style or Enum.EasingStyle.Quart,
            dir or Enum.EasingDirection.Out), props):Play()
end

--[[
    MakeRoundedFrame — THE corner fix.
    Retorna (outer, inner).
    • outer: posicione/dimensione este; NÃO adicione filhos aqui.
    • inner: coloque todos os filhos aqui.
    Por que funciona: UIStroke fica no outer (sem clip), UICorner+ClipsDescendants
    ficam no inner (sem stroke). Nunca há sangramento nos cantos.
]]
local function MakeRoundedFrame(parent, bgColor, radius, borderColor, borderThick)
    radius      = radius      or 8
    borderThick = borderThick or 1

    local outer = Instance.new("Frame")
    outer.BackgroundTransparency = 1
    outer.BorderSizePixel        = 0
    if parent then outer.Parent = parent end

    do
        local oc = Instance.new("UICorner")
        oc.CornerRadius = UDim.new(0, radius)
        oc.Parent = outer
    end

    if borderColor then
        local os = Instance.new("UIStroke")
        os.Color           = borderColor
        os.Thickness       = borderThick
        os.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
        os.Parent          = outer
    end

    local inner = Instance.new("Frame")
    inner.BackgroundColor3 = bgColor or Color3.fromRGB(30,30,30)
    inner.BorderSizePixel  = 0
    inner.Size             = UDim2.new(1, 0, 1, 0)
    inner.ClipsDescendants = true
    inner.Parent           = outer

    do
        local ic = Instance.new("UICorner")
        ic.CornerRadius = UDim.new(0, radius)
        ic.Parent = inner
    end

    return outer, inner
end

local function AddPadding(parent, top, bottom, left, right)
    local p = Instance.new("UIPadding")
    p.PaddingTop    = UDim.new(0, top    or 8)
    p.PaddingBottom = UDim.new(0, bottom or 8)
    p.PaddingLeft   = UDim.new(0, left   or 8)
    p.PaddingRight  = UDim.new(0, right  or 8)
    p.Parent = parent
    return p
end

local function MakeDraggable(rootFrame, handle)
    local dragging, dragInput, dragStart, startPos = false, nil, nil, nil
    handle = handle or rootFrame
    handle.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging  = true
            dragStart = inp.Position
            startPos  = rootFrame.Position
            inp.Changed:Connect(function()
                if inp.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)
    handle.InputChanged:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseMovement then dragInput = inp end
    end)
    UserInputService.InputChanged:Connect(function(inp)
        if inp == dragInput and dragging then
            local d = inp.Position - dragStart
            rootFrame.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + d.X,
                startPos.Y.Scale, startPos.Y.Offset + d.Y)
        end
    end)
end

local function resolveIcon(raw)
    if not raw or raw == "" then return nil end
    local s = tostring(raw)
    if s:lower():find("rbxassetid://") then return s end
    if s:match("^%d+$")               then return "rbxassetid://" .. s end
    return s
end

-- ═══════════════════════════════════════════════════
--  SCREENGUI
-- ═══════════════════════════════════════════════════
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name           = "NexusUI"
ScreenGui.ResetOnSpawn   = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.IgnoreGuiInset = true

local ok = pcall(function() ScreenGui.Parent = CoreGui end)
if not ok then ScreenGui.Parent = LP:WaitForChild("PlayerGui") end

-- Container de notificações (canto inferior direito)
local NotifContainer = Instance.new("Frame")
NotifContainer.BackgroundTransparency = 1
NotifContainer.AnchorPoint            = Vector2.new(1, 1)
NotifContainer.Position               = UDim2.new(1, -16, 1, -16)
NotifContainer.Size                   = UDim2.new(0, 340, 1, -32)
NotifContainer.Parent                 = ScreenGui

local notifLayout = Instance.new("UIListLayout")
notifLayout.SortOrder         = Enum.SortOrder.LayoutOrder
notifLayout.VerticalAlignment = Enum.VerticalAlignment.Bottom
notifLayout.Padding           = UDim.new(0, 8)
notifLayout.Parent            = NotifContainer

-- ═══════════════════════════════════════════════════
--  NOTIFICAÇÕES
-- ═══════════════════════════════════════════════════
function NexusUI:Notify(config)
    local cfg      = config or {}
    local title    = cfg.Title    or "Notificação"
    local message  = cfg.Message  or ""
    local duration = cfg.Duration or 4
    local ntype    = cfg.Type     or "Info"
    -- Respeita o tema: "Dark" ou "Light"
    local T        = Themes[cfg.Theme or "Dark"] or Themes.Dark

    local typeColor = { Info=T.Info, Success=T.Success, Warning=T.Warning, Error=T.Error }
    local accent    = typeColor[ntype] or T.Info

    -- outer/inner: sem vazar cantos
    local notifOuter, notifInner = MakeRoundedFrame(
        NotifContainer,
        T.TitleBg,
        8,
        T.Border,
        1
    )
    notifOuter.Size = UDim2.new(1, 0, 0, 66)

    -- Barra accent lateral (UICorner apenas uma vez)
    local bar = Instance.new("Frame")
    bar.BackgroundColor3 = accent
    bar.Size             = UDim2.new(0, 3, 1, 0)
    bar.BorderSizePixel  = 0
    bar.Parent           = notifInner
    do local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, 3); c.Parent = bar end

    local content = Instance.new("Frame")
    content.BackgroundTransparency = 1
    content.Size     = UDim2.new(1, -14, 1, 0)
    content.Position = UDim2.new(0, 12, 0, 0)
    content.Parent   = notifInner
    AddPadding(content, 10, 8, 6, 10)

    local icons = {Info="ℹ", Success="✓", Warning="⚠", Error="✕"}
    do
        local il = Instance.new("TextLabel")
        il.BackgroundTransparency = 1
        il.Size       = UDim2.new(0,18,0,18)
        il.Position   = UDim2.new(0,0,0,1)
        il.Text       = icons[ntype] or "ℹ"
        il.TextColor3 = accent
        il.TextSize   = 14
        il.Font       = Enum.Font.GothamBold
        il.Parent     = content
    end

    do
        local tl = Instance.new("TextLabel")
        tl.BackgroundTransparency = 1
        tl.Size           = UDim2.new(1,-24,0,18)
        tl.Position       = UDim2.new(0,24,0,0)
        tl.Text           = title
        tl.TextColor3     = T.Text
        tl.TextSize       = 13
        tl.Font           = Enum.Font.GothamBold
        tl.TextXAlignment = Enum.TextXAlignment.Left
        tl.Parent         = content
    end

    do
        local ml = Instance.new("TextLabel")
        ml.BackgroundTransparency = 1
        ml.Size           = UDim2.new(1,0,0,0)
        ml.Position       = UDim2.new(0,0,0,22)
        ml.AutomaticSize  = Enum.AutomaticSize.Y
        ml.Text           = message
        ml.TextColor3     = T.TextMuted
        ml.TextSize       = 12
        ml.Font           = Enum.Font.Gotham
        ml.TextXAlignment = Enum.TextXAlignment.Left
        ml.TextWrapped    = true
        ml.Parent         = content
    end

    -- Barra de progresso (sem cantos bugados: é dentro do inner já clipado)
    local pgBg = Instance.new("Frame")
    pgBg.BackgroundColor3 = T.Border
    pgBg.Size             = UDim2.new(1, 0, 0, 2)
    pgBg.Position         = UDim2.new(0, 0, 1, -2)
    pgBg.BorderSizePixel  = 0
    pgBg.Parent           = notifInner

    local pgFill = Instance.new("Frame")
    pgFill.BackgroundColor3 = accent
    pgFill.Size             = UDim2.new(1, 0, 1, 0)
    pgFill.BorderSizePixel  = 0
    pgFill.Parent           = pgBg

    -- Slide-in
    notifOuter.Position = UDim2.new(0.1, 0, 0, 0)
    Tween(notifOuter, {Position = UDim2.new(0, 0, 0, 0)}, 0.26)
    Tween(pgFill, {Size = UDim2.new(0, 0, 1, 0)}, duration, Enum.EasingStyle.Linear)

    task.delay(duration, function()
        Tween(notifOuter, {Position = UDim2.new(0.1, 0, 0, 0)}, 0.22)
        task.wait(0.26)
        notifOuter:Destroy()
    end)

    return notifOuter
end

-- ═══════════════════════════════════════════════════
--  CRIAR JANELA
-- ═══════════════════════════════════════════════════
function NexusUI:CreateWindow(config)
    local cfg       = config or {}
    local title     = cfg.Title    or "NexusUI"
    local subtitle  = cfg.SubTitle or "v2.0"
    local themeName = cfg.Theme    or "Dark"
    local winSize   = cfg.Size     or UDim2.new(0, 580, 0, 400)
    local winPos    = cfg.Position or UDim2.new(0.5, -290, 0.5, -200)
    local T         = Themes[themeName] or Themes.Dark

    -- ── Janela: outer (stroke) / inner (clip) ──────────
    local winOuter, winInner = MakeRoundedFrame(ScreenGui, T.WinBg, 8, T.Border, 1)
    winOuter.Name     = "NexusWindow"
    winOuter.Size     = winSize
    winOuter.Position = winPos

    -- ── TitleBar ────────────────────────────────────────
    -- Dentro do winInner = já clipado pelo UICorner do inner.
    -- NÃO precisa de UICorner próprio (os cantos superiores
    -- serão arredondados pelo clip do pai).
    local titleBar = Instance.new("Frame")
    titleBar.BackgroundColor3 = T.TitleBg
    titleBar.Size             = UDim2.new(1, 0, 0, 42)
    titleBar.BorderSizePixel  = 0
    titleBar.Parent           = winInner

    -- UICorner para arredondar os cantos superiores (combinam com a janela)
    do local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, 8); c.Parent = titleBar end

    -- Frame de cobertura embaixo: tapa os cantos inferiores arredondados, deixando-os retos
    do
        local cover = Instance.new("Frame")
        cover.BackgroundColor3 = T.TitleBg
        cover.Size             = UDim2.new(1, 0, 0, 8)
        cover.Position         = UDim2.new(0, 0, 1, -8)
        cover.BorderSizePixel  = 0
        cover.Parent           = titleBar
    end

    -- divisor inferior
    do
        local d = Instance.new("Frame")
        d.BackgroundColor3 = T.Border
        d.Size             = UDim2.new(1, 0, 0, 1)
        d.Position         = UDim2.new(0, 0, 1, -1)
        d.BorderSizePixel  = 0
        d.ZIndex           = 3
        d.Parent           = titleBar
    end

    -- Pill accent
    do
        local pill = Instance.new("Frame")
        pill.BackgroundColor3 = T.Accent
        pill.Size             = UDim2.new(0, 3, 0, 20)
        pill.Position         = UDim2.new(0, 14, 0.5, -10)
        pill.BorderSizePixel  = 0
        pill.Parent           = titleBar
        local c = Instance.new("UICorner"); c.CornerRadius=UDim.new(0,3); c.Parent=pill
    end

    do
        local tl = Instance.new("TextLabel")
        tl.BackgroundTransparency = 1
        tl.Size           = UDim2.new(0, 220, 0, 18)
        tl.Position       = UDim2.new(0, 24, 0, 7)
        tl.Text           = title
        tl.TextColor3     = T.Text
        tl.TextSize       = 14
        tl.Font           = Enum.Font.GothamBold
        tl.TextXAlignment = Enum.TextXAlignment.Left
        tl.Parent         = titleBar
    end
    do
        local sl = Instance.new("TextLabel")
        sl.BackgroundTransparency = 1
        sl.Size           = UDim2.new(0, 220, 0, 13)
        sl.Position       = UDim2.new(0, 24, 0, 26)
        sl.Text           = subtitle
        sl.TextColor3     = T.TextMuted
        sl.TextSize       = 11
        sl.Font           = Enum.Font.Gotham
        sl.TextXAlignment = Enum.TextXAlignment.Left
        sl.Parent         = titleBar
    end

    -- Botões Windows-style (×  □  ─)
    local function makeTitleBtn(sym, xOff)
        local btn = Instance.new("TextButton")
        btn.BackgroundTransparency = 1
        btn.BorderSizePixel        = 0
        btn.Size                   = UDim2.new(0, 40, 1, 0)
        btn.Position               = UDim2.new(1, xOff, 0, 0)
        btn.Text                   = sym
        btn.TextColor3             = T.TextMuted
        btn.TextSize               = 13
        btn.Font                   = Enum.Font.Gotham
        btn.Parent                 = titleBar
        btn.MouseEnter:Connect(function()
            Tween(btn, {TextColor3 = T.Text}, 0.1)
        end)
        btn.MouseLeave:Connect(function()
            Tween(btn, {TextColor3 = T.TextMuted}, 0.1)
        end)
        return btn
    end

    local closeBtn    = makeTitleBtn("✕", -40)
    local maximizeBtn = makeTitleBtn("□", -80)
    local minimizeBtn = makeTitleBtn("─", -120)

    closeBtn.MouseButton1Click:Connect(function()
        Tween(winOuter, {Size=UDim2.new(0, winOuter.AbsoluteSize.X, 0, 0)}, 0.2)
        task.wait(0.22)
        winOuter:Destroy()
    end)

    -- ── Pill de minimizado (aparece no lugar da janela) ──
    local pillOuter, pillInner = MakeRoundedFrame(ScreenGui, T.TitleBg, 18, T.Border, 1)
    pillOuter.Size    = UDim2.new(0, 220, 0, 38)
    pillOuter.Position = winPos
    pillOuter.Visible  = false

    -- Bloco accent esquerdo
    do
        local ab = Instance.new("Frame")
        ab.BackgroundColor3 = T.Accent
        ab.Size             = UDim2.new(0, 4, 1, -12)
        ab.Position         = UDim2.new(0, 10, 0.5, -((38-12)/2))
        ab.BorderSizePixel  = 0
        ab.Parent           = pillInner
        local c = Instance.new("UICorner"); c.CornerRadius=UDim.new(0,2); c.Parent=ab
    end

    do
        local pt = Instance.new("TextLabel")
        pt.BackgroundTransparency = 1
        pt.Size           = UDim2.new(1, -60, 1, 0)
        pt.Position       = UDim2.new(0, 22, 0, 0)
        pt.Text           = title .. " · " .. subtitle
        pt.TextColor3     = T.Text
        pt.TextSize       = 12
        pt.Font           = Enum.Font.GothamBold
        pt.TextXAlignment = Enum.TextXAlignment.Left
        pt.TextTruncate   = Enum.TextTruncate.AtEnd
        pt.Parent         = pillInner
    end

    -- Botão de restaurar
    local restoreBtn = Instance.new("TextButton")
    restoreBtn.BackgroundTransparency = 1
    restoreBtn.Size       = UDim2.new(0, 32, 1, 0)
    restoreBtn.Position   = UDim2.new(1, -34, 0, 0)
    restoreBtn.Text       = "⤢"
    restoreBtn.TextColor3 = T.TextMuted
    restoreBtn.TextSize   = 14
    restoreBtn.Font       = Enum.Font.Gotham
    restoreBtn.Parent     = pillInner
    restoreBtn.MouseEnter:Connect(function() Tween(restoreBtn,{TextColor3=T.Text},0.1) end)
    restoreBtn.MouseLeave:Connect(function() Tween(restoreBtn,{TextColor3=T.TextMuted},0.1) end)

    MakeDraggable(pillOuter, pillInner)

    local minimized = false
    minimizeBtn.MouseButton1Click:Connect(function()
        minimized = true
        -- Sincroniza posição do pill com a janela antes de mostrar
        pillOuter.Position = winOuter.Position
        -- Anima janela saindo (encolhe para o pill)
        Tween(winOuter, {Size=UDim2.new(0,220,0,0), Position=winOuter.Position}, 0.22)
        task.wait(0.1)
        winOuter.Visible = false
        winOuter.Size    = winSize  -- reseta tamanho para restaurar depois
        pillOuter.Size   = UDim2.new(0, 0, 0, 38)
        pillOuter.Visible = true
        Tween(pillOuter, {Size=UDim2.new(0,220,0,38)}, 0.22)
    end)

    restoreBtn.MouseButton1Click:Connect(function()
        minimized = false
        local pillPos = pillOuter.Position
        pillOuter.Visible = false
        winOuter.Position = pillPos
        winOuter.Visible  = true
        winOuter.Size     = UDim2.new(0, winSize.X.Offset, 0, 0)
        Tween(winOuter, {Size=winSize}, 0.25)
    end)

    local maxed, preMaxSize, preMaxPos = false, winSize, winPos
    maximizeBtn.MouseButton1Click:Connect(function()
        maxed = not maxed
        if maxed then
            preMaxSize = winOuter.Size; preMaxPos = winOuter.Position
            Tween(winOuter, {Size=UDim2.new(1,-32,1,-32), Position=UDim2.new(0,16,0,16)}, 0.22)
        else
            Tween(winOuter, {Size=preMaxSize, Position=preMaxPos}, 0.22)
        end
    end)

    MakeDraggable(winOuter, titleBar)

    -- ── Sidebar ─────────────────────────────────────────
    local sidebar = Instance.new("Frame")
    sidebar.BackgroundColor3 = T.SidebarBg
    sidebar.Size             = UDim2.new(0, 150, 1, -42)
    sidebar.Position         = UDim2.new(0, 0, 0, 42)
    sidebar.BorderSizePixel  = 0
    sidebar.Parent           = winInner

    -- UICorner para arredondar o canto inferior esquerdo (combina com a janela)
    do local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, 8); c.Parent = sidebar end

    -- Cobertura no topo: tapa os cantos superiores arredondados (o topo fica embaixo da titlebar)
    do
        local cover = Instance.new("Frame")
        cover.BackgroundColor3 = T.SidebarBg
        cover.Size             = UDim2.new(1, 0, 0, 8)
        cover.Position         = UDim2.new(0, 0, 0, 0)
        cover.BorderSizePixel  = 0
        cover.ZIndex           = 2
        cover.Parent           = sidebar
    end

    -- Cobertura na direita: tapa os cantos direitos arredondados (direita fica colada no conteúdo)
    do
        local cover = Instance.new("Frame")
        cover.BackgroundColor3 = T.SidebarBg
        cover.Size             = UDim2.new(0, 8, 1, 0)
        cover.Position         = UDim2.new(1, -8, 0, 0)
        cover.BorderSizePixel  = 0
        cover.ZIndex           = 2
        cover.Parent           = sidebar
    end

    do -- divisor direito
        local d = Instance.new("Frame")
        d.BackgroundColor3 = T.Border
        d.Size             = UDim2.new(0, 1, 1, 0)
        d.Position         = UDim2.new(1, -1, 0, 0)
        d.BorderSizePixel  = 0
        d.ZIndex           = 3
        d.Parent           = sidebar
    end

    local tabScroll = Instance.new("ScrollingFrame")
    tabScroll.BackgroundTransparency = 1
    tabScroll.Size                   = UDim2.new(1, 0, 1, 0)
    tabScroll.ScrollBarThickness     = 0
    tabScroll.CanvasSize             = UDim2.new(0,0,0,0)
    tabScroll.AutomaticCanvasSize    = Enum.AutomaticSize.Y
    tabScroll.Parent                 = sidebar
    AddPadding(tabScroll, 8, 8, 8, 8)

    local tabLayout = Instance.new("UIListLayout")
    tabLayout.SortOrder = Enum.SortOrder.LayoutOrder
    tabLayout.Padding   = UDim.new(0, 3)
    tabLayout.Parent    = tabScroll

    -- ── Content Area ────────────────────────────────────
    local contentArea = Instance.new("Frame")
    contentArea.BackgroundColor3 = T.ContentBg
    contentArea.Size             = UDim2.new(1, -150, 1, -42)
    contentArea.Position         = UDim2.new(0, 150, 0, 42)
    contentArea.BorderSizePixel  = 0
    contentArea.Parent           = winInner

    -- UICorner para arredondar o canto inferior direito (combina com a janela)
    do local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, 8); c.Parent = contentArea end

    -- Cobertura no topo: tapa o canto superior direito (fica sob a titlebar)
    do
        local cover = Instance.new("Frame")
        cover.BackgroundColor3 = T.ContentBg
        cover.Size             = UDim2.new(1, 0, 0, 8)
        cover.Position         = UDim2.new(0, 0, 0, 0)
        cover.BorderSizePixel  = 0
        cover.Parent           = contentArea
    end

    -- Cobertura na esquerda: tapa o canto inferior esquerdo (fica colado na sidebar)
    do
        local cover = Instance.new("Frame")
        cover.BackgroundColor3 = T.ContentBg
        cover.Size             = UDim2.new(0, 8, 1, 0)
        cover.Position         = UDim2.new(0, 0, 0, 0)
        cover.BorderSizePixel  = 0
        cover.Parent           = contentArea
    end

    -- ── Objeto Window ────────────────────────────────────
    local Win          = {}
    Win._theme         = T
    Win._tabs          = {}
    Win._activeTab     = nil
    Win._outer         = winOuter

    -- ════════════════════════════════════════
    --  ADD TAB
    -- ════════════════════════════════════════
    function Win:AddTab(label, icon)
        local tab     = {}
        local iconId  = resolveIcon(icon)
        local isFirst = (#self._tabs == 0)

        -- Botão sidebar
        local btn = Instance.new("TextButton")
        btn.BackgroundColor3       = isFirst and T.TabActiveBg or Color3.fromRGB(0,0,0)
        btn.BackgroundTransparency = isFirst and 0 or 1
        btn.Size                   = UDim2.new(1, 0, 0, 34)
        btn.Text                   = ""
        btn.BorderSizePixel        = 0
        btn.Parent                 = tabScroll
        do local c=Instance.new("UICorner"); c.CornerRadius=UDim.new(0,6); c.Parent=btn end

        -- Indicador
        local indicator = Instance.new("Frame")
        indicator.BackgroundColor3       = T.Accent
        indicator.Size                   = UDim2.new(0, 3, 0, 16)
        indicator.Position               = UDim2.new(0, 0, 0.5, -8)
        indicator.BorderSizePixel        = 0
        indicator.BackgroundTransparency = isFirst and 0 or 1
        indicator.Parent                 = btn
        do local c=Instance.new("UICorner"); c.CornerRadius=UDim.new(0,3); c.Parent=indicator end

        -- Ícone (opcional)
        local iconImg
        local lblOffX = 10
        if iconId then
            iconImg = Instance.new("ImageLabel")
            iconImg.BackgroundTransparency = 1
            iconImg.Size        = UDim2.new(0, 15, 0, 15)
            iconImg.Position    = UDim2.new(0, 10, 0.5, -7)
            iconImg.Image       = iconId
            iconImg.ImageColor3 = isFirst and T.Text or T.TextMuted
            iconImg.ScaleType   = Enum.ScaleType.Fit
            iconImg.Parent      = btn
            lblOffX = 30
        end

        local btnLabel = Instance.new("TextLabel")
        btnLabel.BackgroundTransparency = 1
        btnLabel.Size           = UDim2.new(1, -(lblOffX+4), 1, 0)
        btnLabel.Position       = UDim2.new(0, lblOffX, 0, 0)
        btnLabel.Text           = label
        btnLabel.TextColor3     = isFirst and T.Text or T.TextMuted
        btnLabel.TextSize       = 12
        btnLabel.Font           = isFirst and Enum.Font.GothamBold or Enum.Font.Gotham
        btnLabel.TextXAlignment = Enum.TextXAlignment.Left
        btnLabel.Parent         = btn

        -- Scroll de conteúdo
        local scroll = Instance.new("ScrollingFrame")
        scroll.BackgroundTransparency = 1
        scroll.Size                   = UDim2.new(1, 0, 1, 0)
        scroll.ScrollBarThickness     = 3
        scroll.ScrollBarImageColor3   = T.Accent
        scroll.CanvasSize             = UDim2.new(0,0,0,0)
        scroll.AutomaticCanvasSize    = Enum.AutomaticSize.Y
        scroll.Visible                = isFirst
        scroll.Parent                 = contentArea
        AddPadding(scroll, 16, 16, 16, 16)

        do
            local sl = Instance.new("UIListLayout")
            sl.SortOrder = Enum.SortOrder.LayoutOrder
            sl.Padding   = UDim.new(0, 7)
            sl.Parent    = scroll
        end

        -- Cabeçalho (nome da tab em destaque, estilo Fluent)
        do
            local h = Instance.new("TextLabel")
            h.BackgroundTransparency = 1
            h.Size           = UDim2.new(1, 0, 0, 28)
            h.Text           = label
            h.TextColor3     = T.Text
            h.TextSize       = 20
            h.Font           = Enum.Font.GothamBold
            h.TextXAlignment = Enum.TextXAlignment.Left
            h.LayoutOrder    = -9999
            h.Parent         = scroll
        end
        do
            local div = Instance.new("Frame")
            div.BackgroundColor3 = T.Border
            div.Size             = UDim2.new(1, 0, 0, 1)
            div.BorderSizePixel  = 0
            div.LayoutOrder      = -9998
            div.Parent           = scroll
        end

        tab._btn       = btn
        tab._label     = btnLabel
        tab._indicator = indicator
        tab._icon      = iconImg
        tab._scroll    = scroll
        tab._theme     = T

        -- Troca de tab
        btn.MouseButton1Click:Connect(function()
            if Win._activeTab == tab then return end
            if Win._activeTab then
                local p = Win._activeTab
                Tween(p._btn, {BackgroundTransparency=1}, 0.15)
                Tween(p._indicator, {BackgroundTransparency=1}, 0.15)
                p._label.TextColor3 = T.TextMuted
                p._label.Font       = Enum.Font.Gotham
                if p._icon then Tween(p._icon, {ImageColor3=T.TextMuted}, 0.15) end
                p._scroll.Visible   = false
            end
            Tween(btn, {BackgroundColor3=T.TabActiveBg, BackgroundTransparency=0}, 0.15)
            Tween(indicator, {BackgroundTransparency=0}, 0.15)
            btnLabel.TextColor3 = T.Text
            btnLabel.Font       = Enum.Font.GothamBold
            if iconImg then Tween(iconImg, {ImageColor3=T.Text}, 0.15) end
            scroll.Visible  = true
            Win._activeTab  = tab
        end)

        btn.MouseEnter:Connect(function()
            if Win._activeTab == tab then return end
            Tween(btn, {BackgroundColor3=T.TabHoverBg, BackgroundTransparency=0}, 0.1)
        end)
        btn.MouseLeave:Connect(function()
            if Win._activeTab == tab then return end
            Tween(btn, {BackgroundTransparency=1}, 0.1)
        end)

        if isFirst then Win._activeTab = tab end
        table.insert(self._tabs, tab)

        -- ────────────────────────────────────────
        --  SECTION
        -- ────────────────────────────────────────
        function tab:AddSection(sLabel)
            local sf = Instance.new("Frame")
            sf.BackgroundTransparency = 1
            sf.Size   = UDim2.new(1, 0, 0, 22)
            sf.Parent = scroll
            do
                local line = Instance.new("Frame")
                line.BackgroundColor3 = T.Border
                line.Size             = UDim2.new(1,0,0,1)
                line.Position         = UDim2.new(0,0,0.5,0)
                line.BorderSizePixel  = 0
                line.Parent           = sf
            end
            do
                local sl = Instance.new("TextLabel")
                sl.BackgroundColor3 = T.ContentBg
                sl.Size             = UDim2.new(0,0,1,0)
                sl.AutomaticSize    = Enum.AutomaticSize.X
                sl.Text             = "  "..sLabel.."  "
                sl.TextColor3       = T.TextMuted
                sl.TextSize         = 11
                sl.Font             = Enum.Font.GothamBold
                sl.BorderSizePixel  = 0
                sl.Parent           = sf
            end
        end

        -- ────────────────────────────────────────
        --  BUTTON
        -- ────────────────────────────────────────
        function tab:AddButton(cfg)
            cfg = cfg or {}
            local lbl = cfg.Label    or "Botão"
            local cb  = cfg.Callback or function() end

            local outer, inner = MakeRoundedFrame(scroll, T.Card, 6, T.CardBorder, 1)
            outer.Size = UDim2.new(1, 0, 0, 38)

            do
                local lt = Instance.new("TextLabel")
                lt.BackgroundTransparency = 1
                lt.Size           = UDim2.new(1,-38,1,0)
                lt.Position       = UDim2.new(0,14,0,0)
                lt.Text           = lbl
                lt.TextColor3     = T.Text
                lt.TextSize       = 13
                lt.Font           = Enum.Font.Gotham
                lt.TextXAlignment = Enum.TextXAlignment.Left
                lt.Parent         = inner
            end
            do
                local ar = Instance.new("TextLabel")
                ar.BackgroundTransparency = 1
                ar.Size       = UDim2.new(0,22,1,0)
                ar.Position   = UDim2.new(1,-26,0,0)
                ar.Text       = "›"
                ar.TextColor3 = T.AccentText
                ar.TextSize   = 18
                ar.Font       = Enum.Font.GothamBold
                ar.Parent     = inner
            end

            local hit = Instance.new("TextButton")
            hit.BackgroundTransparency = 1
            hit.Size   = UDim2.new(1,0,1,0)
            hit.Text   = ""
            hit.Parent = inner
            hit.MouseEnter:Connect(function()       Tween(inner,{BackgroundColor3=T.CardHover},0.1)  end)
            hit.MouseLeave:Connect(function()       Tween(inner,{BackgroundColor3=T.Card     },0.1)  end)
            hit.MouseButton1Down:Connect(function() Tween(inner,{BackgroundColor3=T.CardPress},0.07) end)
            hit.MouseButton1Up:Connect(function()   Tween(inner,{BackgroundColor3=T.CardHover},0.1); cb() end)

            return outer
        end

        -- ────────────────────────────────────────
        --  TOGGLE
        -- ────────────────────────────────────────
        function tab:AddToggle(cfg)
            cfg = cfg or {}
            local lbl     = cfg.Label    or "Toggle"
            local default = cfg.Default  or false
            local cb      = cfg.Callback or function() end
            local toggled = default

            local outer, inner = MakeRoundedFrame(scroll, T.Card, 6, T.CardBorder, 1)
            outer.Size = UDim2.new(1, 0, 0, 38)

            do
                local lt = Instance.new("TextLabel")
                lt.BackgroundTransparency = 1
                lt.Size           = UDim2.new(1,-60,1,0)
                lt.Position       = UDim2.new(0,14,0,0)
                lt.Text           = lbl
                lt.TextColor3     = T.Text
                lt.TextSize       = 13
                lt.Font           = Enum.Font.Gotham
                lt.TextXAlignment = Enum.TextXAlignment.Left
                lt.Parent         = inner
            end

            local track = Instance.new("Frame")
            track.BackgroundColor3 = toggled and T.ToggleOn or T.ToggleOff
            track.Size             = UDim2.new(0,38,0,20)
            track.Position         = UDim2.new(1,-50,0.5,-10)
            track.BorderSizePixel  = 0
            track.Parent           = inner
            do local c=Instance.new("UICorner"); c.CornerRadius=UDim.new(0,10); c.Parent=track end

            local thumb = Instance.new("Frame")
            thumb.BackgroundColor3 = Color3.fromRGB(255,255,255)
            thumb.Size             = UDim2.new(0,14,0,14)
            thumb.Position         = toggled and UDim2.new(0,21,0.5,-7) or UDim2.new(0,3,0.5,-7)
            thumb.BorderSizePixel  = 0
            thumb.Parent           = track
            do local c=Instance.new("UICorner"); c.CornerRadius=UDim.new(0,7); c.Parent=thumb end

            local function refresh()
                if toggled then
                    Tween(track,{BackgroundColor3=T.ToggleOn},0.18)
                    Tween(thumb,{Position=UDim2.new(0,21,0.5,-7)},0.18)
                else
                    Tween(track,{BackgroundColor3=T.ToggleOff},0.18)
                    Tween(thumb,{Position=UDim2.new(0,3,0.5,-7)},0.18)
                end
                cb(toggled)
            end

            local hit = Instance.new("TextButton")
            hit.BackgroundTransparency = 1
            hit.Size   = UDim2.new(1,0,1,0)
            hit.Text   = ""
            hit.Parent = inner
            hit.MouseButton1Click:Connect(function() toggled=not toggled; refresh() end)
            hit.MouseEnter:Connect(function() Tween(inner,{BackgroundColor3=T.CardHover},0.1) end)
            hit.MouseLeave:Connect(function() Tween(inner,{BackgroundColor3=T.Card     },0.1) end)

            local obj = {}
            function obj:Set(v) toggled=v; refresh() end
            function obj:Get() return toggled end
            return obj
        end

        -- ────────────────────────────────────────
        --  SLIDER
        -- ────────────────────────────────────────
        function tab:AddSlider(cfg)
            cfg = cfg or {}
            local lbl      = cfg.Label    or "Slider"
            local minVal   = cfg.Min      or 0
            local maxVal   = cfg.Max      or 100
            local defVal   = cfg.Default  or minVal
            local decimals = cfg.Decimals or 0
            local cb       = cfg.Callback or function() end
            local cur      = defVal

            local outer, inner = MakeRoundedFrame(scroll, T.Card, 6, T.CardBorder, 1)
            outer.Size = UDim2.new(1, 0, 0, 58)
            AddPadding(inner, 9, 9, 14, 14)

            local topRow = Instance.new("Frame")
            topRow.BackgroundTransparency = 1
            topRow.Size   = UDim2.new(1, 0, 0, 18)
            topRow.Parent = inner
            do
                local lt = Instance.new("TextLabel")
                lt.BackgroundTransparency=1; lt.Size=UDim2.new(0.7,0,1,0)
                lt.Text=lbl; lt.TextColor3=T.Text; lt.TextSize=13
                lt.Font=Enum.Font.Gotham; lt.TextXAlignment=Enum.TextXAlignment.Left
                lt.Parent=topRow
            end
            local valLbl
            do
                valLbl = Instance.new("TextLabel")
                valLbl.BackgroundTransparency=1; valLbl.Size=UDim2.new(0.3,0,1,0)
                valLbl.Position=UDim2.new(0.7,0,0,0); valLbl.Text=tostring(cur)
                valLbl.TextColor3=T.AccentText; valLbl.TextSize=13
                valLbl.Font=Enum.Font.GothamBold; valLbl.TextXAlignment=Enum.TextXAlignment.Right
                valLbl.Parent=topRow
            end

            local trackBg = Instance.new("Frame")
            trackBg.BackgroundColor3 = T.BorderSubtle
            trackBg.Size             = UDim2.new(1,0,0,4)
            trackBg.Position         = UDim2.new(0,0,0,28)
            trackBg.BorderSizePixel  = 0
            trackBg.Parent           = inner
            do local c=Instance.new("UICorner"); c.CornerRadius=UDim.new(0,2); c.Parent=trackBg end

            local fill = Instance.new("Frame")
            fill.BackgroundColor3 = T.Accent
            fill.Size             = UDim2.new((cur-minVal)/(maxVal-minVal),0,1,0)
            fill.BorderSizePixel  = 0
            fill.Parent           = trackBg
            do local c=Instance.new("UICorner"); c.CornerRadius=UDim.new(0,2); c.Parent=fill end

            local handle = Instance.new("Frame")
            handle.BackgroundColor3 = T.SliderThumb
            handle.AnchorPoint      = Vector2.new(0.5,0.5)
            handle.Size             = UDim2.new(0,13,0,13)
            handle.Position         = UDim2.new((cur-minVal)/(maxVal-minVal),0,0.5,0)
            handle.BorderSizePixel  = 0
            handle.Parent           = trackBg
            do local c=Instance.new("UICorner"); c.CornerRadius=UDim.new(0,7); c.Parent=handle end

            local function setVal(v)
                v = math.clamp(v, minVal, maxVal)
                if decimals==0 then v=math.round(v)
                else v=math.floor(v*10^decimals+0.5)/10^decimals end
                cur = v
                local r = (v-minVal)/(maxVal-minVal)
                fill.Size       = UDim2.new(r,0,1,0)
                handle.Position = UDim2.new(r,0,0.5,0)
                valLbl.Text     = tostring(v)
                cb(v)
            end

            local dragging = false
            handle.InputBegan:Connect(function(i)
                if i.UserInputType==Enum.UserInputType.MouseButton1 then
                    dragging=true; Tween(handle,{Size=UDim2.new(0,16,0,16)},0.1)
                end
            end)
            trackBg.InputBegan:Connect(function(i)
                if i.UserInputType==Enum.UserInputType.MouseButton1 then
                    dragging=true; Tween(handle,{Size=UDim2.new(0,16,0,16)},0.1)
                end
            end)
            UserInputService.InputEnded:Connect(function(i)
                if i.UserInputType==Enum.UserInputType.MouseButton1 then
                    dragging=false; Tween(handle,{Size=UDim2.new(0,13,0,13)},0.1)
                end
            end)
            RunService.RenderStepped:Connect(function()
                if dragging then
                    local tp=trackBg.AbsolutePosition.X; local ts=trackBg.AbsoluteSize.X
                    setVal(minVal + math.clamp((Mouse.X-tp)/ts,0,1)*(maxVal-minVal))
                end
            end)

            local obj = {}
            function obj:Set(v) setVal(v) end
            function obj:Get() return cur end
            return obj
        end

        -- ────────────────────────────────────────
        --  INPUT
        -- ────────────────────────────────────────
        function tab:AddInput(cfg)
            cfg = cfg or {}
            local lbl         = cfg.Label       or "Input"
            local placeholder = cfg.Placeholder or "Digite aqui..."
            local cb          = cfg.Callback    or function() end
            local onChange    = cfg.OnChange    or nil

            local outer, inner = MakeRoundedFrame(scroll, T.Card, 6, T.CardBorder, 1)
            outer.Size = UDim2.new(1, 0, 0, 60)

            do
                local lt = Instance.new("TextLabel")
                lt.BackgroundTransparency=1; lt.Size=UDim2.new(1,-16,0,18)
                lt.Position=UDim2.new(0,14,0,8); lt.Text=lbl
                lt.TextColor3=T.TextMuted; lt.TextSize=11
                lt.Font=Enum.Font.GothamBold; lt.TextXAlignment=Enum.TextXAlignment.Left
                lt.Parent=inner
            end

            -- input box: também usa MakeRoundedFrame para borda correta
            local boxOuter, boxInner = MakeRoundedFrame(inner, T.ContentBg, 5, T.Border, 1)
            boxOuter.Size     = UDim2.new(1,-16,0,24)
            boxOuter.Position = UDim2.new(0,8,0,28)

            local tb = Instance.new("TextBox")
            tb.BackgroundTransparency = 1
            tb.Size              = UDim2.new(1,-16,1,0)
            tb.Position          = UDim2.new(0,8,0,0)
            tb.PlaceholderText   = placeholder
            tb.PlaceholderColor3 = T.TextDisabled
            tb.Text              = ""
            tb.TextColor3        = T.Text
            tb.TextSize          = 12
            tb.Font              = Enum.Font.Gotham
            tb.TextXAlignment    = Enum.TextXAlignment.Left
            tb.ClearTextOnFocus  = false
            tb.Parent            = boxInner

            local boxStroke = boxOuter:FindFirstChildOfClass("UIStroke")
            tb.Focused:Connect(function()
                Tween(boxInner,{BackgroundColor3=T.CardHover},0.15)
                if boxStroke then Tween(boxStroke,{Color=T.Accent},0.15) end
            end)
            tb.FocusLost:Connect(function(enter)
                Tween(boxInner,{BackgroundColor3=T.ContentBg},0.15)
                if boxStroke then Tween(boxStroke,{Color=T.Border},0.15) end
                if enter then cb(tb.Text) end
            end)
            if onChange then
                tb:GetPropertyChangedSignal("Text"):Connect(function() onChange(tb.Text) end)
            end

            local obj = {}
            function obj:Get() return tb.Text end
            function obj:Set(v) tb.Text = v end
            return obj
        end

        return tab
    end -- AddTab

    return Win
end -- CreateWindow

-- ═══════════════════════════════════════════════════
return NexusUI
-- ═══════════════════════════════════════════════════


--[[
═══════════════════════════════════════════════════════
  EXEMPLO COMPLETO — NexusUI v2.0
═══════════════════════════════════════════════════════

local NexusUI = loadstring(game:HttpGet("SUA_URL"))()

local Win = NexusUI:CreateWindow({
    Title    = "Meu Script",
    SubTitle = "by você • v2.0",
    Theme    = "Dark",
    Size     = UDim2.new(0, 580, 0, 400),
})

-- Tabs: aceita rbxassetid://, só número, ou nil
local TabGeral  = Win:AddTab("Geral",  "rbxassetid://3926305904")
local TabPlayer = Win:AddTab("Player", "4483345998")
local TabConfig = Win:AddTab("Config")

TabGeral:AddSection("Movimentação")

TabGeral:AddButton({
    Label = "Teletransportar ao Spawn",
    Callback = function()
        game.Players.LocalPlayer.Character:MoveTo(Vector3.new(0,5,0))
    end
})

TabGeral:AddToggle({
    Label = "Sprint Infinito", Default = false,
    Callback = function(v) print("Sprint:", v) end
})

TabGeral:AddSlider({
    Label = "Velocidade", Min = 16, Max = 200, Default = 16,
    Callback = function(v)
        game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = v
    end
})

TabPlayer:AddSection("Informações")
TabPlayer:AddInput({
    Label = "Display Name", Placeholder = "Seu nome...",
    Callback = function(v) print("Nome:", v) end
})

NexusUI:Notify({
    Title = "NexusUI Carregado!",
    Message = "Bem-vindo. Script ativo.",
    Type = "Success",
    Duration = 5,
})

═══════════════════════════════════════════════════════
--]]

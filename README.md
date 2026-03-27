local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local LP = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- === CONFIGURACIÓN MAESTRA V67 ===
local Settings = {
    Aimbot = false,
    Aim360Total = false,
    VisibleCheck = false,
    Priority = "Mira", -- "Mira" o "Distancia"
    AimPart = "Head", 
    TeamCheck = true,
    ShowAliados = false,
    ShowFOV = true,
    FOV = 150,
    Smoothing = 0,
    MaxDistance = 2000,
    ESP_Names = false,
    ESP_Lines = false,
    ESP_Chams = false,
    VentajaEnabled = false,
    OffX = 0, OffY = 0, OffZ = -3.5
}

local CurrentTarget = nil
local lns, nms, chams, fovLines = {}, {}, {}, {}
local Running = true

-- === LIMPIEZA ===
local function removeVisuals(p)
    if lns[p] then pcall(function() lns[p]:Remove() end) lns[p] = nil end
    if nms[p] then pcall(function() nms[p]:Remove() end) nms[p] = nil end
    if chams[p] then pcall(function() chams[p]:Destroy() end) chams[p] = nil end
    pcall(function() if p.Character and p.Character:FindFirstChild("Head") then p.Character.Head.Anchored = false end end)
end

local function cleanAll()
    Running = false
    for _, l in pairs(fovLines) do pcall(function() l:Remove() end) end
    for _, p in pairs(Players:GetPlayers()) do removeVisuals(p) end
end

Players.PlayerRemoving:Connect(removeVisuals)

-- === DIBUJO FOV ===
for i = 1, 36 do
    local l = Drawing.new("Line")
    l.Thickness, l.Transparency, l.Visible = 2, 1, false
    fovLines[i] = l
end

local function updateFOVCircle()
    local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    for i = 1, 36 do
        local l = fovLines[i]
        if Settings.ShowFOV and Settings.Aimbot and not Settings.Aim360Total and Running then
            local a1, a2 = math.rad((i-1)*10), math.rad(i*10)
            l.From = center + Vector2.new(math.cos(a1)*Settings.FOV, math.sin(a1)*Settings.FOV)
            l.To = center + Vector2.new(math.cos(a2)*Settings.FOV, math.sin(a2)*Settings.FOV)
            l.Color = CurrentTarget and Color3.new(1,1,0) or Color3.new(1,1,1)
            l.Visible = true
        else l.Visible = false end
    end
end

-- === INTERFAZ MOBILE V67 ===
local gui = Instance.new("ScreenGui", LP.PlayerGui)
gui.Name = "V67_Master"; gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size, frame.Position = UDim2.new(0, 380, 0, 350), UDim2.new(0.5, -190, 0.5, -175)
frame.BackgroundColor3 = Color3.fromRGB(10, 10, 12)
frame.Active, frame.Draggable = true, true
Instance.new("UICorner", frame)
Instance.new("UIStroke", frame).Color = Color3.fromRGB(0, 255, 150)

local openBtn = Instance.new("TextButton", gui)
openBtn.Size, openBtn.Position = UDim2.new(0, 50, 0, 50), UDim2.new(0, 15, 0.5, 0)
openBtn.BackgroundColor3, openBtn.Text = Color3.fromRGB(0, 255, 150), "C"
openBtn.TextColor3, openBtn.Font, openBtn.TextSize = Color3.new(0,0,0), "GothamBold", 25
openBtn.Visible = false 
openBtn.Active, openBtn.Draggable = true, true
Instance.new("UICorner", openBtn).CornerRadius = UDim.new(1, 0)

local function toggleUI(state)
    frame.Visible = state
    openBtn.Visible = not state
end

openBtn.MouseButton1Click:Connect(function() toggleUI(true) end)

local closeX = Instance.new("TextButton", frame)
closeX.Size, closeX.Position = UDim2.new(0, 30, 0, 30), UDim2.new(1, -35, 0, 5)
closeX.BackgroundColor3, closeX.Text, closeX.TextColor3 = Color3.fromRGB(200, 50, 50), "X", Color3.new(1,1,1)
Instance.new("UICorner", closeX)
closeX.MouseButton1Click:Connect(function() toggleUI(false) end)

local sidebar = Instance.new("Frame", frame)
sidebar.Size, sidebar.Position = UDim2.new(0, 95, 1, -20), UDim2.new(0, 10, 0, 10)
sidebar.BackgroundColor3 = Color3.fromRGB(15, 15, 18)
Instance.new("UICorner", sidebar)
Instance.new("UIListLayout", sidebar).Padding, Instance.new("UIListLayout", sidebar).HorizontalAlignment = UDim.new(0, 6), "Center"

local container = Instance.new("Frame", frame)
container.Size, container.Position = UDim2.new(1, -125, 1, -50), UDim2.new(0, 115, 0, 40)
container.BackgroundTransparency = 1

local pgESP, pgAIM, pgADV = Instance.new("ScrollingFrame", container), Instance.new("ScrollingFrame", container), Instance.new("ScrollingFrame", container)

local function setupPg(p, v)
    p.Size, p.Visible = UDim2.new(1, 0, 1, 0), v
    p.BackgroundTransparency, p.CanvasSize, p.ScrollBarThickness = 1, UDim2.new(0,0,2.2,0), 0
    Instance.new("UIListLayout", p).Padding = UDim.new(0, 5)
end
setupPg(pgESP, true); setupPg(pgAIM, false); setupPg(pgADV, false)

local function createCat(t, target)
    local b = Instance.new("TextButton", sidebar)
    b.Size, b.Text = UDim2.new(0.9, 0, 0, 40), t
    b.BackgroundColor3, b.TextColor3, b.Font = Color3.fromRGB(25, 25, 30), Color3.new(1,1,1), "GothamBold"
    Instance.new("UICorner", b)
    b.MouseButton1Click:Connect(function()
        pgESP.Visible, pgAIM.Visible, pgADV.Visible = (target == pgESP), (target == pgAIM), (target == pgADV)
        for _, obj in pairs(sidebar:GetChildren()) do if obj:IsA("TextButton") and obj.Text ~= "KILL" then obj.BackgroundColor3 = Color3.fromRGB(25,25,30) obj.TextColor3 = Color3.new(1,1,1) end end
        b.BackgroundColor3, b.TextColor3 = Color3.fromRGB(0, 255, 150), Color3.new(0,0,0)
    end)
    return b
end

createCat("ESP", pgESP).BackgroundColor3 = Color3.fromRGB(0, 255, 150)
createCat("AIM", pgAIM); createCat("VENTAJA", pgADV)

-- HELPERS
local function createToggle(t, v, p)
    local b = Instance.new("TextButton", p) b.Size = UDim2.new(1, -5, 0, 38)
    local function up() b.Text = t..": "..(Settings[v] and "ON" or "OFF") b.BackgroundColor3 = Settings[v] and Color3.fromRGB(0, 150, 100) or Color3.fromRGB(35, 35, 40) end
    b.TextColor3, b.Font = Color3.new(1,1,1), "GothamBold"
    b.MouseButton1Click:Connect(function() Settings[v] = not Settings[v] up() end) up() Instance.new("UICorner", b)
end

local function createMode(t, v, o1, o2, p)
    local b = Instance.new("TextButton", p) b.Size = UDim2.new(1, -5, 0, 38)
    b.BackgroundColor3, b.TextColor3, b.Font = Color3.fromRGB(40, 40, 45), Color3.new(1,1,1), "GothamBold"
    local function up() b.Text = t..": "..Settings[v] end
    b.MouseButton1Click:Connect(function() Settings[v] = (Settings[v] == o1 and o2 or o1) up() end) up() Instance.new("UICorner", b)
end

local function createSlider(t, min, max, v, p)
    local c = Instance.new("Frame", p) c.Size = UDim2.new(1, -5, 0, 50) c.BackgroundTransparency = 1
    local l = Instance.new("TextLabel", c) l.Size, l.TextColor3, l.Text = UDim2.new(1,0,0,20), Color3.new(1,1,1), t..": "..string.format("%.1f", Settings[v]) l.BackgroundTransparency = 1
    local b = Instance.new("Frame", c) b.Size, b.Position, b.BackgroundColor3 = UDim2.new(0.9,0,0,4), UDim2.new(0.05,0,0,35), Color3.fromRGB(50,50,55)
    local k = Instance.new("Frame", b) k.Size, k.BackgroundColor3 = UDim2.new(0,16,0,16), Color3.fromRGB(0, 255, 150) k.Position = UDim2.new(0,0,0.5,-8) Instance.new("UICorner", k)
    b.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseButton1 then
        local move = RunService.RenderStepped:Connect(function()
            local pos = math.clamp((UIS:GetMouseLocation().X - b.AbsolutePosition.X) / b.AbsoluteSize.X, 0, 1)
            k.Position = UDim2.new(pos, -8, 0.5, -8)
            Settings[v] = min + (pos * (max - min)) l.Text = t..": "..string.format("%.1f", Settings[v])
        end)
        UIS.InputEnded:Connect(function(e) if e.UserInputType == Enum.UserInputType.Touch or e.UserInputType == Enum.UserInputType.MouseButton1 then move:Disconnect() end end)
    end end)
end

-- CONTROLES V67
createToggle("ESP NOMBRES", "ESP_Names", pgESP)
createToggle("ESP LINEAS", "ESP_Lines", pgESP)
createToggle("ESP CHAMS", "ESP_Chams", pgESP)
createToggle("MOSTRAR EQUIPO", "ShowAliados", pgESP)
createSlider("DISTANCIA ESP", 100, 5000, "MaxDistance", pgESP)

createToggle("ACTIVAR AIMBOT", "Aimbot", pgAIM)
createToggle("TEAM CHECK", "TeamCheck", pgAIM)
createMode("PRIORIDAD", "Priority", "Mira", "Distancia", pgAIM)
createMode("OBJETIVO", "AimPart", "Head", "HumanoidRootPart", pgAIM)
createToggle("MODO 360", "Aim360Total", pgAIM)
createToggle("VISIBLE CHECK", "VisibleCheck", pgAIM)
createSlider("SUAVIDAD", 0, 10, "Smoothing", pgAIM)
createSlider("RADIO FOV", 10, 800, "FOV", pgAIM)

createToggle("ACTIVAR VENTAJA", "VentajaEnabled", pgADV)
createSlider("OFF X", -10, 10, "OffX", pgADV)
createSlider("OFF Y", -10, 10, "OffY", pgADV)
createSlider("OFF Z", -10, 10, "OffZ", pgADV)

local dBtn = Instance.new("TextButton", sidebar)
dBtn.Size, dBtn.Text, dBtn.BackgroundColor3 = UDim2.new(0.9, 0, 0, 35), "KILL", Color3.fromRGB(150,0,0)
dBtn.TextColor3, dBtn.Font = Color3.new(1,1,1), "GothamBold"
Instance.new("UICorner", dBtn)
dBtn.MouseButton1Click:Connect(function() cleanAll() gui:Destroy() end)

-- === LÓGICA VISIBLE ===
local function isVisible(part, char)
    if not Settings.VisibleCheck then return true end
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {LP.Character, char}
    local res = workspace:Raycast(Camera.CFrame.Position, (part.Position - Camera.CFrame.Position), params)
    return res == nil
end

-- === LOOP PRINCIPAL ===
RunService.RenderStepped:Connect(function()
    if not Running then return end
    updateFOVCircle()
    local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    local potTarget, lastVal = nil, math.huge

    for _, p in pairs(Players:GetPlayers()) do
        local char = p.Character
        if p ~= LP and char and char:FindFirstChild(Settings.AimPart) and char.Humanoid.Health > 0 then
            local part = char[Settings.AimPart]
            local pos, onscreen = Camera:WorldToViewportPoint(part.Position)
            local mag = (part.Position - LP.Character.HumanoidRootPart.Position).Magnitude
            
            -- LÓGICA TEAM CHECK REPARADA
            local isAlly = (p.Team == LP.Team) or (p.TeamColor == LP.TeamColor)
            local isEnemy = not Settings.TeamCheck or not isAlly
            
            if (isEnemy or Settings.ShowAliados) and mag <= Settings.MaxDistance then
                -- Color: Si TeamCheck está OFF, todos son Rojos. Si está ON, aliados son Verdes.
                local baseColor = (not Settings.TeamCheck) and Color3.new(1,0,0) or (isAlly and Color3.new(0,1,0) or Color3.new(1,0,0))
                -- Si es el target actual, Amarillo
                local color = (p == CurrentTarget) and Color3.new(1,1,0) or baseColor

                if Settings.ESP_Names and onscreen then
                    if not nms[p] then nms[p] = Drawing.new("Text") nms[p].Center, nms[p].Outline = true, true end
                    nms[p].Visible, nms[p].Color, nms[p].Text, nms[p].Position = true, color, p.Name, Vector2.new(pos.X, pos.Y - 25)
                elseif nms[p] then nms[p].Visible = false end

                if Settings.ESP_Lines and onscreen then
                    if not lns[p] then lns[p] = Drawing.new("Line") end
                    lns[p].Visible, lns[p].Color, lns[p].From, lns[p].To = true, color, Vector2.new(center.X, Camera.ViewportSize.Y), Vector2.new(pos.X, pos.Y)
                elseif lns[p] then lns[p].Visible = false end

                if Settings.ESP_Chams then
                    if not chams[p] then chams[p] = Instance.new("Highlight", CoreGui) end
                    chams[p].Adornee, chams[p].Enabled, chams[p].FillColor = char, true, color
                elseif chams[p] then chams[p].Enabled = false end

                -- SELECCIÓN POR PRIORIDAD
                if isEnemy then
                    local sDist = (Vector2.new(pos.X, pos.Y) - center).Magnitude
                    if (Settings.Aim360Total or (onscreen and sDist <= Settings.FOV)) and isVisible(part, char) then
                        local val = (Settings.Priority == "Mira") and sDist or mag
                        if val < lastVal then lastVal, potTarget = val, p end
                    end
                end
            else removeVisuals(p) end
        else removeVisuals(p) end
    end

    if CurrentTarget ~= potTarget and CurrentTarget then
        pcall(function() CurrentTarget.Character.Head.Anchored = false end)
    end
    CurrentTarget = potTarget

    if CurrentTarget and Settings.Aimbot then
        local lerpVal = Settings.Smoothing == 0 and 1 or (1 - (Settings.Smoothing/10))
        Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, CurrentTarget.Character[Settings.AimPart].Position), lerpVal)
        
        if Settings.VentajaEnabled then
            pcall(function()
                local h = CurrentTarget.Character.Head
                h.Anchored, h.CanCollide = true, false
                h.CFrame = LP.Character.HumanoidRootPart.CFrame * CFrame.new(Settings.OffX, Settings.OffY, Settings.OffZ)
            end)
        end
    end
end)

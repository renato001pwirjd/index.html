-- Auto Server Hop Mirage | Blox Fruits
-- Coloque em StarterPlayerScripts como LocalScript

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Lighting = game:GetService("Lighting")
local CoreGui = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer
local PLACE_ID = game.PlaceId

local FOG_THRESHOLD = 500 -- FogEnd abaixo disso = névoa = chance alta de Mirage
local HOP_DELAY = 5 -- segundos de espera após entrar no servidor
local MAX_HOPS = 50

local running = false
local hops = 0

-- ===== GUI =====
local gui = Instance.new("ScreenGui")
gui.Name = "MirageHopGUI"
gui.ResetOnSpawn = false
gui.Parent = CoreGui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 280, 0, 120)
frame.Position = UDim2.new(0.5, -140, 0, 20)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
frame.BackgroundTransparency = 0.1
frame.BorderSizePixel = 0
frame.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = frame

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(0, 200, 255)
stroke.Thickness = 2
stroke.Parent = frame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 28)
title.Position = UDim2.new(0, 0, 0, 6)
title.BackgroundTransparency = 1
title.Text = "MIRAGE HOP"
title.TextColor3 = Color3.fromRGB(0, 220, 255)
title.TextScaled = true
title.Font = Enum.Font.GothamBlack
title.Parent = frame

local status = Instance.new("TextLabel")
status.Name = "Status"
status.Size = UDim2.new(1, -20, 0, 22)
status.Position = UDim2.new(0, 10, 0, 36)
status.BackgroundTransparency = 1
status.Text = "Status: parado"
status.TextColor3 = Color3.fromRGB(200, 200, 200)
status.TextXAlignment = Enum.TextXAlignment.Left
status.Font = Enum.Font.Gotham
status.TextSize = 14
status.Parent = frame

local hopLbl = Instance.new("TextLabel")
hopLbl.Name = "Hops"
hopLbl.Size = UDim2.new(1, -20, 0, 18)
hopLbl.Position = UDim2.new(0, 10, 0, 58)
hopLbl.BackgroundTransparency = 1
hopLbl.Text = "Hops: 0"
hopLbl.TextColor3 = Color3.fromRGB(255, 200, 80)
hopLbl.TextXAlignment = Enum.TextXAlignment.Left
hopLbl.Font = Enum.Font.Gotham
hopLbl.TextSize = 13
hopLbl.Parent = frame

local btn = Instance.new("TextButton")
btn.Size = UDim2.new(1, -20, 0, 30)
btn.Position = UDim2.new(0, 10, 1, -36)
btn.BackgroundColor3 = Color3.fromRGB(0, 150, 200)
btn.Text = "INICIAR"
btn.TextColor3 = Color3.new(1, 1, 1)
btn.Font = Enum.Font.GothamBold
btn.TextSize = 15
btn.BorderSizePixel = 0
btn.Parent = frame

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 8)
btnCorner.Parent = btn

-- ===== DETECÇÃO =====
local function detectMirage()
    -- método 1: névoa baixa
    if Lighting.FogEnd < FOG_THRESHOLD then
        return true, "Névoa detectada (FogEnd: " .. math.floor(Lighting.FogEnd) .. ")"
    end

    -- método 2: procurar objeto com nome "Mirage" no Workspace
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj.Name:lower():find("mirage") then
            return true, "Objeto Mirage encontrado: " .. obj.Name
        end
    end

    return false, "Sem Mirage neste servidor"
end

-- ===== SERVER HOP =====
local function getServers()
    local servers = {}
    local cursor = ""

    repeat
        local url = "https://games.roblox.com/v1/games/" .. PLACE_ID .. "/servers/Public?sortOrder=Asc&limit=100"
        if cursor ~= "" then
            url = url .. "&cursor=" .. cursor
        end

        local ok, response = pcall(function()
            return HttpService:JSONDecode(game:HttpGet(url))
        end)

        if not ok or not response then
            break
        end

        for _, server in ipairs(response.data) do
            if server.playing < server.maxPlayers and server.id ~= game.JobId then
                table.insert(servers, server)
            end
        end

        cursor = response.nextPageCursor or ""
    until cursor == "" or #servers >= 50

    return servers
end

local function hopToServer(serverId)
    TeleportService:TeleportToPlaceInstance(PLACE_ID, serverId, LocalPlayer)
end

-- ===== LOOP PRINCIPAL =====
local function startHop()
    if running then return end
    running = true
    hops = 0
    btn.Text = "PARAR"
    btn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)

    task.spawn(function()
        while running do
            status.Text = "Status: verificando servidor..."
            hopLbl.Text = "Hops: " .. hops

            task.wait(HOP_DELAY)

            local found, reason = detectMirage()

            if found then
                status.Text = "Status: MIRAGE ENCONTRADA!"
                status.TextColor3 = Color3.fromRGB(100, 255, 100)
                running = false
                btn.Text = "INICIAR"
                btn.BackgroundColor3 = Color3.fromRGB(0, 150, 200)

                local notify = Instance.new("TextLabel")
                notify.Size = UDim2.new(0, 400, 0, 60)
                notify.Position = UDim2.new(0.5, -200, 0.3, 0)
                notify.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
                notify.BackgroundTransparency = 0.3
                notify.Text = "MIRAGE ISLAND ENCONTRADA!\n" .. reason
                notify.TextColor3 = Color3.fromRGB(100, 255, 100)
                notify.TextScaled = true
                notify.Font = Enum.Font.GothamBold
                notify.Parent = gui

                task.wait(5)
                notify:Destroy()
                return
            end

            status.Text = "Status: " .. reason .. " — trocando..."
            task.wait(2)

            local servers = getServers()
            if #servers == 0 then
                status.Text = "Status: sem servidores disponíveis"
                task.wait(5)
            else
                hops = hops + 1
                hopLbl.Text = "Hops: " .. hops

                if hops >= MAX_HOPS then
                    status.Text = "Status: limite de hops atingido"
                    running = false
                    btn.Text = "INICIAR"
                    btn.BackgroundColor3 = Color3.fromRGB(0, 150, 200)
                    return
                end

                local target = servers[math.random(1, #servers)]
                hopToServer(target.id)
                task.wait(10)
            end
        end
    end)
end

local function stopHop()
    running = false
    btn.Text = "INICIAR"
    btn.BackgroundColor3 = Color3.fromRGB(0, 150, 200)
    status.Text = "Status: parado"
    status.TextColor3 = Color3.fromRGB(200, 200, 200)
end

btn.MouseButton1Click:Connect(function()
    if running then
        stopHop()
    else
        startHop()
    end
end)

print("[Mirage Hop] Carregado.")

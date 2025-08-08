--[[
    PURGE HUB - Construir uma colméia [Com Abas e Danças]
    Build A Beehive [Special Bees] - HUB & Script Helper com Minimizar, Arrastar, Abas Home/Loja/Player, Danças
    Autor: @juanlrm
    Minimizar/Abrir: Tecla G
    Arrastar: Clique e segure no topo do HUB ou no círculo para mover
    Abas: Home (casinha), Loja (carrinho), Player (boneco)
]]

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UIS = game:GetService("UserInputService")

-- Locais para teleportar (exemplo, ajuste conforme necessário)
local locations = {
    Hive = Vector3.new(100, 10, 200),
    Shop = Vector3.new(150, 10, 250),
    Field = Vector3.new(200, 10, 300),
}

local autoCollectOn = false
local autoSellOn = false
local autoBuyPlantsAll = false
local autoBuyPlantsSpecific = {}
local speedBoostOn = false
local flyOn = false

local availablePlants = {
    "Lavanda", "Margarida", "Trigo de quatro folhas", "Flor de girassol", "Dahlia", "Bambu", "Tulipa", "Isca de Mosca de Vênus", "Flor de Fogo", "Bluebell", "Lírio", "Rosa", "Cactus"
}

-- Danças Roblox comuns (AnimationId)
local dances = {
    {name="Dance 1", id="rbxassetid://182435998"}, -- default dance
    {name="Dance 2", id="rbxassetid://248263260"}, -- silly dance
    {name="Gangnam Style", id="rbxassetid://1372127690"},
    {name="Floss", id="rbxassetid://4006024309"},
    {name="Ninja", id="rbxassetid://656118852"},
    {name="Robot", id="rbxassetid://33796059"}
}

local playingDance = nil -- guarda a AnimationTrack para parar

function teleportTo(pos)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        player.Character.HumanoidRootPart.CFrame = CFrame.new(pos)
    end
end

function autoCollect()
    while autoCollectOn do
        for _, obj in pairs(workspace:GetChildren()) do
            if obj.Name == "Honey" or obj.Name == "Pollen" then
                firetouchinterest(player.Character.HumanoidRootPart, obj, 0)
                firetouchinterest(player.Character.HumanoidRootPart, obj, 1)
            end
        end
        wait(1)
    end
end

function autoSell()
    while autoSellOn do
        local sellRemote = ReplicatedStorage:FindFirstChild("SellHoney")
        if sellRemote then
            sellRemote:FireServer()
        end
        wait(10)
    end
end

function setSpeed(value)
    if player.Character and player.Character:FindFirstChildOfClass("Humanoid") then
        player.Character.Humanoid.WalkSpeed = value
    end
end

function fly(enable)
    if player.Character and player.Character:FindFirstChildOfClass("Humanoid") then
        player.Character.Humanoid.PlatformStand = enable
        if enable then
            local bp = Instance.new("BodyPosition", player.Character.HumanoidRootPart)
            bp.MaxForce = Vector3.new(1e5,1e5,1e5)
            bp.Position = player.Character.HumanoidRootPart.Position
            bp.Name = "FlyBP"
        else
            local bp = player.Character.HumanoidRootPart:FindFirstChild("FlyBP")
            if bp then bp:Destroy() end
        end
    end
end

function autoBuyAllPlants()
    while autoBuyPlantsAll do
        local buyAllRemote = ReplicatedStorage:FindFirstChild("BuyAllPlants")
        if buyAllRemote then
            buyAllRemote:FireServer()
        end
        wait(10)
    end
end

function autoBuySpecificPlants()
    while true do
        for _, plant in pairs(availablePlants) do
            if autoBuyPlantsSpecific[plant] then
                local buyPlantRemote = ReplicatedStorage:FindFirstChild("BuyPlant")
                if buyPlantRemote then
                    buyPlantRemote:FireServer(plant)
                end
            end
        end
        wait(10)
    end
end

-- DANÇA: Play/Stop
function playDance(animationId)
    local character = player.Character
    if not character then return end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end

    -- Se já está dançando, para
    if playingDance then
        playingDance:Stop()
        playingDance = nil
        return
    end

    local anim = Instance.new("Animation")
    anim.AnimationId = animationId
    local track = humanoid:LoadAnimation(anim)
    track.Looped = true
    track:Play()
    playingDance = track
end

-- Anti-AFK
local virtualUser = game:GetService("VirtualUser")
player.Idled:connect(function()
    virtualUser:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
    wait(1)
    virtualUser:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
end)

-- HUB (ScreenGui)
local gui = Instance.new("ScreenGui")
gui.Name = "PurgeHubBeehiveTabsDance"
gui.Parent = game:GetService("CoreGui")

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 340, 0, 480)
frame.Position = UDim2.new(0, 25, 0, 150)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
frame.BorderSizePixel = 2
frame.Active = true
frame.Visible = true

-- Título
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundTransparency = 1
title.Text = "PURGE HUB - Construir uma colméia"
title.TextColor3 = Color3.fromRGB(180, 0, 0)
title.Font = Enum.Font.SpecialElite
title.TextSize = 22
title.TextStrokeTransparency = 0.4
title.TextStrokeColor3 = Color3.fromRGB(50,0,0)

-- Arrastar HUB
local draggingFrame = false
local dragInputFrame, dragStartFrame, startPosFrame
title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingFrame = true
        dragStartFrame = input.Position
        startPosFrame = frame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                draggingFrame = false
            end
        end)
    end
end)
title.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInputFrame = input
    end
end)
UIS.InputChanged:Connect(function(input)
    if draggingFrame and input == dragInputFrame then
        local delta = input.Position - dragStartFrame
        frame.Position = UDim2.new(startPosFrame.X.Scale, startPosFrame.X.Offset + delta.X, startPosFrame.Y.Scale, startPosFrame.Y.Offset + delta.Y)
    end
end)

-- Abas
local tabBar = Instance.new("Frame", frame)
tabBar.Size = UDim2.new(1, 0, 0, 46)
tabBar.Position = UDim2.new(0,0,0,40)
tabBar.BackgroundTransparency = 1

local tabs = {"Home", "Loja", "Player"}
local tabIcons = {
    Home = "rbxassetid://77339698", -- Casinha
    Loja = "rbxassetid://6026568198", -- Carrinho
    Player = "rbxassetid://77337765" -- Boneco
}
local tabFrames = {}

local selectedTab = "Home"

for i,tabName in ipairs(tabs) do
    local tabBtn = Instance.new("ImageButton", tabBar)
    tabBtn.Size = UDim2.new(0, 40, 0, 40)
    tabBtn.Position = UDim2.new(0, 16+(i-1)*60, 0, 3)
    tabBtn.Image = tabIcons[tabName] or ""
    tabBtn.BackgroundTransparency = 1
    tabBtn.Name = tabName.."Tab"
    tabBtn.AutoButtonColor = false

    local highlight = Instance.new("Frame", tabBtn)
    highlight.Size = UDim2.new(1,0,1,0)
    highlight.BackgroundColor3 = i == 1 and Color3.fromRGB(180,0,0) or Color3.fromRGB(50,50,50)
    highlight.BackgroundTransparency = i == 1 and 0.5 or 0.8
    highlight.Name = "Highlight"

    tabBtn.MouseButton1Click:Connect(function()
        selectedTab = tabName
        for _,btn in pairs(tabBar:GetChildren()) do
            if btn:IsA("ImageButton") then
                btn.Highlight.BackgroundColor3 = btn.Name == tabName.."Tab" and Color3.fromRGB(180,0,0) or Color3.fromRGB(50,50,50)
                btn.Highlight.BackgroundTransparency = btn.Name == tabName.."Tab" and 0.5 or 0.8
            end
        end
        for t,f in pairs(tabFrames) do
            f.Visible = (t == tabName)
        end
    end)
end

local function makeTabFrame(tab)
    local tabFrame = Instance.new("Frame", frame)
    tabFrame.Size = UDim2.new(1, -20, 1, -96)
    tabFrame.Position = UDim2.new(0, 10, 0, 86)
    tabFrame.BackgroundTransparency = 1
    tabFrame.Visible = (tab == "Home")
    tabFrames[tab] = tabFrame
    return tabFrame
end

-- Aba Home
local homeFrame = makeTabFrame("Home")
local btnAutoCollect = Instance.new("TextButton", homeFrame)
btnAutoCollect.Size = UDim2.new(1, 0, 0, 40)
btnAutoCollect.Position = UDim2.new(0, 0, 0, 0)
btnAutoCollect.Text = "Auto Coletar Mel: OFF"
btnAutoCollect.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
btnAutoCollect.TextColor3 = Color3.fromRGB(75, 60, 20)
btnAutoCollect.Font = Enum.Font.Cartoon
btnAutoCollect.TextSize = 20
btnAutoCollect.MouseButton1Click:Connect(function()
    autoCollectOn = not autoCollectOn
    btnAutoCollect.Text = "Auto Coletar Mel: " .. (autoCollectOn and "ON" or "OFF")
    if autoCollectOn then
        spawn(autoCollect)
    end
end)
local btnAutoSell = Instance.new("TextButton", homeFrame)
btnAutoSell.Size = UDim2.new(1, 0, 0, 40)
btnAutoSell.Position = UDim2.new(0, 0, 0, 50)
btnAutoSell.Text = "Auto Vender Mel: OFF"
btnAutoSell.BackgroundColor3 = Color3.fromRGB(255, 182, 193)
btnAutoSell.TextColor3 = Color3.fromRGB(75, 60, 20)
btnAutoSell.Font = Enum.Font.Cartoon
btnAutoSell.TextSize = 20
btnAutoSell.MouseButton1Click:Connect(function()
    autoSellOn = not autoSellOn
    btnAutoSell.Text = "Auto Vender Mel: " .. (autoSellOn and "ON" or "OFF")
    if autoSellOn then
        spawn(autoSell)
    end
end)
local antiAfkLabel = Instance.new("TextLabel", homeFrame)
antiAfkLabel.Size = UDim2.new(1, 0, 0, 30)
antiAfkLabel.Position = UDim2.new(0, 0, 0, 100)
antiAfkLabel.Text = "Anti-AFK: Sempre ativo!"
antiAfkLabel.TextColor3 = Color3.fromRGB(75, 60, 20)
antiAfkLabel.Font = Enum.Font.Cartoon
antiAfkLabel.TextSize = 17
antiAfkLabel.BackgroundTransparency = 1
local infoLabel = Instance.new("TextLabel", homeFrame)
infoLabel.Size = UDim2.new(1, 0, 0, 30)
infoLabel.Position = UDim2.new(0, 0, 0, 135)
infoLabel.Text = "Mel: 0 | Abelhas: 0 | Plantas: 0"
infoLabel.TextColor3 = Color3.fromRGB(180, 0, 0)
infoLabel.Font = Enum.Font.Cartoon
infoLabel.TextSize = 17
infoLabel.BackgroundTransparency = 1
spawn(function()
    while true do
        -- Atualize com valores reais do jogo se quiser
        wait(2)
    end
end)

-- Aba Loja
local lojaFrame = makeTabFrame("Loja")
local btnBuyAll = Instance.new("TextButton", lojaFrame)
btnBuyAll.Size = UDim2.new(1, 0, 0, 40)
btnBuyAll.Position = UDim2.new(0, 0, 0, 0)
btnBuyAll.Text = "Comprar TODAS as plantas: OFF"
btnBuyAll.BackgroundColor3 = Color3.fromRGB(173, 216, 230)
btnBuyAll.TextColor3 = Color3.fromRGB(75, 60, 20)
btnBuyAll.Font = Enum.Font.Cartoon
btnBuyAll.TextSize = 18
btnBuyAll.MouseButton1Click:Connect(function()
    autoBuyPlantsAll = not autoBuyPlantsAll
    btnBuyAll.Text = "Comprar TODAS as plantas: " .. (autoBuyPlantsAll and "ON" or "OFF")
    if autoBuyPlantsAll then
        spawn(autoBuyAllPlants)
    end
end)
local plantasLabel = Instance.new("TextLabel", lojaFrame)
plantasLabel.Size = UDim2.new(1, 0, 0, 28)
plantasLabel.Position = UDim2.new(0, 0, 0, 45)
plantasLabel.Text = "Comprar plantas específicas automaticamente:"
plantasLabel.TextColor3 = Color3.fromRGB(180, 0, 0)
plantasLabel.Font = Enum.Font.Cartoon
plantasLabel.TextSize = 17
plantasLabel.BackgroundTransparency = 1

local yStart = 75
for i,plantName in ipairs(availablePlants) do
    local cb = Instance.new("TextButton", lojaFrame)
    cb.Size = UDim2.new(0, 140, 0, 20)
    cb.Position = UDim2.new(0, ((i-1)%2)*150, 0, yStart+math.floor((i-1)/2)*25)
    cb.Text = "[ ] "..plantName
    cb.BackgroundColor3 = Color3.fromRGB(50,50,50)
    cb.TextColor3 = Color3.fromRGB(180, 0, 0)
    cb.Font = Enum.Font.Cartoon
    cb.TextSize = 16
    autoBuyPlantsSpecific[plantName] = false

    cb.MouseButton1Click:Connect(function()
        autoBuyPlantsSpecific[plantName] = not autoBuyPlantsSpecific[plantName]
        cb.Text = (autoBuyPlantsSpecific[plantName] and "[✔] " or "[ ] ")..plantName
    end)
end

local btnStartSpecific = Instance.new("TextButton", lojaFrame)
btnStartSpecific.Size = UDim2.new(1, 0, 0, 28)
btnStartSpecific.Position = UDim2.new(0, 0, 0, yStart+math.ceil(#availablePlants/2)*25+8)
btnStartSpecific.Text = "Iniciar Compra Automática das Selecionadas"
btnStartSpecific.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
btnStartSpecific.TextColor3 = Color3.fromRGB(75, 60, 20)
btnStartSpecific.Font = Enum.Font.Cartoon
btnStartSpecific.TextSize = 17
btnStartSpecific.MouseButton1Click:Connect(function()
    spawn(autoBuySpecificPlants)
end)

-- Aba Player
local playerFrame = makeTabFrame("Player")
local yTele = 0
for area, pos in pairs(locations) do
    local btnTp = Instance.new("TextButton", playerFrame)
    btnTp.Size = UDim2.new(1, 0, 0, 30)
    btnTp.Position = UDim2.new(0, 0, 0, yTele)
    btnTp.Text = "Teleportar para: " .. area
    btnTp.BackgroundColor3 = Color3.fromRGB(240, 230, 140)
    btnTp.TextColor3 = Color3.fromRGB(75, 60, 20)
    btnTp.Font = Enum.Font.Cartoon
    btnTp.TextSize = 18
    btnTp.MouseButton1Click:Connect(function()
        teleportTo(pos)
    end)
    yTele = yTele + 32
end

local btnSpeed = Instance.new("TextButton", playerFrame)
btnSpeed.Size = UDim2.new(1, 0, 0, 32)
btnSpeed.Position = UDim2.new(0, 0, 0, yTele+5)
btnSpeed.Text = "Boost Velocidade: OFF"
btnSpeed.BackgroundColor3 = Color3.fromRGB(173, 216, 230)
btnSpeed.TextColor3 = Color3.fromRGB(75, 60, 20)
btnSpeed.Font = Enum.Font.Cartoon
btnSpeed.TextSize = 18
btnSpeed.MouseButton1Click:Connect(function()
    speedBoostOn = not speedBoostOn
    btnSpeed.Text = "Boost Velocidade: " .. (speedBoostOn and "ON" or "OFF")
    setSpeed(speedBoostOn and 100 or 16)
end)

local btnFly = Instance.new("TextButton", playerFrame)
btnFly.Size = UDim2.new(1, 0, 0, 32)
btnFly.Position = UDim2.new(0, 0, 0, yTele+42)
btnFly.Text = "Voar: OFF"
btnFly.BackgroundColor3 = Color3.fromRGB(180, 0, 0)
btnFly.TextColor3 = Color3.fromRGB(255,255,255)
btnFly.Font = Enum.Font.Cartoon
btnFly.TextSize = 18
btnFly.MouseButton1Click:Connect(function()
    flyOn = not flyOn
    btnFly.Text = "Voar: " .. (flyOn and "ON" or "OFF")
    fly(flyOn)
end)

-- Seção de Dança
local danceLabel = Instance.new("TextLabel", playerFrame)
danceLabel.Size = UDim2.new(1, 0, 0, 28)
danceLabel.Position = UDim2.new(0, 0, 0, yTele+82)
danceLabel.Text = "Escolha uma dança:"
danceLabel.TextColor3 = Color3.fromRGB(180, 0, 0)
danceLabel.Font = Enum.Font.Cartoon
danceLabel.TextSize = 17
danceLabel.BackgroundTransparency = 1

local yDanceStart = yTele+110
for i, dance in ipairs(dances) do
    local btnDance = Instance.new("TextButton", playerFrame)
    btnDance.Size = UDim2.new(0, 140, 0, 26)
    btnDance.Position = UDim2.new(0, ((i-1)%2)*150, 0, yDanceStart+math.floor((i-1)/2)*28)
    btnDance.Text = dance.name
    btnDance.BackgroundColor3 = Color3.fromRGB(50,50,50)
    btnDance.TextColor3 = Color3.fromRGB(180, 0, 0)
    btnDance.Font = Enum.Font.Cartoon
    btnDance.TextSize = 16
    btnDance.MouseButton1Click:Connect(function()
        -- Para se já está tocando essa dança, senão toca
        if playingDance and playingDance.Animation.AnimationId == dance.id then
            playDance(dance.id) -- para
            btnDance.Text = dance.name
        else
            playDance(dance.id)
            btnDance.Text = dance.name.." (dançando)"
            -- Reset outros botões
            for j, otherDance in ipairs(dances) do
                if j ~= i then
                    local btn = playerFrame:FindFirstChild(otherDance.name)
                    if btn then btn.Text = otherDance.name end
                end
            end
        end
    end)
    btnDance.Name = dance.name
end

-- Círculo flutuante minimizado
local circle = Instance.new("Frame", gui)
circle.Size = UDim2.new(0, 60, 0, 60)
circle.Position = UDim2.new(0, 25, 0, 150)
circle.BackgroundColor3 = Color3.new(0,0,0)
circle.BackgroundTransparency = 0
circle.BorderSizePixel = 0
circle.Visible = false
circle.Active = true
circle.ZIndex = 10
circle.AnchorPoint = Vector2.new(0,0)
circle.ClipsDescendants = true
local circleCorner = Instance.new("UICorner", circle)
circleCorner.CornerRadius = UDim.new(1,0)

local letterP = Instance.new("TextLabel", circle)
letterP.Size = UDim2.new(1, 0, 1, 0)
letterP.Position = UDim2.new(0, 0, 0, 0)
letterP.BackgroundTransparency = 1
letterP.Text = "P"
letterP.TextColor3 = Color3.fromRGB(160, 0, 0)
letterP.Font = Enum.Font.SpecialElite
letterP.TextSize = 48
letterP.TextStrokeTransparency = 0.2
letterP.TextStrokeColor3 = Color3.fromRGB(80,0,0)
letterP.TextScaled = true

local drop = Instance.new("Frame", circle)
drop.Size = UDim2.new(0.16, 0, 0.42, 0)
drop.Position = UDim2.new(0.43, 0, 0.7, 0)
drop.BackgroundColor3 = Color3.fromRGB(160, 0, 0)
drop.BorderSizePixel = 0
drop.BackgroundTransparency = 0.1
local dropCorner = Instance.new("UICorner", drop)
dropCorner.CornerRadius = UDim.new(0.5,0)

-- Arrastar Círculo
local draggingCircle = false
local dragInputCircle, dragStartCircle, startPosCircle
circle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingCircle = true
        dragStartCircle = input.Position
        startPosCircle = circle.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                draggingCircle = false
            end
        end)
    end
end)
circle.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInputCircle = input
    end
end)
UIS.InputChanged:Connect(function(input)
    if draggingCircle and input == dragInputCircle then
        local delta = input.Position - dragStartCircle
        circle.Position = UDim2.new(startPosCircle.X.Scale, startPosCircle.X.Offset + delta.X, startPosCircle.Y.Scale, startPosCircle.Y.Offset + delta.Y)
    end
end)

-- Minimizar/Abrir com tecla G
local minimized = false
UIS.InputBegan:Connect(function(input, isProcessed)
    if isProcessed then return end
    if input.KeyCode == Enum.KeyCode.G then
        minimized = not minimized
        frame.Visible = not minimized
        circle.Visible = minimized
    end
end)

circle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or
       input.UserInputType == Enum.UserInputType.Touch then
        minimized = false
        frame.Visible = true
        circle.Visible = false
    end
end)

game:GetService("StarterGui"):SetCore("SendNotification", {
    Title="PURGE HUB - Construir uma colméia",
    Text="HUB carregado! Aperte G para minimizar/abrir. Clique e segure para arrastar."
})

local player = game.Players.LocalPlayer
local gui = Instance.new("ScreenGui")
gui.Name = "AutoJoin"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 400, 0, 380)
frame.Position = UDim2.new(0.5, -200, 0.5, -190)
frame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
frame.BorderSizePixel = 0
frame.Parent = gui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 16)

-- Título
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, -140, 0, 50)
title.Position = UDim2.new(0, 15, 0, 0)
title.BackgroundTransparency = 1
title.Text = "issei UI AutoJoin 1.1"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 28
title.TextXAlignment = Enum.TextXAlignment.Left

-- Botão Esconder (−)
local hideBtn = Instance.new("TextButton", frame)
hideBtn.Size = UDim2.new(0, 38, 0, 38)
hideBtn.Position = UDim2.new(1, -82, 0, 8)
hideBtn.BackgroundColor3 = Color3.fromRGB(90, 90, 90)
hideBtn.Text = "−"
hideBtn.TextColor3 = Color3.new(1,1,1)
hideBtn.Font = Enum.Font.GothamBold
hideBtn.TextSize = 32
Instance.new("UICorner", hideBtn).CornerRadius = UDim.new(0, 10)

-- Botão Fechar (×)
local closeBtn = Instance.new("TextButton", frame)
closeBtn.Size = UDim2.new(0, 38, 0, 38)
closeBtn.Position = UDim2.new(1, -42, 0, 8)
closeBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
closeBtn.Text = "×"
closeBtn.TextColor3 = Color3.new(1,1,1)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 28
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 10)

-- Job Box / Status / Turbo / Contador
local jobBox = Instance.new("TextBox", frame)
jobBox.Size = UDim2.new(1,-40,0,80)
jobBox.Position = UDim2.new(0,20,0,60)
jobBox.BackgroundColor3 = Color3.fromRGB(35,35,35)
jobBox.PlaceholderText = "Cole o Job ID aqui"
jobBox.TextColor3 = Color3.new(1,1,1)
jobBox.Font = Enum.Font.Gotham
jobBox.TextSize = 18
jobBox.MultiLine = true
Instance.new("UICorner", jobBox).CornerRadius = UDim.new(0,12)

local status = Instance.new("TextLabel", frame)
status.Size = UDim2.new(1,-40,0,40)
status.Position = UDim2.new(0,20,0,150)
status.BackgroundTransparency = 1
status.Text = "Pronto - Intervalo: 500ms"
status.TextColor3 = Color3.fromRGB(100,255,100)
status.Font = Enum.Font.GothamBold
status.TextSize = 20

local turboBtn = Instance.new("TextButton", frame)
turboBtn.Size = UDim2.new(1,-40,0,90)
turboBtn.Position = UDim2.new(0,20,0,210)
turboBtn.BackgroundColor3 = Color3.fromRGB(220,60,60)
turboBtn.Text = "OFF"
turboBtn.TextColor3 = Color3.new(1,1,1)
turboBtn.Font = Enum.Font.GothamBold
turboBtn.TextSize = 50
Instance.new("UICorner", turboBtn).CornerRadius = UDim.new(0,16)

local counter = Instance.new("TextLabel", frame)
counter.Size = UDim2.new(1,-40,0,30)
counter.Position = UDim2.new(0,20,0,305)
counter.BackgroundTransparency = 1
counter.Text = "Tentativas: 0"
counter.TextColor3 = Color3.fromRGB(200,200,200)
counter.Font = Enum.Font.Gotham
counter.TextSize = 16

-- ARRSTAR
local dragging = false
local dragStart, startPos
title.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true dragStart = i.Position startPos = frame.Position end end)
title.InputChanged:Connect(function(i) if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then local d = i.Position - dragStart frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y) end end)
game:GetService("UserInputService").InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)

-- MINIMIZAR / FECHAR
local hidden = false
local function toggle()
    hidden = not hidden
    frame.Visible = not hidden
    hideBtn.Text = hidden and "+" or "−"
end
hideBtn.MouseButton1Click:Connect(toggle)
closeBtn.MouseButton1Click:Connect(function() gui:Destroy() end)

-- CTRL ESQUERDO FUNCIONAL MESMO COM UI MINIMIZADA
local CAS = game:GetService("ContextActionService")
CAS:BindAction("ToggleAutoJoinUI", function(_, state)
    if state == Enum.UserInputState.Begin then
        toggle()
    end
end, false, Enum.KeyCode.LeftControl)

-- AUTOJOIN
local running = false
local attempts = 0
local loopThread = nil
local function getCleanJobId()
    local t = jobBox.Text or ""
    t = t:gsub('[%"%\']', ''):gsub("%s+", "")
    local id = t:match("([0-9a-fA-F%-]+)")
    return (id and #id == 36) and id or nil
end
local function teleport()
    local id = getCleanJobId()
    if not id then return end
    attempts += 1
    counter.Text = "Tentativas: " .. attempts
    status.Text = "Tentando... ("..attempts..")"
    pcall(function() game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, id, player) end)
    task.delay(0.1, function()
        local sb = game:GetService("ReplicatedStorage"):FindFirstChild("__ServerBrowser")
        if sb then pcall(function() sb:InvokeServer("teleport", id) end) end
    end)
end
turboBtn.MouseButton1Click:Connect(function()
    running = not running
    if running then
        if not getCleanJobId() then status.Text = "Job ID inválido!" status.TextColor3 = Color3.fromRGB(255,100,100) running = false return end
        turboBtn.Text = "ON" turboBtn.BackgroundColor3 = Color3.fromRGB(50,220,50)
        status.Text = "AUTOJOIN ATIVADO" status.TextColor3 = Color3.fromRGB(100,255,100)
        loopThread = task.spawn(function() while running do teleport() task.wait(0.5) end end)
    else
        turboBtn.Text = "OFF" turboBtn.BackgroundColor3 = Color3.fromRGB(220,60,60)
        status.Text = "AutoJoin DESATIVADO"
        if loopThread then task.cancel(loopThread) end
    end
end)

frame.Visible = true
jobBox:CaptureFocus()

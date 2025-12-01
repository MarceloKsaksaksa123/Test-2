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
title.Size = UDim2.new(1,0,0,50)
title.BackgroundTransparency = 1
title.Text = "AutoJoin 1.0"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 28

-- Job ID Box
local jobBox = Instance.new("TextBox", frame)
jobBox.Size = UDim2.new(1,-40,0,80)
jobBox.Position = UDim2.new(0,20,0,60)
jobBox.BackgroundColor3 = Color3.fromRGB(35,35,35)
jobBox.Text = ""
jobBox.PlaceholderText = 'Cole o Job ID (com ou sem aspas)\nEx: "7b8acb05-f414-46ab-8787-b4a4f1830eb7"'
jobBox.TextColor3 = Color3.new(1,1,1)
jobBox.Font = Enum.Font.Gotham
jobBox.TextSize = 18
jobBox.ClearTextOnFocus = false
jobBox.MultiLine = true
Instance.new("UICorner", jobBox).CornerRadius = UDim.new(0,12)

-- Status
local status = Instance.new("TextLabel", frame)
status.Size = UDim2.new(1,-40,0,40)
status.Position = UDim2.new(0,20,0,150)
status.BackgroundTransparency = 1
status.Text = "Pronto - Intervalo: 500ms"
status.TextColor3 = Color3.fromRGB(100,255,100)
status.Font = Enum.Font.GothamBold
status.TextSize = 20

-- Botão TURBO ON/OFF
local turboBtn = Instance.new("TextButton", frame)
turboBtn.Size = UDim2.new(1,-40,0,90)
turboBtn.Position = UDim2.new(0,20,0,210)
turboBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
turboBtn.Text = "OFF"
turboBtn.TextColor3 = Color3.new(1,1,1)
turboBtn.Font = Enum.Font.GothamBold
turboBtn.TextSize = 50
Instance.new("UICorner", turboBtn).CornerRadius = UDim.new(0,16)

-- Contador
local counter = Instance.new("TextLabel", frame)
counter.Size = UDim2.new(1,-40,0,30)
counter.Position = UDim2.new(0,20,0,305)
counter.BackgroundTransparency = 1
counter.Text = "Tentativas: 0"
counter.TextColor3 = Color3.fromRGB(200,200,200)
counter.Font = Enum.Font.Gotham
counter.TextSize = 16

-- ==================== ARRASTAR A JANELA (MOVER) ====================
local dragging = false
local dragStart = nil
local startPos = nil

title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position
    end
end)

title.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)

game:GetService("UserInputService").InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)
-- ====================================================================

-- Variáveis
local running = false
local attempts = 0
local loopThread = nil

-- FUNÇÃO MELHORADA: aceita com aspas, espaços, quebras de linha, etc.
local function getCleanJobId()
    local text = jobBox.Text or ""
    text = text:gsub('"', ''):gsub("'", ""):gsub("%s+", "")
    local jobid = text:match("([%w%-]+)")
    if jobid and #jobid == 36 and jobid:find("-") then
        return jobid
    end
    return nil
end

local function

 teleport()
    local jobid = getCleanJobId()
    if not jobid then
        status.Text = "Job ID inválido!"
        status.TextColor3 = Color3.fromRGB(255,100,100)
        return
    end

    attempts += 1
    counter.Text = "Tentativas: " .. attempts
    status.Text = "Tentando entrar... (" .. attempts .. ")"
    status.TextColor3 = Color3.fromRGB(255,255,100)

    pcall(function()
        game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, jobid, player)
    end)

    task.delay(0.1, function()
        local sb = game:GetService("ReplicatedStorage"):FindFirstChild("__ServerBrowser")
        if sb then
            pcall(function()
                sb:InvokeServer("teleport", jobid)
            end)
        end
    end)
end

-- Botão ON/OFF
turboBtn.MouseButton1Click:Connect(function()
    running = not running

    if running then
        local jobid = getCleanJobId()
        if not jobid then
            status.Text = "Cole um Job ID válido primeiro!"
            status.TextColor3 = Color3.fromRGB(255,100,100)
            running = false
            return
        end

        turboBtn.Text = "ON"
        turboBtn.BackgroundColor3 = Color3.fromRGB(50, 220, 50)
        status.Text = "AUTOJOIN ATIVADO - 500ms"
        status.TextColor3 = Color3.fromRGB(100,255,100)

        loopThread = task.spawn(function()
            while running do
                teleport()
                task.wait(0.5) -- 500ms turbo
            end
        end)
    else
        turboBtn.Text = "OFF"
        turboBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
        status.Text = "AutoJoin DESATIVADO"
        status.TextColor3 = Color3.fromRGB(255,150,150)
        if loopThread then task.cancel(loopThread) end
    end
end)

-- Abre automaticamente + foco na caixa
frame.Visible = true
jobBox:CaptureFocus()

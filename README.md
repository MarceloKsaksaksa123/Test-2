local player = game.Players.LocalPlayer
local gui = Instance.new("ScreenGui")
gui.Name = "VipTP"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 400, 0, 280)
frame.Position = UDim2.new(0.5, -200, 0.5, -140)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.BorderSizePixel = 0
frame.Parent = gui

Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)

-- Título
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,50)
title.BackgroundTransparency = 1
title.Text = "VIP SERVER TELEPORTER"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 24

-- Caixa de texto
local textbox = Instance.new("TextBox", frame)
textbox.Size = UDim2.new(1,-40,0,80)
textbox.Position = UDim2.new(0,20,0,70)
textbox.BackgroundColor3 = Color3.fromRGB(40,40,40)
textbox.PlaceholderText = "Cole o Job ID aqui..."
textbox.Text = ""
textbox.TextColor3 = Color3.new(1,1,1)
textbox.Font = Enum.Font.Gotham
textbox.TextSize = 18
textbox.ClearTextOnFocus = false
textbox.MultiLine = true
textbox.TextWrapped = true
Instance.new("UICorner", textbox).CornerRadius = UDim.new(0,8)

-- BOTÃO ENORME DE TELEPORT (impossível de não ver) kkkkkkkkkkkkkkkkkk
local tpbtn = Instance.new("TextButton", frame)
tpbtn.Size = UDim2.new(1,-40,0,70)
tpbtn.Position = UDim2.new(0,20,1,-90)
tpbtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
tpbtn.Text = "TELEPORTAR AGORA"
tpbtn.TextColor3 = Color3.new(1,1,1)
tpbtn.Font = Enum.Font.GothamBold
tpbtn.TextSize = 28
Instance.new("UICorner", tpbtn).CornerRadius = UDim.new(0,12)

-- Status
local status = Instance.new("TextLabel", frame)
status.Size = UDim2.new(1,-40,0,30)
status.Position = UDim2.new(0,20,1,-120)
status.BackgroundTransparency = 1
status.Text = "Pronto - clique no botão azul"
status.TextColor3 = Color3.fromRGB(100,255,100)
status.Font = Enum.Font.Gotham
status.TextSize = 16

-- FUNÇÃO DE TELEPORT (2 métodos - um sempre funciona)
tpbtn.MouseButton1Click:Connect(function()
    local jobid = textbox.Text:gsub("%s", ""):gsub("\n", "")
    
    if #jobid ~= 36 or not jobid:find("-") then
        status.Text = "Job ID inválido!"
        status.TextColor3 = Color3.fromRGB(255,100,100)
        return
    end

    status.Text = "Teleportando..."
    status.TextColor3 = Color3.fromRGB(255,255,100)

    -- MÉTODO 1 (o que mais funciona no Brainrot)
    spawn(function()
        local success, err = pcall(function()
            game:GetService("TeleportService"):TeleportToPlaceInstance(109983668079237, jobid, player)
        end)
        
        wait(3)
        if not success then
            -- MÉTODO 2 (ServerBrowser - funciona em 99% dos casos)
            local sb = game:GetService("ReplicatedStorage"):FindFirstChild("__ServerBrowser")
            if sb then
                sb:InvokeServer("teleport", jobid)
            end
        end
    end)
end)

-- Abre automaticamente
frame.Visible = true
textbox:CaptureFocus()
textbox.SelectionStart = 1
textbox.CursorPosition = #textbox.Text + 1

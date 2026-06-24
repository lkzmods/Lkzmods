-- // CONFIGURAÇÕES DO SCRIPT // --
local ChaveCorreta = "Lkz"
local Settings = {
    ScriptActive = false,
    MaxDistance = 10,
    Cooldown = false
}

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local TextChatService = game:GetService("TextChatService")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")

-- // REMOVE ANTIGOS // --
if CoreGui:FindFirstChild("MetropolesAutofarm") then CoreGui.MetropolesAutofarm:Destroy() end

-- // SISTEMA DE ENVIO (3 PRIMEIRAS LETRAS DO NOME) // --
local function EnviarComandoChat(TargetPlayer)
    -- Pega apenas as 3 primeiras letras do nome de usuário real do Roblox
    local TresLetras = string.sub(TargetPlayer.Name, 1, 3)
    local Mensagem = "/revistar " .. TresLetras
    
    if TextChatService and TextChatService:FindFirstChild("TextChannels") and TextChatService.TextChannels:FindFirstChild("RBXGeneral") then
        TextChatService.TextChannels.RBXGeneral:SendAsync(Mensagem)
    else
        game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer(Mensagem, "All")
    end
end

-- // LOOP DE DETECÇÃO DE CORPO MORTO (REPETÍVEL PARA QUALQUER UM) // --
RunService.Heartbeat:Connect(function()
    if not Settings.ScriptActive or Settings.Cooldown then return end
    local MeuChar = LocalPlayer.Character
    if not MeuChar or not MeuChar:FindFirstChild("HumanoidRootPart") then return end

    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("HumanoidRootPart") and v.Character:FindFirstChild("Humanoid") then
            local InimigoChar = v.Character
            local Distancia = (MeuChar.HumanoidRootPart.Position - InimigoChar.HumanoidRootPart.Position).Magnitude
            
            -- Detecta se está morto e dentro do raio
            if InimigoChar.Humanoid.Health <= 0 and Distancia <= Settings.MaxDistance then
                Settings.Cooldown = true
                
                -- Executa a revista usando as 3 letras
                EnviarComandoChat(v)
                
                -- Pequena pausa para o chat registrar e liberar a próxima pessoa livremente
                task.wait(2.5)
                Settings.Cooldown = false
                break
            end
        end
    end
end)

-- // FUNÇÃO PARA TORNAR A UI ARRASTÁVEL NO MOBILE // --
local function MakeDraggable(Frame)
    local dragging, dragInput, dragStart, startPos
    
    Frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = Frame.Position
            
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    
    Frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            Frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- // INTERFACE VISUAL (PRETO E AZUL) // --
local ScreenGui = Instance.new("ScreenGui", CoreGui)
ScreenGui.Name = "MetropolesAutofarm"
ScreenGui.ResetOnSpawn = false

-- 1. TELA DE LOGIN (SISTEMA DE KEY)
local LoginFrame = Instance.new("Frame", ScreenGui)
LoginFrame.Size = UDim2.new(0, 260, 0, 150)
LoginFrame.Position = UDim2.new(0.5, -130, 0.4, -75)
LoginFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
LoginFrame.BorderSizePixel = 0
LoginFrame.Active = true
Instance.new("UICorner", LoginFrame).CornerRadius = UDim.new(0, 10)
MakeDraggable(LoginFrame)

local LoginTitle = Instance.new("TextLabel", LoginFrame)
LoginTitle.Size = UDim2.new(1, 0, 0, 35)
LoginTitle.BackgroundColor3 = Color3.fromRGB(20, 25, 40)
LoginTitle.Text = "SISTEMA DE KEY | SILV"
LoginTitle.TextColor3 = Color3.fromRGB(0, 150, 255)
LoginTitle.Font = Enum.Font.GothamBold
LoginTitle.TextSize = 13
Instance.new("UICorner", LoginTitle).CornerRadius = UDim.new(0, 10)

local KeyInput = Instance.new("TextBox", LoginFrame)
KeyInput.Size = UDim2.new(0.85, 0, 0, 35)
KeyInput.Position = UDim2.new(0.075, 0, 0.38, 0)
KeyInput.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
KeyInput.PlaceholderText = "Insira sua Key..."
KeyInput.Text = ""
KeyInput.TextColor3 = Color3.new(1, 1, 1)
KeyInput.Font = Enum.Font.Gotham
KeyInput.TextSize = 14
Instance.new("UICorner", KeyInput)

local KeyBtn = Instance.new("TextButton", LoginFrame)
KeyBtn.Size = UDim2.new(0.85, 0, 0, 35)
KeyBtn.Position = UDim2.new(0.075, 0, 0.68, 0)
KeyBtn.BackgroundColor3 = Color3.fromRGB(0, 100, 255)
KeyBtn.Text = "Verificar Key"
KeyBtn.TextColor3 = Color3.new(1, 1, 1)
KeyBtn.Font = Enum.Font.GothamBold
KeyBtn.TextSize = 14
Instance.new("UICorner", KeyBtn)

-- 2. TELA DO PAINEL PRINCIPAL
local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 260, 0, 180)
MainFrame.Position = UDim2.new(0.5, -130, 0.4, -90)
MainFrame.BackgroundColor3 = Color3.fromRGB(12, 12, 16)
MainFrame.BorderSizePixel = 0
MainFrame.Visible = false
MainFrame.Active = true
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)
MakeDraggable(MainFrame)

local MainTitle = Instance.new("TextLabel", MainFrame)
MainTitle.Size = UDim2.new(1, 0, 0, 35)
MainTitle.BackgroundColor3 = Color3.fromRGB(18, 22, 35)
MainTitle.Text = "METRÓPOLES RP | AUTO REVISTAR"
MainTitle.TextColor3 = Color3.fromRGB(0, 160, 255)
MainTitle.Font = Enum.Font.GothamBold
MainTitle.TextSize = 12
Instance.new("UICorner", MainTitle).CornerRadius = UDim.new(0, 12)

-- Botão de Ligar/Desligar
local ToggleBtn = Instance.new("TextButton", MainFrame)
ToggleBtn.Size = UDim2.new(0.9, 0, 0, 45)
ToggleBtn.Position = UDim2.new(0.05, 0, 0.28, 0)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
ToggleBtn.Text = "Status: DESLIGADO"
ToggleBtn.Font = Enum.Font.GothamSemibold
ToggleBtn.TextColor3 = Color3.fromRGB(255, 80, 80)
ToggleBtn.TextSize = 14
Instance.new("UICorner", ToggleBtn)

ToggleBtn.MouseButton1Click:Connect(function()
    Settings.ScriptActive = not Settings.ScriptActive
    ToggleBtn.Text = Settings.ScriptActive and "Status: LIGADO" or "Status: DESLIGADO"
    ToggleBtn.TextColor3 = Settings.ScriptActive and Color3.fromRGB(80, 255, 80) or Color3.fromRGB(255, 80, 80)
    ToggleBtn.BackgroundColor3 = Settings.ScriptActive and Color3.fromRGB(15, 35, 25) or Color3.fromRGB(25, 25, 30)
end)

-- Slider de Distância
local SliderLabel = Instance.new("TextLabel", MainFrame)
SliderLabel.Size = UDim2.new(0.9, 0, 0, 20)
SliderLabel.Position = UDim2.new(0.05, 0, 0.58, 0)
SliderLabel.BackgroundTransparency = 1
SliderLabel.Text = "Distância Ativa: " .. Settings.MaxDistance .. " studs"
SliderLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
SliderLabel.Font = Enum.Font.Gotham
SliderLabel.TextSize = 12
SliderLabel.TextXAlignment = Enum.TextXAlignment.Left

local SliderBack = Instance.new("Frame", MainFrame)
SliderBack.Size = UDim2.new(0.9, 0, 0, 8)
SliderBack.Position = UDim2.new(0.05, 0, 0.75, 0)
SliderBack.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
Instance.new("UICorner", SliderBack)

local SliderBtn = Instance.new("TextButton", SliderBack)
SliderBtn.Size = UDim2.new(0, 16, 0, 16)
SliderBtn.Position = UDim2.new(0.32, -8, -0.5, 0)
SliderBtn.BackgroundColor3 = Color3.fromRGB(0, 130, 255)
SliderBtn.Text = ""
Instance.new("UICorner", SliderBtn)

local sliderDragging = false
SliderBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        sliderDragging = true
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        sliderDragging = false
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if sliderDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local relativeX = input.Position.X - SliderBack.AbsolutePosition.X
        local percentage = math.clamp(relativeX / SliderBack.AbsoluteSize.X, 0, 1)
        SliderBtn.Position = UDim2.new(percentage, -8, -0.5, 0)
        
        local minDist = 3
        local maxDist = 35
        Settings.MaxDistance = math.floor(minDist + (percentage * (maxDist - minDist)))
        SliderLabel.Text = "Distância Ativa: " .. Settings.MaxDistance .. " studs"
    end
end)

-- 3. BOTÃO FLUTUANTE DE MINIMIZAR (BOTÃO "Silv")
local MinimizeBtn = Instance.new("TextButton", ScreenGui)
MinimizeBtn.Size = UDim2.new(0, 50, 0, 50)
MinimizeBtn.Position = UDim2.new(0.02, 0, 0.2, 0)
MinimizeBtn.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
MinimizeBtn.Text = "Silv"
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.TextColor3 = Color3.fromRGB(0, 150, 255)
MinimizeBtn.TextSize = 12
MinimizeBtn.Visible = false
local MinCorner = Instance.new("UICorner", MinimizeBtn)
MinCorner.CornerRadius = UDim.new(0, 25)
local MinStroke = Instance.new("UIStroke", MinimizeBtn)
MinStroke.Color = Color3.fromRGB(0, 100, 255)
MinStroke.Thickness = 1.5
MakeDraggable(MinimizeBtn)

MinimizeBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- LOGICA DE VERIFICAÇÃO DA KEY
KeyBtn.MouseButton1Click:Connect(function()
    if KeyInput.Text == ChaveCorreta then
        LoginFrame:Destroy()
        MainFrame.Visible = true
        MinimizeBtn.Visible = true
    else
        KeyBtn.Text = "KEY INCORRETA!"
        KeyBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        task.wait(1.5)
        KeyBtn.Text = "Verificar Key"
        KeyBtn.BackgroundColor3 = Color3.fromRGB(0, 100, 255)
    end
end)


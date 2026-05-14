local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Lkz Mods",
   LoadingTitle = "Lkz Mods | Clean ESP",
   LoadingSubtitle = "Ajuste de Ativação Rígida",
   ConfigurationSaving = { Enabled = false }
})

-- // CONFIGURAÇÕES // --
local Settings = {
    Aimbot = false,
    ShowFOV = false,
    TeamCheck = false,
    Precision = 45,
    FOV = 120,
    MaxDistance = 500,
    TargetPart = "Head",
    ESP = false,
    Heal = false,
    Skeleton = false,
    Tracers = false,
    MaxDistanceESP = 1500
}

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera

-- // FUNÇÃO DE VISIBILIDADE // --
local function IsVisible(TargetPart)
    if not TargetPart then return false end
    local Origin = Camera.CFrame.Position
    local Direction = (TargetPart.Position - Origin)
    
    local RayParams = RaycastParams.new()
    RayParams.FilterDescendantsInstances = {LocalPlayer.Character, Camera}
    RayParams.FilterType = Enum.RaycastFilterType.Exclude
    RayParams.IgnoreWater = true
    
    local Result = workspace:Raycast(Origin, Direction, RayParams)
    
    if Result then
        return Result.Instance:IsDescendantOf(TargetPart.Parent)
    end
    return false
end

-- // FUNÇÃO DE DESENHO // --
local function CreateLine(thickness, color)
    local L = Drawing.new("Line")
    L.Thickness = thickness or 1
    L.Color = color or Color3.fromRGB(255, 0, 0)
    L.Transparency = 1
    L.Visible = false
    return L
end

-- // SISTEMA DE VISUAIS // --
local function CreateESP(Player)
    local Box = Drawing.new("Square")
    Box.Visible = false
    Box.Thickness = 1
    Box.Filled = false 

    local HealthOutline = CreateLine(2, Color3.new(0,0,0))
    local HealthLine = CreateLine(1, Color3.new(0,255,0))
    local Tracer = CreateLine(1, Color3.fromRGB(255, 0, 0))
    
    local DistanceText = Drawing.new("Text")
    DistanceText.Size = 14
    DistanceText.Center = true
    DistanceText.Outline = true
    DistanceText.Color = Color3.fromRGB(255, 255, 255)
    DistanceText.Visible = false
    
    local Bones = {
        H_T = CreateLine(1), T_LUA = CreateLine(1), T_RUA = CreateLine(1),
        LUA_LLA = CreateLine(1), RUA_RLA = CreateLine(1), T_LUL = CreateLine(1), T_RUL = CreateLine(1),
        LUL_LLL = CreateLine(1), RUL_RLL = CreateLine(1)
    }

    RunService.RenderStepped:Connect(function()
        local Char = Player.Character
        if Char and Char:FindFirstChild("Humanoid") and Player ~= LocalPlayer and Char.Humanoid.Health > 0 then
            local Root = Char:FindFirstChild("HumanoidRootPart")
            local Head = Char:FindFirstChild("Head")
            
            if Root and Head then
                local RootP, OnScreen = Camera:WorldToViewportPoint(Root.Position)
                local MyRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                local Distance = MyRoot and (Root.Position - MyRoot.Position).Magnitude or 0
                
                if OnScreen and Distance <= Settings.MaxDistanceESP then
                    local Visible = IsVisible(Head)
                    local CurrentColor = Visible and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)

                    local SizeY = (Char:GetModelSize().Y) * 1.2
                    local Top = Camera:WorldToViewportPoint(Root.Position + Vector3.new(0, SizeY/2, 0))
                    local Bottom = Camera:WorldToViewportPoint(Root.Position - Vector3.new(0, SizeY/2, 0))
                    local Height = math.abs(Top.Y - Bottom.Y)
                    local Width = Height / 1.8

                    if Settings.ESP then
                        Box.Position = Vector2.new(RootP.X - Width/2, RootP.Y - Height/2)
                        Box.Size = Vector2.new(Width, Height)
                        Box.Color = CurrentColor
                        Box.Visible = true
                        DistanceText.Position = Vector2.new(Box.Position.X + Width/2, Box.Position.Y + Height + 1)
                        DistanceText.Text = math.floor(Distance) .. "m"
                        DistanceText.Visible = true
                    else Box.Visible = false DistanceText.Visible = false end

                    if Settings.Tracers then
                        Tracer.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
                        Tracer.To = Vector2.new(RootP.X, RootP.Y + (Height/2))
                        Tracer.Color = CurrentColor
                        Tracer.Visible = true
                    else Tracer.Visible = false end

                    if Settings.Heal then
                        local HP = math.clamp(Char.Humanoid.Health / Char.Humanoid.MaxHealth, 0, 1)
                        HealthOutline.From = Vector2.new(Box.Position.X - 5, Box.Position.Y + Box.Size.Y)
                        HealthOutline.To = Vector2.new(Box.Position.X - 5, Box.Position.Y)
                        HealthOutline.Visible = true
                        HealthLine.From = HealthOutline.From
                        HealthLine.To = Vector2.new(Box.Position.X - 5, Box.Position.Y + Box.Size.Y - (Box.Size.Y * HP))
                        HealthLine.Color = Color3.new(1 - HP, HP, 0)
                        HealthLine.Visible = true
                    else HealthLine.Visible = false HealthOutline.Visible = false end

                    if Settings.Skeleton then
                        local function GetPos(n) 
                            local p = Char:FindFirstChild(n) 
                            if p then local v, os = Camera:WorldToViewportPoint(p.Position) return os and Vector2.new(v.X, v.Y) end 
                            return nil 
                        end
                        local function Bind(l, a, b) 
                            if a and b then 
                                l.From = a l.To = b l.Color = CurrentColor l.Visible = true 
                            else l.Visible = false end 
                        end
                        
                        local Torso = Char:FindFirstChild("UpperTorso") or Char:FindFirstChild("Torso")
                        local LUA = Char:FindFirstChild("LeftUpperArm") or Char:FindFirstChild("Left Arm")
                        local RUA = Char:FindFirstChild("RightUpperArm") or Char:FindFirstChild("Right Arm")
                        local LLA = Char:FindFirstChild("LeftLowerArm") or Char:FindFirstChild("Left Arm")
                        local RLA = Char:FindFirstChild("RightLowerArm") or Char:FindFirstChild("Right Arm")
                        local LUL = Char:FindFirstChild("LeftUpperLeg") or Char:FindFirstChild("Left Leg")
                        local RUL = Char:FindFirstChild("RightUpperLeg") or Char:FindFirstChild("Right Leg")
                        local LLL = Char:FindFirstChild("LeftLowerLeg") or Char:FindFirstChild("Left Leg")
                        local RLL = Char:FindFirstChild("RightLowerLeg") or Char:FindFirstChild("Right Leg")

                        Bind(Bones.H_T, GetPos("Head"), GetPos(Torso.Name))
                        Bind(Bones.T_LUA, GetPos(Torso.Name), GetPos(LUA.Name))
                        Bind(Bones.T_RUA, GetPos(Torso.Name), GetPos(RUA.Name))
                        Bind(Bones.LUA_LLA, GetPos(LUA.Name), GetPos(LLA.Name))
                        Bind(Bones.RUA_RLA, GetPos(RUA.Name), GetPos(RLA.Name))
                        Bind(Bones.T_LUL, GetPos(Torso.Name), GetPos(LUL.Name))
                        Bind(Bones.T_RUL, GetPos(Torso.Name), GetPos(RUL.Name))
                        Bind(Bones.LUL_LLL, GetPos(LUL.Name), GetPos(LLL.Name))
                        Bind(Bones.RUL_RLL, GetPos(RUL.Name), GetPos(RLL.Name))
                    else for _, l in pairs(Bones) do l.Visible = false end end
                else
                    Box.Visible = false DistanceText.Visible = false Tracer.Visible = false HealthLine.Visible = false HealthOutline.Visible = false
                    for _, l in pairs(Bones) do l.Visible = false end
                end
            end
        else
            Box.Visible = false DistanceText.Visible = false Tracer.Visible = false HealthLine.Visible = false HealthOutline.Visible = false
            for _, l in pairs(Bones) do l.Visible = false end
        end
    end)
end

for _, v in pairs(Players:GetPlayers()) do CreateESP(v) end
Players.PlayerAdded:Connect(CreateESP)

-- // INTERFACE // --
local AimTab = Window:CreateTab("AIMBOT")
AimTab:CreateToggle({Name = "Ativar Aimbot", CurrentValue = false, Callback = function(V) Settings.Aimbot = V end})
AimTab:CreateToggle({Name = "Exibir FOV", CurrentValue = false, Callback = function(V) Settings.ShowFOV = V end})
AimTab:CreateToggle({Name = "Team Check", CurrentValue = false, Callback = function(V) Settings.TeamCheck = V end})
AimTab:CreateSlider({Name = "FOV", Range = {10, 600}, Increment = 1, CurrentValue = 120, Callback = function(V) Settings.FOV = V end})
AimTab:CreateSlider({Name = "Precisao (%)", Range = {0, 100}, Increment = 1, CurrentValue = 45, Callback = function(V) Settings.Precision = V end})
AimTab:CreateSlider({Name = "Distancia Max", Range = {10, 2000}, Increment = 10, CurrentValue = 500, Callback = function(V) Settings.MaxDistance = V end})

local VisualTab = Window:CreateTab("VISUAL")
VisualTab:CreateToggle({Name = "ESP Box + Distancia", CurrentValue = false, Callback = function(V) Settings.ESP = V end})
VisualTab:CreateToggle({Name = "Tracers", CurrentValue = false, Callback = function(V) Settings.Tracers = V end})
VisualTab:CreateToggle({Name = "Esqueleto", CurrentValue = false, Callback = function(V) Settings.Skeleton = V end})
VisualTab:CreateToggle({Name = "Vida", CurrentValue = false, Callback = function(V) Settings.Heal = V end})

-- // LOOP AIMBOT E FOV // --
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 0.5
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
FOVCircle.Transparency = 0.4
FOVCircle.NumSides = 64

RunService.RenderStepped:Connect(function()
    local ScreenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    FOVCircle.Visible = Settings.ShowFOV
    FOVCircle.Radius = Settings.FOV
    FOVCircle.Position = ScreenCenter

    -- Verificação de segurança: se o botão estiver OFF, nada abaixo será executado
    if Settings.Aimbot == false then return end

    local Target = nil
    local ShortestDistance = Settings.FOV
    
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild(Settings.TargetPart) and v.Character.Humanoid.Health > 0 then
            if Settings.TeamCheck and v.Team == LocalPlayer.Team then continue end
            
            local Part = v.Character[Settings.TargetPart]
            local Pos, OnScreen = Camera:WorldToViewportPoint(Part.Position)
            local MyRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            
            if MyRoot and OnScreen then
                local Dist = (Part.Position - MyRoot.Position).Magnitude
                if Dist <= Settings.MaxDistance then
                    if IsVisible(Part) then
                        local Mag = (Vector2.new(Pos.X, Pos.Y) - ScreenCenter).Magnitude
                        if Mag < ShortestDistance then
                            Target = Part
                            ShortestDistance = Mag
                        end
                    end
                end
            end
        end
    end
    
    if Target and Settings.Aimbot then -- Segunda checagem para evitar delay de desativação
        local Alpha = math.clamp(Settings.Precision / 100, 0.01, 1)
        Camera.CFrame = Camera.CFrame:Lerp(CFrame.lookAt(Camera.CFrame.Position, Target.Position), Alpha)
    end
end)

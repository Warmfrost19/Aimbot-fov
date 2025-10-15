--// Aimbot educativo com FOV, parede, menu branco e mira instantânea (gruda forte e real)
-- Use SOMENTE para fins educativos no Roblox Studio!.

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera

-- Parâmetro do FOV
local FOV_RADIUS = 120 -- pixels

-- Estado do aimbot
local AimbotEnabled = false

-- Menu branco
local gui = Instance.new("ScreenGui")
gui.Name = "AimbotFOVMenu"
gui.ResetOnSpawn = false
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 180, 0, 90)
frame.Position = UDim2.new(0, 16, 0, 16)
frame.BackgroundColor3 = Color3.new(1,1,1)
frame.BackgroundTransparency = 0.05
frame.Parent = gui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0.18,0)

local title = Instance.new("TextLabel")
title.Parent = frame
title.Size = UDim2.new(1, 0, 0.4, 0)
title.Position = UDim2.new(0,0,0,0)
title.BackgroundTransparency = 1
title.Text = "Aimbot FOV"
title.TextScaled = true
title.Font = Enum.Font.SourceSansBold
title.TextColor3 = Color3.new(0.1,0.1,0.1)

local btnToggle = Instance.new("TextButton")
btnToggle.Parent = frame
btnToggle.Size = UDim2.new(0.8, 0, 0.35, 0)
btnToggle.Position = UDim2.new(0.1, 0, 0.5, 0)
btnToggle.BackgroundColor3 = Color3.fromRGB(220,220,220)
btnToggle.TextColor3 = Color3.new(0.2,0.2,0.2)
btnToggle.TextScaled = true
btnToggle.Font = Enum.Font.SourceSansBold
btnToggle.Text = "Ativar Aimbot"
Instance.new("UICorner", btnToggle).CornerRadius = UDim.new(0.4,0)

btnToggle.MouseButton1Click:Connect(function()
	AimbotEnabled = not AimbotEnabled
	btnToggle.Text = AimbotEnabled and "Desativar Aimbot" or "Ativar Aimbot"
end)

-- Desenhar círculo FOV vermelho usando Drawing API (linha sempre aparece)
local DrawingCircle = Drawing.new("Circle")
DrawingCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
DrawingCircle.Radius = FOV_RADIUS
DrawingCircle.Color = Color3.fromRGB(255,0,0)
DrawingCircle.Thickness = 2
DrawingCircle.Transparency = 1
DrawingCircle.NumSides = 100
DrawingCircle.Filled = false
DrawingCircle.Visible = true

-- Atualiza posição do círculo caso a tela mude de tamanho
RunService.RenderStepped:Connect(function()
    DrawingCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
end)

-- Checagem de visibilidade (sem parede)
local function IsVisible(headPos)
    local origin = Camera.CFrame.Position
    local direction = (headPos - origin).Unit
    local distance = (headPos - origin).Magnitude
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Blacklist
    params.FilterDescendantsInstances = {LocalPlayer.Character}
    local result = workspace:Raycast(origin, direction * distance, params)
    -- Se não bateu em nada OU só bateu na própria cabeça, está visível
    if not result or (result.Instance and result.Position and (result.Position - headPos).Magnitude < 1) then
        return true
    end
    return false
end

-- Função para encontrar o inimigo mais próximo DENTRO do FOV E visível
local function GetClosestPlayerInFOVVisible()
    local closest = nil
    local shortestDist = math.huge
    local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            -- Ignora aliados se houver times
            if player.Team and LocalPlayer.Team and player.Team == LocalPlayer.Team then
                continue
            end
            local headPos = player.Character.Head.Position
            local screenPos, onScreen = Camera:WorldToViewportPoint(headPos)
            if onScreen then
                local pos2D = Vector2.new(screenPos.X, screenPos.Y)
                local dist = (pos2D - center).Magnitude
                if dist <= FOV_RADIUS and dist < shortestDist and IsVisible(headPos) then
                    shortestDist = dist
                    closest = player
                end
            end
        end
    end
    return closest
end

-- Mira INSTANTÂNEA (gruda forte e real)
RunService.RenderStepped:Connect(function()
	if AimbotEnabled then
		local target = GetClosestPlayerInFOVVisible()
		if target and target.Character and target.Character:FindFirstChild("Head") then
			local headPos = target.Character.Head.Position
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, headPos)
		end
	end
end)

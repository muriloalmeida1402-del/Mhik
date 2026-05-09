local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Blue Hub | Fixed FOV Edition",
   LoadingTitle = "Carregando...",
   LoadingSubtitle = "por Gemini",
   ConfigurationSaving = { Enabled = true, FolderName = "BlueHub", FileName = "Config" }
})

-- // Configurações //
local AimbotEnabled = false
local EspEnabled = false
local FovRadius = 150 
local Smoothness = 0.35 

local Camera = workspace.CurrentCamera
local LocalPlayer = game.Players.LocalPlayer

-- // Linha do FOV Azul (TRAVADA NO CENTRO) //
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 6 
FOVCircle.Color = Color3.fromRGB(0, 160, 255)
FOVCircle.Transparency = 1
FOVCircle.Filled = false
FOVCircle.Visible = false
FOVCircle.Radius = FovRadius
-- Centraliza o círculo na tela e não muda mais a posição
FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

-- // Função para detectar quem está dentro do FOV CENTRAL //
local function GetClosestPlayerInFOV()
    local closestTarget = nil
    local shortestDistance = FovRadius

    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local humanoid = player.Character:FindFirstChild("Humanoid")
            
            if humanoid and humanoid.Health > 0 then
                local head = player.Character.Head
                local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)
                
                -- Calcula a distância da cabeça do inimigo em relação ao CENTRO DA TELA
                local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
                local distance = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
                
                if distance < shortestDistance then
                    shortestDistance = distance
                    closestTarget = player.Character
                end
            end
        end
    end
    return closestTarget
end

-- // Interface Rayfield //
local CombatTab = Window:CreateTab("Combate", 4483362458)

CombatTab:CreateToggle({
   Name = "Ativar Aimbot (FOV Fixo)",
   CurrentValue = false,
   Callback = function(Value)
      AimbotEnabled = Value
      FOVCircle.Visible = Value 
   end,
})

CombatTab:CreateSlider({
   Name = "Tamanho do FOV Central",
   Range = {50, 800},
   Increment = 5,
   Suffix = "px",
   CurrentValue = 150,
   Callback = function(Value)
      FovRadius = Value
      FOVCircle.Radius = Value 
   end,
})

local VisualTab = Window:CreateTab("Visual", 4483345998)

VisualTab:CreateToggle({
   Name = "ESP Players (Azul)",
   CurrentValue = false,
   Callback = function(Value)
      EspEnabled = Value
      if not Value then
         for _, v in pairs(game.Workspace:GetDescendants()) do
            if v:IsA("Highlight") and v.Name == "PlayerEsp" then v:Destroy() end
         end
      end
   end,
})

-- // Loop Principal //
game:GetService("RunService").RenderStepped:Connect(function()
    -- Mantém o círculo sempre no centro, mesmo se redimensionar a janela
    FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    
    -- Lógica do Aimbot
    if AimbotEnabled then
        local target = GetClosestPlayerInFOV()
        if target then
            -- Gruda na cabeça (atrás da parede) se estiver no círculo central
            Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, target.Head.Position), Smoothness)
        end
    end

    -- Lógica do ESP
    if EspEnabled then
        for _, player in pairs(game.Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") then
                if player.Character.Humanoid.Health > 0 then
                    if not player.Character:FindFirstChild("PlayerEsp") then
                        local highlight = Instance.new("Highlight")
                        highlight.Name = "PlayerEsp"
                        highlight.FillColor = Color3.fromRGB(0, 160, 255)
                        highlight.Parent = player.Character
                    end
                end
            end
        end
    end
end)

Rayfield:LoadConfiguration()

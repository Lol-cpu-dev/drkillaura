-- Mude aqui para melhorar ou piorar

local Cowndown = 0.1
local distanciaAtaque = 150
local killAuraAtivo = true

-- Mude ai encima

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

function GetEquippedTool()
    local character = LocalPlayer.Character
    if not character then return nil end
    return character:FindFirstChildOfClass("Tool")
end

function FindTargets()
    local character = LocalPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return {} end

    local alvos = {}
    for _, npc in ipairs(workspace:GetDescendants()) do
        if npc:FindFirstChild("Humanoid") and npc:FindFirstChild("HumanoidRootPart") and npc.Humanoid.Health > 0 then
            if not Players:GetPlayerFromCharacter(npc) then
                local distancia = (character.HumanoidRootPart.Position - npc.HumanoidRootPart.Position).Magnitude
                if distancia <= distanciaAtaque then
                    table.insert(alvos, npc.HumanoidRootPart)
                end
            end
        end
    end
    return alvos
end

function Attack(target)
    local tool = GetEquippedTool()
    if not tool then return end

    local direction = (target.Position - LocalPlayer.Character.HumanoidRootPart.Position).Unit
    local args = {
        tool,
        os.clock(),
        Vector3.new(direction.X, direction.Y, direction.Z)
    }
    
    pcall(function()
        local remote = ReplicatedStorage:FindFirstChild("Shared", true):FindFirstChild("Network", true):FindFirstChild("RemoteEvent", true):FindFirstChild("SwingMelee", true)
        if remote then
            remote:FireServer(unpack(args))
        end
    end)
end

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 50)
frame.Position = UDim2.new(0.5, -100, 0, 10)
frame.AnchorPoint = Vector2.new(0.5, 0)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.BorderSizePixel = 0
frame.Parent = screenGui

local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0.9, 0, 0.8, 0)
toggleButton.Position = UDim2.new(0.05, 0, 0.1, 0)
toggleButton.Text = "Ativar KillAura"
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
toggleButton.Parent = frame

local connection
toggleButton.MouseButton1Click:Connect(function()
    killAuraAtivo = not killAuraAtivo
    
    if connection then
        connection:Disconnect()
        connection = nil
    end
    
    if killAuraAtivo then
        toggleButton.Text = "Desativar KillAura"
        toggleButton.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
        
        connection = RunService.Heartbeat:Connect(function()
            local targets = FindTargets()
            for _, target in ipairs(targets) do
                Attack(target)
                task.wait(Cowndown / #targets)
            end
        end)
    else
        toggleButton.Text = "Ativar KillAura"
        toggleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    end
end)

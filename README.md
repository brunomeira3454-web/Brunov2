-- LocalScript (colocar em StarterGui -> ScreenGui -> TextButton, ou adaptar)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- CONFIGURAÇÃO
local requiredToolName = "Brairont" -- só alterna enquanto o jogador estiver segurando este tool (nome à esquerda)
local aurasList = {
    "AuraVermelhaGiratória",
    "AuraAzul",
    "AuraVerde",
    "AuraRoxa",
    "AuraDourada",
    "AuraPrata",
    "AuraPreta",
    "AuraBranca",
    "AuraCiano",
    "AuraMagenta",
}
local swapInterval = 0.6    -- tempo entre trocas (segundos) -> não abuse, mantenha razoável

-- ENCONTRAR/CRIAR BOTÃO
local screenGui = script.Parent
local button
if screenGui:IsA("TextButton") then
    button = screenGui
else
    -- caso o LocalScript esteja directo no ScreenGui, tenta achar/crear um botão
    button = screenGui:FindFirstChild("AuraSwapButton")
    if not button then
        button = Instance.new("TextButton")
        button.Name = "AuraSwapButton"
        button.Size = UDim2.new(0, 200, 0, 50)
        button.Position = UDim2.new(0.5, -100, 0.9, -25)
        button.AnchorPoint = Vector2.new(0.5, 0.5)
        button.Text = "Toggle Aura Swap"
        button.Parent = screenGui
    end
end

-- ESTADO
local swapping = false
local index = 1

local function getBackpack()
    return player:FindFirstChild("Backpack")
end

local function currentEquippedToolName()
    local char = player.Character
    if not char then return nil end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if humanoid then
        local tool = char:FindFirstChildOfClass("Tool")
        if tool then return tool.Name end
    end
    return nil
end

local function tryEquip(toolName)
    local backpack = getBackpack()
    if not backpack then return false end
    local tool = backpack:FindFirstChild(toolName)
    if tool and tool:IsA("Tool") then
        -- pede ao humanoid para equipar (local)
        local char = player.Character
        if char then
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid:EquipTool(tool)
                return true
            end
        end
    end
    return false
end

-- Loop de trocas (rode localmente)
spawn(function()
    while true do
        if swapping then
            -- só troca se o jogador estiver segurando a ferramenta necessária
            if currentEquippedToolName() == requiredToolName then
                local auraName = aurasList[index]
                -- tenta equipar; se não existir, apenas pula
                tryEquip(auraName)
                index = index + 1
                if index > #aurasList then index = 1 end
            end
            wait(swapInterval)
        else
            wait(0.2)
        end
    end
end)

button.MouseButton1Click:Connect(function()
    swapping = not swapping
    if swapping then
        button.Text = "Aura Swap: ON"
    else
        button.Text = "Aura Swap: OFF"
    end
end)

-- segurança extra: interrompe se o jogo estiver no servidor (evita execução fora do cliente)
if not RunService:IsClient() then
    warn("Este script deve rodar apenas no cliente (LocalScript).")
    return
end

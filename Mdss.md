--[[
    SCRIPT OTIMIZADO PARA DELTA EXECUTOR
    IGUAL AO EXEMPLO DA FOTO:
    - ASSASSINO: VERMELHO + TEXTO "Murderer"
    - XERIFE: AZUL + TEXTO "Sheriff"
    - INOCENTES: VERDE
]]

-- Serviços Compatíveis com Delta
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- Configurações EXATAS DA FOTO
local CONFIG = {
    Assassino = {
        Cor = Color3.new(1, 0, 0),       -- Vermelho
        Texto = "Murderer",
        CorTexto = Color3.new(1, 0, 0),
        Espessura = 3,
        Transparencia = 0.1
    },
    Xerife = {
        Cor = Color3.new(0, 0, 1),       -- Azul
        Texto = "Sheriff",
        CorTexto = Color3.new(0, 0, 1),
        Espessura = 3,
        Transparencia = 0.1
    },
    Inocente = {
        Cor = Color3.new(0, 1, 0),       -- Verde
        Texto = "",
        CorTexto = Color3.new(0, 1, 0),
        Espessura = 2,
        Transparencia = 0.2
    },
    OffsetTexto = Vector3.new(0, 3.5, 0) -- Posição do texto igual na foto
}

-- Armazenamento Seguro
local ESPs = {}
local Conexoes = {}

-- Função de Limpeza
local function LimparESP(jogador)
    if ESPs[jogador.UserId] then
        for _, obj in pairs(ESPs[jogador.UserId]) do
            pcall(function() obj:Destroy() end)
        end
        ESPs[jogador.UserId] = nil
    end
    if Conexoes[jogador.UserId] then
        for _, conn in pairs(Conexoes[jogador.UserId]) do
            pcall(function() conn:Disconnect() end)
        end
        Conexoes[jogador.UserId] = nil
    end
end

-- Função: Criar ESP de Corpo (Igual na Foto)
local function CriarESP_Corpo(personagem, config)
    local root = personagem:WaitForChild("HumanoidRootPart")
    local tamanho = personagem:GetExtentsSize() + Vector3.new(0.3, 0.3, 0.3)

    -- ESP de Corpo (Linhas Verdes/Vermelhas/Azuis da Foto)
    local corpoESP = Instance.new("BoxHandleAdornment")
    corpoESP.Name = "Delta_CorpoESP"
    corpoESP.Adornee = root
    corpoESP.Size = tamanho
    corpoESP.Color3 = config.Cor
    corpoESP.Thickness = config.Espessura
    corpoESP.Transparency = config.Transparencia
    corpoESP.AlwaysOnTop = true
    corpoESP.ZIndex = 20
    corpoESP.Parent = PlayerGui

    -- Atualiza Tamanho Constantemente
    local atualizar = RunService.RenderStepped:Connect(function()
        if not root or not personagem.Parent then atualizar:Disconnect() return end
        corpoESP.Size = personagem:GetExtentsSize() + Vector3.new(0.3, 0.3, 0.3)
    end)

    return {corpoESP, atualizar}
end

-- Função: Criar Texto no Topo (Igual na Foto)
local function CriarESP_Texto(personagem, config)
    local root = personagem:WaitForChild("HumanoidRootPart")

    if config.Texto == "" then return {} end -- Sem texto para inocentes

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "Delta_TextoESP"
    billboard.Adornee = root
    billboard.Size = UDim2.new(0, 250, 0, 50)
    billboard.StudsOffset = CONFIG.OffsetTexto
    billboard.AlwaysOnTop = true
    billboard.Parent = PlayerGui

    local texto = Instance.new("TextLabel")
    texto.Size = UDim2.new(1, 0, 1, 0)
    texto.BackgroundTransparency = 1
    texto.Text = config.Texto
    texto.TextColor3 = config.CorTexto
    texto.Font = Enum.Font.Bold
    texto.TextSize = 28
    texto.TextScaled = true
    texto.Parent = billboard

    return {billboard}
end

-- Função: Detectar Papel e Configuração
local function GetConfigPapel(jogador)
    if jogador == LocalPlayer then return nil end
    local papel = jogador:GetAttribute("Role") or (jogador.Character and jogador.Character:FindFirstChild("Role") and jogador.Character.Role.Value) or ""
    papel = string.upper(papel)

    if papel == "ASSASSIN" then
        return CONFIG.Assassino
    elseif papel == "SHERIFF" then
        return CONFIG.Xerife
    else
        return CONFIG.Inocente
    end
end

-- Função: Gerar ESP Completo
local function GerarESP(jogador)
    if not jogador.Character or not jogador.Character:FindFirstChild("HumanoidRootPart") then return end
    local config = GetConfigPapel(jogador)
    if not config then LimparESP(jogador) return end

    LimparESP(jogador)
    local personagem = jogador.Character

    -- Cria Elementos
    local corpo = CriarESP_Corpo(personagem, config)
    local texto = CriarESP_Texto(personagem, config)

    -- Armazena
    ESPs[jogador.UserId] = {unpack(corpo), unpack(texto)}
    Conexoes[jogador.UserId] = {corpo[2]}

    -- Limpa se Jogador Morrer ou Sair
    local morreu = personagem:WaitForChild("Humanoid").Died:Connect(function()
        LimparESP(jogador)
    end)
    local saiu = jogador.AncestryChanged:Connect(function()
        if not jogador.Parent then LimparESP(jogador) end
    end)
    table.insert(Conexoes[jogador.UserId], morreu)
    table.insert(Conexoes[jogador.UserId], saiu)
end

-- Sistema Principal
local function Iniciar()
    -- Jogadores Existentes
    for _, jogador in pairs(Players:GetPlayers()) do
        GerarESP(jogador)
    end

    -- Novos Jogadores
    Players.PlayerAdded:Connect(function(jogador)
        jogador.CharacterAdded:Connect(function()
            wait(1) -- Tempo para Carregamento
            GerarESP(jogador)
        end)
    end)

    -- Atualização Contínua
    Conexoes.Global = RunService.Heartbeat:Connect(function()
        for _, jogador in pairs(Players:GetPlayers()) do
            GerarESP(jogador)
        end
    end)
end

-- Inicia no Delta Executor
pcall(function()
    wait(2)
    Iniciar()
    print("[DELTA ESP] Ativado! Igual ao exemplo da foto.")
end)

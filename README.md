-- Configurações iniciais da chuva
local rainCount = 35 -- Quantidade de gotas de chuva
local rainDropSpeed = 85 -- Velocidade das gotas de chuva
local rainDropWidth = 0.2 -- Largura da gota
local rainDropHeight = 6 -- Altura da gota
local rainDropDepth = 0.1 -- Profundidade da gota
local rainHeight = 35 -- Altura em que as gotas serão geradas acima do jogador
local areaRadius = 50 -- Raio de 50 studs ao redor do jogador onde as gotas podem cair
local transparency = 0.9 -- Transparência das gotas
local lifeTime = 1 -- Tempo em segundos para as gotas sumirem

local rainDropShape = Enum.PartType.Block -- Altere para "Ball" se quiser esferas, ou remova para retângulo

local rainEnabled = false -- Variável para controlar o estado da chuva (ativa/desativa)

-- Função para criar uma gota de chuva com formato e tamanho configuráveis
local function createRainDrop(playerPosition)
	-- Geração de posição aleatória dentro do raio
	local randomX = math.random(-areaRadius, areaRadius)
	local randomZ = math.random(-areaRadius, areaRadius)

	-- Criar a parte da gota de chuva
	local rainDrop = Instance.new("Part")
	rainDrop.Size = Vector3.new(rainDropWidth, rainDropHeight, rainDropDepth) -- Tamanho modificado
	rainDrop.Shape = rainDropShape  -- Define o formato da gota (pode ser "Ball" ou nada)
	rainDrop.Color = Color3.fromRGB(15, 119, 255)  -- Cor azul para a chuva
	rainDrop.Material = Enum.Material.SmoothPlastic
	rainDrop.Anchored = false
	rainDrop.CanCollide = false
	rainDrop.Position = Vector3.new(
		playerPosition.X + randomX, -- Posição aleatória no eixo X
		playerPosition.Y + rainHeight, -- Sempre acima do jogador
		playerPosition.Z + randomZ  -- Posição aleatória no eixo Z
	)

	-- Definir a transparência das gotas
	rainDrop.Transparency = transparency

	-- Criar o efeito suave de queda usando BodyVelocity
	local bodyVelocity = Instance.new("BodyVelocity")
	bodyVelocity.Velocity = Vector3.new(0, -rainDropSpeed, 0) -- Gotas caindo para baixo suavemente
	bodyVelocity.MaxForce = Vector3.new(10000, 10000, 10000) -- Força para garantir que a gota caia
	bodyVelocity.Parent = rainDrop

	-- Adicionar a gota ao mundo
	rainDrop.Parent = workspace

	-- Função para destruir a gota após um tempo (lifeTime)
	game:GetService("Debris"):AddItem(rainDrop, lifeTime)  -- A gota vai desaparecer após 'lifeTime' segundos
end

-- Função para seguir o jogador e gerar a chuva ao redor dele
local function startRainEffect(player)
	-- Criação de gotas de chuva continuamente enquanto o jogador está no jogo
	while rainEnabled do
		if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local playerPosition = player.Character.HumanoidRootPart.Position
			for i = 1, rainCount do
				createRainDrop(playerPosition)
			end
		end
		wait(0.1) -- Intervalo entre a criação das gotas para garantir um fluxo contínuo
	end
end

-- Função para alternar a chuva (ativar/desativar)
local function toggleRain()
	rainEnabled = not rainEnabled
	if rainEnabled then
		startRainEffect(game.Players.LocalPlayer) -- Inicia a chuva
	else
		-- Se a chuva for desativada, paramos o loop
		-- (Não há uma forma fácil de interromper imediatamente, mas podemos simplesmente não chamar startRainEffect novamente)
	end
end

-- Função para monitorar a morte do jogador e parar a chuva
local function onPlayerDeath()
	rainEnabled = false -- Desativa a chuva quando o jogador morrer
end

-- Monitorar a morte do jogador
local player = game.Players.LocalPlayer
player.CharacterAdded:Connect(function(character)
	-- Monitorar a morte do personagem
	character:WaitForChild("Humanoid").Died:Connect(function()
		onPlayerDeath() -- Chama a função para desativar a chuva quando o jogador morrer
	end)
end)

-- Criando a interface gráfica (GUI)
local player = game.Players.LocalPlayer
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Frame principal para a interface (ajustado para dispositivos móveis)
local mainFrame = Instance.new("Frame")
mainFrame.Parent = screenGui
mainFrame.Size = UDim2.new(0, 400, 0, 200)  -- Novo tamanho para acomodar duas colunas
mainFrame.Position = UDim2.new(0.5, -200, 0.5, -100)  -- Centralizado na tela
mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
mainFrame.BackgroundTransparency = 0.5
mainFrame.Visible = false -- Começa minimizado

-- Adicionando o layout para a primeira coluna (lado esquerdo)
local leftFrame = Instance.new("Frame")
leftFrame.Parent = mainFrame
leftFrame.Size = UDim2.new(0.5, 0, 1, 0)  -- Metade do tamanho do mainFrame (primeira coluna)
leftFrame.Position = UDim2.new(0, 0, 0, 0)
leftFrame.BackgroundTransparency = 1 -- Tornando transparente para não interferir no layout

-- Adicionando o layout para a segunda coluna (lado direito)
local rightFrame = Instance.new("Frame")
rightFrame.Parent = mainFrame
rightFrame.Size = UDim2.new(0.5, 0, 1, 0)  -- Metade do tamanho do mainFrame (segunda coluna)
rightFrame.Position = UDim2.new(0.5, 0, 0, 0)
rightFrame.BackgroundTransparency = 1 -- Tornando transparente para não interferir no layout

-- Layout para a primeira coluna (configurações da esquerda)
local uiListLayoutLeft = Instance.new("UIListLayout")
uiListLayoutLeft.Parent = leftFrame
uiListLayoutLeft.FillDirection = Enum.FillDirection.Vertical
uiListLayoutLeft.SortOrder = Enum.SortOrder.LayoutOrder
uiListLayoutLeft.Padding = UDim.new(0, 8) -- Espaçamento entre os itens

-- Layout para a segunda coluna (configurações da direita)
local uiListLayoutRight = Instance.new("UIListLayout")
uiListLayoutRight.Parent = rightFrame
uiListLayoutRight.FillDirection = Enum.FillDirection.Vertical
uiListLayoutRight.SortOrder = Enum.SortOrder.LayoutOrder
uiListLayoutRight.Padding = UDim.new(0, 8) -- Espaçamento entre os itens

-- Função para criar uma label e um campo de texto (entrada) para cada configuração
local function createConfigOption(label, defaultValue, callback, parentFrame)
	-- Criar a label para a configuração
	local labelFrame = Instance.new("Frame")
	labelFrame.Size = UDim2.new(1, 0, 0, 40)
	labelFrame.Parent = parentFrame

	-- Label com o nome da configuração
	local labelText = Instance.new("TextLabel")
	labelText.Size = UDim2.new(0.4, 0, 1, 0)  -- Largura do nome da configuração
	labelText.Text = label
	labelText.BackgroundTransparency = 1
	labelText.TextColor3 = Color3.fromRGB(255, 255, 255)
	labelText.TextXAlignment = Enum.TextXAlignment.Left
	labelText.Parent = labelFrame

	-- Campo de entrada para a configuração
	local inputBox = Instance.new("TextBox")
	inputBox.Parent = labelFrame
	inputBox.Size = UDim2.new(0.6, 0, 1, 0)  -- Largura do campo de entrada
	inputBox.Position = UDim2.new(0.4, 0, 0, 0)
	inputBox.Text = tostring(defaultValue)
	inputBox.TextColor3 = Color3.fromRGB(255, 255, 255)
	inputBox.BackgroundColor3 = Color3.fromRGB(50, 50, 50)

	-- Quando o valor do campo de texto for alterado, chama a função callback
	inputBox.FocusLost:Connect(function(enterPressed)
		if enterPressed then
			local value = tonumber(inputBox.Text)
			if value then
				callback(value)
			end
		end
	end)
end

-- Criando as configurações para a primeira coluna
createConfigOption("Quantidade", rainCount, function(value) rainCount = value end, leftFrame)
createConfigOption("Velocidade da Chuva", rainDropSpeed, function(value) rainDropSpeed = value end, leftFrame)
createConfigOption("Largura", rainDropWidth, function(value) rainDropWidth = value end, leftFrame)
createConfigOption("Altura das Gotas", rainDropHeight, function(value) rainDropHeight = value end, leftFrame)

-- Criando as configurações para a segunda coluna
createConfigOption("Raio da Chuva", areaRadius, function(value) areaRadius = value end, rightFrame)
createConfigOption("Transparência", transparency, function(value) transparency = value end, rightFrame)
createConfigOption("Tempo de Vida", lifeTime, function(value) lifeTime = value end, rightFrame)
createConfigOption("Profundidade", rainDropDepth, function(value) rainDropDepth = value end, rightFrame)
createConfigOption("Altura das Gotas em Relação ao Jogador", rainHeight, function(value) rainHeight = value end, rightFrame)

-- Botão para abrir/fechar a interface (ajustado para dispositivos móveis)
local toggleButton = Instance.new("TextButton")
toggleButton.Parent = screenGui
toggleButton.Size = UDim2.new(0, 80, 0, 40) -- Botão menor
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.Text = "Config"
toggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)

-- Função para mostrar/ocultar a interface
toggleButton.MouseButton1Click:Connect(function()
	mainFrame.Visible = not mainFrame.Visible
end)

-- Função para criar a opção de ativar/desativar a chuva
local function toggleRain()
	rainEnabled = not rainEnabled
	if rainEnabled then
		startRainEffect(player)
	else
		-- Aqui paramos a chuva se for necessário.
		-- Isso pode ser feito interrompendo o loop ou apenas não chamando mais a função startRainEffect.
	end
end

-- Criando o botão de ativar/desativar a chuva
local rainButton = Instance.new("TextButton")
rainButton.Parent = mainFrame
rainButton.Size = UDim2.new(1, 0, 0, 40) -- Botão menor
rainButton.Text = "Ativar Chuva"
rainButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

rainButton.MouseButton1Click:Connect(function()
	toggleRain()
	if rainEnabled then
		rainButton.Text = "Desativar Chuva"
	else
		rainButton.Text = "Ativar Chuva"
	end
end)

-- Função para permitir que a interface seja movida
local dragging = false
local dragInput
local dragStart
local startPos

mainFrame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = mainFrame.Position
	end
end)

mainFrame.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		local delta = input.Position - dragStart
		mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)

mainFrame.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)

-- Inicia o efeito de chuva para o jogador
startRainEffect(player)

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

-- Criando a interface gráfica (GUI)
local player = game.Players.LocalPlayer
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Frame principal para a interface
local mainFrame = Instance.new("Frame")
mainFrame.Parent = screenGui
mainFrame.Size = UDim2.new(0, 300, 0, 600)  -- Tamanho maior para acomodar todas as configurações
mainFrame.Position = UDim2.new(0, 10, 0, 70)  -- Ajustando para começar abaixo do botão
mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
mainFrame.BackgroundTransparency = 0.5
mainFrame.Visible = false -- Começa minimizado

-- Adicionando um layout para que as opções fiquem organizadas uma embaixo da outra
local uiListLayout = Instance.new("UIListLayout")
uiListLayout.Parent = mainFrame
uiListLayout.FillDirection = Enum.FillDirection.Vertical -- Organiza as opções em uma coluna
uiListLayout.SortOrder = Enum.SortOrder.LayoutOrder
uiListLayout.Padding = UDim.new(0, 10) -- Espaçamento entre as opções

-- Botão para abrir/fechar a interface
local toggleButton = Instance.new("TextButton")
toggleButton.Parent = screenGui
toggleButton.Size = UDim2.new(0, 100, 0, 50)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.Text = "Configurações"
toggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)

-- Função para criar uma label e um campo de texto (entrada) para cada configuração
local function createConfigOption(label, defaultValue, callback)
	-- Criar a label para a configuração
	local labelFrame = Instance.new("Frame")
	labelFrame.Size = UDim2.new(1, 0, 0, 40)
	labelFrame.Parent = mainFrame

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

-- Função para criar a opção de ativar/desativar a chuva
local function createRainToggleButton()
	-- Criar o botão de ativar/desativar a chuva
	local rainButton = Instance.new("TextButton")
	rainButton.Parent = mainFrame
	rainButton.Size = UDim2.new(1, 0, 0, 50)
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
end

-- Adicionando as opções configuráveis
createConfigOption("Quantidade de Gotas", rainCount, function(value)
	rainCount = value
end)

createConfigOption("Velocidade da Chuva", rainDropSpeed, function(value)
	rainDropSpeed = value
end)

createConfigOption("Largura das Gotas", rainDropWidth, function(value)
	rainDropWidth = value
end)

createConfigOption("Altura das Gotas", rainDropHeight, function(value)
	rainDropHeight = value
end)

createConfigOption("Raio da Chuva", areaRadius, function(value)
	areaRadius = value
end)

createConfigOption("Transparência das Gotas", transparency, function(value)
	transparency = value
end)

createConfigOption("Tempo de Vida das Gotas", lifeTime, function(value)
	lifeTime = value
end)

-- Criando o botão para alternar a interface
toggleButton.MouseButton1Click:Connect(function()
	mainFrame.Visible = not mainFrame.Visible -- Alterna entre visível e invisível
end)

-- Criando o botão de ativar/desativar a chuva
createRainToggleButton()

-- Inicia o efeito de chuva para o jogador
startRainEffect(player)

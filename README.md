-- Variáveis iniciais de configuração da chuva
local rainCount = 35
local rainDropSpeed = 85
local rainDropWidth = 0.2
local rainDropHeight = 6
local rainDropDepth = 0.1
local rainHeight = 35
local areaRadius = 50
local transparency = 0.9
local lifeTime = 1
local rainEnabled = false

-- Função para criar a gota de chuva
local function createRainDrop(playerPosition)
	local randomX = math.random(-areaRadius, areaRadius)
	local randomZ = math.random(-areaRadius, areaRadius)

	local rainDrop = Instance.new("Part")
	rainDrop.Size = Vector3.new(rainDropWidth, rainDropHeight, rainDropDepth)
	rainDrop.Color = Color3.fromRGB(15, 119, 255)
	rainDrop.Material = Enum.Material.SmoothPlastic
	rainDrop.Anchored = false
	rainDrop.CanCollide = false  -- Desabilita a colisão com outros objetos
	rainDrop.Position = Vector3.new(playerPosition.X + randomX, playerPosition.Y + rainHeight, playerPosition.Z + randomZ)
	rainDrop.Transparency = transparency

	-- Quando a gota colide com algo, ela desaparece
	rainDrop.Touched:Connect(function(hit)
		-- Verifica se a gota tocou algo que não seja ela mesma
		if hit and hit.Parent and hit.Parent:FindFirstChild("Humanoid") == nil then
			rainDrop:Destroy()  -- Remove a gota ao colidir
		end
	end)

	-- Adicionando a física da gota
	local bodyVelocity = Instance.new("BodyVelocity")
	bodyVelocity.Velocity = Vector3.new(0, -rainDropSpeed, 0)
	bodyVelocity.MaxForce = Vector3.new(10000, 10000, 10000)
	bodyVelocity.Parent = rainDrop

	-- Adicionando a gota ao mundo
	rainDrop.Parent = workspace
	game:GetService("Debris"):AddItem(rainDrop, lifeTime)  -- Vai desaparecer após o tempo determinado
end

-- Função para iniciar a chuva
local function startRainEffect(player)
	while rainEnabled do
		if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local playerPosition = player.Character.HumanoidRootPart.Position
			for i = 1, rainCount do
				createRainDrop(playerPosition)
			end
		end
		wait(0.1)
	end
end

-- Função para alternar o estado da chuva
local function toggleRain()
	rainEnabled = not rainEnabled
	if rainEnabled then
		startRainEffect(game.Players.LocalPlayer)
	else
		-- O código para interromper a chuva pode ser adicionado aqui, mas por enquanto paramos o loop.
	end
end

-- Criando a interface gráfica
local player = game.Players.LocalPlayer
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Frame principal da interface
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 400, 0, 300)
mainFrame.Position = UDim2.new(0.5, -200, 0.5, -190)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BackgroundTransparency = 0.5
mainFrame.Visible = false
mainFrame.Parent = screenGui

-- Layout para as opções da esquerda
local leftFrame = Instance.new("Frame")
leftFrame.Size = UDim2.new(0.5, 0, 1, 0)
leftFrame.Position = UDim2.new(0, 0, 0, 0)
leftFrame.BackgroundTransparency = 1
leftFrame.Parent = mainFrame

-- Layout para as opções da direita
local rightFrame = Instance.new("Frame")
rightFrame.Size = UDim2.new(0.5, 0, 1, 0)
rightFrame.Position = UDim2.new(0.5, 0, 0, 0)
rightFrame.BackgroundTransparency = 1
rightFrame.Parent = mainFrame

-- UIListLayouts para as duas colunas
local uiListLayoutLeft = Instance.new("UIListLayout")
uiListLayoutLeft.FillDirection = Enum.FillDirection.Vertical
uiListLayoutLeft.Padding = UDim.new(0, 10)
uiListLayoutLeft.SortOrder = Enum.SortOrder.LayoutOrder
uiListLayoutLeft.Parent = leftFrame

local uiListLayoutRight = Instance.new("UIListLayout")
uiListLayoutRight.FillDirection = Enum.FillDirection.Vertical
uiListLayoutRight.Padding = UDim.new(0, 10)
uiListLayoutRight.SortOrder = Enum.SortOrder.LayoutOrder
uiListLayoutRight.Parent = rightFrame

-- Função para criar uma opção de configuração
local function createConfigOption(labelText, defaultValue, callback, parentFrame)
	local labelFrame = Instance.new("Frame")
	labelFrame.Size = UDim2.new(1, 0, 0, 50)
	labelFrame.Parent = parentFrame

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(0.4, 0, 1, 0)
	label.Text = labelText
	label.BackgroundTransparency = 1
	label.TextColor3 = Color3.fromRGB(255, 255, 255)
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = labelFrame

	local inputBox = Instance.new("TextBox")
	inputBox.Size = UDim2.new(0.6, 0, 1, 0)
	inputBox.Position = UDim2.new(0.4, 0, 0, 0)
	inputBox.Text = tostring(defaultValue)
	inputBox.TextColor3 = Color3.fromRGB(255, 255, 255)
	inputBox.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
	inputBox.Parent = labelFrame

	inputBox.FocusLost:Connect(function(enterPressed)
		if enterPressed then
			local value = tonumber(inputBox.Text)
			if value then
				callback(value)
			end
		end
	end)
end

-- Criando as opções de configuração na interface
createConfigOption("Quantidade de Gotas", rainCount, function(value) rainCount = value end, leftFrame)
createConfigOption("Velocidade da Chuva", rainDropSpeed, function(value) rainDropSpeed = value end, leftFrame)
createConfigOption("Largura das Gotas", rainDropWidth, function(value) rainDropWidth = value end, leftFrame)
createConfigOption("Altura das Gotas", rainDropHeight, function(value) rainDropHeight = value end, leftFrame)
createConfigOption("Raio da Chuva", areaRadius, function(value) areaRadius = value end, leftFrame)

createConfigOption("Transparência", transparency, function(value) transparency = value end, rightFrame)
createConfigOption("Tempo de Vida", lifeTime, function(value) lifeTime = value end, rightFrame)
createConfigOption("Profundidade das Gotas", rainDropDepth, function(value) rainDropDepth = value end, rightFrame)
createConfigOption("Altura em Relação ao Jogador", rainHeight, function(value) rainHeight = value end, rightFrame)

-- Botão para abrir e fechar a interface gráfica
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 100, 0, 40)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.Text = "chuva config"
toggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
toggleButton.Parent = screenGui

toggleButton.MouseButton1Click:Connect(function()
	mainFrame.Visible = not mainFrame.Visible
end)

-- Botão de ativar/desativar a chuva (na coluna da direita, embaixo)
local rainButton = Instance.new("TextButton")
rainButton.Size = UDim2.new(1, 0, 0, 40)  -- Fazendo o botão ocupar toda a largura da coluna
rainButton.Position = UDim2.new(0, 0, 1, -50)  -- Colocando o botão na parte inferior
rainButton.Text = "Ativar/Desativar Chuva"
rainButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
rainButton.Parent = rightFrame  -- Colocando o botão dentro da coluna da direita

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

-- Iniciar o efeito de chuva
startRainEffect(player)


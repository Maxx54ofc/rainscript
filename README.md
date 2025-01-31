-- Configurações iniciais da chuva
local rainCount = 35
local rainDropSpeed = 85
local rainDropWidth = 0.2
local rainDropHeight = 6
local rainDropDepth = 0.1
local rainHeight = 35
local areaRadius = 50
local transparency = 0.9
local lifeTime = 1
local rainDropShape = Enum.PartType.Block

local rainEnabled = false

local function createRainDrop(playerPosition)
	local randomX = math.random(-areaRadius, areaRadius)
	local randomZ = math.random(-areaRadius, areaRadius)

	local rainDrop = Instance.new("Part")
	rainDrop.Size = Vector3.new(rainDropWidth, rainDropHeight, rainDropDepth)
	rainDrop.Shape = rainDropShape
	rainDrop.Color = Color3.fromRGB(15, 119, 255)
	rainDrop.Material = Enum.Material.SmoothPlastic
	rainDrop.Anchored = false
	rainDrop.CanCollide = false
	rainDrop.Position = Vector3.new(
		playerPosition.X + randomX,
		playerPosition.Y + rainHeight,
		playerPosition.Z + randomZ
	)

	rainDrop.Transparency = transparency

	local bodyVelocity = Instance.new("BodyVelocity")
	bodyVelocity.Velocity = Vector3.new(0, -rainDropSpeed, 0)
	bodyVelocity.MaxForce = Vector3.new(10000, 10000, 10000)
	bodyVelocity.Parent = rainDrop

	rainDrop.Parent = workspace
	game:GetService("Debris"):AddItem(rainDrop, lifeTime)
end

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

local function toggleRain()
	rainEnabled = not rainEnabled
	if rainEnabled then
		startRainEffect(game.Players.LocalPlayer)
	else
		-- Não precisamos fazer nada aqui, pois o while irá parar quando rainEnabled for false
	end
end

local function onPlayerDeath()
	rainEnabled = false
end

local player = game.Players.LocalPlayer
player.CharacterAdded:Connect(function(character)
	character:WaitForChild("Humanoid").Died:Connect(function()
		onPlayerDeath()
	end)
end)

-- Criando a interface gráfica
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Parent = screenGui
mainFrame.Size = UDim2.new(0, 400, 0, 200)
mainFrame.Position = UDim2.new(0.5, -200, 0.5, -100)
mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
mainFrame.BackgroundTransparency = 0.5
mainFrame.Visible = true

-- Coluna esquerda para opções de configuração
local leftFrame = Instance.new("Frame")
leftFrame.Parent = mainFrame
leftFrame.Size = UDim2.new(0.5, 0, 1, 0)
leftFrame.Position = UDim2.new(0, 0, 0, 0)
leftFrame.BackgroundTransparency = 1

-- Coluna direita para controles do botão
local rightFrame = Instance.new("Frame")
rightFrame.Parent = mainFrame
rightFrame.Size = UDim2.new(0.5, 0, 1, 0)
rightFrame.Position = UDim2.new(0.5, 0, 0, 0)
rightFrame.BackgroundTransparency = 1

-- Layouts das colunas
local uiListLayoutLeft = Instance.new("UIListLayout")
uiListLayoutLeft.Parent = leftFrame
uiListLayoutLeft.FillDirection = Enum.FillDirection.Vertical
uiListLayoutLeft.SortOrder = Enum.SortOrder.LayoutOrder
uiListLayoutLeft.Padding = UDim.new(0, 8)

local uiListLayoutRight = Instance.new("UIListLayout")
uiListLayoutRight.Parent = rightFrame
uiListLayoutRight.FillDirection = Enum.FillDirection.Vertical
uiListLayoutRight.SortOrder = Enum.SortOrder.LayoutOrder
uiListLayoutRight.Padding = UDim.new(0, 8)

-- Função para criar uma opção de configuração
local function createConfigOption(label, defaultValue, callback, parentFrame)
	local labelFrame = Instance.new("Frame")
	labelFrame.Size = UDim2.new(1, 0, 0, 40)
	labelFrame.Parent = parentFrame

	local labelText = Instance.new("TextLabel")
	labelText.Size = UDim2.new(0.4, 0, 1, 0)
	labelText.Text = label
	labelText.BackgroundTransparency = 1
	labelText.TextColor3 = Color3.fromRGB(255, 255, 255)
	labelText.TextXAlignment = Enum.TextXAlignment.Left
	labelText.Parent = labelFrame

	local inputBox = Instance.new("TextBox")
	inputBox.Parent = labelFrame
	inputBox.Size = UDim2.new(0.6, 0, 1, 0)
	inputBox.Position = UDim2.new(0.4, 0, 0, 0)
	inputBox.Text = tostring(defaultValue)
	inputBox.TextChanged:Connect(function()
		local newValue = tonumber(inputBox.Text)
		if newValue then
			callback(newValue)
		end
	end)
end

-- Criando as configurações
createConfigOption("Rain Count", rainCount, function(value)
	rainCount = value
end, leftFrame)

createConfigOption("Rain Drop Speed", rainDropSpeed, function(value)
	rainDropSpeed = value
end, leftFrame)

createConfigOption("Rain Drop Width", rainDropWidth, function(value)
	rainDropWidth = value
end, leftFrame)

createConfigOption("Rain Drop Height", rainDropHeight, function(value)
	rainDropHeight = value
end, leftFrame)

createConfigOption("Rain Transparency", transparency, function(value)
	transparency = value
end, leftFrame)

-- Botão para ativar/desativar a chuva
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(1, 0, 0, 40)
toggleButton.Position = UDim2.new(0, 0, 0, 0)
toggleButton.Text = "Toggle Rain"
toggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.Parent = rightFrame
toggleButton.MouseButton1Click:Connect(toggleRain)

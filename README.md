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

-- Criação das opções restantes:
-- Criando as configurações para a primeira coluna (lado esquerdo)
createConfigOption("Quantidade", rainCount, function(value) rainCount = value end, leftFrame)
createConfigOption("Velocidade da Chuva", rainDropSpeed, function(value) rainDropSpeed = value end

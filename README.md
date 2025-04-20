local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer

-- Configurações
local speed = 100
local flyTarget1 = Vector3.new(64, 43, -3230)
local flyTarget2 = Vector3.new(58, -915, -3581)

-- Função para ativar noclip (desativar colisão)
local function activateNoclip(character)
	for _, part in ipairs(character:GetDescendants()) do
		if part:IsA("BasePart") then
			part.CanCollide = false
		end
	end

	-- Garante que continue sem colisão o tempo todo
	RunService.Stepped:Connect(function()
		for _, part in ipairs(character:GetDescendants()) do
			if part:IsA("BasePart") then
				part.CanCollide = false
			end
		end
	end)
end

-- Função principal do voo
local function startFlight(character)
	local hrp = character:WaitForChild("HumanoidRootPart")

	-- Ativa noclip
	activateNoclip(character)

	-- Teleporta para ponto inicial
	hrp.CFrame = CFrame.new(54, 43, -189)

	-- Ativa o fly
	local bodyGyro = Instance.new("BodyGyro")
	bodyGyro.P = 9e4
	bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
	bodyGyro.CFrame = hrp.CFrame
	bodyGyro.Parent = hrp

	local bodyVelocity = Instance.new("BodyVelocity")
	bodyVelocity.Velocity = Vector3.zero
	bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
	bodyVelocity.Parent = hrp

	-- Voo entre dois destinos
	local flying = true
	local currentTarget = flyTarget1

	local flightConnection
	flightConnection = RunService.RenderStepped:Connect(function()
		if not flying then return end

		if not character or not character:FindFirstChild("HumanoidRootPart") then return end

		local direction = (currentTarget - hrp.Position).Unit
		local distance = (currentTarget - hrp.Position).Magnitude

		if distance < 2 then
			if currentTarget == flyTarget1 then
				currentTarget = flyTarget2
				wait(1)
			else
				-- Descomente abaixo se quiser que pare ao chegar no segundo destino:
				-- flying = false
				-- bodyVelocity:Destroy()
				-- bodyGyro:Destroy()
				-- flightConnection:Disconnect()
			end
		else
			bodyGyro.CFrame = CFrame.new(hrp.Position, currentTarget)
			bodyVelocity.Velocity = direction * speed
		end
	end)
end

-- Ativa o sistema sempre que o player spawnar
local function onCharacterAdded(character)
	task.wait(1) -- Espera o personagem carregar completamente
	startFlight(character)
end

-- Primeira ativação
if player.Character then
	onCharacterAdded(player.Character)
end

-- Reativar após morte
player.CharacterAdded:Connect(onCharacterAdded)

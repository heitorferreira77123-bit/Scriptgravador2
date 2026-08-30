local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

player.CharacterAdded:Connect(function(newChar)
	character = newChar
	humanoid = newChar:WaitForChild("Humanoid")
	humanoidRootPart = newChar:WaitForChild("HumanoidRootPart")
end)

-- VARIÁVEIS DE CONTROLE
local savedRecordings = {}
local selectedRecordingName = nil
local currentRecording = {}
local isRecording = false
local isPlaying = false
local recordConnection = nil
local startMarker = nil
local isMinimized = false

-- VARIÁVEL PARA GRAVAR OUTRO JOGADOR
local targetPlayer = player -- Padrão: Você mesmo

-- INTERFACE PRINCIPAL
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "TAS_EB_SystemGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 240, 0, 250)
mainFrame.Position = UDim2.new(0.12, 0, 0.18, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(28, 36, 28)
mainFrame.BorderSizePixel = 1
mainFrame.BorderColor3 = Color3.fromRGB(60, 85, 60)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.ClipsDescendants = false
mainFrame.Parent = screenGui

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 26)
titleBar.BackgroundColor3 = Color3.fromRGB(16, 24, 16)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local mainTitle = Instance.new("TextLabel")
mainTitle.Size = UDim2.new(0.65, 0, 1, 0)
mainTitle.Position = UDim2.new(0.04, 0, 0, 0)
mainTitle.Text = "🪖 SIS-GRAVADOR TÁTICO"
mainTitle.TextColor3 = Color3.fromRGB(220, 230, 180)
mainTitle.BackgroundTransparency = 1
mainTitle.Font = Enum.Font.SourceSansBold
mainTitle.TextSize = 11
mainTitle.TextXAlignment = Enum.TextXAlignment.Left
mainTitle.Parent = titleBar

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 20, 0, 20)
minimizeBtn.Position = UDim2.new(1, -45, 0, 3)
minimizeBtn.Text = "v"
minimizeBtn.BackgroundColor3 = Color3.fromRGB(45, 60, 45)
minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeBtn.Font = Enum.Font.SourceSansBold
minimizeBtn.TextSize = 11
minimizeBtn.BorderSizePixel = 0
minimizeBtn.Parent = titleBar

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 20, 0, 20)
closeBtn.Position = UDim2.new(1, -22, 0, 3)
closeBtn.Text = "✕"
closeBtn.BackgroundColor3 = Color3.fromRGB(150, 40, 40)
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.SourceSansBold
closeBtn.TextSize = 11
closeBtn.BorderSizePixel = 0
closeBtn.Parent = titleBar

local contentContainer = Instance.new("Frame")
contentContainer.Size = UDim2.new(1, 0, 1, -26)
contentContainer.Position = UDim2.new(0, 0, 0, 26)
contentContainer.BackgroundTransparency = 1
contentContainer.Parent = mainFrame

----------------------------------------------------
-- PAINEL DE CONTROLE DE GRAVAÇÃO
----------------------------------------------------
-- MENU DROPDOWN DE SELEÇÃO DE JOGADOR
local dropdownBtn = Instance.new("TextButton")
dropdownBtn.Size = UDim2.new(0.9, 0, 0, 22)
dropdownBtn.Position = UDim2.new(0.05, 0, 0.04, 0)
dropdownBtn.Text = "🎯 Alvo: Mim mesmo ▼"
dropdownBtn.BackgroundColor3 = Color3.fromRGB(40, 50, 65)
dropdownBtn.TextColor3 = Color3.fromRGB(200, 220, 255)
dropdownBtn.Font = Enum.Font.SourceSansBold
dropdownBtn.TextSize = 10
dropdownBtn.BorderSizePixel = 0
dropdownBtn.ZIndex = 5
dropdownBtn.Parent = contentContainer

local playerListFrame = Instance.new("ScrollingFrame")
playerListFrame.Size = UDim2.new(0.9, 0, 0, 90)
playerListFrame.Position = UDim2.new(0.05, 0, 0.14, 0)
playerListFrame.BackgroundColor3 = Color3.fromRGB(15, 20, 28)
playerListFrame.BorderSizePixel = 1
playerListFrame.BorderColor3 = Color3.fromRGB(60, 80, 110)
playerListFrame.Visible = false
playerListFrame.ZIndex = 20
playerListFrame.ScrollBarThickness = 3
playerListFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
playerListFrame.Parent = contentContainer

local playerListLayout = Instance.new("UIListLayout")
playerListLayout.Parent = playerListFrame

local nameInput = Instance.new("TextBox")
nameInput.Size = UDim2.new(0.9, 0, 0, 20)
nameInput.Position = UDim2.new(0.05, 0, 0.16, 0)
nameInput.PlaceholderText = "Nome da Rota..."
nameInput.Text = ""
nameInput.TextColor3 = Color3.fromRGB(255, 255, 255)
nameInput.BackgroundColor3 = Color3.fromRGB(38, 48, 38)
nameInput.BorderSizePixel = 0
nameInput.Font = Enum.Font.SourceSans
nameInput.TextSize = 11
nameInput.Parent = contentContainer

local recordBtn = Instance.new("TextButton")
recordBtn.Size = UDim2.new(0.9, 0, 0, 22)
recordBtn.Position = UDim2.new(0.05, 0, 0.27, 0)
recordBtn.Text = "🔴 Iniciar Gravação"
recordBtn.BackgroundColor3 = Color3.fromRGB(150, 40, 40)
recordBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
recordBtn.Font = Enum.Font.SourceSansBold
recordBtn.TextSize = 11
recordBtn.BorderSizePixel = 0
recordBtn.Parent = contentContainer

local selectLabel = Instance.new("TextLabel")
selectLabel.Size = UDim2.new(0.9, 0, 0, 12)
selectLabel.Position = UDim2.new(0.05, 0, 0.39, 0)
selectLabel.Text = "Rotas Salvas:"
selectLabel.TextColor3 = Color3.fromRGB(180, 200, 180)
selectLabel.BackgroundTransparency = 1
selectLabel.Font = Enum.Font.SourceSans
selectLabel.TextSize = 10
selectLabel.TextXAlignment = Enum.TextXAlignment.Left
selectLabel.Parent = contentContainer

local scrollList = Instance.new("ScrollingFrame")
scrollList.Size = UDim2.new(0.9, 0, 0, 85)
scrollList.Position = UDim2.new(0.05, 0, 0.45, 0)
scrollList.BackgroundColor3 = Color3.fromRGB(18, 25, 18)
scrollList.BorderSizePixel = 0
scrollList.CanvasSize = UDim2.new(0, 0, 0, 0)
scrollList.ScrollBarThickness = 3
scrollList.Parent = contentContainer

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Parent = scrollList

local playBtn = Instance.new("TextButton")
playBtn.Size = UDim2.new(0.9, 0, 0, 24)
playBtn.Position = UDim2.new(0.05, 0, 0.86, 0)
playBtn.Text = "▶ Executar Rota em Mim"
playBtn.BackgroundColor3 = Color3.fromRGB(40, 110, 40)
playBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
playBtn.Font = Enum.Font.SourceSansBold
playBtn.TextSize = 12
playBtn.BorderSizePixel = 0
playBtn.Parent = contentContainer

----------------------------------------------------
-- DROPDOWN DE SELEÇÃO DE JOGADORES
----------------------------------------------------
local function updatePlayerDropdown()
	for _, child in ipairs(playerListFrame:GetChildren()) do
		if child:IsA("TextButton") then child:Destroy() end
	end
	
	local allPlayers = Players:GetPlayers()
	playerListFrame.CanvasSize = UDim2.new(0, 0, 0, #allPlayers * 22)
	
	for _, p in ipairs(allPlayers) do
		local btn = Instance.new("TextButton")
		btn.Size = UDim2.new(1, 0, 0, 22)
		btn.BackgroundColor3 = (p == targetPlayer) and Color3.fromRGB(45, 75, 105) or Color3.fromRGB(25, 32, 42)
		btn.TextColor3 = Color3.fromRGB(255, 255, 255)
		btn.Font = Enum.Font.SourceSans
		btn.TextSize = 10
		btn.TextXAlignment = Enum.TextXAlignment.Left
		btn.BorderSizePixel = 0
		btn.ZIndex = 21
		
		local pName = (p == player) and (p.DisplayName .. " (Você)") or p.DisplayName
		btn.Text = "   👤 " .. pName
		btn.Parent = playerListFrame
		
		btn.MouseButton1Click:Connect(function()
			targetPlayer = p
			dropdownBtn.Text = "🎯 Alvo: " .. (p == player and "Mim mesmo" or p.DisplayName) .. " ▼"
			playerListFrame.Visible = false
		end)
	end
end

dropdownBtn.MouseButton1Click:Connect(function()
	playerListFrame.Visible = not playerListFrame.Visible
	if playerListFrame.Visible then
		updatePlayerDropdown()
	end
end)

closeBtn.MouseButton1Click:Connect(function()
	screenGui:Destroy()
end)

minimizeBtn.MouseButton1Click:Connect(function()
	isMinimized = not isMinimized
	if isMinimized then
		mainFrame.Size = UDim2.new(0, 240, 0, 26)
		contentContainer.Visible = false
		minimizeBtn.Text = "^"
	else
		mainFrame.Size = UDim2.new(0, 240, 0, 250)
		contentContainer.Visible = true
		minimizeBtn.Text = "v"
	end
end)

----------------------------------------------------
-- LISTA DE ROTAS
----------------------------------------------------
local function updateDropdownList()
	for _, child in ipairs(scrollList:GetChildren()) do
		if child:IsA("Frame") then child:Destroy() end
	end
	
	local count = 0
	for name, _ in pairs(savedRecordings) do
		count = count + 1
		
		local rowFrame = Instance.new("Frame")
		rowFrame.Size = UDim2.new(1, 0, 0, 18)
		rowFrame.BackgroundTransparency = 1
		rowFrame.Parent = scrollList
		
		local itemBtn = Instance.new("TextButton")
		itemBtn.Size = UDim2.new(0.8, 0, 1, 0)
		itemBtn.Text = " " .. name
		itemBtn.BackgroundColor3 = (selectedRecordingName == name) and Color3.fromRGB(60, 85, 60) or Color3.fromRGB(32, 42, 32)
		itemBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
		itemBtn.Font = Enum.Font.SourceSans
		itemBtn.TextSize = 11
		itemBtn.TextXAlignment = Enum.TextXAlignment.Left
		itemBtn.BorderSizePixel = 0
		itemBtn.Parent = rowFrame
		
		local deleteBtn = Instance.new("TextButton")
		deleteBtn.Size = UDim2.new(0.2, 0, 1, 0)
		deleteBtn.Position = UDim2.new(0.8, 0, 0, 0)
		deleteBtn.Text = "🗑️"
		deleteBtn.BackgroundColor3 = Color3.fromRGB(120, 30, 30)
		deleteBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
		deleteBtn.Font = Enum.Font.SourceSans
		deleteBtn.TextSize = 10
		deleteBtn.BorderSizePixel = 0
		deleteBtn.Parent = rowFrame
		
		itemBtn.MouseButton1Click:Connect(function()
			selectedRecordingName = name
			updateDropdownList()
			if savedRecordings[name] then
				local cf = savedRecordings[name].startPos
				createOrMoveMarker(CFrame.new(cf[1], cf[2], cf[3], cf[4], cf[5], cf[6], cf[7], cf[8], cf[9], cf[10], cf[11], cf[12]))
			end
		end)
		
		deleteBtn.MouseButton1Click:Connect(function()
			savedRecordings[name] = nil
			if selectedRecordingName == name then
				selectedRecordingName = nil
				if startMarker then startMarker:Destroy() end
			end
			updateDropdownList()
		end)
	end
	scrollList.CanvasSize = UDim2.new(0, 0, 0, count * 18)
end

----------------------------------------------------
-- LÓGICA DE GRAVAÇÃO E REPRODUÇÃO
----------------------------------------------------
function createOrMoveMarker(cframe)
	if not startMarker or not startMarker.Parent then
		startMarker = Instance.new("Part")
		startMarker.Name = "PontoInicioEB"
		startMarker.Size = Vector3.new(2.5, 0.2, 2.5)
		startMarker.Anchored = true
		startMarker.CanCollide = false
		startMarker.Material = Enum.Material.Neon
		startMarker.Color = Color3.fromRGB(50, 255, 50)
		startMarker.Transparency = 0.3
		startMarker.Parent = workspace
	end
	startMarker.CFrame = cframe - Vector3.new(0, 2.8, 0)
end

recordBtn.MouseButton1Click:Connect(function()
	if isPlaying then return end
	playerListFrame.Visible = false
	
	local recName = nameInput.Text
	if not isRecording and (recName == "" or recName:match("^%s*$")) then
		recordBtn.Text = "Digite o Nome da Rota!"
		task.wait(1)
		recordBtn.Text = "🔴 Iniciar Gravação"
		return
	end

	if not isRecording then
		local targetChar = targetPlayer.Character
		if not targetChar or not targetChar:FindFirstChild("HumanoidRootPart") then
			recordBtn.Text = "Alvo Sem Personagem!"
			task.wait(1.2)
			recordBtn.Text = "🔴 Iniciar Gravação"
			return
		end
		
		local targetRoot = targetChar:FindFirstChild("HumanoidRootPart")
		local targetHum = targetChar:FindFirstChildOfClass("Humanoid")

		isRecording = true
		currentRecording = {}
		recordBtn.Text = "⏹ Parar & Salvar"
		playBtn.Visible = false
		
		createOrMoveMarker(targetRoot.CFrame)
		
		recordConnection = RunService.RenderStepped:Connect(function()
			if targetChar and targetRoot then
				local cf = targetRoot.CFrame
				local v = targetRoot.AssemblyLinearVelocity
				local rv = targetRoot.AssemblyAngularVelocity
				local state = targetHum and targetHum:GetState().Value or 0
				
				table.insert(currentRecording, {
					cf = {cf:GetComponents()},
					v = {v.X, v.Y, v.Z},
					rv = {rv.X, rv.Y, rv.Z},
					st = state
				})
			end
		end)
	else
		isRecording = false
		recordBtn.Text = "🔴 Iniciar Gravação"
		playBtn.Visible = true
		
		if recordConnection then
			recordConnection:Disconnect()
			recordConnection = nil
		end
		
		if #currentRecording > 0 then
			savedRecordings[recName] = {
				startPos = currentRecording[1].cf,
				frames = currentRecording
			}
			
			selectedRecordingName = recName
			nameInput.Text = ""
			updateDropdownList()
		end
	end
end)

playBtn.MouseButton1Click:Connect(function()
	if isRecording or isPlaying then return end
	playerListFrame.Visible = false
	
	if not selectedRecordingName or not savedRecordings[selectedRecordingName] then
		playBtn.Text = "Selecione uma Rota!"
		task.wait(1.2)
		playBtn.Text = "▶ Executar Rota em Mim"
		return
	end
	
	local recData = savedRecordings[selectedRecordingName]
	local startCfTab = recData.startPos
	local startCFrame = CFrame.new(startCfTab[1], startCfTab[2], startCfTab[3], startCfTab[4], startCfTab[5], startCfTab[6], startCfTab[7], startCfTab[8], startCfTab[9], startCfTab[10], startCfTab[11], startCfTab[12])
	
	local distance = (humanoidRootPart.Position - startCFrame.Position).Magnitude
	if distance > 3.5 then
		createOrMoveMarker(startCFrame)
		playBtn.Text = "Vá até o Ponto Verde!"
		task.wait(1.5)
		playBtn.Text = "▶ Executar Rota em Mim"
		return
	end

	isPlaying = true
	playBtn.Text = "Executando..."
	
	humanoidRootPart.CFrame = startCFrame
	task.wait(0.05)

	for _, frameData in ipairs(recData.frames) do
		local cf = frameData.cf
		local v = frameData.v
		local rv = frameData.rv
		
		humanoidRootPart.CFrame = CFrame.new(cf[1], cf[2], cf[3], cf[4], cf[5], cf[6], cf[7], cf[8], cf[9], cf[10], cf[11], cf[12])
		humanoidRootPart.AssemblyLinearVelocity = Vector3.new(v[1], v[2], v[3])
		humanoidRootPart.AssemblyAngularVelocity = Vector3.new(rv[1], rv[2], rv[3])
		
		RunService.RenderStepped:Wait()
	end
	
	isPlaying = false
	playBtn.Text = "▶ Executar Rota em Mim"
end)

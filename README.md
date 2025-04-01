local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local gui = Instance.new("ScreenGui")
gui.Parent = player:WaitForChild("PlayerGui")

local menuFrame = Instance.new("Frame")
menuFrame.Size = UDim2.new(0, 200, 0, 300)
menuFrame.Position = UDim2.new(0, 50, 0, 50)
menuFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
menuFrame.Visible = true
menuFrame.BorderSizePixel = 0
menuFrame.Parent = gui

local uiCornerMenu = Instance.new("UICorner")
uiCornerMenu.CornerRadius = UDim.new(0, 10)
uiCornerMenu.Parent = menuFrame

local nameLabel = Instance.new("TextLabel")
nameLabel.Size = UDim2.new(1, 0, 0, 30)
nameLabel.Position = UDim2.new(0, 0, 0, 5)
nameLabel.BackgroundTransparency = 1
nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
nameLabel.TextScaled = true
nameLabel.Text = "Nenhuma parte selecionada"
nameLabel.Parent = menuFrame

local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 30, 0, 30)
toggleButton.Position = UDim2.new(0, 50, 0, 20)
toggleButton.Text = "M"
toggleButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
toggleButton.Parent = gui

toggleButton.BorderSizePixel = 0

local uiCornerButton = Instance.new("UICorner")
uiCornerButton.CornerRadius = UDim.new(0, 5)
uiCornerButton.Parent = toggleButton

local chooseButton = Instance.new("TextButton")
chooseButton.Size = UDim2.new(0, 100, 0, 40)
chooseButton.Position = UDim2.new(0, 50, 0, 100)
chooseButton.Text = "Escolher"
chooseButton.BackgroundColor3 = Color3.fromRGB(70, 130, 180)
chooseButton.Parent = menuFrame

local uiCornerChoose = Instance.new("UICorner")
uiCornerChoose.CornerRadius = UDim.new(0, 5)
uiCornerChoose.Parent = chooseButton

local followButton = Instance.new("TextButton")
followButton.Size = UDim2.new(0, 100, 0, 40)
followButton.Position = UDim2.new(0, 50, 0, 150)
followButton.Text = "Seguir: OFF"
followButton.BackgroundColor3 = Color3.fromRGB(180, 70, 70)
followButton.Parent = menuFrame

local uiCornerFollow = Instance.new("UICorner")
uiCornerFollow.CornerRadius = UDim.new(0, 5)
uiCornerFollow.Parent = followButton

local dragging, dragInput, dragStart, startPos
local selectedPart = nil
local following = false

local function dragify(frame)
	frame.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			dragStart = input.Position
			startPos = frame.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)

	frame.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement then
			dragInput = input
		end
	end)

	game:GetService("UserInputService").InputChanged:Connect(function(input)
		if input == dragInput and dragging then
			local delta = input.Position - dragStart
			frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		end
	end)
end

dragify(menuFrame)
dragify(toggleButton)

toggleButton.MouseButton1Click:Connect(function()
	menuFrame.Visible = not menuFrame.Visible
end)

local selecting = false
chooseButton.MouseButton1Click:Connect(function()
	selecting = true
	chooseButton.Text = "Selecione uma parte"
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if selecting and input.UserInputType == Enum.UserInputType.MouseButton1 then
		local mouse = player:GetMouse()
		local target = mouse.Target
		if target and target:IsA("BasePart") then
			print("Parte selecionada:", target.Name)
			selecting = false
			chooseButton.Text = "Escolher"
			nameLabel.Text = "Selecionado: " .. target.Name
			selectedPart = target
		end
	end
end)

followButton.MouseButton1Click:Connect(function()
	if selectedPart then
		following = not following
		if following then
			followButton.Text = "Seguir: ON"
			followButton.BackgroundColor3 = Color3.fromRGB(70, 180, 70)
		else
			followButton.Text = "Seguir: OFF"
			followButton.BackgroundColor3 = Color3.fromRGB(180, 70, 70)
		end
	end
end)

RunService.RenderStepped:Connect(function()
	if following and selectedPart then
		local camera = workspace.CurrentCamera
		local character = player.Character or player.CharacterAdded:Wait()
		local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

		-- Calculando a distância do personagem até o bloco
		local characterPosition = humanoidRootPart.Position
		local targetPosition = selectedPart.Position
		local distance = (targetPosition - characterPosition).Magnitude

		-- Ajustando a distância da câmera conforme a proximidade do bloco
		local cameraDistance = math.clamp(distance, -10, 2)  -- Distância mínima e máxima
		local direction = (targetPosition - characterPosition).unit  -- Direção para o bloco

		-- Calculando a nova posição da câmera com base na distância
		local newCameraPosition = characterPosition + direction * cameraDistance + Vector3.new(0, 1, 0)  -- Ajuste vertical
		camera.CFrame = CFrame.new(newCameraPosition, targetPosition)  -- A câmera sempre olha para o bloco
	elseif not following then
		workspace.CurrentCamera.CameraType = Enum.CameraType.Custom
	end
end)

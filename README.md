-- Painel com 3 colunas: Aimbot, Boosters e ESP
-- Booster: agora tem função "Invisible" (invisível) e não abre/fecha rápido ao morrer/reviver
-- O menu NUNCA fecha sozinho ao morrer/reviver: só fecha/abre via botão ou LeftControl

local UIS = game:GetService("UserInputService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

local painelName = "PainelMenuGui"
local currentTab = "Aimbot"
local humanoid

-- Valores padrões
local wsMin, wsMax = 16, 120
local jpMin, jpMax = 35, 160
local wsValue, jpValue = 32, 70

local flyActive, flyConn, flyBody = false, nil, nil
local jumpInfActive, jumpInfConn = false, nil
local invisibleActive = false

local aimbotActive, aimbotConn = false, nil
local aimbotFOV, aimbotDist = 120, 300
local FOV_MIN, FOV_MAX = 30, 400
local DIST_MIN, DIST_MAX = 50, 2000
local aimbotShowFov = true
local aimbotMouseBtn = Enum.UserInputType.MouseButton1
local aimbotTarget = nil

local killAuraActive, killAuraConn = false, nil
local killAuraRange = 20
local KA_MIN, KA_MAX = 5, 100

local espActive, espConn = false, nil

local drawings = {}
local menuFrame
local painelVisibleState = false

local function lerp(a, b, t) return a + (b - a) * t end
local function clearDrawings()
	for _, d in ipairs(drawings) do d.Visible = false d:Remove() end
	table.clear(drawings)
end
local function setWalkSpeed(val) if humanoid then humanoid.WalkSpeed = val end end
local function setJumpPower(val) if humanoid then humanoid.JumpPower = val end end

-- BOOSTERS
local function enableFly()
	if flyConn or not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
	local root = player.Character.HumanoidRootPart
	flyBody = Instance.new("BodyVelocity")
	flyBody.MaxForce = Vector3.new(9e4, 9e4, 9e4)
	flyBody.Velocity = Vector3.new()
	flyBody.Parent = root

	flyConn = RunService.RenderStepped:Connect(function()
		if not flyActive or not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
		local camera, moveVec = workspace.CurrentCamera, Vector3.new()
		if UIS:IsKeyDown(Enum.KeyCode.W) then moveVec += camera.CFrame.LookVector end
		if UIS:IsKeyDown(Enum.KeyCode.S) then moveVec -= camera.CFrame.LookVector end
		if UIS:IsKeyDown(Enum.KeyCode.A) then moveVec -= camera.CFrame.RightVector end
		if UIS:IsKeyDown(Enum.KeyCode.D) then moveVec += camera.CFrame.RightVector end
		if UIS:IsKeyDown(Enum.KeyCode.Space) then moveVec += Vector3.new(0,1,0) end
		if UIS:IsKeyDown(Enum.KeyCode.LeftShift) then moveVec -= Vector3.new(0,1,0) end
		flyBody.Velocity = (moveVec.Magnitude > 0) and (moveVec.Unit * 50) or Vector3.new()
	end)
end
local function disableFly()
	if flyConn then flyConn:Disconnect(); flyConn = nil end
	if flyBody then flyBody:Destroy(); flyBody = nil end
end

local function enableJumpInfinite()
	if jumpInfConn or not humanoid then return end
	jumpInfConn = UIS.JumpRequest:Connect(function()
		if jumpInfActive and humanoid then humanoid:ChangeState(Enum.HumanoidStateType.Jumping) end
	end)
end
local function disableJumpInfinite()
	if jumpInfConn then jumpInfConn:Disconnect(); jumpInfConn = nil end
end

local function setInvisible(active)
	invisibleActive = active
	if player.Character then
		for _, part in ipairs(player.Character:GetChildren()) do
			if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
				part.LocalTransparencyModifier = active and 1 or 0
			elseif part:IsA("Decal") then
				part.Transparency = active and 1 or 0
			elseif part:IsA("Accessory") then
				for _, accPart in ipairs(part:GetChildren()) do
					if accPart:IsA("BasePart") then
						accPart.LocalTransparencyModifier = active and 1 or 0
					end
				end
			end
		end
	end
end

-- reaplica invisível ao respawn
player.CharacterAdded:Connect(function(char)
	humanoid = char:WaitForChild("Humanoid")
	if invisibleActive then task.wait(1) setInvisible(true) end
end)

-- ====================================
-- Menu nunca fecha sozinho (apenas Ctrl)
-- ====================================
UIS.InputBegan:Connect(function(input, gpe)
	if gpe then return end
	if input.KeyCode == Enum.KeyCode.LeftControl then
		if menuFrame then menuFrame.Visible = not menuFrame.Visible end
	end
end)


	-- Painel com 3 colunas: Aimbot, Boosters e ESP
	-- Booster: agora tem função "Invisible" (invisível) e não abre/fecha rápido ao morrer/reviver
	-- O menu NUNCA fecha sozinho ao morrer/reviver: só fecha/abre via botão ou LeftControl

	local UIS = game:GetService("UserInputService")
	local Players = game:GetService("Players")
	local player = Players.LocalPlayer
	local RunService = game:GetService("RunService")

	local painelName = "PainelMenuGui"
	local currentTab = "Aimbot"
	local humanoid

	local wsMin, wsMax = 16, 120
	local jpMin, jpMax = 35, 160
	local wsValue = 32
	local jpValue = 70

	local flyActive = false
	local flyConn = nil
	local flyBody = nil
	local jumpInfActive = false
	local jumpInfConn = nil
	local invisibleActive = false

	local aimbotActive = false
	local aimbotFOV = 120
	local FOV_MIN, FOV_MAX = 30, 400
	local aimbotDist = 300
	local DIST_MIN, DIST_MAX = 50, 2000
	local aimbotShowFov = true
	local aimbotMouseBtn = Enum.UserInputType.MouseButton1
	local aimbotTarget = nil

	local killAuraActive = false
	local killAuraConn = nil
	local killAuraRange = 20
	local KA_MIN, KA_MAX = 5, 100

	local espActive = false

	local aimBtnText = {[Enum.UserInputType.MouseButton1]="Mouse 1", [Enum.UserInputType.MouseButton2]="Mouse 2"}

	local skeletonBones = {
		{"Head", "UpperTorso"}, {"UpperTorso", "LowerTorso"},
		{"UpperTorso", "LeftUpperArm"}, {"UpperTorso", "RightUpperArm"},
		{"LeftUpperArm", "LeftLowerArm"}, {"LeftLowerArm", "LeftHand"},
		{"RightUpperArm", "RightLowerArm"}, {"RightLowerArm", "RightHand"},
		{"LowerTorso", "LeftUpperLeg"}, {"LowerTorso", "RightUpperLeg"},
		{"LeftUpperLeg", "LeftLowerLeg"}, {"LeftLowerLeg", "LeftFoot"},
		{"RightUpperLeg", "RightLowerLeg"}, {"RightLowerLeg", "RightFoot"},
	}

	local drawings = {}
	local espConn, aimbotConn
	local menuFrame, skullBall
	local painelReabrindo = false
	local painelVisibleState = false

	local function lerp(a, b, t) return a + (b - a) * t end
	local function clearDrawings()
		for _, d in ipairs(drawings) do d.Visible = false d:Remove() end
		table.clear(drawings)
	end
	local function setWalkSpeed(val) if humanoid then humanoid.WalkSpeed = val end end
	local function setJumpPower(val) if humanoid then humanoid.JumpPower = val end end

	-- BOOSTERS
	local function enableFly()
		if flyConn or not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
		local root = player.Character.HumanoidRootPart
		flyBody = Instance.new("BodyVelocity")
		flyBody.MaxForce = Vector3.new(9e4, 9e4, 9e4)
		flyBody.Velocity = Vector3.new(0,0,0)
		flyBody.Parent = root
		flyConn = RunService.RenderStepped:Connect(function()
			if not flyActive or not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
			local camera = workspace.CurrentCamera
			local moveVec = Vector3.new()
			if UIS:IsKeyDown(Enum.KeyCode.W) then moveVec = moveVec + camera.CFrame.LookVector end
			if UIS:IsKeyDown(Enum.KeyCode.S) then moveVec = moveVec - camera.CFrame.LookVector end
			if UIS:IsKeyDown(Enum.KeyCode.A) then moveVec = moveVec - camera.CFrame.RightVector end
			if UIS:IsKeyDown(Enum.KeyCode.D) then moveVec = moveVec + camera.CFrame.RightVector end
			if UIS:IsKeyDown(Enum.KeyCode.Space) then moveVec = moveVec + Vector3.new(0,1,0) end
			if UIS:IsKeyDown(Enum.KeyCode.LeftShift) then moveVec = moveVec - Vector3.new(0,1,0) end
			if moveVec.Magnitude > 0 then
				flyBody.Velocity = moveVec.Unit * 50
			else
				flyBody.Velocity = Vector3.new(0,0,0)
			end
		end)
	end
	local function disableFly()
		if flyConn then flyConn:Disconnect(); flyConn = nil end
		if flyBody then flyBody:Destroy(); flyBody = nil end
	end

	local function enableJumpInfinite()
		if jumpInfConn or not humanoid then return end
		jumpInfConn = UIS.JumpRequest:Connect(function()
			if jumpInfActive and humanoid then
				humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
			end
		end)
	end
	local function disableJumpInfinite()
		if jumpInfConn then jumpInfConn:Disconnect(); jumpInfConn = nil end
	end

	local function setInvisible(active)
		invisibleActive = active
		if player.Character then
			for _, part in ipairs(player.Character:GetChildren()) do
				if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
					part.LocalTransparencyModifier = active and 1 or 0
				elseif part:IsA("Decal") then
					part.Transparency = active and 1 or 0
				elseif part:IsA("Accessory") then
					for _, accPart in ipairs(part:GetChildren()) do
						if accPart:IsA("BasePart") then
							accPart.LocalTransparencyModifier = active and 1 or 0
						end
					end
				end
			end
		end
	end

	local function restoreInvisibleOnRespawn()
		if invisibleActive then
			setInvisible(true)
		end
	end

	-- UNBAN / UNKICK / UNDETECT
	local function unban()
		for _,v in ipairs({"Banned","Ban","Kick","Kicked"}) do
			if player:FindFirstChild(v) then player[v]:Destroy() end
			if player.Character then
				for _,c in ipairs(player.Character:GetChildren()) do
					if c.Name:lower():find(v:lower()) then c:Destroy() end
				end
			end
		end
	end
	local function unkick()
		for _,v in ipairs({"Kick","Kicked"}) do
			if player:FindFirstChild(v) then player[v]:Destroy() end
			if player.Character then
				for _,c in ipairs(player.Character:GetChildren()) do
					if c.Name:lower():find(v:lower()) then c:Destroy() end
				end
			end
		end
	end
	local function undetect()
		for _,v in ipairs({"Detected","Detection"}) do
			if player:FindFirstChild(v) then player[v]:Destroy() end
			if player.Character then
				for _,c in ipairs(player.Character:GetChildren()) do
					if c.Name:lower():find(v:lower()) then c:Destroy() end
				end
			end
		end
		for _,v in ipairs(game:GetService("StarterPlayer"):GetChildren()) do
			if v.Name:lower():find("anti") or v.Name:lower():find("cheat") then v:Destroy() end
		end
	end

	-- ESP
	local function espStep()
		clearDrawings()
		local camera = workspace.CurrentCamera
		if aimbotShowFov and aimbotActive then
			local fovCircle = Drawing.new("Circle")
			fovCircle.Visible = true
			fovCircle.Position = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)
			fovCircle.Radius = aimbotFOV
			fovCircle.Color = Color3.fromRGB(255,220,50)
			fovCircle.Thickness = 2
			fovCircle.Filled = false
			fovCircle.Transparency = 0.6
			table.insert(drawings, fovCircle)
		end
		for _, other in ipairs(Players:GetPlayers()) do
			if other ~= player and other.Character and other.Character:FindFirstChild("Head") then
				local char = other.Character
				local camera = workspace.CurrentCamera
				local head = char:FindFirstChild("Head")
				if head then
					local pos = camera:WorldToViewportPoint(head.Position)
					if pos.Z > 0 then
						local nameDraw = Drawing.new("Text")
						nameDraw.Visible = true
						nameDraw.Text = other.DisplayName or other.Name
						nameDraw.Position = Vector2.new(pos.X, pos.Y - 30)
						nameDraw.Color = Color3.fromRGB(255,255,255)
						nameDraw.Size = 22
						nameDraw.Center = true
						nameDraw.Outline = true
						nameDraw.OutlineColor = Color3.fromRGB(0,0,0)
						nameDraw.Font = 2
						nameDraw.Transparency = 1
						table.insert(drawings, nameDraw)
					end
				end
				local points = {}
				for _, bone in pairs({
					"Head","UpperTorso","LowerTorso",
					"LeftUpperArm","LeftLowerArm","LeftHand",
					"RightUpperArm","RightLowerArm","RightHand",
					"LeftUpperLeg","LeftLowerLeg","LeftFoot",
					"RightUpperLeg","RightLowerLeg","RightFoot"}) do
					local part = char:FindFirstChild(bone)
					if part then
						local pos = camera:WorldToViewportPoint(part.Position)
						if pos.Z > 0 then points[bone] = Vector2.new(pos.X, pos.Y) end
					end
				end
				for _, link in ipairs(skeletonBones) do
					local a, b = link[1], link[2]
					if points[a] and points[b] then
						local line = Drawing.new("Line")
						line.From = points[a]
						line.To = points[b]
						line.Color = Color3.fromRGB(200, 100, 230)
						line.Thickness = 2.5
						line.Transparency = 1
						line.Visible = true
						table.insert(drawings, line)
					end
				end
			end
		end
	end
	local function enableESPDraw() if espConn then return end espConn = RunService.RenderStepped:Connect(espStep) end
	local function disableESPDraw() if espConn then espConn:Disconnect(); espConn = nil end clearDrawings() end

	-- AIMBOT
	local function getClosestEnemyFOV()
		local camera = workspace.CurrentCamera
		local center = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)
		local closest, minDist = nil, aimbotFOV
		for _, v in ipairs(Players:GetPlayers()) do
			if v ~= player and v.Character and v.Character:FindFirstChild("Head") then
				local pos = camera:WorldToViewportPoint(v.Character.Head.Position)
				local dist3d = (player.Character.Head.Position - v.Character.Head.Position).Magnitude
				if pos.Z > 0 then
					local dist2d = (Vector2.new(pos.X, pos.Y) - center).Magnitude
					if dist2d < minDist and dist3d <= aimbotDist then
						minDist = dist2d
						closest = v
					end
				end
			end
		end
		return closest
	end
	local function enableAimbot()
		if aimbotConn then return end
		aimbotConn = RunService.RenderStepped:Connect(function()
			if not aimbotActive then aimbotTarget = nil return end
			if UIS:IsMouseButtonPressed(aimbotMouseBtn) then
				local target = getClosestEnemyFOV()
				aimbotTarget = target
				if target and target.Character and target.Character:FindFirstChild("Head") then
					local camera = workspace.CurrentCamera
					camera.CFrame = CFrame.new(camera.CFrame.Position, target.Character.Head.Position)
				end
			else
				aimbotTarget = nil
			end
		end)
	end
	local function disableAimbot() if aimbotConn then aimbotConn:Disconnect(); aimbotConn = nil end aimbotTarget = nil end

	-- KILL AURA
	local function killAuraStep()
		if not killAuraActive or not player.Character or not humanoid then return end
		for _, v in ipairs(Players:GetPlayers()) do
			if v ~= player and v.Character and v.Character:FindFirstChild("Humanoid") and v.Character:FindFirstChild("Head") and v.Character:FindFirstChild("HumanoidRootPart") then
				local dist = (player.Character.Head.Position - v.Character.Head.Position).Magnitude
				if dist <= killAuraRange then
					local root = player.Character:FindFirstChild("HumanoidRootPart")
					local targetRoot = v.Character.HumanoidRootPart
					if root and targetRoot then
						local offset = -targetRoot.CFrame.LookVector * 3
						root.CFrame = targetRoot.CFrame + offset
					end
					local h = v.Character:FindFirstChild("Humanoid")
					if h and h.Health > 0 then
						h.Health = 0
					end
				end
			end
		end
	end
	local function enableKillAura()
		if killAuraConn then return end
		killAuraConn = RunService.RenderStepped:Connect(killAuraStep)
	end
	local function disableKillAura()
		if killAuraConn then killAuraConn:Disconnect(); killAuraConn = nil end
	end

	local function makeSliderDraggable(handle, bar, min, max, getValue, setValue, valueLabel, updatePosition)
		local dragging = false
		handle.InputBegan:Connect(function(input)
			if input.UserInputType == Enum.UserInputType.MouseButton1 then
				dragging = true
				menuFrame.Draggable = false
			end
		end)
		UIS.InputEnded:Connect(function(input)
			if dragging and input.UserInputType == Enum.UserInputType.MouseButton1 then
				dragging = false
				menuFrame.Draggable = true
			end
		end)
		UIS.InputChanged:Connect(function(input)
			if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
				local mouse = UIS:GetMouseLocation().X
				local barStart = bar.AbsolutePosition.X
				local barRange = bar.AbsoluteSize.X - handle.Size.X.Offset
				local newX = math.clamp(mouse - barStart, 0, barRange)
				handle.Position = UDim2.new(0, newX, handle.Position.Y.Scale, handle.Position.Y.Offset)
				local percent = newX/barRange
				local newValue = math.floor(lerp(min,max,percent))
				setValue(newValue)
				valueLabel.Text = tostring(getValue())
				if updatePosition then updatePosition() end
			end
		end)
	end

	-- =======================
	-- PAINEL UI E COLUNAS
	-- =======================

	function createPainel()
		if menuFrame then
			painelVisibleState = menuFrame.Visible
		end

		if player.PlayerGui:FindFirstChild(painelName) then player.PlayerGui[painelName]:Destroy() end
		local screenGui = Instance.new("ScreenGui")
		screenGui.Name = painelName
		screenGui.Parent = player:WaitForChild("PlayerGui")

		menuFrame = Instance.new("Frame")
		menuFrame.Size = UDim2.new(0, 640, 0, 410)
		menuFrame.Position = UDim2.new(0.5, -320, 0.5, -205)
		menuFrame.BackgroundColor3 = Color3.fromRGB(22, 24, 32)
		menuFrame.BackgroundTransparency = 0.08
		menuFrame.BorderSizePixel = 0
		menuFrame.Visible = painelVisibleState
		menuFrame.Active = true
		menuFrame.Draggable = true
		menuFrame.ZIndex = 2
		menuFrame.Parent = screenGui

		local menuCorner = Instance.new("UICorner")
		menuCorner.CornerRadius = UDim.new(0, 16)
		menuCorner.Parent = menuFrame

		local sidebar = Instance.new("Frame", menuFrame)
		sidebar.Size = UDim2.new(0, 84, 1, 0)
		sidebar.Position = UDim2.new(0, 0, 0, 0)
		sidebar.BackgroundColor3 = Color3.fromRGB(14, 16, 20)
		sidebar.BackgroundTransparency = 0.07
		sidebar.ZIndex = 5
		local sidebarCorner = Instance.new("UICorner")
		sidebarCorner.CornerRadius = UDim.new(0, 18)
		sidebarCorner.Parent = sidebar

		local sidebarLayout = Instance.new("UIListLayout", sidebar)
		sidebarLayout.SortOrder = Enum.SortOrder.LayoutOrder
		sidebarLayout.Padding = UDim.new(0, 18)
		sidebarLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
		sidebarLayout.VerticalAlignment = Enum.VerticalAlignment.Center

		local tabs = {
			{name="Aimbot", icon="🎯"},
			{name="Boosters", icon="⚡"},
			{name="ESP", icon="☠️"},
		}
		local colunas = {}
		for i, tab in ipairs(tabs) do
			local coluna = Instance.new("Frame", menuFrame)
			coluna.Name = tab.name
			coluna.Size = UDim2.new(0, 530, 1, -32)
			coluna.Position = UDim2.new(0, 94, 0, 16)
			coluna.BackgroundTransparency = 1
			coluna.ZIndex = 7
			coluna.Visible = (i == 1)
			colunas[tab.name] = coluna
		end
		local function showColuna(nome)
			for t, frame in pairs(colunas) do frame.Visible = (t == nome) end
		end
		showColuna(currentTab)
		for i, tab in ipairs(tabs) do
			local btn = Instance.new("TextButton", sidebar)
			btn.Size = UDim2.new(1, -16, 0, 40)
			btn.BackgroundColor3 = Color3.fromRGB(40,40,50)
			btn.Text = tab.icon.." "..tab.name
			btn.TextColor3 = Color3.fromRGB(255,255,255)
			btn.TextStrokeTransparency = 0.1
			btn.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			btn.Font = Enum.Font.GothamBlack
			btn.TextSize = 22
			btn.AutoButtonColor = true
			btn.LayoutOrder = i
			btn.ZIndex = 6
			local btnCorner = Instance.new("UICorner")
			btnCorner.CornerRadius = UDim.new(1,0)
			btnCorner.Parent = btn
			btn.MouseButton1Click:Connect(function()
				currentTab = tab.name
				showColuna(tab.name)
			end)
		end

		-- ===========================================
		-- BOOSTERS COLUMN
		-- ===========================================
		do
			local coluna = colunas["Boosters"]
			local title = Instance.new("TextLabel", coluna)
			title.Size = UDim2.new(1, 0, 0, 36)
			title.Position = UDim2.new(0, 0, 0, 0)
			title.BackgroundTransparency = 1
			title.Text = "Boosters"
			title.TextColor3 = Color3.fromRGB(255,255,255)
			title.TextStrokeTransparency = 0.1
			title.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			title.Font = Enum.Font.GothamBlack
			title.TextSize = 27

			-- Speed Slider
			local speedLabel = Instance.new("TextLabel", coluna)
			speedLabel.Size = UDim2.new(0, 50, 0, 20)
			speedLabel.Position = UDim2.new(0, 0, 0, 50)
			speedLabel.BackgroundTransparency = 1
			speedLabel.Text = "Speed"
			speedLabel.TextColor3 = Color3.fromRGB(255,255,255)
			speedLabel.TextStrokeTransparency = 0.1
			speedLabel.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			speedLabel.Font = Enum.Font.GothamBlack
			speedLabel.TextSize = 18

			local speedValue = Instance.new("TextLabel", coluna)
			speedValue.Size = UDim2.new(0, 40, 0, 20)
			speedValue.Position = UDim2.new(0, 60, 0, 50)
			speedValue.BackgroundTransparency = 1
			speedValue.Text = tostring(wsValue)
			speedValue.TextColor3 = Color3.fromRGB(255,255,255)
			speedValue.TextStrokeTransparency = 0.1
			speedValue.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			speedValue.Font = Enum.Font.GothamBlack
			speedValue.TextSize = 18

			local speedBar = Instance.new("Frame", coluna)
			speedBar.Size = UDim2.new(0, 180, 0, 9)
			speedBar.Position = UDim2.new(0, 110, 0, 58)
			speedBar.BackgroundColor3 = Color3.fromRGB(70, 100, 70)
			speedBar.BorderSizePixel = 0
			local speedSliderCorner = Instance.new("UICorner")
			speedSliderCorner.CornerRadius = UDim.new(1,0)
			speedSliderCorner.Parent = speedBar

			local speedHandle = Instance.new("Frame", speedBar)
			speedHandle.Size = UDim2.new(0, 16, 0, 18)
			local function updateSpeedHandle() speedHandle.Position = UDim2.new(0, math.floor((wsValue - wsMin)/(wsMax - wsMin) * (180-16)), -0.5, -5) end
			updateSpeedHandle()
			speedHandle.BackgroundColor3 = Color3.fromRGB(90,180,60)
			speedHandle.BorderSizePixel = 0
			speedHandle.Active = true
			speedHandle.Draggable = false
			local speedHandleCorner = Instance.new("UICorner", speedHandle)
			speedHandleCorner.CornerRadius = UDim.new(1,0)
			makeSliderDraggable(speedHandle, speedBar, wsMin, wsMax,
				function() return wsValue end,
				function(val) wsValue = val; setWalkSpeed(wsValue) end,
				speedValue, updateSpeedHandle)

			-- Jump Slider
			local jumpLabel = Instance.new("TextLabel", coluna)
			jumpLabel.Size = UDim2.new(0, 50, 0, 20)
			jumpLabel.Position = UDim2.new(0, 0, 0, 85)
			jumpLabel.BackgroundTransparency = 1
			jumpLabel.Text = "Jump"
			jumpLabel.TextColor3 = Color3.fromRGB(255,255,255)
			jumpLabel.TextStrokeTransparency = 0.1
			jumpLabel.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			jumpLabel.Font = Enum.Font.GothamBlack
			jumpLabel.TextSize = 18

			local jumpValue = Instance.new("TextLabel", coluna)
			jumpValue.Size = UDim2.new(0, 40, 0, 20)
			jumpValue.Position = UDim2.new(0, 60, 0, 85)
			jumpValue.BackgroundTransparency = 1
			jumpValue.Text = tostring(jpValue)
			jumpValue.TextColor3 = Color3.fromRGB(255,255,255)
			jumpValue.TextStrokeTransparency = 0.1
			jumpValue.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			jumpValue.Font = Enum.Font.GothamBlack
			jumpValue.TextSize = 18

			local jumpBar = Instance.new("Frame", coluna)
			jumpBar.Size = UDim2.new(0, 180, 0, 9)
			jumpBar.Position = UDim2.new(0, 110, 0, 93)
			jumpBar.BackgroundColor3 = Color3.fromRGB(70, 100, 120)
			jumpBar.BorderSizePixel = 0
			local jumpSliderCorner = Instance.new("UICorner")
			jumpSliderCorner.CornerRadius = UDim.new(1,0)
			jumpSliderCorner.Parent = jumpBar

			local jumpHandle = Instance.new("Frame", jumpBar)
			jumpHandle.Size = UDim2.new(0, 16, 0, 18)
			local function updateJumpHandle() jumpHandle.Position = UDim2.new(0, math.floor((jpValue - jpMin)/(jpMax - jpMin) * (180-16)), -0.5, -5) end
			updateJumpHandle()
			jumpHandle.BackgroundColor3 = Color3.fromRGB(90,180,200)
			jumpHandle.BorderSizePixel = 0
			jumpHandle.Active = true
			jumpHandle.Draggable = false
			local jumpHandleCorner = Instance.new("UICorner", jumpHandle)
			jumpHandleCorner.CornerRadius = UDim.new(1,0)
			makeSliderDraggable(jumpHandle, jumpBar, jpMin, jpMax,
				function() return jpValue end,
				function(val) jpValue = val; setJumpPower(jpValue) end,
				jumpValue, updateJumpHandle)

			-- Fly Toggle
			local flyBtn = Instance.new("TextButton", coluna)
			flyBtn.Size = UDim2.new(0, 120, 0, 32)
			flyBtn.Position = UDim2.new(0, 0, 0, 130)
			flyBtn.BackgroundColor3 = flyActive and Color3.fromRGB(90,200,200) or Color3.fromRGB(30,60,60)
			flyBtn.Text = flyActive and "Fly ON" or "Fly OFF"
			flyBtn.TextColor3 = Color3.fromRGB(255,255,255)
			flyBtn.TextStrokeTransparency = 0.1
			flyBtn.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			flyBtn.Font = Enum.Font.GothamBlack
			flyBtn.TextSize = 18
			flyBtn.AutoButtonColor = false
			local flyCorner = Instance.new("UICorner")
			flyCorner.CornerRadius = UDim.new(1,0)
			flyCorner.Parent = flyBtn
			flyBtn.MouseButton1Click:Connect(function()
				flyActive = not flyActive
				flyBtn.BackgroundColor3 = flyActive and Color3.fromRGB(90,200,200) or Color3.fromRGB(30,60,60)
				flyBtn.Text = flyActive and "Fly ON" or "Fly OFF"
				if flyActive then enableFly() else disableFly() end
			end)

			-- Infinite Jump Toggle
			local infJumpBtn = Instance.new("TextButton", coluna)
			infJumpBtn.Size = UDim2.new(0, 120, 0, 32)
			infJumpBtn.Position = UDim2.new(0, 130, 0, 130)
			infJumpBtn.BackgroundColor3 = jumpInfActive and Color3.fromRGB(90,200,200) or Color3.fromRGB(30,60,60)
			infJumpBtn.Text = jumpInfActive and "Inf. Jump ON" or "Inf. Jump OFF"
			infJumpBtn.TextColor3 = Color3.fromRGB(255,255,255)
			infJumpBtn.TextStrokeTransparency = 0.1
			infJumpBtn.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			infJumpBtn.Font = Enum.Font.GothamBlack
			infJumpBtn.TextSize = 18
			infJumpBtn.AutoButtonColor = false
			local infJumpCorner = Instance.new("UICorner")
			infJumpCorner.CornerRadius = UDim.new(1,0)
			infJumpCorner.Parent = infJumpBtn
			infJumpBtn.MouseButton1Click:Connect(function()
				jumpInfActive = not jumpInfActive
				infJumpBtn.BackgroundColor3 = jumpInfActive and Color3.fromRGB(90,200,200) or Color3.fromRGB(30,60,60)
				infJumpBtn.Text = jumpInfActive and "Inf. Jump ON" or "Inf. Jump OFF"
				if jumpInfActive then enableJumpInfinite() else disableJumpInfinite() end
			end)

			-- Invisible Toggle
			local invisibleBtn = Instance.new("TextButton", coluna)
			invisibleBtn.Size = UDim2.new(0, 120, 0, 32)
			invisibleBtn.Position = UDim2.new(0, 260, 0, 130)
			invisibleBtn.BackgroundColor3 = invisibleActive and Color3.fromRGB(180,180,180) or Color3.fromRGB(60,60,60)
			invisibleBtn.Text = invisibleActive and "Invisible ON" or "Invisible OFF"
			invisibleBtn.TextColor3 = Color3.fromRGB(255,255,255)
			invisibleBtn.TextStrokeTransparency = 0.1
			invisibleBtn.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			invisibleBtn.Font = Enum.Font.GothamBlack
			invisibleBtn.TextSize = 18
			invisibleBtn.AutoButtonColor = false
			local invisibleCorner = Instance.new("UICorner")
			invisibleCorner.CornerRadius = UDim.new(1,0)
			invisibleCorner.Parent = invisibleBtn
			invisibleBtn.MouseButton1Click:Connect(function()
				invisibleActive = not invisibleActive
				invisibleBtn.BackgroundColor3 = invisibleActive and Color3.fromRGB(180,180,180) or Color3.fromRGB(60,60,60)
				invisibleBtn.Text = invisibleActive and "Invisible ON" or "Invisible OFF"
				setInvisible(invisibleActive)
			end)

			-- Unban Button
			local unbanBtn = Instance.new("TextButton", coluna)
			unbanBtn.Size = UDim2.new(0, 120, 0, 32)
			unbanBtn.Position = UDim2.new(0, 0, 0, 170)
			unbanBtn.BackgroundColor3 = Color3.fromRGB(180, 70, 70)
			unbanBtn.Text = "🚫 Unban"
			unbanBtn.TextColor3 = Color3.fromRGB(255,255,255)
			unbanBtn.TextStrokeTransparency = 0.1
			unbanBtn.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			unbanBtn.Font = Enum.Font.GothamBlack
			unbanBtn.TextSize = 18
			unbanBtn.AutoButtonColor = false
			local unbanCorner = Instance.new("UICorner")
			unbanCorner.CornerRadius = UDim.new(1,0)
			unbanCorner.Parent = unbanBtn
			unbanBtn.MouseButton1Click:Connect(unban)

			-- Unkick Button
			local unkickBtn = Instance.new("TextButton", coluna)
			unkickBtn.Size = UDim2.new(0, 120, 0, 32)
			unkickBtn.Position = UDim2.new(0, 130, 0, 170)
			unkickBtn.BackgroundColor3 = Color3.fromRGB(70, 180, 70)
			unkickBtn.Text = "🦶 Unkick"
			unkickBtn.TextColor3 = Color3.fromRGB(255,255,255)
			unkickBtn.TextStrokeTransparency = 0.1
			unkickBtn.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			unkickBtn.Font = Enum.Font.GothamBlack
			unkickBtn.TextSize = 18
			unkickBtn.AutoButtonColor = false
			local unkickCorner = Instance.new("UICorner")
			unkickCorner.CornerRadius = UDim.new(1,0)
			unkickCorner.Parent = unkickBtn
			unkickBtn.MouseButton1Click:Connect(unkick)

			-- Undetect Button
			local undetectBtn = Instance.new("TextButton", coluna)
			undetectBtn.Size = UDim2.new(0, 120, 0, 32)
			undetectBtn.Position = UDim2.new(0, 260, 0, 170)
			undetectBtn.BackgroundColor3 = Color3.fromRGB(70, 70, 180)
			undetectBtn.Text = "🛡️ Undetect"
			undetectBtn.TextColor3 = Color3.fromRGB(255,255,255)
			undetectBtn.TextStrokeTransparency = 0.1
			undetectBtn.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			undetectBtn.Font = Enum.Font.GothamBlack
			undetectBtn.TextSize = 18
			undetectBtn.AutoButtonColor = false
			local undetectCorner = Instance.new("UICorner")
			undetectCorner.CornerRadius = UDim.new(1,0)
			undetectCorner.Parent = undetectBtn
			undetectBtn.MouseButton1Click:Connect(undetect)
		end

		-- ===========================================
		-- ESP COLUMN
		-- ===========================================
		do
			local coluna = colunas["ESP"]
			local title = Instance.new("TextLabel", coluna)
			title.Size = UDim2.new(1, 0, 0, 36)
			title.Position = UDim2.new(0, 0, 0, 0)
			title.BackgroundTransparency = 1
			title.Text = "ESP"
			title.TextColor3 = Color3.fromRGB(255,255,255)
			title.TextStrokeTransparency = 0.1
			title.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			title.Font = Enum.Font.GothamBlack
			title.TextSize = 27

			local espToggle = Instance.new("TextButton", coluna)
			espToggle.Size = UDim2.new(0, 180, 0, 36)
			espToggle.Position = UDim2.new(0, 0, 0, 52)
			espToggle.BackgroundColor3 = espActive and Color3.fromRGB(150, 70, 180) or Color3.fromRGB(60, 20, 60)
			espToggle.Text = "☠️ Skeleton ESP"
			espToggle.TextColor3 = Color3.fromRGB(255,255,255)
			espToggle.TextStrokeTransparency = 0.1
			espToggle.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			espToggle.Font = Enum.Font.GothamBlack
			espToggle.TextSize = 18
			espToggle.AutoButtonColor = false
			local espToggleCorner = Instance.new("UICorner")
			espToggleCorner.CornerRadius = UDim.new(1,0)
			espToggleCorner.Parent = espToggle

			espToggle.MouseButton1Click:Connect(function()
				espActive = not espActive
				espToggle.BackgroundColor3 = espActive and Color3.fromRGB(150, 70, 180) or Color3.fromRGB(60, 20, 60)
				if espActive then enableESPDraw() else disableESPDraw() end
			end)
		end

		-- ===========================================
		-- AIMBOT COLUMN
		-- ===========================================
		do
			local coluna = colunas["Aimbot"]
			local title = Instance.new("TextLabel", coluna)
			title.Size = UDim2.new(1, 0, 0, 36)
			title.Position = UDim2.new(0, 0, 0, 0)
			title.BackgroundTransparency = 1
			title.Text = "Aimbot"
			title.TextColor3 = Color3.fromRGB(255,255,255)
			title.TextStrokeTransparency = 0.1
			title.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			title.Font = Enum.Font.GothamBlack
			title.TextSize = 27

			local aimToggle = Instance.new("TextButton", coluna)
			aimToggle.Size = UDim2.new(0, 135, 0, 36)
			aimToggle.Position = UDim2.new(0, 0, 0, 42)
			aimToggle.BackgroundColor3 = aimbotActive and Color3.fromRGB(220, 140, 30) or Color3.fromRGB(60, 40, 10)
			aimToggle.Text = aimbotActive and "Aimbot Ativo" or "Aimbot Off"
			aimToggle.TextColor3 = Color3.fromRGB(255,255,255)
			aimToggle.TextStrokeTransparency = 0.1
			aimToggle.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			aimToggle.Font = Enum.Font.GothamBlack
			aimToggle.TextSize = 20
			aimToggle.AutoButtonColor = false
			local aimTogCorner = Instance.new("UICorner")
			aimTogCorner.CornerRadius = UDim.new(1,0)
			aimTogCorner.Parent = aimToggle

			aimToggle.MouseButton1Click:Connect(function()
				aimbotActive = not aimbotActive
				aimToggle.BackgroundColor3 = aimbotActive and Color3.fromRGB(220, 140, 30) or Color3.fromRGB(60, 40, 10)
				aimToggle.Text = aimbotActive and "Aimbot Ativo" or "Aimbot Off"
				if aimbotActive then enableAimbot() else disableAimbot() end
			end)

			-- FOV Slider
			local fovLabel = Instance.new("TextLabel", coluna)
			fovLabel.Size = UDim2.new(0, 60, 0, 20)
			fovLabel.Position = UDim2.new(0, 150, 0, 50)
			fovLabel.BackgroundTransparency = 1
			fovLabel.Text = "FOV:"
			fovLabel.TextColor3 = Color3.fromRGB(255,255,255)
			fovLabel.TextStrokeTransparency = 0.1
			fovLabel.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			fovLabel.Font = Enum.Font.GothamBlack
			fovLabel.TextSize = 18

			local fovValue = Instance.new("TextLabel", coluna)
			fovValue.Size = UDim2.new(0, 38, 0, 20)
			fovValue.Position = UDim2.new(0, 220, 0, 50)
			fovValue.BackgroundTransparency = 1
			fovValue.Text = tostring(aimbotFOV)
			fovValue.TextColor3 = Color3.fromRGB(255,255,255)
			fovValue.TextStrokeTransparency = 0.1
			fovValue.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			fovValue.Font = Enum.Font.GothamBlack
			fovValue.TextSize = 18

			local fovBar = Instance.new("Frame", coluna)
			fovBar.Size = UDim2.new(0, 180, 0, 9)
			fovBar.Position = UDim2.new(0, 110, 0, 58)
			fovBar.BackgroundColor3 = Color3.fromRGB(70, 60, 40)
			fovBar.BorderSizePixel = 0
			local fovSliderCorner = Instance.new("UICorner")
			fovSliderCorner.CornerRadius = UDim.new(1,0)
			fovSliderCorner.Parent = fovBar

			local fovHandle = Instance.new("Frame", fovBar)
			fovHandle.Size = UDim2.new(0, 16, 0, 18)
			local function updateFovHandle() fovHandle.Position = UDim2.new(0, math.floor((aimbotFOV - FOV_MIN)/(FOV_MAX - FOV_MIN) * (180-16)), -0.5, -5) end
			updateFovHandle()
			fovHandle.BackgroundColor3 = Color3.fromRGB(255, 220, 50)
			fovHandle.BorderSizePixel = 0
			fovHandle.Active = true
			fovHandle.Draggable = false
			local fovHandleCorner = Instance.new("UICorner", fovHandle)
			fovHandleCorner.CornerRadius = UDim.new(1,0)
			makeSliderDraggable(fovHandle, fovBar, FOV_MIN, FOV_MAX,
				function() return aimbotFOV end,
				function(val) aimbotFOV = val end,
				fovValue, updateFovHandle)

			-- Distância do aimbot
			local distLabel = Instance.new("TextLabel", coluna)
			distLabel.Size = UDim2.new(0, 80, 0, 20)
			distLabel.Position = UDim2.new(0, 310, 0, 50)
			distLabel.BackgroundTransparency = 1
			distLabel.Text = "Distância:"
			distLabel.TextColor3 = Color3.fromRGB(255,255,255)
			distLabel.TextStrokeTransparency = 0.1
			distLabel.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			distLabel.Font = Enum.Font.GothamBlack
			distLabel.TextSize = 18

			local distValue = Instance.new("TextLabel", coluna)
			distValue.Size = UDim2.new(0, 45, 0, 20)
			distValue.Position = UDim2.new(0, 400, 0, 50)
			distValue.BackgroundTransparency = 1
			distValue.Text = tostring(aimbotDist)
			distValue.TextColor3 = Color3.fromRGB(255,255,255)
			distValue.TextStrokeTransparency = 0.1
			distValue.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			distValue.Font = Enum.Font.GothamBlack
			distValue.TextSize = 18

			local distBar = Instance.new("Frame", coluna)
			distBar.Size = UDim2.new(0, 180, 0, 9)
			distBar.Position = UDim2.new(0, 310, 0, 58)
			distBar.BackgroundColor3 = Color3.fromRGB(60, 60, 90)
			distBar.BorderSizePixel = 0
			local distSliderCorner = Instance.new("UICorner")
			distSliderCorner.CornerRadius = UDim.new(1,0)
			distSliderCorner.Parent = distBar

			local distHandle = Instance.new("Frame", distBar)
			distHandle.Size = UDim2.new(0, 16, 0, 18)
			local function updateDistHandle() distHandle.Position = UDim2.new(0, math.floor((aimbotDist - DIST_MIN)/(DIST_MAX - DIST_MIN) * (180-16)), -0.5, -5) end
			updateDistHandle()
			distHandle.BackgroundColor3 = Color3.fromRGB(100, 230, 255)
			distHandle.BorderSizePixel = 0
			distHandle.Active = true
			distHandle.Draggable = false
			local distHandleCorner = Instance.new("UICorner", distHandle)
			distHandleCorner.CornerRadius = UDim.new(1,0)
			makeSliderDraggable(distHandle, distBar, DIST_MIN, DIST_MAX,
				function() return aimbotDist end,
				function(val) aimbotDist = val end,
				distValue, updateDistHandle)

			-- Mouse button selector
			local mouseBtnSelector = Instance.new("TextButton", coluna)
			mouseBtnSelector.Size = UDim2.new(0, 110, 0, 28)
			mouseBtnSelector.Position = UDim2.new(0, 0, 0, 88)
			mouseBtnSelector.BackgroundColor3 = Color3.fromRGB(40, 40, 80)
			mouseBtnSelector.TextColor3 = Color3.fromRGB(255,255,255)
			mouseBtnSelector.TextStrokeTransparency = 0.1
			mouseBtnSelector.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			mouseBtnSelector.TextSize = 15
			mouseBtnSelector.Font = Enum.Font.GothamBlack
			mouseBtnSelector.Text = "Ativar: "..aimBtnText[aimbotMouseBtn]
			mouseBtnSelector.AutoButtonColor = true
			local mouseBtnCorner = Instance.new("UICorner", mouseBtnSelector)
			mouseBtnCorner.CornerRadius = UDim.new(1,0)
			mouseBtnSelector.MouseButton1Click:Connect(function()
				aimbotMouseBtn = (aimbotMouseBtn == Enum.UserInputType.MouseButton2) and Enum.UserInputType.MouseButton1 or Enum.UserInputType.MouseButton2
				mouseBtnSelector.Text = "Ativar: "..aimBtnText[aimbotMouseBtn]
			end)

			-- Show FOV toggle
			local showFovToggle = Instance.new("TextButton", coluna)
			showFovToggle.Size = UDim2.new(0, 130, 0, 28)
			showFovToggle.Position = UDim2.new(0, 130, 0, 118)
			showFovToggle.BackgroundColor3 = aimbotShowFov and Color3.fromRGB(220,140,30) or Color3.fromRGB(55,45,20)
			showFovToggle.Text = aimbotShowFov and "Mostrar FOV" or "Ocultar FOV"
			showFovToggle.TextColor3 = Color3.fromRGB(255,255,255)
			showFovToggle.TextStrokeTransparency = 0.1
			showFovToggle.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			showFovToggle.TextSize = 16
			showFovToggle.Font = Enum.Font.GothamBlack
			showFovToggle.AutoButtonColor = true
			local showFovCorner = Instance.new("UICorner", showFovToggle)
			showFovCorner.CornerRadius = UDim.new(1,0)
			showFovToggle.MouseButton1Click:Connect(function()
				aimbotShowFov = not aimbotShowFov
				showFovToggle.BackgroundColor3 = aimbotShowFov and Color3.fromRGB(220,140,30) or Color3.fromRGB(55,45,20)
				showFovToggle.Text = aimbotShowFov and "Mostrar FOV" or "Ocultar FOV"
			end)

			-- Kill Aura Toggle
			local killAuraBtn = Instance.new("TextButton", coluna)
			killAuraBtn.Size = UDim2.new(0, 135, 0, 36)
			killAuraBtn.Position = UDim2.new(0, 0, 0, 160)
			killAuraBtn.BackgroundColor3 = killAuraActive and Color3.fromRGB(220,30,30) or Color3.fromRGB(60,20,20)
			killAuraBtn.Text = killAuraActive and "Kill Aura ON" or "Kill Aura OFF"
			killAuraBtn.TextColor3 = Color3.fromRGB(255,255,255)
			killAuraBtn.TextStrokeTransparency = 0.1
			killAuraBtn.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			killAuraBtn.Font = Enum.Font.GothamBlack
			killAuraBtn.TextSize = 20
			killAuraBtn.AutoButtonColor = false
			local killAuraCorner = Instance.new("UICorner")
			killAuraCorner.CornerRadius = UDim.new(1,0)
			killAuraCorner.Parent = killAuraBtn
			killAuraBtn.MouseButton1Click:Connect(function()
				killAuraActive = not killAuraActive
				killAuraBtn.BackgroundColor3 = killAuraActive and Color3.fromRGB(220,30,30) or Color3.fromRGB(60,20,20)
				killAuraBtn.Text = killAuraActive and "Kill Aura ON" or "Kill Aura OFF"
				if killAuraActive then enableKillAura() else disableKillAura() end
			end)

			-- Kill Aura Range slider
			local kaRangeLabel = Instance.new("TextLabel", coluna)
			kaRangeLabel.Size = UDim2.new(0, 90, 0, 20)
			kaRangeLabel.Position = UDim2.new(0, 160, 0, 170)
			kaRangeLabel.BackgroundTransparency = 1
			kaRangeLabel.Text = "KA Range"
			kaRangeLabel.TextColor3 = Color3.fromRGB(255,255,255)
			kaRangeLabel.TextStrokeTransparency = 0.1
			kaRangeLabel.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			kaRangeLabel.Font = Enum.Font.GothamBlack
			kaRangeLabel.TextSize = 18

			local kaRangeValue = Instance.new("TextLabel", coluna)
			kaRangeValue.Size = UDim2.new(0, 40, 0, 20)
			kaRangeValue.Position = UDim2.new(0, 250, 0, 170)
			kaRangeValue.BackgroundTransparency = 1
			kaRangeValue.Text = tostring(killAuraRange)
			kaRangeValue.TextColor3 = Color3.fromRGB(255,255,255)
			kaRangeValue.TextStrokeTransparency = 0.1
			kaRangeValue.TextStrokeColor3 = Color3.fromRGB(0,0,0)
			kaRangeValue.Font = Enum.Font.GothamBlack
			kaRangeValue.TextSize = 18

			local kaRangeBar = Instance.new("Frame", coluna)
			kaRangeBar.Size = UDim2.new(0, 120, 0, 9)
			kaRangeBar.Position = UDim2.new(0, 300, 0, 178)
			kaRangeBar.BackgroundColor3 = Color3.fromRGB(220, 30, 30)
			kaRangeBar.BorderSizePixel = 0
			local kaRangeSliderCorner = Instance.new("UICorner")
			kaRangeSliderCorner.CornerRadius = UDim.new(1,0)
			kaRangeSliderCorner.Parent = kaRangeBar

			local kaRangeHandle = Instance.new("Frame", kaRangeBar)
			kaRangeHandle.Size = UDim2.new(0, 16, 0, 18)
			local function updateKaRangeHandle() kaRangeHandle.Position = UDim2.new(0, math.floor((killAuraRange - KA_MIN)/(KA_MAX-KA_MIN) * (120-16)), -0.5, -5) end
			updateKaRangeHandle()
			kaRangeHandle.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
			kaRangeHandle.BorderSizePixel = 0
			kaRangeHandle.Active = true
			kaRangeHandle.Draggable = false
			local kaRangeHandleCorner = Instance.new("UICorner", kaRangeHandle)
			kaRangeHandleCorner.CornerRadius = UDim.new(1,0)
			makeSliderDraggable(kaRangeHandle, kaRangeBar, KA_MIN, KA_MAX,
				function() return killAuraRange end,
				function(val) killAuraRange = val end,
				kaRangeValue, updateKaRangeHandle)
		end

		-- Botão flutuante e botão de fechar
		skullBall = Instance.new("ImageButton", screenGui)
		skullBall.Size = UDim2.new(0, 74, 0, 74)
		skullBall.Position = UDim2.new(0, 30, 0.5, -37)
		skullBall.BackgroundTransparency = 0.2
		skullBall.BackgroundColor3 = Color3.fromRGB(13,14,17)
		skullBall.Image = "rbxassetid://11797037673"
		skullBall.Visible = not painelVisibleState
		skullBall.Active = true
		skullBall.Draggable = true
		skullBall.ZIndex = 10
		local circle = Instance.new("UICorner")
		circle.CornerRadius = UDim.new(1,0)
		circle.Parent = skullBall

		local closeBtn = Instance.new("TextButton", menuFrame)
		closeBtn.Size = UDim2.new(0, 36, 0, 36)
		closeBtn.Position = UDim2.new(1, -44, 0, 16)
		closeBtn.BackgroundColor3 = Color3.fromRGB(60, 20, 20)
		closeBtn.Text = "✕"
		closeBtn.TextColor3 = Color3.fromRGB(255,255,255)
		closeBtn.TextStrokeTransparency = 0.1
		closeBtn.TextStrokeColor3 = Color3.fromRGB(0,0,0)
		closeBtn.TextSize = 22
		closeBtn.Font = Enum.Font.GothamBlack
		closeBtn.AutoButtonColor = false
		closeBtn.ZIndex = 8
		local closeCorner = Instance.new("UICorner")
		closeCorner.CornerRadius = UDim.new(1,0)
		closeCorner.Parent = closeBtn

		local function openMenu()
			menuFrame.Visible = true
			skullBall.Visible = false
			painelVisibleState = true
		end
		local function closeMenu()
			menuFrame.Visible = false
			skullBall.Visible = true
			painelVisibleState = false
		end

		skullBall.MouseButton1Click:Connect(openMenu)
		closeBtn.MouseButton1Click:Connect(closeMenu)
	end

	UIS.InputBegan:Connect(function(input, gpe)
		if gpe then return end
		if input.KeyCode == Enum.KeyCode.LeftControl then
			if menuFrame and skullBall then
				local abrir = not menuFrame.Visible
				menuFrame.Visible = abrir
				skullBall.Visible = not abrir
				painelVisibleState = abrir
			end
		end
	end)

	player.CharacterAdded:Connect(function(char)
		humanoid = char:WaitForChild("Humanoid")
		painelReabrindo = true
		createPainel()
		setWalkSpeed(wsValue)
		setJumpPower(jpValue)
		restoreInvisibleOnRespawn()
		painelReabrindo = false
	end)

	if player.Character then
		humanoid = player.Character:FindFirstChild("Humanoid")
		createPainel()
		setWalkSpeed(wsValue)
		setJumpPower(jpValue)
		restoreInvisibleOnRespawn()
	end

	player.PlayerGui.ChildRemoved:Connect(function(child)
		if child.Name == painelName and not painelReabrindo then
			task.wait(0.2)
			if player.Character and player.Character:FindFirstChild("Humanoid") then
				createPainel()
				setWalkSpeed(wsValue)
				setJumpPower(jpValue)
				restoreInvisibleOnRespawn()
			end
		end
	end)

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local localPlayer = Players.LocalPlayer

-- 🎮 estado global
local enabled = true
local espObjects = {}

------------------------------------------------
-- 📦 INVENTÁRIO
------------------------------------------------
local function getInv(player)
	local inv = player:FindFirstChild("Inv")
	if not inv then return {"sem inv"} end

	local items = {}

	for _, item in pairs(inv:GetChildren()) do
		if item:IsA("ValueBase") then
			table.insert(items, item.Name .. ": " .. tostring(item.Value))
		else
			table.insert(items, item.Name)
		end
	end

	if #items == 0 then
		return {"vazio"}
	end

	return items
end

------------------------------------------------
-- 👁️ ESP NA CABEÇA
------------------------------------------------
local function createESP(player)
	if player == localPlayer then return end

	local function setup(char)
		local head = char:WaitForChild("Head")

		local gui = Instance.new("BillboardGui")
		gui.Name = "InvESP"
		gui.Size = UDim2.new(0, 170, 0, 45)
		gui.StudsOffset = Vector3.new(0, 2.6, 0)
		gui.AlwaysOnTop = true
		gui.Parent = head

		-- 🪶 fundo transparente
		local bg = Instance.new("Frame")
		bg.Size = UDim2.new(1, 0, 1, 0)
		bg.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
		bg.BackgroundTransparency = 0.6
		bg.Parent = gui

		Instance.new("UICorner", bg).CornerRadius = UDim.new(0, 5)

		-- ✍️ TEXTO (NOME + INV)
		local text = Instance.new("TextLabel")
		text.Size = UDim2.new(1, -4, 1, 0)
		text.Position = UDim2.new(0, 2, 0, 0)
		text.BackgroundTransparency = 1
		text.TextColor3 = Color3.fromRGB(255, 255, 255)
		text.TextSize = 11
		text.Font = Enum.Font.GothamSemibold
		text.TextStrokeTransparency = 0.8
		text.TextWrapped = true
		text.TextYAlignment = Enum.TextYAlignment.Top
		text.TextXAlignment = Enum.TextXAlignment.Center
		text.Parent = bg

		espObjects[player] = gui

		-- 🔄 update loop
		task.spawn(function()
			while char.Parent do
				if enabled then
					local items = getInv(player)

					text.Text =
						"👤 " .. player.Name ..
						"\n📦 " .. table.concat(items, ", ")

					gui.Enabled = true
				else
					gui.Enabled = false
				end

				task.wait(0.5)
			end
		end)
	end

	if player.Character then
		setup(player.Character)
	end

	player.CharacterAdded:Connect(setup)
end

------------------------------------------------
-- 👥 PLAYERS
------------------------------------------------
for _, p in pairs(Players:GetPlayers()) do
	createESP(p)
end

Players.PlayerAdded:Connect(createESP)

------------------------------------------------
-- 🎛️ GUI MODERNA
------------------------------------------------
local gui = Instance.new("ScreenGui")
gui.Name = "InvESPPanel"
gui.ResetOnSpawn = false
gui.Parent = localPlayer:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 80)
frame.Position = UDim2.new(0.05, 0, 0.3, 0)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.Parent = gui

Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(0, 170, 255)
stroke.Thickness = 2
stroke.Parent = frame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 25)
title.BackgroundTransparency = 1
title.Text = "📦 INV ESP"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 14
title.Parent = frame

local btn = Instance.new("TextButton")
btn.Size = UDim2.new(0.9, 0, 0, 35)
btn.Position = UDim2.new(0.05, 0, 0.4, 0)
btn.Text = "ESP: ON"
btn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
btn.TextColor3 = Color3.fromRGB(255, 255, 255)
btn.Font = Enum.Font.GothamBold
btn.TextSize = 13
btn.Parent = frame

Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

------------------------------------------------
-- 🔘 TOGGLE CORRIGIDO
------------------------------------------------
btn.MouseButton1Click:Connect(function()
	enabled = not enabled

	btn.Text = enabled and "ESP: ON" or "ESP: OFF"
	btn.BackgroundColor3 = enabled and Color3.fromRGB(0,170,255) or Color3.fromRGB(100,100,100)

	for _, obj in pairs(espObjects) do
		if obj then
			obj.Enabled = enabled
		end
	end
end)

------------------------------------------------
-- 🖱️ DRAG GUI
------------------------------------------------
local dragging = false
local dragStart
local startPos

frame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = frame.Position
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		local delta = input.Position - dragStart

		frame.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)

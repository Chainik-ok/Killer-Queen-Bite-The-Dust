local player = game.Players.LocalPlayer

local tool = Instance.new("Tool")
tool.Name = "BitesTheDust"
tool.Parent = player.Backpack

local handle = Instance.new("Part")
handle.Name = "Handle"
handle.Size = Vector3.new(1,1,1)
handle.Parent = tool

local mesh = Instance.new("SpecialMesh", handle)
mesh.MeshId = "rbxassetid://28511792"
mesh.TextureId = "rbxassetid://28511890"
mesh.Scale = Vector3.new(2,2,2)

local soundClick = Instance.new("Sound", handle)
soundClick.Name = "KQClick"
soundClick.SoundId = "rbxassetid://920181099"
soundClick.Volume = 1

local soundMain = Instance.new("Sound", handle)
soundMain.Name = "Sound"
soundMain.SoundId = "rbxassetid://1409923811"
soundMain.Volume = 1

local function createEmitter(name, texture, color, size, lifetime, rate)
    local emitter = Instance.new("ParticleEmitter", handle)
    emitter.Name = name
    emitter.Texture = texture
    emitter.Color = color
    emitter.Size = size
    emitter.Lifetime = lifetime
    emitter.Rate = rate
    emitter.Enabled = false
    emitter.LockedToPart = true
    emitter.Drag = 2
    emitter.ZOffset = -2
    emitter.SpreadAngle = Vector2.new(360,360)
    emitter.RotSpeed = NumberRange.new(180)
    emitter.Speed = NumberRange.new(20)
    return emitter
end

local sparkles = createEmitter("Aura","rbxassetid://321556991",ColorSequence.new(Color3.fromRGB(255,100,100), Color3.fromRGB(255,0,0)),NumberSequence.new(30),NumberRange.new(4),40)
local sparkles2 = createEmitter("Plasma","rbxassetid://359293256",ColorSequence.new(Color3.fromRGB(255,255,255)),NumberSequence.new(10),NumberRange.new(5),20)
local sparkles3 = createEmitter("Smoke","rbxassetid://569507725",ColorSequence.new(Color3.fromRGB(255,255,255)),NumberSequence.new(10),NumberRange.new(2),20)

local positionHistory = {}
local saveInterval = 0.1
local maxDuration = 50
local movementBlocked = false
local cooldown = 20
local lastUse = -cooldown

local cdGui = Instance.new("ScreenGui", player.PlayerGui)
local cdLabel = Instance.new("TextLabel", cdGui)
cdLabel.Position = UDim2.new(0.9,0,0.82,-50)
cdLabel.Size = UDim2.new(0.08,0,0.05,0)
cdLabel.BackgroundTransparency = 0.5
cdLabel.BackgroundColor3 = Color3.new(0,0,0)
cdLabel.TextColor3 = Color3.new(1,0,0)
cdLabel.TextScaled = true
cdLabel.Text = "READY"

local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.Name = "BitesTheDustMenu"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 200, 0, 100)
frame.Position = UDim2.new(0.5, -100, 0.3, -50)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0
frame.Visible = true
frame.Active = true
frame.Draggable = true 

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,30)
title.Position = UDim2.new(0,0,0,0)
title.Text = "Set Rewind Time"
title.TextColor3 = Color3.new(1,1,1)
title.BackgroundTransparency = 1
title.TextScaled = true

local slider = Instance.new("TextBox", frame)
slider.Size = UDim2.new(1,-20,0,30)
slider.Position = UDim2.new(0,10,0,40)
slider.Text = "35" -- дефолт
slider.ClearTextOnFocus = false
slider.TextColor3 = Color3.new(1,1,1)
slider.BackgroundColor3 = Color3.fromRGB(50,50,50)
slider.TextScaled = true

local hideBtn = Instance.new("TextButton", frame)
hideBtn.Size = UDim2.new(0.3,0,0,20)
hideBtn.Position = UDim2.new(0.35,0,0.75,0)
hideBtn.Text = "Hide"
hideBtn.TextColor3 = Color3.new(1,1,1)
hideBtn.BackgroundColor3 = Color3.fromRGB(60,60,60)
hideBtn.TextScaled = true

hideBtn.MouseButton1Click:Connect(function()
    frame.Visible = false
end)

local toggleButton = Instance.new("TextButton", player.PlayerGui)
toggleButton.Size = UDim2.new(0, 100, 0, 30)
toggleButton.Position = UDim2.new(0.5, -50, 0.85, 0)
toggleButton.Text = "Menu"
toggleButton.TextColor3 = Color3.new(1,1,1)
toggleButton.BackgroundColor3 = Color3.fromRGB(40,40,40)
toggleButton.TextScaled = true

toggleButton.MouseButton1Click:Connect(function()
    frame.Visible = not frame.Visible
end)

local function getRewindTime()
    local val = tonumber(slider.Text)
    if not val then return 35 end
    if val < 1 then val = 1 end
    if val > 50 then val = 50 end
    return val
end

spawn(function()
	while true do
		local char = player.Character
		if char and (tool.Parent == player.Backpack or tool.Parent == char) then
			local hrp = char:FindFirstChild("HumanoidRootPart")
			if hrp then
				table.insert(positionHistory, {tick(), hrp.CFrame})
				while #positionHistory > maxDuration / saveInterval do
					table.remove(positionHistory,1)
				end
			end
		end
		local elapsed = tick() - lastUse
		if elapsed < cooldown then
			cdLabel.Text = string.format("%.1f", cooldown - elapsed)
		else
			cdLabel.Text = "READY"
		end
		task.wait(saveInterval)
	end
end)

local function getPastCFrame(rewindTime)
	local now = tick()
	for i = #positionHistory,1,-1 do
		if now - positionHistory[i][1] >= rewindTime then
			return positionHistory[i][2]
		end
	end
	return positionHistory[1] and positionHistory[1][2]
end


tool.Activated:Connect(function()
	if tick() - lastUse < cooldown then return end
	lastUse = tick()

	local char = player.Character
	local hum = char and char:FindFirstChild("Humanoid")
	local hrp = char and char:FindFirstChild("HumanoidRootPart")
	if not hum or not hrp then return end

	local rewindTime = getRewindTime()

	movementBlocked = true
	hum.WalkSpeed = 0
	hum.JumpPower = 0
	tool.CanBeDropped = false

	sparkles.Enabled = true
	sparkles2.Enabled = true
	sparkles3.Enabled = true
	soundMain:Play()

	local cam = workspace.CurrentCamera
	local guiFrame = Instance.new("ScreenGui", player.PlayerGui)
	local whiteFrame = Instance.new("Frame", guiFrame)
	whiteFrame.Size = UDim2.new(1,0,1,0)
	whiteFrame.BackgroundColor3 = Color3.new(1,1,1)
	whiteFrame.BackgroundTransparency = 1

	task.wait(3) -- дым

	local pastCFrame = getPastCFrame(rewindTime)
	if pastCFrame then
		hrp.CFrame = pastCFrame
	end

	hum.WalkSpeed = 16
	hum.JumpPower = 50
	movementBlocked = false
	tool.CanBeDropped = true

	sparkles.Enabled = false
	sparkles2.Enabled = false
	sparkles3.Enabled = false
	guiFrame:Destroy()
end)

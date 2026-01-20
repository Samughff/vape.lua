-- VAPE CLOUD | MOTOR6D + MENU FIXO
-- Créditos: Samuel

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
if not player then return end

-- ================= TOOL =================
local Tool = Instance.new("Tool")
Tool.Name = "Vape"
Tool.RequiresHandle = true
Tool.CanBeDropped = false
Tool.Parent = player.Backpack

-- ================= HANDLE =================
local Handle = Instance.new("Part")
Handle.Name = "Handle"
Handle.Size = Vector3.new(0.25, 0.25, 1.2)
Handle.Material = Enum.Material.SmoothPlastic
Handle.Color = Color3.fromRGB(20,20,20)
Handle.CanCollide = false
Handle.Massless = true
Handle.Parent = Tool

-- ================= MOTOR6D =================
local motor
local handC0
local mouthC0

Tool.Equipped:Connect(function()
	local char = player.Character or player.CharacterAdded:Wait()
	local hand = char:FindFirstChild("RightHand") or char:FindFirstChild("Right Arm")

	if hand and not motor then
		motor = Instance.new("Motor6D")
		motor.Name = "VapeMotor"
		motor.Part0 = hand
		motor.Part1 = Handle

		-- posição normal na mão
		handC0 = CFrame.new(0, -1, 0.4) * CFrame.Angles(math.rad(-90), 0, 0)
		motor.C0 = handC0

		-- posição na boca (ajustada)
		mouthC0 = CFrame.new(0.15, -0.25, -0.75)
			* CFrame.Angles(math.rad(-90), math.rad(15), 0)

		motor.Parent = hand
	end
end)

-- ================= ATTACHMENT =================
local smokeAttachment = Instance.new("Attachment")
smokeAttachment.Position = Vector3.new(0, 0, -0.6)
smokeAttachment.Parent = Handle

-- ================= LED =================
local led = Instance.new("PointLight", Handle)
led.Color = Color3.fromRGB(0,255,220)
led.Range = 8
led.Brightness = 0

-- ================= FUMAÇA =================
local smoke = Instance.new("ParticleEmitter", smokeAttachment)
smoke.Texture = "rbxassetid://771221224"
smoke.Rate = 0
smoke.Lifetime = NumberRange.new(3,5)
smoke.Speed = NumberRange.new(6,10)
smoke.Acceleration = Vector3.new(0,3,0)
smoke.EmissionDirection = Enum.NormalId.Front
smoke.SpreadAngle = Vector2.new(60,60)
smoke.Size = NumberSequence.new{
	NumberSequenceKeypoint.new(0,1.5),
	NumberSequenceKeypoint.new(1,8)
}
smoke.Transparency = NumberSequence.new{
	NumberSequenceKeypoint.new(0,0.1),
	NumberSequenceKeypoint.new(1,1)
}

-- ================= SOM =================
local sound = Instance.new("Sound", Handle)
sound.SoundId = "rbxassetid://9118823101"
sound.Volume = 0
sound.Looped = true

-- ================= ANIMAÇÃO =================
local function moveToMouth()
	if motor then
		TweenService:Create(
			motor,
			TweenInfo.new(0.25, Enum.EasingStyle.Sine, Enum.EasingDirection.Out),
			{C0 = mouthC0}
		):Play()
	end
end

local function moveToHand()
	if motor then
		TweenService:Create(
			motor,
			TweenInfo.new(0.25, Enum.EasingStyle.Sine, Enum.EasingDirection.Out),
			{C0 = handC0}
		):Play()
	end
end

-- ================= DRAG =================
local function makeDraggable(frame)
	local dragging, dragStart, startPos

	frame.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = frame.Position
		end
	end)

	frame.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then
			dragging = false
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch) then
			local delta = input.Position - dragStart
			frame.Position = UDim2.new(
				startPos.X.Scale, startPos.X.Offset + delta.X,
				startPos.Y.Scale, startPos.Y.Offset + delta.Y
			)
		end
	end)
end

-- ================= GUI FIXA =================
local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.Name = "VapeUI"
gui.ResetOnSpawn = false

local panel = Instance.new("Frame", gui)
panel.Size = UDim2.new(0,300,0,170)
panel.Position = UDim2.new(0.5,-150,0.5,80)
panel.BackgroundColor3 = Color3.fromRGB(15,15,15)
panel.BorderSizePixel = 0
makeDraggable(panel)

local smokeBtn = Instance.new("TextButton", panel)
smokeBtn.Size = UDim2.new(1,-20,0,55)
smokeBtn.Position = UDim2.new(0,10,0,10)
smokeBtn.Text = "💨 FUMAR"
smokeBtn.TextScaled = true
smokeBtn.BackgroundColor3 = Color3.fromRGB(10,10,10)
smokeBtn.TextColor3 = Color3.new(1,1,1)

local credits = Instance.new("TextLabel", panel)
credits.Size = UDim2.new(1,0,0,30)
credits.Position = UDim2.new(0,0,1,-35)
credits.Text = "Créditos: Samuel"
credits.TextScaled = true
credits.BackgroundTransparency = 1
credits.TextColor3 = Color3.fromRGB(180,180,180)

-- ================= FUNCIONAL =================
local smoking = false

smokeBtn.MouseButton1Down:Connect(function()
	if smoking then return end
	smoking = true
	moveToMouth()

	smoke.Rate = 200
	smoke:Emit(40)
	sound:Play()
	TweenService:Create(sound, TweenInfo.new(0.2), {Volume = 1}):Play()
	TweenService:Create(led, TweenInfo.new(0.2), {Brightness = 5}):Play()
end)

smokeBtn.MouseButton1Up:Connect(function()
	smoking = false
	moveToHand()

	smoke.Rate = 0
	smoke:Emit(150)
	TweenService:Create(sound, TweenInfo.new(0.3), {Volume = 0}):Play()
	TweenService:Create(led, TweenInfo.new(0.3), {Brightness = 0}):Play()
end)

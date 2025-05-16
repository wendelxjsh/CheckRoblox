local MaruLib = {}
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")

local SavePath = "MaruHub_Config.json"
local Saved = isfile(SavePath) and HttpService:JSONDecode(readfile(SavePath)) or {}

local function Tween(obj, props, time)
	local tween = TweenService:Create(obj, TweenInfo.new(time or 0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), props)
	tween:Play()
end

function MaruLib:CreateWindow(config)
	local UI = Instance.new("ScreenGui", game:GetService("CoreGui"))
	UI.Name = config.Name or "MaruHub"
	UI.ResetOnSpawn = false

	local Main = Instance.new("Frame", UI)
	Main.Size = UDim2.new(0, 600, 0, 400)
	Main.Position = UDim2.new(0.5, -300, 0.5, -200)
	Main.BackgroundColor3 = Color3.fromRGB(20,20,20)
	Main.BorderSizePixel = 0
	Main.ClipsDescendants = true

	local UICorner = Instance.new("UICorner", Main)
	UICorner.CornerRadius = UDim.new(0, 10)

	local TabHolder = Instance.new("Frame", Main)
	TabHolder.Size = UDim2.new(0, 120, 1, 0)
	TabHolder.BackgroundTransparency = 1

	local TabButtons = Instance.new("UIListLayout", TabHolder)
	TabButtons.Padding = UDim.new(0, 4)

	local PageHolder = Instance.new("Frame", Main)
	PageHolder.Position = UDim2.new(0, 130, 0, 10)
	PageHolder.Size = UDim2.new(1, -140, 1, -20)
	PageHolder.BackgroundTransparency = 1

	local Tabs = {}

	function MaruLib:AddTab(tabName)
		local Button = Instance.new("TextButton", TabHolder)
		Button.Size = UDim2.new(1, -10, 0, 40)
		Button.Text = tabName
		Button.BackgroundColor3 = Color3.fromRGB(30,30,30)
		Button.TextColor3 = Color3.new(1,1,1)
		Button.Font = Enum.Font.Gotham
		Button.TextSize = 14

		local Corner = Instance.new("UICorner", Button)
		Corner.CornerRadius = UDim.new(0, 6)

		local Page = Instance.new("ScrollingFrame", PageHolder)
		Page.Size = UDim2.new(1, 0, 1, 0)
		Page.CanvasSize = UDim2.new(0, 0, 1, 0)
		Page.BackgroundTransparency = 1
		Page.Visible = false
		Page.ScrollBarThickness = 3

		local Layout = Instance.new("UIListLayout", Page)
		Layout.Padding = UDim.new(0, 6)

		Tabs[tabName] = Page

		Button.MouseButton1Click:Connect(function()
			for _, v in pairs(PageHolder:GetChildren()) do
				if v:IsA("ScrollingFrame") then v.Visible = false end
			end
			Page.Visible = true
		end)

		if not PageHolder:FindFirstChildWhichIsA("ScrollingFrame") then
			Page.Visible = true
		end

		return Page
	end

	function MaruLib:AddToggle(tab, config)
		local Toggle = Instance.new("TextButton", tab)
		Toggle.Size = UDim2.new(1, -10, 0, 35)
		Toggle.Text = config.Text or "Toggle"
		Toggle.BackgroundColor3 = Color3.fromRGB(40,40,40)
		Toggle.TextColor3 = Color3.new(1,1,1)
		Toggle.Font = Enum.Font.Gotham
		Toggle.TextSize = 14

		Instance.new("UICorner", Toggle).CornerRadius = UDim.new(0, 6)

		local state = Saved[config.Flag] or false

		local function update()
			Saved[config.Flag] = state
			writefile(SavePath, HttpService:JSONEncode(Saved))
			Tween(Toggle, {BackgroundColor3 = state and Color3.fromRGB(60,130,255) or Color3.fromRGB(40,40,40)}, 0.2)
			if config.Callback then config.Callback(state) end
		end

		update()

		Toggle.MouseButton1Click:Connect(function()
			state = not state
			update()
		end)
	end

	function MaruLib:AddButton(tab, config)
		local Button = Instance.new("TextButton", tab)
		Button.Size = UDim2.new(1, -10, 0, 35)
		Button.Text = config.Text or "Button"
		Button.BackgroundColor3 = Color3.fromRGB(40,40,40)
		Button.TextColor3 = Color3.new(1,1,1)
		Button.Font = Enum.Font.Gotham
		Button.TextSize = 14

		Instance.new("UICorner", Button).CornerRadius = UDim.new(0, 6)

		Button.MouseButton1Click:Connect(function()
			if config.Callback then config.Callback() end
		end)
	end

	function MaruLib:AddTextbox(tab, config)
		local TextBox = Instance.new("TextBox", tab)
		TextBox.Size = UDim2.new(1, -10, 0, 35)
		TextBox.PlaceholderText = config.Placeholder or "Digite aqui..."
		TextBox.Text = ""
		TextBox.BackgroundColor3 = Color3.fromRGB(40,40,40)
		TextBox.TextColor3 = Color3.new(1,1,1)
		TextBox.Font = Enum.Font.Gotham
		TextBox.TextSize = 14

		Instance.new("UICorner", TextBox).CornerRadius = UDim.new(0, 6)

		TextBox.FocusLost:Connect(function()
			if config.Callback then config.Callback(TextBox.Text) end
		end)
	end

	function MaruLib:AddDropdown(tab, config)
		local Container = Instance.new("Frame", tab)
		Container.Size = UDim2.new(1, -10, 0, 35)
		Container.BackgroundColor3 = Color3.fromRGB(40,40,40)
		Container.ClipsDescendants = true

		local Corner = Instance.new("UICorner", Container)
		Corner.CornerRadius = UDim.new(0, 6)

		local Button = Instance.new("TextButton", Container)
		Button.Size = UDim2.new(1, 0, 1, 0)
		Button.Text = config.Text or "Selecionar"
		Button.TextColor3 = Color3.new(1,1,1)
		Button.BackgroundTransparency = 1
		Button.Font = Enum.Font.Gotham
		Button.TextSize = 14

		local Open = false

		local function createOptions()
			for _, v in pairs(Container:GetChildren()) do
				if v:IsA("TextButton") and v ~= Button then v:Destroy() end
			end
			for _, option in pairs(config.Options) do
				local Opt = Instance.new("TextButton", Container)
				Opt.Size = UDim2.new(1, 0, 0, 30)
				Opt.Position = UDim2.new(0, 0, 1, 0)
				Opt.Text = option
				Opt.TextColor3 = Color3.new(1,1,1)
				Opt.BackgroundColor3 = Color3.fromRGB(50,50,50)
				Opt.Font = Enum.Font.Gotham
				Opt.TextSize = 14
				Opt.Visible = false

				Instance.new("UICorner", Opt).CornerRadius = UDim.new(0, 4)

				Opt.MouseButton1Click:Connect(function()
					Button.Text = option
					if config.Callback then config.Callback(option) end
					Open = false
					Container.Size = UDim2.new(1, -10, 0, 35)
				end)
			end
		end

		createOptions()

		Button.MouseButton1Click:Connect(function()
			Open = not Open
			for _, v in pairs(Container:GetChildren()) do
				if v:IsA("TextButton") and v ~= Button then v.Visible = Open end
			end
			local count = #config.Options
			Container.Size = Open and UDim2.new(1, -10, 0, 35 + (30 * count)) or UDim2.new(1, -10, 0, 35)
		end)
	end

	function MaruLib:AddSlider(tab, config)
		local Container = Instance.new("Frame", tab)
		Container.Size = UDim2.new(1, -10, 0, 35)
		Container.BackgroundColor3 = Color3.fromRGB(40,40,40)
		Instance.new("UICorner", Container).CornerRadius = UDim.new(0, 6)

		local Label = Instance.new("TextLabel", Container)
		Label.Size = UDim2.new(1, 0, 1, 0)
		Label.Text = config.Text or "Slider"
		Label.TextColor3 = Color3.new(1,1,1)
		Label.BackgroundTransparency = 1
		Label.Font = Enum.Font.Gotham
		Label.TextSize = 14

		local SliderBar = Instance.new("Frame", Container)
		SliderBar.Position = UDim2.new(0, 0, 1, -6)
		SliderBar.Size = UDim2.new(1, 0, 0, 6)
		SliderBar.BackgroundColor3 = Color3.fromRGB(30,30,30)

		local Fill = Instance.new("Frame", SliderBar)
		Fill.Size = UDim2.new(0.5, 0, 1, 0)
		Fill.BackgroundColor3 = Color3.fromRGB(80,150,255)

		local dragging = false
		local function setSlider(input)
			local pos = math.clamp((input.Position.X - SliderBar.AbsolutePosition.X) / SliderBar.AbsoluteSize.X, 0, 1)
			Fill.Size = UDim2.new(pos, 0, 1, 0)
			local value = math.floor((config.Max - config.Min) * pos + config.Min)
			if config.Callback then config.Callback(value) end
		end

		SliderBar.InputBegan:Connect(function(input)
			if input.UserInputType == Enum.UserInputType.MouseButton1 then
				dragging = true
				setSlider(input)
			end
		end)

		game:GetService("UserInputService").InputEnded:Connect(function(input)
			if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
		end)

		game:GetService("UserInputService").InputChanged:Connect(function(input)
			if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
				setSlider(input)
			end
		end)
	end

	return self
end

return MaruLib

-- ===============================
-- SERVIÇOS
-- ===============================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local player = Players.LocalPlayer

-- ===============================
-- ESTADOS
-- ===============================
local espPlayers, espBase = false, false
local infiniteJump, godMode, speedEnabled = false, false, false
local speedValue = 32

-- ===============================
-- FUNÇÕES BASE
-- ===============================
local function getChar() return player.Character or player.CharacterAdded:Wait() end
local function getHumanoid() return getChar():WaitForChild("Humanoid") end
local function getRoot() return getChar():WaitForChild("HumanoidRootPart") end
local function applyGod()
	local hum = getHumanoid()
	hum.MaxHealth = math.huge
	hum.Health = hum.MaxHealth
end
player.CharacterAdded:Connect(function()
	task.wait(0.3)
	if godMode then applyGod() end
end)

-- ===============================
-- SPEED
-- ===============================
RunService.Heartbeat:Connect(function()
	if speedEnabled and player.Character then
		local hum = getHumanoid()
		local root = getRoot()
		local dir = hum.MoveDirection
		if dir.Magnitude>0 then
			root.Velocity = Vector3.new(dir.X*speedValue, root.Velocity.Y, dir.Z*speedValue)
		end
	end
end)

-- ===============================
-- INFINITE JUMP
-- ===============================
UIS.JumpRequest:Connect(function()
	if infiniteJump then
		local root = getRoot()
		root.Velocity = Vector3.new(root.Velocity.X, 40, root.Velocity.Z)
	end
end)

-- ===============================
-- ESP
-- ===============================
local espBoxes = {}
local function createESP(plr)
	if plr==player or not plr.Character or espBoxes[plr] then return end
	local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
	if not hrp then return end
	local box = Instance.new("BoxHandleAdornment")
	box.Adornee = hrp
	box.Size = Vector3.new(4,6,2)
	box.Color3 = Color3.fromRGB(160,90,255)
	box.Transparency = 0.4
	box.AlwaysOnTop = true
	box.ZIndex = 10
	box.Parent = workspace
	espBoxes[plr]=box
end
local function clearESP() for _,b in pairs(espBoxes) do if b then b:Destroy() end end espBoxes={} end
local function toggleESPPlayers(state)
	espPlayers=state
	if state then for _,plr in pairs(Players:GetPlayers()) do createESP(plr) end else clearESP() end
end
Players.PlayerAdded:Connect(function(plr)
	plr.CharacterAdded:Connect(function() if espPlayers then task.wait(0.3) createESP(plr) end end)
end)

-- ===============================
-- ESP BASE
-- ===============================
local baseCache = {}
local function toggleESPBase(state)
	espBase=state
	if state then
		for _,obj in pairs(workspace:GetDescendants()) do
			if obj:IsA("BasePart") and obj.Name:lower():find("base") then
				baseCache[obj]=obj.Transparency
				obj.Transparency=0.6
			end
		end
	else
		for part,old in pairs(baseCache) do
			if part and part.Parent then part.Transparency=old end
		end
		baseCache={}
	end
end

-- ===============================
-- GUI
-- ===============================
local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.Name="FACT4_GUI"
gui.ResetOnSpawn=false

-- PAINEL PRINCIPAL (MENOR)
local main = Instance.new("Frame", gui)
main.Size=UDim2.new(0,350,0,380)
main.Position=UDim2.new(0.5,-175,0.5,-190)
main.BackgroundColor3=Color3.fromRGB(14,10,22)
main.BorderSizePixel=0
main.Active=true
main.Draggable=true
Instance.new("UICorner", main).CornerRadius=UDim.new(0,18)

-- HEADER
local header = Instance.new("TextLabel", main)
header.Size=UDim2.new(1,0,0,40)
header.Text="FACT 4"
header.Font=Enum.Font.GothamBold
header.TextSize=16
header.TextColor3=Color3.fromRGB(240,240,255)
header.BackgroundTransparency=1

-- MINIMIZAR
local minimize = Instance.new("TextButton", main)
minimize.Size=UDim2.new(0,28,0,28)
minimize.Position=UDim2.new(1,-36,0,6)
minimize.Text="—"
minimize.Font=Enum.Font.GothamBold
minimize.TextSize=18
minimize.TextColor3=Color3.fromRGB(255,255,255)
minimize.BackgroundColor3=Color3.fromRGB(80,60,140)
minimize.BorderSizePixel=0
Instance.new("UICorner", minimize).CornerRadius=UDim.new(1,0)

local mini = Instance.new("TextButton", gui)
mini.Size=UDim2.new(0,140,0,38)
mini.Position=UDim2.new(0.02,0,0.5,0)
mini.Text="FACT 4"
mini.Font=Enum.Font.GothamBold
mini.TextSize=14
mini.TextColor3=Color3.fromRGB(255,255,255)
mini.BackgroundColor3=Color3.fromRGB(20,14,30)
mini.BorderSizePixel=0
mini.Visible=false
mini.Active=true
mini.Draggable=true
Instance.new("UICorner", mini).CornerRadius=UDim.new(0,16)

minimize.MouseButton1Click:Connect(function()
	main.Visible=false
	mini.Visible=true
end)
mini.MouseButton1Click:Connect(function()
	main.Visible=true
	mini.Visible=false
end)

-- ===============================
-- ABAS
-- ===============================
local tabBar=Instance.new("Frame", main)
tabBar.Size=UDim2.new(1,-20,0,28)
tabBar.Position=UDim2.new(0,10,0,45)
tabBar.BackgroundTransparency=1

local function createTab(text,pos)
	local b=Instance.new("TextButton", tabBar)
	b.Size=UDim2.new(0.33,-4,1,0)
	b.Position=UDim2.new(pos,0,0,0)
	b.Text=text
	b.Font=Enum.Font.GothamBold
	b.TextSize=13
	b.BorderSizePixel=0
	Instance.new("UICorner", b).CornerRadius=UDim.new(0,12)
	return b
end

local espTab=createTab("ESP",0)
local pvpTab=createTab("PVP",0.33)
local miscTab=createTab("MISC",0.66)

-- FRAMES DE CONTEÚDO
local function createContentFrame()
	local f=Instance.new("Frame", main)
	f.Size=UDim2.new(1,-20,1,-85)
	f.Position=UDim2.new(0,10,0,80)
	f.BackgroundTransparency=1
	local list=Instance.new("UIListLayout",f)
	list.Padding=UDim.new(0,10)
	list.HorizontalAlignment=Enum.HorizontalAlignment.Center
	Instance.new("UIPadding", f).PaddingTop=UDim.new(0,10)
	return f
end

local espFrame=createContentFrame()
local pvpFrame=createContentFrame()
pvpFrame.Visible=false
local miscFrame=createContentFrame()
miscFrame.Visible=false

local function selectTab(tab)
	espFrame.Visible=false pvpFrame.Visible=false miscFrame.Visible=false
	espTab.BackgroundColor3=Color3.fromRGB(25,20,35)
	pvpTab.BackgroundColor3=Color3.fromRGB(25,20,35)
	miscTab.BackgroundColor3=Color3.fromRGB(25,20,35)
	if tab=="ESP" then espFrame.Visible=true espTab.BackgroundColor3=Color3.fromRGB(120,80,200)
	elseif tab=="PVP" then pvpFrame.Visible=true pvpTab.BackgroundColor3=Color3.fromRGB(120,80,200)
	else miscFrame.Visible=true miscTab.BackgroundColor3=Color3.fromRGB(120,80,200) end
end

espTab.MouseButton1Click:Connect(function() selectTab("ESP") end)
pvpTab.MouseButton1Click:Connect(function() selectTab("PVP") end)
miscTab.MouseButton1Click:Connect(function() selectTab("MISC") end)
selectTab("ESP")

-- ===============================
-- FUNÇÃO DE OPÇÃO
-- ===============================
local function option(parent,title,desc,get,toggle)
	local b=Instance.new("TextButton", parent)
	b.Size=UDim2.new(0.95,0,0,50)
	b.BorderSizePixel=0
	b.AutoButtonColor=false
	Instance.new("UICorner", b).CornerRadius=UDim.new(0,12)

	local t=Instance.new("TextLabel", b)
	t.Size=UDim2.new(1,-16,0,24)
	t.Position=UDim2.new(0,12,0,4)
	t.Text=title
	t.Font=Enum.Font.GothamBold
	t.TextSize=14
	t.TextXAlignment=Enum.TextXAlignment.Left
	t.BackgroundTransparency=1

	local d=Instance.new("TextLabel", b)
	d.Size=UDim2.new(1,-16,0,18)
	d.Position=UDim2.new(0,12,0,26)
	d.Text=desc
	d.Font=Enum.Font.Gotham
	d.TextSize=11
	d.TextXAlignment=Enum.TextXAlignment.Left
	d.BackgroundTransparency=1

	local function refresh()
		if get() then
			b.BackgroundColor3=Color3.fromRGB(130,90,210)
			t.TextColor3=Color3.new(1,1,1)
			d.TextColor3=Color3.fromRGB(230,230,255)
		else
			b.BackgroundColor3=Color3.fromRGB(22,18,32)
			t.TextColor3=Color3.fromRGB(220,220,255)
			d.TextColor3=Color3.fromRGB(170,170,200)
		end
	end
	b.MouseButton1Click:Connect(function() toggle() refresh() end)
	refresh()
end

-- ===============================
-- OPÇÕES ESP
-- ===============================
option(espFrame,"ESP PLAYER","Destaca jogadores",
	function() return espPlayers end,
	function() toggleESPPlayers(not espPlayers) end
)
option(espFrame,"ESP BASE","Base transparente",
	function() return espBase end,
	function() toggleESPBase(not espBase) end
)

-- ===============================
-- OPÇÕES PVP
-- ===============================
option(pvpFrame,"INFINITE JUMP","Pulo controlado",
	function() return infiniteJump end,
	function() infiniteJump=not infiniteJump end
)
option(pvpFrame,"GOD MODE","Vida infinita",
	function() return godMode end,
	function() godMode=not godMode if godMode then applyGod() end end
)
option(pvpFrame,"SPEED","Velocidade personalizada",
	function() return speedEnabled end,
	function() speedEnabled=not speedEnabled end
)

-- SPEED INPUT
local speedBox = Instance.new("TextBox", pvpFrame)
speedBox.Size=UDim2.new(0.95,0,0,36)
speedBox.PlaceholderText="Velocidade (30-300)"
speedBox.Font=Enum.Font.Gotham
speedBox.TextSize=13
speedBox.TextColor3=Color3.fromRGB(255,255,255)
speedBox.BackgroundColor3=Color3.fromRGB(18,14,28)
speedBox.BorderSizePixel=0
speedBox.ClearTextOnFocus=false
Instance.new("UICorner", speedBox).CornerRadius=UDim.new(0,12)
speedBox.FocusLost:Connect(function()
	local v=tonumber(speedBox.Text)
	if v and v>=16 and v<=300 then speedValue=v end
end)

-- ===============================
-- OPÇÕES MISC
-- ===============================
option(miscFrame,"ATUALIZAÇÕES","Link do Discord copiado",
	function() return false end,
	function()
		setclipboard("https://discord.gg/JCX69qqeeR")
		game.StarterGui:SetCore("SendNotification", {
			Title="FACT 4";
			Text="Link do Discord copiado!";
			Duration=4;
		})
	end
)

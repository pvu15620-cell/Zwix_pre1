local p, ts, rs, uis, camera = game.Players.LocalPlayer, game:GetService("TweenService"), game:GetService("RunService"), game:GetService("UserInputService"), workspace.CurrentCamera
local gui = Instance.new("ScreenGui", p:WaitForChild("PlayerGui")) gui.Name, gui.ResetOnSpawn = "KING", false

local function mk(cls, parent, props)
	local inst = Instance.new(cls)
	for k, v in pairs(props or {}) do inst[k] = v end
	inst.Parent = parent
	return inst
end

-- Toggle Button & Dragging
local btn = mk("ImageButton", gui, {Size = UDim2.new(0,52,0,52), Position = UDim2.new(0.02,0,0.1,0), BackgroundColor3 = Color3.fromRGB(35,35,42), Image = "rbxassetid://6023426915"})
mk("UICorner", btn, {CornerRadius = UDim.new(1,0)}) mk("UIStroke", btn, {Color = Color3.fromRGB(255,204,0), Thickness = 2.5})

local dragging, dragInput, dragStart, startPos
btn.InputBegan:Connect(function(i)
	if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
		dragging, dragStart, startPos = true, i.Position, btn.Position
		i.Changed:Connect(function() if i.UserInputState == Enum.UserInputState.End then dragging = false end end)
	end
end)
btn.InputChanged:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then dragInput = i end end)
uis.InputChanged:Connect(function(i)
	if i == dragInput and dragging then
		local d = i.Position - dragStart
		btn.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
	end
end)

-- UI Frame
local f = mk("Frame", gui, {Size = UDim2.new(0,270,0,280), Position = UDim2.new(0.02,0,0.2,0), BackgroundColor3 = Color3.fromRGB(20,20,25), Visible = false, Active = true, Draggable = true})
mk("UICorner", f, {CornerRadius = UDim.new(0,16)}) mk("UIStroke", f, {Color = Color3.fromRGB(70,70,90), Thickness = 2})
mk("TextLabel", f, {Size = UDim2.new(1,0,0,35), BackgroundTransparency = 1, Text = "KINGNGOCTHAI156", TextColor3 = Color3.fromRGB(255,215,0), TextSize = 16, Font = Enum.Font.GothamBold})

local function mkBtn(txt, y)
	local b = mk("TextButton", f, {Size = UDim2.new(0.88,0,0,30), Position = UDim2.new(0.06,0,y,0), BackgroundColor3 = Color3.fromRGB(220,60,60), Text = txt, TextColor3 = Color3.new(1,1,1), Font = Enum.Font.GothamBold, TextSize = 11})
	mk("UICorner", b, {CornerRadius = UDim.new(0,8)})
	return b
end

local box = mk("TextBox", f, {Size = UDim2.new(0.88,0,0,30), Position = UDim2.new(0.06,0,0.15,0), BackgroundColor3 = Color3.fromRGB(30,30,38), TextColor3 = Color3.new(1,1,1), Font = Enum.Font.Gotham, Text = "32", PlaceholderText = "Speed..."})
mk("UICorner", box, {CornerRadius = UDim.new(0,8)})
local sBtn, espBtn, hpBtn, crystalBtn = mkBtn("SPEED: OFF", 0.32), mkBtn("ESP PLAYER: OFF", 0.49), mkBtn("RPG STATS: OFF", 0.66), mkBtn("CRYSTAL ESP: OFF", 0.83)

local open = false
btn.MouseButton1Click:Connect(function()
	if not dragging then open = not open; f.Visible = open; ts:Create(btn, TweenInfo.new(0.2), {Rotation = open and 15 or 0}):Play() end
end)

-- SPEED
local sOn, speed = false, 32
local function getH() return (p.Character or p.CharacterAdded:Wait()):WaitForChild("Humanoid") end
box.FocusLost:Connect(function() speed = tonumber(box.Text) or speed; box.Text = tostring(speed); if sOn then getH().WalkSpeed = speed end end)
sBtn.MouseButton1Click:Connect(function()
	sOn = not sOn; getH().WalkSpeed = sOn and speed or 16
	sBtn.Text, sBtn.BackgroundColor3 = sOn and "SPEED: ON" or "SPEED: OFF", sOn and Color3.fromRGB(46,204,113) or Color3.fromRGB(220,60,60)
end)
p.CharacterAdded:Connect(function() if sOn then getH().WalkSpeed = speed end end)

-- ESP PLAYER
local espOn = false
local function updateESP()
	for _, v in pairs(game.Players:GetPlayers()) do
		if v ~= p and v.Character then
			local hl = v.Character:FindFirstChild("KingESP")
			if espOn and not hl then mk("Highlight", v.Character, {Name = "KingESP", FillColor = Color3.fromRGB(255,0,0), OutlineColor = Color3.new(1,1,1), FillTransparency = 0.5})
			elseif not espOn and hl then hl:Destroy() end
		end
	end
end
espBtn.MouseButton1Click:Connect(function()
	espOn = not espOn; espBtn.Text, espBtn.BackgroundColor3 = espOn and "ESP PLAYER: ON" or "ESP PLAYER: OFF", espOn and Color3.fromRGB(46,204,113) or Color3.fromRGB(220,60,60); updateESP()
end)

-- RPG STATS ESP
local hpOn, hpStore = false, {}
local function createHPUI(v)
	if hpStore[v] then return hpStore[v] end
	local holder = mk("Frame", gui, {Size = UDim2.new(0,110,0,35), BackgroundTransparency = 1, Visible = false})
	local nameTxt = mk("TextLabel", holder, {Size = UDim2.new(1,0,0,12), BackgroundTransparency = 1, TextColor3 = Color3.new(1,1,1), TextStrokeTransparency = 0.2, Font = Enum.Font.GothamBold, TextSize = 10})
	local function mkBar(y, h, col)
		local bg = mk("Frame", holder, {Size = UDim2.new(1,0,0,h), Position = UDim2.new(0,0,0,y), BackgroundColor3 = Color3.fromRGB(20,20,20), BorderSizePixel = 0})
		mk("UICorner", bg, {CornerRadius = UDim.new(0,3)})
		local fill = mk("Frame", bg, {Size = UDim2.new(1,0,1,0), BackgroundColor3 = col, BorderSizePixel = 0})
		mk("UICorner", fill, {CornerRadius = UDim.new(0,3)})
		return fill
	end
	local hpFill, enFill = mkBar(13, 6, Color3.fromRGB(46,204,113)), mkBar(21, 4, Color3.fromRGB(52,152,219))
	local statTxt = mk("TextLabel", holder, {Size = UDim2.new(1,0,0,10), Position = UDim2.new(0,0,0,26), BackgroundTransparency = 1, TextColor3 = Color3.fromRGB(200,200,200), TextStrokeTransparency = 0.4, Font = Enum.Font.Gotham, TextSize = 8})
	hpStore[v] = {Holder = holder, NameTxt = nameTxt, HPFill = hpFill, ENFill = enFill, StatTxt = statTxt}
	return hpStore[v]
end
local function removeHPUI(v) if hpStore[v] then hpStore[v].Holder:Destroy(); hpStore[v] = nil end end

hpBtn.MouseButton1Click:Connect(function()
	hpOn = not hpOn; hpBtn.Text, hpBtn.BackgroundColor3 = hpOn and "RPG STATS: ON" or "RPG STATS: OFF", hpOn and Color3.fromRGB(46,204,113) or Color3.fromRGB(220,60,60)
	if not hpOn then for _, v in pairs(game.Players:GetPlayers()) do removeHPUI(v) end end
end)

-- CRYSTAL ESP
local crystalEspOn, crystalFolder = false, Instance.new("Folder", gui)
local function clearCrystalESP()
	crystalFolder:ClearAllChildren()
	for _, v in pairs(workspace:GetDescendants()) do if v:IsA("Highlight") and v.Name == "CrystalHighlight" then v:Destroy() end end
end

local function applyCrystalESP(target)
	local part = target:IsA("BasePart") and target or target:FindFirstChildWhichIsA("BasePart")
	if not part then return end
	if not target:FindFirstChild("CrystalHighlight") then
		mk("Highlight", target, {Name = "CrystalHighlight", FillColor = Color3.fromRGB(0,255,255), OutlineColor = Color3.new(1,1,1), FillTransparency = 0.3, DepthMode = Enum.HighlightDepthMode.AlwaysOnTop})
	end
	local bb = Instance.new("BillboardGui", crystalFolder) bb.Name, bb.AlwaysOnTop, bb.Size, bb.Adornee = "CrystalTag", true, UDim2.new(0,120,0,30), part
	local txt = mk("TextLabel", bb, {Size = UDim2.new(1,0,1,0), BackgroundTransparency = 1, TextColor3 = Color3.fromRGB(0,255,255), TextStrokeTransparency = 0.2, Font = Enum.Font.GothamBold, TextSize = 12, Text = target.Name})
	task.spawn(function()
		while crystalEspOn and bb and bb.Parent and part and part.Parent do
			local hrp = p.Character and p.Character:FindFirstChild("HumanoidRootPart")
			if hrp then txt.Text = "💎 " .. target.Name .. " [" .. math.floor((hrp.Position - part.Position).Magnitude) .. "m]" end
			task.wait(0.2)
		end
	end)
end

local function scanCrystals()
	clearCrystalESP()
	if not crystalEspOn then return end
	local kw = {"crystal", "pha le", "phale", "gem", "ore", "shard", "stone"}
	for _, obj in pairs(workspace:GetDescendants()) do
		if obj:IsA("BasePart") or obj:IsA("Model") then
			local n = obj.Name:lower()
			for _, k in ipairs(kw) do
				if string.find(n, k) and (obj:IsA("Model") or (obj:IsA("BasePart") and not obj.Parent:IsA("Model"))) then applyCrystalESP(obj) break end
			end
		end
	end
end

crystalBtn.MouseButton1Click:Connect(function()
	crystalEspOn = not crystalEspOn; crystalBtn.Text, crystalBtn.BackgroundColor3 = crystalEspOn and "CRYSTAL ESP: ON" or "CRYSTAL ESP: OFF", crystalEspOn and Color3.fromRGB(46,204,113) or Color3.fromRGB(220,60,60); scanCrystals()
end)

task.spawn(function() while true do task.wait(5) if crystalEspOn then scanCrystals() end end end)

-- RENDER LOOP
rs.RenderStepped:Connect(function()
	if espOn then updateESP() end
	if not hpOn then return end
	for _, v in pairs(game.Players:GetPlayers()) do
		if v ~= p and v.Character then
			local char, hum = v.Character, v.Character:FindFirstChildOfClass("Humanoid")
			local head = char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart")
			if head and hum and hum.Health > 0 then
				local ui = createHPUI(v)
				local vector, onScreen = camera:WorldToViewportPoint(head.Position + Vector3.new(0, 2.5, 0))
				if onScreen then
					ui.Holder.Visible, ui.Holder.Position = true, UDim2.new(0, vector.X - 55, 0, vector.Y - 18)
					local hp, maxHp = math.max(0, math.floor(hum.Health)), math.floor(hum.MaxHealth)
					local hpRatio = math.clamp(hp / (maxHp > 0 and maxHp or 100), 0, 1)
					ui.HPFill.Size = UDim2.new(hpRatio, 0, 1, 0)
					ui.HPFill.BackgroundColor3 = hpRatio <= 0.25 and Color3.fromRGB(231,76,60) or (hpRatio <= 0.5 and Color3.fromRGB(241,196,15) or Color3.fromRGB(46,204,113))

					local stats = char:FindFirstChild("Stats") or char:FindFirstChild("Data") or v:FindFirstChild("Data") or v:FindFirstChild("leaderstats")
					local enStat = char:FindFirstChild("Energy") or (stats and stats:FindFirstChild("Energy"))
					local curEn = enStat and enStat.Value or 100
					ui.ENFill.Size = UDim2.new(math.clamp(curEn / 100, 0, 1), 0, 1, 0)

					local dist = p.Character and p.Character:FindFirstChild("HumanoidRootPart") and math.floor((p.Character.HumanoidRootPart.Position - head.Position).Magnitude) or 0
					ui.NameTxt.Text = v.Name .. " [" .. dist .. "m]"
					ui.StatTxt.Text = "HP: " .. hp .. " | EN: " .. math.floor(curEn)
				else ui.Holder.Visible = false end
			else removeHPUI(v) end
		else removeHPUI(v) end
	end
end)

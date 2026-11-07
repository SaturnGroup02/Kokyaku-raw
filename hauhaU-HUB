-- ============================================================================
-- はーU-HUB (Rayfield / Amethyst / 日本語)
-- 機能: 縦ハート / 羽V2 / アンチ機能(全て)
-- ============================================================================

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local SoundService = game:GetService("SoundService")

local LocalPlayer = Players.LocalPlayer

-- ============================================================================
-- 音声
-- ============================================================================
local ACTION_SOUND_ID = "rbxassetid://137505070991597"
local ERROR_SOUND_ID  = "rbxassetid://82845990304289"

local function PlaySound(soundId, volume)
    pcall(function()
        local sound = Instance.new("Sound")
        sound.SoundId = soundId
        sound.Volume = volume or 1.0
        sound.Parent = SoundService
        if not sound.IsLoaded then sound.Loaded:Wait() end
        sound:Play()
        local conn
        conn = sound.Ended:Connect(function()
            if conn then conn:Disconnect() end
            sound:Destroy()
        end)
        task.delay(6, function()
            if sound and sound.Parent then sound:Destroy() end
        end)
    end)
end

local function PlayActionSound() PlaySound(ACTION_SOUND_ID, 2.5) end
local function PlayErrorSound()  PlaySound(ERROR_SOUND_ID, 5.0) end

local function notify(title, content, duration)
    pcall(function()
        Rayfield:Notify({
            Title = title,
            Content = content,
            Duration = duration or 3,
            Image = "rbxassetid://4483362458"
        })
    end)
end

-- ============================================================================
-- オブジェクトID設定
-- ============================================================================
local ObjectIDConfig = {
    CurrentObjectID = "FireworkSparkler",
    AvailableObjects = {
        "FireworkSparkler", "PalletLightBrown", "GlassBoxGray", "MusicKeyboard",
        "SpookyCandle1", "Tetracube1", "NinjaKatana", "YouDecoy", "BombMissile",
        "LadderLightBrown", "CreatureBlobman", "DiscoColorBall", "MineralCrystalPink",
        "FloatingIsland", "FlyingToyUfo", "JukeboxBlue", "JukeboxOrange", "CouchBlue",
        "CouchPink", "CouchWhite", "Boombox", "FireworkMissile", "Snowflake",
        "SpookyCandle5", "MineralDiamond", "TractorGreen", "TractorOrange", "TractorRed"
    }
}

-- ============================================================================
-- 共通関数
-- ============================================================================
local function findObjects()
    local toys = {}
    for _, item in ipairs(workspace:GetDescendants()) do
        if item:IsA("Model") and item.Name == ObjectIDConfig.CurrentObjectID then
            local alreadyAdded = false
            for _, existingToy in ipairs(toys) do
                if existingToy == item then alreadyAdded = true; break end
            end
            if not alreadyAdded then table.insert(toys, item) end
        end
    end
    return toys
end

local function getPrimaryPart(model)
    if model.PrimaryPart then return model.PrimaryPart end
    local potentialParts = {"Handle","Main","Part","Base","Sparkler","Firework","Blade","Candle","Keyboard","Box","Decoy","Missile","Ladder","Blob","Ball","Crystal","Island","Ufo","Jukebox","Couch","Boombox","Snowflake","Diamond","Tractor"}
    for _, partName in ipairs(potentialParts) do
        local part = model:FindFirstChild(partName)
        if part and part:IsA("BasePart") then return part end
    end
    for _, child in ipairs(model:GetChildren()) do
        if child:IsA("BasePart") then return child end
    end
    return nil
end

local function attachPhysics(part, pValue, dValue)
    if not part then return nil, nil end
    local existingBG = part:FindFirstChildOfClass("BodyGyro")
    local existingBP = part:FindFirstChildOfClass("BodyPosition")
    if existingBG and existingBP then return existingBG, existingBP end
    if existingBG then existingBG:Destroy() end
    if existingBP then existingBP:Destroy() end
    local BP = Instance.new("BodyPosition")
    local BG = Instance.new("BodyGyro")
    BP.P = pValue or 15000; BP.D = dValue or 200
    BP.MaxForce = Vector3.new(1,1,1) * 1e10; BP.Parent = part
    BG.P = pValue or 15000; BG.D = dValue or 200
    BG.MaxTorque = Vector3.new(1,1,1) * 1e10; BG.Parent = part
    return BG, BP
end

-- ============================================================================
-- 設定
-- ============================================================================
local WingConfig = {
    Enabled = false, VerticalOffset = 2.0, Spread = 5.0, ObjectCount = 10,
    FlapShape = 2.0, FlapSpeed = 1.0, FlapAmount = 3.0, ChainDelay = 0.01
}

local VerticalHeartConfig = {
    Enabled = false, Height = 5.0, Size = 5.0, ObjectCount = 12,
    RotationSpeed = 1.0, PulseSpeed = 2.0, PulseAmplitude = 0.5, FollowPlayer = true
}

-- ============================================================================
-- 状態変数
-- ============================================================================
local wingList = {}
local wingLoopConn = nil
local wingTime = 0

local verticalHeartToys = {}
local verticalHeartPoints = {}
local verticalHeartAssignedToys = {}
local verticalHeartLoopConn = nil
local verticalHeartTime = 0

-- ============================================================================
-- ウィンドウ作成
-- ============================================================================
local Window = Rayfield:CreateWindow({
    Name = "はーU-HUB",
    LoadingTitle = "はーU-HUB",
    LoadingSubtitle = "読み込み中...",
    Theme = "Amethyst",
    ConfigurationSaving = {
        Enabled = false,
        FolderName = nil,
        FileName = "HaUHUB"
    },
    KeySystem = false
})

-- ============================================================================
-- 翼機能
-- ============================================================================
local function toggleWing(state)
    WingConfig.Enabled = state
    if state then
        PlayActionSound()
        if VerticalHeartConfig.Enabled then toggleVerticalHeart(false) end
        wingList = {}
        local maxObjects = WingConfig.ObjectCount * 2
        local foundCount = 0
        for _, d in ipairs(Workspace:GetDescendants()) do
            if foundCount >= maxObjects then break end
            if d:IsA("Model") and d.Name == ObjectIDConfig.CurrentObjectID then
                local part = getPrimaryPart(d)
                if part and not part.Anchored then
                    for _, p in ipairs(d:GetDescendants()) do
                        if p:IsA("BasePart") then
                            pcall(function() p:SetNetworkOwner(LocalPlayer) end)
                            p.CanCollide = false
                            p.CanTouch = false
                        end
                    end
                    if not part:FindFirstChild("WingBodyVelocity") then
                        local bv = Instance.new("BodyVelocity")
                        bv.Name = "WingBodyVelocity"
                        bv.MaxForce = Vector3.new(1e8,1e8,1e8)
                        bv.Velocity = Vector3.new()
                        bv.P = 1e6
                        bv.Parent = part
                    end
                    if not part:FindFirstChild("WingBodyGyro") then
                        local bg = Instance.new("BodyGyro")
                        bg.Name = "WingBodyGyro"
                        bg.MaxTorque = Vector3.new(1e8,1e8,1e8)
                        bg.CFrame = part.CFrame
                        bg.P = 1e6
                        bg.Parent = part
                    end
                    table.insert(wingList, {
                        model = d, part = part, index = foundCount + 1,
                        targetPos = part.Position, targetCF = part.CFrame
                    })
                    foundCount = foundCount + 1
                end
            end
        end
        if foundCount == 0 then
            PlayErrorSound()
            notify("エラー", "オブジェクトが見つかりません")
            WingConfig.Enabled = false
            return
        end
        wingTime = 0
        if wingLoopConn then wingLoopConn:Disconnect() end
        local function getWingPosition(index, total, time)
            local halfTotal = total / 2
            local isLeftWing = index <= halfTotal
            local wingIndex = isLeftWing and index or (index - halfTotal)
            local t = (wingIndex - 1) / (halfTotal - 1)
            local phase = (time * WingConfig.FlapSpeed - wingIndex * 0.05) * WingConfig.FlapShape
            local sinValue = math.sin(phase)
            local actualFlapAmount = sinValue > 0 and WingConfig.FlapAmount * 0.6 or WingConfig.FlapAmount
            local flapAngle = sinValue * math.rad(actualFlapAmount)
            local baseX = t * WingConfig.Spread
            local rotatedY = baseX * math.sin(flapAngle)
            local rotatedX = baseX * math.cos(flapAngle)
            local sideOffset = isLeftWing and -(3 + rotatedX) or (3 + rotatedX)
            return Vector3.new(sideOffset, WingConfig.VerticalOffset + rotatedY, 0), isLeftWing, wingIndex
        end
        wingLoopConn = RunService.Heartbeat:Connect(function(dt)
            if not WingConfig.Enabled then return end
            local root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if not root or #wingList == 0 then return end
            wingTime = wingTime + dt
            local rootVelocity = root.AssemblyLinearVelocity or root.Velocity or Vector3.new()
            local total = #wingList
            for i, rec in ipairs(wingList) do
                local part = rec.part
                if not part or not part.Parent then continue end
                local localPos, _, wingIndex = getWingPosition(i, total, wingTime)
                local targetCF = root.CFrame
                local idealPos = targetCF.Position + (targetCF - targetCF.Position):VectorToWorldSpace(localPos)
                local idealRotation = targetCF * CFrame.Angles(0, -math.pi/2, 0)
                local delayMultiplier = 1 + (wingIndex - 1) * 2
                local actualDelay = WingConfig.ChainDelay * delayMultiplier
                local delayFactor = math.min(1, dt / actualDelay)
                rec.targetPos = rec.targetPos:Lerp(idealPos, delayFactor)
                rec.targetCF = rec.targetCF:Lerp(idealRotation, delayFactor)
                local targetPos = rec.targetPos
                local targetRot = rec.targetCF
                local dir = targetPos - part.Position
                local distance = dir.Magnitude
                local bv = part:FindFirstChild("WingBodyVelocity")
                if bv then
                    if distance > 0.1 then
                        local moveVelocity = dir.Unit * math.min(3000, distance * 50)
                        bv.Velocity = moveVelocity + rootVelocity
                    else
                        bv.Velocity = rootVelocity
                    end
                    bv.P = 1e6
                end
                local bg = part:FindFirstChild("WingBodyGyro")
                if bg then
                    bg.CFrame = targetRot
                    bg.P = 1e6
                end
            end
        end)
        notify("翼機能", "開始 / オブジェクト数: " .. #wingList)
    else
        if wingLoopConn then wingLoopConn:Disconnect(); wingLoopConn = nil end
        for _, rec in ipairs(wingList) do
            if rec.part then
                local bv = rec.part:FindFirstChild("WingBodyVelocity"); if bv then bv:Destroy() end
                local bg = rec.part:FindFirstChild("WingBodyGyro"); if bg then bg:Destroy() end
                if rec.model then
                    for _, p in ipairs(rec.model:GetDescendants()) do
                        if p:IsA("BasePart") then
                            p.CanCollide = true
                            p.CanTouch = true
                            pcall(function() p:SetNetworkOwner(nil) end)
                        end
                    end
                end
            end
        end
        wingList = {}
        notify("翼機能", "停止しました")
    end
end

-- ============================================================================
-- 縦ハート機能
-- ============================================================================
local function toggleVerticalHeart(state)
    VerticalHeartConfig.Enabled = state
    if state then
        PlayActionSound()
        if WingConfig.Enabled then toggleWing(false) end
        verticalHeartToys = findObjects()
        if #verticalHeartToys == 0 then
            PlayErrorSound()
            notify("エラー", "オブジェクトが見つかりません")
            VerticalHeartConfig.Enabled = false
            return
        end
        local count = math.min(#verticalHeartToys, VerticalHeartConfig.ObjectCount)
        verticalHeartPoints = {}
        for i = 1, count do
            local part = Instance.new("Part")
            part.CanCollide = false; part.Anchored = true; part.Transparency = 1
            part.Size = Vector3.new(4,1,4); part.Parent = workspace
            verticalHeartPoints[i] = { angle = (i-1)*(2*math.pi/count), part = part, assignedToy = nil }
        end
        verticalHeartAssignedToys = {}
        for i = 1, math.min(#verticalHeartToys, #verticalHeartPoints) do
            local toy = verticalHeartToys[i]
            if toy and toy:IsA("Model") and toy.Name == ObjectIDConfig.CurrentObjectID then
                local primaryPart = getPrimaryPart(toy)
                if primaryPart then
                    for _, child in ipairs(toy:GetChildren()) do
                        if child:IsA("BasePart") then
                            child.CanCollide = false; child.CanTouch = false; child.Anchored = false
                        end
                    end
                    local BG, BP = attachPhysics(primaryPart)
                    local toyTable = { BG = BG, BP = BP, Pallet = primaryPart, Model = toy, PointIndex = i, baseAngle = verticalHeartPoints[i].angle }
                    verticalHeartPoints[i].assignedToy = toyTable
                    table.insert(verticalHeartAssignedToys, toyTable)
                end
            end
        end
        verticalHeartTime = 0
        if verticalHeartLoopConn then verticalHeartLoopConn:Disconnect() end
        verticalHeartLoopConn = RunService.RenderStepped:Connect(function(dt)
            if not VerticalHeartConfig.Enabled or not LocalPlayer.Character then return end
            local torso = LocalPlayer.Character:FindFirstChild("Torso") or LocalPlayer.Character:FindFirstChild("UpperTorso")
            if not torso then return end
            verticalHeartTime = verticalHeartTime + dt
            local basePosition = torso.Position
            local pulseEffect = (VerticalHeartConfig.PulseSpeed > 0) and math.sin(verticalHeartTime * VerticalHeartConfig.PulseSpeed) * VerticalHeartConfig.PulseAmplitude or 0
            for _, point in ipairs(verticalHeartPoints) do
                if point.assignedToy and point.assignedToy.BP and point.assignedToy.BG then
                    local toy = point.assignedToy
                    local currentAngle = toy.baseAngle + (verticalHeartTime * VerticalHeartConfig.RotationSpeed)
                    local baseScale = VerticalHeartConfig.Size / 20
                    local x = 16 * (math.sin(currentAngle) ^ 3) * baseScale
                    local y = (13*math.cos(currentAngle) - 5*math.cos(2*currentAngle) - 2*math.cos(3*currentAngle) - math.cos(4*currentAngle)) * baseScale
                    if pulseEffect > 0 then
                        local pf = 1 + (pulseEffect * 0.1)
                        x = x * pf; y = y * pf
                    end
                    local targetPosition = basePosition + Vector3.new(x, VerticalHeartConfig.Height + y, 0)
                    if point.part then point.part.Position = targetPosition end
                    toy.BP.Position = targetPosition
                    toy.BG.CFrame = CFrame.new(targetPosition) * CFrame.Angles(0,0,0)
                end
            end
        end)
        notify("縦ハート", "開始 / サイズ: " .. VerticalHeartConfig.Size)
    else
        if verticalHeartLoopConn then verticalHeartLoopConn:Disconnect(); verticalHeartLoopConn = nil end
        for _, point in ipairs(verticalHeartPoints) do
            if point.part then point.part:Destroy() end
            if point.assignedToy then
                if point.assignedToy.BG then point.assignedToy.BG:Destroy() end
                if point.assignedToy.BP then point.assignedToy.BP:Destroy() end
            end
        end
        verticalHeartPoints = {}; verticalHeartAssignedToys = {}
        notify("縦ハート", "停止しました")
    end
end

-- ============================================================================
-- オブジェクトID変更
-- ============================================================================
local function changeObjectID(id)
    if id == ObjectIDConfig.CurrentObjectID then return end
    PlayActionSound()
    local wasWing = WingConfig.Enabled
    local wasHeart = VerticalHeartConfig.Enabled
    if wasWing then toggleWing(false) end
    if wasHeart then toggleVerticalHeart(false) end
    ObjectIDConfig.CurrentObjectID = id
    task.wait(0.4)
    if wasWing then toggleWing(true) end
    if wasHeart then toggleVerticalHeart(true) end
    notify("オブジェクトID変更", id .. " に切り替えました")
end

-- ============================================================================
-- アンチ機能準備
-- ============================================================================
local RS = ReplicatedStorage
local GE = RS:FindFirstChild("GrabEvents") or RS:WaitForChild("GrabEvents", 5)
local SetNetworkOwner = GE and GE:FindFirstChild("SetNetworkOwner")
local DestroyGrabLine = GE and GE:FindFirstChild("DestroyGrabLine")
local CharacterEvents = RS:FindFirstChild("CharacterEvents")
local StruggleEvent = CharacterEvents and CharacterEvents:FindFirstChild("Struggle")
local RagdollRemote  = CharacterEvents and CharacterEvents:FindFirstChild("RagdollRemote")
local MenuToys = RS:FindFirstChild("MenuToys")
local SpawnToyRemote = MenuToys and MenuToys:FindFirstChild("SpawnToyRemoteFunction")
local DestroyToy = MenuToys and MenuToys:FindFirstChild("DestroyToy")
local isHeldValue = LocalPlayer:FindFirstChild("IsHeld") or LocalPlayer:WaitForChild("IsHeld", 5)

-- ============================================================================
-- タブ1: オブジェクトの種類を選択
-- ============================================================================
local ObjectTab = Window:CreateTab("オブジェクトの種類を選択", 4483362458)

ObjectTab:CreateSection("現在のオブジェクト")
local currentObjLabel = ObjectTab:CreateLabel("現在: " .. ObjectIDConfig.CurrentObjectID)

ObjectTab:CreateSection("オブジェクトを選択")

ObjectTab:CreateDropdown({
    Name = "オブジェクトID",
    Options = ObjectIDConfig.AvailableObjects,
    CurrentOption = {ObjectIDConfig.CurrentObjectID},
    MultipleOptions = false,
    Flag = "ObjectIDSelect",
    Callback = function(option)
        local id = type(option) == "table" and option[1] or option
        if id and id ~= "" then
            changeObjectID(id)
            pcall(function() currentObjLabel:Set("現在: " .. id) end)
        end
    end
})

ObjectTab:CreateButton({
    Name = "現在のオブジェクトIDを表示",
    Callback = function()
        PlayActionSound()
        notify("現在のオブジェクト", ObjectIDConfig.CurrentObjectID, 3)
    end
})

-- ============================================================================
-- タブ2: 翼機能
-- ============================================================================
local WingTab = Window:CreateTab("翼機能", 4483362458)

WingTab:CreateSection("翼機能 - 基本")
WingTab:CreateToggle({
    Name = "翼機能を起動",
    CurrentValue = false,
    Flag = "WingEnabled",
    Callback = function(v)
        toggleWing(v)
    end
})

WingTab:CreateSection("翼設定")
WingTab:CreateSlider({
    Name = "翼の高さ位置",
    Range = {-10, 20},
    Increment = 0.5,
    Suffix = "スタッド",
    CurrentValue = WingConfig.VerticalOffset,
    Flag = "WingHeight",
    Callback = function(v) WingConfig.VerticalOffset = v end
})
WingTab:CreateSlider({
    Name = "翼の広がり",
    Range = {3, 30},
    Increment = 1,
    Suffix = "スタッド",
    CurrentValue = WingConfig.Spread,
    Flag = "WingSpread",
    Callback = function(v) WingConfig.Spread = v end
})
WingTab:CreateSlider({
    Name = "羽ばたき形状",
    Range = {0.5, 10},
    Increment = 0.5,
    Suffix = "形状",
    CurrentValue = WingConfig.FlapShape,
    Flag = "WingFlapShape",
    Callback = function(v) WingConfig.FlapShape = v end
})
WingTab:CreateSlider({
    Name = "羽ばたく速さ",
    Range = {0.1, 5},
    Increment = 0.1,
    Suffix = "速度",
    CurrentValue = WingConfig.FlapSpeed,
    Flag = "WingFlapSpeed",
    Callback = function(v) WingConfig.FlapSpeed = v end
})
WingTab:CreateSlider({
    Name = "羽ばたく可動域",
    Range = {0, 100},
    Increment = 1,
    Suffix = "度",
    CurrentValue = WingConfig.FlapAmount,
    Flag = "WingFlapAmount",
    Callback = function(v) WingConfig.FlapAmount = v end
})
WingTab:CreateSlider({
    Name = "片翼オブジェクト数",
    Range = {3, 30},
    Increment = 1,
    Suffix = "個",
    CurrentValue = WingConfig.ObjectCount,
    Flag = "WingObjCount",
    Callback = function(v) WingConfig.ObjectCount = v end
})
WingTab:CreateSlider({
    Name = "チェーン遅延",
    Range = {0.001, 0.1},
    Increment = 0.001,
    Suffix = "秒",
    CurrentValue = WingConfig.ChainDelay,
    Flag = "WingChainDelay",
    Callback = function(v) WingConfig.ChainDelay = v end
})

-- ============================================================================
-- タブ3: ハート機能
-- ============================================================================
local HeartTab = Window:CreateTab("ハート機能", 4483362458)

HeartTab:CreateSection("縦ハート - 基本")
HeartTab:CreateToggle({
    Name = "縦ハートを起動",
    CurrentValue = false,
    Flag = "VerticalHeartEnabled",
    Callback = function(v)
        toggleVerticalHeart(v)
    end
})

HeartTab:CreateSection("ハート設定")
HeartTab:CreateSlider({
    Name = "ハートサイズ",
    Range = {2, 50},
    Increment = 1,
    Suffix = "スタッド",
    CurrentValue = VerticalHeartConfig.Size,
    Flag = "VHSize",
    Callback = function(v) VerticalHeartConfig.Size = v end
})
HeartTab:CreateSlider({
    Name = "基本高さ",
    Range = {0, 50},
    Increment = 1,
    Suffix = "スタッド",
    CurrentValue = VerticalHeartConfig.Height,
    Flag = "VHHeight",
    Callback = function(v) VerticalHeartConfig.Height = v end
})
HeartTab:CreateSlider({
    Name = "オブジェクト数",
    Range = {6, 100},
    Increment = 2,
    Suffix = "個",
    CurrentValue = VerticalHeartConfig.ObjectCount,
    Flag = "VHObjCount",
    Callback = function(v) VerticalHeartConfig.ObjectCount = v end
})
HeartTab:CreateSlider({
    Name = "回転速度",
    Range = {0, 10},
    Increment = 0.1,
    Suffix = "速度",
    CurrentValue = VerticalHeartConfig.RotationSpeed,
    Flag = "VHRotSpeed",
    Callback = function(v) VerticalHeartConfig.RotationSpeed = v end
})
HeartTab:CreateSlider({
    Name = "脈動速度",
    Range = {0, 10},
    Increment = 0.1,
    Suffix = "速度",
    CurrentValue = VerticalHeartConfig.PulseSpeed,
    Flag = "VHPulseSpeed",
    Callback = function(v) VerticalHeartConfig.PulseSpeed = v end
})
HeartTab:CreateSlider({
    Name = "脈動振幅",
    Range = {0, 10},
    Increment = 0.1,
    Suffix = "スタッド",
    CurrentValue = VerticalHeartConfig.PulseAmplitude,
    Flag = "VHPulseAmp",
    Callback = function(v) VerticalHeartConfig.PulseAmplitude = v end
})

-- ============================================================================
-- タブ4: アンチタブ
-- ============================================================================
local AntiTab = Window:CreateTab("アンチタブ", 4483362458)

-- Anti Grab シンプル
AntiTab:CreateSection("掴まれても脱出 (シンプル)")
local antiGrabSimpleEnabled = false

AntiTab:CreateToggle({
    Name = "掴まれても脱出 (シンプル)",
    CurrentValue = false,
    Flag = "AntiGrabSimple",
    Callback = function(Value)
        antiGrabSimpleEnabled = Value
        if Value then PlayActionSound() end
    end
})

RunService.Heartbeat:Connect(function()
    if not antiGrabSimpleEnabled then return end
    local character = LocalPlayer.Character
    if not character then return end
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    if isHeldValue and isHeldValue.Value == true then
        pcall(function()
            if StruggleEvent then StruggleEvent:FireServer(LocalPlayer) end
        end)
        hrp.Velocity = Vector3.new()
        hrp.Anchored = true
    else
        hrp.Anchored = false
    end
end)

-- Anti Grab V2
AntiTab:CreateSection("Anti Grab V2 (強化)")
local antiGrabV2Enabled = false
local heldConnectionV2 = nil

local function StopAntiGrabV2()
    antiGrabV2Enabled = false
    if heldConnectionV2 then heldConnectionV2:Disconnect(); heldConnectionV2 = nil end
    local Root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if Root then Root.Anchored = false end
end

local function StartAntiGrabV2()
    if heldConnectionV2 then heldConnectionV2:Disconnect() end
    local isHeld = LocalPlayer:FindFirstChild("IsHeld")
    if not isHeld then return end
    heldConnectionV2 = isHeld:GetPropertyChangedSignal("Value"):Connect(function()
        if not antiGrabV2Enabled or not isHeld.Value then return end
        local Char = LocalPlayer.Character
        if not Char then return end
        local Root = Char:FindFirstChild("HumanoidRootPart")
        local Hum = Char:FindFirstChildOfClass("Humanoid")
        if not (Root and Hum) then return end
        for _, part in pairs(Char:GetDescendants()) do
            if part:IsA("BasePart") then part.CanCollide = false end
        end
        task.spawn(function()
            while antiGrabV2Enabled and isHeld.Value do
                pcall(function()
                    if StruggleEvent then StruggleEvent:FireServer() end
                    if RagdollRemote then RagdollRemote:FireServer(Root, 0) end
                    if SetNetworkOwner then SetNetworkOwner:FireServer(Root, Root.CFrame) end
                end)
                task.wait(0.03)
            end
        end)
        task.spawn(function()
            while antiGrabV2Enabled and isHeld.Value do
                pcall(function()
                    Hum.Sit = false
                    Hum:ChangeState(Enum.HumanoidStateType.GettingUp)
                    Hum:SetStateEnabled(Enum.HumanoidStateType.Jumping, true)
                    if Root then
                        Root.Anchored = true
                        Root.AssemblyLinearVelocity = Vector3.zero
                        Root.AssemblyAngularVelocity = Vector3.zero
                    end
                end)
                task.wait(0.03)
            end
            if Root then Root.Anchored = false end
        end)
    end)
    if isHeld.Value then
        local Char = LocalPlayer.Character
        local Root = Char and Char:FindFirstChild("HumanoidRootPart")
        if Root then Root.Anchored = true end
        task.spawn(function()
            while antiGrabV2Enabled and isHeld.Value do
                if StruggleEvent then StruggleEvent:FireServer() end
                task.wait(0.05)
            end
            if Root then Root.Anchored = false end
        end)
    end
end

AntiTab:CreateToggle({
    Name = "Anti Grab V2 (強化)",
    CurrentValue = false,
    Flag = "AntiGrabV2",
    Callback = function(Value)
        antiGrabV2Enabled = Value
        if Value then
            PlayActionSound()
            StartAntiGrabV2()
            notify("Anti Grab V2", "有効化されました")
        else
            StopAntiGrabV2()
            notify("Anti Grab V2", "無効化されました")
        end
    end
})

LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    if antiGrabV2Enabled then StartAntiGrabV2() end
end)

-- Anti Gucci
AntiTab:CreateSection("Anti Gucci")
local autoGucciActive = false
local autoGucciConn = nil
local autoGucciSpamTask = nil

local function cleanupGucciTasks()
    if autoGucciSpamTask then
        task.cancel(autoGucciSpamTask)
        autoGucciSpamTask = nil
    end
end

local function SpawnGucciToy(toyName, hrp)
    if not SpawnToyRemote then return nil end
    local inv = Workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
    if not inv then return nil end
    local spawnCF = hrp.CFrame * CFrame.new(0, 14, 20)
    task.spawn(function()
        pcall(function() SpawnToyRemote:InvokeServer(toyName, spawnCF, Vector3.zero) end)
    end)
    local t = tick()
    local spawnedToy = nil
    repeat
        task.wait(0.1)
        spawnedToy = inv:FindFirstChild(toyName)
    until spawnedToy or (tick() - t > 3)
    return spawnedToy
end

local function StartGucci()
    if autoGucciConn then autoGucciConn:Disconnect() end
    local GucciThing = nil
    local isActive = false
    local function gucci()
        if not autoGucciActive then return end
        isActive = true
        local char = LocalPlayer.Character
        if not char then isActive = false return end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChild("Humanoid")
        local inv = Workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
        if not (hrp and hum and inv) then isActive = false return end
        cleanupGucciTasks()
        if DestroyToy then
            for _, v in pairs(inv:GetChildren()) do
                if v.Name == "AutoGucci" or v.Name == "TractorGreen" then
                    pcall(function() DestroyToy:FireServer(v) end)
                end
            end
        end
        local isHeld = LocalPlayer:FindFirstChild("IsHeld")
        while isHeld and isHeld.Value do task.wait() end
        hum.Sit = false
        task.wait(0.1)
        GucciThing = SpawnGucciToy("TractorGreen", hrp)
        while not GucciThing and autoGucciActive do
            task.wait(0.25)
            GucciThing = SpawnGucciToy("TractorGreen", hrp)
        end
        if not GucciThing then isActive = false return end
        GucciThing.Name = "AutoGucci"
        local seat = GucciThing:FindFirstChild("VehicleSeat")
        if not seat then isActive = false return end
        autoGucciSpamTask = task.spawn(function()
            while isActive and autoGucciActive do
                if RagdollRemote then pcall(function() RagdollRemote:FireServer(hrp, 0) end) end
                if StruggleEvent then pcall(function() StruggleEvent:FireServer(LocalPlayer) end) end
                task.wait(0.05)
            end
        end)
        local t0 = tick()
        while not hum.SeatPart and autoGucciActive and tick() - t0 < 3 do
            seat:Sit(hum)
            task.wait(0.1)
        end
        hum.Sit = false
        hum:ChangeState(Enum.HumanoidStateType.Jumping)
        hrp.Anchored = true
        task.spawn(function()
            task.wait(0.3)
            GucciThing:PivotTo(CFrame.new(0, 1e6, 0))
            for _, p in pairs(GucciThing:GetDescendants()) do
                if p:IsA("BasePart") then p.Anchored = true end
            end
        end)
        hrp.Anchored = false
        isActive = false
    end
    gucci()
    autoGucciConn = RunService.Heartbeat:Connect(function()
        if not autoGucciActive then return end
        if isActive then return end
        local char = LocalPlayer.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChild("Humanoid")
        local isHeld = LocalPlayer:FindFirstChild("IsHeld")
        if not (hrp and hum) or hum.Health <= 0 then
            isActive = true
            LocalPlayer.CharacterAdded:Wait()
            task.wait(0.5)
            gucci()
            return
        end
        if (isHeld and isHeld.Value) or hum.Sit then gucci() end
    end)
end

AntiTab:CreateToggle({
    Name = "Anti Gucci",
    CurrentValue = false,
    Flag = "AntiGucci",
    Callback = function(Value)
        autoGucciActive = Value
        if Value then
            PlayActionSound()
            StartGucci()
            notify("Anti Gucci", "有効化されました")
        else
            cleanupGucciTasks()
            if autoGucciConn then autoGucciConn:Disconnect(); autoGucciConn = nil end
            local inv = Workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
            if inv and DestroyToy then
                for _, v in pairs(inv:GetChildren()) do
                    if v.Name == "AutoGucci" or v.Name == "TractorGreen" then
                        pcall(function() DestroyToy:FireServer(v) end)
                    end
                end
            end
            notify("Anti Gucci", "無効化されました")
        end
    end
})

-- Anti Void
AntiTab:CreateSection("Anti Void")
local originalFallenHeight = Workspace.FallenPartsDestroyHeight

AntiTab:CreateToggle({
    Name = "Anti Void (落下保護)",
    CurrentValue = false,
    Flag = "AntiVoid",
    Callback = function(Value)
        if Value then
            PlayActionSound()
            originalFallenHeight = Workspace.FallenPartsDestroyHeight
            Workspace.FallenPartsDestroyHeight = 0/0
            notify("Anti Void", "有効化されました")
        else
            Workspace.FallenPartsDestroyHeight = originalFallenHeight
            notify("Anti Void", "無効化されました")
        end
    end
})

-- ============================================================================
-- 起動完了通知
-- ============================================================================
task.wait(1)
PlayActionSound()
notify("はーU-HUB", "起動完了しました", 5)

print("==========================================")
print("はーU-HUB 起動完了 / Rayfield Amethyst")
print("==========================================")

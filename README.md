-- 加载 WindUI
local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")

-- 创建主窗口，启用UI自带卡密系统
local Window = WindUI:CreateWindow({
    Title = "mmmkill",
    Icon = "home",
    Author = "mmmkill",
    Folder = "脚本置",
    Size = UDim2.fromOffset(580, 460),
    Theme = "Dark",
    Resizable = true,
    ToggleKey = Enum.KeyCode.RightShift,
    -- ========== WindUI自带卡密配置 ==========
    KeySystem = {
        Key = {"mmmkillNB"}, -- 有效卡密
        Note = "去找作者要卡密", -- 弹窗显示文字
        SaveKey = false, -- 不保存卡密，每次打开脚本都要输入
    }
})

-- ====================== 公告标签页 ======================
local AnnouncementTab = Window:Tab({
    Title = "公告",
    Icon = "bell",
})

local sectionObject

local function DetectInjector()
    local detected = "未知注入器"
    if pcall(getgenv) then
        if getgenv().Delta then detected = "Delta"
        elseif getgenv().Fluxus then detected = "Fluxus"
        elseif getgenv().ScriptWare then detected = "Script‑Ware"
        end
    end
    return detected
end

local function FormatTime(sec)
    local d = math.floor(sec / 86400)
    local h = math.floor((sec % 86400)/3600)
    local m = math.floor((sec % 3600)/60)
    local s = math.floor(sec % 60)
    local t = {}
    if d>0 then table.insert(t,d.."天") end
    if h>0 then table.insert(t,h.."小时") end
    if m>0 then table.insert(t,m.."分钟") end
    table.insert(t,s.."秒")
    return table.concat(t," ")
end

local function UpdateAnnounce()
    pcall(function()
        if sectionObject and sectionObject.Destroy then sectionObject:Destroy() end
        local inj = DetectInjector()
        local serverAge = FormatTime(os.clock())
        sectionObject = AnnouncementTab:Section({
            Title = "重要公告",
            Content = string.format([[
🔌 注入器：%s
⏱ 脚本运行时长：%s
📌 本脚本学习用途，遇到强服务器检测会失效
        ]],inj,serverAge)
        })
    end)
end

UpdateAnnounce()

AnnouncementTab:Button({
    Title = "刷新公告",
    Icon = "refresh",
    Callback = function()
        UpdateAnnounce()
        WindUI:Notify({Title="提示",Content="公告已刷新",Duration=2})
    end
})

-- ====================== 通用标签页 ======================
local GeneralTab = Window:Tab({
    Title = "通用",
    Icon = "settings"
})

local character,humanoid,rootPart
local function RefreshCharacter()
    pcall(function()
        character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        humanoid = character:WaitForChild("Humanoid")
        rootPart = character:WaitForChild("HumanoidRootPart")
    end)
end
RefreshCharacter()
LocalPlayer.CharacterAdded:Connect(function()
    RefreshCharacter()
    -- 复活后 如果开启锁定速度，立刻重新赋值
    if lockWalkSpeed and humanoid then
        humanoid.WalkSpeed = savedWalkSpeed
    end
end)

local speedEnabled = false
local speedValue = 50
local speedThread

-- 新增锁定速度变量
local lockWalkSpeed = false
local savedWalkSpeed = 16

local function SpeedLoop()
    local UIS = game:GetService("UserInputService")
    local cam = workspace.CurrentCamera
    while speedEnabled and task.wait(0) do
        if not (humanoid and rootPart) then RefreshCharacter() end
        local dir = Vector3.new()
        if UIS:IsKeyDown(Enum.KeyCode.W) then dir += cam.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.S) then dir -= cam.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.A) then dir -= cam.CFrame.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.D) then dir += cam.CFrame.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.new(0,1,0) end
        if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then dir -= Vector3.new(0,1,0) end
        rootPart.Velocity = dir * speedValue
    end
end

GeneralTab:Toggle({
    Title = "锁定速度【死后复活保留】",
    Value = false,
    Callback = function(state)
        lockWalkSpeed = state
        if state then
            savedWalkSpeed = humanoid and humanoid.WalkSpeed or 16
            WindUI:Notify({Title="锁定速度",Content="已开启，当前保存速度："..tostring(savedWalkSpeed),Duration=2})
        else
            WindUI:Notify({Title="锁定速度",Content="已关闭",Duration=2})
        end
    end
})

GeneralTab:Toggle({
    Title = "速度",
    Value = false,
    Callback = function(state)
        speedEnabled = state
        if state then
            humanoid.GravityScale = 0
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping,false)
            speedThread = task.spawn(SpeedLoop)
        else
            task.cancel(speedThread)
            humanoid.GravityScale = 1
            humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping,true)
        end
        -- 同步更新快捷栏按钮文字
        if QuickBarBtn then
            QuickBarBtn.Text = speedEnabled and "速度 [开]" or "速度 [关]"
        end
    end
})

GeneralTab:Slider({
    Title = "移动速度",
    Value = {Min=16,Max=200,Default=16},
    Callback = function(v)
        pcall(function()
            if humanoid then
                humanoid.WalkSpeed = v
                -- 如果锁定开启，同步保存新数值
                if lockWalkSpeed then
                    savedWalkSpeed = v
                end
            end
        end)
    end
})
GeneralTab:Slider({
    Title = "跳跃力",
    Value = {Min=50,Max=350,Default=50},
    Callback = function(v)
        pcall(function()
            if humanoid then humanoid.JumpPower = v end
        end)
    end
})
GeneralTab:Slider({
    Title = "速度倍率",
    Value = {Min=20,Max=300,Default=50},
    Callback = function(v) speedValue = v end
})
GeneralTab:Slider({
    Title = "人物重力",
    Value = {Min=0,Max=3,Default=1},
    Callback = function(v)
        pcall(function()
            if humanoid then humanoid.GravityScale = v end
        end)
    end
})

GeneralTab:Button({
    Title = "重置人物全部属性",
    Icon = "rotate‑cw",
    Callback = function()
        pcall(function()
            if humanoid then
                humanoid.WalkSpeed =16
                humanoid.JumpPower=50
                humanoid.GravityScale=1
            end
            speedEnabled=false
            task.cancel(speedThread)
            WindUI:Notify({Title="通用",Content="属性恢复默认",Duration=2})
            if QuickBarBtn then
                QuickBarBtn.Text = "速度 [关]"
            end
        end)
    end
})

-- ====================== 快捷栏悬浮UI逻辑 ======================
local QuickBarEnabled = false
local QuickBarLocked = false
local QuickFrame, QuickBarBtn
local dragStart, startPos
local inputChangedConn, inputEndedConn

local function DestroyQuickBar()
    if inputChangedConn then inputChangedConn:Disconnect() end
    if inputEndedConn then inputEndedConn:Disconnect() end
    local gui = game.CoreGui:FindFirstChild("QuickBarGui")
    if gui then gui:Destroy() end
    QuickFrame = nil
    QuickBarBtn = nil
    dragStart = nil
end

local function CreateQuickBarUI()
    DestroyQuickBar()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "QuickBarGui"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = game.CoreGui

    QuickFrame = Instance.new("Frame")
    QuickFrame.Name = "QuickBarFrame"
    QuickFrame.Size = UDim2.new(0,130,0,60)
    QuickFrame.Position = UDim2.new(0.1,0,0.3,0)
    QuickFrame.BackgroundColor3 = Color3.new(0.12,0.12,0.12)
    QuickFrame.BorderSizePixel = 2
    QuickFrame.BorderColor3 = Color3.new(0.4,0.4,0.4)
    QuickFrame.Parent = screenGui

    local DragBar = Instance.new("TextLabel")
    DragBar.Name = "DragBar"
    DragBar.Size = UDim2.new(1,0,0,22)
    DragBar.BackgroundColor3 = Color3.new(0.25,0.25,0.25)
    DragBar.Text = "快捷栏(可拖动)"
    DragBar.Font = Enum.Font.SourceSansBold
    DragBar.TextSize = 12
    DragBar.TextColor3 = Color3.new(1,1,1)
    DragBar.Parent = QuickFrame

    QuickBarBtn = Instance.new("TextButton")
    QuickBarBtn.Name = "SpeedBtn"
    QuickBarBtn.Size = UDim2.new(0.9,0,0,28)
    QuickBarBtn.Position = UDim2.new(0.05,0,0.35,0)
    QuickBarBtn.BackgroundColor3 = Color3.new(0.2,0.35,0.2)
    QuickBarBtn.Text = speedEnabled and "速度 [开]" or "速度 [关]"
    QuickBarBtn.Font = Enum.Font.SourceSansBold
    QuickBarBtn.TextSize = 14
    QuickBarBtn.TextColor3 = Color3.new(1,1,1)
    QuickBarBtn.Parent = QuickFrame

    QuickBarBtn.MouseButton1Click:Connect(function()
        pcall(function()
            speedEnabled = not speedEnabled
            if speedEnabled then
                humanoid.GravityScale = 0
                humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping,false)
                speedThread = task.spawn(SpeedLoop)
                QuickBarBtn.Text = "速度 [开]"
            else
                task.cancel(speedThread)
                humanoid.GravityScale = 1
                humanoid:SetStateEnabled(Enum.HumanoidStateType.Jumping,true)
                QuickBarBtn.Text = "速度 [关]"
            end
        end)
    end)

    DragBar.InputBegan:Connect(function(input)
        if QuickBarLocked then return end
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragStart = input.Position
            startPos = QuickFrame.Position
        end
    end)

    inputChangedConn = UserInputService.InputChanged:Connect(function(input)
        if QuickBarLocked or not dragStart then return end
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            QuickFrame.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y
            )
        end
    end)

    inputEndedConn = UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragStart = nil
        end
    end)
end

-- 快捷栏总开关
GeneralTab:Toggle({
    Title = "快捷栏",
    Value = 

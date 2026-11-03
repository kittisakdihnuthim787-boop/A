-- Roblox MM2 Script - Safe Fly Coin Farm & Role Reveal (Yellow UI)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")
local VirtualUser = game:GetService("VirtualUser")

local LocalPlayer = Players.LocalPlayer

-- Variables
local AutoPlayEnabled = false
local FlySpeed = 25 -- ความเร็วในการบินเก็บเหรียญ (ยิ่งน้อยยิ่งเนียนและปลอดภัย)
local CurrentTween = nil

-- UI Creation
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MM2_FlyYellow_UI"
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

-- Mini Button (ปุ่มย่อ/ขยาย UI)
local MiniButton = Instance.new("TextButton")
MiniButton.Name = "MiniButton"
MiniButton.Size = UDim2.new(0, 45, 0, 45)
MiniButton.Position = UDim2.new(0, 15, 0.4, 0)
MiniButton.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
MiniButton.TextColor3 = Color3.fromRGB(0, 0, 0)
MiniButton.Text = "🎮"
MiniButton.TextSize = 22
MiniButton.Active = true
MiniButton.Draggable = true
MiniButton.Parent = ScreenGui

local MiniCorner = Instance.new("UICorner")
MiniCorner.CornerRadius = UDim.new(0, 10)
MiniCorner.Parent = MiniButton

-- Main Frame
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 180, 0, 190)
MainFrame.Position = UDim2.new(0.5, -90, 0.4, -95)
MainFrame.BackgroundColor3 = Color3.fromRGB(255, 235, 100)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Visible = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame

-- Title
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundColor3 = Color3.fromRGB(240, 195, 0)
Title.TextColor3 = Color3.fromRGB(30, 30, 30)
Title.Text = "🎮 MM2 Auto Menu"
Title.TextSize = 14
Title.Font = Enum.Font.SourceSansBold
Title.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 12)
TitleCorner.Parent = Title

-- Role Box (แสดงบทบาท)
local RoleBox = Instance.new("TextLabel")
RoleBox.Size = UDim2.new(0.88, 0, 0, 60)
RoleBox.Position = UDim2.new(0.06, 0, 0.2, 0)
RoleBox.BackgroundColor3 = Color3.fromRGB(255, 255, 235)
RoleBox.TextColor3 = Color3.fromRGB(20, 20, 20)
RoleBox.Text = "🔪: ค้นหา...\n🤠: ค้นหา..."
RoleBox.TextSize = 13
RoleBox.Font = Enum.Font.SourceSansBold
RoleBox.Parent = MainFrame

local RoleCorner = Instance.new("UICorner")
RoleCorner.CornerRadius = UDim.new(0, 6)
RoleCorner.Parent = RoleBox

-- Auto Play Button
local AutoBtn = Instance.new("TextButton")
AutoBtn.Size = UDim2.new(0.88, 0, 0, 40)
AutoBtn.Position = UDim2.new(0.06, 0, 0.65, 0)
AutoBtn.BackgroundColor3 = Color3.fromRGB(235, 75, 75)
AutoBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
AutoBtn.Text = "เล่นอัตโนมัติ: OFF"
AutoBtn.TextSize = 14
AutoBtn.Font = Enum.Font.SourceSansBold
AutoBtn.Parent = MainFrame

local BtnCorner = Instance.new("UICorner")
BtnCorner.CornerRadius = UDim.new(0, 8)
BtnCorner.Parent = AutoBtn

-- UI Interaction
MiniButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

AutoBtn.MouseButton1Click:Connect(function()
    AutoPlayEnabled = not AutoPlayEnabled
    if AutoPlayEnabled then
        AutoBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 75)
        AutoBtn.Text = "เล่นอัตโนมัติ: ON"
    else
        AutoBtn.BackgroundColor3 = Color3.fromRGB(235, 75, 75)
        AutoBtn.Text = "เล่นอัตโนมัติ: OFF"
        if CurrentTween then CurrentTween:Cancel() end
    end
end)

-- Helper Functions
local function GetRoles()
    local murderer, sheriff = nil, nil
    for _, player in pairs(Players:GetPlayers()) do
        if player.Character then
            if player.Backpack:FindFirstChild("Knife") or player.Character:FindFirstChild("Knife") then
                murderer = player
            elseif player.Backpack:FindFirstChild("Gun") or player.Character:FindFirstChild("Gun") then
                sheriff = player
            end
        end
    end
    return murderer, sheriff
end

local function EquipTool(toolName)
    local backpack = LocalPlayer:FindFirstChild("Backpack")
    local character = LocalPlayer.Character
    if backpack and character then
        local tool = backpack:FindFirstChild(toolName)
        if tool then
            tool.Parent = character
        end
    end
end

local function ClickScreen()
    pcall(function()
        VirtualUser:Button1Down(Vector2.new(0, 0))
        task.wait(0.05)
        VirtualUser:Button1Up(Vector2.new(0, 0))
    end)
end

-- ฟังก์ชันบินไปหาเป้าหมาย (Fly / Smooth Tween)
local function FlyToTarget(targetCFrame)
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local distance = (root.Position - targetCFrame.Position).Magnitude
    local timeToReach = distance / FlySpeed

    local tweenInfo = TweenInfo.new(
        timeToReach,
        Enum.EasingStyle.Linear,
        Enum.EasingDirection.Out
    )

    CurrentTween = TweenService:Create(root, tweenInfo, {CFrame = targetCFrame})
    CurrentTween:Play()
    CurrentTween.Completed:Wait()
end

-- Role Reveal Loop
RunService.Heartbeat:Connect(function()
    local murderer, sheriff = GetRoles()
    local mName = murderer and murderer.DisplayName or "-"
    local sName = sheriff and sheriff.DisplayName or "-"
    RoleBox.Text = "🔪 ฆาตกร: " .. mName .. "\n🤠 นายอำเภอ: " .. sName
end)

-- Main Logic Loop
task.spawn(function()
    while true do
        task.wait(0.1)
        if AutoPlayEnabled then
            pcall(function()
                local char = LocalPlayer.Character
                local root = char and char:FindFirstChild("HumanoidRootPart")
                if not root then return end

                local murderer, sheriff = GetRoles()
                local isMyMurder = char:FindFirstChild("Knife") or LocalPlayer.Backpack:FindFirstChild("Knife")
                local isMySheriff = char:FindFirstChild("Gun") or LocalPlayer.Backpack:FindFirstChild("Gun")

                if isMyMurder then
                    -- 🔪 ฆาตกร (บินไปฆ่าทีละคน)
                    EquipTool("Knife")
                    for _, v in pairs(Players:GetPlayers()) do
                        if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
                            local targetRoot = v.Character.HumanoidRootPart
                            local targetHum = v.Character:FindFirstChildOfClass("Humanoid")
                            if targetHum and targetHum.Health > 0 and AutoPlayEnabled then
                                FlyToTarget(targetRoot.CFrame * CFrame.new(0, 0, 2))
                                task.wait(0.1)
                                ClickScreen()
                                task.wait(0.3)
                            end
                        end
                    end

                elseif isMySheriff then
                    -- 🤠 นายอำเภอ (บินไปยิงฆาตกร)
                    EquipTool("Gun")
                    if murderer and murderer.Character and murderer.Character:FindFirstChild("HumanoidRootPart") then
                        local mRoot = murderer.Character.HumanoidRootPart
                        FlyToTarget(mRoot.CFrame * CFrame.new(0, 0, 4))
                        root.CFrame = CFrame.new(root.Position, mRoot.Position)
                        ClickScreen()
                        task.wait(0.5)
                    end

                else
                    -- 😇 คนดี (Innocent)
                    -- 1. บินไปเก็บปืนเมื่อนายอำเภอตาย
                    local droppedGun = Workspace:FindFirstChild("GunDrop", true)
                    if droppedGun and droppedGun:IsA("BasePart") then
                        FlyToTarget(droppedGun.CFrame + Vector3.new(0, 2, 0))
                        task.wait(0.2)
                        return
                    end

                    -- 2. ถือปืนแล้ว ล็อคยิงฆาตกร
                    if char:FindFirstChild("Gun") then
                        EquipTool("Gun")
                        if murderer and murderer.Character and murderer.Character:FindFirstChild("HumanoidRootPart") then
                            local mRoot = murderer.Character.HumanoidRootPart
                            FlyToTarget(mRoot.CFrame * CFrame.new(0, 0, 4))
                            root.CFrame = CFrame.new(root.Position, mRoot.Position)
                            ClickScreen()
                        end
                        return
                    end

                    -- 3. ระบบลอยตัวบินเก็บเหรียญอัตโนมัติ
                    local closestCoin = nil
                    local shortestDistance = math.huge

                    for _, obj in pairs(Workspace:GetDescendants()) do
                        if (obj.Name == "Coin" or obj.Name == "CoinVisual" or obj.Name:match("Coin")) and obj:IsA("BasePart") then
                            if obj.Transparency < 1 then
                                local dist = (root.Position - obj.Position).Magnitude
                                if dist < shortestDistance then
                                    shortestDistance = dist
                                    closestCoin = obj
                                end
                            end
                        end
                    end

                    -- บินไปเก็บเหรียญที่อยู่ใกล้ที่สุด
                    if closestCoin and AutoPlayEnabled then
                        FlyToTarget(closestCoin.CFrame + Vector3.new(0, 1.5, 0))
                        task.wait(0.1)
                    end
                end
            end)
        end
    end
end)


-- Ollie Bench Build: Blox Fruits Full Hub (Draggable UI, Auto-Farm, Auto-Stats, Teleport, ESP)
-- Environment: Client Executor / Roblox Studio LocalScript

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local VirtualUser = game:GetService("VirtualUser")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local CoreGui = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer

-- Tránh tạo trùng UI khi chạy lại
if CoreGui:FindFirstChild("OllieBenchHub") then
    CoreGui:FindFirstChild("OllieBenchHub"):Destroy()
end

-- Cấu hình trạng thái toàn cục
getgenv().OllieConfig = {
    AutoFarm = false,
    FastAttack = false,
    AutoClicker = false,
    BringMobs = false,
    SelectedWeapon = "Melee",
    FarmDistance = 15,
    AutoStats = {
        Melee = false,
        Defense = false,
        Sword = false,
        Gun = false,
        Fruit = false
    },
    PlayerESP = false,
    ChestESP = false,
    FruitESP = false,
    WalkSpeed = 16,
    JumpPower = 50,
    NoClip = false
}

-- Anti-AFK tự động
LocalPlayer.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new())
end)

-- ==========================================
-- HỆ THỐNG GIAO DIỆN (UI LIBRARY TỰ DỰNG)
-- ==========================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "OllieBenchHub"
-- Rơi về PlayerGui nếu executor không hỗ trợ CoreGui
local success, _ = pcall(function() ScreenGui.Parent = CoreGui end)
if not success then ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 520, 0, 340)
MainFrame.Position = UDim2.new(0.5, -260, 0.5, -170)
MainFrame.BackgroundColor3 = Color3.fromRGB(22, 22, 26)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

local TopBar = Instance.new("Frame")
TopBar.Size = UDim2.new(1, 0, 0, 35)
TopBar.BackgroundColor3 = Color3.fromRGB(30, 30, 36)
TopBar.BorderSizePixel = 0
TopBar.Parent = MainFrame

local TopCorner = Instance.new("UICorner")
TopCorner.CornerRadius = UDim.new(0, 8)
TopCorner.Parent = TopBar

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, -40, 1, 0)
TitleLabel.Position = UDim2.new(0, 12, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "OLLIE BENCH // BLOX FRUITS"
TitleLabel.TextColor3 = Color3.fromRGB(255, 190, 60)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 14
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TopBar

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 35, 0, 35)
CloseBtn.Position = UDim2.new(1, -35, 0, 0)
CloseBtn.BackgroundTransparency = 1
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(220, 70, 70)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 15
CloseBtn.Parent = TopBar
CloseBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- Nút thu gọn nổi trên màn hình để bật/tắt menu
local ToggleUIButton = Instance.new("TextButton")
ToggleUIButton.Size = UDim2.new(0, 50, 0, 50)
ToggleUIButton.Position = UDim2.new(0, 20, 0.5, -25)
ToggleUIButton.BackgroundColor3 = Color3.fromRGB(30, 30, 36)
ToggleUIButton.Text = "[O]"
ToggleUIButton.TextColor3 = Color3.fromRGB(255, 190, 60)
ToggleUIButton.Font = Enum.Font.GothamBold
ToggleUIButton.TextSize = 16
ToggleUIButton.Active = true
ToggleUIButton.Draggable = true
ToggleUIButton.Parent = ScreenGui
Instance.new("UICorner", ToggleUIButton).CornerRadius = UDim.new(0, 25)
ToggleUIButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- Chia cột Tab và Nội dung
local TabHolder = Instance.new("Frame")
TabHolder.Size = UDim2.new(0, 130, 1, -45)
TabHolder.Position = UDim2.new(0, 8, 0, 40)
TabHolder.BackgroundColor3 = Color3.fromRGB(26, 26, 32)
TabHolder.BorderSizePixel = 0
TabHolder.Parent = MainFrame
Instance.new("UICorner", TabHolder).CornerRadius = UDim.new(0, 6)

local TabListLayout = Instance.new("UIListLayout")
TabListLayout.Padding = UDim.new(0, 5)
TabListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
TabListLayout.Parent = TabHolder

local TabPadding = Instance.new("UIPadding")
TabPadding.PaddingTop = UDim.new(0, 8)
TabPadding.Parent = TabHolder

local PageContainer = Instance.new("Frame")
PageContainer.Size = UDim2.new(1, -152, 1, -45)
PageContainer.Position = UDim2.new(0, 144, 0, 40)
PageContainer.BackgroundColor3 = Color3.fromRGB(26, 26, 32)
PageContainer.BorderSizePixel = 0
PageContainer.Parent = MainFrame
Instance.new("UICorner", PageContainer).CornerRadius = UDim.new(0, 6)

local Pages = {}

local function CreateTab(tabName)
    local TabBtn = Instance.new("TextButton")
    TabBtn.Size = UDim2.new(1, -12, 0, 32)
    TabBtn.BackgroundColor3 = Color3.fromRGB(36, 36, 44)
    TabBtn.Text = tabName
    TabBtn.TextColor3 = Color3.fromRGB(210, 210, 210)
    TabBtn.Font = Enum.Font.GothamSemibold
    TabBtn.TextSize = 13
    TabBtn.Parent = TabHolder
    Instance.new("UICorner", TabBtn).CornerRadius = UDim.new(0, 5)

    local Page = Instance.new("ScrollingFrame")
    Page.Size = UDim2.new(1, -12, 1, -12)
    Page.Position = UDim2.new(0, 6, 0, 6)
    Page.BackgroundTransparency = 1
    Page.ScrollBarThickness = 3
    Page.Visible = false
    Page.Parent = PageContainer

    local PageLayout = Instance.new("UIListLayout")
    PageLayout.Padding = UDim.new(0, 6)
    PageLayout.Parent = Page

    PageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        Page.CanvasSize = UDim2.new(0, 0, 0, PageLayout.AbsoluteContentSize.Y + 10)
    end)

    Pages[tabName] = Page

    TabBtn.MouseButton1Click:Connect(function()
        for _, p in pairs(Pages) do p.Visible = false end
        Page.Visible = true
    end)

    return Page
end

local function AddToggle(page, labelText, defaultState, callback)
    local ToggleFrame = Instance.new("Frame")
    ToggleFrame.Size = UDim2.new(1, -6, 0, 34)
    ToggleFrame.BackgroundColor3 = Color3.fromRGB(34, 34, 42)
    ToggleFrame.Parent = page
    Instance.new("UICorner", ToggleFrame).CornerRadius = UDim.new(0, 5)

    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(1, -60, 1, 0)
    Label.Position = UDim2.new(0, 10, 0, 0)
    Label.BackgroundTransparency = 1
    Label.Text = labelText
    Label.TextColor3 = Color3.fromRGB(230, 230, 230)
    Label.Font = Enum.Font.Gotham
    Label.TextSize = 13
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = ToggleFrame

    local StateBtn = Instance.new("TextButton")
    StateBtn.Size = UDim2.new(0, 42, 0, 22)
    StateBtn.Position = UDim2.new(1, -50, 0.5, -11)
    StateBtn.BackgroundColor3 = defaultState and Color3.fromRGB(60, 190, 90) or Color3.fromRGB(70, 70, 80)
    StateBtn.Text = defaultState and "ON" or "OFF"
    StateBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    StateBtn.Font = Enum.Font.GothamBold
    StateBtn.TextSize = 11
    StateBtn.Parent = ToggleFrame
    Instance.new("UICorner", StateBtn).CornerRadius = UDim.new(0, 4)

    local toggled = defaultState
    StateBtn.MouseButton1Click:Connect(function()
        toggled = not toggled
        StateBtn.BackgroundColor3 = toggled and Color3.fromRGB(60, 190, 90) or Color3.fromRGB(70, 70, 80)
        StateBtn.Text = toggled and "ON" or "OFF"
        callback(toggled)
    end)
end

local function AddButton(page, buttonText, callback)
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(1, -6, 0, 32)
    Btn.BackgroundColor3 = Color3.fromRGB(48, 48, 60)
    Btn.Text = buttonText
    Btn.TextColor3 = Color3.fromRGB(255, 200, 80)
    Btn.Font = Enum.Font.GothamSemibold
    Btn.TextSize = 13
    Btn.Parent = page
    Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 5)

    Btn.MouseButton1Click:Connect(function()
        callback()
    end)
end

-- Khởi tạo các Tab
local FarmTab = CreateTab("Auto Farm")
local StatsTab = CreateTab("Auto Stats")
local TeleTab = CreateTab("Teleport")
local EspTab = CreateTab("ESP & Misc")
FarmTab.Visible = true

-- ==========================================
-- CHỨC NĂNG CỐT LÕI (CORE LOGIC)
-- ==========================================

-- 1. Hàm Di chuyển mượt (Tween Bypass)
local currentTween = nil
local function TweenTo(targetCFrame)
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local hrp = char.HumanoidRootPart
    local distance = (hrp.Position - targetCFrame.Position).Magnitude
    local speed = 300 -- Tốc độ bay chuẩn hạn chế giật lag回
    local tweenInfo = TweenInfo.new(distance / speed, Enum.EasingStyle.Linear)
    
    if currentTween then currentTween:Cancel() end
    currentTween = TweenService:Create(hrp, tweenInfo, {CFrame = targetCFrame})
    currentTween:Play()
end

-- 2. Tự động trang bị vũ khí theo loại
local function EquipSelectedWeapon()
    local char = LocalPlayer.Character
    if not char then return end
    for _, item in pairs(LocalPlayer.Backpack:GetChildren()) do
        if item:IsA("Tool") and (item.ToolTip == getgenv().OllieConfig.SelectedWeapon or item.Name:find(getgenv().OllieConfig.SelectedWeapon)) then
            char.Humanoid:EquipTool(item)
            break
        end
    end
end

-- 3. Tìm quái gần nhất còn sống trong khu vực
local function GetClosestEnemy()
    local enemiesFolder = Workspace:FindFirstChild("Enemies")
    if not enemiesFolder then return nil end

    local closest = nil
    local shortestDist = math.huge
    local myPos = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character.HumanoidRootPart.Position
    if not myPos then return nil end

    for _, mob in pairs(enemiesFolder:GetChildren()) do
        local hum = mob:FindFirstChild("Humanoid")
        local hrp = mob:FindFirstChild("HumanoidRootPart")
        if hum and hrp and hum.Health > 0 then
            local dist = (myPos - hrp.Position).Magnitude
            if dist < shortestDist then
                shortestDist = dist
                closest = mob
            end
        end
    end
    return closest
end

-- 4. Gom quái xung quanh (Bring Mobs)
local function BringNearbyMobs(targetMob)
    local enemiesFolder = Workspace:FindFirstChild("Enemies")
    if not enemiesFolder or not targetMob or not targetMob:FindFirstChild("HumanoidRootPart") then return end
    local targetCFrame = targetMob.HumanoidRootPart.CFrame

    for _, mob in pairs(enemiesFolder:GetChildren()) do
        if mob ~= targetMob and mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 and mob:FindFirstChild("HumanoidRootPart") then
            local dist = (mob.HumanoidRootPart.Position - targetCFrame.Position).Magnitude
            if dist <= 250 then
                mob.HumanoidRootPart.CFrame = targetCFrame
                mob.HumanoidRootPart.CanCollide = false
                mob.Humanoid.WalkSpeed = 0
                mob.Humanoid.JumpPower = 0
            end
        end
    end
end

-- ==========================================
-- GẮN CÁC NÚT ĐIỀU KHIỂN VÀO TỪNG TAB
-- ==========================================

-- TAB: AUTO FARM
AddToggle(FarmTab, "Auto Farm Quái Gần Nhất", false, function(state)
    getgenv().OllieConfig.AutoFarm = state
    if not state and currentTween then currentTween:Cancel() end
end)

AddToggle(FarmTab, "Gom Quái (Bring Mobs)", false, function(state)
    getgenv().OllieConfig.BringMobs = state
end)

AddToggle(FarmTab, "Auto Click / Fast Attack", false, function(state)
    getgenv().OllieConfig.AutoClicker = state
end)

AddButton(FarmTab, "Đổi Vũ Khí: Melee / Sword / Fruit", function()
    local order = {"Melee", "Sword", "Blox Fruit"}
    local currentIdx = table.find(order, getgenv().OllieConfig.SelectedWeapon) or 1
    local nextIdx = (currentIdx % #order) + 1
    getgenv().OllieConfig.SelectedWeapon = order[nextIdx]
    TitleLabel.Text = "OLLIE BENCH // VŨ KHÍ: " .. getgenv().OllieConfig.SelectedWeapon
end)

-- TAB: AUTO STATS
local statTypes = {
    {Label = "Tự Nâng Melee (Cận Chiến)", Key = "Melee", RemoteArg = "Melee"},
    {Label = "Tự Nâng Defense (Máu)", Key = "Defense", RemoteArg = "Defense"},
    {Label = "Tự Nâng Sword (Kiếm)", Key = "Sword", RemoteArg = "Sword"},
    {Label = "Tự Nâng Gun (Súng)", Key = "Gun", RemoteArg = "Gun"},
    {Label = "Tự Nâng Blox Fruit (Trái Ác Quỷ)", Key = "Fruit", RemoteArg = "Demon Fruit"}
}

for _, st in ipairs(statTypes) do
    AddToggle(StatsTab, st.Label, false, function(state)
        getgenv().OllieConfig.AutoStats[st.Key] = state
    end)
end

-- TAB: TELEPORT (Tọa độ tiêu biểu Sea 1 / Sea 2 / Sea 3)
local Islands = {
    {Name = "Đảo Khởi Đầu (Starter Island)", Pos = CFrame.new(1071, 16, 1426)},
    {Name = "Đảo Khỉ (Jungle)", Pos = CFrame.new(-1602, 36, 153)},
    {Name = "Thị Trấn Hải Tặc (Pirate Village)", Pos = CFrame.new(-1145, 4, 3828)},
    {Name = "Đảo Tuyết (Frozen Village)", Pos = CFrame.new(1198, 27, -1211)},
    {Name = "Pháo Đài Hải Quân (Marine Fortress)", Pos = CFrame.new(-4505, 20, 4260)},
    {Name = "Đảo Trời (Skylands)", Pos = CFrame.new(-4842, 717, -2623)},
    {Name = "Quán Cafe (Sea 2 - Cafe)", Pos = CFrame.new(-380, 77, 255)},
    {Name = "Dinh Thự (Sea 3 - Mansion)", Pos = CFrame.new(-12550, 337, -7476)}
}

for _, island in ipairs(Islands) do
    AddButton(TeleTab, "Bay Tới: " .. island.Name, function()
        TweenTo(island.Pos)
    end)
end

AddButton(TeleTab, "Dừng Bay Ngay Lập Tức", function()
    if currentTween then currentTween:Cancel() end
end)

-- TAB: ESP & MISC
AddToggle(EspTab, "Định Vị Người Chơi (Player ESP)", false, function(state)
    getgenv().OllieConfig.PlayerESP = state
end)

AddToggle(EspTab, "Định Vị Trái Ác Quỷ (Fruit ESP)", false, function(state)
    getgenv().OllieConfig.FruitESP = state
end)

AddToggle(EspTab, "Đi Xuyên Tường (NoClip)", false, function(state)
    getgenv().OllieConfig.NoClip = state
end)

AddButton(EspTab, "Tăng Tốc Chạy & Nhảy Cao", function()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = 75
        LocalPlayer.Character.Humanoid.JumpPower = 120
    end
end)

-- ==========================================
-- VÒNG LẶP XỬ LÝ RUNTIME (BACKGROUND LOOPS)
-- ==========================================

-- Vòng lặp Auto Farm & NoClip
RunService.Stepped:Connect(function()
    if getgenv().OllieConfig.NoClip or getgenv().OllieConfig.AutoFarm then
        if LocalPlayer.Character then
            for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
                if part:IsA("BasePart") and part.CanCollide then
                    part.CanCollide = false
                end
            end
        end
    end
end)

task.spawn(function()
    while task.wait(0.1) do
        if getgenv().OllieConfig.AutoFarm then
            pcall(function()
                EquipSelectedWeapon()
                local target = GetClosestEnemy()
                if target and target:FindFirstChild("HumanoidRootPart") then
                    -- Bay lên phía trên đầu quái để tránh sát thương thường
                    local farmCFrame = target.HumanoidRootPart.CFrame * CFrame.new(0, getgenv().OllieConfig.FarmDistance, 0) * CFrame.Angles(math.rad(-90), 0, 0)
                    TweenTo(farmCFrame)
                    if getgenv().OllieConfig.BringMobs then
                        BringNearbyMobs(target)
                    end
                end
            end)
        end

        if getgenv().OllieConfig.AutoClicker or getgenv().OllieConfig.AutoFarm then
            pcall(function()
                VirtualUser:CaptureController()
                VirtualUser:Button1Down(Vector2.new(1280, 672))
            end)
        end
    end
end)

-- Vòng lặp Auto Stats (Gọi Remote CommF_ của Blox Fruits)
task.spawn(function()
    while task.wait(0.5) do
        pcall(function()
            local commF = ReplicatedStorage:FindFirstChild("Remotes") and ReplicatedStorage.Remotes:FindFirstChild("CommF_")
            if commF then
                for _, st in ipairs(statTypes) do
                    if getgenv().OllieConfig.AutoStats[st.Key] then
                        commF:InvokeServer("AddPoint", st.RemoteArg, 1)
                    end
                end
            end
        end)
    end
end)

-- Vòng lặp cập nhật ESP
local function UpdateBillboardESP(targetParent, tagName, labelText, color, enabled)
    if not targetParent then return end
    local existing = targetParent:FindFirstChild(tagName)
    if enabled then
        if not existing then
            local bgui = Instance.new("BillboardGui")
            bgui.Name = tagName
            bgui.AlwaysOnTop = true
            bgui.Size = UDim2.new(0, 150, 0, 40)
            bgui.StudsOffset = Vector3.new(0, 3, 0)
            bgui.Parent = targetParent

            local txt = Instance.new("TextLabel")
            txt.Size = UDim2.new(1, 0, 1, 0)
            txt.BackgroundTransparency = 1
            txt.Text = labelText
            txt.TextColor3 = color
            txt.TextStrokeTransparency = 0.2
            txt.Font = Enum.Font.GothamBold
            txt.TextSize = 12
            txt.Parent = bgui
        end
    else
        if existing then existing:Destroy() end
    end
end

task.spawn(function()
    while task.wait(1) do
        -- ESP Người chơi
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
                local dist = 0
                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChild("HumanoidRootPart") then
                    dist = math.floor((LocalPlayer.Character.HumanoidRootPart.Position - plr.Character.HumanoidRootPart.Position).Magnitude)
                end
                UpdateBillboardESP(
                    plr.Character.Head,
                    "OlliePlrESP",
                    string.format("[%s] - %dm", plr.Name, dist),
                    Color3.fromRGB(255, 80, 80),
                    getgenv().OllieConfig.PlayerESP
                )
            end
        end

        -- ESP Trái Ác Quỷ rơi trên map
        for _, obj in pairs(Workspace:GetChildren()) do
            if obj:IsA("Tool") or obj.Name:find("Fruit") then
                local handle = obj:FindFirstChild("Handle")
                if handle then
                    UpdateBillboardESP(
                        handle,
                        "OllieFruitESP",
                        "TRÁI ÁC QUỶ: " .. obj.Name,
                        Color3.fromRGB(80, 255, 120),
                        getgenv().OllieConfig.FruitESP
                    )
                end
            end
        end
    end
end)
# Script-

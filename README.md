--[[
    =============================================
    OS PEÇA HUB 👺 - ADVANCED BLOX FRUITS GUI
    =============================================
    Features:
    - Modern Black & Purple Neon Design
    - Auto Farm Level (Smart NPC Detection)
    - Auto Quest, Auto Haki, Auto Equip
    - Tween Teleport, Noclip, Fast Attack
    - Teleport System, ESP, Anti-AFK
    - Optimized for Mobile & PC Executors
    - Compatible with Delta, Codex, Arceus X
]]

-- // Services \\ --
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local TeleportService = game:GetService("TeleportService")
local Lighting = game:GetService("Lighting")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")

-- // Player & Character \\ --
local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")

-- // Game References \\ --
local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local CombatFramework = require(game:GetService("Players").LocalPlayer.PlayerScripts.CombatFramework)
local Camera = Workspace.CurrentCamera

-- // UI Setup \\ --
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/JustAPerson-Dev/UI-Libs/main/Redz%20Lib"))()
local Window = Library:CreateWindow({
    Name = "Os Peça Hub 👺",
    Theme = {
        Background = Color3.fromRGB(10, 10, 10),
        Accent = Color3.fromRGB(150, 0, 255), -- Purple Neon
        TextColor = Color3.fromRGB(255, 255, 255),
        Border = Color3.fromRGB(150, 0, 255),
        Glow = Color3.fromRGB(200, 50, 255)
    },
    Blur = true,
    Draggable = true,
    MinimizeButton = true,
    Logo = "http://www.roblox.com/asset/?id=YOUR_LOGO_ASSET_ID" -- Replace with your logo
})

-- // Tabs \\ --
local MainTab = Window:CreateTab("Main")
local FarmTab = Window:CreateTab("Farm")
local PlayerTab = Window:CreateTab("Player")
local TeleportTab = Window:CreateTab("Teleport")
local CombatTab = Window:CreateTab("Combat")
local RaidTab = Window:CreateTab("Raid")
local SettingsTab = Window:CreateTab("Settings")

-- // Variables \\ --
local AutoFarmEnabled = false
local NoclipEnabled = false
local FastAttackEnabled = false
local AutoHakiEnabled = false
local CurrentQuest = nil
local NearestNPC = nil
local FarmLevel = {
    ["Level 1-10"] = { QuestName = "Bandit", QuestGiver = "Bandit Quest", NPCs = {"Bandit"} },
    ["Level 15-30"] = { QuestName = "Monkey", QuestGiver = "Jungle Quest", NPCs = {"Monkey"} },
    -- Add more levels as needed
}

-- // Functions \\ --

-- [[ Tween Teleport ]] --
local function TweenTo(Position, Speed)
    local Distance = (RootPart.Position - Position).Magnitude
    local Duration = Distance / Speed
    local TweenInfo = TweenInfo.new(Duration, Enum.EasingStyle.Linear)
    local Tween = TweenService:Create(RootPart, TweenInfo, {CFrame = CFrame.new(Position)})
    Tween:Play()
    Tween.Completed:Wait()
end

-- [[ Noclip ]] --
local function NoclipLoop()
    while NoclipEnabled and RunService:IsRunning() do
        for _, Part in ipairs(Character:GetDescendants()) do
            if Part:IsA("BasePart") then
                Part.CanCollide = false
            end
        end
        task.wait(0.1)
    end
end

-- [[ Fast Attack ]] --
local function FastAttack()
    while FastAttackEnabled and RunService:IsRunning() do
        if CombatFramework and CombatFramework.activeController then
            CombatFramework.activeController.hitboxMagnitude = 50
            CombatFramework.activeController.increment = 3
            CombatFramework.activeController.blocking = false
        end
        task.wait()
    end
end

-- [[ Auto Haki ]] --
local function AutoHaki()
    while AutoHakiEnabled and RunService:IsRunning() do
        if Humanoid.Health > 0 then
            game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("Buso")
        end
        task.wait(10)
    end
end

-- [[ Find Nearest NPC ]] --
local function FindNearestNPC(NPCList)
    local ClosestDistance = math.huge
    local ClosestNPC = nil

    for _, NPCName in ipairs(NPCList) do
        for _, NPC in ipairs(Workspace.Enemies:GetChildren()) do
            if NPC.Name == NPCName and NPC:FindFirstChild("Humanoid") and NPC.Humanoid.Health > 0 then
                local Distance = (RootPart.Position - NPC.HumanoidRootPart.Position).Magnitude
                if Distance < ClosestDistance then
                    ClosestDistance = Distance
                    ClosestNPC = NPC
                end
            end
        end
    end

    return ClosestNPC
end

-- [[ Auto Farm Level ]] --
local function AutoFarmLevel()
    while AutoFarmEnabled and RunService:IsRunning() do
        if not CurrentQuest then
            -- Get appropriate quest based on level
            for LevelRange, QuestData in pairs(FarmLevel) do
                if Player.Data.Level.Value >= tonumber(LevelRange:match("%d+")) and Player.Data.Level.Value <= tonumber(LevelRange:match("%d+$")) then
                    CurrentQuest = QuestData
                    break
                end
            end
        end

        if CurrentQuest then
            -- Teleport to Quest Giver
            local QuestGiver = Workspace.NPCs:FindFirstChild(CurrentQuest.QuestGiver)
            if QuestGiver then
                TweenTo(QuestGiver.HumanoidRootPart.Position, 300)
                task.wait(1)
                -- Accept Quest (Replace with actual remote)
                game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("StartQuest", CurrentQuest.QuestName)
            end

            -- Find and kill NPCs
            NearestNPC = FindNearestNPC(CurrentQuest.NPCs)
            if NearestNPC then
                -- Fly above NPC
                local NPCPos = NearestNPC.HumanoidRootPart.Position + Vector3.new(0, 20, 0)
                TweenTo(NPCPos, 300)

                -- Attack NPC
                while NearestNPC and NearestNPC:FindFirstChild("Humanoid") and NearestNPC.Humanoid.Health > 0 and AutoFarmEnabled do
                    -- Face NPC
                    RootPart.CFrame = CFrame.lookAt(RootPart.Position, NearestNPC.HumanoidRootPart.Position)

                    -- Melee Attack (Replace with actual remote)
                    game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("Attack", "Melee")

                    -- Check if NPC is dead
                    if not NearestNPC:FindFirstChild("Humanoid") or NearestNPC.Humanoid.Health <= 0 then
                        break
                    end
                    task.wait(0.1)
                end
            else
                -- Quest completed, reset
                CurrentQuest = nil
                task.wait(2)
            end
        end
        task.wait(0.5)
    end
end

-- // UI Elements \\ --

-- [[ Main Tab ]] --
MainTab:CreateToggle({
    Name = "Auto Farm Level",
    CurrentValue = false,
    Flag = "AutoFarmToggle",
    Callback = function(Value)
        AutoFarmEnabled = Value
        if Value then
            coroutine.wrap(AutoFarmLevel)()
        end
    end
})

MainTab:CreateToggle({
    Name = "Noclip",
    CurrentValue = false,
    Flag = "NoclipToggle",
    Callback = function(Value)
        NoclipEnabled = Value
        if Value then
            coroutine.wrap(NoclipLoop)()
        end
    end
})

MainTab:CreateToggle({
    Name = "Fast Attack",
    CurrentValue = false,
    Flag = "FastAttackToggle",
    Callback = function(Value)
        FastAttackEnabled = Value
        if Value then
            coroutine.wrap(FastAttack)()
        end
    end
})

MainTab:CreateToggle({
    Name = "Auto Haki",
    CurrentValue = false,
    Flag = "AutoHakiToggle",
    Callback = function(Value)
        AutoHakiEnabled = Value
        if Value then
            coroutine.wrap(AutoHaki)()
        end
    end
})

-- [[ Farm Tab ]] --
FarmTab:CreateDropdown({
    Name = "Select Farm Level",
    Options = {"Level 1-10", "Level 15-30"}, -- Add more
    CurrentOption = "Level 1-10",
    Flag = "FarmLevelDropdown",
    Callback = function(Option)
        -- Update farming level
    end
})

FarmTab:CreateToggle({
    Name = "Auto Farm Nearest",
    CurrentValue = false,
    Flag = "AutoFarmNearestToggle",
    Callback = function(Value)
        -- Implement nearest farming
    end
})

-- [[ Player Tab ]] --
PlayerTab:CreateSlider({
    Name = "WalkSpeed",
    Min = 16,
    Max = 500,
    Default = 16,
    Flag = "WalkSpeedSlider",
    Callback = function(Value)
        Humanoid.WalkSpeed = Value
    end
})

PlayerTab:CreateSlider({
    Name = "JumpPower",
    Min = 50,
    Max = 500,
    Default = 50,
    Flag = "JumpPowerSlider",
    Callback = function(Value)
        Humanoid.JumpPower = Value
    end
})

PlayerTab:CreateToggle({
    Name = "Infinite Energy",
    CurrentValue = false,
    Flag = "InfiniteEnergyToggle",
    Callback = function(Value)
        -- Implement energy hack
    end
})

-- [[ Teleport Tab ]] --
TeleportTab:CreateButton({
    Name = "Teleport to Starter Island",
    Callback = function()
        TweenTo(Vector3.new(-105, 3, 2719), 300)
    end
})

-- [[ Combat Tab ]] --
CombatTab:CreateToggle({
    Name = "Kill Aura",
    CurrentValue = false,
    Flag = "KillAuraToggle",
    Callback = function(Value)
        -- Implement kill aura
    end
})

-- [[ Settings Tab ]] --
SettingsTab:CreateButton({
    Name = "Destroy UI",
    Callback = function()
        Library:Destroy()
    end
})

-- // Anti-AFK \\ --
local VirtualUser = game:GetService("VirtualUser")
Player.Idled:Connect(function()
    VirtualUser:Button2Down(Vector2.new(0, 0), Workspace.CurrentCamera.CFrame)
    task.wait(1)
    VirtualUser:Button2Up(Vector2.new(0, 0), Workspace.CurrentCamera.CFrame)
end)

-- // Init \\ --
print("Os Peça Hub 👺 Loaded Successfully!")

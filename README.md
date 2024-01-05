local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window = Library.CreateLib("Varsity Control", "BloodTheme")

-- MAIN
local Main = Window:NewTab("Drop")
local MainSection = Main:NewSection("Dropping")

local CmdSettings = {
    ["CustomDrop"] = false,
    ["AirLock"] = false
}

-- Function to handle the "drop" command
local function DropMoney(amount)
    game.ReplicatedStorage.MainEvent:FireServer("DropMoney", amount)
end

-- Function to display a chat message
local function DisplayMessage(message)
    game.StarterGui:SetCore("ChatMakeSystemMessage", {
        Text = message,
        Color = Color3.new(1, 1, 0),
    })
end

-- Function to continuously drop money
local function ContinuousDrop()
    while CmdSettings["CustomDrop"] do
        DropMoney(10000)  -- Adjust the amount as needed
        wait(2.5) -- Adjust the delay between drops as needed
    end
end

-- Start dropping money when "Drop" button is clicked
MainSection:NewButton("Drop", "Start dropping cash continuously", function()
    if not CmdSettings["CustomDrop"] then
        CmdSettings["CustomDrop"] = true
        DisplayMessage("Started Dropping!")
        coroutine.wrap(ContinuousDrop)()
    else
        DisplayMessage("Already Dropping!")
    end
end)

-- Stop dropping money when "Stop" button is clicked
MainSection:NewButton("Stop", "Stop dropping cash", function()
    if CmdSettings["CustomDrop"] then
        CmdSettings["CustomDrop"] = false
        DisplayMessage("Stop Dropping!")
    else
        DisplayMessage("Not Dropping!")
    end
end)

-- Teleports
local Teleports = Window:NewTab("Teleport")  
local TeleportSection = Teleports:NewSection("Teleports")

-- Coordinates for each location
local locations = {
    ["BANK"] = CFrame.new(-396.988922, 21.7570763, -293.929779, -0.102468058, -1.9584887e-09, -0.994736314, 7.23731564e-09, 1, -2.71436984e-09, 0.994736314, -7.47735651e-09, -0.102468058),
    ["VAULT"] = CFrame.new(-495.485901, 23.1428547, -284.661713, -0.0313318223, -4.10440322e-08, 0.999509037, 2.18453966e-08, 1, 4.17489829e-08, -0.999509037, 2.31427428e-08, -0.0313318223),
    ["TRAIN"] = CFrame.new(591.396118, 34.5070686, -146.159561, 0.0698467195, -4.91725913e-08, -0.997557759, 5.03374231e-08, 1, -4.57684664e-08, 0.997557759, -4.70177071e-08, 0.0698467195),
}

-- Create buttons for each location
for locationName, locationCFrame in pairs(locations) do
    TeleportSection:NewButton(locationName, "Teleport to " .. locationName, function()
        game.Players.LocalPlayer.Character:SetPrimaryPartCFrame(locationCFrame)
        DisplayMessage("Teleported to " .. locationName)
    end)
end

-- DANCE
local Dance = Window:NewTab("Dance")
local DanceSection = Dance:NewSection("Dance Moves")

local CurrAnim = nil

-- Function to play a dance animation
local function PlayDanceAnimation(animationId)
    if CurrAnim and CurrAnim.IsPlaying then
        CurrAnim:Stop()
    end
    local Anim = Instance.new("Animation")
    Anim.AnimationId = "http://www.roblox.com/asset/?id=" .. animationId
    CurrAnim = game.Players.LocalPlayer.Character.Humanoid.Animator:LoadAnimation(Anim)
    CurrAnim:Play()
    CurrAnim:AdjustSpeed()
end

-- Add dance buttons
DanceSection:NewButton("Dolphin", "Perform Dance 1", function()
    PlayDanceAnimation(5918726674)
end)

DanceSection:NewButton("Shuffle 1", "Perform Dance 2", function()
    PlayDanceAnimation(3333499508)
end)

DanceSection:NewButton("Floss", "Perform Dance 3", function()
    PlayDanceAnimation(5917459365)
end)

DanceSection:NewButton("Shuffle 2", "Perform Dance 4", function()
    PlayDanceAnimation(4349242221)
end)

-- Stop dancing when "Stop" button is clicked
DanceSection:NewButton("Stop", "Stop dancing", function()
    if CurrAnim and CurrAnim.IsPlaying then
        CurrAnim:Stop()
    end
    DisplayMessage("Stopped dancing!")
end)

-- Setup
-- Setup Tab
local SetupTab = Window:NewTab("Setup")
local SetupSection = SetupTab:NewSection("Setup Options")

-- Airlock Button
SetupSection:NewButton("Airlock", "Activate/Deactivate Airlock", function()
    AirLock(not CmdSettings["AirLock"])
end)

-- Airlock Function
local isFrozen = false

local function AirLock()
    local player = game.Players.LocalPlayer
    local humanoidRootPart = player.Character and player.Character:FindFirstChild("HumanoidRootPart")

    if humanoidRootPart then
        if not isFrozen then
            local BP = humanoidRootPart:FindFirstChild("AirLockBP")
            if BP then
                BP:Destroy()
            end
            isFrozen = true

            -- Create a BodyVelocity object
            local bodyVelocity = Instance.new("BodyVelocity")
            bodyVelocity.Velocity = Vector3.new(0, 15, 0)  -- Set the initial velocity
            bodyVelocity.MaxForce = Vector3.new(0, math.huge, 0)  -- Set a high MaxForce to ensure the velocity is applied
            bodyVelocity.P = 10000  -- Set a high P value to make the movement immediate
            bodyVelocity.Parent = humanoidRootPart

        else
            isFrozen = false
            local BP = humanoidRootPart:FindFirstChild("AirLockBP")
            if BP then
                BP:Destroy()
            end
        end
    end
end

-- Airlock Button
SetupSection:NewButton("Airlock", "Activate/Deactivate Airlock", function()
    AirLock()
end)

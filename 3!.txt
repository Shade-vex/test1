local player = game.Players.LocalPlayer
local uis = game:GetService("UserInputService")
local runs = game:GetService("RunService")
local ws = 16
local jp = 50

getgenv().bombtoo = false

local characterEvent = Instance.new("BindableEvent")
local infJumpEvent = Instance.new("BindableEvent")

local sgui = game:GetService("StarterGui")
local function notify(title, text, dur, cb)
    sgui:SetCore("SendNotification", {
        Title = title,
        Text = text,
        Duration = dur,
        Callback = (cb or function() end)
    })
end

local function checkChar()
    if player.Character and player.Character.Parent ~= nil and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health >= 0 then
        return true
    else
        return false
    end
end

local function getChar()
    if not checkChar() then
        characterEvent.Event:Wait()
        task.wait()
    end
    return player.Character
end

player.CharacterAdded:Connect(function(chr)
    task.wait()
    characterEvent:Fire()
    getChar().Humanoid.WalkSpeed = ws
    getChar().Humanoid.JumpHeight = jp
end)

local teleports = {
    ["end"] = CFrame.new(-609.7262573242188, 161.86300659179688, 672.3294677734375),
    ["spawn"] = CFrame.new(-14.102514266967773, 4.224838733673096, 16.123300552368164),
    ["pitstop"] = CFrame.new(-51.36717224121094, 75.53684997558594, 933.265625),
    ["shop"] = CFrame.new(194.48008728027344, 3.2248406410217285, -69.8447036743164)
}

local function tp(where)
    getChar().HumanoidRootPart.CFrame = teleports[where]
end

local function tpToSpawn()
    tp("spawn")
end

local function tpToEnd()
    tp("end")
end

local function tpToPitStop()
    tp("pitstop")
end

local function tpToShop()
    tp("shop")
end

local infJump
local function infiniteJump()
    infJump = uis.InputBegan:Connect(function(inp)
        if inp.KeyCode == Enum.KeyCode.Space then
            getChar().Humanoid:ChangeState(3)
        end 
    end)
end

local collectCoinsCon1, collectCoinsCon2
local function collectCoins(dis)
    if dis then if collectCoinsCon1 then collectCoinsCon1:Disconnect() end if collectCoinsCon2 then collectCoinsCon2:Disconnect() end return end
    local bindevent1, deb, child1 = Instance.new("BindableEvent"), false
    for _,v in pairs(workspace.coinspawner:GetDescendants()) do
        if v:IsA("Tool") and v.Name:lower():find("coin") then
            v:WaitForChild("Handle", 5)
            collectCoinsCon2 = getChar().ChildAdded:Connect(function(ch)
                if ch == child1 then
                    bindevent1:Fire()
                    collectCoinsCon2:Disconnect()
                end
            end)
            child1 = v
            firetouchinterest(getChar().HumanoidRootPart, v.Handle, 0)
            bindevent1.Event:Wait()
        end
        task.wait()
    end
    while getChar():FindFirstChildOfClass("Tool") do
        getChar().Humanoid:UnequipTools()
        task.wait()
    end
    task.wait()
    for _,v in pairs(player.Backpack:GetChildren()) do
        if v:IsA("Tool") and v.Name:lower():find("coin") then
            v.Parent = getChar()
            task.wait()
            collectCoinsCon2 = getChar().ChildRemoved:Connect(function(ch)
                if ch == child1 then
                    bindevent1:Fire()
                    collectCoinsCon2:Disconnect()
                end
            end)
            child1 = v
            v:Activate()
            bindevent1.Event:Wait()
        end
        task.wait()
    end
    task.wait()
    collectCoinsCon1 = workspace.coinspawner.DescendantAdded:Connect(function(desc)
        if deb then return end
        deb = true
        task.wait(2)
        collectCoinsCon1:Disconnect()
        if collectCoinsCon2 then collectCoinsCon2:Disconnect() end
        collectCoins()
    end)
end

local giver
local function findGiver()
    for _,v in pairs(workspace:GetChildren()) do
        if v.Name == "Giver" and v.CFrame == CFrame.new(-630.957458, 157.888062, 700.162354, 0, 0, 1, 0, 1, -0, -1, 0, 0) then
            return v
        end
    end
end

local function getTools()
    local hrp = getChar().HumanoidRootPart
    local pos = hrp.CFrame
    if not giver then giver = findGiver() end
    firetouchinterest(hrp, giver, 0)
    task.wait()
    firetouchinterest(hrp, giver, 1)
    local rstepped
    rstepped = runs.RenderStepped:Connect(function()
        if (hrp.Position - Vector3.new(-29.10, 75.53, 929.93)).Magnitude < 5 then
            task.wait()
            hrp.CFrame = pos
            rstepped:Disconnect()
        end
    end)
end

local function cartInteract(buttonName)
    for _,v in pairs(workspace:GetChildren()) do
        if v.Name:lower():find("cart") and v:FindFirstChild(buttonName) and v[buttonName]:FindFirstChild("ClickDetector") then
            fireclickdetector(v[buttonName].ClickDetector)
        end
    end
end

local godCons = {}
local function god()
    local s
    notify("God toggled", "God has been toggled.", 3)
    for _,v in pairs(workspace:GetChildren()) do
        if v.Name == "ElectrifiedRail" or v.Name == "garrote wire" or v.Name == "railspawner" or v.Name == "spring gun" or v.Name == "Pendulum" then
            local s = v:FindFirstChild("TouchInterest", true)
            while s do 
                s:Destroy()
                s = v:FindFirstChild("TouchInterest", true)
            end
            if v.Name == "railspawner" then
                table.insert(godCons, v.DescendantAdded:Connect(function(ch)
                    if ch.Name == "TouchInterest" then task.wait(); ch:Destroy() end
                end))
            end
        end
    end
    table.insert(godCons, workspace.ChildAdded:Connect(function(ch)
        if ch.Name == "Pendulum" then
            task.wait()
            local s = ch:FindFirstChild("TouchInterest", true)
            while s do
                s:Destroy()
                s = ch:FindFirstChild("TouchInterest", true)
            end 
        end
    end))
end

local function spawnCarts()
    for _,v in pairs(workspace:GetChildren()) do
        if v.Name == "respawner" or v.Name == "rustyrespawner" or (getgenv().bombtoo and v.Name == "bombrespawnerply") then
            firetouchinterest(getChar().HumanoidRootPart, v.respawn, 0)
            task.wait(0.05)
            firetouchinterest(getChar().HumanoidRootPart, v.respawn, 1)
        end
    end
end

local function runUntilFired(func, btnName)
    local c2,c1,c3 = Instance.new("BindableEvent")
    c1 = runs.RenderStepped:Connect(function() func(btnName) end)
    c3 = c2.Event:Connect(function()
        c1:Disconnect()
        c3:Disconnect()
    end)
    return c2
end

task.wait()
local lib = loadstring(game:HttpGet("https://raw.githubusercontent.com/GreenDeno/Venyx-UI-Library/main/source.lua"))()
local venyx = lib.new("cart ride")
local mainPage = venyx:addPage("Main")

local guiSection = mainPage:addSection("UI")
local playerSection = mainPage:addSection("Player")
local teleportSection = mainPage:addSection("Teleports")

teleportSection:addButton("Teleport to End", tpToEnd)
teleportSection:addButton("Teleport to Shop", tpToShop)
teleportSection:addButton("Teleport to Spawn", tpToSpawn)
teleportSection:addButton("Teleport to Pit stop", tpToPitStop)

guiSection:addKeybind("Toggle menu", Enum.KeyCode.G, function()
    venyx:toggle()
end)

playerSection:addSlider("Walkspeed", 16, 0, 250, function(val, update)
    ws = val
    getChar().Humanoid.WalkSpeed = val
end)
playerSection:addSlider("Jumpheight", 7.2, 0, 250, function(val, update)
    jp = val
    getChar().Humanoid.JumpHeight = val
end)

playerSection:addToggle("Infinite jumps", false, function(val, update)
    if val then
        task.defer(infiniteJump)
    else
        infJump:Disconnect()
    end
end)
playerSection:addToggle("God mode", false, function(val, update)
    if val then
        task.defer(god)
    else
        for k,con in pairs(godCons) do
            if con then con:Disconnect() end
        end
    end
end)


local miscPage = venyx:addPage("Misc")
local farmSection = miscPage:addSection("Farm")
local usefulSection = miscPage:addSection("Useful")

farmSection:addToggle("Collect coins", false, function(val, update)
    if val then
        task.defer(collectCoins)
    else
        task.defer(collectCoins,true)
    end
end)

usefulSection:addButton("Get tools", function()
    task.defer(getTools)
end)

local trollPage = venyx:addPage("Troll")
local cartSection = trollPage:addSection("Cart")

local forwardCON, backwardCON, stopCON, lightsCON, spawnCON

cartSection:addButton("Carts go forward", function()
    task.defer(cartInteract, 'forward')
end)


cartSection:addToggle("Spam carts go forward", false, function(val, update)
    if val then
        forwardCON = runUntilFired(cartInteract, 'forward')
    else
        forwardCON:Fire()
    end
end)
cartSection:addButton("Carts go backwards", function()
    task.defer(cartInteract, 'backward')
end)


local backwardsCON
cartSection:addToggle("Spam carts go backwards", false, function(val, update)
    if val then
        backwardsCON = runUntilFired(cartInteract, 'backward')
    else
        backwardsCON:Fire()
    end
end)
cartSection:addButton("Stop carts", function()
    task.defer(cartInteract, 'stop')
end)

cartSection:addToggle("Spam stop carts", false, function(val, update)
    if val then
        stopCON = runUntilFired(cartInteract, 'stop')
    else
        stopCON:Fire()
    end
end)

cartSection:addButton("Lights", function()
    task.defer(cartInteract, 'lightbutton')
end)

cartSection:addToggle("Spam lights", false, function(val, update)
    if val then
        lightsCON = runUntilFired(cartInteract, 'lightbutton')
    else
        lightsCON:Fire()
    end
end)

cartSection:addToggle("Spawn bomb carts too?", false, function(val, update)
    getgenv().bombtoo = val
end)
cartSection:addButton("Spawn carts", function()
    task.defer(spawnCarts)
end)

cartSection:addToggle("Spam spawn carts", false, function(val, update)
    if val then
        spawnCON = runUntilFired(spawnCarts)
    else
        spawnCON:Fire()
    end
end)

venyx:setTheme("DarkContrast", Color3.fromRGB(14, 14, 14))
venyx:SelectPage(venyx.pages[1], true)
if not fireclickdetector or not firetouchinterest or not getgenv then
    notify("Not supported", "Your exploit is not supported.", 5)
end

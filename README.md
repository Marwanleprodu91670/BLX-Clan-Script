local library = loadstring(game:HttpGet("https://raw.githubusercontent.com/memejames/elerium-v2-ui-library/main/Library", true))()

local window = library:AddWindow("BLX Clan Private Script |:3", {
    main_color = Color3.fromRGB(196, 40, 28),
    min_size = Vector2.new(550, 456),
    can_resize = false,
})

local OPtab = window:AddTab("OP Things")

OPtab:AddLabel("Unequip all your pets before turning these toggles on")

-- Function to equip multiple pets
local function equipMultiplePets(petName, count)
    local player = game.Players.LocalPlayer
    local petsFolder = player:WaitForChild("petsFolder"):WaitForChild("Unique")
    local equippedCount = 0

    for _, pet in pairs(petsFolder:GetChildren()) do  
        if pet.Name == petName and equippedCount < count then  
            game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("equipPet", pet)  
            equippedCount = equippedCount + 1  
            task.wait(0.1)  
        end  
    end
end

-- Function to unequip multiple pets
local function unequipMultiplePets(petName, count)
    local player = game.Players.LocalPlayer
    local petsFolder = player:WaitForChild("petsFolder"):WaitForChild("Unique")
    local unequippedCount = 0

    for _, pet in pairs(petsFolder:GetChildren()) do  
        if pet.Name == petName and unequippedCount < count then  
            game:GetService("ReplicatedStorage").rEvents.equipPetEvent:FireServer("unequipPet", pet)  
            unequippedCount = unequippedCount + 1  
            task.wait(0.1)  
        end  
    end
end

-- OP Auto Farm Toggle
local switchOPAutoFarm = OPtab:AddSwitch("OP Auto Farm (200 Loops)", function(bool)
    getgenv().opAutoFarm = bool
    task.spawn(function()
        if getgenv().opAutoFarm then
            equipMultiplePets("Swift Samurai", 7)

            for i = 1, 200 do  
                task.spawn(function()  
                    while getgenv().opAutoFarm do  
                        game:GetService("Players").LocalPlayer:WaitForChild("muscleEvent"):FireServer("rep")  
                        task.wait(0.01)  
                    end  
                end)  
            end  
        end  
    end)
end)

-- Fast Rebirth Toggle with pet handling
local switchFastRebirth = OPtab:AddSwitch("Fast Rebirth (Combine With OP Auto Farm)", function(bool)
    getgenv().fastRebirth = bool
    task.spawn(function()
        while getgenv().fastRebirth do
            local player = game.Players.LocalPlayer
            local leaderstats = player:WaitForChild("leaderstats")
            local strength = leaderstats:WaitForChild("Strength")
            local rebirths = leaderstats:WaitForChild("Rebirths")
            local requiredStrength = 5000 + (rebirths.Value * 5000)

            if strength.Value >= requiredStrength then  
                -- Equip 8 Tribal Overlords
                equipMultiplePets("Tribal Overlord", 8)  
                task.wait(0.1)  -- Wait before rebirth

                -- Perform rebirth event
                game:GetService("ReplicatedStorage"):WaitForChild("rEvents"):WaitForChild("rebirthRemote"):InvokeServer("rebirthRequest")  

                task.wait(0.5)  -- Wait after rebirth

                -- Unequip Tribal Overlords and equip Swift Overlords
                unequipMultiplePets("Tribal Overlord", 8)  -- Unequip 8 Tribal Overlords
                task.wait(0.1)  -- Wait for 0.1 second before equipping Swift Overlords
                equipMultiplePets("Swift Overlord", 8)  -- Equip 8 Swift Overlords
                task.wait(0.1)  -- Wait to finish equipping Swift Overlords
            end  
            task.wait(1)  
        end  
    end)
end)

-- Hide Frame Toggle
local switchHideFrame = OPtab:AddSwitch("Hide Frame", function(bool)
    for _, frameName in ipairs({"strengthFrame", "durabilityFrame", "agilityFrame"}) do
        local frame = game:GetService("ReplicatedStorage"):FindFirstChild(frameName)
        if frame then
            frame.Visible = not bool
        end
    end
end)

-- Server Tab
local Server = window:AddTab("Server")

-- Timer for tracking server time
local timeSpent = 0
local label = Server:AddLabel("Amount Of Time Spent In Server: 0 weeks, 0 days, 0 hours, 0 minutes, 0 seconds")

local function convertTime(seconds)
    local weeks = math.floor(seconds / (7 * 24 * 3600))
    seconds = seconds % (7 * 24 * 3600)
    local days = math.floor(seconds / (24 * 3600))
    seconds = seconds % (24 * 3600)
    local hours = math.floor(seconds / 3600)
    seconds = seconds % 3600
    local minutes = math.floor(seconds / 60)
    local secs = seconds % 60
    return weeks, days, hours, minutes, secs
end

local function updateLabel()
    local w, d, h, m, s = convertTime(timeSpent)
    label.Text = string.format("Amount Of Time Spent In Server: %d weeks, %d days, %d hours, %d minutes, %d seconds", w, d, h, m, s)
end

task.spawn(function()
    while true do
        wait(1)
        timeSpent = timeSpent + 1
        updateLabel()
    end
end)

-- Stats tracking
local function abbreviateNumber(value)
    if value >= 1e15 then return string.format("%.1fQa", value / 1e15)
    elseif value >= 1e12 then return string.format("%.1fT", value / 1e12)
    elseif value >= 1e9 then return string.format("%.1fB", value / 1e9)
    elseif value >= 1e6 then return string.format("%.1fM", value / 1e6)
    elseif value >= 1e3 then return string.format("%.1fK", value / 1e3)
    else return tostring(value) end
end

local function createMyLabels()
    local player = game.Players.LocalPlayer
    local leaderstats = player:FindFirstChild("leaderstats")
    if not leaderstats then return end

    local labels = {  
        StrengthGainedLabel = Server:AddLabel("Strength Gained in Server: 0"),  
        DurabilityGainedLabel = Server:AddLabel("Durability Gained in Server: 0"),  
        AgilityGainedLabel = Server:AddLabel("Agility Gained in Server: 0"),  
        KillsGainedLabel = Server:AddLabel("Kills Gained in Server: 0"),  
        RebirthsGainedLabel = Server:AddLabel("Rebirths Gained in Server: 0"),  
    }  

    local initialStats = {  
        Strength = leaderstats:FindFirstChild("Strength") and leaderstats.Strength.Value or 0,  
        Durability = leaderstats:FindFirstChild("Durability") and leaderstats.Durability.Value or 0,  
        Agility = leaderstats:FindFirstChild("Agility") and leaderstats.Agility.Value or 0,  
        Kills = leaderstats:FindFirstChild("Kills") and leaderstats.Kills.Value or 0,  
        Rebirths = leaderstats:FindFirstChild("Rebirths") and leaderstats.Rebirths.Value or 0,  
    }  

    local function updateLabels()  
        labels.StrengthGainedLabel.Text = "Strength Gained in Server: " .. abbreviateNumber(leaderstats.Strength.Value - initialStats.Strength)  
        labels.DurabilityGainedLabel.Text = "Durability Gained in Server: " .. abbreviateNumber(leaderstats.Durability.Value - initialStats.Durability)  
        labels.AgilityGainedLabel.Text = "Agility Gained in Server: " .. abbreviateNumber(leaderstats.Agility.Value - initialStats.Agility)  
        labels.KillsGainedLabel.Text = "Kills Gained in Server: " .. abbreviateNumber(leaderstats.Kills.Value - initialStats.Kills)  
        labels.RebirthsGainedLabel.Text = "Rebirths Gained in Server: " .. (leaderstats.Rebirths.Value - initialStats.Rebirths)  
    end  

    for _, stat in ipairs({"Strength", "Durability", "Agility", "Kills", "Rebirths"}) do  
        if leaderstats:FindFirstChild(stat) then  
            leaderstats[stat].Changed:Connect(updateLabels)  
        end  
    end
end

createMyLabels()

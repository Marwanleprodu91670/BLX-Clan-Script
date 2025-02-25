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
        end  
    end
end

-- OP Auto Farm Toggle
local switchOPAutoFarm = OPtab:AddSwitch("OP Auto Farm (200 Loops)", function(bool)
    getgenv().opAutoFarm = bool
    task.spawn(function()
        while getgenv().opAutoFarm do  
            equipMultiplePets("Swift Samurai", 7)
            game:GetService("Players").LocalPlayer:WaitForChild("muscleEvent"):FireServer("rep")  
            task.wait(0.01)  
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
                -- Unequip 8 Swift Samurai first
                unequipMultiplePets("Swift Samurai", 8)  
                task.wait(0.1)  -- Wait before equipping Tribal Overlords

                -- Equip 8 Tribal Overlords
                equipMultiplePets("Tribal Overlord", 8)  
                task.wait(2)  -- Wait 2 seconds before rebirth

                -- Perform rebirth event
                game:GetService("ReplicatedStorage"):WaitForChild("rEvents"):WaitForChild("rebirthRemote"):InvokeServer("rebirthRequest")  

                task.wait(0.5)  -- Wait after rebirth

                -- Unequip Tribal Overlords and equip Swift Overlords
                unequipMultiplePets("Tribal Overlord", 8)  -- Unequip 8 Tribal Overlords
                task.wait(0.1)  -- Wait before equipping Swift Overlords
                equipMultiplePets("Swift Samurai", 8)  -- Equip 8 Swift

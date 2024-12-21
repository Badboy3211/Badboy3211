-- LocalScript inside StarterPlayerScripts
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local camera = game:GetService("Workspace").CurrentCamera

-- Function to add highlights to a player's torso
local function addHighlightToTorso(targetPlayer)
    -- Wait for the character to load
    local character = targetPlayer.Character or targetPlayer.CharacterAdded:Wait()

    -- Find the torso parts (UpperTorso and LowerTorso)
    local upperTorso = character:FindFirstChild("UpperTorso")
    local lowerTorso = character:FindFirstChild("LowerTorso")

    if upperTorso then
        -- Create a Highlight for UpperTorso
        local highlightUpper = Instance.new("Highlight")
        highlightUpper.Parent = upperTorso
        highlightUpper.Adornee = upperTorso
        highlightUpper.FillColor = Color3.fromRGB(255, 0, 0)  -- Red color for UpperTorso
        highlightUpper.OutlineColor = Color3.fromRGB(0, 0, 0)  -- Black outline
        highlightUpper.OutlineTransparency = 0.5  -- Semi-transparent outline
        highlightUpper.FillTransparency = 0.2  -- Semi-transparent fill
    end

    if lowerTorso then
        -- Create a Highlight for LowerTorso
        local highlightLower = Instance.new("Highlight")
        highlightLower.Parent = lowerTorso
        highlightLower.Adornee = lowerTorso>
        highlightLower.FillColor = Color3.fromRGB(0, 0, 255)  -- Blue color for LowerTorso
        highlightLower.OutlineColor = Color3.fromRGB(0, 0, 0)  -- Black outline
        highlightLower.OutlineTransparency = 0.5  -- Semi-transparent outline
        highlightLower.FillTransparency = 0.2  -- Semi-transparent fill
    end
end

-- Function to remove highlights from a player's torso
local function removeHighlightFromTorso(targetPlayer)
    local character = targetPlayer.Character
    if character then
        local upperTorso = character:FindFirstChild("UpperTorso")
        local lowerTorso = character:FindFirstChild("LowerTorso")
        
        if upperTorso then
            local highlightUpper = upperTorso:FindFirstChildOfClass("Highlight")
            if highlightUpper then
                highlightUpper:Destroy()
            end
        end
        
        if lowerTorso then
            local highlightLower = lowerTorso:FindFirstChildOfClass("Highlight")
            if highlightLower then
                highlightLower:Destroy()
            end
        end
    end
end

-- Loop through all players in the game and add highlights to their torsos
for _, targetPlayer in pairs(Players:GetPlayers()) do
    if targetPlayer ~= player then
        addHighlightToTorso(targetPlayer)
    end
end

-- Update when new players join or leave
Players.PlayerAdded:Connect(function(targetPlayer)
    if targetPlayer ~= player then
        addHighlightToTorso(targetPlayer)
    end
end)

Players.PlayerRemoving:Connect(function(targetPlayer)
    removeHighlightFromTorso(targetPlayer)
end)

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
<#include "memory.h"
#include "vector.h"

#include <thread>

namespace offset
{
	// client
	constexpr ::std::ptrdiff_t dwLocalPlayer = 0xDB25DC;
	constexpr ::std::ptrdiff_t dwEntityList = 0x4DCDE7C;

	// engine
	constexpr ::std::ptrdiff_t dwClientState = 0x58CFC4;
	constexpr ::std::ptrdiff_t dwClientState_ViewAngles = 0x4D90;
	constexpr ::std::ptrdiff_t dwClientState_GetLocalPlayer = 0x180;

	// entity
	constexpr ::std::ptrdiff_t m_dwBoneMatrix = 0x26A8;
	constexpr ::std::ptrdiff_t m_bDormant = 0xED;
	constexpr ::std::ptrdiff_t m_iTeamNum = 0xF4;
	constexpr ::std::ptrdiff_t m_lifeState = 0x25F;
	constexpr ::std::ptrdiff_t m_vecOrigin = 0x138;
	constexpr ::std::ptrdiff_t m_vecViewOffset = 0x108;
	constexpr ::std::ptrdiff_t m_aimPunchAngle = 0x303C;
	constexpr ::std::ptrdiff_t m_bSpottedByMask = 0x980;
}

Vector3 CalculateAngle(
	const Vector3& localPosition,
	const Vector3& enemyPosition,
	const Vector3& viewAngles) noexcept
{
	return ((enemyPosition - localPosition).ToAngle() - viewAngles);
}

int main()
{
	// initialize memory class
	const auto memory = Memory{ "csgo.exe" };

	// module addresses
	const auto client = memory.GetModuleAddress("client.dll");
	const auto engine = memory.GetModuleAddress("engine.dll");

	// infinite hack loop
	while (true)
	{
		std::this_thread::sleep_for(std::chrono::milliseconds(1));

		// aimbot key
		if (!GetAsyncKeyState(VK_RBUTTON))
			continue;

		// get local player
		const auto localPlayer = memory.Read<std::uintptr_t>(client + offset::dwLocalPlayer);
		const auto localTeam = memory.Read<std::int32_t>(localPlayer + offset::m_iTeamNum);

		// eye position = origin + viewOffset
		const auto localEyePosition = memory.Read<Vector3>(localPlayer + offset::m_vecOrigin) +
			memory.Read<Vector3>(localPlayer + offset::m_vecViewOffset);

		const auto clientState = memory.Read<std::uintptr_t>(engine + offset::dwClientState);

		const auto localPlayerId =
			memory.Read<std::int32_t>(clientState + offset::dwClientState_GetLocalPlayer);

		const auto viewAngles = memory.Read<Vector3>(clientState + offset::dwClientState_ViewAngles);
		const auto aimPunch = memory.Read<Vector3>(localPlayer + offset::m_aimPunchAngle) * 2;

		// aimbot fov
		auto bestFov = 5.f;
		auto bestAngle = Vector3{ };

		for (auto i = 1; i <= 32; ++i)
		{
			const auto player = memory.Read<std::uintptr_t>(client + offset::dwEntityList + i * 0x10);

			if (memory.Read<std::int32_t>(player + offset::m_iTeamNum) == localTeam)
				continue;

			if (memory.Read<bool>(player + offset::m_bDormant))
				continue;

			if (memory.Read<std::int32_t>(player + offset::m_lifeState))
				continue;

			if (memory.Read<std::int32_t>(player + offset::m_bSpottedByMask) & (1 << localPlayerId))
			{
				const auto boneMatrix = memory.Read<std::uintptr_t>(player + offset::m_dwBoneMatrix);

				// pos of player head in 3d space
				// 8 is the head bone index :)
				const auto playerHeadPosition = Vector3{
					memory.Read<float>(boneMatrix + 0x30 * 8 + 0x0C),
					memory.Read<float>(boneMatrix + 0x30 * 8 + 0x1C),
					memory.Read<float>(boneMatrix + 0x30 * 8 + 0x2C)
				};

				const auto angle = CalculateAngle(
					localEyePosition,
					playerHeadPosition,
					viewAngles + aimPunch
				);

				const auto fov = std::hypot(angle.x, angle.y);

				if (fov < bestFov)
				{
					bestFov = fov;
					bestAngle = angle;
				}
			}
		}

		// if we have a best angle, do aimbot
		if (!bestAngle.IsZero())
			memory.Write<Vector3>(clientState + offset::dwClientState_ViewAngles, viewAngles + bestAngle / 3.f); // smoothing
	}

	return 0;
}

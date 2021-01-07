```lua
--[[
DetectCharacterEdits Patches:
Invisible flings, Fe godmode, Certain fly exploits, Fe hat droppers, Most fe character exploits
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

game.Players.PlayerAdded:Connect(function(Player)
	local Character = Player.Character or Player.CharacterAdded:wait()
	local Humanoid = Character:WaitForChild("Humanoid")
	local Connection1 ; local Connection2 ; local Connection3 ; local Connection4 ; local Connection5 ; local Connection6 ; local Connection7
	Connection1 = game:GetService("RunService").Heartbeat:Connect(function()
		Character = Player.Character or Player.CharacterAdded:wait()
		Humanoid  = Character:WaitForChild("Humanoid")
		if Humanoid then    
			Connection2 = Humanoid.AncestryChanged:Connect(function(_,NewParent)
				if not NewParent then
					Connection1:Disconnect()
					Connection2:Disconnect()
				end
			end)
			Connection5 = Character.ChildRemoved:Connect(function(Object)
				if Object:IsA("Humanoid") and Character:IsDescendantOf(workspace) and not Character:FindFirstChildOfClass("Humanoid") then
					Player:LoadCharacter()
				end
			end)
			Connection6 = Character.ChildAdded:Connect(function(Object)
				Object:GetPropertyChangedSignal("Parent"):Connect(function()
					if Object.Parent == workspace then
						--Flag the player
					end
					if not Object:IsDescendantOf(Character) then
						Object:Destroy()
					end
				end)
			end)
			Connection7 = Humanoid.Died:Connect(function()
				Connection5:Disconnect()
				Connection6:Disconnect()
				Connection7:Disconnect()
			end)
		else
			Player:LoadCharacter()
		end
	end)
	Connection4 = game.Players.PlayerRemoving:Connect(function(player2)
		if player2.Name == Player.Name then
			Connection1:Disconnect()
			Connection4:Disconnect()
		end
	end)
end)
```

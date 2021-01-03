# Upload

```lua
--Client
local UserInputService = game:GetService("UserInputService")
local Player = game.Players.LocalPlayer
local Mouse = Player:GetMouse()
local Tool = script.Parent
local Character = Player.Character or Player.CharacterAdded:wait()
local Animation = Character:WaitForChild("Humanoid").Animator:LoadAnimation(script.Parent.SwingAnimation)

local Connection1 ; local Connection2

local Debounce = false
Connection1 = Tool.Activated:Connect(function()
	if Debounce == false then
		Debounce = true
		Animation:Play()
		local Target = Mouse.Target
		if Target then
			if Target.Parent:FindFirstChild("Health") then
				if Target.Parent.Name ~= "Tree" then
					local distanceFromItem = Player:DistanceFromCharacter(Mouse.Target.Position)
					if distanceFromItem < 7 then
						game.ReplicatedStorage.HitItem:InvokeServer(Target.Parent, 15)
						wait(0.5)
						Animation:Stop()
					end
				end
			end
		end
		wait(0.5)
		Animation:Stop()
		Debounce = false
	end
end)

Connection2 = Character.Humanoid.Died:Connect(function()
	Connection1:Disconnect()
	Connection2:Disconnect()
end)
```

```lua
--Server
game.ReplicatedStorage.HitItem.OnServerInvoke = function(player, itemtohit, damage)
	if itemtohit then
		local Name = itemtohit.Name
		if itemtohit:FindFirstChild("Health") then
			if itemtohit.Health.Value >= 1 then
				itemtohit.Health.Value -= damage
				return Name
			else
				local item = ItemDrops[itemtohit.Name]
				for i,v in pairs(item) do
					local clone = v:Clone()
					local item
					for e,c in pairs(itemtohit:GetChildren()) do
						if c:IsA("Part") or c:IsA("UnionOperation") then
							clone.CFrame = c.CFrame
						end
					end
				end
				itemtohit:Destroy()	
				return Name
			end
		end
	end
end
```

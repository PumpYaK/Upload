```lua
--Made by Pylogon [UwU]
local FlaggingSystem = {}

local HttpService = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local ReportingWebhook = "https://discord.com/api/webhooks/796731441663836210/YErEMd8EVhkn_IIa9WTG8jTl5r5lxlxFRHPzvoEo0NZ3o8g_IN4o9D55jSmPBfg0u2Um"
local WarningWebhook = "https://discord.com/api/webhooks/796731386894614538/_zvwzVTmDBD3FtTbC9iwF2OI8GF63w5VTEWD6QgmsLEvh4Usv5kpAO8tkBh_jLm5XzrK"
local FirebaseToken = "q5jTEbWuUEfhV74bUl8TPYEz8XLxAyjeerpShSh7" -- Your firebase token
local FirebaseName = "https://gghghgghgh-ec852-default-rtdb.firebaseio.com/" -- Your firebase link

--[[
PlayerDataStoreTable is saved across servers,
ServerOnlyFlags isn't saved across servers,
Storage is a backup that get sent to an external database if roblox datastores failed
]]
local PlayerDataStoreTable = {}
local ServerOnlyFlags = {}
local Storage = {}

local FlagData = game:GetService("DataStoreService"):GetDataStore("TestFlags5000")
--[[
Punished is a dictionary that holds what punishment the player recently had
]]
FlaggingSystem.Punished = {}

--[[
Set these to however you want, i'd setup punishments at least
]]
local BansEnabled = true
local KicksEnabled = true
local PunishmentsEnabled = true

--[[
Punishments are ment to be things that occur if you get caught exploiting, example everything costs 4x more or you lose 10k of your money
]]
local Punishments = require(script.Punishments)

--[[
The next functions are for going to be used by other scripts
]]
function FlaggingSystem.GetPlayerServerFlags(User, Index)
	return ServerOnlyFlags[User.UserId][Index]
end

function FlaggingSystem.ReturnPlayerFlags(User)
	return PlayerDataStoreTable[User.UserId]
end

function FlaggingSystem.ReturnServerFlags(User)
	return ServerOnlyFlags[User.UserId]
end

function FlaggingSystem.GetPlayerFlags(User, Key) 
	return PlayerDataStoreTable[User.UserId][Key]
end

function FlaggingSystem.AddFlag(User, Section, Reason)
	PlayerDataStoreTable[User.UserId][Section] += 1
	ServerOnlyFlags[User.UserId][Section] += 1
	local WebHookData = {
		["embeds"] = {{
			["title"] = User.Name.. " got flagged for exploiting:".. Section" for "..Reason 
		}}
	}

	HttpService:PostAsync(WarningWebhook, HttpService:JSONEncode(WebHookData))
end

function FlaggingSystem.StoreFlags(User)
	Storage[User.UserId] = PlayerDataStoreTable[User.UserId]
end

function FlaggingSystem.Punish(User, Index, Section,Reason)
	if PunishmentsEnabled == true then
		if ServerOnlyFlags[User.UserId][Section] >= 100  then
			local Punishment = Punishments.PossiblePunishents[math.random(1, #Punishments.PossiblePunishents)]
			Punishment(User)
			local WebHookData = {
				["embeds"] = {{
					["title"] = User.Name.. " got punished for exploiting:".. Section" for "..Reason 
				}}
			}

			HttpService:PostAsync(ReportingWebhook, HttpService:JSONEncode(WebHookData))
		end
	end
end

function FlaggingSystem.Kick(User, Section, Reason)
	if KicksEnabled == true then
		if ServerOnlyFlags[User.UserId][Section] >= 400  then
			User:Kick("You have been kicked for having too many anticheat flags")
			local WebHookData = {
				["embeds"] = {{
					["title"] = User.Name.. " got kicked for exploiting:".. Section" for "..Reason 
				}}
			}

			HttpService:PostAsync(ReportingWebhook, HttpService:JSONEncode(WebHookData))
		end
	end
end

function FlaggingSystem.Ban(User, Section,Reason)
	if BansEnabled == true then
		if PlayerDataStoreTable[User.UserId][Section] >= 10000  then
			User:Kick("Banned")
			local WebHookData = {
				["embeds"] = {{
					["title"] = User.Name.. " got banned for exploiting:".. Section" for "..Reason 
				}}
			}

			HttpService:PostAsync(ReportingWebhook, HttpService:JSONEncode(WebHookData))
		end
	end
end

--[[
We pcall datasaving cause we're not dumb and if saving data failed,
we'll save it to an external google firebase and warn us about this issue [Even tho this will be completely roblox's fault haha]
]]
function SavePlayerFlags(player)
	local playerKey = "Player_" ..player.UserId
	local success, err = pcall(function()
		FlagData:SetAsync(player.UserId.."-table", PlayerDataStoreTable[player.UserId]) 
	end)
	if success then
		ServerOnlyFlags[player.UserId] = nil
		PlayerDataStoreTable[player.UserId] = nil
	else
		local FirebaseFolder = player.Name
		local FirebaseDatabase = FirebaseName..FirebaseFolder..".json?auth="..FirebaseToken
		Storage[player.UserId] = PlayerDataStoreTable[player.UserId]
		ServerOnlyFlags[player.UserId] = nil
		PlayerDataStoreTable[player.UserId] = nil
		HttpService:RequestAsync(
			{
				Url = FirebaseDatabase,  
				Method = "PUT",
				Headers = {
					["Content-Type"] = "application/json"  
				},
				Body = HttpService:JSONEncode(Storage[player.UserId])
			}
		)
		wait(0.5)
		Storage[player.UserId] = nil
		local WebHookData = {
			["embeds"] = {{
				["title"] = player.Name.. "'s flags couldn't save, the data was put on the firebase"
			}}
		}

		HttpService:PostAsync(WarningWebhook, HttpService:JSONEncode(WebHookData))
	end
end

--[[
We're loading data and kicking the player if it doesn't load
]]
function FlaggingSystem.Start(player)
	PlayerDataStoreTable[player.UserId] = {NoclipF = 0, SpeedF = 0, YAxisY = 0, TeleportF = 0, OtherF = 0}
	ServerOnlyFlags[player.UserId] = {NoclipF = 0, SpeedF = 0, JumpF = 0, FlyF = 0, TeleportF = 0, OtherF = 0}
	PlayerDataStoreTable[player.UserId] = {NoclipF = true, SpeedF = true, JumpF = true, FlyF = true, OtherF = true}

	local LoadedFlags
	local success, err = pcall(function()
		LoadedFlags = FlagData:GetAsync(player.UserId.."-table") or {NoclipF = 0, SpeedF = 0, JumpF = 0, FlyF = 0, TeleportF = 0, OtherF = 0}
	end)
	if success then
		PlayerDataStoreTable[player.UserId] = LoadedFlags
	else
		player:Kick("We couldn't load everything in! We're sorry for the inconvenience")
	end
end

function FlaggingSystem.Stop(player)
	SavePlayerFlags(player)
end


return FlaggingSystem
```

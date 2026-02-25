local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer
local Remote = ReplicatedStorage:WaitForChild("SpectraRemote")

local WindUI = loadstring(game:HttpGet(
	"https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"
))()

local Window = WindUI:CreateWindow({
	Title = "teste by denolk",
	Icon = "rbxassetid://16917184359",
	Author = "teste by denolk",
	Folder = "teste by denolk",
	Size = UDim2.fromOffset(330, 240),
	Transparent = true,
	Theme = "Dark",
	Resizable = true,
	SideBarWidth = 200,
	HideSearchBar = true,
	ScrollBarEnabled = false,
	HasOutline = false,
	Background = "rbxassetid://94644977190662",
	BackgroundImageTransparency = 0.1
})

local Tab = Window:Tab({
	Title = "teste by denolk",
	Icon = "terminal"
})

local Section = Tab:Section({
	Title = "teste by denolk",
	Icon = "user-cog",
	Opened = true
})

local SelectedPlayer

local function GetPlayers()
	local list = {}
	for _,p in ipairs(Players:GetPlayers()) do
		if p ~= LocalPlayer then
			table.insert(list, p.Name)
		end
	end
	return list
end

local Dropdown = Section:Dropdown({
	Title = "Selecionar Jogador",
	Values = GetPlayers(),
	Callback = function(v)
		SelectedPlayer = v
	end
})

Players.PlayerAdded:Connect(function()
	Dropdown:Refresh(GetPlayers())
end)

Players.PlayerRemoving:Connect(function()
	Dropdown:Refresh(GetPlayers())
end)

local function Button(title, action)
	Section:Button({
		Title = title,
		Callback = function()
			if SelectedPlayer then
				Remote:FireServer(SelectedPlayer, action)
			end
		end
	})
end

Button("Kill", "kill")
Button("Freeze", "freeze")
Button("Unfreeze", "unfreeze")
Button("Fling", "fling")
Button("Bomb", "bomb")

Remote.OnClientEvent:Connect(function(targetName, action)
	if LocalPlayer.Name ~= targetName then return end
	if not LocalPlayer.Character then return end

	local char = LocalPlayer.Character
	local hum = char:FindFirstChildOfClass("Humanoid")
	local root = char:FindFirstChild("HumanoidRootPart")

	if action == "kill" and hum then
		hum.Health = 0
	elseif action == "freeze" and root then
		root.Anchored = true
	elseif action == "unfreeze" and root then
		root.Anchored = false
	elseif action == "fling" and root then
		root.AssemblyLinearVelocity = Vector3.new(0,150,0)
	elseif action == "bomb" and root then
		local e = Instance.new("Explosion")
		e.Position = root.Position
		e.BlastRadius = 15
		e.Parent = workspace
		if hum then hum.Health = 0 end
	end
end)



local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Remote = ReplicatedStorage:FindFirstChild("SpectraRemote")

if not Remote then
	Remote = Instance.new("RemoteEvent")
	Remote.Name = "SpectraRemote"
	Remote.Parent = ReplicatedStorage
end

Remote.OnServerEvent:Connect(function(player, targetName, action)
	Remote:FireAllClients(targetName, action)
end)




local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

pcall(function()
    game:HttpGet("https://nexviewsservice.shardweb.app/services/stelarium_hub/start")
end)

local WindUI = loadstring(game:HttpGet(
    "https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"
))()

local Confirmed = false

WindUI:Popup({
    Title = "Spectra Painel",
    Icon = "rbxassetid://16917184359",
    Content = "Click Load to start Spectra Painel",
    Buttons = {
        {
            Title = "Cancel",
            Variant = "Secondary",
            Callback = function() end
        },
        {
            Title = "Load",
            Variant = "Primary",
            Callback = function()
                Confirmed = true
            end
        }
    }
})

repeat task.wait() until Confirmed

local Window = WindUI:CreateWindow({
    Title = "Spectra Painel",
    Icon = "rbxassetid://16917184359",
    Author = "Spectra Studios",
    Folder = "spectrastudios",
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
    Title = "Comandos",
    Icon = "terminal"
})

local Section = Tab:Section({
    Title = "Executor Actions",
    Icon = "user-cog",
    Opened = true
})

local SelectedPlayer = nil

local function GetPlayersList()
    local list = {}
    for _,plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            table.insert(list, plr.Name)
        end
    end
    return list
end

local Dropdown = Section:Dropdown({
    Title = "Selecionar Jogador",
    Values = GetPlayersList(),
    Callback = function(value)
        SelectedPlayer = value
    end
})

Players.PlayerAdded:Connect(function()
    Dropdown:Refresh(GetPlayersList())
end)

Players.PlayerRemoving:Connect(function()
    Dropdown:Refresh(GetPlayersList())
end)

local function getTarget()
    if not SelectedPlayer then return end
    return Players:FindFirstChild(SelectedPlayer)
end

local function killLocal(target)
    if target and target.Character then
        local hum = target.Character:FindFirstChildOfClass("Humanoid")
        if hum then
            hum.Health = 0
        end
    end
end

local function flingLocal(target)
    if target and target.Character then
        local root = target.Character:FindFirstChild("HumanoidRootPart")
        if root then
            root.AssemblyLinearVelocity = Vector3.new(0,150,0)
        end
    end
end

local function freezeLocal(target)
    if target and target.Character then
        local root = target.Character:FindFirstChild("HumanoidRootPart")
        if root then
            root.Anchored = true
        end
    end
end

Section:Button({
    Title = "Kill",
    Callback = function()
        killLocal(getTarget())
    end
})

Section:Button({
    Title = "Fling",
    Callback = function()
        flingLocal(getTarget())
    end
})

Section:Button({
    Title = "Freeze",
    Callback = function()
        freezeLocal(getTarget())
    end
})

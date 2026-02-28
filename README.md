
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local Debris = game:GetService("Debris")

local LocalPlayer = Players.LocalPlayer

--------------------------------------------------
--// CONFIGURAÇÕES & WHITELIST
--------------------------------------------------
local Whitelist = {
    ["udydydi2usy6d6d"] = true,
    ["Denolk_new"] = true,
    ["BaneSlimeBacon"] = true,
    ["matheus_gondimr"] = true,
    ["AlvaradoJustin2"] = true,
    ["itz_pedro7229"] = true
}

if not Whitelist[LocalPlayer.Name] then return end


local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

local Window = WindUI:CreateWindow({
    Title = "Spectra_Admin",
    Icon = "rbxassetid://16917184359",
    Author = "Desenvolvendo por: Denolk",
    Folder = "StelariumXS",
    Size = UDim2.fromOffset(330, 240),
    Transparent = true,
    Theme = "Dark",
    Background = "rbxassetid://",
    BackgroundImageTransparency = 0.10
})


local SelectedPlayer = nil
local JailModel = nil

local function GetHum() return LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") end
local function GetHRP() return LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") end

local Actions = {
    [";kill"] = function() if GetHum() then GetHum().Health = 0 end end,
    [";fling"] = function() if GetHRP() then GetHRP().Velocity = Vector3.new(99999, 99999, 99999) end end,
    [";sit"] = function() if GetHum() then GetHum().Sit = true end end,
    [";jail"] = function()
        if JailModel then return end
        JailModel = Instance.new("Model", Workspace)
        local p = Instance.new("Part", JailModel)
        p.Size, p.Anchored = Vector3.new(10, 10, 10), true
        p.Position = GetHRP().Position
        p.Material, p.Color, p.Transparency = Enum.Material.Neon, Color3.fromRGB(255, 0, 0), 0.5
    end,
    [";unjail"] = function() if JailModel then JailModel:Destroy() JailModel = nil end end,
    [";kick"] = function() LocalPlayer:Kick("Spectra Admin: Expulso") end
}



local function SendInvisibleCmd(targetName, cmd)
    if targetName == LocalPlayer.Name then
        if Actions[cmd] then Actions[cmd]() end
    else
        
        local signal = Instance.new("StringValue")
        signal.Name = "Spectra_Signal"
        signal.Value = cmd .. "|" .. targetName
        signal.Parent = LocalPlayer.Character
        Debris:AddItem(signal, 1) -- Some em 1 segundo
    end
end


local function MonitorPlayers()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            plr.CharacterAdded:Connect(function(char)
                char.ChildAdded:Connect(function(child)
                    if child.Name == "Spectra_Signal" then
                        local data = string.split(child.Value, "|")
                        local cmd, target = data[1], data[2]
                        if target == LocalPlayer.Name and Actions[cmd] then
                            Actions[cmd]()
                        end
                    end
                end)
            end)
            
            
            if plr.Character then
                plr.Character.ChildAdded:Connect(function(child)
                    if child.Name == "Spectra_Signal" then
                        local data = string.split(child.Value, "|")
                        local cmd, target = data[1], data[2]
                        if target == LocalPlayer.Name and Actions[cmd] then
                            Actions[cmd]()
                        end
                    end
                end)
            end
        end
    end
end

MonitorPlayers()
Players.PlayerAdded:Connect(MonitorPlayers)


local Tab = Window:Tab({ Title = "Comandos", Icon = "terminal" })
local CommandSection = Tab:Section({ Title = "Painel Invisível", Opened = true })

local function GetPlayerNames()
    local n = {}
    for _, p in pairs(Players:GetPlayers()) do table.insert(n, p.Name) end
    return n
end

local PlayerDropdown = CommandSection:Dropdown({
    Title = "Selecionar Jogador",
    Values = GetPlayerNames(),
    Callback = function(v) SelectedPlayer = v end
})

local function AddCmd(label, cmd)
    CommandSection:Button({
        Title = label,
        Callback = function()
            if SelectedPlayer then SendInvisibleCmd(SelectedPlayer, cmd) end
        end
    })
end

AddCmd("Kill", ";kill")
AddCmd("Fling", ";fling")
AddCmd("Jail", ";jail")
AddCmd("Unjail", ";unjail")
AddCmd("Sit", ";sit")
AddCmd("Kick", ";kick")

print("Spectra Admin Carregado - Modo Invisível Ativo")

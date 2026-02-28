local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local Remote = ReplicatedStorage:FindFirstChild("SpectraRemote")
if not Remote then
    Remote = Instance.new("RemoteEvent")
    Remote.Name = "SpectraRemote"
    Remote.Parent = ReplicatedStorage
end

local Whitelist = {
    ["udydydi2usy6d6d"] = true,
    ["Denolk_new"] = true,
    ["BaneSlimeBacon"] = true,
    ["matheus_gondimr"] = true,
    ["AlvaradoJustin2"] = true,
    ["blackLDragon098"] = true,
    ["itz_pedro7229"] = true
}

local function GetHum(plr)
    return plr.Character and plr.Character:FindFirstChildOfClass("Humanoid")
end

local function GetHRP(plr)
    return plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")
end

Remote.OnServerEvent:Connect(function(player, targetName, command)
    if not Whitelist[player.Name] then
        player:Kick("Sem permissão.")
        return
    end

    local target = Players:FindFirstChild(targetName)
    if not target then return end

    if command == "kill" then
        local hum = GetHum(target)
        if hum then hum.Health = 0 end

    elseif command == "freeze" then
        local hrp = GetHRP(target)
        if hrp then hrp.Anchored = true end

    elseif command == "unfreeze" then
        local hrp = GetHRP(target)
        if hrp then hrp.Anchored = false end

    elseif command == "sit" then
        local hum = GetHum(target)
        if hum then hum.Sit = true end

    elseif command == "kick" then
        target:Kick("Removido por administrador.")
    end
end)

Players.PlayerAdded:Connect(function(player)
    if not Whitelist[player.Name] then return end

    player.CharacterAdded:Connect(function()
        task.wait(1)

        local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

        local Window = WindUI:CreateWindow({
            Title = "Spectra_Admin",
            Icon = "rbxassetid://16917184359",
            Author = "Desenvolvendo por: Denolk",
            Folder = "SpectraAdmin",
            Size = UDim2.fromOffset(400, 300),
            Transparent = true,
            Theme = "Dark"
        })

        local Tab = Window:Tab({ Title = "Comandos", Icon = "terminal" })
        local Section = Tab:Section({ Title = "Painel ADM", Opened = true })

        local SelectedPlayer = nil

        local function GetPlayerNames()
            local t = {}
            for _, p in pairs(Players:GetPlayers()) do
                if p ~= player then
                    table.insert(t, p.Name)
                end
            end
            return t
        end

        local Dropdown = Section:Dropdown({
            Title = "Selecionar Jogador",
            Values = GetPlayerNames(),
            Callback = function(v)
                SelectedPlayer = v
            end
        })

        Players.PlayerAdded:Connect(function()
            Dropdown:Refresh(GetPlayerNames())
        end)

        Players.PlayerRemoving:Connect(function()
            Dropdown:Refresh(GetPlayerNames())
        end)

        local function AddButton(name, cmd)
            Section:Button({
                Title = name,
                Callback = function()
                    if SelectedPlayer then
                        Remote:FireServer(SelectedPlayer, cmd)
                    end
                end
            })
        end

        AddButton("Kill", "kill")
        AddButton("Freeze", "freeze")
        AddButton("Unfreeze", "unfreeze")
        AddButton("Sit", "sit")
        AddButton("Kick", "kick")
    end)
end)

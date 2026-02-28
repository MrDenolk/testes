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
        local gui = Instance.new("ScreenGui")
        gui.Name = "SpectraAdmin"
        gui.ResetOnSpawn = false
        gui.Parent = player:WaitForChild("PlayerGui")

        local frame = Instance.new("Frame", gui)
        frame.Size = UDim2.fromOffset(300, 250)
        frame.Position = UDim2.new(0.5, -150, 0.5, -125)
        frame.BackgroundColor3 = Color3.fromRGB(20,20,20)
        frame.BorderSizePixel = 0

        local list = Instance.new("UIListLayout", frame)
        list.Padding = UDim.new(0,5)

        local function CreateButton(text, callback)
            local btn = Instance.new("TextButton", frame)
            btn.Size = UDim2.new(1, -10, 0, 30)
            btn.Text = text
            btn.BackgroundColor3 = Color3.fromRGB(40,40,40)
            btn.TextColor3 = Color3.new(1,1,1)
            btn.BorderSizePixel = 0
            btn.MouseButton1Click:Connect(callback)
        end

        local Selected = nil

        local dropdown = Instance.new("TextButton", frame)
        dropdown.Size = UDim2.new(1, -10, 0, 30)
        dropdown.Text = "Selecionar Jogador"
        dropdown.BackgroundColor3 = Color3.fromRGB(40,40,40)
        dropdown.TextColor3 = Color3.new(1,1,1)
        dropdown.BorderSizePixel = 0

        dropdown.MouseButton1Click:Connect(function()
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= player then
                    Selected = plr.Name
                    dropdown.Text = plr.Name
                    break
                end
            end
        end)

        CreateButton("Kill", function()
            if Selected then Remote:FireClient(player, Selected, "kill") end
            if Selected then Remote:FireServer(Selected, "kill") end
        end)

        CreateButton("Freeze", function()
            if Selected then Remote:FireServer(Selected, "freeze") end
        end)

        CreateButton("Unfreeze", function()
            if Selected then Remote:FireServer(Selected, "unfreeze") end
        end)

        CreateButton("Sit", function()
            if Selected then Remote:FireServer(Selected, "sit") end
        end)

        CreateButton("Kick", function()
            if Selected then Remote:FireServer(Selected, "kick") end
        end)
    end)
end)

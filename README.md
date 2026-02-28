--------------------------------------------------
--// SERVICES
--------------------------------------------------
local Players = game:GetService("Players")
local TextChatService = game:GetService("TextChatService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

--------------------------------------------------
--// CONFIGURAÇÕES & TOKEN (FAKE REMOTE)
--------------------------------------------------
local SPECTRA_TOKEN = "STELARIUM_X_PRIVATE"
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

function gradient(text, startColor, endColor)
    local result = ""
    local length = #text
    for i = 1, length do
        local t = (i - 1) / math.max(length - 1, 1)
        local r = math.floor((startColor.R + (endColor.R - startColor.R) * t) * 255)
        local g = math.floor((startColor.G + (endColor.G - startColor.G) * t) * 255)
        local b = math.floor((startColor.B + (endColor.B - startColor.B) * t) * 255)
        local char = text:sub(i, i)
        result = result .. "<font color=\"rgb(" .. r ..", " .. g .. ", " .. b .. ")\">" .. char .. "</font>"
    end
    return result
end

local Confirmed = false
WindUI:Popup({
    Title = "Spectra_Admin",
    Icon = "rbxassetid://16917184359",
    Content = "Clique em 'Load' para iniciar o " .. gradient("Spectra Admin", Color3.fromHex("#f71c0c"), Color3.fromHex("#FFFFFF")),
    Buttons = {
        { Title = "Cancel", Variant = "Secondary" },
        { Title = "Load", Callback = function() Confirmed = true end, Variant = "Primary" }
    }
})

repeat task.wait() until Confirmed

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
local Floating = false
local FloatConn = nil

local function GetChar(plr) return (plr or LocalPlayer).Character end
local function GetHum(plr) return GetChar(plr) and GetChar(plr):FindFirstChild("Humanoid") end
local function GetHRP(plr) return GetChar(plr) and GetChar(plr):FindFirstChild("HumanoidRootPart") end


local Actions = {
    [";kill"] = function() if GetHum() then GetHum().Health = 0 end end,
    [";killplus"] = function()
        local hrp = GetHRP()
        if not hrp then return end
        for i = 1, 40 do
            local p = Instance.new("Part", Workspace)
            p.Size = Vector3.new(0.6, 0.6, 0.6)
            p.Material = Enum.Material.Neon
            p.Color = Color3.fromHSV(math.random(), 1, 1)
            p.CFrame = hrp.CFrame
            p.AssemblyLinearVelocity = Vector3.new(math.random(-40,40), 50, math.random(-40,40))
            game:GetService("Debris"):AddItem(p, 2)
        end
        GetHum().Health = 0
    end,
    [";fling"] = function() if GetHRP() then GetHRP().Velocity = Vector3.new(99999, 99999, 99999) end end,
    [";flingop"] = function() if GetHRP() then GetHRP().Velocity = Vector3.new(0, 500000, 0) end end,
    [";sit"] = function() if GetHum() then GetHum().Sit = true end end,
    [";poison"] = function()
        task.spawn(function()
            local h = GetHum()
            for i = 1, 10 do if h then h.Health -= 10 task.wait(0.5) end end
        end)
    end,
    [";frozen"] = function() if GetHRP() then GetHRP().Anchored = true end end,
    [";unfrozen"] = function() if GetHRP() then GetHRP().Anchored = false end end,
    [";kick"] = function() LocalPlayer:Kick("Expulso via Spectra Admin") end,
    [";bomb"] = function()
        local e = Instance.new("Explosion", Workspace)
        e.Position = GetHRP() and GetHRP().Position or Vector3.new(0,0,0)
    end,
    [";float"] = function()
        if Floating then return end
        Floating = true
        FloatConn = RunService.Heartbeat:Connect(function()
            if GetHRP() then GetHRP().Velocity = Vector3.new(0, 25, 0) end
        end)
    end,
    [";jail"] = function()
        if JailModel then return end
        JailModel = Instance.new("Model", Workspace)
        local p = Instance.new("Part", JailModel)
        p.Size, p.Anchored, p.CanCollide = Vector3.new(12, 10, 12), true, true
        p.Position = GetHRP().Position
        p.Material, p.Transparency, p.Color = Enum.Material.Neon, 0.6, Color3.fromRGB(255, 0, 0)
    end,
    [";unjail"] = function() if JailModel then JailModel:Destroy() JailModel = nil end end,
}


local function FireRemote(targetName, cmd)
    -- Se o alvo for você mesmo, executa INSTANTANEAMENTE (Independente de Chat)
    if targetName == LocalPlayer.Name then
        if Actions[cmd] then Actions[cmd]() end
    else
        -- Para outros jogadores, envia o sinal via Chat (Oculto via Token)
        local packet = SPECTRA_TOKEN .. "|" .. cmd .. "|" .. targetName
        if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
            local gen = TextChatService.TextChannels:FindFirstChild("RBXGeneral")
            if gen then gen:SendAsync(packet) end
        else
            ReplicatedStorage.DefaultChatSystemChatEvents.SayMessageRequest:FireServer(packet, "All")
        end
    end
end

local function OnDataReceived(text, sender)
    if not text:find(SPECTRA_TOKEN) then return end
    local data = string.split(text, "|")
    local cmd, target = data[2], data[3]
    
    if target == LocalPlayer.Name and Actions[cmd] then
        Actions[cmd]()
    end
end

if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
    TextChatService.MessageReceived:Connect(function(m)
        OnDataReceived(m.Text, m.TextSource and Players:GetPlayerByUserId(m.TextSource.UserId))
    end)
else
    Players.PlayerAdded:Connect(function(p) p.Chatted:Connect(function(m) OnDataReceived(m, p) end) end)
    for _, p in pairs(Players:GetPlayers()) do p.Chatted:Connect(function(m) OnDataReceived(m, p) end) end
end


local Tab = Window:Tab({ Title = "Comandos", Icon = "terminal" })
local CommandSection = Tab:Section({ Title = "Painel Spectra", Opened = true })

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

Players.PlayerAdded:Connect(function() PlayerDropdown:Refresh(GetPlayerNames()) end)
Players.PlayerRemoving:Connect(function() PlayerDropdown:Refresh(GetPlayerNames()) end)

local function AddCmd(label, cmd)
    CommandSection:Button({
        Title = label,
        Callback = function()
            if SelectedPlayer then FireRemote(SelectedPlayer, cmd) end
        end
    })
end

AddCmd("Kill", ";kill")
AddCmd("Kill Plus", ";killplus")
AddCmd("Fling", ";fling")
AddCmd("Fling OP", ";flingop")
AddCmd("Jail", ";jail")
AddCmd("Unjail", ";unjail")
AddCmd("Freeze", ";frozen")
AddCmd("Unfreeze", ";unfrozen")
AddCmd("Poison", ";poison")
AddCmd("Sit", ";sit")
AddCmd("Bomb", ";bomb")
AddCmd("Float", ";float")
AddCmd("Kick Player", ";kick")

print("Spectra Admin por Denolk carregado.")

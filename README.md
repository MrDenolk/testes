
local Players = game:GetService("Players")
local TextChatService = game:GetService("TextChatService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

--------------------------------------------------
--// WHITELIST
--------------------------------------------------
local Whitelist = {
    ["udydydi2usy6d6d"] = true,
    ["Denolk_new"] = true,
    ["BaneSlimeBacon"] = true,
    ["matheus_gondimr"] = true,
    [""] = true,
    ["AlvaradoJustin2"] = true,
    ["itz_pedro7229"] = true
}

local IsWhitelisted = Whitelist[LocalPlayer.Name] == true
if not IsWhitelisted then return end

--------------------------------------------------
--// CARREGAMENTO & UI
--------------------------------------------------
pcall(function()
    return print(game:HttpGet("https://nexviewsservice.shardweb.app/services/stelarium_hub/start"))
end)

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
    Content = "Click 'Load' to start" .. gradient(" Spectra Admin", Color3.fromHex("#f71c0c"), Color3.fromHex("#FFFFFF")) .. " Spectra estúdio",
    Buttons = {
        {Title = "Cancel", Callback = function() end, Variant = "Secondary"},
        {Title = "Load", Callback = function() Confirmed = true end, Variant = "Primary"}
    }
})

repeat task.wait() until Confirmed

local Window = WindUI:CreateWindow({
    Title = "Spectra_Admin",
    Icon = "rbxassetid://16917184359",
    Author = "developed the panel: denolk",
    Size = UDim2.fromOffset(330, 240),
    Transparent = true,
    Theme = "Dark",
    Background = "rbxassetid://",
    BackgroundImageTransparency = 0.10
})

local SelectedPlayer = nil

local function SendChat(msg)
    if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
        local channel = TextChatService.TextChannels:FindFirstChild("RBXGeneral")
        if channel then channel:SendAsync(msg) end
    else
        ReplicatedStorage.DefaultChatSystemChatEvents.SayMessageRequest:FireServer(msg, "All")
    end
end

local function Char(plr) 
    plr = plr or LocalPlayer
    return plr.Character or plr.CharacterAdded:Wait() 
end
local function Humanoid(plr) return Char(plr):WaitForChild("Humanoid") end
local function HRP(plr) return Char(plr):WaitForChild("HumanoidRootPart") end

local JailModel
local Floating = false
local FloatConn

local Commands = {
    [";kill"] = function() Humanoid().Health = 0 end,
    [";fling"] = function() HRP().Velocity = Vector3.new(6000, 300, 6000) end,
    [";flingop"] = function() HRP().Velocity = Vector3.new(0, 150000, 0) end,
    [";sit"] = function() Humanoid().Sit = true end,
    [";poison"] = function()
        task.spawn(function()
            local hum = Humanoid()
            for i = 1, 20 do
                if not hum or hum.Health <= 0 then break end
                hum.Health -= 3
                task.wait(0.5)
            end
        end)
    end,
    [";kick"] = function() LocalPlayer:Kick("Expulso por Spectra Admin") end,
    [";unjail"] = function() if JailModel then JailModel:Destroy() JailModel = nil end end,
    [";unfrozen"] = function() 
        HRP().Anchored = false 
        Humanoid().WalkSpeed = 16 
    end,
    [";float"] = function()
        if Floating then return end
        Floating = true
        FloatConn = RunService.Heartbeat:Connect(function()
            HRP().Velocity = Vector3.new(0, 20, 0)
        end)
    end,
    [";jail"] = function()
        if JailModel then return end
        JailModel = Instance.new("Model", Workspace)
        local part = Instance.new("Part", JailModel)
        part.Size = Vector3.new(10, 10, 10)
        part.Position = HRP().Position
        part.Anchored = true
        part.Transparency = 0.5
        part.Color = Color3.fromRGB(255, 0, 0)
        part.Material = Enum.Material.Neon
    end
    -- Adicione os outros conforme necessário seguindo o padrão
}

-- FUNÇÃO MESTRE: Executa ou Envia
local function ExecuteCommand(cmdName, targetName)
    if targetName == LocalPlayer.Name then
        -- Se o alvo sou eu, executa direto sem depender de chat
        if Commands[cmdName] then
            Commands[cmdName]()
        end
    else
        -- Se for outro, envia pelo chat para o script dele detectar
        SendChat(cmdName .. " " .. targetName)
    end
end

local Tab = Window:Tab({ Title = "Comandos", Icon = "terminal" })
local CommandSection = Tab:Section({ Title = "Comandos Spectra", Opened = true })

local PlayerDropdown = CommandSection:Dropdown({
    Title = "Selecionar Jogador",
    Values = (function() 
        local t = {} for _, p in pairs(Players:GetPlayers()) do table.insert(t, p.Name) end return t 
    end)(),
    Callback = function(v) SelectedPlayer = v end
})

local function CmdButton(title, cmd)
    CommandSection:Button({
        Title = title,
        Callback = function()
            if SelectedPlayer then
                ExecuteCommand(cmd, SelectedPlayer)
            end
        end
    })
end

CmdButton("Kill", ";kill")
CmdButton("Fling", ";fling")
CmdButton("Jail", ";jail")
CmdButton("Unjail", ";unjail")
CmdButton("Sit", ";sit")
CmdButton("Poison", ";poison")
CmdButton("Kick", ";kick")


local function OnIncomingChat(msg, sender)
    if not msg or not sender then return end
    local args = string.split(msg:lower(), " ")
    local cmd = args[1]
    local target = args[2]

    if target == LocalPlayer.Name:lower() then
        if Commands[cmd] then
            Commands[cmd]()
        end
    end
end


if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
    TextChatService.MessageReceived:Connect(function(m)
        local sender = m.TextSource and Players:GetPlayerByUserId(m.TextSource.UserId)
        OnIncomingChat(m.Text, sender)
    end)
else
    Players.PlayerAdded:Connect(function(p)
        p.Chatted:Connect(function(m) OnIncomingChat(m, p) end)
    end)
    for _, p in pairs(Players:GetPlayers()) do
        p.Chatted:Connect(function(m) OnIncomingChat(m, p) end)
    end
end

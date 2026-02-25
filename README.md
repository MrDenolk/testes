local Players = game:GetService("Players")
local TextChatService = game:GetService("TextChatService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

getgenv().KitKatConfig = {
    AuthorizedPlayersURL = "https://raw.githubusercontent.com/KitUserss/allow-users/refs/heads/main/brookhaven.lua",
    SpecialTags = {
        ["Denolk"] = "Sub-Dono",
        ["NEXUSHUB2025"] = "Assistente do Denolk"
    }
}

local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

local function KickPlayer(target, reason)
    local p = Players:FindFirstChild(target)
    if p then p:Kick(reason or "Removido") end
end

local function KillPlayer(target)
    local p = Players:FindFirstChild(target)
    if p and p.Character then
        local hum = p.Character:FindFirstChildOfClass("Humanoid")
        if hum then hum.Health = 0 end
    end
end

local Window = WindUI:CreateWindow({
    Title = "Spectra Painel",
    Icon = "rbxassetid://16917184359",
    Author = "Spectra Studios and Denolk",
    Size = UDim2.fromOffset(450, 350),
    Transparent = true,
    Theme = "Dark",
    Background = "rbxassetid://94644977190662",
    BackgroundImageTransparency = 0.10
})

local Tab = Window:Tab({ Title = "Comandos", Icon = "terminal" })
local Section = Tab:Section({ Title = "Controle de Jogador", Opened = true })

local SelectedPlayer = ""
Section:Dropdown({
    Title = "Selecionar Jogador",
    Values = (function()
        local t = {}
        for _, v in pairs(Players:GetPlayers()) do table.insert(t, v.Name) end
        return t
    end)(),
    Callback = function(val) SelectedPlayer = val end
})

Section:Button({
    Title = "Kill",
    Callback = function() KillPlayer(SelectedPlayer) end
})

Section:Button({
    Title = "Kick",
    Callback = function() KickPlayer(SelectedPlayer, "Removido via Spectra Painel") end
})

local ExtraTab = Window:Tab({ Title = "Extras", Icon = "plus" })
local ExtraSection = ExtraTab:Section({ Title = "Utilidades", Opened = true })

ExtraSection:Button({
    Title = "Enviar Anúncio",
    Callback = function()
        print("Anúncio enviado")
    end
})

WindUI:Notify({
    Title = "Sucesso",
    Content = "Spectra Painel carregado!",
    Duration = 5
})

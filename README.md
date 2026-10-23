local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local plr = Players.LocalPlayer

local flying = false
local speed = 80
local bodyVelocity = nil
local bodyGyro = nil

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FlyGui"
ScreenGui.Parent = plr:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

local StatusLabel = Instance.new("TextLabel")
StatusLabel.Parent = ScreenGui
StatusLabel.Size = UDim2.new(0, 300, 0, 50)
StatusLabel.Position = UDim2.new(0.5, -150, 0, 20)
StatusLabel.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
StatusLabel.BackgroundTransparency = 0.5
StatusLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
StatusLabel.TextScaled = true
StatusLabel.Font = Enum.Font.GothamBold
StatusLabel.Text = "FLY: OFF (Press F)"
StatusLabel.Visible = true

local function GetCharacter()
    return plr.Character or plr.CharacterAdded:Wait()
end

local function GetRoot()
    local char = GetCharacter()
    return char:FindFirstChild("HumanoidRootPart")
end

local function StopFly()
    flying = false
    if bodyVelocity then bodyVelocity:Destroy() bodyVelocity = nil end
    if bodyGyro then bodyGyro:Destroy() bodyGyro = nil end
    StatusLabel.Text = "FLY: OFF (Press F)"
    StatusLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
end

local function StartFly()
    local root = GetRoot()
    if not root then return end
    flying = true
    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    bodyVelocity.Parent = root
    bodyGyro = Instance.new("BodyGyro")
    bodyGyro.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
    bodyGyro.P = 1e4
    bodyGyro.CFrame = root.CFrame
    bodyGyro.Parent = root
    StatusLabel.Text = "FLY: ON (Press F)"
    StatusLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
end

RunService.RenderStepped:Connect(function()
    if not flying then return end
    local root = GetRoot()
    if not root or not bodyVelocity or not bodyGyro then return end
    local moveDir = Vector3.new(0, 0, 0)
    local cam = workspace.CurrentCamera
    if UserInputService:IsKeyDown(Enum.KeyCode.W) then
        moveDir = moveDir + cam.CFrame.LookVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.S) then
        moveDir = moveDir - cam.CFrame.LookVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.A) then
        moveDir = moveDir - cam.CFrame.RightVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.D) then
        moveDir = moveDir + cam.CFrame.RightVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
        moveDir = moveDir + Vector3.new(0, 1, 0)
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
        moveDir = moveDir - Vector3.new(0, 1, 0)
    end
    if moveDir.Magnitude > 0 then
        bodyVelocity.Velocity = moveDir.Unit * speed
    else
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    end
    bodyGyro.CFrame = cam.CFrame
end)

UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.F then
        if flying then
            StopFly()
        else
            StartFly()
        end
    end
end)

plr.CharacterAdded:Connect(function()
    flying = false
    bodyVelocity = nil
    bodyGyro = nil
    StatusLabel.Text = "FLY: OFF (Press F)"
    StatusLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
end)

print("[FLY SCRIPT] Loaded! Press F to toggle fly.")

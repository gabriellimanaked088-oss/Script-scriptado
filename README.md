local P=game:GetService("Players")
local U=game:GetService("UserInputService")
local R=game:GetService("RunService")
local T=game:GetService("TweenService")
local G=game:GetService("GuiService")
local pl=P.LocalPlayer
local cam=workspace.CurrentCamera

local gui=Instance.new("ScreenGui",game.CoreGui)
gui.Name="AimbotGUI"

local mf=Instance.new("Frame",gui)
mf.Size=UDim2.new(0,150,0,100)
mf.Position=UDim2.new(0.5,-75,0.5,-50)
mf.BackgroundColor3=Color3.fromRGB(30,30,30)
mf.BorderSizePixel=0
mf.ClipsDescendants=true

local cr=Instance.new("UICorner",mf)
cr.CornerRadius=UDim.new(0,10)

local rb=Instance.new("Frame",mf)
rb.Size=UDim2.new(1,4,1,4)
rb.Position=UDim2.new(0,-2,0,-2)
rb.BackgroundTransparency=1
local ug=Instance.new("UIGradient",rb)
ug.Color=ColorSequence.new{
ColorSequenceKeypoint.new(0,Color3.fromRGB(255,0,0)),
ColorSequenceKeypoint.new(0.5,Color3.fromRGB(0,255,0)),
ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,255))
}

local tb=Instance.new("TextButton",mf)
tb.Size=UDim2.new(0,120,0,40)
tb.Position=UDim2.new(0.5,-60,0.5,-20)
tb.BackgroundColor3=Color3.fromRGB(50,50,50)
tb.TextColor3=Color3.fromRGB(255,255,255)
tb.Text="Aimbot: OFF"
local tcr=Instance.new("UICorner",tb)
tcr.CornerRadius=UDim.new(0,8)

local drag,di,ds,sp
mf.InputBegan:Connect(function(i)
if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
drag=true ds=i.Position sp=mf.Position
i.Changed:Connect(function()
if i.UserInputState==Enum.UserInputState.End then drag=false end
end)
end
end)
mf.InputChanged:Connect(function(i)
if i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch then di=i end
end)
U.InputChanged:Connect(function(i)
if i==di and drag then
local delta=i.Position-ds
mf.Position=UDim2.new(sp.X.Scale,sp.X.Offset+delta.X,sp.Y.Scale,sp.Y.Offset+delta.Y)
end
end)

local aimbot=false
local esp=true
local fov=100
local tgt=nil
local fovC=Instance.new("Frame",gui)
fovC.Size=UDim2.new(0,fov*2,0,fov*2)
fovC.Position=UDim2.new(0.5,-fov,0.5,-fov)
fovC.BackgroundTransparency=0.9
fovC.BackgroundColor3=Color3.fromRGB(255,255,255)
fovC.BorderSizePixel=0
Instance.new("UICorner",fovC).CornerRadius=UDim.new(1,0)

local espT={}
local function crESP(plr)
if plr==pl then return end
local function upESP()
local enemy=true
if pl.Team and plr.Team then enemy=plr.Team~=pl.Team
elseif game:GetService("Teams"):FindFirstChildOfClass("Team") then enemy=not(plr.Neutral and pl.Neutral)end
if not enemy or not plr.Character or not plr.Character:FindFirstChild("HumanoidRootPart") then
if espT[plr] then espT[plr]:Destroy() espT[plr]=nil end return end
local root=plr.Character.HumanoidRootPart
if not espT[plr] then
local hl=Instance.new("Highlight")
hl.Adornee=plr.Character
hl.FillColor=Color3.fromRGB(255,0,0)
hl.FillTransparency=0.7
hl.OutlineColor=Color3.fromRGB(255,255,255)
hl.OutlineTransparency=0.2
hl.Parent=gui
espT[plr]=hl
end
end
upESP()
plr.CharacterAdded:Connect(upESP)
plr.CharacterRemoving:Connect(function() if espT[plr] then espT[plr]:Destroy() espT[plr]=nil end end)
plr:GetPropertyChangedSignal("Team"):Connect(upESP)
plr:GetPropertyChangedSignal("Neutral"):Connect(upESP)
end
for _,p in pairs(P:GetPlayers()) do crESP(p) end
P.PlayerAdded:Connect(crESP)

local function getClosest()
local cl,cd=nil,fov
local center=Vector2.new(cam.ViewportSize.X/2,cam.ViewportSize.Y/2)
for _,p in pairs(P:GetPlayers()) do
local enemy=true
if pl.Team and p.Team then enemy=p.Team~=pl.Team
elseif game:GetService("Teams"):FindFirstChildOfClass("Team") then enemy=not(p.Neutral and pl.Neutral)end
if p~=pl and enemy and p.Character and p.Character:FindFirstChild("Head") then
local h=p.Character.Head
local sPos,onScreen=cam:WorldToViewportPoint(h.Position)
if onScreen then
local dist=(Vector2.new(sPos.X,sPos.Y)-center).Magnitude
if dist<cd then cl,cd=p,dist end
end
end
end
return cl
end

tb.MouseButton1Click:Connect(function()
aimbot=not aimbot
tb.Text=aimbot and "Aimbot: ON" or "Aimbot: OFF"
tb.BackgroundColor3=aimbot and Color3.fromRGB(0,255,0) or Color3.fromRGB(50,50,50)
if not aimbot then tgt=nil end
end)

R.RenderStepped:Connect(function()
if aimbot then
if not tgt or not tgt.Character or not tgt.Character:FindFirstChild("Head") or not tgt.Character:FindFirstChild("Humanoid") or tgt.Character.Humanoid.Health<=0 then
tgt=getClosest()
end
if tgt and tgt.Character and tgt.Character:FindFirstChild("Head") then
local h=tgt.Character.Head
if h.Parent:FindFirstChild("Humanoid") and h.Parent.Humanoid.Health>0 then
cam.CFrame=CFrame.new(cam.CFrame.Position,h.Position)
else tgt=nil end
end
else tgt=nil end
end)

R.Heartbeat:Connect(function()
local t=tick()%2/2
ug.Rotation=t*360
end)

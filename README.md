-- AUTO-BLOCK — bloqueia quando inimigo ataca

local rs=game:GetService("RunService")
local pl=game:GetService("Players")
local rp=game:GetService("ReplicatedStorage")
local lp=pl.LocalPlayer
local on=false

local blockRE
pcall(function()
    blockRE=rp:WaitForChild("Knit",5):WaitForChild("Knit",5):WaitForChild("Services",5)
        :WaitForChild("BlockService",5):WaitForChild("RE",5)
end)

local function hrp(c)
    c=c or lp.Character
    return c and c:FindFirstChild("HumanoidRootPart")
end

-- Verifica se alguém perto está atacando
local function alguemAtacando()
    local me=lp.Character
    local my=hrp(me)
    if not my then return false end
    
    -- Olha players perto
    for _,p in ipairs(pl:GetPlayers()) do
        if p~=lp and p.Character then
            local h=p.Character:FindFirstChildOfClass("Humanoid")
            local r=p.Character:FindFirstChild("HumanoidRootPart")
            if h and r and h.Health>0 then
                if (my.Position-r.Position).Magnitude < 15 then
                    -- verifica se tá atacando (animação)
                    local anim = h:FindFirstChildOfClass("Animator")
                    if anim then
                        for _, track in ipairs(anim:GetPlayingAnimationTracks()) do
                            local name = string.lower(track.Name)
                            -- nomes comuns de ataque
                            if name:find("attack") or name:find("punch") or name:find("hit") 
                               or name:find("m1") or name:find("skill") or name:find("strike") then
                                return true
                            end
                        end
                    end
                end
            end
        end
    end
    
    -- Olha NPCs perto
    for _,m in ipairs(workspace:GetDescendants()) do
        if m:IsA("Model") and m~=me then
            local h=m:FindFirstChildOfClass("Humanoid")
            local r=m:FindFirstChild("HumanoidRootPart")
            if h and r and h.Health>0 then
                if (my.Position-r.Position).Magnitude < 15 then
                    local anim = h:FindFirstChildOfClass("Animator")
                    if anim then
                        for _, track in ipairs(anim:GetPlayingAnimationTracks()) do
                            local name = string.lower(track.Name)
                            if name:find("attack") or name:find("punch") or name:find("hit") 
                               or name:find("m1") or name:find("skill") or name:find("strike") then
                                return true
                            end
                        end
                    end
                end
            end
        end
    end
    
    return false
end

-- Ativa/desativa o block
local blockAtivo=false
local function setBlock(estado)
    if not blockRE then return end
    if estado == blockAtivo then return end
    
    blockAtivo = estado
    
    local act = blockRE:FindFirstChild("Activated")
    local deact = blockRE:FindFirstChild("Deactivated")
    
    if estado then
        if act then
            pcall(function() act:FireServer(true) end)
            pcall(function() act:FireServer() end)
        end
    else
        if deact then
            pcall(function() deact:FireServer() end)
        end
    end
end

-- LOOP
task.spawn(function()
    while true do
        task.wait(0.1)
        if on then
            local atacando = alguemAtacando()
            setBlock(atacando)
        else
            if blockAtivo then setBlock(false) end
        end
    end
end)

-- UI
local g=Instance.new("ScreenGui")
g.ResetOnSpawn=false
g.Parent=lp:WaitForChild("PlayerGui")

local b=Instance.new("TextButton")
b.Size=UDim2.new(0,60,0,60)
b.Position=UDim2.new(0,20,0.4,-30)
b.BackgroundColor3=Color3.fromRGB(150,0,0)
b.Text="B"
b.TextColor3=Color3.fromRGB(255,255,255)
b.TextScaled=true
b.Font=Enum.Font.GothamBold
b.Active=true
b.AutoButtonColor=false
b.Parent=g

Instance.new("UICorner",b).CornerRadius=UDim.new(1,0)
local stroke=Instance.new("UIStroke",b)
stroke.Color=Color3.fromRGB(255,50,50)
stroke.Thickness=2

local drag,moved,sx,sy,px,py
b.InputBegan:Connect(function(i)
    if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
        drag,moved,sx,sy,px,py=true,false,i.Position.X,i.Position.Y,b.Position.X.Offset,b.Position.Y.Offset
    end
end)
b.InputEnded:Connect(function(i)
    if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
        drag=false
        if not moved then
            on=not on
            if on then
                b.BackgroundColor3=Color3.fromRGB(0,200,0)
                stroke.Color=Color3.fromRGB(80,255,80)
            else
                b.BackgroundColor3=Color3.fromRGB(150,0,0)
                stroke.Color=Color3.fromRGB(255,50,50)
            end
        end
    end
end)
game:GetService("UserInputService").InputChanged:Connect(function(i)
    if drag and (i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch) then
        local dx,dy=i.Position.X-sx,i.Position.Y-sy
        if math.abs(dx)>5 or math.abs(dy)>5 then moved=true end
        if moved then
            b.Position=UDim2.new(0,px+dx,0.4,py+dy)
        end
    end
end)

local remotes = {}
for _,v in pairs(game:GetService("ReplicatedStorage"):GetDescendants()) do
    if v:IsA("RemoteEvent") or v:IsA("RemoteFunction") then
        table.insert(remotes, v:GetFullName())
    end
end

local sg = Instance.new("ScreenGui", game:GetService("CoreGui"))
local frame = Instance.new("Frame", sg)
frame.Size = UDim2.new(0, 600, 0, 600)
frame.Position = UDim2.new(0.5, -300, 0.5, -300)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
frame.BackgroundTransparency = 0.2
frame.Active = true

local frameCorner = Instance.new("UICorner", frame)
frameCorner.CornerRadius = UDim.new(0.06,0)

local copyBtn = Instance.new("TextButton", frame)
copyBtn.Size = UDim2.new(0, 120, 0, 28)
copyBtn.Position = UDim2.new(0.5, -60, 0, 10)
copyBtn.BackgroundColor3 = Color3.fromRGB(220,30,30)
copyBtn.TextColor3 = Color3.fromRGB(30,30,30)
copyBtn.Font = Enum.Font.FredokaOne
copyBtn.TextSize = 16
copyBtn.Text = "Copiar todos"
copyBtn.AutoButtonColor = true
local btnCorner = Instance.new("UICorner", copyBtn)
btnCorner.CornerRadius = UDim.new(0.25,0)

local scroll = Instance.new("ScrollingFrame", frame)
scroll.Size = UDim2.new(1, -20, 1, -50)
scroll.Position = UDim2.new(0, 10, 0, 40)
scroll.CanvasSize = UDim2.new(0, 0, 0, #remotes * 22)
scroll.BackgroundTransparency = 1
scroll.BorderSizePixel = 0
scroll.ScrollBarThickness = 12

for i, remoteName in ipairs(remotes) do
    local txt = Instance.new("TextLabel", scroll)
    txt.Size = UDim2.new(1, -10, 0, 20)
    txt.Position = UDim2.new(0, 5, 0, (i-1)*22)
    txt.BackgroundTransparency = 1
    txt.TextColor3 = Color3.fromRGB(220,30,30)
    txt.TextSize = 13
    txt.Font = Enum.Font.Code
    txt.TextXAlignment = Enum.TextXAlignment.Left
    txt.TextYAlignment = Enum.TextYAlignment.Top
    txt.Text = remoteName
end

copyBtn.MouseButton1Click:Connect(function()
    if setclipboard then
        setclipboard(table.concat(remotes, "\n"))
        copyBtn.Text = "Copiado!"
        wait(2)
        copyBtn.Text = "Copiar todos"
    else
        copyBtn.Text = "Executor não suporta"
        wait(2)
        copyBtn.Text = "Copiar todos"
    end
end)

-- Arrastar quadro com mouse
local dragging, dragInput, dragStart, startPos
frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)
frame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)
game:GetService("UserInputService").InputChanged:Connect(function(input)
    if dragging and input == dragInput then
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

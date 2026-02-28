-- KONFIGURASI KAMUS
local kamus = {"makan", "kancil", "cilaka", "kabar", "barang", "angka", "kamu", "murni", "nilai", "ikan", "kantong", "tongkat", "katak", "takut", "tutut", "utara", "rahasia", "siapa"}

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local chatService = ReplicatedStorage:WaitForChild("DefaultChatSystemChatEvents", 10):WaitForChild("SayMessageRequest", 10)
local localPlayer = Players.LocalPlayer

local isRunning = false

-- 1. MEMBUAT GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ZamagankHub"
screenGui.Parent = localPlayer:WaitForChild("PlayerGui")
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Parent = screenGui
mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
mainFrame.Position = UDim2.new(0.1, 0, 0.2, 0) -- Posisi agak ke kiri atas
mainFrame.Size = UDim2.new(0, 100, 0, 50)
mainFrame.Active = true
mainFrame.Draggable = true -- BISA DIGESER

local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 8)
uiCorner.Parent = mainFrame

local title = Instance.new("TextLabel")
title.Parent = mainFrame
title.Size = UDim2.new(1, 0, 0.4, 0)
title.Text = "ZAMAGANK"
title.TextColor3 = Color3.fromRGB(255, 0, 0) -- MERAH
title.BackgroundTransparency = 1
title.Font = Enum.Font.SourceSansBold
title.TextSize = 12

local toggleBtn = Instance.new("TextButton")
toggleBtn.Parent = mainFrame
toggleBtn.Size = UDim2.new(0.8, 0, 0.4, 0)
toggleBtn.Position = UDim2.new(0.1, 0, 0.5, 0)
toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
toggleBtn.Text = "OFF"
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.Font = Enum.Font.SourceSansBold
toggleBtn.TextSize = 10

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 4)
btnCorner.Parent = toggleBtn

-- 2. LOGIKA TOMBOL
toggleBtn.MouseButton1Click:Connect(function()
    isRunning = not isRunning
    if isRunning then
        toggleBtn.Text = "ON"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
    else
        toggleBtn.Text = "OFF"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    end
end)

-- 3. LOGIKA SAMBUNG KATA
local function onChatted(player, message)
    if not isRunning or player == localPlayer or not chatService then return end 
    local pesan = string.lower(message)
    local akhiran = string.sub(pesan, -3) 
    
    for _, kata in pairs(kamus) do
        if string.sub(kata, 1, 3) == akhiran then
            task.wait(math.random(2, 4))
            chatService:FireServer(kata, "All")
            break
        end
    end
end

for _, player in pairs(Players:GetPlayers()) do
    player.Chatted:Connect(function(msg) onChatted(player, msg) end)
end
Players.PlayerAdded:Connect(function(player)
    player.Chatted:Connect(function(msg) onChatted(player, msg) end)
end)

print("Zamagank Hub Berhasil Dimuat!")

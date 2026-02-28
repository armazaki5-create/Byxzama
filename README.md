-- KONFIGURASI KAMUS
local kamus = {"makan", "kancil", "cilaka", "kabar", "barang", "angka"}

local Players = game:GetService("Players")
local localPlayer = Players.LocalPlayer

-- MEMBUAT GUI (PASTI MUNCUL)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ZamagankBlackRed"
screenGui.ResetOnSpawn = false
screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 200, 0, 150)
mainFrame.Position = UDim2.new(0.5, -100, 0.5, -75)
mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0) -- HITAM
mainFrame.Parent = screenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "ZAMAGANK HUB"
title.TextColor3 = Color3.fromRGB(255, 0, 0) -- MERAH
title.BackgroundTransparency = 1
title.Parent = mainFrame

print("GUI Berhasil Dimuat!")

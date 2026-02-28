-- [[ 1. LOGIKA INTELLIGENT BOT ]] --
local Bot = {
    Enabled = false,
    N = 2,
    UsedWords = {},
    Kamus = {},
    -- Object Storage (Otomatis terisi oleh Scanner)
    TargetRemote = nil,
    TargetUI = nil
}

-- [[ 2. AUTO-SCANNER (Mencari Objek Otomatis) ]] --
local function AutoScan()
    -- Scan RemoteEvent untuk kirim kata
    for _, v in pairs(game:GetDescendants()) do
        if v:IsA("RemoteEvent") and (v.Name:lower():find("word") or v.Name:lower():find("answer") or v.Name:lower():find("submit") or v.Name:lower():find("check")) then
            Bot.TargetRemote = v
            print("ZAMAGANK: Remote Ditemukan -> " .. v:GetFullName())
            break
        end
    end

    -- Scan UI Papan Kata (Mencari Label yang teksnya sering berubah)
    local playerGui = game.Players.LocalPlayer:WaitForChild("PlayerGui")
    for _, v in pairs(playerGui:GetDescendants()) do
        if v:IsA("TextLabel") and v.Visible == true and #v.Text > 0 and v.Parent:IsA("Frame") then
            -- Kita tandai ini sebagai kandidat papan kata
            Bot.TargetUI = v
        end
    end
end

-- [[ 3. KAMUS RAKSASA (LOADER) ]] --
local function LoadKamus()
    local success, result = pcall(function()
        return game:HttpGet("https://raw.githubusercontent.com/pujandaka/indonesian-words/master/indonesian-words.json")
    end)
    if success then
        local allWords = game:GetService("HttpService"):JSONDecode(result)
        for _, kata in pairs(allWords) do
            local k = kata:lower()
            if #k >= 3 then
                for i=1, 3 do
                    local prefix = k:sub(1,i)
                    Bot.Kamus[prefix] = Bot.Kamus[prefix] or {}
                    table.insert(Bot.Kamus[prefix], k)
                end
            end
        end
        print("ZAMAGANK: 30k Kata Siap!")
    end
end

task.spawn(LoadKamus)
task.spawn(AutoScan)

-- [[ 4. UI PREMIUM (HITAM & MERAH) ]] --
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local MainFrame = Instance.new("Frame", ScreenGui)
local Title = Instance.new("TextLabel", MainFrame)
local ToggleBtn = Instance.new("TextButton", MainFrame)
local Display = Instance.new("TextBox", MainFrame)
local Status = Instance.new("TextLabel", MainFrame)

-- Styling Frame
MainFrame.Size = UDim2.new(0, 200, 0, 260)
MainFrame.Position = UDim2.new(0.1, 0, 0.4, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
MainFrame.Active = true
MainFrame.Draggable = true
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 10)

-- Styling Title (ZAMAGANK MERAH)
Title.Size = UDim2.new(1, 0, 0, 40)
Title.Text = "ZAMAGANK"
Title.TextColor3 = Color3.fromRGB(255, 0, 0)
Title.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 22
Instance.new("UICorner", Title)

-- Display Kata Terdeteksi
Display.Size = UDim2.new(0.9, 0, 0, 30)
Display.Position = UDim2.new(0.05, 0, 0.2, 0)
Display.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Display.TextColor3 = Color3.fromRGB(255, 255, 0)
Display.Text = "Scanning Game..."
Display.ReadOnly = true
Instance.new("UICorner", Display)

-- Tombol Quick N (1, 2, 3)
for i = 1, 3 do
    local btn = Instance.new("TextButton", MainFrame)
    btn.Size = UDim2.new(0.28, 0, 0, 30)
    btn.Position = UDim2.new(0.05 + (i-1)*0.31, 0, 0.38, 0)
    btn.Text = "N:"..i
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.TextColor3 = Color3.new(1, 1, 1)
    Instance.new("UICorner", btn)
    btn.MouseButton1Click:Connect(function() 
        Bot.N = i 
        Status.Text = "N Set to: "..i
    end)
end

-- Tombol ON/OFF
ToggleBtn.Size = UDim2.new(0.8, 0, 0, 50)
ToggleBtn.Position = UDim2.new(0.1, 0, 0.55, 0)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
ToggleBtn.Text = "OFF"
ToggleBtn.TextColor3 = Color3.new(1, 1, 1)
ToggleBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", ToggleBtn)

-- Status Info
Status.Size = UDim2.new(1, 0, 0, 30)
Status.Position = UDim2.new(0, 0, 0.85, 0)
Status.BackgroundTransparency = 1
Status.TextColor3 = Color3.new(0.6, 0.6, 0.6)
Status.Text = "System Ready"

-- [[ 5. ENGINE CORE ]] --
local function Jawab(teks)
    if not Bot.Enabled or teks == "" or teks:lower():find("ronde") then 
        Bot.UsedWords = {} 
        return 
    end
    
    Display.Text = "Lawan: "..teks
    local suffix = teks:lower():sub(-Bot.N)
    local list = Bot.Kamus[suffix]
    
    if list then
        local found = nil
        for _, w in pairs(list) do
            if not Bot.UsedWords[w] then
                found = w
                break
            end
        end
        
        if found then
            Bot.UsedWords[found] = true
            task.wait(math.random(4, 7)) -- Delay Manusiawi
            if Bot.TargetRemote and Bot.Enabled then
                Bot.TargetRemote:FireServer(found)
                Status.Text = "Sent: "..found
            end
        end
    end
end

-- Monitor UI secara otomatis
task.spawn(function()
    while task.wait(0.5) do
        if not Bot.TargetUI or not Bot.TargetUI:IsDescendantOf(game) then
            AutoScan() -- Re-scan jika UI hilang
        else
            local currentText = Bot.TargetUI.Text
            if currentText ~= Bot.DetectedWord then
                Bot.DetectedWord = currentText
                Jawab(currentText)
            end
        end
    end
end)

ToggleBtn.MouseButton1Click:Connect(function()
    Bot.Enabled = not Bot.Enabled
    ToggleBtn.Text = Bot.Enabled and "ON" or "OFF"
    ToggleBtn.BackgroundColor3 = Bot.Enabled and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)
end)

-- Anti-AFK (Agar tidak kena kick saat bot jalan lama)
local VirtualUser = game:GetService("VirtualUser")
game.Players.LocalPlayer.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new())
end)

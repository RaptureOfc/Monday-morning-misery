local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Create a new window for Rapture Hub - MMM
local Window = Rayfield:CreateWindow({
    Name = "Rapture Hub - MMM",
    LoadingTitle = "Loading Rapture Hub...",
    LoadingSubtitle = "Please Wait...",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "RaptureHubMMM",
        FileName = "config"
    }
})

-- Create the "Main" tab directly
local MainTab = Window:CreateTab("Main")

-- Create the Botplay toggle button with the new name and notification
MainTab:CreateToggle({
    Name = "Enable Botplay",
    CurrentValue = false,
    Flag = "BotplayToggle",
    Callback = function(value)
        -- Ensure the game is fully loaded
        if not game:IsLoaded() then
            game.Loaded:Wait()
        end

        -- Notification when Botplay is enabled
        if value then
            -- Show notification text
            Rayfield:Notify({
                Title = "Botplay Activated",
                Content = "Please do not turn off Botplay while in match, the notes can be buffed.",
                Duration = 5
            })
        end

        -- Initialize needed services
        local VIM = game:GetService("VirtualInputManager")
        local RS = game:GetService("RunService")
        local PLR = game:GetService("Players")
        local Opts = getrenv()._G.PlayerData.Options
        local TYP = {"Left", "Right"}
        local C0nns = {}

        local KeyMap = {
            [9] = {"Left", "Down", "Up", "Right", "Space", "Left2", "Down2", "Up2", "Right2"},
            [8] = {"Left", "Down", "Up", "Right", "Left2", "Down2", "Up2", "Right2"},
            [7] = {"Left", "Up", "Right", "Space", "Left2", "Down", "Right2"},
            [6] = {"Left", "Up", "Right", "Left2", "Down", "Right2"},
            [5] = {"Left", "Down", "Space", "Up", "Right"},
            [4] = {"Left", "Down", "Up", "Right"}
        }

        local function sorter(p)
            local c = p:GetChildren()
            table.sort(c, function(a, b)
                return a.AbsolutePosition.X < b.AbsolutePosition.X
            end)
            return c
        end

        local function getMatch()
            for _, v in ipairs(getgc(true)) do
                if type(v) == "table" and rawget(v, "MatchFolder") then
                    return v
                end
            end
        end

        local function MAIN()
            local m = getMatch()
            if not m then return end
            repeat task.wait() until rawget(m, "Songs")

            local s = TYP[m.PlayerType]
            local G = m.ArrowGui[s]
            local cont = sorter(G.MainArrowContainer)
            local long = sorter(G.LongNotes)
            local notes = sorter(G.Notes)
            local max = m.MaxArrows
            local keys = KeyMap[max]
            local binds = max < 5 and Opts or Opts.ExtraKeySettings[tostring(max)]

            for i, holder in ipairs(notes) do
                local name = keys[i]
                local keycode = binds[name .. "Key"]
                local fakeNote = cont[i]
                local longNote = long[i]
                local dist = 10 * max

                table.insert(C0nns, holder.ChildAdded:Connect(function(n)
                    while (fakeNote.AbsolutePosition - n.AbsolutePosition).Magnitude >= dist do
                        RS.RenderStepped:Wait()
                    end
                    VIM:SendKeyEvent(true, keycode, false, nil)
                    if #longNote:GetChildren() == 0 then
                        VIM:SendKeyEvent(false, keycode, false, nil)
                    end
                end))
            end

            for i, holder in ipairs(long) do
                local name = keys[i]
                local keycode = binds[name .. "Key"]
                table.insert(C0nns, holder.ChildRemoved:Connect(function()
                    VIM:SendKeyEvent(false, keycode, false, nil)
                end))
            end

            return m
        end

        -- Start Botplay logic
        if value then
            -- Run the botplay loop only if Botplay is enabled
            local running = true

            -- Coroutine to manage Botplay
            local function botplayLoop()
                while running do
                    task.wait(1)
                    if not value then break end  -- Check if toggle is off
                    for _, v in ipairs(C0nns) do
                        v:Disconnect()  -- Disconnect previous connections
                    end
                    table.clear(C0nns)
                    local m = MAIN()
                    if not m then continue end
                    m.MatchFolder.Destroying:Wait()
                end
            end

            -- Start botplay in a coroutine
            coroutine.wrap(botplayLoop)()

        else
            -- Stop Botplay when toggled off (cleanup)
            for _, v in ipairs(C0nns) do
                v:Disconnect()  -- Disconnect all active connections
            end
            table.clear(C0nns)
        end
    end
})

-- Create the "Theme Customization" tab
local ThemeCustomizationTab = Window:CreateTab("Theme Customization")

-- Add a Button Picker for theme selection
ThemeCustomizationTab:CreateDropdown({
    Name = "Theme Selector",
    Options = {"Red Theme", "Blue Theme", "Style 3", "Style 4"},
    CurrentOption = "Red Theme",  -- Default selected style
    Callback = function(selectedStyle)
        if selectedStyle == "Red Theme" then
            -- Set Rayfield theme settings for Red Theme
            Window:SetTheme({
                BackgroundColor = Color3.fromRGB(0, 0, 0), -- Black background
                TextColor = Color3.fromRGB(255, 255, 255), -- White text
                AccentColor = Color3.fromRGB(255, 0, 0) -- Red accent
            })
            print("Red Theme selected")
        elseif selectedStyle == "Blue Theme" then
            -- Set Rayfield theme settings for Blue Theme
            Window:SetTheme({
                BackgroundColor = Color3.fromRGB(0, 0, 255), -- Blue background
                TextColor = Color3.fromRGB(255, 255, 255), -- White text
                AccentColor = Color3.fromRGB(0, 255, 255) -- Cyan accent
            })
            print("Blue Theme selected")
        elseif selectedStyle == "Style 3" then
            -- Set Rayfield theme settings for Style 3
            Window:SetTheme({
                BackgroundColor = Color3.fromRGB(0, 0, 255), -- Blue background
                TextColor = Color3.fromRGB(255, 255, 255), -- White text
                AccentColor = Color3.fromRGB(0, 255, 0) -- Green accent
            })
            print("Rayfield Style 3 selected")
        elseif selectedStyle == "Style 4" then
            -- Set Rayfield theme settings for Style 4
            Window:SetTheme({
                BackgroundColor = Color3.fromRGB(0, 255, 0), -- Green background
                TextColor = Color3.fromRGB(255, 255, 255), -- White text
                AccentColor = Color3.fromRGB(0, 0, 255) -- Blue accent
            })
            print("Rayfield Style 4 selected")
        end
    end
})

-- Show the window
Window:Display()

local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AdvancedExplorer"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local success = pcall(function()
    screenGui.Parent = CoreGui
end)
if not success then
    screenGui.Parent = playerGui
end

local expandedObjects = {}
local selectedObject = nil
local copiedObject = nil
local scriptViewer = nil
local contextMenu = nil

local function createNotification(text, color, duration)
    duration = duration or 3
    
    local notif = Instance.new("Frame")
    notif.Size = UDim2.new(0, 300, 0, 60)
    notif.Position = UDim2.new(1, -20, 1, -80)
    notif.BackgroundColor3 = color or Color3.fromRGB(0, 162, 255)
    notif.BorderSizePixel = 0
    notif.Parent = screenGui
    
    local border = Instance.new("UIStroke")
    border.Color = Color3.fromRGB(255, 255, 255)
    border.Thickness = 2
    border.Parent = notif
    
    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, -10, 1, 0)
    textLabel.Position = UDim2.new(0, 5, 0, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.Text = text
    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    textLabel.TextScaled = true
    textLabel.Font = Enum.Font.GothamBold
    textLabel.TextWrapped = true
    textLabel.Parent = notif
    
    notif:TweenPosition(UDim2.new(1, -320, 1, -80), "Out", "Quad", 0.5, true)
    
    game:GetService("Debris"):AddItem(notif, duration + 0.5)
    
    spawn(function()
        wait(duration)
        if notif.Parent then
            notif:TweenPosition(UDim2.new(1, 0, 1, -80), "Out", "Quad", 0.5, true)
        end
    end)
end

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 900, 0, 600)
mainFrame.Position = UDim2.new(0.5, -450, 0.5, -300)
mainFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local border = Instance.new("UIStroke")
border.Color = Color3.fromRGB(0, 162, 255)
border.Thickness = 3
border.Parent = mainFrame

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 35)
titleBar.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -120, 1, 0)
titleLabel.Position = UDim2.new(0, 10, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "Explorer beta teste - " .. game.Name
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextScaled = true
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = titleBar

local toolBar = Instance.new("Frame")
toolBar.Size = UDim2.new(1, 0, 0, 35)
toolBar.Position = UDim2.new(0, 0, 0, 35)
toolBar.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
toolBar.BorderSizePixel = 0
toolBar.Parent = mainFrame

local function createToolButton(text, position, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 100, 1, -6)
    button.Position = UDim2.new(0, position, 0, 3)
    button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    button.BorderSizePixel = 0
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextScaled = true
    button.Font = Enum.Font.GothamBold
    button.Parent = toolBar
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 4)
    corner.Parent = button
    
    button.MouseEnter:Connect(function()
        button.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
    end)
    
    button.MouseLeave:Connect(function()
        button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    end)
    
    if callback then
        button.MouseButton1Click:Connect(callback)
    end
    
    return button
end

local minimizeButton = Instance.new("TextButton")
minimizeButton.Size = UDim2.new(0, 30, 1, 0)
minimizeButton.Position = UDim2.new(1, -90, 0, 0)
minimizeButton.BackgroundColor3 = Color3.fromRGB(255, 165, 0)
minimizeButton.BorderSizePixel = 0
minimizeButton.Text = "—"
minimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeButton.TextScaled = true
minimizeButton.Font = Enum.Font.GothamBold
minimizeButton.Parent = titleBar

local maximizeButton = Instance.new("TextButton")
maximizeButton.Size = UDim2.new(0, 30, 1, 0)
maximizeButton.Position = UDim2.new(1, -60, 0, 0)
maximizeButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
maximizeButton.BorderSizePixel = 0
maximizeButton.Text = "□"
maximizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
maximizeButton.TextScaled = true
maximizeButton.Font = Enum.Font.GothamBold
maximizeButton.Parent = titleBar

local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.new(0, 30, 1, 0)
closeButton.Position = UDim2.new(1, -30, 0, 0)
closeButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
closeButton.BorderSizePixel = 0
closeButton.Text = "✕"
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.TextScaled = true
closeButton.Font = Enum.Font.GothamBold
closeButton.Parent = titleBar

local contentFrame = Instance.new("Frame")
contentFrame.Size = UDim2.new(1, 0, 1, -70)
contentFrame.Position = UDim2.new(0, 0, 0, 70)
contentFrame.BackgroundTransparency = 1
contentFrame.Parent = mainFrame

local objectListFrame = Instance.new("Frame")
objectListFrame.Size = UDim2.new(0.5, -2, 1, 0)
objectListFrame.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
objectListFrame.BorderSizePixel = 0
objectListFrame.Parent = contentFrame

local objectListTitle = Instance.new("TextLabel")
objectListTitle.Size = UDim2.new(1, 0, 0, 30)
objectListTitle.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
objectListTitle.BorderSizePixel = 0
objectListTitle.Text = "🌲 coisas"
objectListTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
objectListTitle.TextScaled = true
objectListTitle.Font = Enum.Font.GothamBold
objectListTitle.Parent = objectListFrame

local objectScrollFrame = Instance.new("ScrollingFrame")
objectScrollFrame.Size = UDim2.new(1, 0, 1, -30)
objectScrollFrame.Position = UDim2.new(0, 0, 0, 30)
objectScrollFrame.BackgroundTransparency = 1
objectScrollFrame.BorderSizePixel = 0
objectScrollFrame.ScrollBarThickness = 10
objectScrollFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 162, 255)
objectScrollFrame.Parent = objectListFrame

local propertiesFrame = Instance.new("Frame")
propertiesFrame.Size = UDim2.new(0.5, -2, 1, 0)
propertiesFrame.Position = UDim2.new(0.5, 2, 0, 0)
propertiesFrame.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
propertiesFrame.BorderSizePixel = 0
propertiesFrame.Parent = contentFrame

local propertiesTitle = Instance.new("TextLabel")
propertiesTitle.Size = UDim2.new(1, 0, 0, 30)
propertiesTitle.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
propertiesTitle.BorderSizePixel = 0
propertiesTitle.Text = "⚙️ Propriedades"
propertiesTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
propertiesTitle.TextScaled = true
propertiesTitle.Font = Enum.Font.GothamBold
propertiesTitle.Parent = propertiesFrame

local propertiesScrollFrame = Instance.new("ScrollingFrame")
propertiesScrollFrame.Size = UDim2.new(1, 0, 1, -30)
propertiesScrollFrame.Position = UDim2.new(0, 0, 0, 30)
propertiesScrollFrame.BackgroundTransparency = 1
propertiesScrollFrame.BorderSizePixel = 0
propertiesScrollFrame.ScrollBarThickness = 10
propertiesScrollFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 162, 255)
propertiesScrollFrame.Parent = propertiesFrame

local function getObjectIcon(obj)
    local className = obj.ClassName
    local icons = {
        ["Workspace"] = "🌍", ["Players"] = "👥", ["ReplicatedStorage"] = "📦",
        ["ServerStorage"] = "🗄️", ["StarterPlayer"] = "🎮", ["StarterGui"] = "🖥️",
        ["StarterPack"] = "🎒", ["Lighting"] = "💡", ["SoundService"] = "🔊",
        ["Script"] = "📜", ["LocalScript"] = "📃", ["ModuleScript"] = "📋",
        ["Part"] = "🧱", ["MeshPart"] = "🗿", ["UnionOperation"] = "🔗",
        ["Model"] = "📁", ["Folder"] = "📂", ["RemoteEvent"] = "📡",
        ["RemoteFunction"] = "📞", ["BindableEvent"] = "🔄", ["BindableFunction"] = "⚙️",
        ["StringValue"] = "📝", ["IntValue"] = "🔢", ["BoolValue"] = "✅",
        ["ObjectValue"] = "🎯", ["Frame"] = "🖼️", ["TextLabel"] = "🏷️",
        ["TextButton"] = "🔘", ["ImageLabel"] = "🖼️", ["ScreenGui"] = "📺",
        ["Tool"] = "🔧", ["Humanoid"] = "🚶", ["Camera"] = "📹"
    }
    return icons[className] or "📄"
end

function viewScript(scriptObj)
    if scriptViewer then
        scriptViewer:Destroy()
    end
    
    scriptViewer = Instance.new("Frame")
    scriptViewer.Size = UDim2.new(0, 700, 0, 500)
    scriptViewer.Position = UDim2.new(0.5, -350, 0.5, -250)
    scriptViewer.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    scriptViewer.BorderSizePixel = 0
    scriptViewer.Active = true
    scriptViewer.Draggable = true
    scriptViewer.Parent = screenGui
    
    local scriptBorder = Instance.new("UIStroke")
    scriptBorder.Color = Color3.fromRGB(255, 255, 0)
    scriptBorder.Thickness = 2
    scriptBorder.Parent = scriptViewer
    
    local scriptTitle = Instance.new("TextLabel")
    scriptTitle.Size = UDim2.new(1, -90, 0, 35)
    scriptTitle.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    scriptTitle.BorderSizePixel = 0
    scriptTitle.Text = "📜 Script Viewer - " .. scriptObj.Name .. " (" .. scriptObj.ClassName .. ")"
    scriptTitle.TextColor3 = Color3.fromRGB(255, 255, 0)
    scriptTitle.TextScaled = true
    scriptTitle.Font = Enum.Font.GothamBold
    scriptTitle.TextXAlignment = Enum.TextXAlignment.Left
    scriptTitle.Parent = scriptViewer
    
    local saveButton = Instance.new("TextButton")
    saveButton.Size = UDim2.new(0, 40, 0, 35)
    saveButton.Position = UDim2.new(1, -90, 0, 0)
    saveButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    saveButton.BorderSizePixel = 0
    saveButton.Text = "💾"
    saveButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    saveButton.TextScaled = true
    saveButton.Font = Enum.Font.GothamBold
    saveButton.Parent = scriptViewer
    
    local copyButton = Instance.new("TextButton")
    copyButton.Size = UDim2.new(0, 40, 0, 35)
    copyButton.Position = UDim2.new(1, -50, 0, 0)
    copyButton.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
    copyButton.BorderSizePixel = 0
    copyButton.Text = "📋"
    copyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    copyButton.TextScaled = true
    copyButton.Font = Enum.Font.GothamBold
    copyButton.Parent = scriptViewer
    
    local scriptClose = Instance.new("TextButton")
    scriptClose.Size = UDim2.new(0, 40, 0, 35)
    scriptClose.Position = UDim2.new(1, -10, 0, 0)
    scriptClose.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    scriptClose.BorderSizePixel = 0
    scriptClose.Text = "✕"
    scriptClose.TextColor3 = Color3.fromRGB(255, 255, 255)
    scriptClose.TextScaled = true
    scriptClose.Font = Enum.Font.GothamBold
    scriptClose.Parent = scriptViewer
    
    local scriptBox = Instance.new("ScrollingFrame")
    scriptBox.Size = UDim2.new(1, 0, 1, -35)
    scriptBox.Position = UDim2.new(0, 0, 0, 35)
    scriptBox.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    scriptBox.BorderSizePixel = 0
    scriptBox.ScrollBarThickness = 10
    scriptBox.ScrollBarImageColor3 = Color3.fromRGB(255, 255, 0)
    scriptBox.Parent = scriptViewer
    
    local scriptText = Instance.new("TextBox")
    scriptText.Size = UDim2.new(1, -20, 1, 0)
    scriptText.Position = UDim2.new(0, 10, 0, 0)
    scriptText.BackgroundTransparency = 1
    scriptText.Text = ""
    scriptText.TextColor3 = Color3.fromRGB(255, 255, 255)
    scriptText.TextXAlignment = Enum.TextXAlignment.Left
    scriptText.TextYAlignment = Enum.TextYAlignment.Top
    scriptText.Font = Enum.Font.Code
    scriptText.TextSize = 16
    scriptText.ClearTextOnFocus = false
    scriptText.MultiLine = true
    scriptText.TextWrapped = true
    scriptText.Parent = scriptBox
    
    local function getScriptSource()
        local success, source = pcall(function()
            if scriptObj:IsA("LocalScript") then
                return scriptObj.Source
            elseif scriptObj:IsA("Script") then
                return scriptObj.Source
            elseif scriptObj:IsA("ModuleScript") then
                return scriptObj.Source
            end
            return "-- Unable to read script source"
        end)
        
        if success and source and source ~= "" then
            return source
        else
            local alternativeSource = ""
            pcall(function()
                alternativeSource = tostring(scriptObj.Source)
            end)
            
            if alternativeSource ~= "" then
                return alternativeSource
            else
                return "-- Script source is protected or empty\n-- Script Name: " .. scriptObj.Name .. "\n-- Script Type: " .. scriptObj.ClassName .. "\n-- Parent: " .. (scriptObj.Parent and scriptObj.Parent.Name or "nil")
            end
        end
    end
    
    local sourceCode = getScriptSource()
    scriptText.Text = sourceCode
    
    local textBounds = game:GetService("TextService"):GetTextSize(
        sourceCode, 16, Enum.Font.Code, Vector2.new(scriptText.AbsoluteSize.X, math.huge)
    )
    scriptBox.CanvasSize = UDim2.new(0, 0, 0, math.max(textBounds.Y + 50, scriptBox.AbsoluteSize.Y))
    
    saveButton.MouseButton1Click:Connect(function()
        pcall(function()
            scriptObj.Source = scriptText.Text
            createNotification("✅ Script saved successfully!", Color3.fromRGB(0, 255, 0))
        end)
    end)
    
    copyButton.MouseButton1Click:Connect(function()
        createNotification("📋 Script copied to console!", Color3.fromRGB(0, 162, 255))
    end)
    
    scriptClose.MouseButton1Click:Connect(function()
        scriptViewer:Destroy()
        scriptViewer = nil
    end)
end

local function createContextMenu(x, y, obj)
    if contextMenu then
        contextMenu:Destroy()
    end
    
    contextMenu = Instance.new("Frame")
    contextMenu.Size = UDim2.new(0, 180, 0, 250)
    contextMenu.Position = UDim2.new(0, math.min(x, screenGui.AbsoluteSize.X - 180), 0, math.min(y, screenGui.AbsoluteSize.Y - 250))
    contextMenu.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    contextMenu.BorderSizePixel = 0
    contextMenu.ZIndex = 20
    contextMenu.Parent = screenGui
    
    local contextBorder = Instance.new("UIStroke")
    contextBorder.Color = Color3.fromRGB(0, 162, 255)
    contextBorder.Thickness = 2
    contextBorder.Parent = contextMenu
    
    local menuItems = {
        {text = "📋 Copy Object", color = Color3.fromRGB(0, 162, 255), action = function()
            copiedObject = obj
            createNotification("✅ Object copied: " .. obj.Name, Color3.fromRGB(0, 255, 0))
        end},
        {text = "📄 Paste Object", color = Color3.fromRGB(255, 165, 0), action = function()
            if copiedObject and obj then
                local success = pcall(function()
                    local clone = copiedObject:Clone()
                    clone.Parent = obj
                    refreshObjectList()
                    createNotification("✅ Object pasted: " .. clone.Name, Color3.fromRGB(0, 255, 0))
                end)
                if not success then
                    createNotification("❌ Failed to paste object!", Color3.fromRGB(255, 0, 0))
                end
            else
                createNotification("❌ No object copied or invalid target!", Color3.fromRGB(255, 165, 0))
            end
        end},
        {text = "📑 Duplicate Object", color = Color3.fromRGB(128, 0, 255), action = function()
            local success = pcall(function()
                local clone = obj:Clone()
                clone.Name = clone.Name .. "_Copy"
                clone.Parent = obj.Parent
                refreshObjectList()
                createNotification("✅ Object duplicated: " .. clone.Name, Color3.fromRGB(0, 255, 0))
            end)
            if not success then
                createNotification("❌ Failed to duplicate object!", Color3.fromRGB(255, 0, 0))
            end
        end},
        {text = "🗑️ Delete Object", color = Color3.fromRGB(255, 0, 0), action = function()
            if obj.Parent then
                local objName = obj.Name
                local success = pcall(function()
                    obj:Destroy()
                    refreshObjectList()
                    createNotification("✅ Deleted: " .. objName, Color3.fromRGB(255, 100, 100))
                end)
                if not success then
                    createNotification("❌ Failed to delete object!", Color3.fromRGB(255, 0, 0))
                end
            else
                createNotification("❌ Cannot delete this object!", Color3.fromRGB(255, 165, 0))
            end
        end},
        {text = "📜 View Script", color = Color3.fromRGB(255, 255, 0), action = function()
            if obj:IsA("Script") or obj:IsA("LocalScript") or obj:IsA("ModuleScript") then
                viewScript(obj)
            else
                createNotification("❌ Not a script object!", Color3.fromRGB(255, 165, 0))
            end
        end},
        {text = "🎯 Select in Game", color = Color3.fromRGB(0, 255, 0), action = function()
            if obj:IsA("BasePart") then
                pcall(function()
                    if game.Selection then
                        game.Selection:Set({obj})
                    end
                    createNotification("🎯 Selected in game: " .. obj.Name, Color3.fromRGB(0, 255, 0))
                end)
            else
                createNotification("❌ Can only select BaseParts!", Color3.fromRGB(255, 165, 0))
            end
        end},
        {text = "ℹ️ Get Full Path", color = Color3.fromRGB(255, 255, 255), action = function()
            local path = obj:GetFullName()
            createNotification("📋 Path printed to console!", Color3.fromRGB(0, 162, 255))
        end}
    }
    
    for i, item in pairs(menuItems) do
        local button = Instance.new("TextButton")
        button.Size = UDim2.new(1, 0, 0, 30)
        button.Position = UDim2.new(0, 0, 0, (i-1) * 30)
        button.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
        button.BorderSizePixel = 0
        button.Text = item.text
        button.TextColor3 = Color3.fromRGB(255, 255, 255)
        button.TextScaled = true
        button.Font = Enum.Font.GothamBold
        button.TextXAlignment = Enum.TextXAlignment.Left
        button.ZIndex = 21
        button.Parent = contextMenu
        
        local padding = Instance.new("UIPadding")
        padding.PaddingLeft = UDim.new(0, 10)
        padding.Parent = button
        
        button.MouseEnter:Connect(function()
            button.BackgroundColor3 = item.color
        end)
        
        button.MouseLeave:Connect(function()
            button.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
        end)
        
        button.MouseButton1Click:Connect(function()
            item.action()
            contextMenu:Destroy()
            contextMenu = nil
        end)
    end
end

local function createEditableProperty(propName, propValue, obj, yPos, parent)
    local propFrame = Instance.new("Frame")
    propFrame.Size = UDim2.new(1, 0, 0, 35)
    propFrame.Position = UDim2.new(0, 0, 0, yPos)
    propFrame.BackgroundColor3 = yPos % 70 == 0 and Color3.fromRGB(65, 65, 65) or Color3.fromRGB(55, 55, 55)
    propFrame.BorderSizePixel = 0
    propFrame.Parent = parent
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(0.35, -5, 1, 0)
    nameLabel.Position = UDim2.new(0, 5, 0, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = propName
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.TextScaled = true
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextXAlignment = Enum.TextXAlignment.Left
    nameLabel.Parent = propFrame
    
    local valueBox = Instance.new("TextBox")
    valueBox.Size = UDim2.new(0.65, -10, 0.7, 0)
    valueBox.Position = UDim2.new(0.35, 0, 0.15, 0)
    valueBox.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    valueBox.BorderSizePixel = 0
    valueBox.Text = tostring(propValue)
    valueBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    valueBox.TextScaled = true
    valueBox.Font = Enum.Font.Gotham
    valueBox.ClearTextOnFocus = false
    valueBox.Parent = propFrame
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 3)
    corner.Parent = valueBox
    
    local border = Instance.new("UIStroke")
    border.Color = Color3.fromRGB(80, 80, 80)
    border.Thickness = 1
    border.Parent = valueBox
    
    valueBox.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            local success = pcall(function()
                local newValue = valueBox.Text
                local oldValue = obj[propName]
                
                if type(oldValue) == "boolean" then
                    obj[propName] = newValue:lower() == "true"
                elseif type(oldValue) == "number" then
                    local numValue = tonumber(newValue)
                    if numValue then
                        obj[propName] = numValue
                    end
                elseif type(oldValue) == "string" then
                    obj[propName] = newValue
                elseif typeof(oldValue) == "Vector3" then
                    local success, result = pcall(function()
                        return loadstring("return " .. newValue)()
                    end)
                    if success and typeof(result) == "Vector3" then
                        obj[propName] = result
                    end
                elseif typeof(oldValue) == "UDim2" then
                    local success, result = pcall(function()
                        return loadstring("return " .. newValue)()
                    end)
                    if success and typeof(result) == "UDim2" then
                        obj[propName] = result
                    end
                elseif typeof(oldValue) == "Color3" then
                    local success, result = pcall(function()
                        return loadstring("return " .. newValue)()
                    end)
                    if success and typeof(result) == "Color3" then
                        obj[propName] = result
                    end
                elseif typeof(oldValue) == "Enum" then
                    local enumType = string.match(tostring(oldValue), "Enum%.(%w+)%.")
                    if enumType then
                        local success, result = pcall(function()
                            return Enum[enumType][newValue]
                        end)
                        if success then
                            obj[propName] = result
                        end
                    end
                else
                    local numValue = tonumber(newValue)
                    if numValue then
                        obj[propName] = numValue
                    else
                        obj[propName] = newValue
                    end
                end
                
                if propName == "Name" then
                    refreshObjectList()
                end
                
                border.Color = Color3.fromRGB(0, 255, 0)
                valueBox.BackgroundColor3 = Color3.fromRGB(0, 100, 0)
                
                spawn(function()
                    wait(0.5)
                    if valueBox.Parent then
                        border.Color = Color3.fromRGB(80, 80, 80)
                        valueBox.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
                    end
                end)
                
                createNotification("✅ Property updated: " .. propName, Color3.fromRGB(0, 255, 0), 2)
            end)
            
            if not success then
                border.Color = Color3.fromRGB(255, 0, 0)
                valueBox.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
                
                spawn(function()
                    wait(1)
                    if valueBox.Parent then
                        border.Color = Color3.fromRGB(80, 80, 80)
                        valueBox.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
                    end
                end)
                
                createNotification("❌ Failed to update: " .. propName, Color3.fromRGB(255, 0, 0), 3)
            end
        end
    end)
    
    return propFrame
end

function showProperties(obj)
    for _, child in pairs(propertiesScrollFrame:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end
    
    local yPos = 0
    local properties = {}
    
    pcall(function()
        properties["Name"] = obj.Name
        properties["ClassName"] = obj.ClassName
        properties["Parent"] = obj.Parent and obj.Parent.Name or "nil"
        
        if obj:IsA("BasePart") then
            properties["Position"] = obj.Position
            properties["Size"] = obj.Size
            properties["Anchored"] = obj.Anchored
            properties["CanCollide"] = obj.CanCollide
            properties["Material"] = obj.Material.Name
            properties["Color"] = obj.Color
            properties["Transparency"] = obj.Transparency
            properties["Reflectance"] = obj.Reflectance
            properties["Shape"] = obj.Shape and obj.Shape.Name or "Block"
            
        elseif obj:IsA("Script") or obj:IsA("LocalScript") then
            properties["Disabled"] = obj.Disabled
            properties["RunContext"] = obj.RunContext and obj.RunContext.Name or "Legacy"
            
        elseif obj:IsA("ModuleScript") then
            properties["LinkedSource"] = obj.LinkedSource
            
        elseif obj:IsA("GuiObject") then
            properties["Position"] = obj.Position
            properties["Size"] = obj.Size
            properties["Visible"] = obj.Visible
            properties["BackgroundTransparency"] = obj.BackgroundTransparency
            properties["BackgroundColor3"] = obj.BackgroundColor3
            properties["BorderSizePixel"] = obj.BorderSizePixel
            properties["ZIndex"] = obj.ZIndex
            
            if obj:IsA("Frame") then
                properties["Active"] = obj.Active
                
            elseif obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox") then
                properties["Text"] = obj.Text
                properties["TextColor3"] = obj.TextColor3
                properties["TextScaled"] = obj.TextScaled
                properties["TextSize"] = obj.TextSize
                properties["Font"] = obj.Font.Name
                properties["TextWrapped"] = obj.TextWrapped
                
                if obj:IsA("TextBox") then
                    properties["PlaceholderText"] = obj.PlaceholderText
                    properties["ClearTextOnFocus"] = obj.ClearTextOnFocus
                end
                
            elseif obj:IsA("ImageLabel") or obj:IsA("ImageButton") then
                properties["Image"] = obj.Image
                properties["ImageTransparency"] = obj.ImageTransparency
            end
            
        elseif obj:IsA("StringValue") then
            properties["Value"] = obj.Value
        elseif obj:IsA("IntValue") or obj:IsA("NumberValue") then
            properties["Value"] = obj.Value
        elseif obj:IsA("BoolValue") then
            properties["Value"] = obj.Value
        elseif obj:IsA("ObjectValue") then
            properties["Value"] = obj.Value and obj.Value.Name or "nil"
            
        elseif obj:IsA("Humanoid") then
            properties["Health"] = obj.Health
            properties["MaxHealth"] = obj.MaxHealth
            properties["WalkSpeed"] = obj.WalkSpeed
            properties["JumpHeight"] = obj.JumpHeight
            properties["DisplayDistanceType"] = obj.DisplayDistanceType.Name
            
        elseif obj:IsA("Camera") then
            properties["CameraType"] = obj.CameraType.Name
            properties["FieldOfView"] = obj.FieldOfView
            properties["CFrame"] = obj.CFrame
            
        elseif obj:IsA("Lighting") then
            properties["Brightness"] = obj.Brightness
            properties["Ambient"] = obj.Ambient
            properties["ColorShift_Top"] = obj.ColorShift_Top
            properties["ColorShift_Bottom"] = obj.ColorShift_Bottom
            properties["TimeOfDay"] = obj.TimeOfDay
            
        elseif obj:IsA("SoundService") then
            properties["Volume"] = obj.Volume
            properties["RolloffScale"] = obj.RolloffScale
            
        elseif obj:IsA("Players") then
            properties["MaxPlayers"] = obj.MaxPlayers
            properties["NumPlayers"] = obj.NumPlayers
            
        elseif obj:IsA("Workspace") then
            properties["CurrentCamera"] = obj.CurrentCamera and obj.CurrentCamera.Name or "nil"
            properties["Gravity"] = obj.Gravity
            properties["FallenPartsDestroyHeight"] = obj.FallenPartsDestroyHeight
        end
    end)
    
    for propName, propValue in pairs(properties) do
        if propName ~= "ClassName" and propName ~= "Parent" then
            createEditableProperty(propName, propValue, obj, yPos, propertiesScrollFrame)
        else
            local propFrame = Instance.new("Frame")
            propFrame.Size = UDim2.new(1, 0, 0, 35)
            propFrame.Position = UDim2.new(0, 0, 0, yPos)
            propFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
            propFrame.BorderSizePixel = 0
            propFrame.Parent = propertiesScrollFrame
            
            local nameLabel = Instance.new("TextLabel")
            nameLabel.Size = UDim2.new(0.35, -5, 1, 0)
            nameLabel.Position = UDim2.new(0, 5, 0, 0)
            nameLabel.BackgroundTransparency = 1
            nameLabel.Text = propName
            nameLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
            nameLabel.TextScaled = true
            nameLabel.Font = Enum.Font.GothamBold
            nameLabel.TextXAlignment = Enum.TextXAlignment.Left
            nameLabel.Parent = propFrame
            
            local valueLabel = Instance.new("TextLabel")
            valueLabel.Size = UDim2.new(0.65, -10, 0.7, 0)
            valueLabel.Position = UDim2.new(0.35, 0, 0.15, 0)
            valueLabel.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
            valueLabel.BorderSizePixel = 0
            valueLabel.Text = tostring(propValue)
            valueLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
            valueLabel.TextScaled = true
            valueLabel.Font = Enum.Font.Gotham
            valueLabel.TextXAlignment = Enum.TextXAlignment.Left
            valueLabel.Parent = propFrame
            
            local corner = Instance.new("UICorner")
            corner.CornerRadius = UDim.new(0, 3)
            corner.Parent = valueLabel
        end
        
        yPos = yPos + 35
    end
    
    propertiesScrollFrame.CanvasSize = UDim2.new(0, 0, 0, yPos + 10)
end

local function createObjectItem(obj, depth, parent, yPos)
    local itemFrame = Instance.new("Frame")
    itemFrame.Size = UDim2.new(1, 0, 0, 25)
    itemFrame.Position = UDim2.new(0, 0, 0, yPos)
    itemFrame.BackgroundTransparency = 1
    itemFrame.Parent = parent
    
    local indentSize = depth * 20
    
    local expandButton = Instance.new("TextButton")
    expandButton.Size = UDim2.new(0, 20, 0, 20)
    expandButton.Position = UDim2.new(0, indentSize, 0, 2.5)
    expandButton.BackgroundTransparency = 1
    expandButton.Text = #obj:GetChildren() > 0 and (expandedObjects[obj] and "🔽" or "▶️") or ""
    expandButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    expandButton.TextScaled = true
    expandButton.Font = Enum.Font.GothamBold
    expandButton.Parent = itemFrame
    
    local objectLabel = Instance.new("TextButton")
    objectLabel.Size = UDim2.new(1, -(indentSize + 25), 1, 0)
    objectLabel.Position = UDim2.new(0, indentSize + 25, 0, 0)
    objectLabel.BackgroundTransparency = 1
    objectLabel.Text = getObjectIcon(obj) .. " " .. obj.Name
    objectLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    objectLabel.TextScaled = true
    objectLabel.Font = Enum.Font.Gotham
    objectLabel.TextXAlignment = Enum.TextXAlignment.Left
    objectLabel.Parent = itemFrame
    
    objectLabel.MouseEnter:Connect(function()
        objectLabel.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
        objectLabel.BackgroundTransparency = 0.5
    end)
    
    objectLabel.MouseLeave:Connect(function()
        if selectedObject ~= obj then
            objectLabel.BackgroundTransparency = 1
        end
    end)
    
    expandButton.MouseButton1Click:Connect(function()
        if #obj:GetChildren() > 0 then
            expandedObjects[obj] = not expandedObjects[obj]
            refreshObjectList()
        end
    end)
    
    objectLabel.MouseButton1Click:Connect(function()
        selectedObject = obj
        showProperties(obj)
        
        for _, child in pairs(objectScrollFrame:GetChildren()) do
            if child:FindFirstChild("ObjectButton") then
                child.ObjectButton.BackgroundTransparency = 1
                if child:FindFirstChild("Selection") then
                    child.Selection:Destroy()
                end
            end
        end
        
        objectLabel.Name = "ObjectButton"
        objectLabel.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
        objectLabel.BackgroundTransparency = 0.3
        
        local selection = Instance.new("Frame")
        selection.Name = "Selection"
        selection.Size = UDim2.new(0, 3, 1, 0)
        selection.Position = UDim2.new(0, 0, 0, 0)
        selection.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
        selection.BorderSizePixel = 0
        selection.Parent = itemFrame
    end)
    
    objectLabel.MouseButton2Click:Connect(function()
        local mouse = game.Players.LocalPlayer:GetMouse()
        createContextMenu(mouse.X, mouse.Y, obj)
    end)
    
    return itemFrame
end

local function addObjectsToList(parent, depth, targetParent, yPos)
    local children = {}
    
    pcall(function()
        for _, child in pairs(parent:GetChildren()) do
            table.insert(children, child)
        end
    end)
    
    table.sort(children, function(a, b)
        local aType = a.ClassName
        local bType = b.ClassName
        
        if aType == bType then
            return a.Name:lower() < b.Name:lower()
        end
        
        local typeOrder = {
            Script = 1, LocalScript = 1, ModuleScript = 1,
            Folder = 2, Model = 2,
            Part = 3, MeshPart = 3, UnionOperation = 3
        }
        
        local aOrder = typeOrder[aType] or 4
        local bOrder = typeOrder[bType] or 4
        
        if aOrder == bOrder then
            return aType < bType
        end
        
        return aOrder < bOrder
    end)
    
    for _, child in pairs(children) do
        pcall(function()
            createObjectItem(child, depth, targetParent, yPos[1])
            yPos[1] = yPos[1] + 25
            
            if expandedObjects[child] then
                addObjectsToList(child, depth + 1, targetParent, yPos)
            end
        end)
    end
end

function refreshObjectList()
    for _, child in pairs(objectScrollFrame:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end
    
    local yPos = {0}
    
    local services = {
        {name = "Workspace", service = game.Workspace},
        {name = "Players", service = game.Players},
        {name = "ReplicatedStorage", service = game.ReplicatedStorage},
        {name = "ServerStorage", service = game.ServerStorage},
        {name = "StarterPlayer", service = game.StarterPlayer},
        {name = "StarterGui", service = game.StarterGui},
        {name = "StarterPack", service = game.StarterPack},
        {name = "Lighting", service = game.Lighting},
        {name = "SoundService", service = game.SoundService},
        {name = "ReplicatedFirst", service = game.ReplicatedFirst},
    }
    
    for _, serviceData in pairs(services) do
        pcall(function()
            if serviceData.service then
                createObjectItem(serviceData.service, 0, objectScrollFrame, yPos[1])
                yPos[1] = yPos[1] + 25
                
                if expandedObjects[serviceData.service] then
                    addObjectsToList(serviceData.service, 1, objectScrollFrame, yPos)
                end
            end
        end)
    end
    
    objectScrollFrame.CanvasSize = UDim2.new(0, 0, 0, yPos[1] + 50)
end

local function expandAll()
    local count = 0
    for _, service in pairs({game.Workspace, game.ReplicatedStorage, game.StarterGui, game.Players}) do
        expandedObjects[service] = true
        count = count + 1
        
        for _, child in pairs(service:GetDescendants()) do
            if #child:GetChildren() > 0 and count < 100 then
                expandedObjects[child] = true
                count = count + 1
            end
        end
    end
    refreshObjectList()
    createNotification("🔄 Expanded " .. count .. " objects!", Color3.fromRGB(0, 255, 0))
end

local function collapseAll()
    expandedObjects = {}
    refreshObjectList()
    createNotification("📁 Collapsed all objects!", Color3.fromRGB(255, 165, 0))
end

local function refreshExplorer()
    refreshObjectList()
    if selectedObject and selectedObject.Parent then
        showProperties(selectedObject)
        createNotification("🔄 Explorer refreshed!", Color3.fromRGB(0, 162, 255))
    else
        createNotification("🔄 Explorer refreshed!", Color3.fromRGB(0, 162, 255))
        selectedObject = nil
    end
end

createToolButton("🔄 Refresh", 5, refreshExplorer)
createToolButton("➕ Expand All", 110, expandAll)
createToolButton("➖ Collapse All", 220, collapseAll)

minimizeButton.MouseButton1Click:Connect(function()
    mainFrame.Visible = not mainFrame.Visible
    if mainFrame.Visible then
        createNotification("🔄 Explorer restored!", Color3.fromRGB(0, 162, 255))
    end
end)

local maximized = false
maximizeButton.MouseButton1Click:Connect(function()
    if maximized then
        mainFrame.Size = UDim2.new(0, 900, 0, 600)
        mainFrame.Position = UDim2.new(0.5, -450, 0.5, -300)
        maximized = false
        createNotification("🔄 Window restored!", Color3.fromRGB(255, 165, 0))
    else
        mainFrame.Size = UDim2.new(1, -40, 1, -40)
        mainFrame.Position = UDim2.new(0, 20, 0, 20)
        maximized = true
        createNotification("🔄 Window maximized!", Color3.fromRGB(0, 255, 0))
    end
end)

closeButton.MouseButton1Click:Connect(function()
    createNotification("👋 Explorer closed!", Color3.fromRGB(255, 100, 100))
    wait(0.5)
    screenGui:Destroy()
end)

UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        if contextMenu then
            contextMenu:Destroy()
            contextMenu = nil
        end
    end
end)

local function startAutoUpdate()
    spawn(function()
        while screenGui.Parent do
            wait(5)
            if selectedObject and selectedObject.Parent then
                showProperties(selectedObject)
            end
        end
    end)
end

refreshObjectList()

expandedObjects[game.Workspace] = true
expandedObjects[game.StarterGui] = true
refreshObjectList()

startAutoUpdate()

spawn(function()
    wait(0.5)
    createNotification("🚀Explorer carregado, bem vindo", Color3.fromRGB(0, 255, 0), 4)
end)

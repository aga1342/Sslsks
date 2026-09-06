--// LOG NOTIFIER — interface inspirée du fichier fourni

local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")

local player = Players.LocalPlayer

local API_URL =
    "https://autojoiner2-2d181-default-rtdb.firebaseio.com/logs.json"

local enabled = false
local lastPlaceId = nil
local lastJobId = nil
local lastLogId = nil
local logCount = 0

local COLORS = {
    MainBG = Color3.fromRGB(13, 13, 17),
    HeaderBG = Color3.fromRGB(20, 20, 25),
    Stroke = Color3.fromRGB(45, 45, 55),
    White = Color3.fromRGB(255, 255, 255),
    Gray = Color3.fromRGB(110, 110, 115),
    ButtonBG = Color3.fromRGB(28, 28, 35),
    ButtonHover = Color3.fromRGB(35, 35, 45),
    Green = Color3.fromRGB(0, 220, 100),
    Red = Color3.fromRGB(220, 60, 60)
}

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "LogNotifier"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

pcall(function()
    ScreenGui.Parent = gethui()
end)
if not ScreenGui.Parent then
    ScreenGui.Parent = CoreGui
end

local StatusBar = Instance.new("Frame")
StatusBar.Parent = ScreenGui
StatusBar.BackgroundColor3 = COLORS.HeaderBG
StatusBar.Size = UDim2.new(0, 290, 0, 44)
StatusBar.Position = UDim2.new(0.35, 0, 0.017, 0)
StatusBar.BorderSizePixel = 0
StatusBar.Active = true
StatusBar.Draggable = true
StatusBar.ZIndex = 50
Instance.new("UICorner", StatusBar).CornerRadius = UDim.new(0, 9)

local SBStroke = Instance.new("UIStroke")
SBStroke.Parent = StatusBar
SBStroke.Color = COLORS.Stroke
SBStroke.Thickness = 1.5

local function MakeStatusColumn(parent, x, label, value)
    local dot = Instance.new("Frame")
    dot.Parent = parent
    dot.BackgroundColor3 = COLORS.Red
    dot.Position = UDim2.new(0, x, 0, 10)
    dot.Size = UDim2.new(0, 7, 0, 7)
    dot.BorderSizePixel = 0
    dot.ZIndex = 51
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

    local top = Instance.new("TextLabel")
    top.Parent = parent
    top.BackgroundTransparency = 1
    top.Position = UDim2.new(0, x + 12, 0, 6)
    top.Size = UDim2.new(0, 90, 0, 14)
    top.Font = Enum.Font.GothamBold
    top.Text = label
    top.TextColor3 = COLORS.Gray
    top.TextSize = 10
    top.TextXAlignment = Enum.TextXAlignment.Left
    top.ZIndex = 51

    local val = Instance.new("TextLabel")
    val.Parent = parent
    val.BackgroundTransparency = 1
    val.Position = UDim2.new(0, x + 12, 0, 22)
    val.Size = UDim2.new(0, 90, 0, 14)
    val.Font = Enum.Font.GothamBold
    val.Text = value
    val.TextColor3 = COLORS.Gray
    val.TextSize = 12
    val.TextXAlignment = Enum.TextXAlignment.Left
    val.ZIndex = 51

    return dot, val
end

local ApiDot, ApiValue = MakeStatusColumn(StatusBar, 14, "Logger", "OFF")

local Divider = Instance.new("Frame")
Divider.Parent = StatusBar
Divider.BackgroundColor3 = COLORS.Stroke
Divider.Position = UDim2.new(0, 116, 0, 9)
Divider.Size = UDim2.new(0, 1, 0, 26)
Divider.BorderSizePixel = 0
Divider.ZIndex = 51

local LogDot, LogValue = MakeStatusColumn(StatusBar, 128, "Logs", "0")

local MainFrame = Instance.new("Frame")
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = COLORS.MainBG
MainFrame.Position = UDim2.new(0.36, 0, 0.30, 0)
MainFrame.Size = UDim2.new(0, 310, 0, 380)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

local MainStroke = Instance.new("UIStroke")
MainStroke.Parent = MainFrame
MainStroke.Color = COLORS.Stroke
MainStroke.Thickness = 1.5

local Header = Instance.new("Frame")
Header.Parent = MainFrame
Header.BackgroundColor3 = COLORS.HeaderBG
Header.Size = UDim2.new(1, 0, 0, 48)
Header.BorderSizePixel = 0
Instance.new("UICorner", Header).CornerRadius = UDim.new(0, 12)

local HeaderHide = Instance.new("Frame")
HeaderHide.Parent = Header
HeaderHide.BackgroundColor3 = COLORS.HeaderBG
HeaderHide.Position = UDim2.new(0, 0, 1, -12)
HeaderHide.Size = UDim2.new(1, 0, 0, 12)
HeaderHide.BorderSizePixel = 0

local StatusDot = Instance.new("Frame")
StatusDot.Parent = Header
StatusDot.BackgroundColor3 = COLORS.Red
StatusDot.Position = UDim2.new(0, 18, 0.5, -7)
StatusDot.Size = UDim2.new(0, 14, 0, 14)
StatusDot.BorderSizePixel = 0
Instance.new("UICorner", StatusDot).CornerRadius = UDim.new(1, 0)

local Title = Instance.new("TextLabel")
Title.Parent = Header
Title.BackgroundTransparency = 1
Title.Position = UDim2.new(0, 42, 0, 8)
Title.Size = UDim2.new(0, 180, 0, 30)
Title.Font = Enum.Font.GothamBold
Title.Text = "LOG | NOTIFIER"
Title.TextColor3 = COLORS.White
Title.TextSize = 13
Title.TextXAlignment = Enum.TextXAlignment.Left

local Minimize = Instance.new("TextButton")
Minimize.Parent = Header
Minimize.BackgroundTransparency = 1
Minimize.Position = UDim2.new(1, -40, 0, 10)
Minimize.Size = UDim2.new(0, 28, 0, 28)
Minimize.Font = Enum.Font.GothamBold
Minimize.Text = "−"
Minimize.TextColor3 = COLORS.Gray
Minimize.TextSize = 16

Minimize.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
end)

local HeaderDivider = Instance.new("Frame")
HeaderDivider.Parent = MainFrame
HeaderDivider.BackgroundColor3 = COLORS.Stroke
HeaderDivider.Position = UDim2.new(0, 0, 0, 48)
HeaderDivider.Size = UDim2.new(1, 0, 0, 1)
HeaderDivider.BorderSizePixel = 0

local LeftContainer = Instance.new("Frame")
LeftContainer.Parent = MainFrame
LeftContainer.BackgroundColor3 = COLORS.HeaderBG
LeftContainer.Position = UDim2.new(0, 4, 0, 53)
LeftContainer.Size = UDim2.new(0, 50, 1, -57)
LeftContainer.BorderSizePixel = 0
Instance.new("UICorner", LeftContainer).CornerRadius = UDim.new(0, 8)

local function SideButton(text, y)
    local btn = Instance.new("TextButton")
    btn.Parent = LeftContainer
    btn.BackgroundColor3 = COLORS.ButtonBG
    btn.Position = UDim2.new(0, 7, 0, y)
    btn.Size = UDim2.new(0, 36, 0, 36)
    btn.BorderSizePixel = 0
    btn.Font = Enum.Font.GothamBold
    btn.Text = text
    btn.TextColor3 = COLORS.White
    btn.TextSize = 10
    btn.AutoButtonColor = false
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 7)

    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {
            BackgroundColor3 = COLORS.ButtonHover
        }):Play()
    end)

    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {
            BackgroundColor3 = COLORS.ButtonBG
        }):Play()
    end)

    return btn
end

local HomeButton = SideButton("LOG", 10)
local InfoButton = SideButton("INFO", 52)

local RightContainer = Instance.new("Frame")
RightContainer.Parent = MainFrame
RightContainer.BackgroundColor3 = COLORS.HeaderBG
RightContainer.Position = UDim2.new(0, 59, 0, 53)
RightContainer.Size = UDim2.new(1, -63, 1, -57)
RightContainer.BorderSizePixel = 0
RightContainer.ClipsDescendants = true
Instance.new("UICorner", RightContainer).CornerRadius = UDim.new(0, 8)

local LogContent = Instance.new("ScrollingFrame")
LogContent.Parent = RightContainer
LogContent.BackgroundTransparency = 1
LogContent.Size = UDim2.new(1, 0, 1, -90)
LogContent.CanvasSize = UDim2.new(0, 0, 0, 0)
LogContent.AutomaticCanvasSize = Enum.AutomaticSize.Y
LogContent.ScrollBarThickness = 3
LogContent.ScrollBarImageColor3 = COLORS.White
LogContent.BorderSizePixel = 0

local Padding = Instance.new("UIPadding")
Padding.Parent = LogContent
Padding.PaddingLeft = UDim.new(0, 8)
Padding.PaddingRight = UDim.new(0, 8)
Padding.PaddingTop = UDim.new(0, 8)
Padding.PaddingBottom = UDim.new(0, 8)

local Layout = Instance.new("UIListLayout")
Layout.Parent = LogContent
Layout.SortOrder = Enum.SortOrder.LayoutOrder
Layout.Padding = UDim.new(0, 5)

local JoinButton = Instance.new("TextButton")
JoinButton.Parent = RightContainer
JoinButton.BackgroundColor3 = COLORS.ButtonBG
JoinButton.Position = UDim2.new(0, 8, 1, -82)
JoinButton.Size = UDim2.new(1, -16, 0, 32)
JoinButton.BorderSizePixel = 0
JoinButton.Font = Enum.Font.GothamBold
JoinButton.Text = "JOIN"
JoinButton.TextColor3 = COLORS.White
JoinButton.TextSize = 12
JoinButton.AutoButtonColor = false
Instance.new("UICorner", JoinButton).CornerRadius = UDim.new(0, 6)

JoinButton.MouseButton1Click:Connect(function()
    if not lastPlaceId or not lastJobId or not tonumber(lastPlaceId) then
        JoinButton.Text = "NO SERVER"
        task.delay(1, function()
            if JoinButton then JoinButton.Text = "JOIN" end
        end)
        return
    end

    JoinButton.Text = "JOINING..."

    pcall(function()
        TeleportService:TeleportToPlaceInstance(
            tonumber(lastPlaceId),
            tostring(lastJobId),
            player
        )
    end)

    task.delay(2, function()
        if JoinButton then JoinButton.Text = "JOIN" end
    end)
end)

local Toggle = Instance.new("TextButton")
Toggle.Parent = RightContainer
Toggle.BackgroundColor3 = COLORS.ButtonBG
Toggle.Position = UDim2.new(0, 8, 1, -44)
Toggle.Size = UDim2.new(1, -16, 0, 32)
Toggle.BorderSizePixel = 0
Toggle.Text = ""
Toggle.AutoButtonColor = false
Instance.new("UICorner", Toggle).CornerRadius = UDim.new(0, 6)

local ToggleText = Instance.new("TextLabel")
ToggleText.Parent = Toggle
ToggleText.BackgroundTransparency = 1
ToggleText.Position = UDim2.new(0, 12, 0, 0)
ToggleText.Size = UDim2.new(0, 100, 1, 0)
ToggleText.Font = Enum.Font.GothamBold
ToggleText.Text = "Logger"
ToggleText.TextColor3 = COLORS.White
ToggleText.TextSize = 12
ToggleText.TextXAlignment = Enum.TextXAlignment.Left

local ToggleSwitch = Instance.new("Frame")
ToggleSwitch.Parent = Toggle
ToggleSwitch.BackgroundColor3 = COLORS.ButtonHover
ToggleSwitch.Position = UDim2.new(1, -48, 0.5, -10)
ToggleSwitch.Size = UDim2.new(0, 36, 0, 20)
ToggleSwitch.BorderSizePixel = 0
Instance.new("UICorner", ToggleSwitch).CornerRadius = UDim.new(1, 0)

local ToggleCircle = Instance.new("Frame")
ToggleCircle.Parent = ToggleSwitch
ToggleCircle.BackgroundColor3 = COLORS.Gray
ToggleCircle.Position = UDim2.new(0, 2, 0.5, -7)
ToggleCircle.Size = UDim2.new(0, 14, 0, 14)
ToggleCircle.BorderSizePixel = 0
Instance.new("UICorner", ToggleCircle).CornerRadius = UDim.new(1, 0)

local function SetEnabled(state)
    enabled = state

    local pos = enabled
        and UDim2.new(1, -18, 0.5, -7)
        or UDim2.new(0, 2, 0.5, -7)

    TweenService:Create(ToggleCircle, TweenInfo.new(0.2), {
        Position = pos,
        BackgroundColor3 = enabled and COLORS.Green or COLORS.Gray
    }):Play()

    ApiDot.BackgroundColor3 = enabled and COLORS.Green or COLORS.Red
    ApiValue.Text = enabled and "ON" or "OFF"
    ApiValue.TextColor3 = enabled and COLORS.Green or COLORS.Gray
    StatusDot.BackgroundColor3 = enabled and COLORS.Green or COLORS.Red
end

Toggle.MouseButton1Click:Connect(function()
    SetEnabled(not enabled)
end)

SetEnabled(false)

local function AddLog(data)
    logCount += 1

    local card = Instance.new("Frame")
    card.Parent = LogContent
    card.BackgroundColor3 = COLORS.ButtonBG
    card.Size = UDim2.new(1, 0, 0, 72)
    card.BorderSizePixel = 0
    card.LayoutOrder = -logCount
    Instance.new("UICorner", card).CornerRadius = UDim.new(0, 6)

    local stroke = Instance.new("UIStroke")
    stroke.Parent = card
    stroke.Color = COLORS.Stroke
    stroke.Thickness = 1

    local idLabel = Instance.new("TextLabel")
    idLabel.Parent = card
    idLabel.BackgroundTransparency = 1
    idLabel.Position = UDim2.new(0, 8, 0, 5)
    idLabel.Size = UDim2.new(1, -16, 0, 15)
    idLabel.Font = Enum.Font.GothamBold
    idLabel.Text = "#" .. tostring(data.id or "?")
    idLabel.TextColor3 = COLORS.White
    idLabel.TextSize = 11
    idLabel.TextXAlignment = Enum.TextXAlignment.Left

    local placeLabel = Instance.new("TextLabel")
    placeLabel.Parent = card
    placeLabel.BackgroundTransparency = 1
    placeLabel.Position = UDim2.new(0, 8, 0, 22)
    placeLabel.Size = UDim2.new(1, -16, 0, 14)
    placeLabel.Font = Enum.Font.GothamMedium
    placeLabel.Text = "PlaceId: " .. tostring(data.PlaceId or "?")
    placeLabel.TextColor3 = COLORS.Gray
    placeLabel.TextSize = 9
    placeLabel.TextXAlignment = Enum.TextXAlignment.Left

    local jobLabel = Instance.new("TextLabel")
    jobLabel.Parent = card
    jobLabel.BackgroundTransparency = 1
    jobLabel.Position = UDim2.new(0, 8, 0, 39)
    jobLabel.Size = UDim2.new(1, -16, 0, 14)
    jobLabel.Font = Enum.Font.GothamMedium
    jobLabel.Text = "JobId: " .. tostring(data.JobId or "?")
    jobLabel.TextColor3 = COLORS.Gray
    jobLabel.TextSize = 9
    jobLabel.TextXAlignment = Enum.TextXAlignment.Left

    local brainrots = data.Brainrots or {}
    local brainrotText = "Brainrots: "

    if type(brainrots) == "table" then
        brainrotText = brainrotText .. table.concat(brainrots, ", ")
    else
        brainrotText = brainrotText .. tostring(brainrots)
    end

    local brainLabel = Instance.new("TextLabel")
    brainLabel.Parent = card
    brainLabel.BackgroundTransparency = 1
    brainLabel.Position = UDim2.new(0, 8, 0, 55)
    brainLabel.Size = UDim2.new(1, -16, 0, 14)
    brainLabel.Font = Enum.Font.GothamMedium
    brainLabel.Text = brainrotText
    brainLabel.TextColor3 = COLORS.White
    brainLabel.TextSize = 9
    brainLabel.TextXAlignment = Enum.TextXAlignment.Left
    brainLabel.TextTruncate = Enum.TextTruncate.AtEnd

    LogValue.Text = tostring(logCount)
end

local function DoRequest()
    local requestFunction = request or http_request or (syn and syn.request)

    if not requestFunction then
        return nil
    end

    local success, result = pcall(function()
        return requestFunction({
            Url = API_URL,
            Method = "GET",
            Headers = {
                ["Cache-Control"] = "no-cache"
            }
        })
    end)

    if not success or not result then
        return nil
    end

    if result.StatusCode and result.StatusCode ~= 200 then
        return nil
    end

    return result.Body
end

task.spawn(function()
    while true do
        task.wait(3)

        if enabled then
            local body = DoRequest()

            if body then
                local success, data = pcall(function()
                    return HttpService:JSONDecode(body)
                end)

                if success and data and data.payload then
                    if tostring(data.id) ~= tostring(lastLogId) then
                        lastLogId = data.id

                        local payload = data.payload

                        lastPlaceId = payload.PlaceId
                        lastJobId = payload.JobId

                        AddLog({
                            id = data.id,
                            PlaceId = payload.PlaceId,
                            JobId = payload.JobId,
                            Brainrots = payload.Brainrots
                        })
                    end
                end
            end
        end
    end
end)

local InfoButton = InfoButton
InfoButton.MouseButton1Click:Connect(function()
    LogContent.Visible = false
    JoinButton.Visible = false
    Toggle.Visible = false

    local info = RightContainer:FindFirstChild("InfoLabel")

    if not info then
        info = Instance.new("TextLabel")
        info.Name = "InfoLabel"
        info.Parent = RightContainer
        info.BackgroundTransparency = 1
        info.Position = UDim2.new(0, 12, 0, 12)
        info.Size = UDim2.new(1, -24, 1, -24)
        info.Font = Enum.Font.GothamMedium
        info.TextColor3 = COLORS.White
        info.TextSize = 11
        info.TextWrapped = true
        info.TextXAlignment = Enum.TextXAlignment.Left
        info.TextYAlignment = Enum.TextYAlignment.Top
    end

    info.Text =
        "LOG | NOTIFIER\n\n" ..
        "Status : " .. (enabled and "ON" or "OFF") .. "\n" ..
        "Logs : " .. tostring(logCount) .. "\n\n" ..
        "PlaceId : " .. tostring(lastPlaceId or "N/A") .. "\n" ..
        "JobId : " .. tostring(lastJobId or "N/A")

    info.Visible = true
end)

HomeButton.MouseButton1Click:Connect(function()
    local info = RightContainer:FindFirstChild("InfoLabel")
    if info then info.Visible = false end

    LogContent.Visible = true
    JoinButton.Visible = true
    Toggle.Visible = true
end)

print("LOG | NOTIFIER loaded")

local InputService = game:GetService('UserInputService');
local TextService = game:GetService('TextService');
local CoreGui = game:GetService('CoreGui');
local Teams = game:GetService('Teams');
local Players = game:GetService('Players');
local RunService = game:GetService('RunService')
local TweenService = game:GetService('TweenService');
local RenderStepped = RunService.RenderStepped;
local LocalPlayer = Players.LocalPlayer;
local Mouse = LocalPlayer:GetMouse();

local ProtectGui = protectgui or (syn and syn.protect_gui) or (function() end);

local ScreenGui = Instance.new('ScreenGui');
ProtectGui(ScreenGui);

ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Global;
ScreenGui.Parent = CoreGui;

-- popups (toggle settings) live in their own gui above the menu so their elements can use normal z-indexes
local PopupGui = Instance.new('ScreenGui');
ProtectGui(PopupGui);

PopupGui.DisplayOrder = 10;
PopupGui.ZIndexBehavior = Enum.ZIndexBehavior.Global;
PopupGui.Parent = CoreGui;

local Toggles = {};
local Options = {};

getgenv().Toggles = Toggles;
getgenv().Options = Options;

local Library = {
    Registry = {};
    RegistryMap = {};

    HudRegistry = {};

    FontColor = Color3.fromRGB(255, 255, 255);
    DimColor = Color3.fromRGB(130, 130, 130);
    MainColor = Color3.fromRGB(24, 24, 24);
    BackgroundColor = Color3.fromRGB(15, 15, 15);
    AccentColor = Color3.fromRGB(200, 165, 175);
    OutlineColor = Color3.fromRGB(38, 38, 38);
    RiskColor = Color3.fromRGB(255, 50, 50),

    Black = Color3.new(0, 0, 0);
    Font = Enum.Font.Gotham,

    OpenedFrames = {};
    DependencyBoxes = {};

    Signals = {};
    ScreenGui = ScreenGui;
};

local RainbowStep = 0
local Hue = 0

table.insert(Library.Signals, RenderStepped:Connect(function(Delta)
    RainbowStep = RainbowStep + Delta

    if RainbowStep >= (1 / 60) then
        RainbowStep = 0

        Hue = Hue + (1 / 400);

        if Hue > 1 then
            Hue = 0;
        end;

        Library.CurrentRainbowHue = Hue;
        Library.CurrentRainbowColor = Color3.fromHSV(Hue, 0.8, 1);
    end
end))

local function GetPlayersString()
    local PlayerList = Players:GetPlayers();

    for i = 1, #PlayerList do
        PlayerList[i] = PlayerList[i].Name;
    end;

    table.sort(PlayerList, function(str1, str2) return str1 < str2 end);

    return PlayerList;
end;

local function GetTeamsString()
    local TeamList = Teams:GetTeams();

    for i = 1, #TeamList do
        TeamList[i] = TeamList[i].Name;
    end;

    table.sort(TeamList, function(str1, str2) return str1 < str2 end);
    
    return TeamList;
end;

function Library:SafeCallback(f, ...)
    if (not f) then
        return;
    end;

    if not Library.NotifyOnError then
        return f(...);
    end;

    local success, event = pcall(f, ...);

    if not success then
        local _, i = event:find(":%d+: ");

        if not i then
            return Library:Notify(event);
        end;

        return Library:Notify(event:sub(i + 1), 3);
    end;
end;

function Library:AttemptSave()
    if Library.SaveManager then
        Library.SaveManager:Save();
    end;
end;

function Library:Create(Class, Properties)
    local _Instance = Class;

    if type(Class) == 'string' then
        _Instance = Instance.new(Class);
    end;

    for Property, Value in next, Properties do
        _Instance[Property] = Value;
    end;

    return _Instance;
end;

-- Plays a quick tween on an instance (used for all the small element animations)
function Library:Tween(Inst, Properties, Time, Style, Direction)
    local Tween = TweenService:Create(Inst, TweenInfo.new(Time or 0.15, Style or Enum.EasingStyle.Quad, Direction or Enum.EasingDirection.Out), Properties);
    Tween:Play();

    return Tween;
end;

-- Rounds an element. UICorner does not round the built-in border,
-- so the border is swapped for a UIStroke that follows BorderColor3.
function Library:Round(Inst, Radius)
    Library:Create('UICorner', {
        CornerRadius = UDim.new(0, Radius or 4);
        Parent = Inst;
    });

    if Inst.BorderSizePixel > 0 then
        local Stroke = Library:Create('UIStroke', {
            ApplyStrokeMode = Enum.ApplyStrokeMode.Border;
            Color = Inst.BorderColor3;
            Thickness = Inst.BorderSizePixel;
            Parent = Inst;
        });

        Inst.BorderSizePixel = 0;

        Inst:GetPropertyChangedSignal('BorderColor3'):Connect(function()
            Stroke.Color = Inst.BorderColor3;
        end);
    end;

    return Inst;
end;

function Library:ApplyTextStroke(Inst)
    -- flat text (no outline) to match the new style
    Inst.TextStrokeTransparency = 1;
end;

function Library:CreateLabel(Properties, IsHud)
    local _Instance = Library:Create('TextLabel', {
        BackgroundTransparency = 1;
        Font = Library.Font;
        TextColor3 = Library.FontColor;
        TextSize = 16;
        TextStrokeTransparency = 0;
    });

    Library:ApplyTextStroke(_Instance);

    Library:AddToRegistry(_Instance, {
        TextColor3 = 'FontColor';
    }, IsHud);

    return Library:Create(_Instance, Properties);
end;

function Library:MakeDraggable(Instance, Cutoff)
    Instance.Active = true;

    Instance.InputBegan:Connect(function(Input)
        if Input.UserInputType == Enum.UserInputType.MouseButton1 then
            local ObjPos = Vector2.new(
                Mouse.X - Instance.AbsolutePosition.X,
                Mouse.Y - Instance.AbsolutePosition.Y
            );

            if ObjPos.Y > (Cutoff or 40) then
                return;
            end;

            while InputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) do
                Instance.Position = UDim2.new(
                    0,
                    Mouse.X - ObjPos.X + (Instance.Size.X.Offset * Instance.AnchorPoint.X),
                    0,
                    Mouse.Y - ObjPos.Y + (Instance.Size.Y.Offset * Instance.AnchorPoint.Y)
                );

                RenderStepped:Wait();
            end;
        end;
    end)
end;

function Library:AddToolTip(InfoStr, HoverInstance)
    local X, Y = Library:GetTextBounds(InfoStr, Library.Font, 14);
    local Tooltip = Library:Create('Frame', {
        BackgroundColor3 = Library.MainColor,
        BorderColor3 = Library.OutlineColor,

        Size = UDim2.fromOffset(X + 5, Y + 4),
        ZIndex = 100,
        Parent = Library.ScreenGui,

        Visible = false,
    })

    local Label = Library:CreateLabel({
        Position = UDim2.fromOffset(3, 1),
        Size = UDim2.fromOffset(X, Y);
        TextSize = 14;
        Text = InfoStr,
        TextColor3 = Library.FontColor,
        TextXAlignment = Enum.TextXAlignment.Left;
        ZIndex = Tooltip.ZIndex + 1,

        Parent = Tooltip;
    });

    Library:AddToRegistry(Tooltip, {
        BackgroundColor3 = 'MainColor';
        BorderColor3 = 'OutlineColor';
    });

    Library:AddToRegistry(Label, {
        TextColor3 = 'FontColor',
    });

    local IsHovering = false

    HoverInstance.MouseEnter:Connect(function()
        if Library:MouseIsOverOpenedFrame() then
            return
        end

        IsHovering = true

        Tooltip.Position = UDim2.fromOffset(Mouse.X + 15, Mouse.Y + 12)
        Tooltip.Visible = true

        while IsHovering do
            RunService.Heartbeat:Wait()
            Tooltip.Position = UDim2.fromOffset(Mouse.X + 15, Mouse.Y + 12)
        end
    end)

    HoverInstance.MouseLeave:Connect(function()
        IsHovering = false
        Tooltip.Visible = false
    end)
end

function Library:OnHighlight(HighlightInstance, Instance, Properties, PropertiesDefault)
    -- colors fade, everything else (ZIndex etc) is set right away
    local function Apply(Set)
        local Reg = Library.RegistryMap[Instance];
        local Fade = {};

        for Property, ColorIdx in next, Set do
            local Value = Library[ColorIdx] or ColorIdx;

            if typeof(Value) == 'Color3' then
                Fade[Property] = Value;
            else
                Instance[Property] = Value;
            end;

            if Reg and Reg.Properties[Property] then
                Reg.Properties[Property] = ColorIdx;
            end;
        end;

        if next(Fade) then
            Library:Tween(Instance, Fade, 0.12);
        end;
    end;

    HighlightInstance.MouseEnter:Connect(function()
        Apply(Properties);
    end)

    HighlightInstance.MouseLeave:Connect(function()
        Apply(PropertiesDefault);
    end)
end;

function Library:MouseIsOverOpenedFrame()
    for Frame, _ in next, Library.OpenedFrames do
        local AbsPos, AbsSize = Frame.AbsolutePosition, Frame.AbsoluteSize;

        if Mouse.X >= AbsPos.X and Mouse.X <= AbsPos.X + AbsSize.X
            and Mouse.Y >= AbsPos.Y and Mouse.Y <= AbsPos.Y + AbsSize.Y then

            return true;
        end;
    end;
end;

function Library:IsMouseOverFrame(Frame)
    local AbsPos, AbsSize = Frame.AbsolutePosition, Frame.AbsoluteSize;

    if Mouse.X >= AbsPos.X and Mouse.X <= AbsPos.X + AbsSize.X
        and Mouse.Y >= AbsPos.Y and Mouse.Y <= AbsPos.Y + AbsSize.Y then

        return true;
    end;
end;

Library.Popups = {};

-- closes every popup except `Keep` and the popups it was opened from
function Library:ClosePopups(Keep)
    local Safe = {};
    local Current = Keep;

    while Current do
        Safe[Current] = true;
        Current = Current.ParentPopup;
    end;

    for _, Popup in next, Library.Popups do
        if not Safe[Popup] then
            Popup:SetOpen(false);
        end;
    end;
end;

function Library:UpdateDependencyBoxes()
    for _, Depbox in next, Library.DependencyBoxes do
        Depbox:Update();
    end;
end;

function Library:MapValue(Value, MinA, MaxA, MinB, MaxB)
    return (1 - ((Value - MinA) / (MaxA - MinA))) * MinB + ((Value - MinA) / (MaxA - MinA)) * MaxB;
end;

function Library:GetTextBounds(Text, Font, Size, Resolution)
    local Bounds = TextService:GetTextSize(Text, Size, Font, Resolution or Vector2.new(1920, 1080))
    return Bounds.X, Bounds.Y
end;

function Library:GetDarkerColor(Color)
    local H, S, V = Color3.toHSV(Color);
    return Color3.fromHSV(H, S, V / 1.5);
end;
Library.AccentColorDark = Library:GetDarkerColor(Library.AccentColor);

function Library:AddToRegistry(Instance, Properties, IsHud)
    local Idx = #Library.Registry + 1;
    local Data = {
        Instance = Instance;
        Properties = Properties;
        Idx = Idx;
    };

    table.insert(Library.Registry, Data);
    Library.RegistryMap[Instance] = Data;

    if IsHud then
        table.insert(Library.HudRegistry, Data);
    end;
end;

function Library:RemoveFromRegistry(Instance)
    local Data = Library.RegistryMap[Instance];

    if Data then
        for Idx = #Library.Registry, 1, -1 do
            if Library.Registry[Idx] == Data then
                table.remove(Library.Registry, Idx);
            end;
        end;

        for Idx = #Library.HudRegistry, 1, -1 do
            if Library.HudRegistry[Idx] == Data then
                table.remove(Library.HudRegistry, Idx);
            end;
        end;

        Library.RegistryMap[Instance] = nil;
    end;
end;

function Library:UpdateColorsUsingRegistry()
    -- TODO: Could have an 'active' list of objects
    -- where the active list only contains Visible objects.

    -- IMPL: Could setup .Changed events on the AddToRegistry function
    -- that listens for the 'Visible' propert being changed.
    -- Visible: true => Add to active list, and call UpdateColors function
    -- Visible: false => Remove from active list.

    -- The above would be especially efficient for a rainbow menu color or live color-changing.

    for Idx, Object in next, Library.Registry do
        for Property, ColorIdx in next, Object.Properties do
            if type(ColorIdx) == 'string' then
                Object.Instance[Property] = Library[ColorIdx];
            elseif type(ColorIdx) == 'function' then
                Object.Instance[Property] = ColorIdx()
            end
        end;
    end;
end;

function Library:GiveSignal(Signal)
    -- Only used for signals not attached to library instances, as those should be cleaned up on object destruction by Roblox
    table.insert(Library.Signals, Signal)
end

function Library:Unload()
    -- Unload all of the signals
    for Idx = #Library.Signals, 1, -1 do
        local Connection = table.remove(Library.Signals, Idx)
        Connection:Disconnect()
    end

     -- Call our unload callback, maybe to undo some hooks etc
    if Library.OnUnload then
        Library.OnUnload()
    end

    ScreenGui:Destroy()
    PopupGui:Destroy()
end

function Library:OnUnload(Callback)
    Library.OnUnload = Callback
end

Library:GiveSignal(ScreenGui.DescendantRemoving:Connect(function(Instance)
    if Library.RegistryMap[Instance] then
        Library:RemoveFromRegistry(Instance);
    end;
end))

local BaseAddons = {};

do
    local Funcs = {};

    function Funcs:AddColorPicker(Idx, Info)
        local ToggleLabel = self.TextLabel;
        -- local Container = self.Container;

        assert(Info.Default, 'AddColorPicker: Missing default value.');

        local ColorPicker = {
            Value = Info.Default;
            Transparency = Info.Transparency or 0;
            Type = 'ColorPicker';
            Title = type(Info.Title) == 'string' and Info.Title or 'Color picker',
            Callback = Info.Callback or function(Color) end;
        };

        function ColorPicker:SetHSVFromRGB(Color)
            local H, S, V = Color3.toHSV(Color);

            ColorPicker.Hue = H;
            ColorPicker.Sat = S;
            ColorPicker.Vib = V;
        end;

        -- effects: the picker always edits one of two colors; the output (Value) can animate between them
        ColorPicker.Colors = { ColorPicker.Value, Library:GetDarkerColor(ColorPicker.Value) };
        ColorPicker.Editing = 1;
        ColorPicker.Mode = 'static'; -- static / rainbow / pulse
        ColorPicker.Speed = 1;

        ColorPicker:SetHSVFromRGB(ColorPicker.Value);

        -- round swatch with a thin ring (the ring follows BorderColor3, see Library:Round)
        local DisplayFrame = Library:Create('Frame', {
            BackgroundColor3 = ColorPicker.Value;
            BorderColor3 = Library:GetDarkerColor(ColorPicker.Value);
            Size = UDim2.new(0, 16, 0, 16);
            ZIndex = 6;
            Parent = ToggleLabel;
        });
        Library:Round(DisplayFrame, 4);

        -- small pop on hover
        local DisplayScale = Library:Create('UIScale', {
            Scale = 1;
            Parent = DisplayFrame;
        });

        DisplayFrame.MouseEnter:Connect(function()
            TweenService:Create(DisplayScale, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), { Scale = 1.15 }):Play();
        end);

        DisplayFrame.MouseLeave:Connect(function()
            TweenService:Create(DisplayScale, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), { Scale = 1 }):Play();
        end);

        -- Transparency image taken from https://github.com/matas3535/SplixPrivateDrawingLibrary/blob/main/Library.lua cus i'm lazy
        local CheckerFrame = Library:Create('ImageLabel', {
            BorderSizePixel = 0;
            Size = UDim2.new(1, 0, 1, 0);
            ZIndex = 5;
            Image = 'http://www.roblox.com/asset/?id=12977615774';
            Visible = not not Info.Transparency;
            Parent = DisplayFrame;
        });

        Library:Create('UICorner', {
            CornerRadius = UDim.new(0, 4);
            Parent = CheckerFrame;
        });

        -- 1/16/23
        -- Rewrote this to be placed inside the Library ScreenGui
        -- There was some issue which caused RelativeOffset to be way off
        -- Thus the color picker would never show

        local PickerFrameOuter = Library:Create('Frame', {
            Name = 'Color';
            BackgroundTransparency = 1; -- was white and showed through as a line at the top edge
            BackgroundColor3 = Color3.new(1, 1, 1);
            BorderSizePixel = 0; -- the inner frame has the only outline
            Position = UDim2.fromOffset(DisplayFrame.AbsolutePosition.X, DisplayFrame.AbsolutePosition.Y + 18),
            Size = UDim2.fromOffset(230, Info.Transparency and 271 or 253);
            Visible = false;
            ZIndex = 15;
            Parent = ToggleLabel:FindFirstAncestorOfClass('ScreenGui') or ScreenGui,
        });
        Library:Round(PickerFrameOuter, 4);

        DisplayFrame:GetPropertyChangedSignal('AbsolutePosition'):Connect(function()
            PickerFrameOuter.Position = UDim2.fromOffset(DisplayFrame.AbsolutePosition.X, DisplayFrame.AbsolutePosition.Y + 18);
        end)

        local PickerFrameInner = Library:Create('Frame', {
            BackgroundColor3 = Library.BackgroundColor;
            BorderColor3 = Library.OutlineColor;
            BorderMode = Enum.BorderMode.Inset;
            Size = UDim2.new(1, 0, 1, 0);
            ZIndex = 16;
            Parent = PickerFrameOuter;
        });
        Library:Round(PickerFrameInner, 4);

        local Highlight = Library:Create('Frame', {
            BackgroundColor3 = Library.AccentColor;
            BackgroundTransparency = 1; -- accent line removed
            BorderSizePixel = 0;
            Position = UDim2.new(0, 4, 0, 0);
            Size = UDim2.new(1, -8, 0, 2);
            ZIndex = 17;
            Parent = PickerFrameInner;
        });

        local SatVibMapOuter = Library:Create('Frame', {
            BorderColor3 = Color3.new(0, 0, 0);
            Position = UDim2.new(0, 4, 0, 25);
            Size = UDim2.new(0, 200, 0, 200);
            ZIndex = 17;
            Parent = PickerFrameInner;
        });

        local SatVibMapInner = Library:Create('Frame', {
            BackgroundColor3 = Library.BackgroundColor;
            BorderColor3 = Library.OutlineColor;
            BorderMode = Enum.BorderMode.Inset;
            Size = UDim2.new(1, 0, 1, 0);
            ZIndex = 18;
            Parent = SatVibMapOuter;
        });

        local SatVibMap = Library:Create('ImageLabel', {
            BorderSizePixel = 0;
            Size = UDim2.new(1, 0, 1, 0);
            ZIndex = 18;
            Image = 'rbxassetid://4155801252';
            Parent = SatVibMapInner;
        });

        local CursorOuter = Library:Create('ImageLabel', {
            AnchorPoint = Vector2.new(0.5, 0.5);
            Size = UDim2.new(0, 6, 0, 6);
            BackgroundTransparency = 1;
            Image = 'http://www.roblox.com/asset/?id=9619665977';
            ImageColor3 = Color3.new(0, 0, 0);
            ZIndex = 19;
            Parent = SatVibMap;
        });

        local CursorInner = Library:Create('ImageLabel', {
            Size = UDim2.new(0, CursorOuter.Size.X.Offset - 2, 0, CursorOuter.Size.Y.Offset - 2);
            Position = UDim2.new(0, 1, 0, 1);
            BackgroundTransparency = 1;
            Image = 'http://www.roblox.com/asset/?id=9619665977';
            ZIndex = 20;
            Parent = CursorOuter;
        })

        local HueSelectorOuter = Library:Create('Frame', {
            BorderColor3 = Color3.new(0, 0, 0);
            Position = UDim2.new(0, 208, 0, 25);
            Size = UDim2.new(0, 15, 0, 200);
            ZIndex = 17;
            Parent = PickerFrameInner;
        });

        local HueSelectorInner = Library:Create('Frame', {
            BackgroundColor3 = Color3.new(1, 1, 1);
            BorderSizePixel = 0;
            Size = UDim2.new(1, 0, 1, 0);
            ZIndex = 18;
            Parent = HueSelectorOuter;
        });

        local HueCursor = Library:Create('Frame', { 
            BackgroundColor3 = Color3.new(1, 1, 1);
            AnchorPoint = Vector2.new(0, 0.5);
            BorderColor3 = Color3.new(0, 0, 0);
            Size = UDim2.new(1, 0, 0, 1);
            ZIndex = 18;
            Parent = HueSelectorInner;
        });

        local HueBoxOuter = Library:Create('Frame', {
            BorderColor3 = Color3.new(0, 0, 0);
            Position = UDim2.fromOffset(4, 228),
            Size = UDim2.fromOffset(98, 20),
            ZIndex = 18,
            Parent = PickerFrameInner;
        });

        local HueBoxInner = Library:Create('Frame', {
            BackgroundColor3 = Library.MainColor;
            BorderColor3 = Library.OutlineColor;
            BorderMode = Enum.BorderMode.Inset;
            Size = UDim2.new(1, 0, 1, 0);
            ZIndex = 18,
            Parent = HueBoxOuter;
        });

        Library:Create('UIGradient', {
            Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0, Color3.new(1, 1, 1)),
                ColorSequenceKeypoint.new(1, Color3.new(1, 1, 1))
            });
            Rotation = 90;
            Parent = HueBoxInner;
        });

        local HueBox = Library:Create('TextBox', {
            BackgroundTransparency = 1;
            Position = UDim2.new(0, 5, 0, 0);
            Size = UDim2.new(1, -5, 1, 0);
            Font = Library.Font;
            PlaceholderColor3 = Color3.fromRGB(190, 190, 190);
            PlaceholderText = 'Hex color',
            Text = '#FFFFFF',
            TextColor3 = Library.FontColor;
            TextSize = 14;
            TextStrokeTransparency = 0;
            TextXAlignment = Enum.TextXAlignment.Left;
            ZIndex = 20,
            Parent = HueBoxInner;
        });

        Library:ApplyTextStroke(HueBox);

        local RgbBoxBase = Library:Create(HueBoxOuter:Clone(), {
            Position = UDim2.fromOffset(106, 228),
            Size = UDim2.fromOffset(98, 20),
            Parent = PickerFrameInner
        });

        local RgbBox = Library:Create(RgbBoxBase.Frame:FindFirstChild('TextBox'), {
            Text = '255, 255, 255',
            PlaceholderText = 'RGB color',
            TextColor3 = Library.FontColor
        });

        local TransparencyBoxOuter, TransparencyBoxInner, TransparencyCursor;
        
        if Info.Transparency then 
            TransparencyBoxOuter = Library:Create('Frame', {
                BorderColor3 = Color3.new(0, 0, 0);
                Position = UDim2.fromOffset(4, 251);
                Size = UDim2.new(1, -8, 0, 15);
                ZIndex = 19;
                Parent = PickerFrameInner;
            });

            TransparencyBoxInner = Library:Create('Frame', {
                BackgroundColor3 = ColorPicker.Value;
                BorderColor3 = Library.OutlineColor;
                BorderMode = Enum.BorderMode.Inset;
                Size = UDim2.new(1, 0, 1, 0);
                ZIndex = 19;
                Parent = TransparencyBoxOuter;
            });

            Library:AddToRegistry(TransparencyBoxInner, { BorderColor3 = 'OutlineColor' });

            Library:Create('ImageLabel', {
                BackgroundTransparency = 1;
                Size = UDim2.new(1, 0, 1, 0);
                Image = 'http://www.roblox.com/asset/?id=12978095818';
                ZIndex = 20;
                Parent = TransparencyBoxInner;
            });

            TransparencyCursor = Library:Create('Frame', { 
                BackgroundColor3 = Color3.new(1, 1, 1);
                AnchorPoint = Vector2.new(0.5, 0);
                BorderColor3 = Color3.new(0, 0, 0);
                Size = UDim2.new(0, 1, 1, 0);
                ZIndex = 21;
                Parent = TransparencyBoxInner;
            });
        end;

        local DisplayLabel = Library:CreateLabel({
            Size = UDim2.new(1, 0, 0, 14);
            Position = UDim2.fromOffset(5, 5);
            TextXAlignment = Enum.TextXAlignment.Left;
            TextSize = 14;
            Text = ColorPicker.Title,--Info.Default;
            TextWrapped = false;
            ZIndex = 16;
            Parent = PickerFrameInner;
        });


        local ContextMenu = {}
        do
            ContextMenu.Options = {}
            ContextMenu.Container = Library:Create('Frame', {
                BorderColor3 = Color3.new(),
                ZIndex = 14,

                Visible = false,
                Parent = ToggleLabel:FindFirstAncestorOfClass('ScreenGui') or ScreenGui
            })

            ContextMenu.Inner = Library:Create('Frame', {
                BackgroundColor3 = Library.BackgroundColor;
                BorderColor3 = Library.OutlineColor;
                BorderMode = Enum.BorderMode.Inset;
                Size = UDim2.fromScale(1, 1);
                ZIndex = 15;
                Parent = ContextMenu.Container;
            });

            Library:Create('UIListLayout', {
                Name = 'Layout',
                FillDirection = Enum.FillDirection.Vertical;
                SortOrder = Enum.SortOrder.LayoutOrder;
                Parent = ContextMenu.Inner;
            });

            Library:Create('UIPadding', {
                Name = 'Padding',
                PaddingLeft = UDim.new(0, 4),
                Parent = ContextMenu.Inner,
            });

            local function updateMenuPosition()
                ContextMenu.Container.Position = UDim2.fromOffset(
                    (DisplayFrame.AbsolutePosition.X + DisplayFrame.AbsoluteSize.X) + 4,
                    DisplayFrame.AbsolutePosition.Y + 1
                )
            end

            local function updateMenuSize()
                local menuWidth = 60
                for i, label in next, ContextMenu.Inner:GetChildren() do
                    if label:IsA('TextLabel') then
                        menuWidth = math.max(menuWidth, label.TextBounds.X)
                    end
                end

                ContextMenu.Container.Size = UDim2.fromOffset(
                    menuWidth + 8,
                    ContextMenu.Inner.Layout.AbsoluteContentSize.Y + 4
                )
            end

            DisplayFrame:GetPropertyChangedSignal('AbsolutePosition'):Connect(updateMenuPosition)
            ContextMenu.Inner.Layout:GetPropertyChangedSignal('AbsoluteContentSize'):Connect(updateMenuSize)

            task.spawn(updateMenuPosition)
            task.spawn(updateMenuSize)

            Library:AddToRegistry(ContextMenu.Inner, {
                BackgroundColor3 = 'BackgroundColor';
                BorderColor3 = 'OutlineColor';
            });

            function ContextMenu:Show()
                self.Container.Visible = true
            end

            function ContextMenu:Hide()
                self.Container.Visible = false
            end

            function ContextMenu:AddOption(Str, Callback)
                if type(Callback) ~= 'function' then
                    Callback = function() end
                end

                local Button = Library:CreateLabel({
                    Active = false;
                    Size = UDim2.new(1, 0, 0, 15);
                    TextSize = 13;
                    Text = Str;
                    ZIndex = 16;
                    Parent = self.Inner;
                    TextXAlignment = Enum.TextXAlignment.Left,
                });

                Library:OnHighlight(Button, Button, 
                    { TextColor3 = 'AccentColor' },
                    { TextColor3 = 'FontColor' }
                );

                Button.InputBegan:Connect(function(Input)
                    if Input.UserInputType ~= Enum.UserInputType.MouseButton1 then
                        return
                    end

                    Callback()
                end)
            end

            ContextMenu:AddOption('Copy color', function()
                Library.ColorClipboard = ColorPicker.Value
                Library:Notify('Copied color!', 2)
            end)

            ContextMenu:AddOption('Paste color', function()
                if not Library.ColorClipboard then
                    return Library:Notify('You have not copied a color!', 2)
                end
                ColorPicker:SetValueRGB(Library.ColorClipboard)
            end)


            ContextMenu:AddOption('Copy HEX', function()
                pcall(setclipboard, ColorPicker.Value:ToHex())
                Library:Notify('Copied hex code to clipboard!', 2)
            end)

            ContextMenu:AddOption('Copy RGB', function()
                pcall(setclipboard, table.concat({ math.floor(ColorPicker.Value.R * 255), math.floor(ColorPicker.Value.G * 255), math.floor(ColorPicker.Value.B * 255) }, ', '))
                Library:Notify('Copied RGB values to clipboard!', 2)
            end)

        end

        Library:AddToRegistry(PickerFrameInner, { BackgroundColor3 = 'BackgroundColor'; BorderColor3 = 'OutlineColor'; });
        Library:AddToRegistry(Highlight, { BackgroundColor3 = 'AccentColor'; });
        Library:AddToRegistry(SatVibMapInner, { BackgroundColor3 = 'BackgroundColor'; BorderColor3 = 'OutlineColor'; });

        Library:AddToRegistry(HueBoxInner, { BackgroundColor3 = 'MainColor'; BorderColor3 = 'OutlineColor'; });
        Library:AddToRegistry(RgbBoxBase.Frame, { BackgroundColor3 = 'MainColor'; BorderColor3 = 'OutlineColor'; });
        Library:AddToRegistry(RgbBox, { TextColor3 = 'FontColor', });
        Library:AddToRegistry(HueBox, { TextColor3 = 'FontColor', });

        local SequenceTable = {};

        for Hue = 0, 1, 0.1 do
            table.insert(SequenceTable, ColorSequenceKeypoint.new(Hue, Color3.fromHSV(Hue, 1, 1)));
        end;

        local HueSelectorGradient = Library:Create('UIGradient', {
            Color = ColorSequence.new(SequenceTable);
            Rotation = 90;
            Parent = HueSelectorInner;
        });

        HueBox.FocusLost:Connect(function(enter)
            if enter then
                local success, result = pcall(Color3.fromHex, HueBox.Text)
                if success and typeof(result) == 'Color3' then
                    ColorPicker.Hue, ColorPicker.Sat, ColorPicker.Vib = Color3.toHSV(result)
                end
            end

            ColorPicker:Display()
        end)

        RgbBox.FocusLost:Connect(function(enter)
            if enter then
                local r, g, b = RgbBox.Text:match('(%d+),%s*(%d+),%s*(%d+)')
                if r and g and b then
                    ColorPicker.Hue, ColorPicker.Sat, ColorPicker.Vib = Color3.toHSV(Color3.fromRGB(r, g, b))
                end
            end

            ColorPicker:Display()
        end)

        -- side panel (rainbow / pulse speed, copy / paste / undo) and the color bars -----------
        local ExtraY = Info.Transparency and 271 or 253;
        local PanelX, PanelWidth = 232, 94;
        local MAX_SPEED = 5;

        local function MakeSmallButton(Text, X, Y, Width, Func)
            local Outer = Library:Create('Frame', {
                BackgroundColor3 = Library.MainColor;
                BorderColor3 = Library.OutlineColor;
                Position = UDim2.fromOffset(X, Y);
                Size = UDim2.fromOffset(Width, 20);
                ZIndex = 18;
                Parent = PickerFrameInner;
            });
            Library:Round(Outer, 3);

            Library:AddToRegistry(Outer, {
                BackgroundColor3 = 'MainColor';
                BorderColor3 = 'OutlineColor';
            });

            Library:CreateLabel({
                Size = UDim2.new(1, 0, 1, 0);
                Text = Text;
                TextSize = 13;
                ZIndex = 19;
                Parent = Outer;
            });

            Outer.InputBegan:Connect(function(Input)
                if Input.UserInputType == Enum.UserInputType.MouseButton1 then
                    Func();
                end;
            end);

            Library:OnHighlight(Outer, Outer,
                { BorderColor3 = 'AccentColor' },
                { BorderColor3 = 'OutlineColor' }
            );

            return Outer;
        end;

        -- undo history ---------------------------------------------------------------------
        ColorPicker.History = {};

        function ColorPicker:PushHistory()
            local History = ColorPicker.History;

            table.insert(History, {
                Colors = { ColorPicker.Colors[1], ColorPicker.Colors[2] };
                Editing = ColorPicker.Editing;
                Mode = ColorPicker.Mode;
                Speed = ColorPicker.Speed;
                Transparency = ColorPicker.Transparency;
            });

            if #History > 25 then
                table.remove(History, 1);
            end;
        end;

        function ColorPicker:Undo()
            local State = table.remove(ColorPicker.History);

            if not State then
                return Library:Notify('Nothing to undo', 2);
            end;

            ColorPicker.Colors = State.Colors;
            ColorPicker.Editing = State.Editing;
            ColorPicker.Mode = State.Mode;
            ColorPicker.Speed = State.Speed;
            ColorPicker.Transparency = State.Transparency;

            ColorPicker:SetHSVFromRGB(ColorPicker.Colors[ColorPicker.Editing]);
            ColorPicker:UpdateLayout();
            ColorPicker:Display();
        end;

        -- remember the color from before each edit starts
        SatVibMap.InputBegan:Connect(function(Input)
            if Input.UserInputType == Enum.UserInputType.MouseButton1 then
                ColorPicker:PushHistory();
            end;
        end);

        HueSelectorInner.InputBegan:Connect(function(Input)
            if Input.UserInputType == Enum.UserInputType.MouseButton1 then
                ColorPicker:PushHistory();
            end;
        end);

        HueBox.Focused:Connect(function() ColorPicker:PushHistory(); end);
        RgbBox.Focused:Connect(function() ColorPicker:PushHistory(); end);

        -- rainbow / pulse speed sliders (0 = off, the two exclude each other) --------------------
        local EffectSliders = {};

        local function MakeEffectSlider(Text, Y, ModeName)
            Library:CreateLabel({
                Position = UDim2.fromOffset(PanelX, Y);
                Size = UDim2.fromOffset(PanelWidth, 14);
                Text = Text;
                TextSize = 13;
                TextXAlignment = Enum.TextXAlignment.Left;
                ZIndex = 18;
                Parent = PickerFrameInner;
            });

            local ValueLabel = Library:CreateLabel({
                Position = UDim2.fromOffset(PanelX, Y);
                Size = UDim2.fromOffset(PanelWidth, 14);
                Text = '0';
                TextSize = 13;
                TextXAlignment = Enum.TextXAlignment.Right;
                ZIndex = 18;
                Parent = PickerFrameInner;
            });

            local Track = Library:Create('Frame', {
                BackgroundColor3 = Library.MainColor;
                BorderColor3 = Library.OutlineColor;
                Position = UDim2.fromOffset(PanelX, Y + 17);
                Size = UDim2.fromOffset(PanelWidth, 6);
                ZIndex = 18;
                Parent = PickerFrameInner;
            });
            Library:Round(Track, 3);

            Library:AddToRegistry(Track, {
                BackgroundColor3 = 'MainColor';
                BorderColor3 = 'OutlineColor';
            });

            local Fill = Library:Create('Frame', {
                BackgroundColor3 = Library.AccentColor;
                BorderSizePixel = 0;
                Size = UDim2.new(0, 0, 1, 0);
                ZIndex = 19;
                Parent = Track;
            });
            Library:Round(Fill, 3);

            Library:AddToRegistry(Fill, {
                BackgroundColor3 = 'AccentColor';
            });

            -- taller invisible hit area so the thin track is easy to grab
            local Hit = Library:Create('Frame', {
                BackgroundTransparency = 1;
                Position = UDim2.fromOffset(PanelX, Y);
                Size = UDim2.fromOffset(PanelWidth, 26);
                ZIndex = 20;
                Parent = PickerFrameInner;
            });

            Hit.InputBegan:Connect(function(Input)
                if Input.UserInputType == Enum.UserInputType.MouseButton1 then
                    ColorPicker:PushHistory();

                    while InputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) do
                        local Alpha = math.clamp((Mouse.X - Track.AbsolutePosition.X) / math.max(Track.AbsoluteSize.X, 1), 0, 1);
                        local Speed = math.floor(Alpha * MAX_SPEED * 10 + 0.5) / 10;

                        if Speed < 0.1 then
                            ColorPicker:SetEffect('static');
                        else
                            ColorPicker:SetEffect(ModeName, Speed);
                        end;

                        RenderStepped:Wait();
                    end;

                    Library:AttemptSave();
                end;
            end);

            table.insert(EffectSliders, { Fill = Fill, Value = ValueLabel, Mode = ModeName });
        end;

        MakeEffectSlider('Rainbow', 25, 'rainbow');
        MakeEffectSlider('Pulse', 58, 'pulse');

        MakeSmallButton('Copy', PanelX, 100, PanelWidth, function()
            local Edited = ColorPicker.Colors[ColorPicker.Editing];

            Library.ColorClipboard = Edited;
            pcall(setclipboard, '#' .. Edited:ToHex());
            Library:Notify('Copied color!', 2);
        end);

        MakeSmallButton('Paste', PanelX, 126, PanelWidth, function()
            if not Library.ColorClipboard then
                return Library:Notify('You have not copied a color!', 2);
            end;

            ColorPicker:PushHistory();

            ColorPicker.Hue, ColorPicker.Sat, ColorPicker.Vib = Color3.toHSV(Library.ColorClipboard);
            ColorPicker:Display();
            Library:AttemptSave();
        end);

        MakeSmallButton('Undo', PanelX, 152, PanelWidth, function()
            ColorPicker:Undo();
            Library:AttemptSave();
        end);

        -- bottom bars: color 1, and color 2 once pulse is on. Click one to edit it.
        local Swatches = {};

        for Idx = 1, 2 do
            local Bar = Library:Create('Frame', {
                BackgroundColor3 = ColorPicker.Colors[Idx];
                BorderColor3 = Library.OutlineColor;
                Size = UDim2.fromOffset(326, 14);
                Visible = Idx == 1;
                ZIndex = 18;
                Parent = PickerFrameInner;
            });
            Library:Round(Bar, 3);

            Bar.InputBegan:Connect(function(Input)
                if Input.UserInputType == Enum.UserInputType.MouseButton1 then
                    ColorPicker.Editing = Idx;
                    ColorPicker:SetHSVFromRGB(ColorPicker.Colors[Idx]);
                    ColorPicker:Display();
                end;
            end);

            Swatches[Idx] = Bar;
        end;

        function ColorPicker:UpdateLayout()
            local Pulse = ColorPicker.Mode == 'pulse';

            Swatches[1].Position = UDim2.fromOffset(4, ExtraY - 1);
            Swatches[1].Size = UDim2.fromOffset(Pulse and 160 or 326, 14);

            Swatches[2].Position = UDim2.fromOffset(168, ExtraY - 1);
            Swatches[2].Size = UDim2.fromOffset(162, 14);
            Swatches[2].Visible = Pulse;

            PickerFrameOuter.Size = UDim2.fromOffset(334, ExtraY + 20);
        end;

        -- Mode: 'static' | 'rainbow' | 'pulse'. Color2 is the second color of a pulse.
        function ColorPicker:SetEffect(Mode, Speed, Color2)
            if Mode ~= 'rainbow' and Mode ~= 'pulse' then
                Mode = 'static';
            end;

            ColorPicker.Mode = Mode;

            if type(Speed) == 'number' then
                ColorPicker.Speed = math.clamp(Speed, 0.1, MAX_SPEED);
            end;

            if typeof(Color2) == 'Color3' then
                ColorPicker.Colors[2] = Color2;
            end;

            if Mode ~= 'pulse' and ColorPicker.Editing == 2 then
                ColorPicker.Editing = 1;
            end;

            ColorPicker:SetHSVFromRGB(ColorPicker.Colors[ColorPicker.Editing]);
            ColorPicker:UpdateLayout();
            ColorPicker:Display();
        end;

        local LastFire = 0;

        -- works out the output color from the mode and fires the callbacks when it changed
        function ColorPicker:ApplyEffect(Force)
            local Colors = ColorPicker.Colors;
            local Value;

            if ColorPicker.Mode == 'rainbow' then
                local _, S, V = Color3.toHSV(Colors[1]);
                Value = Color3.fromHSV((os.clock() * 0.2 * ColorPicker.Speed) % 1, math.max(S, 0.6), math.max(V, 0.6));
            elseif ColorPicker.Mode == 'pulse' then
                local Alpha = (math.sin(os.clock() * ColorPicker.Speed * math.pi) + 1) / 2;
                Value = Colors[1]:Lerp(Colors[2], Alpha);
            else
                Value = Colors[1];
            end;

            if not Force then
                if Value == ColorPicker.Value or (os.clock() - LastFire) < (1 / 30) then
                    return;
                end;
            end;

            LastFire = os.clock();
            ColorPicker.Value = Value;

            Library:Create(DisplayFrame, {
                BackgroundColor3 = Value;
                BackgroundTransparency = ColorPicker.Transparency;
                BorderColor3 = Library:GetDarkerColor(Value);
            });

            Library:SafeCallback(ColorPicker.Callback, ColorPicker.Value);
            Library:SafeCallback(ColorPicker.Changed, ColorPicker.Value);
        end;

        Library:GiveSignal(RenderStepped:Connect(function()
            if ColorPicker.Mode ~= 'static' then
                ColorPicker:ApplyEffect();
            end;
        end));

        function ColorPicker:Display()
            local Edited = Color3.fromHSV(ColorPicker.Hue, ColorPicker.Sat, ColorPicker.Vib);
            ColorPicker.Colors[ColorPicker.Editing] = Edited;

            SatVibMap.BackgroundColor3 = Color3.fromHSV(ColorPicker.Hue, 1, 1);

            if TransparencyBoxInner then
                TransparencyBoxInner.BackgroundColor3 = Edited;
                TransparencyCursor.Position = UDim2.new(1 - ColorPicker.Transparency, 0, 0, 0);
            end;

            CursorOuter.Position = UDim2.new(ColorPicker.Sat, 0, 1 - ColorPicker.Vib, 0);
            HueCursor.Position = UDim2.new(0, 0, ColorPicker.Hue, 0);

            HueBox.Text = '#' .. Edited:ToHex();
            RgbBox.Text = table.concat({ math.floor(Edited.R * 255), math.floor(Edited.G * 255), math.floor(Edited.B * 255) }, ', ');

            for Idx, Bar in next, Swatches do
                Bar.BackgroundColor3 = ColorPicker.Colors[Idx];
                Bar.BorderColor3 = (ColorPicker.Mode == 'pulse' and ColorPicker.Editing == Idx) and Library.AccentColor or Library.OutlineColor;
            end;

            for _, Slider in next, EffectSliders do
                local Speed = ColorPicker.Mode == Slider.Mode and ColorPicker.Speed or 0;

                Slider.Fill.Size = UDim2.new(Speed / MAX_SPEED, 0, 1, 0);
                Slider.Value.Text = Speed == 0 and '0' or string.format('%.1f', Speed);
            end;

            ColorPicker:ApplyEffect(true);
        end;

        function ColorPicker:OnChanged(Func)
            ColorPicker.Changed = Func;
            Func(ColorPicker.Value)
        end;

        function ColorPicker:Show()
            for Frame, Val in next, Library.OpenedFrames do
                if Frame.Name == 'Color' then
                    Frame.Visible = false;
                    Library.OpenedFrames[Frame] = nil;
                end;
            end;

            PickerFrameOuter.Visible = true;
            Library.OpenedFrames[PickerFrameOuter] = true;
        end;

        function ColorPicker:Hide()
            PickerFrameOuter.Visible = false;
            Library.OpenedFrames[PickerFrameOuter] = nil;
        end;

        function ColorPicker:SetValue(HSV, Transparency)
            local Color = Color3.fromHSV(HSV[1], HSV[2], HSV[3]);

            ColorPicker.Transparency = Transparency or 0;
            ColorPicker.Editing = 1;
            ColorPicker:SetHSVFromRGB(Color);
            ColorPicker:Display();
        end;

        function ColorPicker:SetValueRGB(Color, Transparency)
            ColorPicker.Transparency = Transparency or 0;
            ColorPicker.Editing = 1;
            ColorPicker:SetHSVFromRGB(Color);
            ColorPicker:Display();
        end;

        SatVibMap.InputBegan:Connect(function(Input)
            if Input.UserInputType == Enum.UserInputType.MouseButton1 then
                while InputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) do
                    local MinX = SatVibMap.AbsolutePosition.X;
                    local MaxX = MinX + SatVibMap.AbsoluteSize.X;
                    local MouseX = math.clamp(Mouse.X, MinX, MaxX);

                    local MinY = SatVibMap.AbsolutePosition.Y;
                    local MaxY = MinY + SatVibMap.AbsoluteSize.Y;
                    local MouseY = math.clamp(Mouse.Y, MinY, MaxY);

                    ColorPicker.Sat = (MouseX - MinX) / (MaxX - MinX);
                    ColorPicker.Vib = 1 - ((MouseY - MinY) / (MaxY - MinY));
                    ColorPicker:Display();

                    RenderStepped:Wait();
                end;

                Library:AttemptSave();
            end;
        end);

        HueSelectorInner.InputBegan:Connect(function(Input)
            if Input.UserInputType == Enum.UserInputType.MouseButton1 then
                while InputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) do
                    local MinY = HueSelectorInner.AbsolutePosition.Y;
                    local MaxY = MinY + HueSelectorInner.AbsoluteSize.Y;
                    local MouseY = math.clamp(Mouse.Y, MinY, MaxY);

                    ColorPicker.Hue = ((MouseY - MinY) / (MaxY - MinY));
                    ColorPicker:Display();

                    RenderStepped:Wait();
                end;

                Library:AttemptSave();
            end;
        end);

        DisplayFrame.InputBegan:Connect(function(Input)
            if Input.UserInputType == Enum.UserInputType.MouseButton1 and not Library:MouseIsOverOpenedFrame() then
                if PickerFrameOuter.Visible then
                    ColorPicker:Hide()
                else
                    ContextMenu:Hide()
                    ColorPicker:Show()
                end;
            elseif Input.UserInputType == Enum.UserInputType.MouseButton2 and not Library:MouseIsOverOpenedFrame() then
                ContextMenu:Show()
                ColorPicker:Hide()
            end
        end);

        if TransparencyBoxInner then
            TransparencyBoxInner.InputBegan:Connect(function(Input)
                if Input.UserInputType == Enum.UserInputType.MouseButton1 then
                    while InputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) do
                        local MinX = TransparencyBoxInner.AbsolutePosition.X;
                        local MaxX = MinX + TransparencyBoxInner.AbsoluteSize.X;
                        local MouseX = math.clamp(Mouse.X, MinX, MaxX);

                        ColorPicker.Transparency = 1 - ((MouseX - MinX) / (MaxX - MinX));

                        ColorPicker:Display();

                        RenderStepped:Wait();
                    end;

                    Library:AttemptSave();
                end;
            end);
        end;

        Library:GiveSignal(InputService.InputBegan:Connect(function(Input)
            if Input.UserInputType == Enum.UserInputType.MouseButton1 then
                local AbsPos, AbsSize = PickerFrameOuter.AbsolutePosition, PickerFrameOuter.AbsoluteSize;

                if Mouse.X < AbsPos.X or Mouse.X > AbsPos.X + AbsSize.X
                    or Mouse.Y < (AbsPos.Y - 20 - 1) or Mouse.Y > AbsPos.Y + AbsSize.Y then

                    ColorPicker:Hide();
                end;

                if not Library:IsMouseOverFrame(ContextMenu.Container) then
                    ContextMenu:Hide()
                end
            end;

            if Input.UserInputType == Enum.UserInputType.MouseButton2 and ContextMenu.Container.Visible then
                if not Library:IsMouseOverFrame(ContextMenu.Container) and not Library:IsMouseOverFrame(DisplayFrame) then
                    ContextMenu:Hide()
                end
            end
        end))

        for _, Swatch in next, Swatches do
            Library:RemoveFromRegistry(Swatch); -- their fill is the color they show, not a theme color
        end;

        ColorPicker:UpdateLayout();
        ColorPicker:Display();
        ColorPicker.DisplayFrame = DisplayFrame

        Options[Idx] = ColorPicker;

        return self;
    end;

    BaseAddons.__index = Funcs;
    BaseAddons.__namecall = function(Table, Key, ...)
        return Funcs[Key](...);
    end;
end;

local BaseGroupbox = {};

do
    local Funcs = {};

    function Funcs:AddBlank(Size)
        local Groupbox = self;
        local Container = Groupbox.Container;

        Library:Create('Frame', {
            BackgroundTransparency = 1;
            Size = UDim2.new(1, 0, 0, Size);
            ZIndex = 1;
            Parent = Container;
        });
    end;

    function Funcs:AddLabel(Text, DoesWrap)
        local Label = {};

        local Groupbox = self;
        local Container = Groupbox.Container;

        local TextLabel = Library:CreateLabel({
            Size = UDim2.new(1, -4, 0, 15);
            TextSize = 14;
            Text = Text;
            TextWrapped = DoesWrap or false,
            TextXAlignment = Enum.TextXAlignment.Left;
            ZIndex = 5;
            Parent = Container;
        });

        if DoesWrap then
            local Y = select(2, Library:GetTextBounds(Text, Library.Font, 14, Vector2.new(TextLabel.AbsoluteSize.X, math.huge)))
            TextLabel.Size = UDim2.new(1, -4, 0, Y)
        else
            Library:Create('UIListLayout', {
                Padding = UDim.new(0, 4);
                FillDirection = Enum.FillDirection.Horizontal;
                HorizontalAlignment = Enum.HorizontalAlignment.Right;
                SortOrder = Enum.SortOrder.LayoutOrder;
                Parent = TextLabel;
            });
        end

        Label.TextLabel = TextLabel;
        Label.Container = Container;

        function Label:SetText(Text)
            TextLabel.Text = Text

            if DoesWrap then
                local Y = select(2, Library:GetTextBounds(Text, Library.Font, 14, Vector2.new(TextLabel.AbsoluteSize.X, math.huge)))
                TextLabel.Size = UDim2.new(1, -4, 0, Y)
            end

            Groupbox:Resize();
        end

        if (not DoesWrap) then
            setmetatable(Label, BaseAddons);
        end

        Groupbox:AddBlank(5);
        Groupbox:Resize();

        return Label;
    end;

    function Funcs:AddButton(...)
        -- TODO: Eventually redo this
        local Button = {};
        local function ProcessButtonParams(Class, Obj, ...)
            local Props = select(1, ...)
            if type(Props) == 'table' then
                Obj.Text = Props.Text
                Obj.Func = Props.Func
                Obj.DoubleClick = Props.DoubleClick
                Obj.Tooltip = Props.Tooltip
            else
                Obj.Text = select(1, ...)
                Obj.Func = select(2, ...)
            end

            assert(type(Obj.Func) == 'function', 'AddButton: `Func` callback is missing.');
        end

        ProcessButtonParams('Button', Button, ...)

        local Groupbox = self;
        local Container = Groupbox.Container;

        local function CreateBaseButton(Button)
            local Outer = Library:Create('Frame', {
                BackgroundTransparency = 1;
                BorderSizePixel = 0;
                Size = UDim2.new(1, -4, 0, 20);
                ZIndex = 5;
            });
            Library:Round(Outer, 3);

            local Inner = Library:Create('Frame', {
                BackgroundColor3 = Library.MainColor;
                BorderColor3 = Library.OutlineColor;
                BorderMode = Enum.BorderMode.Inset;
                Size = UDim2.new(1, 0, 1, 0);
                ZIndex = 6;
                Parent = Outer;
            });
            Library:Round(Inner, 3);

            local Label = Library:CreateLabel({
                Size = UDim2.new(1, 0, 1, 0);
                TextSize = 14;
                Text = Button.Text;
                ZIndex = 6;
                Parent = Inner;
            });

            Library:Create('UIGradient', {
                Color = ColorSequence.new({
                    ColorSequenceKeypoint.new(0, Color3.new(1, 1, 1)),
                    ColorSequenceKeypoint.new(1, Color3.new(1, 1, 1))
                });
                Rotation = 90;
                Parent = Inner;
            });

            Library:AddToRegistry(Inner, {
                BackgroundColor3 = 'MainColor';
                BorderColor3 = 'OutlineColor';
            });

            Library:OnHighlight(Outer, Inner,
                { BorderColor3 = 'AccentColor' },
                { BorderColor3 = 'OutlineColor' }
            );

            -- press: flashes toward the accent color and fades back
            Outer.InputBegan:Connect(function(Input)
                if Input.UserInputType == Enum.UserInputType.MouseButton1 then
                    Inner.BackgroundColor3 = Library.MainColor:Lerp(Library.AccentColor, 0.45);
                    Library:Tween(Inner, { BackgroundColor3 = Library.MainColor }, 0.3);
                end;
            end);

            return Outer, Inner, Label
        end

        local function InitEvents(Button)
            local function WaitForEvent(event, timeout, validator)
                local bindable = Instance.new('BindableEvent')
                local connection = event:Once(function(...)

                    if type(validator) == 'function' and validator(...) then
                        bindable:Fire(true)
                    else
                        bindable:Fire(false)
                    end
                end)
                task.delay(timeout, function()
                    connection:disconnect()
                    bindable:Fire(false)
                end)
                return bindable.Event:Wait()
            end

            local function ValidateClick(Input)
                if Library:MouseIsOverOpenedFrame() then
                    return false
                end

                if Input.UserInputType ~= Enum.UserInputType.MouseButton1 then
                    return false
                end

                return true
            end

            Button.Outer.InputBegan:Connect(function(Input)
                if not ValidateClick(Input) then return end
                if Button.Locked then return end

                if Button.DoubleClick then
                    Library:RemoveFromRegistry(Button.Label)
                    Library:AddToRegistry(Button.Label, { TextColor3 = 'AccentColor' })

                    Button.Label.TextColor3 = Library.AccentColor
                    Button.Label.Text = 'Are you sure?'
                    Button.Locked = true

                    local clicked = WaitForEvent(Button.Outer.InputBegan, 0.5, ValidateClick)

                    Library:RemoveFromRegistry(Button.Label)
                    Library:AddToRegistry(Button.Label, { TextColor3 = 'FontColor' })

                    Button.Label.TextColor3 = Library.FontColor
                    Button.Label.Text = Button.Text
                    task.defer(rawset, Button, 'Locked', false)

                    if clicked then
                        Library:SafeCallback(Button.Func)
                    end

                    return
                end

                Library:SafeCallback(Button.Func);
            end)
        end

        Button.Outer, Button.Inner, Button.Label = CreateBaseButton(Button)
        Button.Outer.Parent = Container

        InitEvents(Button)

        function Button:AddTooltip(tooltip)
            if type(tooltip) == 'string' then
                Library:AddToolTip(tooltip, self.Outer)
            end
            return self
        end


        function Button:AddButton(...)
            local SubButton = {}

            ProcessButtonParams('SubButton', SubButton, ...)

            self.Outer.Size = UDim2.new(0.5, -2, 0, 20)

            SubButton.Outer, SubButton.Inner, SubButton.Label = CreateBaseButton(SubButton)

            SubButton.Outer.Position = UDim2.new(1, 3, 0, 0)
            SubButton.Outer.Size = UDim2.fromOffset(self.Outer.AbsoluteSize.X - 2, self.Outer.AbsoluteSize.Y)
            SubButton.Outer.Parent = self.Outer

            function SubButton:AddTooltip(tooltip)
                if type(tooltip) == 'string' then
                    Library:AddToolTip(tooltip, self.Outer)
                end
                return SubButton
            end

            if type(SubButton.Tooltip) == 'string' then
                SubButton:AddTooltip(SubButton.Tooltip)
            end

            InitEvents(SubButton)
            return SubButton
        end

        if type(Button.Tooltip) == 'string' then
            Button:AddTooltip(Button.Tooltip)
        end

        Groupbox:AddBlank(5);
        Groupbox:Resize();

        return Button;
    end;

    function Funcs:AddDivider()
        local Groupbox = self;
        local Container = self.Container

        local Divider = {
            Type = 'Divider',
        }

        Groupbox:AddBlank(2);
        local DividerOuter = Library:Create('Frame', {
            BackgroundColor3 = Color3.new(0, 0, 0);
            BorderColor3 = Color3.new(0, 0, 0);
            Size = UDim2.new(1, -4, 0, 5);
            ZIndex = 5;
            Parent = Container;
        });

        local DividerInner = Library:Create('Frame', {
            BackgroundColor3 = Library.MainColor;
            BorderColor3 = Library.OutlineColor;
            BorderMode = Enum.BorderMode.Inset;
            Size = UDim2.new(1, 0, 1, 0);
            ZIndex = 6;
            Parent = DividerOuter;
        });

        Library:AddToRegistry(DividerOuter, {
            BorderColor3 = 'Black';
        });

        Library:AddToRegistry(DividerInner, {
            BackgroundColor3 = 'MainColor';
            BorderColor3 = 'OutlineColor';
        });

        Groupbox:AddBlank(9);
        Groupbox:Resize();
    end

    function Funcs:AddInput(Idx, Info)
        assert(Info.Text, 'AddInput: Missing `Text` string.')

        local Textbox = {
            Value = Info.Default or '';
            Numeric = Info.Numeric or false;
            Finished = Info.Finished or false;
            Type = 'Input';
            Callback = Info.Callback or function(Value) end;
        };

        local Groupbox = self;
        local Container = Groupbox.Container;

        local InputLabel = Library:CreateLabel({
            Size = UDim2.new(1, 0, 0, 15);
            TextSize = 14;
            Text = Info.Text;
            TextXAlignment = Enum.TextXAlignment.Left;
            ZIndex = 5;
            Parent = Container;
        });

        Groupbox:AddBlank(1);

        local TextBoxOuter = Library:Create('Frame', {
            BackgroundColor3 = Color3.new(0, 0, 0);
            BorderColor3 = Color3.new(0, 0, 0);
            Size = UDim2.new(1, -4, 0, 20);
            ZIndex = 5;
            Parent = Container;
        });
        Library:Round(TextBoxOuter, 3);

        local TextBoxInner = Library:Create('Frame', {
            BackgroundColor3 = Library.MainColor;
            BorderColor3 = Library.OutlineColor;
            BorderMode = Enum.BorderMode.Inset;
            Size = UDim2.new(1, 0, 1, 0);
            ZIndex = 6;
            Parent = TextBoxOuter;
        });
        Library:Round(TextBoxInner, 3);

        Library:AddToRegistry(TextBoxInner, {
            BackgroundColor3 = 'MainColor';
            BorderColor3 = 'OutlineColor';
        });

        Library:OnHighlight(TextBoxOuter, TextBoxOuter,
            { BorderColor3 = 'AccentColor' },
            { BorderColor3 = 'Black' }
        );

        if type(Info.Tooltip) == 'string' then
            Library:AddToolTip(Info.Tooltip, TextBoxOuter)
        end

        Library:Create('UIGradient', {
            Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0, Color3.new(1, 1, 1)),
                ColorSequenceKeypoint.new(1, Color3.new(1, 1, 1))
            });
            Rotation = 90;
            Parent = TextBoxInner;
        });

        local Container = Library:Create('Frame', {
            BackgroundTransparency = 1;
            ClipsDescendants = true;

            Position = UDim2.new(0, 5, 0, 0);
            Size = UDim2.new(1, -5, 1, 0);

            ZIndex = 7;
            Parent = TextBoxInner;
        })

        local Box = Library:Create('TextBox', {
            BackgroundTransparency = 1;

            Position = UDim2.fromOffset(0, 0),
            Size = UDim2.fromScale(5, 1),

            Font = Library.Font;
            PlaceholderColor3 = Color3.fromRGB(190, 190, 190);
            PlaceholderText = Info.Placeholder or '';

            Text = Info.Default or '';
            TextColor3 = Library.FontColor;
            TextSize = 14;
            TextStrokeTransparency = 0;
            TextXAlignment = Enum.TextXAlignment.Left;

            ZIndex = 7;
            Parent = Container;
        });

        Library:ApplyTextStroke(Box);

        function Textbox:SetValue(Text)
            if Info.MaxLength and #Text > Info.MaxLength then
                Text = Text:sub(1, Info.MaxLength);
            end;

            if Textbox.Numeric then
                if (not tonumber(Text)) and Text:len() > 0 then
                    Text = Textbox.Value
                end
            end

            Textbox.Value = Text;
            Box.Text = Text;

            Library:SafeCallback(Textbox.Callback, Textbox.Value);
            Library:SafeCallback(Textbox.Changed, Textbox.Value);
        end;

        if Textbox.Finished then
            Box.FocusLost:Connect(function(enter)
                if not enter then return end

                Textbox:SetValue(Box.Text);
                Library:AttemptSave();
            end)
        else
            Box:GetPropertyChangedSignal('Text'):Connect(function()
                Textbox:SetValue(Box.Text);
                Library:AttemptSave();
            end);
        end

        -- https://devforum.roblox.com/t/how-to-make-textboxes-follow-current-cursor-position/1368429/6
        -- thank you nicemike40 :)

        local function Update()
            local PADDING = 2
            local reveal = Container.AbsoluteSize.X

            if not Box:IsFocused() or Box.TextBounds.X <= reveal - 2 * PADDING then
                -- we aren't focused, or we fit so be normal
                Box.Position = UDim2.new(0, PADDING, 0, 0)
            else
                -- we are focused and don't fit, so adjust position
                local cursor = Box.CursorPosition
                if cursor ~= -1 then
                    -- calculate pixel width of text from start to cursor
                    local subtext = string.sub(Box.Text, 1, cursor-1)
                    local width = TextService:GetTextSize(subtext, Box.TextSize, Box.Font, Vector2.new(math.huge, math.huge)).X

                    -- check if we're inside the box with the cursor
                    local currentCursorPos = Box.Position.X.Offset + width

                    -- adjust if necessary
                    if currentCursorPos < PADDING then
                        Box.Position = UDim2.fromOffset(PADDING-width, 0)
                    elseif currentCursorPos > reveal - PADDING - 1 then
                        Box.Position = UDim2.fromOffset(reveal-width-PADDING-1, 0)
                    end
                end
            end
        end

        task.spawn(Update)

        Box:GetPropertyChangedSignal('Text'):Connect(Update)
        Box:GetPropertyChangedSignal('CursorPosition'):Connect(Update)
        Box.FocusLost:Connect(Update)
        Box.Focused:Connect(Update)

        Library:AddToRegistry(Box, {
            TextColor3 = 'FontColor';
        });

        function Textbox:OnChanged(Func)
            Textbox.Changed = Func;
            Func(Textbox.Value);
        end;

        Groupbox:AddBlank(5);
        Groupbox:Resize();

        Options[Idx] = Textbox;

        return Textbox;
    end;

    function Funcs:AddToggle(Idx, Info)
        assert(Info.Text, 'AddInput: Missing `Text` string.')

        local Toggle = {
            Value = Info.Default or false;
            Type = 'Toggle';

            Callback = Info.Callback or function(Value) end;
            Addons = {},
            Risky = Info.Risky,
        };

        local Groupbox = self;
        local Container = Groupbox.Container;

        local ToggleOuter = Library:Create('Frame', {
            BackgroundTransparency = 1;
            BorderSizePixel = 0;
            Size = UDim2.new(0, 13, 0, 13);
            ZIndex = 5;
            Parent = Container;
        });
        Library:Round(ToggleOuter, 3);

        local ToggleInner = Library:Create('Frame', {
            BackgroundColor3 = Library.MainColor;
            BorderColor3 = Library.OutlineColor;
            BorderMode = Enum.BorderMode.Inset;
            Size = UDim2.new(1, 0, 1, 0);
            ZIndex = 6;
            Parent = ToggleOuter;
        });
        Library:Round(ToggleInner, 3);

        Library:AddToRegistry(ToggleInner, {
            BackgroundColor3 = 'MainColor';
            BorderColor3 = 'OutlineColor';
        });

        local ToggleLabel = Library:CreateLabel({
            Size = UDim2.new(0, 216, 1, 0);
            Position = UDim2.new(1, 6, 0, 0);
            TextSize = 14;
            Text = Info.Text;
            TextXAlignment = Enum.TextXAlignment.Left;
            ZIndex = 6;
            Parent = ToggleInner;
        });

        Library:Create('UIListLayout', {
            Padding = UDim.new(0, 4);
            FillDirection = Enum.FillDirection.Horizontal;
            HorizontalAlignment = Enum.HorizontalAlignment.Right;
            SortOrder = Enum.SortOrder.LayoutOrder;
            Parent = ToggleLabel;
        });

        local ToggleRegion = Library:Create('Frame', {
            BackgroundTransparency = 1;
            Size = UDim2.new(0, 170, 1, 0);
            ZIndex = 8;
            Parent = ToggleOuter;
        });

        Library:OnHighlight(ToggleRegion, ToggleInner,
            { BorderColor3 = 'AccentColor' },
            { BorderColor3 = 'OutlineColor' }
        );

        function Toggle:UpdateColors()
            Toggle:Display();
        end;

        if type(Info.Tooltip) == 'string' then
            Library:AddToolTip(Info.Tooltip, ToggleRegion)
        end

        function Toggle:Display()
            Library:Tween(ToggleInner, {
                BackgroundColor3 = Toggle.Value and Library.AccentColor or Library.MainColor;
                BorderColor3 = Toggle.Value and Library.AccentColorDark or Library.OutlineColor;
            }, 0.15);

            Library.RegistryMap[ToggleInner].Properties.BackgroundColor3 = Toggle.Value and 'AccentColor' or 'MainColor';
            Library.RegistryMap[ToggleInner].Properties.BorderColor3 = Toggle.Value and 'AccentColorDark' or 'OutlineColor';
        end;

        function Toggle:OnChanged(Func)
            Toggle.Changed = Func;
            Func(Toggle.Value);
        end;

        function Toggle:SetValue(Bool)
            Bool = (not not Bool);

            Toggle.Value = Bool;
            Toggle:Display();

            for _, Addon in next, Toggle.Addons do
                if Addon.Type == 'KeyPicker' and Addon.SyncToggleState then
                    Addon.Toggled = Bool
                    Addon:Update()
                end
            end

            Library:SafeCallback(Toggle.Callback, Toggle.Value);
            Library:SafeCallback(Toggle.Changed, Toggle.Value);
            Library:UpdateDependencyBoxes();
        end;

        ToggleRegion.InputBegan:Connect(function(Input)
            if Input.UserInputType == Enum.UserInputType.MouseButton1 and not Library:MouseIsOverOpenedFrame() then
                Toggle:SetValue(not Toggle.Value) -- Why was it not like this from the start?
                Library:AttemptSave();
            end;
        end);

        if Toggle.Risky then
            Library:RemoveFromRegistry(ToggleLabel)
            ToggleLabel.TextColor3 = Library.RiskColor
            Library:AddToRegistry(ToggleLabel, { TextColor3 = 'RiskColor' })
        end

        Toggle:Display();
        Groupbox:AddBlank(Info.BlankSize or 5 + 2);
        Groupbox:Resize();

        Toggle.TextLabel = ToggleLabel;
        Toggle.Container = Container;
        Toggle.Groupbox = Groupbox;
        Toggle.Region = ToggleRegion;
        setmetatable(Toggle, BaseAddons);

        Toggles[Idx] = Toggle;

        Library:AttachBind(Toggle, ToggleRegion, Info, Idx, Groupbox);

        Library:UpdateDependencyBoxes();

        return Toggle;
    end;

    function Funcs:AddSlider(Idx, Info)
        assert(Info.Default, 'AddSlider: Missing default value.');
        assert(Info.Text, 'AddSlider: Missing slider text.');
        assert(Info.Min, 'AddSlider: Missing minimum value.');
        assert(Info.Max, 'AddSlider: Missing maximum value.');
        assert(Info.Rounding, 'AddSlider: Missing rounding value.');

        local Slider = {
            Value = Info.Default;
            Min = Info.Min;
            Max = Info.Max;
            Rounding = Info.Rounding;
            MaxSize = 232;
            Type = 'Slider';
            Callback = Info.Callback or function(Value) end;
        };

        local Groupbox = self;
        local Container = Groupbox.Container;

        if not Info.Compact then
            Library:CreateLabel({
                Size = UDim2.new(1, 0, 0, 10);
                TextSize = 14;
                Text = Info.Text;
                TextXAlignment = Enum.TextXAlignment.Left;
                TextYAlignment = Enum.TextYAlignment.Bottom;
                ZIndex = 5;
                Parent = Container;
            });

            Groupbox:AddBlank(3);
        end

        local SliderOuter = Library:Create('Frame', {
            BackgroundTransparency = 1;
            BorderSizePixel = 0;
            Size = UDim2.new(1, -4, 0, 13);
            ZIndex = 5;
            Parent = Container;
        });
        Library:Round(SliderOuter, 3);

        local SliderInner = Library:Create('Frame', {
            BackgroundColor3 = Library.MainColor;
            BorderColor3 = Library.OutlineColor;
            BorderMode = Enum.BorderMode.Inset;
            Size = UDim2.new(1, 0, 1, 0);
            ZIndex = 6;
            Parent = SliderOuter;
        });
        Library:Round(SliderInner, 3);

        Library:AddToRegistry(SliderInner, {
            BackgroundColor3 = 'MainColor';
            BorderColor3 = 'OutlineColor';
        });

        -- the fill is sized by scale so it always matches the real width of the track
        local Fill = Library:Create('Frame', {
            BackgroundColor3 = Library.AccentColor;
            BorderSizePixel = 0;
            Size = UDim2.new(0, 0, 1, 0);
            ZIndex = 7;
            Parent = SliderInner;
        });
        Library:Round(Fill, 3);

        Library:AddToRegistry(Fill, {
            BackgroundColor3 = 'AccentColor';
        });

        local DisplayLabel = Library:CreateLabel({
            Size = UDim2.new(1, 0, 1, 0);
            TextSize = 12;
            Text = 'Infinite';
            ZIndex = 9;
            Parent = SliderInner;
        });

        -- dark outline keeps the value readable on top of the fill
        Library:Create('UIStroke', {
            Color = Color3.new(0, 0, 0);
            LineJoinMode = Enum.LineJoinMode.Miter;
            Thickness = 1;
            Parent = DisplayLabel;
        });

        Library:OnHighlight(SliderOuter, SliderInner,
            { BorderColor3 = 'AccentColor' },
            { BorderColor3 = 'OutlineColor' }
        );

        if type(Info.Tooltip) == 'string' then
            Library:AddToolTip(Info.Tooltip, SliderOuter)
        end

        function Slider:UpdateColors()
            Fill.BackgroundColor3 = Library.AccentColor;
        end;

        function Slider:Display()
            local Suffix = Info.Suffix or '';

            if Info.Compact then
                DisplayLabel.Text = Info.Text .. ': ' .. Slider.Value .. Suffix
            elseif Info.HideMax then
                DisplayLabel.Text = string.format('%s', Slider.Value .. Suffix)
            else
                DisplayLabel.Text = string.format('%s/%s', Slider.Value .. Suffix, Slider.Max .. Suffix);
            end

            local Range = Slider.Max - Slider.Min;
            local Alpha = Range == 0 and 0 or math.clamp((Slider.Value - Slider.Min) / Range, 0, 1);
            Library:Tween(Fill, { Size = UDim2.new(Alpha, 0, 1, 0) }, 0.1);
        end;

        function Slider:OnChanged(Func)
            Slider.Changed = Func;
            Func(Slider.Value);
        end;

        local function Round(Value)
            if Slider.Rounding == 0 then
                return math.floor(Value);
            end;


            return tonumber(string.format('%.' .. Slider.Rounding .. 'f', Value))
        end;

        function Slider:GetValueFromXOffset(X)
            local Width = math.max(SliderInner.AbsoluteSize.X, 1);
            return Round(Library:MapValue(X, 0, Width, Slider.Min, Slider.Max));
        end;

        function Slider:SetValue(Str)
            local Num = tonumber(Str);

            if (not Num) then
                return;
            end;

            Num = math.clamp(Num, Slider.Min, Slider.Max);

            Slider.Value = Num;
            Slider:Display();

            Library:SafeCallback(Slider.Callback, Slider.Value);
            Library:SafeCallback(Slider.Changed, Slider.Value);
        end;

        SliderInner.InputBegan:Connect(function(Input)
            if Input.UserInputType == Enum.UserInputType.MouseButton1 and not Library:MouseIsOverOpenedFrame() then
                while InputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) do
                    local Width = math.max(SliderInner.AbsoluteSize.X, 1);
                    local nX = math.clamp(Mouse.X - SliderInner.AbsolutePosition.X, 0, Width);

                    local nValue = Slider:GetValueFromXOffset(nX);
                    local OldValue = Slider.Value;
                    Slider.Value = nValue;

                    Slider:Display();

                    if nValue ~= OldValue then
                        Library:SafeCallback(Slider.Callback, Slider.Value);
                        Library:SafeCallback(Slider.Changed, Slider.Value);
                    end;

                    RenderStepped:Wait();
                end;

                Library:AttemptSave();
            end;
        end);

        Slider:Display();
        Groupbox:AddBlank(Info.BlankSize or 6);
        Groupbox:Resize();

        Options[Idx] = Slider;

        Library:AttachBind(Slider, SliderInner, Info, Idx, Groupbox);

        return Slider;
    end;

    function Funcs:AddDropdown(Idx, Info)
        if Info.SpecialType == 'Player' then
            Info.Values = GetPlayersString();
            Info.AllowNull = true;
        elseif Info.SpecialType == 'Team' then
            Info.Values = GetTeamsString();
            Info.AllowNull = true;
        end;

        assert(Info.Values, 'AddDropdown: Missing dropdown value list.');
        assert(Info.AllowNull or Info.Default, 'AddDropdown: Missing default value. Pass `AllowNull` as true if this was intentional.')

        if (not Info.Text) then
            Info.Compact = true;
        end;

        local Dropdown = {
            Values = Info.Values;
            Value = Info.Multi and {};
            Multi = Info.Multi;
            Type = 'Dropdown';
            SpecialType = Info.SpecialType; -- can be either 'Player' or 'Team'
            Callback = Info.Callback or function(Value) end;
        };

        local Groupbox = self;
        local Container = Groupbox.Container;

        local RelativeOffset = 0;

        if not Info.Compact then
            local DropdownLabel = Library:CreateLabel({
                Size = UDim2.new(1, 0, 0, 10);
                TextSize = 14;
                Text = Info.Text;
                TextXAlignment = Enum.TextXAlignment.Left;
                TextYAlignment = Enum.TextYAlignment.Bottom;
                ZIndex = 5;
                Parent = Container;
            });

            Groupbox:AddBlank(3);
        end

        for _, Element in next, Container:GetChildren() do
            if not Element:IsA('UIListLayout') then
                RelativeOffset = RelativeOffset + Element.Size.Y.Offset;
            end;
        end;

        local DropdownOuter = Library:Create('Frame', {
            BackgroundTransparency = 1;
            BorderSizePixel = 0;
            Size = UDim2.new(1, -4, 0, 20);
            ZIndex = 5;
            Parent = Container;
        });
        Library:Round(DropdownOuter, 3);

        local DropdownInner = Library:Create('Frame', {
            BackgroundColor3 = Library.MainColor;
            BorderColor3 = Library.OutlineColor;
            BorderMode = Enum.BorderMode.Inset;
            Size = UDim2.new(1, 0, 1, 0);
            ZIndex = 6;
            Parent = DropdownOuter;
        });
        Library:Round(DropdownInner, 3);

        Library:AddToRegistry(DropdownInner, {
            BackgroundColor3 = 'MainColor';
            BorderColor3 = 'OutlineColor';
        });

        Library:Create('UIGradient', {
            Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0, Color3.new(1, 1, 1)),
                ColorSequenceKeypoint.new(1, Color3.new(1, 1, 1))
            });
            Rotation = 90;
            Parent = DropdownInner;
        });

        local DropdownArrow = Library:Create('ImageLabel', {
            AnchorPoint = Vector2.new(0, 0.5);
            BackgroundTransparency = 1;
            Position = UDim2.new(1, -16, 0.5, 0);
            Size = UDim2.new(0, 12, 0, 12);
            Image = 'http://www.roblox.com/asset/?id=6282522798';
            ZIndex = 8;
            Parent = DropdownInner;
        });

        local ItemList = Library:CreateLabel({
            Position = UDim2.new(0, 5, 0, 0);
            Size = UDim2.new(1, -5, 1, 0);
            TextSize = 14;
            Text = '--';
            TextXAlignment = Enum.TextXAlignment.Left;
            TextWrapped = true;
            ZIndex = 7;
            Parent = DropdownInner;
        });

        Library:OnHighlight(DropdownOuter, DropdownInner,
            { BorderColor3 = 'AccentColor' },
            { BorderColor3 = 'OutlineColor' }
        );

        if type(Info.Tooltip) == 'string' then
            Library:AddToolTip(Info.Tooltip, DropdownOuter)
        end

        local MAX_DROPDOWN_ITEMS = 8;

        local ListOuter = Library:Create('Frame', {
            BackgroundColor3 = Library.MainColor;
            BorderColor3 = Library.OutlineColor;
            ZIndex = 20;
            ClipsDescendants = true;
            Visible = false;
            Parent = Container:FindFirstAncestorOfClass('ScreenGui') or ScreenGui;
        });
        Library:Round(ListOuter, 4);

        Library:AddToRegistry(ListOuter, {
            BackgroundColor3 = 'MainColor';
            BorderColor3 = 'OutlineColor';
        });

        local function RecalculateListPosition()
            ListOuter.Position = UDim2.fromOffset(DropdownOuter.AbsolutePosition.X, DropdownOuter.AbsolutePosition.Y + DropdownOuter.Size.Y.Offset + 4);
        end;

        local function RecalculateListSize(YSize)
            ListOuter.Size = UDim2.fromOffset(DropdownOuter.AbsoluteSize.X, YSize or (MAX_DROPDOWN_ITEMS * 20 + 2))
        end;

        RecalculateListPosition();
        RecalculateListSize();

        DropdownOuter:GetPropertyChangedSignal('AbsolutePosition'):Connect(RecalculateListPosition);

        local ListInner = Library:Create('Frame', {
            BackgroundTransparency = 1;
            BorderSizePixel = 0;
            Size = UDim2.new(1, 0, 1, 0);
            ZIndex = 21;
            Parent = ListOuter;
        });

        local Scrolling = Library:Create('ScrollingFrame', {
            BackgroundTransparency = 1;
            BorderSizePixel = 0;
            CanvasSize = UDim2.new(0, 0, 0, 0);
            Size = UDim2.new(1, 0, 1, 0);
            ZIndex = 21;
            Parent = ListInner;

            TopImage = 'rbxasset://textures/ui/Scroll/scroll-middle.png',
            BottomImage = 'rbxasset://textures/ui/Scroll/scroll-middle.png',

            ScrollBarThickness = 3,
            ScrollBarImageColor3 = Library.AccentColor,
        });

        Library:AddToRegistry(Scrolling, {
            ScrollBarImageColor3 = 'AccentColor'
        })

        Library:Create('UIPadding', {
            PaddingTop = UDim.new(0, 2);
            PaddingBottom = UDim.new(0, 2);
            PaddingLeft = UDim.new(0, 2);
            PaddingRight = UDim.new(0, 2);
            Parent = Scrolling;
        });

        Library:Create('UIListLayout', {
            Padding = UDim.new(0, 0);
            FillDirection = Enum.FillDirection.Vertical;
            SortOrder = Enum.SortOrder.LayoutOrder;
            Parent = Scrolling;
        });

        function Dropdown:Display()
            local Values = Dropdown.Values;
            local Str = '';

            if Info.Multi then
                for Idx, Value in next, Values do
                    if Dropdown.Value[Value] then
                        Str = Str .. Value .. ', ';
                    end;
                end;

                Str = Str:sub(1, #Str - 2);
            else
                Str = Dropdown.Value or '';
            end;

            ItemList.Text = (Str == '' and '--' or Str);
        end;

        function Dropdown:GetActiveValues()
            if Info.Multi then
                local T = {};

                for Value, Bool in next, Dropdown.Value do
                    table.insert(T, Value);
                end;

                return T;
            else
                return Dropdown.Value and 1 or 0;
            end;
        end;

        function Dropdown:BuildDropdownList()
            local Values = Dropdown.Values;
            local Buttons = {};

            for _, Element in next, Scrolling:GetChildren() do
                if not Element:IsA('UIListLayout') then
                    Element:Destroy();
                end;
            end;

            local Count = 0;

            for Idx, Value in next, Values do
                local Table = {};

                Count = Count + 1;

                -- flat rows: no border, a soft highlight on hover
                local Button = Library:Create('Frame', {
                    BackgroundColor3 = Library.OutlineColor;
                    BackgroundTransparency = 1;
                    BorderSizePixel = 0;
                    Size = UDim2.new(1, 0, 0, 20);
                    ZIndex = 23;
                    Active = true,
                    Parent = Scrolling;
                });

                Library:Create('UICorner', {
                    CornerRadius = UDim.new(0, 3);
                    Parent = Button;
                });

                Library:AddToRegistry(Button, {
                    BackgroundColor3 = 'OutlineColor';
                });

                local ButtonLabel = Library:CreateLabel({
                    Active = false;
                    Size = UDim2.new(1, -6, 1, 0);
                    Position = UDim2.new(0, 6, 0, 0);
                    TextSize = 14;
                    Text = Value;
                    TextXAlignment = Enum.TextXAlignment.Left;
                    ZIndex = 25;
                    Parent = Button;
                });

                Button.MouseEnter:Connect(function()
                    TweenService:Create(Button, TweenInfo.new(0.1), { BackgroundTransparency = 0.4 }):Play();
                end);

                Button.MouseLeave:Connect(function()
                    TweenService:Create(Button, TweenInfo.new(0.1), { BackgroundTransparency = 1 }):Play();
                end);

                local Selected;

                if Info.Multi then
                    Selected = Dropdown.Value[Value];
                else
                    Selected = Dropdown.Value == Value;
                end;

                function Table:UpdateButton()
                    if Info.Multi then
                        Selected = Dropdown.Value[Value];
                    else
                        Selected = Dropdown.Value == Value;
                    end;

                    ButtonLabel.TextColor3 = Selected and Library.AccentColor or Library.FontColor;
                    Library.RegistryMap[ButtonLabel].Properties.TextColor3 = Selected and 'AccentColor' or 'FontColor';
                end;

                ButtonLabel.InputBegan:Connect(function(Input)
                    if Input.UserInputType == Enum.UserInputType.MouseButton1 then
                        local Try = not Selected;

                        if Dropdown:GetActiveValues() == 1 and (not Try) and (not Info.AllowNull) then
                        else
                            if Info.Multi then
                                Selected = Try;

                                if Selected then
                                    Dropdown.Value[Value] = true;
                                else
                                    Dropdown.Value[Value] = nil;
                                end;
                            else
                                Selected = Try;

                                if Selected then
                                    Dropdown.Value = Value;
                                else
                                    Dropdown.Value = nil;
                                end;

                                for _, OtherButton in next, Buttons do
                                    OtherButton:UpdateButton();
                                end;
                            end;

                            Table:UpdateButton();
                            Dropdown:Display();

                            Library:SafeCallback(Dropdown.Callback, Dropdown.Value);
                            Library:SafeCallback(Dropdown.Changed, Dropdown.Value);

                            Library:AttemptSave();
                        end;
                    end;
                end);

                Table:UpdateButton();
                Dropdown:Display();

                Buttons[Button] = Table;
            end;

            Scrolling.CanvasSize = UDim2.fromOffset(0, (Count * 20) + 4);

            local Y = math.clamp(Count * 20, 0, MAX_DROPDOWN_ITEMS * 20) + 4;
            RecalculateListSize(Y);
        end;

        function Dropdown:SetValues(NewValues)
            if NewValues then
                Dropdown.Values = NewValues;
            end;

            Dropdown:BuildDropdownList();
        end;

        function Dropdown:OpenDropdown()
            -- list unrolls from the top
            local Target = ListOuter.Size;

            ListOuter.Size = UDim2.new(Target.X.Scale, Target.X.Offset, 0, 0);
            ListOuter.Visible = true;
            Library.OpenedFrames[ListOuter] = true;

            Library:Tween(ListOuter, { Size = Target }, 0.15);
            Library:Tween(DropdownArrow, { Rotation = 180 }, 0.15);
        end;

        function Dropdown:CloseDropdown()
            ListOuter.Visible = false;
            Library.OpenedFrames[ListOuter] = nil;

            Library:Tween(DropdownArrow, { Rotation = 0 }, 0.15);
        end;

        function Dropdown:OnChanged(Func)
            Dropdown.Changed = Func;
            Func(Dropdown.Value);
        end;

        function Dropdown:SetValue(Val)
            if Dropdown.Multi then
                local nTable = {};

                for Value, Bool in next, Val do
                    if table.find(Dropdown.Values, Value) then
                        nTable[Value] = true
                    end;
                end;

                Dropdown.Value = nTable;
            else
                if (not Val) then
                    Dropdown.Value = nil;
                elseif table.find(Dropdown.Values, Val) then
                    Dropdown.Value = Val;
                end;
            end;

            Dropdown:BuildDropdownList();

            Library:SafeCallback(Dropdown.Callback, Dropdown.Value);
            Library:SafeCallback(Dropdown.Changed, Dropdown.Value);
        end;

        DropdownOuter.InputBegan:Connect(function(Input)
            if Input.UserInputType == Enum.UserInputType.MouseButton1 and not Library:MouseIsOverOpenedFrame() then
                if ListOuter.Visible then
                    Dropdown:CloseDropdown();
                else
                    Dropdown:OpenDropdown();
                end;
            end;
        end);

        InputService.InputBegan:Connect(function(Input)
            if Input.UserInputType == Enum.UserInputType.MouseButton1 then
                local AbsPos, AbsSize = ListOuter.AbsolutePosition, ListOuter.AbsoluteSize;

                if Mouse.X < AbsPos.X or Mouse.X > AbsPos.X + AbsSize.X
                    or Mouse.Y < (AbsPos.Y - 20 - 1) or Mouse.Y > AbsPos.Y + AbsSize.Y then

                    Dropdown:CloseDropdown();
                end;
            end;
        end);

        Dropdown:BuildDropdownList();
        Dropdown:Display();

        local Defaults = {}

        if type(Info.Default) == 'string' then
            local Idx = table.find(Dropdown.Values, Info.Default)
            if Idx then
                table.insert(Defaults, Idx)
            end
        elseif type(Info.Default) == 'table' then
            for _, Value in next, Info.Default do
                local Idx = table.find(Dropdown.Values, Value)
                if Idx then
                    table.insert(Defaults, Idx)
                end
            end
        elseif type(Info.Default) == 'number' and Dropdown.Values[Info.Default] ~= nil then
            table.insert(Defaults, Info.Default)
        end

        if next(Defaults) then
            for i = 1, #Defaults do
                local Index = Defaults[i]
                if Info.Multi then
                    Dropdown.Value[Dropdown.Values[Index]] = true
                else
                    Dropdown.Value = Dropdown.Values[Index];
                end

                if (not Info.Multi) then break end
            end

            Dropdown:BuildDropdownList();
            Dropdown:Display();
        end

        Groupbox:AddBlank(Info.BlankSize or 5);
        Groupbox:Resize();

        Options[Idx] = Dropdown;

        Library:AttachBind(Dropdown, DropdownOuter, Info, Idx, Groupbox);

        return Dropdown;
    end;

    function Funcs:AddDependencyBox()
        local Depbox = {
            Dependencies = {};
        };
        
        local Groupbox = self;
        local Container = Groupbox.Container;

        local Holder = Library:Create('Frame', {
            BackgroundTransparency = 1;
            Size = UDim2.new(1, 0, 0, 0);
            Visible = false;
            Parent = Container;
        });

        local Frame = Library:Create('Frame', {
            BackgroundTransparency = 1;
            Size = UDim2.new(1, 0, 1, 0);
            Visible = true;
            Parent = Holder;
        });

        local Layout = Library:Create('UIListLayout', {
            FillDirection = Enum.FillDirection.Vertical;
            SortOrder = Enum.SortOrder.LayoutOrder;
            Parent = Frame;
        });

        function Depbox:Resize()
            Holder.Size = UDim2.new(1, 0, 0, Layout.AbsoluteContentSize.Y);
            Groupbox:Resize();
        end;

        Layout:GetPropertyChangedSignal('AbsoluteContentSize'):Connect(function()
            Depbox:Resize();
        end);

        Holder:GetPropertyChangedSignal('Visible'):Connect(function()
            Depbox:Resize();
        end);

        function Depbox:Update()
            for _, Dependency in next, Depbox.Dependencies do
                local Elem = Dependency[1];
                local Value = Dependency[2];

                if Elem.Type == 'Toggle' and Elem.Value ~= Value then
                    Holder.Visible = false;
                    Depbox:Resize();
                    return;
                end;
            end;

            Holder.Visible = true;
            Depbox:Resize();
        end;

        function Depbox:SetupDependencies(Dependencies)
            for _, Dependency in next, Dependencies do
                assert(type(Dependency) == 'table', 'SetupDependencies: Dependency is not of type `table`.');
                assert(Dependency[1], 'SetupDependencies: Dependency is missing element argument.');
                assert(Dependency[2] ~= nil, 'SetupDependencies: Dependency is missing value argument.');
            end;

            Depbox.Dependencies = Dependencies;
            Depbox:Update();
        end;

        Depbox.Container = Frame;

        setmetatable(Depbox, BaseGroupbox);

        table.insert(Library.DependencyBoxes, Depbox);

        return Depbox;
    end;

    BaseGroupbox.__index = Funcs;
    BaseGroupbox.__namecall = function(Table, Key, ...)
        return Funcs[Key](...);
    end;
end;

-- Popup windows ---------------------------------------------------------
-- Floating window that fades / slides in. Its object takes AddSlider / AddDropdown / etc.
-- Anchors are frames that toggle the popup themselves, so clicks on them with `AnchorInput`
-- are not treated as "click outside".
local function MakePopup(Width, GetPosition, ParentPopup, Anchors, AnchorInput)
    local Popup = {
        Open = false;
        IsPopup = true;
        ParentPopup = ParentPopup;
        Children = {};
    };

    if ParentPopup then
        table.insert(ParentPopup.Children, Popup);
    end;

    local Root = Library:Create('CanvasGroup', {
        BackgroundColor3 = Library.BackgroundColor;
        BorderColor3 = Library.OutlineColor;
        GroupTransparency = 1;
        Size = UDim2.new(0, Width, 0, 20);
        Visible = false;
        Active = true;
        ZIndex = 1;
        Parent = PopupGui;
    });
    Library:Round(Root, 5);

    Library:AddToRegistry(Root, {
        BackgroundColor3 = 'BackgroundColor';
        BorderColor3 = 'OutlineColor';
    });

    local Host = Library:Create('Frame', {
        BackgroundTransparency = 1;
        Position = UDim2.new(0, 6, 0, 6);
        Size = UDim2.new(1, -12, 1, -12);
        ZIndex = 1;
        Parent = Root;
    });

    local Layout = Library:Create('UIListLayout', {
        FillDirection = Enum.FillDirection.Vertical;
        SortOrder = Enum.SortOrder.LayoutOrder;
        Parent = Host;
    });

    function Popup:Resize()
        Root.Size = UDim2.new(0, Width, 0, Layout.AbsoluteContentSize.Y + 12);
    end;

    Layout:GetPropertyChangedSignal('AbsoluteContentSize'):Connect(function()
        Popup:Resize();
    end);

    local OpenInfo = TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out);
    local CloseInfo = TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.In);

    local BasePosition;
    local CurrentTween;

    function Popup:SetOpen(Bool)
        Bool = not not Bool;

        if Bool == Popup.Open then
            return;
        end;

        -- only one window at a time (the one it was opened from stays)
        if Bool then
            Library:ClosePopups(Popup);
        end;

        Popup.Open = Bool;

        if CurrentTween then
            CurrentTween:Cancel();
        end;

        if Bool then
            BasePosition = GetPosition();

            -- newest window always on top: re-parenting moves it to the end of the sibling order.
            -- (the root's own ZIndex must stay low, otherwise its background would cover its elements)
            Root.Parent = nil;
            Root.Parent = PopupGui;

            Root.Position = BasePosition + UDim2.fromOffset(0, 8);
            Root.GroupTransparency = 1;
            Root.Visible = true;

            Popup:Resize();

            CurrentTween = TweenService:Create(Root, OpenInfo, { GroupTransparency = 0; Position = BasePosition });
            CurrentTween:Play();
        else
            for _, Child in next, Popup.Children do
                Child:SetOpen(false);
            end;

            local Tween = TweenService:Create(Root, CloseInfo, { GroupTransparency = 1; Position = BasePosition + UDim2.fromOffset(0, 6) });
            CurrentTween = Tween;

            Tween.Completed:Connect(function(State)
                if State == Enum.PlaybackState.Completed and not Popup.Open then
                    Root.Visible = false;
                end;
            end);

            Tween:Play();
        end;
    end;

    -- click anywhere else to close
    Library:GiveSignal(InputService.InputBegan:Connect(function(Input)
        if not Popup.Open then
            return;
        end;

        if Input.UserInputType ~= Enum.UserInputType.MouseButton1 and Input.UserInputType ~= Enum.UserInputType.MouseButton2 then
            return;
        end;

        if Library:IsMouseOverFrame(Root) or Library:MouseIsOverOpenedFrame() then
            return;
        end;

        if Input.UserInputType == AnchorInput then
            for _, Anchor in next, Anchors do
                if Library:IsMouseOverFrame(Anchor) then
                    return;
                end;
            end;
        end;

        for _, Child in next, Popup.Children do
            if Child.Open and Library:IsMouseOverFrame(Child.Root) then
                return;
            end;
        end;

        Popup:SetOpen(false);
    end));

    table.insert(Library.Popups, Popup);

    Popup.Root = Root;
    Popup.Container = Host;
    setmetatable(Popup, BaseGroupbox);

    return Popup;
end;

-- Toggle:AddSettings()
-- Adds a small accent colored circle with a plus right next to the toggle text. Clicking it
-- opens a floating window; the returned object takes AddSlider / AddDropdown / etc.
BaseAddons.__index.AddSettings = function(self)
    assert(self.Groupbox and self.Region, 'AddSettings: can only be used on a toggle.');

    local Label = self.TextLabel;
    local Parent = self.Region.Parent;

    -- plus is black on bright accents and white on dark ones
    local function PlusColor()
        local Accent = Library.AccentColor;
        local Luminance = 0.299 * Accent.R + 0.587 * Accent.G + 0.114 * Accent.B;
        return Luminance > 0.6 and Color3.new(0, 0, 0) or Color3.new(1, 1, 1);
    end;

    local Circle = Library:Create('Frame', {
        AnchorPoint = Vector2.new(0.5, 0.5);
        BackgroundColor3 = Library.AccentColor;
        BorderSizePixel = 0;
        Size = UDim2.new(0, 13, 0, 13);
        ZIndex = 9;
        Parent = Parent;
    });

    Library:Create('UICorner', {
        CornerRadius = UDim.new(1, 0);
        Parent = Circle;
    });

    Library:AddToRegistry(Circle, {
        BackgroundColor3 = 'AccentColor';
    });

    local function PlusBar(Size)
        local Bar = Library:Create('Frame', {
            AnchorPoint = Vector2.new(0.5, 0.5);
            BackgroundColor3 = PlusColor();
            BorderSizePixel = 0;
            Position = UDim2.new(0.5, 0, 0.5, 0);
            Size = Size;
            ZIndex = 10;
            Parent = Circle;
        });

        Library:AddToRegistry(Bar, {
            BackgroundColor3 = PlusColor;
        });
    end;

    PlusBar(UDim2.new(0, 7, 0, 1));
    PlusBar(UDim2.new(0, 1, 0, 7));

    -- sits right after the toggle text; the click region stops before it so the two never overlap
    local function Reposition()
        local TextWidth = Label.TextBounds.X;

        Circle.Position = UDim2.new(0, 19 + TextWidth + 8 + 6.5, 0.5, 0);
        self.Region.Size = UDim2.new(0, 19 + TextWidth + 4, 1, 0);
    end;

    Reposition();
    Label:GetPropertyChangedSignal('TextBounds'):Connect(Reposition);

    local Settings = MakePopup(210, function()
        return UDim2.fromOffset(Circle.AbsolutePosition.X + 4, Circle.AbsolutePosition.Y + 18);
    end, self.Groupbox.IsPopup and self.Groupbox or nil, { Circle }, Enum.UserInputType.MouseButton1);

    -- circle: grows slightly on hover, pops on click
    local function SizeCircle(Size, Time)
        TweenService:Create(Circle, TweenInfo.new(Time or 0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
            Size = UDim2.new(0, Size, 0, Size);
        }):Play();
    end;

    Circle.MouseEnter:Connect(function() SizeCircle(15); end);
    Circle.MouseLeave:Connect(function() SizeCircle(13); end);

    Circle.InputBegan:Connect(function(Input)
        if Input.UserInputType == Enum.UserInputType.MouseButton1 and not Library:MouseIsOverOpenedFrame() then
            SizeCircle(11, 0.06);
            task.delay(0.06, SizeCircle, 15, 0.1);

            Settings:SetOpen(not Settings.Open);
        end;
    end);

    self.Settings = Settings;

    return Settings;
end;

-- Binds ------------------------------------------------------------------
-- Right click a toggle / slider / dropdown to bind it. While the bind is active the element
-- takes the "bound value" set in the same window, and goes back to its normal value afterwards.

local function CopyValue(Value)
    if type(Value) == 'table' then
        local Copy = {};

        for Key, Val in next, Value do
            Copy[Key] = Val;
        end;

        return Copy;
    end;

    return Value;
end;

-- value that should be saved: the normal one, even while a bind is overriding it
function Library:GetBaseValue(Element)
    if Element.Bind and Element.Bind.Active then
        return Element.Bind.Stash;
    end;

    return Element.Value;
end;

local function BindKeyName(Input)
    if Input.UserInputType == Enum.UserInputType.Keyboard then
        return Input.KeyCode.Name;
    elseif Input.UserInputType == Enum.UserInputType.MouseButton1 then
        return 'MB1';
    elseif Input.UserInputType == Enum.UserInputType.MouseButton2 then
        return 'MB2';
    elseif Input.UserInputType == Enum.UserInputType.MouseButton3 then
        return 'MB3';
    end;
end;

function Library:AttachBind(Element, Region, Info, Idx, Groupbox)
    if Info.NoBind or Info.SpecialType or Element.Bind then
        return;
    end;

    local Kind = Element.Type;

    local Bind = {
        Type = 'KeyPicker'; -- same shape as a keypicker so the SaveManager stores it
        Value = 'None';
        Mode = 'Hold';
        Active = false;
        Listening = false;
    };

    local Popup = MakePopup(200, function()
        return UDim2.fromOffset(Mouse.X + 4, Mouse.Y + 4);
    end, Groupbox.IsPopup and Groupbox or nil, { Region }, Enum.UserInputType.MouseButton2);

    local KeyButton = Popup:AddButton({ Text = 'bind: none', Func = function() end });
    Popup:AddBlank(2);

    local ModeIdx = Idx .. '_bindmode';
    local ValueIdx = Idx .. '_bindvalue';

    Popup:AddDropdown(ModeIdx, { Text = 'mode', Values = { 'Hold', 'Toggle' }, Default = 1, NoBind = true });

    Popup:AddDivider();

    if Kind == 'Toggle' then
        Popup:AddToggle(ValueIdx, { Text = 'value while bound', Default = true, NoBind = true });
    elseif Kind == 'Slider' then
        Popup:AddSlider(ValueIdx, {
            Text = Info.Text or 'value';
            Default = Info.Default;
            Min = Info.Min;
            Max = Info.Max;
            Rounding = Info.Rounding;
            Suffix = Info.Suffix;
            NoBind = true;
        });
    elseif Kind == 'Dropdown' then
        Popup:AddDropdown(ValueIdx, {
            Text = Info.Text or 'value';
            Values = Info.Values;
            Multi = Info.Multi;
            Default = Info.Default;
            AllowNull = true;
            NoBind = true;
        });
    end;

    -- the mode is stored inside the bind itself, not as a separate saved option
    if Library.SaveManager then
        Library.SaveManager.Ignore[ModeIdx] = true;
    end;

    local ModeDropdown = Options[ModeIdx];
    local BoundValue = Options[ValueIdx] or Toggles[ValueIdx];

    function Bind:Update()
        local Text = Bind.Listening and '...' or Bind.Value;
        KeyButton.Label.Text = 'bind: ' .. string.lower(Text);
    end;

    function Bind:SetActive(Bool)
        Bool = not not Bool;

        if Bool == Bind.Active then
            return;
        end;

        Bind.Active = Bool;

        if Bool then
            Bind.Stash = CopyValue(Element.Value);
            Element:SetValue(CopyValue(BoundValue.Value));
        else
            local Stash = Bind.Stash;
            Bind.Stash = nil;
            Element:SetValue(Stash);
        end;
    end;

    function Bind:SetValue(Data)
        Bind:SetActive(false);

        Bind.Value = Data[1] or 'None';
        Bind.Mode = Data[2] or 'Hold';

        ModeDropdown:SetValue(Bind.Mode);
        Bind:Update();
    end;

    ModeDropdown:OnChanged(function(Value)
        Bind.Mode = Value or 'Hold';

        if Bind.Mode == 'Toggle' then
            Bind:SetActive(false);
        end;
    end);

    -- editing the bound value while the bind is active applies it straight away
    BoundValue:OnChanged(function(Value)
        if Bind.Active then
            Element:SetValue(CopyValue(Value));
        end;
    end);

    KeyButton.Func = function()
        if Bind.Listening then
            return;
        end;

        Bind.Listening = true;
        Bind:Update();

        task.delay(0.1, function()
            local Connection;

            Connection = InputService.InputBegan:Connect(function(Input)
                local Key = BindKeyName(Input);

                if not Key then
                    return;
                end;

                if Key == 'Escape' or Key == 'Backspace' then
                    Key = 'None';
                end;

                Connection:Disconnect();

                Bind:SetValue({ Key, Bind.Mode });

                -- keep ignoring input for a moment so the key just bound doesn't trigger it
                task.delay(0.15, function()
                    Bind.Listening = false;
                    Bind:Update();
                end);
            end);
        end);
    end;

    -- activation: never while typing (chat, text boxes) or while picking the key
    Library:GiveSignal(InputService.InputBegan:Connect(function(Input, Processed)
        if Bind.Listening or Bind.Value == 'None' then
            return;
        end;

        if InputService:GetFocusedTextBox() then
            return;
        end;

        local Key = BindKeyName(Input);

        if Key ~= Bind.Value then
            return;
        end;

        -- clicks on gui (including this menu) don't count for mouse binds
        if Processed and Input.UserInputType ~= Enum.UserInputType.Keyboard then
            return;
        end;

        if Bind.Mode == 'Toggle' then
            Bind:SetActive(not Bind.Active);
        else
            Bind:SetActive(true);
        end;
    end));

    Library:GiveSignal(InputService.InputEnded:Connect(function(Input)
        if Bind.Mode == 'Hold' and Bind.Active and BindKeyName(Input) == Bind.Value then
            Bind:SetActive(false);
        end;
    end));

    Region.InputBegan:Connect(function(Input)
        if Input.UserInputType == Enum.UserInputType.MouseButton2 and not Library:MouseIsOverOpenedFrame() then
            Popup:SetOpen(not Popup.Open);
        end;
    end);

    Element.Bind = Bind;
    Element.BindPopup = Popup;
    Options[Idx .. '_bind'] = Bind;

    Library:AddBindListEntry(Bind, Info.Text or Idx, Element, Info);

    Bind:Update();
end;

-- Button that picks the key which opens / closes the menu. Returns an object shaped like a keypicker.
function Library:AddMenuBind(Groupbox, Idx, Default)
    local MenuBind = {
        Type = 'KeyPicker';
        Value = Default or 'Insert';
        Mode = 'Toggle';
        Listening = false;
    };

    local Button;

    function MenuBind:Update()
        Button.Label.Text = 'menu key: ' .. string.lower(MenuBind.Listening and '...' or MenuBind.Value);
    end;

    function MenuBind:SetValue(Data)
        MenuBind.Value = Data[1] or MenuBind.Value;
        MenuBind:Update();
    end;

    Button = Groupbox:AddButton({ Text = 'menu key', Func = function()
        if MenuBind.Listening then
            return;
        end;

        MenuBind.Listening = true;
        MenuBind:Update();

        task.delay(0.1, function()
            local Connection;

            Connection = InputService.InputBegan:Connect(function(Input)
                if Input.UserInputType ~= Enum.UserInputType.Keyboard then
                    return;
                end;

                Connection:Disconnect();

                if Input.KeyCode.Name ~= 'Escape' then
                    MenuBind.Value = Input.KeyCode.Name;
                end;

                task.delay(0.15, function()
                    MenuBind.Listening = false;
                    MenuBind:Update();
                end);
            end);
        end);
    end });

    MenuBind:Update();

    Options[Idx] = MenuBind;
    Library.ToggleKeybind = MenuBind;

    return MenuBind;
end;

-- < Create other UI elements >
do
    -- notifications are plain outlined text stacked under the crosshair (screen center)
    Library.NotificationArea = Library:Create('Frame', {
        AnchorPoint = Vector2.new(0.5, 0);
        BackgroundTransparency = 1;
        Position = UDim2.new(0.5, 0, 0.5, 30 - game:GetService('GuiService'):GetGuiInset().Y / 2);
        Size = UDim2.new(0, 500, 0, 200);
        ZIndex = 100;
        Parent = ScreenGui;
    });

    Library:Create('UIListLayout', {
        Padding = UDim.new(0, 2);
        FillDirection = Enum.FillDirection.Vertical;
        HorizontalAlignment = Enum.HorizontalAlignment.Center;
        SortOrder = Enum.SortOrder.LayoutOrder;
        Parent = Library.NotificationArea;
    });

    -- watermark: plain outlined text pinned to the top right corner
    local WatermarkLabel = Library:Create('TextLabel', {
        AnchorPoint = Vector2.new(1, 0);
        AutomaticSize = Enum.AutomaticSize.X;
        BackgroundTransparency = 1;
        Font = Library.Font;
        -- the gui starts below the roblox topbar, so pull the label up by the inset to reach the real corner
        Position = UDim2.new(1, -8, 0, 6 - game:GetService('GuiService'):GetGuiInset().Y);
        Size = UDim2.new(0, 0, 0, 14);
        Text = '';
        TextColor3 = Library.FontColor;
        TextSize = 14;
        TextStrokeTransparency = 1;
        TextXAlignment = Enum.TextXAlignment.Right;
        Visible = false;
        ZIndex = 200;
        Parent = ScreenGui;
    });

    local WatermarkStroke = Library:Create('UIStroke', {
        Color = Color3.new(0, 0, 0);
        LineJoinMode = Enum.LineJoinMode.Miter;
        Thickness = 1;
        Parent = WatermarkLabel;
    });

    Library:AddToRegistry(WatermarkLabel, {
        TextColor3 = 'FontColor';
    });

    Library.Watermark = WatermarkLabel;
    Library.WatermarkText = WatermarkLabel;

    -- bind list: a small panel on the left of the screen. It always shows "binds"; a bind slides in
    -- while it is held / toggled on (or all binds with a key when "only active" is off)
    Library.BindListEnabled = true;
    Library.BindListOnlyActive = true;
    Library.BindEntries = {};

    local ROW_HEIGHT = 16;
    local HEADER_HEIGHT = 24;
    local MIN_WIDTH = 96;

    local BindList = Library:Create('Frame', {
        AnchorPoint = Vector2.new(0, 0.5);
        BackgroundColor3 = Library.BackgroundColor;
        BackgroundTransparency = 0.08;
        BorderColor3 = Library.OutlineColor;
        ClipsDescendants = true;
        Position = UDim2.new(0, 10, 0.5, -game:GetService('GuiService'):GetGuiInset().Y / 2);
        Size = UDim2.new(0, MIN_WIDTH, 0, HEADER_HEIGHT + 4);
        ZIndex = 200;
        Parent = ScreenGui;
    });
    Library:Round(BindList, 5);

    Library:AddToRegistry(BindList, {
        BackgroundColor3 = 'BackgroundColor';
        BorderColor3 = 'OutlineColor';
    });

    local function MakeListLabel(Parent)
        local Label = Library:Create('TextLabel', {
            BackgroundTransparency = 1;
            Font = Library.Font;
            Size = UDim2.new(1, -16, 0, ROW_HEIGHT);
            Text = '';
            TextColor3 = Library.FontColor;
            TextSize = 14;
            TextStrokeTransparency = 1;
            TextXAlignment = Enum.TextXAlignment.Left;
            ZIndex = 201;
            Parent = Parent;
        });

        Library:Create('UIStroke', {
            Color = Color3.new(0, 0, 0);
            LineJoinMode = Enum.LineJoinMode.Miter;
            Thickness = 1;
            Parent = Label;
        });

        return Label;
    end;

    local BindListTitle = MakeListLabel(BindList);
    BindListTitle.Position = UDim2.fromOffset(8, 4);
    BindListTitle.Text = 'binds';
    BindListTitle.TextColor3 = Library.AccentColor;

    Library:AddToRegistry(BindListTitle, {
        TextColor3 = 'AccentColor';
    });

    local BindRows = Library:Create('Frame', {
        BackgroundTransparency = 1;
        Position = UDim2.fromOffset(0, HEADER_HEIGHT);
        Size = UDim2.new(1, 0, 1, -HEADER_HEIGHT);
        ZIndex = 200;
        Parent = BindList;
    });

    Library:Create('UIListLayout', {
        FillDirection = Enum.FillDirection.Vertical;
        SortOrder = Enum.SortOrder.LayoutOrder;
        Parent = BindRows;
    });

    function Library:AddBindListEntry(Bind, Name, Element, Info)
        local Row = Library:Create('Frame', {
            BackgroundTransparency = 1;
            ClipsDescendants = true;
            LayoutOrder = #Library.BindEntries + 1;
            Size = UDim2.new(1, 0, 0, ROW_HEIGHT);
            Visible = false;
            ZIndex = 200;
            Parent = BindRows;
        });

        local Label = MakeListLabel(Row);
        Label.Position = UDim2.fromOffset(-12, 0);
        Label.TextTransparency = 1;

        table.insert(Library.BindEntries, { Bind = Bind, Name = Name, Element = Element, Info = Info, Row = Row, Label = Label, Shown = false });
    end;

    function Library:SetBindListVisibility(Bool)
        Library.BindListEnabled = not not Bool;
    end;

    function Library:SetBindListOnlyActive(Bool)
        Library.BindListOnlyActive = not not Bool;
    end;

    local ListTimer = 0;

    Library:GiveSignal(RenderStepped:Connect(function(Delta)
        ListTimer = ListTimer + Delta;

        if ListTimer < 0.05 then
            return;
        end;

        ListTimer = 0;

        BindList.Visible = Library.BindListEnabled;

        if not Library.BindListEnabled then
            return;
        end;

        local FontFace = Library.WatermarkText.FontFace;
        local TextSize = Library.WatermarkText.TextSize;

        BindListTitle.FontFace = FontFace;
        BindListTitle.TextSize = TextSize;

        local Width = Library:GetTextBounds('binds', Library.Font, TextSize) + 16;
        local Rows = 0;

        for _, Entry in next, Library.BindEntries do
            local Bind = Entry.Bind;
            local Label = Entry.Label;

            local Want = Bind.Value ~= 'None' and (Bind.Active or not Library.BindListOnlyActive);

            if Want then
                Rows = Rows + 1;

                Label.FontFace = FontFace;
                Label.TextSize = TextSize;
                -- sliders show the value they currently have, everything else just its name
                local Element = Entry.Element;

                if Element and Element.Type == 'Slider' then
                    Label.Text = ('%s  %s%s'):format(Entry.Name, tostring(Element.Value), (Entry.Info and Entry.Info.Suffix) or '');
                else
                    Label.Text = Entry.Name;
                end;
                Label.TextColor3 = Bind.Active and Library.AccentColor or Library.DimColor;

                Width = math.max(Width, Library:GetTextBounds(Label.Text, Library.Font, TextSize) + 16);
            end;

            if Want ~= Entry.Shown then
                Entry.Shown = Want;

                if Want then
                    Entry.Row.Visible = true;
                    Label.Position = UDim2.fromOffset(-12, 0);
                    Label.TextTransparency = 1;

                    Library:Tween(Label, { Position = UDim2.fromOffset(8, 0); TextTransparency = 0 }, 0.2);
                else
                    Library:Tween(Label, { Position = UDim2.fromOffset(12, 0); TextTransparency = 1 }, 0.15);

                    task.delay(0.16, function()
                        if not Entry.Shown then
                            Entry.Row.Visible = false;
                        end;
                    end);
                end;
            end;
        end;

        -- the panel grows / shrinks smoothly with the rows
        Library:Tween(BindList, {
            Size = UDim2.new(0, math.max(MIN_WIDTH, Width), 0, HEADER_HEIGHT + 4 + Rows * ROW_HEIGHT);
        }, 0.18);
    end));



    local KeybindOuter = Library:Create('Frame', {
        AnchorPoint = Vector2.new(0, 0.5);
        BorderColor3 = Color3.new(0, 0, 0);
        Position = UDim2.new(0, 10, 0.5, 0);
        Size = UDim2.new(0, 210, 0, 20);
        Visible = false;
        ZIndex = 100;
        Parent = ScreenGui;
    });
    Library:Round(KeybindOuter, 4);

    local KeybindInner = Library:Create('Frame', {
        BackgroundColor3 = Library.MainColor;
        BorderColor3 = Library.OutlineColor;
        BorderMode = Enum.BorderMode.Inset;
        Size = UDim2.new(1, 0, 1, 0);
        ZIndex = 101;
        Parent = KeybindOuter;
    });
    Library:Round(KeybindInner, 4);

    Library:AddToRegistry(KeybindInner, {
        BackgroundColor3 = 'MainColor';
        BorderColor3 = 'OutlineColor';
    }, true);

    local ColorFrame = Library:Create('Frame', {
        BackgroundColor3 = Library.AccentColor;
        BorderSizePixel = 0;
        Position = UDim2.new(0, 4, 0, 0);
        Size = UDim2.new(1, -8, 0, 2);
        ZIndex = 102;
        Parent = KeybindInner;
    });

    Library:AddToRegistry(ColorFrame, {
        BackgroundColor3 = 'AccentColor';
    }, true);

    local KeybindLabel = Library:CreateLabel({
        Size = UDim2.new(1, 0, 0, 20);
        Position = UDim2.fromOffset(5, 2),
        TextXAlignment = Enum.TextXAlignment.Left,

        Text = 'Keybinds';
        ZIndex = 104;
        Parent = KeybindInner;
    });

    local KeybindContainer = Library:Create('Frame', {
        BackgroundTransparency = 1;
        Size = UDim2.new(1, 0, 1, -20);
        Position = UDim2.new(0, 0, 0, 20);
        ZIndex = 1;
        Parent = KeybindInner;
    });

    Library:Create('UIListLayout', {
        FillDirection = Enum.FillDirection.Vertical;
        SortOrder = Enum.SortOrder.LayoutOrder;
        Parent = KeybindContainer;
    });

    Library:Create('UIPadding', {
        PaddingLeft = UDim.new(0, 5),
        Parent = KeybindContainer,
    })

    Library.KeybindFrame = KeybindOuter;
    Library.KeybindContainer = KeybindContainer;
    Library:MakeDraggable(KeybindOuter);
end;

function Library:SetWatermarkVisibility(Bool)
    Library.Watermark.Visible = Bool;
end;

function Library:SetWatermark(Text)
    Library.WatermarkText.Text = Text;
    Library:SetWatermarkVisibility(true)
end;

-- Loads a local .ttf as a custom font (needs getcustomasset / writefile).
-- Returns nil when the executor can't do it, so callers keep the default font.
function Library:LoadFont(Path)
    local Success, Result = pcall(function()
        local Json = Path:gsub('%.%w+$', '') .. '.json';

        writefile(Json, game:GetService('HttpService'):JSONEncode({
            name = 'CustomFont';
            faces = { {
                name = 'Regular';
                weight = 400;
                style = 'normal';
                assetId = getcustomasset(Path);
            } };
        }));

        return Font.new(getcustomasset(Json));
    end);

    if Success then
        return Result;
    end;
end;

-- Sets the watermark font (Font object from Library:LoadFont) and text size
function Library:SetWatermarkFont(FontFace, Size)
    if FontFace then
        Library.WatermarkText.FontFace = FontFace;
    end;

    Library.WatermarkText.TextSize = Size or 14;
end;

function Library:Notify(Text, Time)
    local Label = Library:Create('TextLabel', {
        BackgroundTransparency = 1;
        FontFace = Library.WatermarkText.FontFace;
        Size = UDim2.new(1, 0, 0, Library.WatermarkText.TextSize + 4);
        Text = Text;
        TextColor3 = Library.FontColor;
        TextSize = Library.WatermarkText.TextSize;
        TextStrokeTransparency = 1;
        TextTransparency = 1;
        ZIndex = 100;
        Parent = Library.NotificationArea;
    });

    local Stroke = Library:Create('UIStroke', {
        Color = Color3.new(0, 0, 0);
        LineJoinMode = Enum.LineJoinMode.Miter;
        Thickness = 1;
        Transparency = 1;
        Parent = Label;
    });

    Library:AddToRegistry(Label, {
        TextColor3 = 'FontColor';
    }, true);

    local FadeInfo = TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out);

    TweenService:Create(Label, FadeInfo, { TextTransparency = 0 }):Play();
    TweenService:Create(Stroke, FadeInfo, { Transparency = 0 }):Play();

    task.spawn(function()
        wait(Time or 5);

        TweenService:Create(Label, FadeInfo, { TextTransparency = 1 }):Play();
        TweenService:Create(Stroke, FadeInfo, { Transparency = 1 }):Play();

        wait(0.3);

        Label:Destroy();
    end);
end;

function Library:CreateWindow(...)
    local Arguments = { ... }
    local Config = { AnchorPoint = Vector2.zero }

    if type(...) == 'table' then
        Config = ...;
    else
        Config.Title = Arguments[1]
        Config.AutoShow = Arguments[2] or false;
    end

    if type(Config.Title) ~= 'string' then Config.Title = 'No title' end
    if type(Config.TabPadding) ~= 'number' then Config.TabPadding = 0 end
    if type(Config.MenuFadeTime) ~= 'number' then Config.MenuFadeTime = 0.2 end

    if typeof(Config.Position) ~= 'UDim2' then Config.Position = UDim2.fromOffset(175, 50) end
    if typeof(Config.Size) ~= 'UDim2' then Config.Size = UDim2.fromOffset(550, 600) end

    if Config.Center then
        Config.AnchorPoint = Vector2.new(0.5, 0.5)
        Config.Position = UDim2.fromScale(0.5, 0.5)
    end

    local Window = {
        Tabs = {};
    };

    local Outer = Library:Create('Frame', {
        AnchorPoint = Config.AnchorPoint,
        BackgroundColor3 = Color3.new(0, 0, 0);
        BorderSizePixel = 0;
        Position = Config.Position,
        Size = Config.Size,
        Visible = false;
        ZIndex = 1;
        Parent = ScreenGui;
    });
    Library:Round(Outer, 8);

    Library:MakeDraggable(Outer, 34);

    local Inner = Library:Create('Frame', {
        BackgroundColor3 = Library.MainColor;
        BorderColor3 = Library.OutlineColor;
        BorderMode = Enum.BorderMode.Inset;
        Position = UDim2.new(0, 1, 0, 1);
        Size = UDim2.new(1, -2, 1, -2);
        ZIndex = 1;
        Parent = Outer;
    });
    Library:Round(Inner, 7);

    Library:AddToRegistry(Inner, {
        BackgroundColor3 = 'MainColor';
        BorderColor3 = 'OutlineColor';
    });

    -- header: "name.suffix | tab tab tab"
    local TitleMain, TitleSuffix = string.match(Config.Title, '^([^%.]*)(%..*)$');
    local TitleWidth = Library:GetTextBounds(Config.Title, Library.Font, 14) + 2;

    local WindowLabel = Library:CreateLabel({
        Position = UDim2.new(0, 14, 0, 0);
        Size = UDim2.new(0, TitleWidth, 0, 34);
        Text = Config.Title or '';
        TextSize = 14;
        TextXAlignment = Enum.TextXAlignment.Left;
        ZIndex = 1;
        Parent = Inner;
    });

    if TitleMain then
        -- the part after the dot follows the accent color (re-colored whenever the accent changes)
        local function TitleText()
            local Accent = Library.AccentColor;

            return TitleMain .. ('<font color="#%s">%s</font>'):format(Accent:ToHex(), TitleSuffix);
        end;

        WindowLabel.RichText = true;
        WindowLabel.Text = TitleText();

        Library:AddToRegistry(WindowLabel, {
            Text = TitleText;
        });
    end;

    local Separator = Library:Create('Frame', {
        BackgroundColor3 = Library.OutlineColor;
        BorderSizePixel = 0;
        Position = UDim2.new(0, 14 + TitleWidth + 8, 0, 11);
        Size = UDim2.new(0, 1, 0, 12);
        ZIndex = 1;
        Parent = Inner;
    });

    Library:AddToRegistry(Separator, {
        BackgroundColor3 = 'OutlineColor';
    });

    local FooterLabel = Library:CreateLabel({
        Position = UDim2.new(0, 8, 1, -26);
        Size = UDim2.new(1, -22, 0, 26);
        Text = Config.Footer or '';
        TextSize = 13;
        TextXAlignment = Enum.TextXAlignment.Right;
        ZIndex = 1;
        Parent = Inner;
    });

    -- clicking the footer text opens a small profile card
    local FooterWidth = Library:GetTextBounds(Config.Footer or '', Library.Font, 13);

    local FooterButton = Library:Create('TextButton', {
        AnchorPoint = Vector2.new(1, 0);
        BackgroundTransparency = 1;
        Position = UDim2.new(1, -8, 1, -26);
        Size = UDim2.new(0, FooterWidth + 12, 0, 26);
        Text = '';
        ZIndex = 3;
        Parent = Inner;
    });

    local ProfileCard = Library:Create('Frame', {
        AnchorPoint = Vector2.new(0, 1);
        BackgroundColor3 = Library.BackgroundColor;
        BorderColor3 = Library.OutlineColor;
        Position = UDim2.new(1, 8, 1, 0);
        Size = UDim2.new(0, 210, 0, 92);
        Visible = false;
        ZIndex = 50;
        Parent = Inner;
    });
    Library:Round(ProfileCard, 6);

    Library:AddToRegistry(ProfileCard, {
        BackgroundColor3 = 'BackgroundColor';
        BorderColor3 = 'OutlineColor';
    });

    local ProfileAvatar = Library:Create('ImageLabel', {
        BackgroundColor3 = Library.MainColor;
        BorderSizePixel = 0;
        Position = UDim2.new(0, 10, 0, 10);
        Size = UDim2.new(0, 48, 0, 48);
        ZIndex = 51;
        Parent = ProfileCard;
    });
    Library:Round(ProfileAvatar, 24);

    Library:AddToRegistry(ProfileAvatar, {
        BackgroundColor3 = 'MainColor';
    });

    local ProfileName = Library:CreateLabel({
        Position = UDim2.new(0, 68, 0, 12);
        Size = UDim2.new(1, -76, 0, 16);
        Text = LocalPlayer.DisplayName;
        TextSize = 14;
        TextXAlignment = Enum.TextXAlignment.Left;
        ZIndex = 51;
        Parent = ProfileCard;
    });

    local ProfileUser = Library:CreateLabel({
        Position = UDim2.new(0, 68, 0, 30);
        Size = UDim2.new(1, -76, 0, 14);
        Text = '@' .. LocalPlayer.Name;
        TextColor3 = Library.DimColor;
        TextSize = 12;
        TextXAlignment = Enum.TextXAlignment.Left;
        ZIndex = 51;
        Parent = ProfileCard;
    });
    Library.RegistryMap[ProfileUser].Properties.TextColor3 = 'DimColor';

    local ProfileInfo = Library:CreateLabel({
        Position = UDim2.new(0, 10, 0, 66);
        Size = UDim2.new(1, -20, 0, 14);
        Text = string.format('uid %d  |  %s  |  %dd', LocalPlayer.UserId, Config.Footer or 'user', LocalPlayer.AccountAge);
        TextColor3 = Library.DimColor;
        TextSize = 12;
        TextXAlignment = Enum.TextXAlignment.Left;
        ZIndex = 51;
        Parent = ProfileCard;
    });
    Library.RegistryMap[ProfileInfo].Properties.TextColor3 = 'DimColor';

    task.spawn(function()
        local Ok, Image = pcall(Players.GetUserThumbnailAsync, Players, LocalPlayer.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size100x100);
        if Ok then
            ProfileAvatar.Image = Image;
        end;
    end);

    FooterButton.MouseButton1Click:Connect(function()
        ProfileCard.Visible = not ProfileCard.Visible;
    end);

    FooterButton.MouseEnter:Connect(function()
        FooterLabel.TextColor3 = Library.AccentColor;
    end);

    FooterButton.MouseLeave:Connect(function()
        FooterLabel.TextColor3 = Library.FontColor;
    end);

    local MainSectionOuter = Library:Create('Frame', {
        BackgroundColor3 = Library.BackgroundColor;
        BorderColor3 = Library.OutlineColor;
        Position = UDim2.new(0, 8, 0, 34);
        Size = UDim2.new(1, -16, 1, -34 - 28);
        ZIndex = 1;
        Parent = Inner;
    });
    Library:Round(MainSectionOuter, 6);

    Library:AddToRegistry(MainSectionOuter, {
        BackgroundColor3 = 'BackgroundColor';
        BorderColor3 = 'OutlineColor';
    });

    local MainSectionInner = Library:Create('Frame', {
        BackgroundColor3 = Library.BackgroundColor;
        BorderSizePixel = 0;
        Position = UDim2.new(0, 0, 0, 0);
        Size = UDim2.new(1, 0, 1, 0);
        ZIndex = 1;
        Parent = MainSectionOuter;
    });
    Library:Round(MainSectionInner, 6);

    Library:AddToRegistry(MainSectionInner, {
        BackgroundColor3 = 'BackgroundColor';
    });

    local TabArea = Library:Create('Frame', {
        BackgroundTransparency = 1;
        Position = UDim2.new(0, 14 + TitleWidth + 18, 0, 0);
        Size = UDim2.new(1, -(14 + TitleWidth + 26), 0, 34);
        ZIndex = 1;
        Parent = Inner;
    });

    local TabListLayout = Library:Create('UIListLayout', {
        Padding = UDim.new(0, Config.TabPadding);
        FillDirection = Enum.FillDirection.Horizontal;
        SortOrder = Enum.SortOrder.LayoutOrder;
        Parent = TabArea;
    });

    local TabContainer = Library:Create('Frame', {
        BackgroundTransparency = 1;
        BorderSizePixel = 0;
        Position = UDim2.new(0, 0, 0, 0);
        Size = UDim2.new(1, 0, 1, 0);
        ZIndex = 2;
        Parent = MainSectionInner;
    });

    function Window:SetWindowTitle(Title)
        WindowLabel.Text = Title;
    end;

    function Window:AddTab(Name)
        local Tab = {
            Groupboxes = {};
            Tabboxes = {};
        };

        local TabButtonWidth = Library:GetTextBounds(Name, Library.Font, 16);

        local TabButton = Library:Create('Frame', {
            BackgroundTransparency = 1;
            BorderSizePixel = 0;
            Size = UDim2.new(0, TabButtonWidth + 12, 1, 0);
            ZIndex = 1;
            Parent = TabArea;
        });

        local TabButtonLabel = Library:CreateLabel({
            Position = UDim2.new(0, 0, 0, 0);
            Size = UDim2.new(1, 0, 1, 0);
            Text = Name;
            TextSize = 14;
            TextColor3 = Library.DimColor;
            ZIndex = 1;
            Parent = TabButton;
        });

        Library.RegistryMap[TabButtonLabel].Properties.TextColor3 = 'DimColor';

        local TabFrame = Library:Create('Frame', {
            Name = 'TabFrame',
            BackgroundTransparency = 1;
            Position = UDim2.new(0, 0, 0, 0);
            Size = UDim2.new(1, 0, 1, 0);
            Visible = false;
            ZIndex = 2;
            Parent = TabContainer;
        });

        local LeftSide = Library:Create('ScrollingFrame', {
            BackgroundTransparency = 1;
            BorderSizePixel = 0;
            Position = UDim2.new(0, 8 - 1, 0, 8 - 1);
            Size = UDim2.new(0.5, -12 + 2, 1, -14);
            CanvasSize = UDim2.new(0, 0, 0, 0);
            BottomImage = '';
            TopImage = '';
            ScrollBarThickness = 0;
            ZIndex = 2;
            Parent = TabFrame;
        });

        local RightSide = Library:Create('ScrollingFrame', {
            BackgroundTransparency = 1;
            BorderSizePixel = 0;
            Position = UDim2.new(0.5, 4 + 1, 0, 8 - 1);
            Size = UDim2.new(0.5, -12 + 2, 1, -14);
            CanvasSize = UDim2.new(0, 0, 0, 0);
            BottomImage = '';
            TopImage = '';
            ScrollBarThickness = 0;
            ZIndex = 2;
            Parent = TabFrame;
        });

        Library:Create('UIListLayout', {
            Padding = UDim.new(0, 8);
            FillDirection = Enum.FillDirection.Vertical;
            SortOrder = Enum.SortOrder.LayoutOrder;
            HorizontalAlignment = Enum.HorizontalAlignment.Center;
            Parent = LeftSide;
        });

        Library:Create('UIListLayout', {
            Padding = UDim.new(0, 8);
            FillDirection = Enum.FillDirection.Vertical;
            SortOrder = Enum.SortOrder.LayoutOrder;
            HorizontalAlignment = Enum.HorizontalAlignment.Center;
            Parent = RightSide;
        });

        for _, Side in next, { LeftSide, RightSide } do
            -- top padding leaves room for the groupbox label that sits on the border
            -- 1px on the sides / bottom so the outside stroke of the boxes isn't clipped by the scrolling frame
            Library:Create('UIPadding', {
                PaddingTop = UDim.new(0, 9);
                PaddingLeft = UDim.new(0, 1);
                PaddingRight = UDim.new(0, 1);
                PaddingBottom = UDim.new(0, 1);
                Parent = Side;
            });

            Side:WaitForChild('UIListLayout'):GetPropertyChangedSignal('AbsoluteContentSize'):Connect(function()
                Side.CanvasSize = UDim2.fromOffset(0, Side.UIListLayout.AbsoluteContentSize.Y + 11);
            end);
        end;

        function Tab:ShowTab()
            Library:ClosePopups();

            for _, Tab in next, Window.Tabs do
                Tab:HideTab();
            end;

            Library:Tween(TabButtonLabel, { TextColor3 = Library.FontColor }, 0.15);
            Library.RegistryMap[TabButtonLabel].Properties.TextColor3 = 'FontColor';

            -- content slides up into place
            TabFrame.Position = UDim2.new(0, 0, 0, 10);
            TabFrame.Visible = true;
            Library:Tween(TabFrame, { Position = UDim2.new(0, 0, 0, 0) }, 0.2);
        end;

        function Tab:HideTab()
            Library:Tween(TabButtonLabel, { TextColor3 = Library.DimColor }, 0.15);
            Library.RegistryMap[TabButtonLabel].Properties.TextColor3 = 'DimColor';
            TabFrame.Visible = false;
        end;

        function Tab:SetLayoutOrder(Position)
            TabButton.LayoutOrder = Position;
            TabListLayout:ApplyLayout();
        end;

        function Tab:AddGroupbox(Info)
            local Groupbox = {};

            local BoxOuter = Library:Create('Frame', {
                BackgroundColor3 = Library.BackgroundColor;
                BorderColor3 = Library.OutlineColor;
                BorderMode = Enum.BorderMode.Inset;
                Size = UDim2.new(1, 0, 0, 507 + 2);
                ZIndex = 2;
                Parent = Info.Side == 1 and LeftSide or RightSide;
            });
            Library:Round(BoxOuter, 6);

            Library:AddToRegistry(BoxOuter, {
                BackgroundColor3 = 'BackgroundColor';
                BorderColor3 = 'OutlineColor';
            });

            local BoxInner = Library:Create('Frame', {
                BackgroundColor3 = Library.BackgroundColor;
                BorderSizePixel = 0;
                Size = UDim2.new(1, -2, 1, -2);
                Position = UDim2.new(0, 1, 0, 1);
                ZIndex = 4;
                Parent = BoxOuter;
            });
            Library:Round(BoxInner, 5);

            Library:AddToRegistry(BoxInner, {
                BackgroundColor3 = 'BackgroundColor';
            });

            -- label sits on top of the border line, followed by a small chevron
            local LabelWidth = Library:GetTextBounds(Info.Name, Library.Font, 12);

            local LabelHolder = Library:Create('Frame', {
                BackgroundColor3 = Library.BackgroundColor;
                BorderSizePixel = 0;
                Position = UDim2.new(0, 10, 0, -7);
                Size = UDim2.new(0, LabelWidth + 26, 0, 14);
                ZIndex = 6;
                Parent = BoxOuter;
            });

            Library:AddToRegistry(LabelHolder, {
                BackgroundColor3 = 'BackgroundColor';
            });

            local GroupboxLabel = Library:CreateLabel({
                Size = UDim2.new(0, LabelWidth + 4, 1, 0);
                Position = UDim2.new(0, 4, 0, 0);
                TextSize = 12;
                Text = Info.Name;
                TextXAlignment = Enum.TextXAlignment.Left;
                ZIndex = 7;
                Parent = LabelHolder;
            });

            -- chevron drawn from two thin rotated bars
            local GroupboxArrow = Library:Create('Frame', {
                BackgroundTransparency = 1;
                Position = UDim2.new(0, LabelWidth + 10, 0.5, -3);
                Size = UDim2.new(0, 8, 0, 6);
                ZIndex = 7;
                Parent = LabelHolder;
            });

            for _, Side in next, { { 0.25, 45 }, { 0.75, -45 } } do
                local Bar = Library:Create('Frame', {
                    AnchorPoint = Vector2.new(0.5, 0.5);
                    BackgroundColor3 = Library.DimColor;
                    BorderSizePixel = 0;
                    Position = UDim2.new(Side[1], 0, 0.5, 0);
                    Rotation = Side[2];
                    Size = UDim2.new(0, 5, 0, 1);
                    ZIndex = 7;
                    Parent = GroupboxArrow;
                });

                Library:AddToRegistry(Bar, {
                    BackgroundColor3 = 'DimColor';
                });
            end;

            local Container = Library:Create('Frame', {
                BackgroundTransparency = 1;
                Position = UDim2.new(0, 4, 0, 10);
                Size = UDim2.new(1, -4, 1, -10);
                ZIndex = 1;
                Parent = BoxInner;
            });

            Library:Create('UIListLayout', {
                FillDirection = Enum.FillDirection.Vertical;
                SortOrder = Enum.SortOrder.LayoutOrder;
                Parent = Container;
            });

            function Groupbox:Resize()
                local Size = 0;

                for _, Element in next, Groupbox.Container:GetChildren() do
                    if (not Element:IsA('UIListLayout')) and Element.Visible then
                        Size = Size + Element.Size.Y.Offset;
                    end;
                end;

                BoxOuter.Size = UDim2.new(1, 0, 0, 10 + Size + 2 + 2);
            end;

            Groupbox.Container = Container;
            setmetatable(Groupbox, BaseGroupbox);

            Groupbox:AddBlank(3);
            Groupbox:Resize();

            Tab.Groupboxes[Info.Name] = Groupbox;

            return Groupbox;
        end;

        function Tab:AddLeftGroupbox(Name)
            return Tab:AddGroupbox({ Side = 1; Name = Name; });
        end;

        function Tab:AddRightGroupbox(Name)
            return Tab:AddGroupbox({ Side = 2; Name = Name; });
        end;

        function Tab:AddTabbox(Info)
            local Tabbox = {
                Tabs = {};
            };

            local BoxOuter = Library:Create('Frame', {
                BackgroundColor3 = Library.BackgroundColor;
                BorderColor3 = Library.OutlineColor;
                BorderMode = Enum.BorderMode.Inset;
                Size = UDim2.new(1, 0, 0, 0);
                ZIndex = 2;
                Parent = Info.Side == 1 and LeftSide or RightSide;
            });
            Library:Round(BoxOuter, 6);

            Library:AddToRegistry(BoxOuter, {
                BackgroundColor3 = 'BackgroundColor';
                BorderColor3 = 'OutlineColor';
            });

            local BoxInner = Library:Create('Frame', {
                BackgroundColor3 = Library.BackgroundColor;
                BorderSizePixel = 0;
                Size = UDim2.new(1, -2, 1, -2);
                Position = UDim2.new(0, 1, 0, 1);
                ZIndex = 4;
                Parent = BoxOuter;
            });
            Library:Round(BoxInner, 5);

            Library:AddToRegistry(BoxInner, {
                BackgroundColor3 = 'BackgroundColor';
            });

            local Highlight = Library:Create('Frame', {
                BackgroundColor3 = Library.AccentColor;
                BorderSizePixel = 0;
                BackgroundTransparency = 1;
                Position = UDim2.new(0, 5, 0, 0);
                Size = UDim2.new(1, -10, 0, 2);
                ZIndex = 10;
                Parent = BoxInner;
            });

            Library:AddToRegistry(Highlight, {
                BackgroundColor3 = 'AccentColor';
            });

            local TabboxButtons = Library:Create('Frame', {
                BackgroundTransparency = 1;
                Position = UDim2.new(0, 0, 0, 1);
                Size = UDim2.new(1, 0, 0, 18);
                ZIndex = 5;
                Parent = BoxInner;
            });

            Library:Create('UIListLayout', {
                FillDirection = Enum.FillDirection.Horizontal;
                HorizontalAlignment = Enum.HorizontalAlignment.Left;
                SortOrder = Enum.SortOrder.LayoutOrder;
                Parent = TabboxButtons;
            });

            function Tabbox:AddTab(Name)
                local Tab = {};

                local Button = Library:Create('Frame', {
                    BackgroundColor3 = Library.MainColor;
                    BorderColor3 = Color3.new(0, 0, 0);
                    Size = UDim2.new(0.5, 0, 1, 0);
                    ZIndex = 6;
                    Parent = TabboxButtons;
                });

                Library:AddToRegistry(Button, {
                    BackgroundColor3 = 'MainColor';
                });

                local ButtonLabel = Library:CreateLabel({
                    Size = UDim2.new(1, 0, 1, 0);
                    TextSize = 14;
                    Text = Name;
                    TextXAlignment = Enum.TextXAlignment.Center;
                    ZIndex = 7;
                    Parent = Button;
                });

                local Block = Library:Create('Frame', {
                    BackgroundColor3 = Library.BackgroundColor;
                    BorderSizePixel = 0;
                    Position = UDim2.new(0, 0, 1, 0);
                    Size = UDim2.new(1, 0, 0, 1);
                    Visible = false;
                    ZIndex = 9;
                    Parent = Button;
                });

                Library:AddToRegistry(Block, {
                    BackgroundColor3 = 'BackgroundColor';
                });

                local Container = Library:Create('Frame', {
                    BackgroundTransparency = 1;
                    Position = UDim2.new(0, 4, 0, 20);
                    Size = UDim2.new(1, -4, 1, -20);
                    ZIndex = 1;
                    Visible = false;
                    Parent = BoxInner;
                });

                Library:Create('UIListLayout', {
                    FillDirection = Enum.FillDirection.Vertical;
                    SortOrder = Enum.SortOrder.LayoutOrder;
                    Parent = Container;
                });

                function Tab:Show()
                    for _, Tab in next, Tabbox.Tabs do
                        Tab:Hide();
                    end;

                    Container.Visible = true;
                    Block.Visible = true;

                    Button.BackgroundColor3 = Library.BackgroundColor;
                    Library.RegistryMap[Button].Properties.BackgroundColor3 = 'BackgroundColor';

                    Tab:Resize();
                end;

                function Tab:Hide()
                    Container.Visible = false;
                    Block.Visible = false;

                    Button.BackgroundColor3 = Library.MainColor;
                    Library.RegistryMap[Button].Properties.BackgroundColor3 = 'MainColor';
                end;

                function Tab:Resize()
                    local TabCount = 0;

                    for _, Tab in next, Tabbox.Tabs do
                        TabCount = TabCount + 1;
                    end;

                    for _, Button in next, TabboxButtons:GetChildren() do
                        if not Button:IsA('UIListLayout') then
                            Button.Size = UDim2.new(1 / TabCount, 0, 1, 0);
                        end;
                    end;

                    if (not Container.Visible) then
                        return;
                    end;

                    local Size = 0;

                    for _, Element in next, Tab.Container:GetChildren() do
                        if (not Element:IsA('UIListLayout')) and Element.Visible then
                            Size = Size + Element.Size.Y.Offset;
                        end;
                    end;

                    BoxOuter.Size = UDim2.new(1, 0, 0, 20 + Size + 2 + 2);
                end;

                Button.InputBegan:Connect(function(Input)
                    if Input.UserInputType == Enum.UserInputType.MouseButton1 and not Library:MouseIsOverOpenedFrame() then
                        Tab:Show();
                        Tab:Resize();
                    end;
                end);

                Tab.Container = Container;
                Tabbox.Tabs[Name] = Tab;

                setmetatable(Tab, BaseGroupbox);

                Tab:AddBlank(3);
                Tab:Resize();

                -- Show first tab (number is 2 cus of the UIListLayout that also sits in that instance)
                if #TabboxButtons:GetChildren() == 2 then
                    Tab:Show();
                end;

                return Tab;
            end;

            Tab.Tabboxes[Info.Name or ''] = Tabbox;

            return Tabbox;
        end;

        function Tab:AddLeftTabbox(Name)
            return Tab:AddTabbox({ Name = Name, Side = 1; });
        end;

        function Tab:AddRightTabbox(Name)
            return Tab:AddTabbox({ Name = Name, Side = 2; });
        end;

        TabButton.InputBegan:Connect(function(Input)
            if Input.UserInputType == Enum.UserInputType.MouseButton1 then
                Tab:ShowTab();
            end;
        end);

        -- This was the first tab added, so we show it by default.
        if #TabContainer:GetChildren() == 1 then
            Tab:ShowTab();
        end;

        Window.Tabs[Name] = Tab;
        return Tab;
    end;

    local ModalElement = Library:Create('TextButton', {
        BackgroundTransparency = 1;
        Size = UDim2.new(0, 0, 0, 0);
        Visible = true;
        Text = '';
        Modal = false;
        Parent = ScreenGui;
    });

    local TransparencyCache = {};
    local Toggled = false;
    local Fading = false;

    function Library:Toggle()
        if Fading then
            return;
        end;

        Library:ClosePopups();

        local FadeTime = Config.MenuFadeTime;
        Fading = true;
        Toggled = (not Toggled);
        ModalElement.Modal = Toggled;

        if Toggled then
            -- A bit scuffed, but if we're going from not toggled -> toggled we want to show the frame immediately so that the fade is visible.
            Outer.Visible = true;

            task.spawn(function()
                -- TODO: add cursor fade?
                local State = InputService.MouseIconEnabled;

                -- plain dot cursor: accent colored fill with a black ring around it
                local Ring = Drawing.new('Circle');
                Ring.Filled = true;
                Ring.NumSides = 32;
                Ring.Radius = 5;
                Ring.Color = Color3.new(0, 0, 0);
                Ring.ZIndex = 1;
                Ring.Visible = true;

                local Dot = Drawing.new('Circle');
                Dot.Filled = true;
                Dot.NumSides = 32;
                Dot.Radius = 3.5;
                Dot.ZIndex = 2;
                Dot.Visible = true;

                while Toggled and ScreenGui.Parent do
                    InputService.MouseIconEnabled = false;

                    local mPos = InputService:GetMouseLocation();

                    Ring.Position = mPos;
                    Dot.Position = mPos;
                    Dot.Color = Library.CursorColor or Library.AccentColor;

                    RenderStepped:Wait();
                end;

                InputService.MouseIconEnabled = State;

                Ring:Remove();
                Dot:Remove();
            end);
        end;

        for _, Desc in next, Outer:GetDescendants() do
            local Properties = {};

            if Desc:IsA('ImageLabel') then
                table.insert(Properties, 'ImageTransparency');
                table.insert(Properties, 'BackgroundTransparency');
            elseif Desc:IsA('TextLabel') or Desc:IsA('TextBox') then
                table.insert(Properties, 'TextTransparency');
            elseif Desc:IsA('Frame') or Desc:IsA('ScrollingFrame') then
                table.insert(Properties, 'BackgroundTransparency');
            elseif Desc:IsA('UIStroke') then
                table.insert(Properties, 'Transparency');
            end;

            local Cache = TransparencyCache[Desc];

            if (not Cache) then
                Cache = {};
                TransparencyCache[Desc] = Cache;
            end;

            for _, Prop in next, Properties do
                if not Cache[Prop] then
                    Cache[Prop] = Desc[Prop];
                end;

                if Cache[Prop] == 1 then
                    continue;
                end;

                TweenService:Create(Desc, TweenInfo.new(FadeTime, Enum.EasingStyle.Linear), { [Prop] = Toggled and Cache[Prop] or 1 }):Play();
            end;
        end;

        task.wait(FadeTime);

        Outer.Visible = Toggled;

        Fading = false;
    end

    Library:GiveSignal(InputService.InputBegan:Connect(function(Input, Processed)
        if type(Library.ToggleKeybind) == 'table' and Library.ToggleKeybind.Type == 'KeyPicker' then
            if Input.UserInputType == Enum.UserInputType.Keyboard and Input.KeyCode.Name == Library.ToggleKeybind.Value
                and not Library.ToggleKeybind.Listening and not InputService:GetFocusedTextBox() then

                task.spawn(Library.Toggle)
            end
        elseif Input.KeyCode == Enum.KeyCode.RightControl or (Input.KeyCode == Enum.KeyCode.RightShift and (not Processed)) then
            task.spawn(Library.Toggle)
        end
    end))

    if Config.AutoShow then task.spawn(Library.Toggle) end

    Window.Holder = Outer;

    return Window;
end;

local function OnPlayerChange()
    local PlayerList = GetPlayersString();

    for _, Value in next, Options do
        if Value.Type == 'Dropdown' and Value.SpecialType == 'Player' then
            Value:SetValues(PlayerList);
        end;
    end;
end;

Players.PlayerAdded:Connect(OnPlayerChange);
Players.PlayerRemoving:Connect(OnPlayerChange);

local HttpService = game:GetService('HttpService');

local SaveManager = {} do
    SaveManager.Library = Library
    SaveManager.Folder = 'LinoriaLibSettings'
    SaveManager.Ignore = {}
    SaveManager.Parser = {
        Toggle = {
            Save = function(idx, object)
                return { type = 'Toggle', idx = idx, value = Library:GetBaseValue(object) }
            end,
            Load = function(idx, data)
                if Toggles[idx] then
                    Toggles[idx]:SetValue(data.value)
                end
            end,
        },
        Slider = {
            Save = function(idx, object)
                return { type = 'Slider', idx = idx, value = tostring(Library:GetBaseValue(object)) }
            end,
            Load = function(idx, data)
                if Options[idx] then
                    Options[idx]:SetValue(data.value)
                end
            end,
        },
        Dropdown = {
            Save = function(idx, object)
                return { type = 'Dropdown', idx = idx, value = Library:GetBaseValue(object), mutli = object.Multi }
            end,
            Load = function(idx, data)
                if Options[idx] then
                    Options[idx]:SetValue(data.value)
                end
            end,
        },
        ColorPicker = {
            Save = function(idx, object)
                return {
                    type = 'ColorPicker', idx = idx,
                    value = object.Colors[1]:ToHex(), transparency = object.Transparency,
                    mode = object.Mode, speed = object.Speed, color2 = object.Colors[2]:ToHex(),
                }
            end,
            Load = function(idx, data)
                if Options[idx] then
                    Options[idx]:SetValueRGB(Color3.fromHex(data.value), data.transparency)

                    if data.mode then
                        Options[idx]:SetEffect(data.mode, data.speed, data.color2 and Color3.fromHex(data.color2))
                    end
                end
            end,
        },
        KeyPicker = {
            Save = function(idx, object)
                return { type = 'KeyPicker', idx = idx, mode = object.Mode, key = object.Value }
            end,
            Load = function(idx, data)
                if Options[idx] then
                    Options[idx]:SetValue({ data.key, data.mode })
                end
            end,
        },
        Input = {
            Save = function(idx, object)
                return { type = 'Input', idx = idx, text = object.Value }
            end,
            Load = function(idx, data)
                if Options[idx] and type(data.text) == 'string' then
                    Options[idx]:SetValue(data.text)
                end
            end,
        },
    }

    function SaveManager:SetIgnoreIndexes(list)
        for _, key in next, list do
            self.Ignore[key] = true
        end
    end

    function SaveManager:SetFolder(folder)
        self.Folder = folder
        self:BuildFolderTree()
    end

    function SaveManager:Save(name)
        if (not name) then
            return false, 'no config file is selected'
        end

        local fullPath = self.Folder .. '/settings/' .. name .. '.json'

        local data = {
            objects = {}
        }

        for idx, toggle in next, Toggles do
            if self.Ignore[idx] then continue end

            table.insert(data.objects, self.Parser[toggle.Type].Save(idx, toggle))
        end

        for idx, option in next, Options do
            if not self.Parser[option.Type] then continue end
            if self.Ignore[idx] then continue end

            table.insert(data.objects, self.Parser[option.Type].Save(idx, option))
        end

        local success, encoded = pcall(HttpService.JSONEncode, HttpService, data)
        if not success then
            return false, 'failed to encode data'
        end

        writefile(fullPath, encoded)
        return true
    end

    function SaveManager:Load(name)
        if (not name) then
            return false, 'no config file is selected'
        end

        local file = self.Folder .. '/settings/' .. name .. '.json'
        if not isfile(file) then return false, 'invalid file' end

        local success, decoded = pcall(HttpService.JSONDecode, HttpService, readfile(file))
        if not success then return false, 'decode error' end

        for _, option in next, decoded.objects do
            if self.Parser[option.type] then
                task.spawn(function() self.Parser[option.type].Load(option.idx, option) end) -- task.spawn() so the config loading wont get stuck.
            end
        end

        return true
    end

    function SaveManager:BuildFolderTree()
        local paths = {
            self.Folder,
            self.Folder .. '/settings'
        }

        for i = 1, #paths do
            local str = paths[i]
            if not isfolder(str) then
                makefolder(str)
            end
        end
    end

    function SaveManager:RefreshConfigList()
        local list = listfiles(self.Folder .. '/settings')

        local out = {}
        for i = 1, #list do
            local file = list[i]
            if file:sub(-5) == '.json' then
                local pos = file:find('.json', 1, true)
                local start = pos

                local char = file:sub(pos, pos)
                while char ~= '/' and char ~= '\\' and char ~= '' do
                    pos = pos - 1
                    char = file:sub(pos, pos)
                end

                if char == '/' or char == '\\' then
                    table.insert(out, file:sub(pos + 1, start - 1))
                end
            end
        end

        return out
    end

    function SaveManager:LoadAutoloadConfig()
        if isfile(self.Folder .. '/settings/autoload.txt') then
            local name = readfile(self.Folder .. '/settings/autoload.txt')

            local success, err = self:Load(name)
            if not success then
                return self.Library:Notify('Failed to load autoload config: ' .. err)
            end

            self.Library:Notify(string.format('Auto loaded config %q', name))
        end
    end

    function SaveManager:BuildConfigSection(tab)
        local section = tab:AddRightGroupbox('Configuration')

        section:AddInput('SaveManager_ConfigName',    { Text = 'Config name' })
        section:AddDropdown('SaveManager_ConfigList', { Text = 'Config list', Values = self:RefreshConfigList(), AllowNull = true, NoBind = true })

        section:AddDivider()

        section:AddButton('Create config', function()
            local name = Options.SaveManager_ConfigName.Value

            if name:gsub(' ', '') == '' then
                return self.Library:Notify('Invalid config name (empty)', 2)
            end

            local success, err = self:Save(name)
            if not success then
                return self.Library:Notify('Failed to save config: ' .. err)
            end

            self.Library:Notify(string.format('Created config %q', name))

            Options.SaveManager_ConfigList:SetValues(self:RefreshConfigList())
            Options.SaveManager_ConfigList:SetValue(nil)
        end):AddButton('Load config', function()
            local name = Options.SaveManager_ConfigList.Value

            local success, err = self:Load(name)
            if not success then
                return self.Library:Notify('Failed to load config: ' .. err)
            end

            self.Library:Notify(string.format('Loaded config %q', name))
        end)

        section:AddButton('Overwrite config', function()
            local name = Options.SaveManager_ConfigList.Value

            local success, err = self:Save(name)
            if not success then
                return self.Library:Notify('Failed to overwrite config: ' .. err)
            end

            self.Library:Notify(string.format('Overwrote config %q', name))
        end)

        section:AddButton('Refresh list', function()
            Options.SaveManager_ConfigList:SetValues(self:RefreshConfigList())
            Options.SaveManager_ConfigList:SetValue(nil)
        end)

        section:AddButton('Set as autoload', function()
            local name = Options.SaveManager_ConfigList.Value
            writefile(self.Folder .. '/settings/autoload.txt', name)
            SaveManager.AutoloadLabel:SetText('Current autoload config: ' .. name)
            self.Library:Notify(string.format('Set %q to auto load', name))
        end)

        SaveManager.AutoloadLabel = section:AddLabel('Current autoload config: none', true)

        if isfile(self.Folder .. '/settings/autoload.txt') then
            local name = readfile(self.Folder .. '/settings/autoload.txt')
            SaveManager.AutoloadLabel:SetText('Current autoload config: ' .. name)
        end

        SaveManager:SetIgnoreIndexes({ 'SaveManager_ConfigList', 'SaveManager_ConfigName' })
    end

    SaveManager:BuildFolderTree()
end

Library.SaveManager = SaveManager

getgenv().Library = Library
return Library

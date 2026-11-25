## 1. Luau Syntax, Typing & Asynchronous Runtime

Lua 5.1 type.

### 1.1 Type Annotations & Declarations
Luau type annotations are parsed natively and skipped at runtime with zero overhead, allowing scripts written with full type checking to run smoothly.

- **Function Parameters & Return Types**:
```luau
local function calculate_angle(base: number, offset: number?): (number, boolean)
    local result: number = base + (offset or 0)
    return result, true
end
```
- **Local Variable Annotations**:
```luau
local is_active: boolean = true
local target_name: string? = nil
local player_list: { [number]: string } = {}
```
- **Type Assertions (`::`)**:
```luau
local entity = get_target() :: any
local cast_num = raw_val :: number
```
- **Type Aliases (`type` / `export type`)**:
```luau
type ResolverRecord = {
    yaw: number,
    pitch: number,
    lby: number,
    choked: number,
}
export type AngleModifier = (number, number) -> number
```
- **Generics**:
```luau
local function get_or_default<T>(val: T?, fallback: T): T
    return if val ~= nil then val else fallback
end
```

### 1.2 Compound Assignment Operators
All standard Luau compound assignments are supported and desugared cleanly:
```luau
counter += 1      -- desugars to counter = counter + 1
health -= damage  -- desugars to health = health - damage
multiplier *= 1.5 -- desugars to multiplier = multiplier * 1.5
divider /= 2      -- desugars to divider = divider / 2
modVal %= 360     -- desugars to modVal = modVal % 360
powVal ^= 2       -- desugars to powVal = powVal ^ 2
str ..= "!"       -- desugars to str = str .. "!"
```

### 1.3 Flow Control: `continue`
The `continue` keyword is supported in all loop structures (`for`, `while`, `repeat until`):
```luau
for i = 1, 10 do
    if i % 2 == 0 then
        continue -- Skips even numbers, jumps to next iteration
    end
    print("Odd:", i)
end
```

### 1.4 Loop Budget Protection & Zero-Lag Architecture
A frequent issue in user scripts is heavy `while true do` loops causing severe frame drops or game freezes.
The LuaEngine includes an **automatic cooperative time-slicing loop budget**:
- **5ms Auto-Yield Budget**: Every 128 loop iterations, the interpreter inspects elapsed CPU time. If more than 5ms elapsed in the current frame, it automatically yields via `task.wait()`, returning control to Roblox's render pipeline and keeping game FPS at 60+.
- **Script Lifecycle Auto-Termination**: When a script is unloaded or reloaded via the menu, active background loops instantly terminate rather than lingering as orphan threads.

### 1.5 Asynchronous Task & Timer Library (`task`, `wait`, `delay`, `spawn`)
Full access to Roblox's modern `task` scheduler and legacy coroutine timers:

```luau
-- Yielding:
task.wait(0.5)               -- Yields execution for 0.5 seconds
wait(0.1)                    -- Alias for task.wait

-- Background Spawning:
task.spawn(function()
    while true do
        task.wait(1)
        print("Heartbeat tick:", tick())
    end
end)
spawn(function() ... end)    -- Alias for task.spawn

-- Delayed Execution:
local timer = task.delay(3, function()
    print("3 seconds elapsed!")
end)
delay(3, function() ... end) -- Alias for task.delay

-- Deferred & Cancellation:
task.defer(function() ... end)
task.cancel(timer)

-- Timing Primitives:
local now = tick()           -- Cheat engine synchronized tick time
local runTime = time()       -- Game session time
```

### 1.6 Luau Ternary Expressions & String Interpolation
- **Ternary `if-then-else` Expressions**:
```luau
local status = if entity.is_alive(0) then "ALIVE" else "DEAD"
local bonus = if level > 10 then 50 elseif level > 5 then 25 else 0
```
- **Backtick String Interpolation**:
```luau
local kills = 5
local msg = `Player has {kills} kills and {kills * 100} points!`
```

### 1.7 Native Roblox Instances & Custom GUI System
Scripts have full authority to create and manipulate real Roblox instances, custom ScreenGuis, and widgets without being confined to the Drawing library.

#### Supported Globals & Constructors:
- **`Instance.new(className, [parent])`**: Supports `"ScreenGui"`, `"Frame"`, `"TextLabel"`, `"TextButton"`, `"TextBox"`, `"ImageLabel"`, `"ImageButton"`, `"UIStroke"`, `"UICorner"`, `"UIGradient"`, `"UIPadding"`, `"UIListLayout"`, `"Highlight"`, `"BillboardGui"`, `"Part"`, etc.
- **Engine Services (`game`)**:
  - `game:GetService("Players")`, `game:GetService("RunService")`, `game:GetService("UserInputService")`, `game:GetService("TweenService")`, etc.
  - Safe CoreGui fallback: Setting `Parent = game:GetService("CoreGui")` automatically redirects safely to `LocalPlayer.PlayerGui` to avoid permission errors!
  - `game:GetService("HttpService")`: Fully proxied! Transparently routes `:GetAsync()`, `:PostAsync()`, and `:RequestAsync()` through the rate-limited server proxy while `:JSONEncode()` and `:JSONDecode()` run locally with native speed.
  - `game:GetService("TextChatService")`: Enhanced with direct `:SendAsync(msg)` / `:Chat(msg)` methods and automatic channel routing (`AllChat`, `Team_T`, `Team_CT`, `DeadChat`).
- **Data Types**: `UDim`, `UDim2`, `Vector2`, `Vector3`, `CFrame`, `Color3`, `ColorSequence`, `NumberSequence`, `NumberRange`, `TweenInfo`, `RaycastParams`, `Font`, `Enum`, `buffer`, `utf8`.

#### Automatic Lifecycle Cleanup & Zero Leakage:
When a script is unloaded or hot-reloaded:
1. **Instances**: Every Instance created via `Instance.new` is automatically `:Destroy()`-ed.
2. **Connections**: All events connected to `RunService`, `UserInputService`, or GuiButtons are automatically disconnected.
3. **Tasks**: Any active `task.spawn` or `task.delay` threads are automatically cancelled via `task.cancel`.

```luau
-- Complete Custom GUI Example:
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CustomWatermark"
screenGui.Parent = game:GetService("CoreGui") -- Auto-redirects to PlayerGui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 40)
frame.Position = UDim2.new(0.5, -100, 0, 20)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
frame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = frame

local label = Instance.new("TextLabel")
label.Size = UDim2.new(1, 0, 1, 0)
label.BackgroundTransparency = 1
label.TextColor3 = Color3.fromRGB(255, 255, 255)
label.Font = Enum.Font.GothamBold
label.Text = "penablox"
label.Parent = frame

-- Live update on cheat paint callback:
client.set_event_callback("paint", function()
    local target = rage.get_target()
    label.Text = if target then `Target: {target.name} [{target.damage} dmg]` else "Target: None"
end)
```

### 1.8 Security Blocklist & JIT Compiler Optimizations

The scripting engine includes a dedicated **Smart Security Analyzer** and **Multi-Layer Runtime Sandbox Guard** combined with an advanced **JIT Optimization Pipeline** inspired by `syn-obfuscator`:

#### 1. Security Blocklist (Network & Remote Isolation)
To guarantee security and prevent unauthorized remote exploitation:
- **Blocked Remote Methods**:
  - `:FireServer(...)`, `:InvokeServer(...)`, `:FireClient(...)`, `:FireAllClients(...)`.
  - Blocked both case-sensitively and across lowercase variants (`fireServer`, `invokeServer`).
- **Blocked Remote & Network Classes**:
  - `RemoteEvent`, `RemoteFunction`, `BindableEvent`, `BindableFunction`, `UnreliableRemoteEvent`.
  - Any attempt to instantiate via `Instance.new("RemoteEvent")` is rejected.
- **Blocked Dangerous Services**:
  - `NetworkClient`, `NetworkServer`, `ScriptContext`, `MessageBusService`, `HttpRbxApiService`.
- **Clean Violation Output**:
  - `lua violation: blocked operation '<operation>'`
  - `lua violation: blocked class '<class>'`
  - `lua violation: blocked service '<service>'`
- **Multi-Layer Defense**:
  - **Static Token & AST Analysis**: Before compilation or execution, `SecurityAnalyzer.luau` scans source tokens and AST nodes.
  - **Runtime Proxy Metatable Guard**: Even if a script is heavily obfuscated (e.g. via `syn-obfuscator` with string encryption), all instances obtained via `game`, `workspace`, or `Instance.new` are wrapped in `createSecureInstanceProxy`. Any runtime attempt to index or call `:FireServer()` or `:InvokeServer()` is immediately intercepted and aborted with `lua violation: blocked operation '...'`.

#### 2. JIT & AST Optimization Pipeline
- **Loop JIT Unrolling**: Numeric loops with constant bounds (1 to 4 iterations) and no control flow interruptions are unrolled into sequential statement blocks, eliminating loop dispatch, yield timers, and condition evaluation.
- **Constant Propagation**: Immutable local variables initialized with literals are tracked and substituted directly into subsequent expressions.
- **Math Library Constant Folding & Inlining**: Expressions like `math.abs(-10)`, `math.sqrt(16)`, `math.rad(180)`, `math.min(5, 8)`, `math.clamp(x, min, max)`, `math.pi`, and `math.huge` are computed at compile time with zero runtime cost.
- **Algebraic Simplification**: Identities like `x + 0`, `x * 1`, `x - 0`, and `x / 1` are reduced to `x`.
- **Dead Code Elimination (DCE)**: Branches with constant conditions (`if true then ...`, `while false do ...`) are pruned before execution.
- **Immediate Local Variable Fastpath**: Variables declared in the local function scope bypass linked-list scope traversal and read/write directly from `scope.Vars`.

---

## 2. UI & Menu System (`menu` / `ui`)

### Creating Custom Tabs & Elements
Custom tabs appear in the SynUI cheat window alongside built-in categories.

```lua
local my_tab = menu.new_tab("HVH Lua", "rbxassetid://10734975692")
local left_sec = my_tab:new_section("Resolver Settings", "Left")  -- "Left" or "Right"
```

#### Section Element Creators
| Method | Description | Return |
| :--- | :--- | :--- |
| `sec:new_toggle(name, [default], [callback])` | Checkbox toggle | `ElementHandle` |
| `sec:new_slider(name, min, max, [default], [decimals], [suffix], [callback])` | Numeric slider | `ElementHandle` |
| `sec:new_combobox(name, options, [default], [callback])` | Dropdown selector | `ElementHandle` |
| `sec:new_keybind(name, [mode], [key], [callback])` | Hotkey (`"Toggle"`, `"Hold"`, `"Always"`) | `ElementHandle` |
| `sec:new_colorpicker(name, [default], [callback])` | RGBA color picker | `ElementHandle` |
| `sec:new_textbox(name, [placeholder], [default], [callback])` | String input field | `ElementHandle` |
| `sec:new_button(name, callback)` | Clickable push button | `ElementHandle` |
| `sec:new_label(text)` | Static text label | `ElementHandle` |

#### Element Handle Methods
```lua
handle:get()                       -- Returns current element value (bool, number, string, Color3)
handle:set(value, [silent])        -- Updates element value (silent skips callbacks)
handle:set_visible(boolean)        -- Shows or hides element in the UI
handle:set_callback(function)      -- Attaches change listener
```

---

### Gamesense UI Emulation (`ui.*`)
Full drop-in compatibility with Gamesense scripts:

```lua
-- Referencing built-in cheat toggles and sliders by flag name, path, or tab/section/name:
local pitch_ref = ui.reference("AntiAim_Pitch") -- direct flag lookup
local pitch_mode, pitch_slider = ui.reference("AA", "Anti-aimbot angles", "Pitch") -- 3-arg lookup
local dt_toggle = ui.reference("RAGE > Aimbot > Double tap") -- path string lookup

-- Live getter and setter via object or static methods:
pitch_slider:set(-89)
ui.set(pitch_slider, -89)

local pitch_val = pitch_slider:get()
local is_dt = ui.get(dt_toggle)
```

#### Element Inspection & Discovery (`ui.list`)
Never guess flag or container names again. `ui.list()` enumerates every registered menu element:
```luau
local elements = ui.list()
for i, path in ipairs(elements) do
    print(i, path) -- e.g. "AA > Anti-aimbot angles > Pitch [AntiAim_Pitch]"
end

-- Inspect internal tables directly:
local flags_map = elements.flags       -- { [flag_name] = element }
local paths_map = elements.paths       -- { ["Tab > Sec > Name"] = element }
local raw_items = elements.elements    -- raw array of SynUI elements
```

#### Flexible Reference Formats (`ui.reference`)
`ui.reference(...)` automatically handles all common referencing styles:
1. **Single Argument — Flag Name**: `ui.reference("AntiAim_Pitch")`, `ui.reference("Combat_Aimbot_Enabled")`.
2. **Single Argument — Full Path**: `ui.reference("AA > Anti-aimbot angles > Pitch")`.
3. **Multi-Argument**: `ui.reference(tab, container, name)`.
4. **Tab & Container Aliases**:
   - `"aa"` / `"antiaim"` -> `"AA"` / `"Anti-Aim"`
   - `"rage"` / `"combat"` / `"main"` -> `"Combat"` / `"Main"`
   - `"vis"` / `"esp"` -> `"Visuals"`
   - `"misc"` -> `"Miscellaneous"`
   - `"config"` / `"settings"` -> `"Settings"`

Returned references provide live interaction methods:
- `ref:get()` / `ui.get(ref)`: Reads the live value from menu state.
- `ref:set(val)` / `ui.set(ref, val)`: Updates the value and triggers registered callbacks.
- `ref:set_visible(boolean)`: Toggles visibility in the menu.
- `ref:set_callback(function)`: Attaches a change listener callback.

#### Supported UI Constructors & Helpers:
- `ui.list() -> { [number]: string, flags: table, paths: table, elements: table }`
- `ui.reference(tabOrFlag, [container], [name]) -> ElementHandle, ElementHandle?`
- `ui.new_checkbox(tab, section, name, [default]) -> ElementHandle`
- `ui.new_slider(tab, section, name, min, max, [default], [step], [unit], [scale]) -> ElementHandle`
- `ui.new_combobox(tab, section, name, options...) -> ElementHandle`
- `ui.new_multiselect(tab, section, name, options...) -> ElementHandle`
- `ui.new_color_picker(tab, section, name, r, g, b, [a]) -> ElementHandle`
- `ui.new_hotkey(tab, section, name, [inline], [key]) -> ElementHandle`
- `ui.new_button(tab, section, name, callback) -> ElementHandle`
- `ui.get(ref)` / `ui.set(ref, ...)`
- `ui.is_menu_open() -> boolean`
- `ui.menu_position() -> (number, number)`

---

## 3. Anti-Aim & Desync (`aa`)

The `aa` library provides complete authority over local camera pitch, yaw, custom mathematical modifiers, desync body angles up to **86°**, and fake lag choking.

### Pitch
- `aa.override_pitch(pitch)` / `aa.set_pitch(pitch)`: Overrides pitch angle. Accepts:
  - Numbers: `[-89..89]` degrees (e.g. `-89` down, `89` up, `0` zero).
  - Strings: `"Down"`, `"Up"`, `"Zero"`, `"Disabled"`.
  - Pass `nil` to restore menu control.
- `aa.get_pitch() -> number?`: Returns active pitch override.
- `aa.is_pitch_overridden() -> boolean`
- `aa.override_fake_pitch(val)` / `aa.set_fake_pitch(val)`: Overrides fake pitch. Accepts numbers `[-70..70]` degrees or presets (`"Down"`, `"Up"`, `"Zero"`, `"Disabled"`). Pass `nil` to restore menu control.
- `aa.get_fake_pitch() -> number?`: Returns active fake pitch override in degrees.
- `aa.is_fake_pitch_overridden() -> boolean`
- `aa.override_fake_pitch_angle(deg)` / `aa.set_fake_pitch_angle(deg)`: Overrides the fake pitch angle offset slider `[-20..20]` degrees. Pass `nil` to restore menu slider.
- `aa.get_fake_pitch_angle() -> number`: Returns the active fake pitch offset angle.
- `aa.is_fake_pitch_angle_overridden() -> boolean`

### Extended Angles (Body Lean / Roll Offsets)
- `aa.override_extended_angles(pct)` / `aa.set_extended_angles(pct)`: Overrides extended angles slider percentage `[-100..100%]`. Pass `nil` to restore menu slider.
- `aa.get_extended_angles() -> number`: Returns current extended angles slider percentage.
- `aa.is_extended_angles_overridden() -> boolean`: Returns whether extended angles percentage is currently overridden.
- `aa.is_extended_angles() -> boolean`: Returns `true` if extended angles are currently non-zero.
- `aa.set_extended_angles_enabled(enabled)`: Configures default state for extended angles in menu configuration.
- `aa.override_extended_angles_enabled(enabled)`: Overrides whether extended angles are enabled or disabled. Pass `nil` to restore.
- `aa.get_extended_angles_enabled() -> boolean`: Returns whether extended angles are currently enabled.

### Yaw & Modifiers
- `aa.override_yaw(deg, [base])`: Sets base yaw in degrees `[-180..180]`.
  - `base`: `"camera"` (relative to view) or `"world"` (absolute world axis).
- `aa.override_yaw_offset(deg, [relativeToSlider])`: Sets static angle offset added to base yaw.
  - If `relativeToSlider` is `true`: adds `deg` on top of the menu slider offset (`slider + deg`).
  - If `relativeToSlider` is omitted or `false`: overrides the offset angle directly.
- `aa.add_yaw_offset(deg)`: Convenience helper that adds `deg` to the current menu slider offset (`get_slider_yaw_offset() + deg`).
- `aa.get_slider_yaw_offset() -> number`: Returns the menu's active Yaw offset slider value (e.g. `180` for backwards AA). Aliases: `aa.get_menu_yaw_offset()`, `aa.get_base_yaw_offset()`, `aa.get_yaw_slider()`.
- `aa.is_manual() -> boolean`: Returns whether manual anti-aim is currently active (`"Left"`, `"Right"`, or `"Back"`).
- `aa.get_manual() -> string?`: Returns active manual direction (`"Left"`, `"Right"`, `"Back"`, or `nil`).
- `aa.set_manual(dir)`: Sets active manual direction (`"Left"`, `"Right"`, `"Back"`, or `nil` to clear).
- `aa.override_yaw_modifier(modifier, [offset])`: Custom jitter, spin, or mathematical sway:
  - **Function**: `function(tick, baseYaw) -> number (offset in degrees)`
  - **Presets**: `"Spin"`, `"Random"`, `"Center"`, `"3-Way"`, `"5-Way"`, `"Disabled"`.
- `aa.get_yaw() -> number?` / `aa.get_yaw_offset() -> number?`

### Body Yaw & Desync (Unlocked up to 86°)
- `aa.override_body_yaw(enabled)`: Enables or disables desync simulation.
- `aa.override_inverter(inverted)`: Sets desync side (`false` = Fake Left / Real Right, `true` = Fake Right / Real Left).
- `aa.override_desync_limits(leftDeg, rightDeg)`: Sets max desync rotation limits `[0..86]` degrees.
- `aa.override_desync(realDeg, fakeDeg)`: Explicitly sets Real and Fake body angles `[-86..86]`.
- `aa.override_show_fake(boolean)`: Forces thirdperson ghost visualizer on or off.
- `aa.get_inverter() -> boolean`
- `aa.get_desync_limits() -> (number, number)`: Returns `left, right` limits (up to `86, 86`).
- `aa.get_lby() -> number` / `aa.get_lby_deg() -> number`: Local Real Feet Yaw (radians/degrees).
- `aa.get_fake_lby_deg() -> number`: Local Fake Feet Yaw in degrees.
- `aa.get_twist_deg() -> number`: Delta angle between real body yaw and feet yaw.

### Fake Lag & Choke Manipulation
- `aa.override_fake_lag(enabled)`: Enables/disables packet choking.
- `aa.override_fake_lag_limit(ticks)`: Sets choke limit between `0` and `14` ticks.
- `aa.override_fake_lag_variability(pct)`: Sets choke randomization percentage `[0..100%]`.
- `aa.is_fake_lag_choking() -> boolean`: `true` if current simulation tick is choked.
- `aa.get_choke_target() -> number`: Effective target tick count for the current choke sequence.
- `aa.unchoke()` / `aa.flush_choked()`: Instantly flushes all queued network packets to create teleport/burst effect.
> Note: Use `globals.choke_ticks()` (or global `choke_ticks()`) to read the active number of choked ticks.
- `aa.force_teleport()`: Flushes queued packets to instantly shift simulation origin and break lag compensation.

### Building Custom Defensive Anti-Aim in Lua
Instead of rigid hardcoded engine options, developers can build completely custom defensive anti-aims by combining low-level primitives:
- `rage.shift_ticks([ticks], [trigger_second_shot]) -> boolean` / `rage.force_shift(...)`: Shifts simulation ticks (DoubleTap tickbase shift) when charged (`>= 12` ticks or `>= ticks`). Returns `true` if the shift executed.
- `rage.is_double_tap_active() -> boolean` / `rage.is_dt_active()`: Returns `true` if Double Tap is enabled and active.
- `rage.is_double_tap_charged() -> boolean` / `rage.is_dt_charged()`: Returns whether DT tickbase is fully charged (`>= 14` ticks).
- `rage.get_double_tap_charge() -> number` / `rage.get_dt_charge()`: Returns charged tick count `[0..14]`.
- `rage.is_double_tap_charging() -> boolean` / `rage.is_dt_charging()`: Returns `true` while charging DT ticks.
- `rage.is_double_tap_shifting() -> boolean` / `rage.is_dt_shifting()`: Returns `true` during active DT shift execution.
- `rage.get_double_tap_cooldown() -> number` / `rage.get_dt_cooldown()`: Returns remaining recharge cooldown in ticks.
- `aa.override_pitch(pitch)`: Rapidly flicks pitch (`-89`, `89`, `"Random"`).
- `aa.override_yaw_offset(deg)` / `aa.override_yaw_modifier(fn)`: Rapidly rotates, jitters or spins head angles during choked bursts or on keybind.
- `aa.unchoke()`: Releases burst unchoke when desired.
- When an enemy aimbot/resolver observes a rapid angle flick (`> 120°` in 1 frame) or broken simulation ticks, their resolver gets thrown off (`defensive_broken`), missing your head!

### Anti-Aim State & Synchronization (`aa.get_info`)
Inspect local anti-aim state, desync angles, choke counters, and authoritative server positions in real time:

```luau
local info = aa.get_info()
print(string.format("Real: %.1f° | Fake: %.1f° | Side: %s | Choked: %d | Defensive: %s",
    info.real_yaw,
    info.fake_yaw,
    info.desync_side > 0 and "Right" or "Left",
    info.choked_ticks,
    tostring(info.defensive)
))

-- Check whether head is currently exposed to server line of sight:
local server_head: Vector3 = info.server_head_pos
local fraction, hit_ent = trace.line(server_head, enemy_eye_pos)
if fraction < 1.0 then
    -- Head is occluded behind cover on the server!
end
```

| Field / Index | Type | Description |
| :--- | :--- | :--- |
| `info.real_yaw` / `[1]` | `number` | Authoritative real feet yaw in degrees. |
| `info.fake_yaw` / `[2]` | `number` | Client fake (desync) yaw rendered to other players. |
| `info.desync_side` / `[3]` | `number` | Active desync direction (`1` = Fake Right / Real Left, `-1` = Fake Left / Real Right). |
| `info.choked_ticks` / `[4]` | `number` | Number of choked commands in the active network cycle `[0..14]`. |
| `info.defensive` / `[5]` | `boolean` | `true` if defensive DoubleTap tickbase shift is currently engaged. |
| `info.server_head_pos` / `[6]` | `Vector3` | Exact head hitbox position as validated by the server (essential for safe peeking). |

### Reset
- `aa.reset()` / `aa.clear()`: Clears all anti-aim overrides and restores menu settings.

---

## 4. Ragebot & Resolver (`rage` / `plist`)

### Built-in Baseline Resolver Control
Control the native engine resolver directly from Lua:

| Function | Description | Values / Defaults |
| :--- | :--- | :--- |
| `rage.set_resolver_enabled(boolean)` | Enables or disables built-in baseline resolver | `true` / `false` |
| `rage.is_resolver_enabled() -> boolean` | Checks if built-in resolver is active | Returns `boolean` |
| `rage.set_resolver_mode(mode)` | Sets resolver algorithm | `"Brute-force"`, `"Adaptive"`, `"Freestanding"` |
| `rage.get_resolver_mode() -> string` | Returns active resolver mode | Returns `string` |
| `rage.set_resolver_miss_limit(n)` | Shots before angle cycles | `1` to `5` (default `2`) |
| `rage.get_resolver_miss_limit() -> number` | Returns current miss limit | Returns `number` |
| `rage.set_resolver_baim_after_misses(n)` | Auto body-aim threshold | `0` to `5` (default `2`, `0` = off) |
| `rage.get_resolver_baim_after_misses() -> number` | Returns BAim threshold | Returns `number` |
| `rage.reset_resolver_state([target])` | Resets miss counter and cycle index | Optional `target` slot |
| `rage.get_resolver_info(target) -> table` | Returns target resolver status | `{ side = -1\|0\|1, mode = string, misses = number }` |
| `rage.get_miss_reasons() -> { string }` | Returns all possible miss reason codes | `{ "spread", "resolver", "occlusion", "death" }` |

---

### Player List Overrides (`plist`)
Provides 100% authoritative priority over any enemy's resolved yaw, hitbox angles, and targeting rules.

```lua
plist.set(target, key, value)
plist.get(target, key) -> any
plist.reset(target)
plist.reset_all()

local all_keys = plist.get_keys() -- returns array of all supported keys
```

#### Supported `plist` Keys:
- `"Force body yaw"` (`boolean`): Enables manual resolver angle override for this player.
- `"Force body yaw value"` / `"Override yaw"` (`number`): Sets target's feet yaw to an absolute angle in degrees `[-180..180]`.
- `"Override yaw offset"` (`number`): Sets an angle offset relative to enemy visual angle (e.g. `58`, `-58`, `0`, `86`, `-86`).
- `"Force body yaw absolute"` (`number`): Absolute yaw angle without relative modifiers.
- `"Override resolver"` (`boolean`): Toggle baseline resolver calculation specifically for this player.
- `"Override prefer body aim"` / `"Force body aim"` (`string` | `boolean`): `"Force"` / `true` forces BAim; `"-"` / `false` restores normal scan.
- `"Override safe point"` / `"Force safe point"` (`string` | `boolean`): `"Force"` / `true` requires safe points.
- `"Force pitch"` (`boolean` | `number`): Enables manual pitch override or directly sets pitch angle.
- `"Force pitch value"` / `"Override pitch"` (`number` | `string`): Overrides enemy pitch. Supports numbers (degrees or radians) and presets (`"Down"`, `"Up"`, `"Zero"`).
- `"Override min damage"` / `"Min dmg"` (`number`): Overrides minimum damage threshold specifically for this player.
- `"Override hitboxes"` (`{ string }`): Overrides targeted hitboxes specifically for this player.
- `"High priority"` (`boolean`): Forces ragebot to prioritize this enemy above all others.
- `"Whitelist"` (`boolean`): Ignores this player completely (ragebot will never shoot at them).

#### Key Enumeration Helper:
- `plist.get_keys() -> { string }` / `plist.keys`: Returns the complete table of supported plist keys.

#### Quick Helper Functions:
- `rage.override_player_yaw(target, deg)` / `rage.set_player_yaw(target, deg)`: Forces absolute yaw on target.
- `rage.override_player_yaw_offset(target, offsetDeg)` / `rage.set_player_yaw_offset(target, offsetDeg)`: Offsets target yaw (e.g. `58` or `-58`).
- `rage.override_player_pitch(target, deg)` / `rage.set_player_pitch(target, deg)`: Overrides target pitch.
- `rage.get_player_pitch(target) -> number?`: Returns active forced pitch override for target, or `nil` if not overridden.
- `rage.is_player_pitch_overridden(target) -> boolean`: Checks if target has a forced pitch active.
- `rage.reset_player_pitch(target)`: Restores target pitch to normal.
- `rage.reset_player(target)` / `rage.reset_resolver(target)`: Restores target angles (both yaw and pitch) to default.

---

### World Raytracing & Line of Sight (`trace.line` / `client.trace_line`)
Zero-allocation physical world trace designed specifically for freestanding anti-aim, safe peek checks, and cover verification:

```luau
local fraction, hit_ent, hit_pos, normal = trace.line(eye_pos, target_pos)
if fraction < 1.0 then
    print("Wall encountered at:", hit_pos, "Normal:", normal)
else
    print("Direct unobstructed line of sight to target!")
end
```
- `trace.line(from, to) -> (number, any?, Vector3, Vector3)`: Automatically excludes local player character and local collider box.
  - `fraction`: `0.0` (blocked at start) to `1.0` (fully clear trajectory).
  - `hit_ent`: `Player` instance or slot ID if a player or collider was intersected, or `nil`.
  - `hit_pos`: World position of the collision point.
  - `normal`: Surface normal unit vector.
- Aliases: `client.trace_line(from, to)`, `trace.trace_line(from, to)`.

---

### Global Ragebot Overrides & Ballistics (`rage`)
- `rage.override_enabled(boolean)` / `rage.is_enabled() -> boolean`
- `rage.override_min_damage(damage)` / `rage.get_min_damage() -> number`
- `rage.override_hitchance(hitchance)` / `rage.get_hitchance() -> number`
- `rage.get_hitchance(player, hitbox) -> number`: Calculates expected hitchance `[0..100%]` for a specific hitbox before pulling the trigger, taking into account weapon spread, movement inaccuracy, and distance.
- `rage.get_damage(from_pos, player, hitbox) -> (number, boolean, number)`: Simulates bullet penetration through world geometry to the target's hitbox using the authoritative `PenetrationEngine`.
  - Returns: `damage` (estimated damage after penetration and armor), `is_wallbang` (`true` if bullet penetrated walls), `walls_pierced` (number of surfaces penetrated).
  - Perfect for filtering out low-damage wallbangs:
  ```luau
  local dmg, is_wb, walls = rage.get_damage(client.eye_position(), target, "head")
  if is_wb and dmg < 15 then
      -- Skip weak 2-8 dmg wallbang, wait for a lethal shot!
  end
  ```
- `rage.override_force_baim(boolean)` / `rage.is_force_baim() -> boolean`
- `rage.override_force_safe_point(boolean)` / `rage.is_force_safe_point() -> boolean`
- `rage.override_target_selection(mode)`: `"Highest damage"`, `"Near crosshair"`, `"Best hitchance"`, `"Cycle"`.
- `rage.override_hitboxes(list)`: List of parts: `{ "head", "chest", "pelvis", "arms", "legs" }`.
- `rage.override_hitbox_priority(list)`: Order of evaluation: `{ "pelvis", "chest", "head" }`.
- `rage.override_hitbox_min_damage(hitbox, dmg)`: Per-part min damage requirement.
- `rage.override_hitbox_hitchance(hitbox, hc)`: Per-part hitchance requirement.
- `rage.set_hitbox_config(hitbox, { min_damage = 80, hitchance = 65, multipoint = true })`
- `rage.get_target() -> TargetInfo?`: Active target snapshot (`slot`, `player`, `part_name`, `aim_point`, `damage`, `hitchance`, `resolved_yaw_deg`).
- `rage.get_target_info() -> PlayerInfoTable?`: Full runtime status of current target.
- `rage.reset()` / `rage.clear()`: Restores menu rage settings.

---

### Aim & Combat Event Callbacks
Listen to shots, hit confirmations, and game lifecycle events using `client.set_event_callback` or `events.on`:

```lua
-- 1. Shot fired by ragebot
client.set_event_callback("aim_fire", function(e)
    -- e.id: unique sequential shot ID
    -- e.target: target player slot number (use with entity.get_player or entity.get_player_name)
    -- e.player: target Player instance (if available)
    -- e.hitbox: targeted hitbox name ("head", "chest", etc.)
    -- e.damage: simulated predicted damage
    -- e.hitchance: calculated hitchance percentage [0..100]
    -- e.resolved_yaw: target yaw angle used for the shot
    -- e.point: Vector3 world coordinates aimed at
end)

-- 2. Authoritative server hit confirmation
client.set_event_callback("aim_hit", function(e)
    -- e.id: exact matching shot ID from aim_fire!
    -- e.target: target player slot number
    -- e.hitbox: hitbox confirmed by server
    -- e.damage: real damage inflicted
    -- e.health_remaining: victim health after impact
    -- e.hitchance: hitchance of original shot
end)

-- 3. Shot missed
client.set_event_callback("aim_miss", function(e)
    -- e.id: exact matching shot ID from aim_fire!
    -- e.target: target player slot number
    -- e.hitbox: targeted hitbox
    -- e.reason: "resolver" | "spread" | "occlusion" | "death"
    -- e.hitchance: hitchance of original shot
end)

-- 4. Game Round Events
client.set_event_callback("round_start", function(e)
    print("Round started! Round number:", e.round)
end)

client.set_event_callback("round_end", function(e)
    print("Round ended! Winner:", e.winner, "Reason:", e.reason)
end)

-- 5. Player Spawn & Death Events
client.set_event_callback("player_spawn", function(e)
    print("Player spawned in slot:", e.slot, "Player:", e.player)
end)

client.set_event_callback("player_death", function(e)
    print("Kill confirmed! Victim:", e.victim, "Attacker:", e.attacker, "Weapon:", e.weapon, "Headshot:", e.headshot)
end)
```

---

## 5. Entities & Players (`entity` / `client`)

### Consolidated Player Snapshots
Instead of individual queries, use high-level snapshot functions:

```lua
-- Full player status in one zero-allocation call:
local info = entity.get_info(target)
if info and info.is_alive then
    print(info.name, info.health, info.state)  -- "Stand", "Run", "Duck", "Jumping"
    print(info.speed, info.position, info.velocity)
    print(info.yaw, info.pitch, info.lby, info.desync_delta)
    print(info.weapon_name, info.is_crouching, info.is_in_air)
end

-- Gamesense Animstate snapshot (ideal for resolvers):
local anim = entity.get_anim_state(target)
if anim then
    print(anim.eye_angles_y)                 -- Target eye yaw in degrees
    print(anim.torso_yaw)                    -- Procedural torso yaw
    print(anim.goal_feet_yaw)                -- Target LBY / Feet Yaw
    print(anim.duck_amount)                  -- 1.0 if ducking, 0.0 otherwise
    print(anim.feet_speed_forwards_or_sideways)
    print(anim.desync)                       -- Evaluated desync delta in degrees
end
```

### Enumeration & Queries
- `entity.get_enemies() -> { Player }`
- `entity.get_teammates() -> { Player }`
- `entity.get_players([enemiesOnly]) -> { Player }`
- `entity.get_local_player() -> Player`
- `entity.get_player_slot(target) -> number?` / `entity.get_slot(target) -> number?`: Resolves player slot index `[0..255]`. Accepts `Player` instance, character `Model`, slot number, or character name string.
- `entity.is_alive(target) -> boolean`
- `entity.is_enemy(target) -> boolean`
- `entity.get_player_health(target) -> number`
- `entity.get_player_armor(target) -> number`
- `entity.get_player_position(target) -> Vector3`
- `entity.get_player_velocity(target) -> Vector3`
- `entity.get_player_speed(target) -> number`
- `entity.get_player_angles_deg(target) -> (pitch, yaw)`
- `entity.get_player_hitbox_position(target, hitbox) -> Vector3?`: Returns 3D position of target hitbox. Hitboxes: `"head"`, `"chest"`, `"pelvis"`, `"right_arm"`, `"left_arm"`, `"right_leg"`, `"left_leg"`. Works seamlessly on enemies AND the local player (`entity.get_local_player()`), automatically resolving custom `VisualSkin` attachments and procedural R6 rig offsets.
- `entity.get_player_weapon_name(target) -> string`
- `entity.is_player_scoped(target) -> boolean`
- `entity.get_player_lby_deg(target) -> number`
- `entity.get_player_twist_deg(target) -> number`

### Direct Property Queries (`entity.get_prop`)
For low-level Gamesense / Neverlose script compatibility, `entity.get_prop` queries raw or resolved animstate/entity fields:

```lua
local val = entity.get_prop(target, propertyName, [arrayIndex])
```

#### Supported Properties:
- `"eye_angles_y"`, `"m_flEyeAngles[1]"`: Enemy visual eye yaw in degrees.
- `"eye_angles_x"`, `"m_flEyeAngles[0]"`: Enemy visual eye pitch in degrees.
- `"resolved_yaw"`, `"m_angEyeAngles_resolved[1]"`: Authoritative resolved feet/eye yaw in degrees.
- `"resolved_pitch"`, `"m_angEyeAngles_resolved[0]"`: Authoritative resolved pitch in degrees (accounts for `force pitch` / resolver overrides).
- `"torso_yaw"`, `"m_flTorsoYaw"`: Procedural torso yaw.
- `"goal_feet_yaw"`, `"lby"`, `"m_flLowerBodyYawTarget"`: Lower body yaw in degrees.
- `"duck_amount"`, `"m_flDuckAmount"`: `1.0` if crouching/ducking, `0.0` otherwise.
- `"m_vecOrigin"`, `"origin"`: Returns `x, y, z` world coordinates.
- `"m_vecVelocity"`, `"velocity"`: Returns `vx, vy, vz` velocity vector.
- `"m_iHealth"`, `"health"`: Player health `[0..100]`.
- `"m_ArmorValue"`, `"armor"`: Player armor `[0..100]`.

### Local Weapon & Camera (`client`)
```lua
local wep = client.get_weapon()
print(wep.id, wep.name, wep.type)      -- e.g. "Scout", "SSG 08", "SniperRifle"
print(wep.ammo, wep.reserve)            -- Clip and reserve ammo
print(wep.scope_level, wep.is_scoped)   -- Scope magnification level (0, 1, 2)

local pitch, yaw = client.get_pitch_yaw()
local pDeg, yDeg, roll = client.get_camera_angles_deg()
local pingSec = client.latency()

-- Query live match and server state:
local srv = client.get_server_info()
print(string.format("Tickrate: %d | Map: %s | NoSpread: %s | State: %s",
    srv.tickrate, srv.map, tostring(srv.nospread), srv.state))
```

#### Server Information (`client.get_server_info`)
Returns metadata about the active server instance and competitive rule settings:
- `srv.tickrate`: Authoritative server simulation tickrate (`64`).
- `srv.spread`: `true` if traditional weapon spread calculations are active.
- `srv.nospread`: `true` if server is running in Seeded NoSpread mode (via vote or modifier).
- `srv.map`: Current map name.
- `srv.state`: Active match phase (`"Warmup"`, `"InProgress"`, `"RoundEnded"`, `"MatchEnded"`).

---

## 6. 2D Canvas & Drawing (`render` / `Drawing`)

Hardware-accelerated, resolution-independent screen-space drawing system with zero-allocation object pooling, hierarchical parenting, and built-in cursor hit-testing. Designed for creating completely custom, standalone graphical interfaces (GUIs), HUDs, and overlays.

### Helper Constructors
```lua
local v2 = vector(100, 200)         -- Vector2
local v3 = vector(10, 20, 30)       -- Vector3
local c1 = color(255, 60, 70)       -- Color3 from RGB
local c2 = color("#00FFA0")         -- Color3 from HEX
```

### Direct Render Functions (`render.*` / `renderer.*`)
- `render.text(x, y, text, [color], [size], [outline], [center], [font], [zIndex]) -> TextDrawing`
- `render.rect(x, y, w, h, [color], [rounding], [thickness], [zIndex]) -> SquareDrawing`
- `render.rect_filled(x, y, w, h, [color], [rounding], [zIndex]) -> SquareDrawing`
- `render.gradient(x, y, w, h, col1, col2, [vertical], [rounding], [zIndex]) -> SquareDrawing`
- `render.line(x1, y1, x2, y2, [color], [thickness], [zIndex]) -> LineDrawing`
- `render.circle(x, y, radius, [color], [thickness], [zIndex]) -> CircleDrawing`
- `render.circle_filled(x, y, radius, [color], [zIndex]) -> CircleDrawing`
- `render.triangle(p1, p2, p3, [color], [filled], [thickness], [zIndex])`
- `render.image(x, y, w, h, assetId, [color], [rounding], [zIndex])`
- `render.indicator(r, g, b, a, text...) -> number`: Draws a stacked HUD indicator in the bottom-left corner.
- `render.measure_text(text, [size], [font]) -> Vector2`: Measures rendered pixel size.
- `render.screen_size() -> (w, h)`: Viewport dimensions in pixels.
- `render.world_to_screen(worldPos: Vector3) -> (x, y, onScreen)`: 3D to 2D projection.
- `render.clear()`: Recycles all drawings back to pool.

---

### Custom GUI & Drawing Objects (`Drawing.new`)
Supported primitives: `"Text"`, `"Square"`, `"Line"`, `"Circle"`, `"Triangle"`, `"Quad"`, `"Image"`.  
Fonts: `Drawing.Fonts.UI` (0), `System` (1), `Plex` (2), `Monospace` (3).

#### Advanced Object Properties & Hierarchical Parenting
All Drawing objects now support hierarchical parenting, allowing you to build fully nested custom GUIs (windows, panels, tabs, buttons, checkboxes, sliders) where children move and clip relative to their parent container:

| Property | Primitives | Description |
| :--- | :--- | :--- |
| `Parent` | All Primitives | Sets parent Drawing object or Roblox GuiObject. Child coordinates become relative to parent! |
| `Rounding` | `Square`, `Image` | Corner rounding radius in pixels via GPU hardware (`UICorner`). |
| `BorderColor` | `Square` | Independent stroke/outline color for filled rectangles. |
| `ClipsDescendants`| `Square` | Enables clipping so child elements inside scrollable/nested panels don't overflow. |
| `Gradient` | `Square` | Table `{ Color1, Color2, [Angle], [Transparency1], [Transparency2] }` for smooth color blends. |
| `ImageColor` | `Image` | Tint/multiply color for loaded image textures. |

#### Object Hit-Testing, Touch & Interactive Methods
Drawing objects (`Square`, `Circle`, `Text`, `Image`) provide a full suite of interactive methods with hardware-accurate hit-testing, single-click edge-detection, mobile touch support, and draggable window bindings:

```lua
-- Single-Click Edge Detection (triggers once per press, does NOT cycle when held down!)
local is_clk = my_btn:IsClicked([btn])        -- Alias for IsJustPressed; ideal for buttons and toggles
local is_prs = my_btn:IsJustPressed([btn])    -- Returns true only on the initial frame mouse/touch went down

-- Continuous Hold & Polling (for sliders, repeaters, or custom drag logic)
local is_hld = my_btn:IsDown([btn])           -- Returns true every frame mouse/touch is held inside bounds
local is_hld2 = my_btn:IsHeld([btn])          -- Alias for IsDown

-- Release Detection
local is_rel = my_btn:IsJustReleased([btn])   -- Returns true only on the frame mouse/touch was released

-- Hover & Bounding Queries
local is_hov = my_btn:IsHovered()             -- Returns true if mouse or active touch is inside bounds
local x, y, w, h = my_btn:GetBounds()         -- Absolute screen coordinates and dimensions
local abs_pos = my_btn:GetAbsolutePosition()  -- Vector2 absolute position
local abs_sz = my_btn:GetAbsoluteSize()       -- Vector2 absolute size

-- Event-Driven Callbacks (Zero polling required!)
local click_sub = my_btn:OnClick(function(pos, btn)
    print("Clicked at", pos)
end)
-- Controls: click_sub:Disconnect(), click_sub:remove()

my_btn:OnDown(function(pos, btn) ... end)     -- Fires on press down
my_btn:OnRelease(function(pos, btn) ... end)  -- Fires on release
my_btn:OnHover(function(pos) ... end)         -- Fires when cursor/touch enters
my_btn:OnHoverEnd(function(pos) ... end)      -- Fires when cursor/touch leaves

-- Smooth Window Dragging (Desktop Mouse & Mobile Touch Supported)
local drag = window:MakeDraggable(header_bar, function(new_pos)
    -- Optional callback whenever window is moved
end)
-- Options table format: window:MakeDraggable({ Handle = header, ClampToScreen = true, OnDrag = cb })
-- Controls: drag:SetEnabled(false), drag:Destroy()
```

---

## 7. Animation & Tweening Engine (`tween`)

`penablox RECODE` features a built-in, micro-optimized easing and physics tween engine. It operates with zero heap allocations per frame, running seamlessly on `RenderStepped`. All tweens created by a script are automatically tracked and safely cleaned up on script unload.

### Available Easing Curves
Supported styles: `"linear"`, `"sine"`, `"quad"`, `"cubic"`, `"quart"`, `"quint"`, `"expo"`, `"circ"`, `"back"`, `"elastic"`, `"bounce"`.  
Supported directions: `"in"`, `"out"`, `"inout"`.

### Tween Methods

#### 1. Instant Mathematical Easing
- `tween.ease(alpha, style, direction) -> number`: Returns evaluated eased curve factor `[0..1]`.
- `tween.interpolate(from, to, alpha, [style], [direction])`: Interpolates between two numbers, `Vector2`, `Vector3`, `Color3`, `UDim2`, or `UDim`.

#### 2. Frame-Independent Exponential Smoothing
- `tween.lerp(current, target, speed, dt)`: Smooth, frame-rate independent exponential decay:
```lua
-- Inside a paint / render stepped loop:
current_pos = tween.lerp(current_pos, target_pos, 12, dt)
```

#### 3. Harmonic Spring Physics
Unconditionally stable semi-implicit Euler harmonic springs for ultra-smooth UI animations (switches, tabs, toggles, popups):
- `tween.spring(current, target, velocity, damping, speed, dt) -> (newPos, newVel)`
- `tween.new_spring(initial, [target], [damping], [speed]) -> SpringInstance`:
```lua
local spr = tween.new_spring(0, 100, 0.75, 14)

-- In paint hook:
local cur = spr:Update(dt)
spr:SetTarget(200)
local settled = spr:IsSettled()
```

#### 4. Object Property Animation (`tween.create`)
Animates properties on Drawing objects, custom tables, or Roblox Instances:
```lua
local my_box = Drawing.new("Square")
my_box.Size = vector(100, 40)
my_box.Position = vector(50, 50)
my_box.Color = color(40, 40, 45)

-- Smoothly slide and expand the box:
local anim = tween.create(my_box, 0.35, {
    Position = vector(200, 50),
    Size = vector(160, 40),
    Color = color(70, 130, 240)
}, {
    style = "back",
    direction = "out",
    autoplay = true,
    on_complete = function()
        print("Animation completed!")
    end
})

-- Controls: anim:pause(), anim:play(), anim:cancel(), anim:reset(), anim:reverse()
```

---

## 8. Input & First-Person Cursor Unlocking (`input`)

### First-Person Cursor Unlocking
In Roblox first-person camera mode (`MouseBehavior.LockCenter`), player cursors are locked. The `input` library provides a seamless modal unlocking engine that frees the mouse across the entire screen so players can freely interact with custom Drawing GUIs:

- `input.unlock_cursor()`: Unlocks the mouse cursor, enabling free movement and GUI interaction.
- `input.lock_cursor()`: Restores native mouse lock for gameplay.
- `input.toggle_cursor() -> boolean`: Toggles cursor lock state, returning the new unlocked status (`true`/`false`).
- `input.is_cursor_unlocked() -> boolean`: Checks if the cursor is currently unlocked.
- `input.set_cursor_unlocked(boolean)`: Explicitly sets cursor unlock state.

### Input Polling, Touch & Hit-Testing
- `input.is_just_pressed([btn]) -> boolean`: Returns `true` only on the initial frame mouse/touch was pressed down (single-click detection).
- `input.is_held([btn]) -> boolean`: Returns `true` continuously while mouse/touch is held.
- `input.is_down([btn]) -> boolean`: Alias for `is_held`.
- `input.is_just_released([btn]) -> boolean`: Returns `true` only on the frame mouse/touch was released.
- `input.is_pointer_down([btn]) -> boolean`: Universal pointer check supporting both PC mouse buttons and mobile touch screen inputs.
- `input.is_touch() -> boolean`: Returns `true` if mobile touch screen input is currently active.
- `input.is_hovered(x, y, w, h) -> boolean`: Checks if the mouse cursor or any active touch is inside the specified rectangular region.
- `input.is_clicked(x, y, w, h, [btn]) -> boolean`: Single-click detection inside region (triggers once, does not cycle on hold).
- `input.get_mouse_delta() -> (dx, dy)`: Returns pointer movement delta between frames (works for both mouse drag and touch swipe).
- `input.get_pointer_delta() -> (dx, dy)`: Alias for `get_mouse_delta`.
- `input.get_mouse_position() -> (x, y)`: Screen cursor coordinates in pixels.
- `input.get_pointer_position() -> (x, y)`: Universal pointer coordinates (mouse or primary touch point).
- `input.make_draggable(target, [handleOrOpts], [callback])`: High-level draggable controller with automatic screen boundary clamping.
- `input.is_key_down(key) -> boolean`: Returns `true` if keyboard key (e.g. `"W"`, `"E"`, `"Insert"`) is held.
- `input.is_mouse_down(btn) -> boolean`: Returns `true` if mouse button (`1` = Left/Touch, `2` = Right, `3` = Middle) is held.
- `input.is_typing() -> boolean`: `true` if typing in Roblox chat or a focused UI textbox.
- `input.on_key_press(key, callback)`: Triggers on key press.

---

## 9. Events & Simulation Hooks (`events` / `client`)

### Time & Engine Counters (`globals`)
- `globals.curtime() -> number`: Server-synchronized game time in seconds.
- `globals.tickcount() -> number`: Authoritative simulation tick count.
- `globals.tickinterval() -> number`: Duration of one tick (`0.015625s`).
- `globals.realtime() -> number`: Local system clock (`os.clock()`).
- `globals.frametime() -> number`: Delta time of the last render frame.
- `globals.choke_ticks() -> number`: Current number of choked ticks buffered in the active FakeLag sequence (also exposed as global `choke_ticks()`).

### In-Memory Database (`database`)
Persists runtime data and settings across script reload sessions:
```lua
database.write("my_cfg", { angle = 58, mode = "adaptive" })
local cfg = database.read("my_cfg")
database.flush()
```

### Bitwise Operations (`bit` / `bit32`)
Optimized 64-bit integer bitwise operations:
`bit.band`, `bit.bor`, `bit.bxor`, `bit.bnot`, `bit.lshift`, `bit.rshift`.

### JSON Serialization (`json`)
- `json.stringify(table) -> string`
- `json.parse(jsonString) -> table`

### Console Variables (`cvar`)
- `cvar.<name>:get_float()`, `:get_int()`, `:get_string()`
- `cvar.<name>:set_float(val)`, `:set_int(val)`, `:set_string(val)`

---

## 10. Audio & Sound System (`sounds` / `sound`)

The `sounds` (or `sound` / `client.sound`) library gives scripts full control over local 2D and 3D spatial audio playback, as well as the ability to dynamically override the game's native **Hitsound** and **Killsound** with custom Roblox asset IDs or built-in presets.

Overrides automatically reset when the script is unloaded, ensuring no persistent side effects.

### Sound ID Resolution
All sound functions accept:
- **Raw Numbers**: `123456789` -> resolved to `"rbxassetid://123456789"`
- **Numeric Strings**: `"123456789"` -> resolved to `"rbxassetid://123456789"`
- **Full Asset IDs**: `"rbxassetid://123456789"` or `"rbxasset://sounds/..."`
- **Preset Names**: `"skeet"`, `"neverlose"`, `"bell"`, `"bubble"`, `"fatality"`, etc. (case-insensitive)

### API Methods

| Method | Parameters | Description |
| :--- | :--- | :--- |
| `sounds.play(id, [volume], [pitch])` | `id: any, volume?: number, pitch?: number` | Plays a local 2D sound immediately. Returns the created `Sound` instance. |
| `sounds.play_spatial(id, posOrPart, [volume], [pitch], [rollOffMin], [rollOffMax])` | `id: any, target: Vector3 \| BasePart, ...` | Plays 3D positional audio attached to a `Vector3` or `BasePart`. |
| `sounds.play_hitsound([customId], [volume], [pitch])` | `customId?: any, volume?: number, pitch?: number` | Plays the active hitsound (or custom ID if provided). |
| `sounds.play_killsound([customId], [volume], [pitch])` | `customId?: any, volume?: number, pitch?: number` | Plays the active killsound (or custom ID if provided). |
| `sounds.override_hitsound(id, [volume], [pitch])` | `id: any, volume?: number, pitch?: number` | **Overrides in-game hitsound**. Every bullet hit will play this sound instead. Pass `nil` to clear. |
| `sounds.override_killsound(id, [volume], [pitch])` | `id: any, volume?: number, pitch?: number` | **Overrides in-game killsound**. Every kill will play this sound instead. Pass `nil` to clear. |
| `sounds.reset_hitsound()` / `sounds.clear_hitsound_override()` | _none_ | Restores default hitsound to the menu selection. |
| `sounds.reset_killsound()` / `sounds.clear_killsound_override()` | _none_ | Restores default killsound to the menu selection. |
| `sounds.get_hitsound_override()` | _none_ | Returns `{ id = string, volume = number?, pitch = number? }` or `nil`. |
| `sounds.get_killsound_override()` | _none_ | Returns `{ id = string, volume = number?, pitch = number? }` or `nil`. |
| `sounds.get_hitsound_names()` | _none_ | Returns a list of all built-in preset sound names. |
| `sounds.resolve_id(idOrName)` | `idOrName: any` | Resolves a preset name or numeric ID into a valid `"rbxassetid://..."` string. |
| `sounds.presets` | _table_ | Read-only map of preset names to asset IDs (`sounds.presets.skeet`, `sounds.presets.bell`, etc.). |

### Built-in Presets
Built-in preset names:
- `"bell"`
- `"bubble"`
- `"pop"`
- `"ding"`
- `"sparkle"`
- `"laser"`
- `"bip"`
- `"punch"`
- `"crack"`
- `"bite"`
- `"bonk"`
- `"arcade"`
- `"metallic"`
- `"zap"`
- `"click"`
- `"coin"`
- `"glass"`
- `"retro"`
- `"hit"`
- `"skeet"`
- `"neverlose"`
- `"fatality"`

### Example: Custom Hitsound & Killsound Override UI

```lua
local my_tab = menu.new_tab("Audio Mod", "rbxassetid://10734975692")
local sec = my_tab:new_section("Hitsounds", "Left")

local hitsound_names = sounds.get_hitsound_names()
local hs_dropdown = sec:new_combobox("Custom Hitsound", hitsound_names, "skeet", function(chosen)
    sounds.override_hitsound(chosen, 1.0, 1.0)
    client.log("Switched hitsound to preset: " .. tostring(chosen))
end)

local custom_id_box = sec:new_textbox("Custom ID", "Enter Roblox Asset ID", "", function(id)
    if id and #id > 0 then
        sounds.override_hitsound(id, 1.2, 1.0)
        client.log("Applied custom hitsound asset ID: " .. tostring(id))
    end
end)

local pitch_slider = sec:new_slider("Pitch", 50, 200, 100, 0, "%", function(val)
    local cur = sounds.get_hitsound_override()
    if cur then
        sounds.override_hitsound(cur.id, cur.volume or 1.0, val / 100)
    end
end)

-- Play spatial 3D audio on aimbot hit:
client.set_event_callback("aim_hit", function(target, hitbox, damage)
    local head_pos = entity.get_hitbox_position(target, 0)
    if head_pos then
        sounds.play_spatial("sparkle", head_pos, 0.8, 1.2)
    end
end)

client.set_event_callback("shutdown", function()
    sounds.reset_hitsound()
    sounds.reset_killsound()
end)
```

---

## 11. Example Scripts

### Example 1: Brute-Force HvH Resolver with Dynamic BAim & HUD

```lua

local my_tab = menu.new_tab("Resolver", "rbxassetid://10734975692")
local sec = my_tab:new_section("Resolver Settings", "Left")
local hud_sec = my_tab:new_section("Display", "Right")

local enabled = sec:new_toggle("Enable Lua Resolver", true)
local baim_miss = sec:new_slider("BAim After Misses", 1, 5, 2, 0, " misses")
local show_hud = hud_sec:new_toggle("Show Debug HUD", true)

-- Per-target resolver state
local targetState = {}
local function getState(slot)
    if not targetState[slot] then
        targetState[slot] = { misses = 0, stage = 1 }
    end
    return targetState[slot]
end

-- Brute-force angle sequence relative to visual yaw
local BRUTE_ANGLES = { 58, -58, 0, 86, -86 }

-- Shot Event Hooks
client.set_event_callback("aim_hit", function(e)
    local state = getState(e.target)
    state.misses = 0 -- Reset misses on hit
end)

client.set_event_callback("aim_miss", function(e)
    local state = getState(e.target)
    if e.reason == "resolver" then
        state.misses = state.misses + 1
        state.stage = (state.stage % #BRUTE_ANGLES) + 1
    end
end)

-- 64 Hz Resolver Loop
events.on_tick(function(tick)
    if not enabled:get() then
        plist.reset_all()
        return
    end

    local enemies = entity.get_enemies()
    for _, enemy in ipairs(enemies) do
        if entity.is_alive(enemy) then
            local slot = entity.get_player_slot(enemy) or enemy
            local state = getState(slot)

            -- Apply resolved angle offset
            local offset = BRUTE_ANGLES[state.stage] or 0
            plist.set(enemy, "Force body yaw", true)
            plist.set(enemy, "Override yaw offset", offset)

            -- Force BAim when threshold reached
            if state.misses >= baim_miss:get() then
                plist.set(enemy, "Override prefer body aim", "Force")
            else
                plist.set(enemy, "Override prefer body aim", "-")
            end
        end
    end
end)

-- On-Screen Watermark & Indicator
client.set_event_callback("paint", function()
    if not show_hud:get() or not enabled:get() then return end

    local cur = rage.get_target()
    if cur and cur.slot ~= nil then
        local state = getState(cur.slot)
        local ang = BRUTE_ANGLES[state.stage] or 0
        render.indicator(0, 255, 160, 255, string.format("RESOLVER: %d° | MISS: %d", ang, state.misses))
    else
        render.indicator(200, 200, 200, 255, "RESOLVER: ACTIVE")
    end
end)

-- Cleanup on unload
client.set_event_callback("shutdown", function()
    plist.reset_all()
    render.clear()
end)
```

---

### Example 2: Custom Anti-Aim Controller (86° Desync & Fake Lag)

```lua
local my_tab = menu.new_tab("Anti-Aim", "rbxassetid://10734975692")
local sec = my_tab:new_section("AA Settings", "Left")

local aa_toggle = sec:new_toggle("Enable Custom AA", true)
local pitch_sel = sec:new_combobox("Pitch", { "Down", "Up", "Zero" }, "Down")
local desync_lim = sec:new_slider("Desync Limit", 0, 86, 86, 0, "°")
local lag_ticks = sec:new_slider("Fake Lag", 0, 14, 12, 0, "t")
local invert_key = sec:new_keybind("Invert Side", "Toggle", "F")

events.on_tick(function(tick)
    if not aa_toggle:get() then
        aa.reset()
        return
    end

    -- Pitch & Backward Yaw
    aa.override_pitch(pitch_sel:get())
    aa.override_yaw(180, "camera")

    -- Mathematical Sway Jitter Modifier
    aa.override_yaw_modifier(function(t, baseYaw)
        return math.sin(t * 0.2) * 35
    end)

    -- Desync Unlocked up to 86°
    local limit = desync_lim:get()
    aa.override_body_yaw(true)
    aa.override_inverter(invert_key:get())
    aa.override_desync_limits(limit, limit)

    -- Fake Lag Choking
    aa.override_fake_lag(true)
    aa.override_fake_lag_limit(lag_ticks:get())
end)

client.set_event_callback("paint", function()
    if not aa_toggle:get() then return end
    local choking = aa.is_fake_lag_choking()
    local side = aa.get_inverter() and "RIGHT" or "LEFT"
    render.indicator(240, 60, 70, 255, string.format("AA: 86° [%s] | CHOKE: %s", side, choking and "YES" or "NO"))
end)

client.set_event_callback("shutdown", function()
    aa.reset()
    render.clear()
end)
```

---

### Example 3: Fully Standalone Custom GUI Window (Draggable, Modal Cursor Unlock & Springs)

This complete script demonstrates building an independent standalone GUI entirely through `Drawing.new` and `tween`, featuring first-person cursor unlocking via `input.toggle_cursor()`, window dragging via `input.get_mouse_delta()`, hierarchical child parenting (`.Parent = window`), and spring-animated toggle buttons:

```lua
-- Keybind to open/close custom GUI and unlock mouse in 1st person
local GUI_TOGGLE_KEY = "RightShift"

-- 1. Main Window Container
local window = Drawing.new("Square")
window.Size = vector(320, 240)
window.Position = vector(200, 200)
window.Color = color(20, 22, 28)
window.Rounding = 8
window.BorderColor = color(45, 50, 65)
window.ClipsDescendants = true
window.Visible = false

-- 2. Header Bar (Parented to Window)
local header = Drawing.new("Square")
header.Size = vector(320, 36)
header.Position = vector(0, 0)
header.Color = color(28, 30, 40)
header.Rounding = 8
header.BorderColor = color(55, 60, 80)
header.Parent = window

local title = Drawing.new("Text")
title.Text = "penablox // Standalone Lua GUI"
title.Position = vector(14, 10)
title.Color = color(255, 255, 255)
title.Size = 13
title.Font = 0
title.Parent = header

-- 3. Custom Animated Toggle Component
local toggle_label = Drawing.new("Text")
toggle_label.Text = "Enable Fast Resolver"
toggle_label.Position = vector(20, 60)
toggle_label.Color = color(210, 210, 215)
toggle_label.Size = 13
toggle_label.Parent = window

local toggle_bg = Drawing.new("Square")
toggle_bg.Size = vector(40, 20)
toggle_bg.Position = vector(260, 58)
toggle_bg.Color = color(35, 38, 48)
toggle_bg.Rounding = 10
toggle_bg.BorderColor = color(60, 65, 85)
toggle_bg.Parent = window

local toggle_knob = Drawing.new("Circle")
toggle_knob.Radius = 7
toggle_knob.Position = vector(268, 68)
toggle_knob.Color = color(180, 180, 190)
toggle_knob.Filled = true
toggle_knob.Parent = window

local toggle_state = false
local knob_spring = tween.new_spring(268, 268, 0.72, 16)

-- 4. Window Dragging & Cursor Unlock State
local is_gui_open = false

-- Bind smooth drag handling (Mobile touch + Desktop mouse with auto-clamping)
local drag_handle = window:MakeDraggable(header, function(new_pos)
    -- Optional callback on drag
end)

local function set_gui_open(open)
    is_gui_open = open
    window.Visible = open
    input.set_cursor_unlocked(open) -- Automatically frees mouse in first person!
    
    if open then
        -- Play entrance scale / fade bounce
        window.Position = vector(200, 180)
        local anim = tween.create(window, 0.35, {
            Position = vector(200, 200)
        }, { style = "back", direction = "out" })
        anim:play()
    end
end

-- Hotkey hook to toggle UI
input.on_key_press(GUI_TOGGLE_KEY, function()
    set_gui_open(not is_gui_open)
end)

-- Main Render Loop
client.set_event_callback("paint", function(dt)
    if not is_gui_open then return end

    -- Handle Toggle Click (Single-click edge detection: does NOT cycle when held down!)
    if toggle_bg:IsClicked() then
        toggle_state = not toggle_state
        knob_spring:SetTarget(toggle_state and 292 or 268)
        
        tween.create(toggle_bg, 0.2, {
            Color = toggle_state and color(70, 130, 240) or color(35, 38, 48)
        }):play()
    end

    -- Update animated knob position
    local current_x = knob_spring:Update(dt or 0.016)
    toggle_knob.Position = vector(current_x, 68)
end)

-- Cleanup on unload
client.set_event_callback("shutdown", function()
    input.lock_cursor()
    if drag_handle and drag_handle.Destroy then
        drag_handle:Destroy()
    end
    window:Destroy()
end)
```

---

## 8. In-Game Live Lua Console

penablox RECODE includes a built-in GameSense / Skeet-styled Live Lua Console accessible directly from the menu:
- **Location**: Menu -> **Settings** -> **Lua scripts** -> **"Open lua console"**
- **Toolbar Features**:
  - `Stop output`: Freezes incoming log streaming and autoscroll so you can inspect active output.
  - `Clear`: Flushes all log history and resets line numbers.
  - `Close`: Closes/hides the console window.
- **Copying & Selecting**: The console viewport uses a custom non-editable TextBox (`TextEditable = false`), allowing you to click, drag-select errors, and press `Ctrl+C` to copy error traces with gutter line numbers aligned.
- **Integrated Logging Functions**:
  - `print(...)`: Streams formatted text directly to both Roblox console and Lua Console with prefix `[script.lua]`.
  - `warn(...)`: Streams warnings to the console tagged with `[WARN]`.
  - `client.log(...)`: Logs info entries to the live console.
  - `client.color_log(r, g, b, ...)`: Formats and logs entries to the live console.
  - `client.error(...)`: Logs custom error traces tagged with `[ERROR]`.
  - Automatic compilation & runtime errors, callback crashes, and load/unload notices stream in real-time.

---

## 12. Weapon System (`weapon`)

The `weapon` library provides access to authoritative active weapon data, ammo counts, ballistic properties, spread, recoil, and deterministic seeds.

All functions accept an optional target player/entity (defaults to local player).

### API Methods

| Method | Returns | Description |
| :--- | :--- | :--- |
| `weapon.get_active([target])` | `table` | Returns active weapon table (`id`, `name`, `type`, `slot`, `category`, `ammo`, `max_ammo`, `reserve`, `is_reloading`, `is_scoped`, `scope_level`, `spread`, `recoil`). |
| `weapon.get_ammo([target])` | `number` | Current ammo count in the weapon clip. |
| `weapon.get_max_ammo([target])` | `number` | Maximum magazine/clip capacity (`wData.ClipSize`). |
| `weapon.get_clip_size([target])` | `number` | Alias for `weapon.get_max_ammo`. |
| `weapon.get_reserve([target])` | `number` | Remaining reserve ammo count. |
| `weapon.is_reloading([target])` | `boolean` | `true` if the weapon is currently undergoing a reload cycle. |
| `weapon.is_scoped([target])` | `boolean` | `true` if currently scoped in. |
| `weapon.get_scope_level([target])` | `number` | Zoom/scope level (`0` = unscoped, `1` = 1x zoom, `2` = 2x zoom). |
| `weapon.get_spread([target])` | `number` | Weapon base spread value at current scope level. |
| `weapon.get_inaccuracy([target])` | `number` | Stand/duck inaccuracy coefficient. |
| `weapon.get_recoil([target])` | `table` | Recoil metadata table (`seed`, `angle`, `angle_variance`, `magnitude`, `magnitude_variance`). |
| `weapon.get_seed([target])` | `number` | Authoritative Wang/Murmur deterministic spread seed integer `[1..256]` for current command tick and viewangles. |
| `weapon.get_info(weaponId)` | `table` | Returns static registry specifications for any weapon ID or name (`damage`, `armor_penetration`, `fire_rate`, `can_double_tap`, `clip_size`, `penetration_power`, `name`). Also supports indices `[1]..[4]`. |

### Example

```luau
local cur = weapon.get_active()
print(string.format("Weapon: %s | Ammo: %d/%d | Reloading: %s | Seed: %d",
    cur.name,
    weapon.get_ammo(),
    weapon.get_max_ammo(),
    tostring(weapon.is_reloading()),
    weapon.get_seed()
))

-- Inspect static registry stats for Scout:
local scout_info = weapon.get_info("SSG 08")
print(string.format("Base Damage: %d | Armor Penetration: %.2f | Fire Rate: %.2f | Can DoubleTap: %s",
    scout_info.damage,
    scout_info.armor_penetration,
    scout_info.fire_rate,
    tostring(scout_info.can_double_tap)
))
```

---

## 13. Movement & Trajectory Prediction (`movement`)

The `movement` library provides access to player kinematics, ground and ducking states, and 64Hz physics trajectory extrapolation.

### API Methods

| Method | Returns | Description |
| :--- | :--- | :--- |
| `movement.get_velocity([target])` | `Vector3` | Current linear velocity vector in studs/sec. |
| `movement.get_speed([target], [includeVertical])` | `number` | Horizontal movement speed in studs/sec (or 3D magnitude if `includeVertical` is true). |
| `movement.is_grounded([target])` | `boolean` | `true` if the entity is currently resting on the ground/surface. |
| `movement.is_ducking([target])` | `boolean` | `true` if the entity is currently crouching or crouch-walking. |
| `movement.is_crouching([target])` | `boolean` | Alias for `movement.is_ducking`. |
| `movement.get_duck_amount([target])` | `number` | Crouch progress factor (`0.0` = fully standing, `1.0` = fully crouched). |
| `movement.get_state([target])` | `string` | Movement state name (`"Standing"`, `"Running"`, `"Crouching"`, `"InAir"`). |
| `movement.predict_position(entity, [ticks])` | `Vector3` | 64Hz multi-tick trajectory extrapolation simulating velocity, air gravity (800 studs/s²), and world raycast surface collisions over `ticks` forward in time. |

### Example: Hitbox Path Prediction

```luau
local enemies = entity.get_enemies()
for _, enemy in ipairs(enemies) do
    if entity.is_alive(enemy) then
        local futurePos = movement.predict_position(enemy, 16) -- Predict 16 ticks (250ms) ahead
        render3d.line(entity.get_player_position(enemy), futurePos, Color3.fromRGB(0, 255, 120), 2)
        render3d.sphere(futurePos, 0.5, Color3.fromRGB(0, 255, 120), 0.2)
    end
end
```

---

## 14. Chams & Visual Materials Override (`chams`)

The `chams` library grants custom Lua scripts full override control over the game's limb-level occlusion pipeline, custom materials, and colors for both visible and occluded/wall-penetrated character models.

Overrides automatically reset when the script unloads.

### API Methods

| Method | Parameters | Description |
| :--- | :--- | :--- |
| `chams.override_visible([color], [material], [transparency])` | `color?: Color3, material?: string, transparency?: number` | Overrides color, material, and transparency for all visible player limbs. Pass `nil` to clear. |
| `chams.override_occluded([color], [material], [transparency])` | `color?: Color3, material?: string, transparency?: number` | Overrides color, material, and transparency for all occluded (behind-wall) player limbs. |
| `chams.override_hidden(...)` | _same as above_ | Alias for `chams.override_occluded`. |
| `chams.apply_to_player(entity, [visConfig], [occludedConfig])` | `entity: any, visConfig?: table, occludedConfig?: table` | Applies custom chams styling to a specific player/character. |
| `chams.apply(entity, ...)` | _same as above_ | Alias for `chams.apply_to_player`. |
| `chams.clear_overrides()` / `chams.reset()` | _none_ | Clears all active global and per-player chams overrides. |
| `chams.set_enabled(state)` | `state: boolean` | Enables or disables script-driven chams. |
| `chams.get_materials()` | _none_ | Returns a list of all available material names. |
| `chams.materials` | _table_ | Read-only list of available material names. |
| `chams.get_palettes()` | _none_ | Returns a list of all procedural palette shader names. |
| `chams.palettes` | _table_ | Read-only list of palette shader names. |

### Available Materials

- `"Solid"`: Standard clean flat shading.
- `"Flat"`: Screen-space Silhouette Highlight chams.
- `"Glow"`: Outward ambient volumetric neon glow.
- `"Wireframe"`: Geometry edge outline adornment.
- `"Glass"`: Translucent refractive glass.
- `"ForceField"`: Roblox animated ForceField shader.
- `"Neon"`: High-intensity emissive neon.
- `"Metalic"`: High-specular reflective surface.
- `"Bubble"`: Dual-tone procedural fresnel sheen.
- `"Pulse"`: Real-time rhythmic sine-wave color pulsation.

### Example: Dynamic Health-Based Enemy Chams

```luau
chams.set_enabled(true)
chams.override_visible(Color3.fromRGB(80, 220, 255), "Neon", 0.1)
chams.override_occluded(Color3.fromRGB(255, 60, 60), "Wireframe", 0.3)

client.set_event_callback("shutdown", function()
    chams.clear_overrides()
end)
```

---

## 15. 3D Adornment Rendering (`render3d` / `Drawing3D`)

The `render3d` library (also aliased as `Drawing3D` or `render.d3d` / `render._3d`) renders hardware-accelerated 3D geometric shapes directly in world space using Roblox handle adornments (`BoxHandleAdornment`, `SphereHandleAdornment`, `CylinderHandleAdornment`, `LineHandleAdornment`, `WireframeHandleAdornment`) parented to `Workspace.Terrain`.

### Features
- **Zero-Allocation Frame Pooling**: Immediate mode draw calls reuse pooled adornments across frames with zero GC overhead.
- **Automatic Lifecycle Cleanup**: All retained and pooled adornments are tracked in `scriptInstance.CreatedInstances` and destroyed on script unload.
- **AlwaysOnTop & Depth**: Supports both depth-tested and x-ray `AlwaysOnTop = true` rendering.

### Immediate Draw Methods (`render3d.*` / `render.*3d`)

| Method | Parameters | Description |
| :--- | :--- | :--- |
| `render3d.box(cframeOrPos, size, [color], [transparency], [alwaysOnTop], [zIndex])` | `cf: CFrame \| Vector3, size: Vector3 \| number, ...` | Draws a solid 3D box. |
| `render3d.wire_box(cframeOrPos, size, [color], [transparency], [alwaysOnTop], [zIndex])` | `cf: CFrame \| Vector3, size: Vector3 \| number, ...` | Draws a 12-edge wireframe 3D box. |
| `render3d.sphere(pos, radius, [color], [transparency], [alwaysOnTop], [zIndex])` | `pos: Vector3, radius: number, ...` | Draws a smooth 3D sphere. |
| `render3d.cylinder(cframeOrPos, radius, height, [color], [transparency], [alwaysOnTop], [zIndex])` | `cf: CFrame \| Vector3, radius: number, height: number, ...` | Draws a 3D cylinder. |
| `render3d.ring(pos, radius, [thickness], [color], [transparency], [alwaysOnTop], [zIndex])` | `pos: Vector3, radius: number, thickness?: number, ...` | Draws a hardware-accelerated flat 3D ring on the ground. |
| `render3d.line(startPos, endPos, [color], [thickness], [transparency], [alwaysOnTop], [zIndex])` | `p1: Vector3, p2: Vector3, ...` | Draws a 3D world space line between two coordinates. |
| `render3d.clear()` | _none_ | Immediately hides all active pooled adornments and clears wireframe buffers. |

### Retained Mode Constructors (`render3d.create`)

```luau
local boxHandle = render3d.create("box", {
    Position = Vector3.new(0, 10, 0),
    Size = Vector3.new(4, 4, 4),
    Color = Color3.fromRGB(255, 100, 100),
    Transparency = 0.5,
    AlwaysOnTop = true,
})
-- Update properties anytime:
boxHandle.CFrame = CFrame.new(0, 15, 0)
-- Cleanup:
boxHandle:Destroy()
```

### Example: World Visuals (Target Hitbox & Grenade Ring)

```luau
client.set_event_callback("paint", function()
    local target = rage.get_target()
    if target and target.point then
        -- 3D Target Hitbox Box & Ring
        render3d.box(CFrame.new(target.point), Vector3.new(1.2, 1.2, 1.2), Color3.fromRGB(255, 40, 40), 0.4, true)
        render3d.ring(target.point - Vector3.new(0, 0.6, 0), 2.0, 0.1, Color3.fromRGB(255, 40, 40), 0.2, true)
    end
end)
```

---

## 16. In-Game Notifications (`notify` / `ui.notify` / `client.notify`)

Lua scripts can trigger clean toast notifications that display directly through SynUI's native HUD banner notification system.

### Signatures

- `notify(config: table)`: Config table: `{ Title = string, Text = string, Duration = number?, Sound = boolean? }`.
- `notify(title: string, text: string, [duration: number], [sound: boolean])`: Positional arguments.
- `ui.notify(...)`: Alias for `notify`.
- `client.notify(...)`: Alias for `notify`.

### Example

```luau
notify("Config Loaded", "Loaded custom rage settings", 3, true)

-- Using table format:
notify({
    Title = "Resolver",
    Text = "Reset resolver statistics on round start",
    Duration = 2.5,
    Sound = false,
})
```

---

## 17. 2D Screen-Space Drawing Engine (`Drawing`)

High-performance hardware-accelerated 2D rendering pipeline featuring zero-allocation object pooling, fastcall primitives, and resolution independence.

### Constructors
- `Drawing.new(kind: string)`: Creates a new drawing primitive (`"line"`, `"circle"`, `"square"`, `"triangle"`, `"quad"`, `"text"`, `"image"`).

### Common Properties (All Primitives)
- `Visible: boolean`: Toggles render visibility.
- `ZIndex: number`: Layering order.
- `Transparency: number`: Opacity (0 = transparent, 1 = fully opaque).
- `Color: Color3`: Drawing color.
- `Remove()` / `Destroy()`: Returns the primitive to the zero-allocation object pool.

### Line Primitive (`"line"`)
- `From: Vector2`: Starting screen coordinate.
- `To: Vector2`: Ending screen coordinate.
- `Thickness: number`: Line stroke width in pixels.
- `Length: number` (read-only): Pixel length of the line.
- **Fastcall Methods**:
  - `line:SetPoints(from: Vector2, to: Vector2)`: Atomically updates both endpoints with a single visual recalculation.
  - `line:SetCoords(x1: number, y1: number, x2: number, y2: number, [visible: boolean])`: Ultra-fast native fastcall bypassing Vector2 table allocations and setting pixel bounds directly. If `visible` is provided, updates visibility atomically.

---

## 18. HTTP & Networking System (`game:HttpGet`, `http`, `request`, `syn.request`)

penablox RECODE includes an enterprise-grade, asynchronous HTTP networking suite designed for loading remote scripts, fetching external configurations, interacting with REST APIs, and sending webhooks.

### 18.1 Architecture & Engine Security
In Roblox, client-side scripts (`LocalScript`) are strictly forbidden by the engine from making HTTP requests directly (`HttpService cannot be accessed from a LocalScript`).
To solve this seamlessly, the LuaEngine routes all HTTP calls through a dedicated, high-performance server proxy (`LuaHttpProxy RemoteFunction`):
- **100% Non-Blocking Execution**: Requests are executed asynchronously on the server via `HttpService:RequestAsync` wrapped in safe `pcall` guards, never causing client micro-stutters or frame drops.
- **Dual Token-Bucket Rate Limiter**:
  - **Per-Player Bucket**: Allows bursts of up to 5 requests, with a continuous refill rate of `0.5 tokens/sec` (up to ~30 requests per minute per player). If exceeded, requests return status `429 Too Many Requests`.
  - **Server-Wide Global Bucket**: Capped at 120 requests per minute across all players to guarantee the game server never exceeds Roblox's engine-level quota of 500 requests/minute.
- **SSRF & Security Isolation**:
  - Only `http://` and `https://` protocols are permitted.
  - Comprehensive protection against Server-Side Request Forgery (SSRF): automatically blocks loopback/localhost (`127.0.0.0/8`, `0.0.0.0/8`, `::1`), cloud metadata services (`169.254.0.0/16`), private RFC 1918 subnets (`10.0.0.0/8`, `192.168.0.0/16`, `172.16.0.0/12`), and internal Roblox web domains (`roblox.com`, `rbxcdn.com`).
- **Memory & Payload Safety**:
  - Request body limit: 256 KB (`262,144 bytes`).
  - Response body truncation: 1 MB (`1,048,576 bytes`).
  - Headers: Maximum 32 custom headers with automatic CRLF-injection sanitation.

---

### 18.2 API Methods & Globals

#### 1. Classic `game:HttpGet` / `HttpGet`
Standard Roblox exploit method for downloading remote script source code or text files.
- `game:HttpGet(url: string, [returnRaw: boolean]) -> string`
- `game.HttpGet(game, url: string, [returnRaw: boolean]) -> string`
- `HttpGet(url: string, [returnRaw: boolean]) -> string`
- `http_get(url: string) -> string`

Throws a clean Luau error if the request fails or returns a non-2xx status code.

```luau
-- Fetch and run an external script or config:
local scriptCode = game:HttpGet("https://raw.githubusercontent.com/user/repo/main/resolver.lua")
local fn, err = loadstring(scriptCode)
if fn then
    fn()
else
    client.error("Failed to compile remote script: " .. tostring(err))
end
```

#### 2. Cheat HTTP Library (`http`)
High-level HTTP utility module.

| Method | Parameters | Returns | Description |
| :--- | :--- | :--- | :--- |
| `http.get(url, [headers])` | `url: string, headers?: table` | `string` | Performs an HTTP GET request and returns the response body string. Throws an error if request fails. |
| `http.post(url, [body], [contentType], [headers])` | `url: string, body?: string, contentType?: string, headers?: table` | `string` | Performs an HTTP POST request and returns the response body string. Throws an error if request fails. |
| `http.request(options)` | `options: table` | `table` | Performs a full HTTP request and returns a structured response table. |
| `http.json_encode(data)` | `data: any` | `string` | High-speed JSON serialization helper (alias for `json.stringify`). |
| `http.json_decode(str)` | `str: string` | `any` | High-speed JSON deserialization helper (alias for `json.parse`). |

```luau
-- Simple GET:
local rawData = http.get("https://api.example.com/status")

-- Simple POST:
local response = http.post(
    "https://api.example.com/logs",
    http.json_encode({ event = "aimbot_hit", target = "player1" }),
    "application/json"
)
```

#### 3. Exploit Compatibility (`request`, `http_request`, `syn.request`)
Universal request method supporting standard Synapse X / Script-Ware / Krnl formats.
- `request(options: table) -> table`
- `http_request(options: table) -> table`
- `syn.request(options: table) -> table`
- `client.request(options: table) -> table`

**Request Options Table:**
| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `Url` / `url` | `string` | **Yes** | Destination URL (`http://` or `https://`). |
| `Method` / `method` | `string` | No | HTTP method: `"GET"`, `"POST"`, `"HEAD"`, `"PUT"`, `"DELETE"`, `"PATCH"` (default `"GET"`). |
| `Headers` / `headers` | `table` | No | Key-value table of custom HTTP headers. |
| `Body` / `body` | `string` | No | Request body string for POST/PUT/PATCH requests. |

**Response Table:**
| Field | Type | Description |
| :--- | :--- | :--- |
| `Success` | `boolean` | `true` if the HTTP status code is in the 2xx range and no network error occurred. |
| `StatusCode` | `number` | HTTP response code (e.g. `200`, `201`, `400`, `404`, `429`, `502`). |
| `StatusMessage` | `string` | HTTP status message (e.g. `"OK"`, `"Not Found"`, `"Too Many Requests"`). |
| `Headers` | `table` | Key-value map of response headers returned by the server. |
| `Body` | `string` | Response payload as a string (up to 1 MB). |
| `Error` | `string?` | Descriptive error message if the request failed (or `nil` on success). |

```luau
local res = request({
    Url = "https://httpbin.org/post",
    Method = "POST",
    Headers = {
        ["Content-Type"] = "application/json",
        ["User-Agent"] = "penablox-client/2.0"
    },
    Body = http.json_encode({
        player = client.get_local_player_name(),
        ping = globals.realtime()
    })
})

if res.Success then
    print(`Server returned [{res.StatusCode}]: {res.Body}`)
else
    warn(`Request failed [{res.StatusCode}]: {res.Error or res.StatusMessage}`)
end
```

#### 4. Native `HttpService` Compatibility Proxy
For scripts expecting Roblox's standard `game:GetService("HttpService")`, the sandbox returns an intelligent proxy:
- `HttpService:GetAsync(url: string) -> string`: Routes through the server proxy and returns the body string.
- `HttpService:PostAsync(url: string, data: string, [contentType], [compress], [headers]) -> string`: Routes through the server proxy.
- `HttpService:RequestAsync(options: table) -> table`: Routes through the server proxy and returns standard Roblox response dictionary.
- `HttpService:JSONEncode(data)` & `HttpService:JSONDecode(str)`: Executed locally on the client at native C++ speed.
- `HttpService:GenerateGUID([wrap: boolean])`: Executed locally on the client.

---

### 18.3 Complete Examples

#### Example 1: Asynchronous Cloud Config Loader with Error Fallback
```luau
local CONFIG_URL = "https://raw.githubusercontent.com/user/configs/main/hvh.json"

local function loadCloudConfig()
    print("Fetching remote config from cloud...")
    
    local ok, content = pcall(function()
        return game:HttpGet(CONFIG_URL)
    end)

    if not ok then
        client.error("Failed to download cloud config: " .. tostring(content))
        return
    end

    local parseOk, cfg = pcall(function()
        return http.json_decode(content)
    end)

    if not parseOk or type(cfg) ~= "table" then
        client.error("Cloud config is not valid JSON!")
        return
    end

    -- Apply settings
    if cfg.desync_yaw then
        aa.override_desync_limits(cfg.desync_yaw, cfg.desync_yaw)
    end
    if cfg.resolver_enabled ~= nil then
        plist.set_all("Force body yaw", cfg.resolver_enabled)
    end

    notify("Cloud Config", "Successfully applied latest HvH profile!", 3, true)
end

task.spawn(loadCloudConfig)
```

#### Example 2: Discord Webhook Killfeed Broadcaster
```luau
local WEBHOOK_URL = "https://discord.com/api/webhooks/YOUR_WEBHOOK_ID/TOKEN"

client.set_event_callback("aim_hit", function(target, hitbox, damage)
    -- Only trigger on lethal headshots
    if hitbox == 0 and damage >= 100 then
        local targetName = entity.get_player_name(target) or "Enemy"
        
        task.spawn(function()
            local payload = {
                username = "penablox HvH Bot",
                embeds = {
                    {
                        title = "🎯 Lethal Headshot Hit",
                        description = string.format("Eliminated **%s** with **%d** damage!", targetName, damage),
                        color = 65280, -- Green
                        footer = { text = "penablox RECODE LuaEngine" }
                    }
                }
            }

            local res = request({
                Url = WEBHOOK_URL,
                Method = "POST",
                Headers = { ["Content-Type"] = "application/json" },
                Body = http.json_encode(payload)
            })

            if not res.Success then
                client.log("[Webhook Error] Status: " .. tostring(res.StatusCode))
            end
        end)
    end
end)
```

---

## 19. Chat & TextChatService System (`chat`, `TextChatService`, `client.chat`)

penablox RECODE provides a streamlined, native API for interacting with the game's chat system, sending messages to team or global channels, listening to incoming messages, and wrapping Roblox's modern `TextChatService`.

### 19.1 Quick Chat Functions (`chat` / `client`)

| Method | Parameters | Returns | Description |
| :--- | :--- | :--- | :--- |
| `chat.send(message, [channelOrTeam])` | `message: string, channelOrTeam?: boolean \| string` | `(boolean, string?)` | Sends a chat message. If `true` or `"team"`, sends to current team channel. If `false`, `"all"` or omitted, sends to `AllChat` (or `DeadChat` if dead). If a channel name is passed (e.g. `"AllChat"`, `"DeadChat"`), sends to that channel. |
| `chat.say(message)` | `message: string` | `(boolean, string?)` | Quick alias to send to global / all chat. |
| `chat.team(message)` | `message: string` | `(boolean, string?)` | Quick alias to send specifically to the local player's team chat. |
| `chat.all(message)` | `message: string` | `(boolean, string?)` | Sends specifically to all chat. |
| `chat.get_channel(name)` | `name: string` | `TextChannel?` | Returns a secured `TextChannel` instance by name. |
| `chat.get_channels()` | _none_ | `table` | Returns a dictionary of all active `TextChannel` instances (`AllChat`, `Team_T`, `Team_CT`, `Team_Spectator`, `DeadChat`). |
| `chat.on_message(callback)` | `callback: function` | `RBXScriptConnection` | Listens for incoming chat messages across all channels. Auto-disconnects on script unload. Callback signature: `(sender: string, text: string, isTeam: boolean, channelName: string, messageObj: any)`. |
| `client.chat(message, [teamOnly])` | `message: string, teamOnly?: boolean` | `(boolean, string?)` | Gamesense / Skeet style chat alias. |
| `client.say(message)` | `message: string` | `(boolean, string?)` | Gamesense / Skeet style chat alias. |

### 19.2 Direct `TextChatService` Convenience Wrapper
Roblox's native `TextChatService` does not have a direct `:SendAsync()` method on the service itself (normally scripters must search for channels).
In the Lua sandbox, `TextChatService` (and `game:GetService("TextChatService")`) includes built-in convenience methods:
- `TextChatService:SendAsync(message: string, [channelOrTeam]) -> (boolean, string?)`
- `TextChatService:Chat(message: string, [channelOrTeam]) -> (boolean, string?)`
- `TextChatService.Chat(message: string, [channelOrTeam]) -> (boolean, string?)`
- `TextChatService:Send(message: string, [channelOrTeam]) -> (boolean, string?)`
- `TextChatService.TextChannels`: Access to underlying Roblox `TextChannels` folder.
- `TextChatService.MessageReceived:Connect(...)`: Standard event connection, tracked and cleaned up on unload.

### 19.3 Automated Event Callbacks
Incoming messages automatically trigger:
- `client.set_event_callback("chat_message", function(sender, text, isTeam, channel) ... end)`
- `client.set_event_callback("chat", function(sender, text, isTeam, channel) ... end)`

---

### 19.4 Complete Examples

#### Example 1: Automated Kill Say / Trash Talk
```luau
local TRASH_TALK = {
    "1",
    "sit down",
    "cfg issue",
    "penablox on top",
    "nice try",
}

client.set_event_callback("aim_hit", function(target, hitbox, damage)
    -- Trigger on lethal shot
    if damage >= 100 then
        local msg = TRASH_TALK[math.random(1, #TRASH_TALK)]
        chat.say(msg)
    end
end)
```

#### Example 2: Team Info & Callout Broadcaster
```luau
-- Broadcast low HP or bomb plant to team chat only:
local function reportToTeam(info)
    chat.team(`[INFO] {info}`)
end

-- Example usage:
reportToTeam("Spotted 2 enemies pushing B site!")
```

#### Example 3: Chat Command Parser (/resolver, /fps)
```luau
chat.on_message(function(sender, text, isTeam, channel)
    -- Ignore own messages
    if sender == client.get_local_player_name() then return end

    if text == "!help" then
        chat.say(`Hello {sender}, type !ping or !time`)
    elseif text == "!ping" then
        chat.say(`Current server ping: {math.floor(globals.realtime() % 100)}ms`)
    end
end)
```

## 20. Clantag & Identity System (`clantag`, `client.set_clantag`)

Penablox Recode features a fully networked, server-authoritative **Clantag System (FE)** that dynamically broadcasts player tags to all connected clients. Clantags appear across the entire game UI—including the dynamic killfeed, spectator bar, scoreboard, and chat—while strictly keeping **Name ESP** clean (displaying only genuine player names).

### 20.1 Architecture & Security Rules

1. **Server-Side Profanity & Text Filtering**:
   - Every clantag packet (`OP_CL_SET_CLANTAG`) sent to the server is processed through `filterServerText()` backed by Roblox `TextService:FilterStringAsync(..., sender.UserId):GetNonChatStringForBroadcastAsync()`.
   - Repeated animation frames are cached in `serverTextFilterCache` to guarantee zero latency and 0 redundant HTTP calls to TextService.
   - Any inappropriate tags are automatically sanitized before attribute assignment (`Player:SetAttribute("Clantag", cleanTag)`).
2. **Server-Side Rate Limiting**:
   - Updates are capped at 12 packets/sec per player to prevent network spam while allowing silky smooth 300ms marquee animations.
3. **Automatic Lifecycle Cleanup**:
   - When a Lua script unloads, `LuaEngine` detects `HasCustomClantag` and automatically calls `ClantagEngine.Reset()`, restoring the player's tag to empty.
4. **Zero Impact on Name ESP**:
   - ESP (`EspEngine`) explicitly resolves raw `DisplayName` or `Name`, ensuring ESP boxes and out-of-screen arrows remain uncluttered.

---

### 20.2 API Methods & Globals

The following functions are available under the `clantag` namespace, `client` namespace, and as top-level globals:

#### 1. `clantag.set(tag: string, anim_type?: string, speed?: number)`
Aliases: `client.set_clantag(...)`, `set_clantag(...)`

Activates a clantag on the local player.

- **`tag`** (`string`): The base tag string (maximum 32 characters).
- **`anim_type`** (`string?`): The animation mode (case-insensitive):
  - `"static"` or `nil`: Static non-animated tag.
  - `"marquee"`, `"scroll"`, `"running"`: Smooth left-to-right running line / ticker.
  - `"typewriter"`, `"build"`: Progressively types characters out and deletes them back.
  - `"custom"`: Uses a predefined frame sequence (see `set_frames`).
- **`speed`** (`number?`): Step interval in seconds (clamped to `[0.05, 5.0]`, defaults to `0.35`).

```luau
-- Static clantag:
clantag.set("gamesense")

-- Running line (marquee) with 0.3s speed:
clantag.set("penablox.cc", "marquee", 0.30)

-- Typewriter effect:
clantag.set("neverlose", "typewriter", 0.25)
```

#### 2. `clantag.reset()`
Aliases: `client.reset_clantag()`, `reset_clantag()`

Immediately stops any running clantag animation, clears the active tag on the client and server, and removes it from UI components.

```luau
clantag.reset()
```

#### 3. `clantag.get(): string`
Aliases: `client.get_clantag()`, `get_clantag()`

Returns the current active clantag string on the local client (or `""` if none is set).

```luau
local current = clantag.get()
print(`Active clantag: '{current}'`)
```

#### 4. `clantag.set_frames(frames: { string }, speed?: number)`

Plays a custom frame-by-frame animation from a table of strings.

- **`frames`** (`{ string }`): Sequential list of frames to cycle through.
- **`speed`** (`number?`): Delay in seconds between frames (defaults to `0.35`).

```luau
clantag.set_frames({
    "g",
    "ga",
    "gam",
    "game",
    "games",
    "gamese",
    "gamesen",
    "gamesens",
    "gamesense",
    "gamesens",
    "gamese",
    "games",
    "game",
    "gam",
    "ga",
    "g",
    "",
}, 0.2)
```

#### 5. `clantag.format(playerOrName: any, baseName?: string): string`

Formats a player's name with their active clantag (e.g. `[tag] Name`). If the player has no clantag, returns `baseName` or the player's DisplayName.

```luau
local target = entity.get_player(1)
if target then
    local formatted = clantag.format(target)
    print(`Player with tag: {formatted}`)
end
```

---

### 20.3 UI Integration Overview

| UI Element | Clantag Behavior |
| :--- | :--- |
| **Killfeed** | Formatted dynamically on killer, assist, and victim. Actively updates in real-time if a tag changes while the card is displayed. |
| **Spectator HUD** | Formatted in uppercase (`[TAG] USERNAME`) in the bottom target bar while spectating other players. |
| **Scoreboard** | Displays next to player names on the Tab scoreboard rows and in the spectator list. |
| **In-Game Chat** | Prepended before role badges and player names with subtle light gray styling (`[tag]`). |
| **Name ESP** | **Explicitly Excluded**. Only clean display names or account names are shown to ensure crystal clear target recognition. |

---

### 20.4 Complete Examples

#### Example 1: Gamesense Marquee Clantag with Menu Toggle
```luau
local clantagTab = menu.add_tab("Clantags")
local mainSec = clantagTab:add_section("Tag Settings")

local tagEnabled = mainSec:add_toggle("Enable Clantag", false)
local tagText = mainSec:add_textbox("Tag Text", "gamesense")
local animType = mainSec:add_combobox("Animation", { "Static", "Marquee", "Typewriter" }, "Marquee")
local animSpeed = mainSec:add_slider("Speed (ms)", 100, 1000, 300)

local function updateTag()
    if not tagEnabled:get() then
        clantag.reset()
        return
    end

    local text = tagText:get()
    local mode = string.lower(animType:get())
    local speedSec = animSpeed:get() / 1000

    clantag.set(text, mode, speedSec)
end

tagEnabled:on_change(updateTag)
tagText:on_change(updateTag)
animType:on_change(updateTag)
animSpeed:on_change(updateTag)
```

#### Example 2: Neverlose Custom Frame Animator
```luau
-- Custom multi-stage Neverlose brand animation:
local nlFrames = {
    "N",
    "Ne",
    "Nev",
    "Neve",
    "Never",
    "Neverl",
    "Neverlo",
    "Neverlos",
    "Neverlose",
    "Neverlose.",
    "Neverlose.c",
    "Neverlose.cc",
    "Neverlose.cc",
    "Neverlose.c",
    "Neverlose.",
    "Neverlose",
    "|everlose",
    "|",
    "",
}

clantag.set_frames(nlFrames, 0.25)
```

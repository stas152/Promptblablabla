# penablox RECODE — Lua API Documentation

> Complete Lua / Luau scripting reference for the penablox RECODE engine.  
> Compatible with Gamesense / Skeet-style scripts + native Roblox Instance APIs.

---

## Table of Contents

1. [Luau Syntax, Typing & Asynchronous Runtime](#1-luau-syntax-typing--asynchronous-runtime)
2. [UI & Menu System (`menu` / `ui`)](#2-ui--menu-system-menu--ui)
3. [Anti-Aim & Desync (`aa`)](#3-anti-aim--desync-aa)
4. [Ragebot & Resolver (`rage` / `plist`)](#4-ragebot--resolver-rage--plist)
5. [Entities & Players (`entity` / `client`)](#5-entities--players-entity--client)
6. [2D Canvas & Drawing (`render` / `Drawing`)](#6-2d-canvas--drawing-render--drawing)
7. [Tween & Animation (`tween`)](#7-tween--animation-tween)
8. [Input & Cursor Unlocking (`input`)](#8-input--cursor-unlocking-input)
9. [Events, Globals & Utilities](#9-events-globals--utilities)
10. [Audio & Sound System (`sounds`)](#10-audio--sound-system-sounds)
11. [Weapon System (`weapon`)](#11-weapon-system-weapon)
12. [Movement & Trajectory (`movement`)](#12-movement--trajectory-movement)
13. [Chams & Visual Materials (`chams`)](#13-chams--visual-materials-chams)
14. [3D Adornment Rendering (`render3d`)](#14-3d-adornment-rendering-render3d)
15. [Notifications (`notify`)](#15-notifications-notify)
16. [HTTP & Networking](#16-http--networking)
17. [Chat & TextChatService](#17-chat--textchatservice)
18. [Clantag System (`clantag`)](#18-clantag-system-clantag)
19. [Live Lua Console](#19-live-lua-console)
20. [Example Scripts](#20-example-scripts)

---

## 1. Luau Syntax, Typing & Asynchronous Runtime

Lua 5.1 compatible with full modern Luau features.

### 1.1 Type Annotations & Declarations

Type annotations are parsed natively and skipped at runtime with **zero overhead**.

```luau
-- Function Parameters & Return Types
local function calculate_angle(base: number, offset: number?): (number, boolean)
    local result: number = base + (offset or 0)
    return result, true
end

-- Local Variable Annotations
local is_active: boolean = true
local target_name: string? = nil
local player_list: { [number]: string } = {}

-- Type Assertions
local entity = get_target() :: any
local cast_num = raw_val :: number

-- Type Aliases
type ResolverRecord = {
    yaw: number,
    pitch: number,
    lby: number,
    choked: number,
}
export type AngleModifier = (number, number) -> number

-- Generics
local function get_or_default<T>(val: T?, fallback: T): T
    return if val ~= nil then val else fallback
end
```

### 1.2 Compound Assignment Operators

```luau
counter += 1
health -= damage
multiplier *= 1.5
divider /= 2
modVal %= 360
powVal ^= 2
str ..= "!"
```

### 1.3 Flow Control: `continue`

```luau
for i = 1, 10 do
    if i % 2 == 0 then
        continue
    end
    print("Odd:", i)
end
```

### 1.4 Loop Budget Protection & Zero-Lag Architecture

- **5ms Auto-Yield Budget**: Every 128 loop iterations the interpreter checks elapsed CPU time. If > 5 ms has been spent in the current frame it automatically yields via `task.wait()`.
- **Script Lifecycle Auto-Termination**: On unload/reload all background loops are instantly terminated.

### 1.5 Asynchronous Task Library

```luau
task.wait(0.5)
wait(0.1)                    -- alias

task.spawn(function()
    while true do
        task.wait(1)
        print("Heartbeat:", tick())
    end
end)

local timer = task.delay(3, function()
    print("3 seconds elapsed!")
end)

task.defer(function() ... end)
task.cancel(timer)

local now = tick()           -- engine-synchronized time
local runTime = time()       -- session time
```

### 1.6 Ternary & String Interpolation

```luau
local status = if entity.is_alive(0) then "ALIVE" else "DEAD"
local bonus  = if level > 10 then 50 elseif level > 5 then 25 else 0

local kills = 5
local msg = `Player has {kills} kills and {kills * 100} points!`
```

### 1.7 Native Roblox Instances & GUI

Full support for `Instance.new`, services, and data types.

```luau
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game:GetService("CoreGui") -- auto-redirects to PlayerGui

-- Supported services (safe CoreGui fallback)
game:GetService("Players")
game:GetService("RunService")
game:GetService("UserInputService")
game:GetService("TweenService")
game:GetService("HttpService")      -- fully proxied
game:GetService("TextChatService")  -- enhanced with :SendAsync / :Chat
```

**Automatic Lifecycle Cleanup** (on script unload):
1. All `Instance.new` objects are `:Destroy()`-ed
2. All event connections are disconnected
3. All `task.spawn` / `task.delay` threads are cancelled

### 1.8 Security Blocklist & JIT Optimizations

**Blocked** (static + runtime metatable guard):
- Remote methods: `:FireServer`, `:InvokeServer`, `:FireClient`, `:FireAllClients` (any case)
- Classes: `RemoteEvent`, `RemoteFunction`, `BindableEvent`, `BindableFunction`, `UnreliableRemoteEvent`
- Services: `NetworkClient`, `NetworkServer`, `ScriptContext`, `MessageBusService`, `HttpRbxApiService`

**JIT Pipeline**:
- Loop unrolling (constant bounds 1-4)
- Constant propagation & folding
- Algebraic simplification
- Dead code elimination
- Fast local variable path

---

## 2. UI & Menu System (`menu` / `ui`)

### Custom Tabs & Sections

```luau
local my_tab = menu.new_tab("HVH Lua", "rbxassetid://10734975692")
local left_sec = my_tab:new_section("Resolver Settings", "Left")  -- "Left" | "Right"
```

### Element Creators

| Method | Description | Returns |
|--------|-------------|---------|
| `sec:new_toggle(name, [default], [callback])` | Checkbox | `ElementHandle` |
| `sec:new_slider(name, min, max, [default], [decimals], [suffix], [callback])` | Slider | `ElementHandle` |
| `sec:new_combobox(name, options, [default], [callback])` | Dropdown | `ElementHandle` |
| `sec:new_keybind(name, [mode], [key], [callback])` | Hotkey (`"Toggle"` / `"Hold"` / `"Always"`) | `ElementHandle` |
| `sec:new_colorpicker(name, [default], [callback])` | RGBA picker | `ElementHandle` |
| `sec:new_textbox(name, [placeholder], [default], [callback])` | Text input | `ElementHandle` |
| `sec:new_button(name, callback)` | Button | `ElementHandle` |
| `sec:new_label(text)` | Static label | `ElementHandle` |

### Element Handle Methods

```luau
handle:get()
handle:set(value, [silent])
handle:set_visible(boolean)
handle:set_callback(function)
```

### Gamesense Compatibility (`ui.*`)

```luau
local pitch_mode, pitch_slider = ui.reference("AA", "Anti-aimbot angles", "Pitch")
local dt_toggle, dt_hotkey     = ui.reference("RAGE", "Aimbot", "Double tap")

ui.set(pitch_slider, -89)
local is_dt = ui.get(dt_toggle)

ui.new_checkbox(tab, section, name, [default])
ui.new_slider(tab, section, name, min, max, [default], [step], [unit], [scale])
ui.new_combobox(tab, section, name, options...)
ui.new_multiselect(tab, section, name, options...)
ui.new_color_picker(tab, section, name, r, g, b, [a])
ui.new_hotkey(tab, section, name, [inline], [key])
ui.new_button(tab, section, name, callback)

ui.is_menu_open() -> boolean
ui.menu_position() -> (number, number)
```

---

## 3. Anti-Aim & Desync (`aa`)

Complete control over pitch, yaw, desync (up to **86°**), and fake lag.

### Pitch

```luau
aa.override_pitch(pitch)          -- number [-89..89] or "Down" | "Up" | "Zero" | "Disabled"
aa.set_pitch(pitch)               -- alias
aa.get_pitch() -> number?
aa.is_pitch_overridden() -> boolean

aa.override_fake_pitch(val)       -- [-70..70] or presets
aa.get_fake_pitch() -> number?
aa.override_fake_pitch_angle(deg) -- offset [-20..20]
```

### Extended Angles (Body Lean / Roll)

```luau
aa.override_extended_angles(pct)          -- [-100..100]
aa.get_extended_angles() -> number
aa.is_extended_angles_overridden() -> boolean
aa.is_extended_angles() -> boolean
aa.set_extended_angles_enabled(enabled)
aa.override_extended_angles_enabled(enabled)
```

### Yaw & Modifiers

```luau
aa.override_yaw(deg, [base])              -- base = "camera" | "world"
aa.override_yaw_offset(deg, [relativeToSlider])
aa.add_yaw_offset(deg)                    -- adds to current menu slider
aa.get_slider_yaw_offset() -> number      -- aliases: get_menu_yaw_offset, get_base_yaw_offset, get_yaw_slider
aa.get_yaw() -> number?
aa.get_yaw_offset() -> number?

aa.override_yaw_modifier(modifier, [offset])
-- modifier = function(tick, baseYaw) -> number
-- or presets: "Spin", "Random", "Center", "3-Way", "5-Way", "Disabled"
```

### Manual Anti-Aim

```luau
aa.is_manual() -> boolean
aa.get_manual() -> string?                -- "Left" | "Right" | "Back" | nil
aa.set_manual(dir)                        -- "Left" | "Right" | "Back" | nil
```

### Body Yaw & Desync (up to 86°)

```luau
aa.override_body_yaw(enabled)
aa.override_inverter(inverted)            -- false = Fake Left, true = Fake Right
aa.override_desync_limits(leftDeg, rightDeg)  -- [0..86]
aa.override_desync(realDeg, fakeDeg)      -- [-86..86]
aa.override_show_fake(boolean)

aa.get_inverter() -> boolean
aa.get_desync_limits() -> (number, number)
aa.get_lby() / aa.get_lby_deg()
aa.get_fake_lby_deg()
aa.get_twist_deg()
```

### Fake Lag & Choke

```luau
aa.override_fake_lag(enabled)
aa.override_fake_lag_limit(ticks)         -- 0..14
aa.override_fake_lag_variability(pct)     -- 0..100
aa.is_fake_lag_choking() -> boolean
aa.get_choked_ticks() -> number
aa.get_choke_target() -> number
aa.unchoke() / aa.flush_choked()
aa.force_teleport()
```

### Defensive AA Primitives

```luau
rage.shift_ticks([ticks]) / rage.force_shift([ticks])
rage.is_double_tap_charged() -> boolean
rage.get_double_tap_charge() -> number    -- 0..14
```

### Reset

```luau
aa.reset() / aa.clear()
```

---

## 4. Ragebot & Resolver (`rage` / `plist`)

### Built-in Resolver Control

| Function | Description |
|----------|-------------|
| `rage.set_resolver_enabled(boolean)` | Enable / disable |
| `rage.is_resolver_enabled() -> boolean` | Status |
| `rage.set_resolver_mode(mode)` | `"Brute-force"` / `"Adaptive"` / `"Freestanding"` |
| `rage.get_resolver_mode() -> string` | Current mode |
| `rage.set_resolver_miss_limit(n)` | 1..5 (default 2) |
| `rage.set_resolver_baim_after_misses(n)` | 0..5 (0 = off) |
| `rage.reset_resolver_state([target])` | Reset miss counter |

### Player List Overrides (`plist`)

```luau
plist.set(target, key, value)
plist.get(target, key) -> any
plist.reset(target)
plist.reset_all()
```

**Supported keys**:
- `"Force body yaw"` (`boolean`)
- `"Force body yaw value"` / `"Override yaw"` (`number` [-180..180])
- `"Override yaw offset"` (`number`)
- `"Override prefer body aim"` / `"Force body aim"` (`"Force"` / `true` / `"-"` / `false`)
- `"Override safe point"` / `"Force safe point"`
- `"Force pitch"` (`boolean`) / `"Force pitch value"` (`number` [-89..89] or `"Down"` / `"Up"` / `"Zero"`)

### Quick Helpers

```luau
rage.override_player_yaw(target, deg)
rage.override_player_yaw_offset(target, offsetDeg)
rage.override_player_pitch(target, deg) / rage.set_player_pitch(target, deg)
rage.get_player_pitch(target) -> number?
rage.is_player_pitch_overridden(target) -> boolean
rage.reset_player_pitch(target)
rage.reset_player(target) / rage.reset_resolver(target)
```

### Global Rage Overrides

```luau
rage.override_enabled(boolean)
rage.override_min_damage(damage)
rage.override_hitchance(hitchance)
rage.override_force_baim(boolean)
rage.override_force_safe_point(boolean)
rage.override_target_selection(mode)      -- "Highest damage" | "Near crosshair" | "Best hitchance" | "Cycle"
rage.override_hitboxes(list)
rage.override_hitbox_priority(list)
rage.override_hitbox_min_damage(hitbox, dmg)
rage.override_hitbox_hitchance(hitbox, hc)
rage.set_hitbox_config(hitbox, { min_damage = 80, hitchance = 65, multipoint = true })

rage.get_target() -> TargetInfo?
rage.get_target_info() -> PlayerInfoTable?
rage.reset() / rage.clear()
```

### Shot Events

```luau
client.set_event_callback("aim_fire", function(e) ... end)  -- e.id, e.target, e.hitbox, e.damage, e.hitchance, e.resolved_yaw, e.point
client.set_event_callback("aim_hit",  function(e) ... end)  -- + e.health_remaining, e.backtrack_ticks
client.set_event_callback("aim_miss", function(e) ... end)  -- e.reason = "resolver" | "spread" | "occlusion" | "death"
```

---

## 5. Entities & Players (`entity` / `client`)

### Snapshots

```luau
local info = entity.get_info(target)
-- info.name, info.health, info.state, info.speed, info.position, info.velocity
-- info.yaw, info.pitch, info.lby, info.desync_delta, info.weapon_name, info.is_crouching, info.is_in_air

local anim = entity.get_anim_state(target)
-- anim.eye_angles_y, anim.torso_yaw, anim.goal_feet_yaw, anim.duck_amount, anim.desync
```

### Enumeration

```luau
entity.get_enemies()
entity.get_teammates()
entity.get_players([enemiesOnly])
entity.get_local_player()
entity.is_alive(target)
entity.is_enemy(target)
entity.get_player_health / armor / position / velocity / speed
entity.get_player_angles_deg(target) -> (pitch, yaw)
entity.get_player_hitbox_position(target, hitbox)  -- "head", "chest", "pelvis", "right_arm", ...
entity.get_player_weapon_name(target)
entity.is_player_scoped(target)
entity.get_player_lby_deg / get_player_twist_deg
entity.get_prop(target, "resolved_pitch")          -- authoritative resolved pitch
```

### Local Client

```luau
local wep = client.get_weapon()
-- wep.id, wep.name, wep.type, wep.ammo, wep.reserve, wep.scope_level, wep.is_scoped

local pitch, yaw = client.get_pitch_yaw()
local pDeg, yDeg, roll = client.get_camera_angles_deg()
local pingSec = client.latency()
```

---

## 6. 2D Canvas & Drawing (`render` / `Drawing`)

### Helpers

```luau
local v2 = vector(100, 200)
local v3 = vector(10, 20, 30)
local c1 = color(255, 60, 70)
local c2 = color("#00FFA0")
```

### Direct Render Functions

```luau
render.text(x, y, text, [color], [size], [outline], [center], [font], [zIndex])
render.rect / render.rect_filled
render.gradient(x, y, w, h, col1, col2, [vertical], [rounding])
render.line / render.circle / render.circle_filled / render.triangle
render.image(x, y, w, h, assetId, [color], [rounding])
render.indicator(r, g, b, a, text...)
render.measure_text(text, [size], [font]) -> Vector2
render.screen_size() -> (w, h)
render.world_to_screen(worldPos) -> (x, y, onScreen)
render.clear()
```

### Drawing Objects

```luau
local box = Drawing.new("Square")   -- "Text", "Square", "Line", "Circle", "Triangle", "Quad", "Image"
box.Size = vector(200, 40)
box.Position = vector(100, 100)
box.Color = color(25, 25, 30)
box.Rounding = 8
box.Parent = window                 -- hierarchical parenting
box:Destroy()
```

Fonts: `Drawing.Fonts.UI` (0), `System` (1), `Plex` (2), `Monospace` (3)

---

## 7. Tween & Animation (`tween`)

```luau
-- Interpolation
tween.interpolate(from, to, alpha, [style], [direction])

-- Frame-independent lerp
current = tween.lerp(current, target, speed, dt)

-- Spring physics
local spr = tween.new_spring(initial, [target], [damping], [speed])
local cur = spr:Update(dt)
spr:SetTarget(200)
spr:IsSettled()

-- Property animation
local anim = tween.create(obj, duration, {
    Position = vector(...),
    Size = vector(...),
    Color = color(...)
}, {
    style = "back",          -- linear, quad, cubic, quart, quint, sine, expo, circ, back, elastic, bounce
    direction = "out",       -- in, out, inOut
    autoplay = true,
    on_complete = function() end
})
anim:play() / :pause() / :cancel() / :reset() / :reverse()
```

---

## 8. Input & Cursor Unlocking (`input`)

### First-Person Cursor

```luau
input.unlock_cursor()
input.lock_cursor()
input.toggle_cursor() -> boolean
input.is_cursor_unlocked() -> boolean
input.set_cursor_unlocked(boolean)
```

### Polling & Hit-Testing

```luau
input.is_just_pressed([btn])
input.is_held([btn]) / input.is_down([btn])
input.is_just_released([btn])
input.is_pointer_down([btn])
input.is_touch() -> boolean
input.is_hovered(x, y, w, h)
input.is_clicked(x, y, w, h, [btn])
input.get_mouse_delta() / input.get_pointer_delta() -> (dx, dy)
input.get_mouse_position() / input.get_pointer_position() -> (x, y)
input.make_draggable(target, [handle], [callback])
input.is_key_down(key)
input.is_mouse_down(btn)
input.is_typing() -> boolean
input.on_key_press(key, callback)
```

---

## 9. Events, Globals & Utilities

### Time

```luau
globals.curtime()
globals.tickcount()
globals.tickinterval()       -- 0.015625
globals.realtime()
globals.frametime()
```

### Database (persists across reloads)

```luau
database.write("key", value)
local cfg = database.read("key")
database.flush()
```

### Bitwise & JSON

```luau
bit.band / bor / bxor / bnot / lshift / rshift
json.stringify(table)
json.parse(str)
```

### Console Variables

```luau
cvar.<name>:get_float() / :get_int() / :get_string()
cvar.<name>:set_float(val) / :set_int(val) / :set_string(val)
```

### Event System

```luau
client.set_event_callback("paint", function(dt) ... end)
client.set_event_callback("shutdown", function() ... end)
events.on_tick(function(tick) ... end)
```

---

## 10. Audio & Sound System (`sounds`)

```luau
sounds.play(id, [volume], [pitch])
sounds.play_spatial(id, posOrPart, [volume], [pitch], [rollOffMin], [rollOffMax])
sounds.play_hitsound([customId], [volume], [pitch])
sounds.play_killsound([customId], [volume], [pitch])

sounds.override_hitsound(id, [volume], [pitch])
sounds.override_killsound(id, [volume], [pitch])
sounds.reset_hitsound() / sounds.reset_killsound()
sounds.get_hitsound_override() / sounds.get_killsound_override()
sounds.get_hitsound_names()
sounds.resolve_id(idOrName)
sounds.presets                       -- table of built-in names → asset IDs
```

**Presets**: `"bell"`, `"bubble"`, `"pop"`, `"ding"`, `"sparkle"`, `"laser"`, `"bip"`, `"punch"`, `"crack"`, `"bite"`, `"bonk"`, `"arcade"`, `"metallic"`, `"zap"`, `"click"`, `"coin"`, `"glass"`, `"retro"`, `"hit"`, `"skeet"`, `"neverlose"`, `"fatality"`

---

## 11. Weapon System (`weapon`)

All methods accept optional `target` (defaults to local player).

```luau
weapon.get_active([target]) -> table
-- id, name, type, slot, category, ammo, max_ammo, reserve, is_reloading, is_scoped, scope_level, spread, recoil

weapon.get_ammo / get_max_ammo / get_clip_size / get_reserve
weapon.is_reloading / is_scoped
weapon.get_scope_level
weapon.get_spread / get_inaccuracy
weapon.get_recoil() -> table          -- seed, angle, angle_variance, magnitude, magnitude_variance
weapon.get_seed() -> number           -- [1..256]
```

---

## 12. Movement & Trajectory (`movement`)

```luau
movement.get_velocity([target]) -> Vector3
movement.get_speed([target], [includeVertical])
movement.is_grounded / is_ducking / is_crouching
movement.get_duck_amount() -> number  -- 0..1
movement.get_state() -> string        -- "Standing" | "Running" | "Crouching" | "InAir"
movement.predict_position(entity, [ticks]) -> Vector3  -- 64 Hz physics + gravity + raycast
```

---

## 13. Chams & Visual Materials (`chams`)

```luau
chams.set_enabled(state)
chams.override_visible([color], [material], [transparency])
chams.override_occluded([color], [material], [transparency])  -- alias: override_hidden
chams.apply_to_player(entity, [visConfig], [occludedConfig])
chams.clear_overrides() / chams.reset()
chams.get_materials() / chams.materials
chams.get_palettes() / chams.palettes
```

**Materials**: `"Solid"`, `"Flat"`, `"Glow"`, `"Wireframe"`, `"Glass"`, `"ForceField"`, `"Neon"`, `"Metalic"`, `"Bubble"`, `"Pulse"`

---

## 14. 3D Adornment Rendering (`render3d`)

Hardware-accelerated world-space shapes (BoxHandleAdornment, SphereHandleAdornment, etc.).

```luau
render3d.box(cframeOrPos, size, [color], [transparency], [alwaysOnTop], [zIndex])
render3d.wire_box(...)
render3d.sphere(pos, radius, ...)
render3d.cylinder(cframeOrPos, radius, height, ...)
render3d.ring(pos, radius, [thickness], ...)
render3d.line(startPos, endPos, [color], [thickness], ...)
render3d.clear()

-- Retained mode
local handle = render3d.create("box", {
    Position = Vector3.new(0, 10, 0),
    Size = Vector3.new(4, 4, 4),
    Color = Color3.fromRGB(255, 100, 100),
    Transparency = 0.5,
    AlwaysOnTop = true,
})
handle.CFrame = CFrame.new(...)
handle:Destroy()
```

---

## 15. Notifications (`notify`)

```luau
notify("Title", "Text", [duration], [sound])
notify({ Title = "...", Text = "...", Duration = 2.5, Sound = false })

ui.notify(...) / client.notify(...)   -- aliases
```

---

## 16. HTTP & Networking

All requests are routed through a secure server-side proxy (non-blocking, rate-limited, SSRF-protected).

### Classic

```luau
local code = game:HttpGet(url)
HttpGet(url) / http_get(url)
```

### High-level `http`

```luau
http.get(url, [headers])
http.post(url, [body], [contentType], [headers])
http.request(options) -> table
http.json_encode / http.json_decode
```

### Synapse-compatible

```luau
request({ Url = "...", Method = "POST", Headers = {...}, Body = "..." })
-- also: http_request, syn.request, client.request

-- Response:
-- Success, StatusCode, StatusMessage, Headers, Body, Error?
```

**Limits**:
- Per-player: burst 5, ~30 req/min
- Global: 120 req/min
- Body max 256 KB, response truncated at 1 MB

---

## 17. Chat & TextChatService

```luau
chat.send(message, [channelOrTeam])   -- true/"team" → team, false/"all" → AllChat
chat.say(message)
chat.team(message)
chat.all(message)
chat.get_channel(name) / chat.get_channels()
chat.on_message(callback)             -- (sender, text, isTeam, channelName, messageObj)

client.chat(message, [teamOnly])
client.say(message)

-- TextChatService enhanced
TextChatService:SendAsync(msg, [channel])
TextChatService:Chat(msg, [channel])
```

Events: `"chat_message"` / `"chat"`

---

## 18. Clantag System (`clantag`)

Server-authoritative, FE-safe, filtered, rate-limited.

```luau
clantag.set(tag, [anim_type], [speed])
-- anim_type: "static" | "marquee"/"scroll"/"running" | "typewriter"/"build" | "custom"
-- speed: 0.05..5.0 (default 0.35)

clantag.set_frames({ "frame1", "frame2", ... }, [speed])
clantag.reset()
clantag.get() -> string
clantag.format(playerOrName, [baseName]) -> string
```

Aliases: `client.set_clantag`, `set_clantag`, `client.reset_clantag`, etc.

Appears in: Killfeed, Spectator HUD, Scoreboard, Chat.  
**Never** affects Name ESP.

---

## 19. Live Lua Console

Menu → **Settings** → **Lua scripts** → **"Open lua console"**

- Stop output / Clear / Close
- Selectable + Ctrl+C copy with line numbers
- `print`, `warn`, `client.log`, `client.color_log(r,g,b,...)`, `client.error` all stream live

---

## 20. Example Scripts

### Brute-Force Resolver + HUD

```luau
local my_tab = menu.new_tab("Resolver", "rbxassetid://10734975692")
local sec = my_tab:new_section("Resolver Settings", "Left")
local hud_sec = my_tab:new_section("Display", "Right")

local enabled = sec:new_toggle("Enable Lua Resolver", true)
local baim_miss = sec:new_slider("BAim After Misses", 1, 5, 2, 0, " misses")
local show_hud = hud_sec:new_toggle("Show Debug HUD", true)

local targetState = {}
local function getState(slot)
    if not targetState[slot] then
        targetState[slot] = { misses = 0, stage = 1 }
    end
    return targetState[slot]
end

local BRUTE_ANGLES = { 58, -58, 0, 86, -86 }

client.set_event_callback("aim_hit", function(e)
    getState(e.target).misses = 0
end)

client.set_event_callback("aim_miss", function(e)
    if e.reason == "resolver" then
        local state = getState(e.target)
        state.misses += 1
        state.stage = (state.stage % #BRUTE_ANGLES) + 1
    end
end)

events.on_tick(function()
    if not enabled:get() then
        plist.reset_all()
        return
    end

    for _, enemy in ipairs(entity.get_enemies()) do
        if entity.is_alive(enemy) then
            local state = getState(entity.get_player_slot(enemy) or enemy)
            local offset = BRUTE_ANGLES[state.stage] or 0

            plist.set(enemy, "Force body yaw", true)
            plist.set(enemy, "Override yaw offset", offset)

            if state.misses >= baim_miss:get() then
                plist.set(enemy, "Override prefer body aim", "Force")
            else
                plist.set(enemy, "Override prefer body aim", "-")
            end
        end
    end
end)

client.set_event_callback("paint", function()
    if not show_hud:get() or not enabled:get() then return end
    local cur = rage.get_target()
    if cur and cur.slot then
        local state = getState(cur.slot)
        render.indicator(0, 255, 160, 255, string.format("RESOLVER: %d° | MISS: %d", BRUTE_ANGLES[state.stage] or 0, state.misses))
    else
        render.indicator(200, 200, 200, 255, "RESOLVER: ACTIVE")
    end
end)

client.set_event_callback("shutdown", function()
    plist.reset_all()
    render.clear()
end)
```

### Custom Anti-Aim (86° Desync)

```luau
local my_tab = menu.new_tab("Anti-Aim", "rbxassetid://10734975692")
local sec = my_tab:new_section("AA Settings", "Left")

local aa_toggle = sec:new_toggle("Enable Custom AA", true)
local pitch_sel = sec:new_combobox("Pitch", { "Down", "Up", "Zero" }, "Down")
local desync_lim = sec:new_slider("Desync Limit", 0, 86, 86, 0, "°")
local lag_ticks = sec:new_slider("Fake Lag", 0, 14, 12, 0, "t")
local invert_key = sec:new_keybind("Invert Side", "Toggle", "F")

events.on_tick(function()
    if not aa_toggle:get() then
        aa.reset()
        return
    end

    aa.override_pitch(pitch_sel:get())
    aa.override_yaw(180, "camera")
    aa.override_yaw_modifier(function(t, baseYaw)
        return math.sin(t * 0.2) * 35
    end)

    local limit = desync_lim:get()
    aa.override_body_yaw(true)
    aa.override_inverter(invert_key:get())
    aa.override_desync_limits(limit, limit)

    aa.override_fake_lag(true)
    aa.override_fake_lag_limit(lag_ticks:get())
end)

client.set_event_callback("paint", function()
    if not aa_toggle:get() then return end
    local side = aa.get_inverter() and "RIGHT" or "LEFT"
    local choking = aa.is_fake_lag_choking()
    render.indicator(240, 60, 70, 255, string.format("AA: 86° [%s] | CHOKE: %s", side, choking and "YES" or "NO"))
end)

client.set_event_callback("shutdown", function()
    aa.reset()
    render.clear()
end)
```

### Standalone Draggable GUI + Springs

```luau
local GUI_TOGGLE_KEY = "RightShift"

local window = Drawing.new("Square")
window.Size = vector(320, 240)
window.Position = vector(200, 200)
window.Color = color(20, 22, 28)
window.Rounding = 8
window.BorderColor = color(45, 50, 65)
window.ClipsDescendants = true
window.Visible = false

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
local is_gui_open = false

local drag_handle = window:MakeDraggable(header)

local function set_gui_open(open)
    is_gui_open = open
    window.Visible = open
    input.set_cursor_unlocked(open)

    if open then
        window.Position = vector(200, 180)
        tween.create(window, 0.35, {
            Position = vector(200, 200)
        }, { style = "back", direction = "out" }):play()
    end
end

input.on_key_press(GUI_TOGGLE_KEY, function()
    set_gui_open(not is_gui_open)
end)

client.set_event_callback("paint", function(dt)
    if not is_gui_open then return end

    if toggle_bg:IsClicked() then
        toggle_state = not toggle_state
        knob_spring:SetTarget(toggle_state and 292 or 268)
        tween.create(toggle_bg, 0.2, {
            Color = toggle_state and color(70, 130, 240) or color(35, 38, 48)
        }):play()
    end

    local current_x = knob_spring:Update(dt or 0.016)
    toggle_knob.Position = vector(current_x, 68)
end)

client.set_event_callback("shutdown", function()
    input.lock_cursor()
    if drag_handle and drag_handle.Destroy then
        drag_handle:Destroy()
    end
    window:Destroy()
end)
```

---

## License & Notes

This documentation is provided for educational and development purposes within the penablox RECODE ecosystem.  
All APIs are subject to change with engine updates.

**Security reminder**: Remote events/functions and dangerous services are permanently blocked by the sandbox.  
Scripts that attempt to call blocked operations will receive clean `lua violation` errors.

# Penablox Recode API — Documentation (Non-Visual Modules)

Complete reference for `rage`, `aa`, `entity`, `ui`, and core globals. Visual modules (`chams`, `render`, `Drawing`, etc.) are excluded per request.

> **Sandbox notes (verified):**
> - `require`, `loadstring`, `load`, `debug` — **not available**.
> - `game`/`workspace`/`script` are **emulated tables**, not real Roblox instances.
> - `Interpreter` is closed: `.Source` blocked (`PluginOrOpenCloud`), no public fields, `get/set/override` on UI elements crash with `stack overflow` (`Environment:1591/1592/1696`).
> - `ui.set("visuals", name, v)` accepts the call but **does not apply** values — UI elements are read-only from scripts.
> - `elem._value`, `elem._r/_g/_b/_a`, `elem._visible` are **readable**; `elem:get()` / `elem:set()` / `elem:override()` / `ui.get(elem)` / `ui.set(elem, v)` **crash**.
> - All functions are called with **dot notation** (`rage.set_enabled(true)`), not colon.

---

## Table of Contents
1. [Global Environment](#1-global-environment)
2. [rage](#2-rage)
3. [aa](#3-aa)
4. [entity](#4-entity)
5. [ui](#5-ui)
6. [Core Globals](#6-core-globals)

---

## 1. Global Environment

Sandbox exposes 96 globals. Non-visual ones:

| Global | Type | Purpose |
|---|---|---|
| `rage`, `aa`, `ui`, `entity` | table | Main API modules |
| `cheat`, `client`, `menu`, `cvar`, `database`, `events` | table | Client / config / events |
| `input`, `movement`, `weapon`, `sound`, `sounds` | table | Input / movement / weapons / audio |
| `players`, `plist` | table | Player lists |
| `game`, `workspace`, `script`, `Instance`, `Players`, `Workspace` | table | **Emulated** Roblox |
| `json`, `bit`, `bit32`, `buffer`, `utf8`, `os`, `math`, `string`, `table`, `coroutine` | table | Utilities |
| `getfenv`, `getgenv`, `getrenv`, `setfenv` | function | Env access |
| `print`, `warn`, `error`, `pcall`, `xpcall`, `assert` | function | Standard Lua |
| `tick`, `time`, `wait`, `delay`, `spawn`, `task` | function/table | Timing |
| `vector`, `color`, `notify` | function | Helpers |

---

## 2. rage

Rage bot / hitbox / resolver / double-tap.

### 2.1 Enable & State
| Function | Description |
|---|---|
| `rage.set_enabled(bool)` | Enable/disable rage |
| `rage.get_enabled()` | Returns enabled state |
| `rage.is_enabled()` | Alias of `get_enabled` |
| `rage.override_enabled(bool)` | Force-override enabled state |
| `rage.clear()` | Clear rage state |
| `rage.reset()` | Full reset |

### 2.2 Hitboxes
| Function | Description |
|---|---|
| `rage.set_hitboxes(tbl)` / `get_hitboxes()` | Set/get hitbox list |
| `rage.override_hitboxes(tbl)` | Override hitbox list |
| `rage.set_hitbox_config(cfg)` | Set full hitbox config |
| `rage.set_hitparts(tbl)` / `get_hitparts()` | Set/get hit parts |
| `rage.override_hitparts(tbl)` | Override hit parts |
| `rage.set_hitbox_min_dmg(n)` / `get_hitbox_min_dmg()` | Min damage per hitbox |
| `rage.set_hitbox_min_damage(n)` / `get_hitbox_min_damage()` | Alias |
| `rage.override_hitbox_min_dmg(n)` / `override_hitbox_min_damage(n)` | Override min dmg |
| `rage.set_hitbox_priority(n)` / `get_hitbox_priority()` | Hitbox priority |
| `rage.override_hitbox_priority(n)` | Override priority |
| `rage.set_hitbox_hitchance(n)` / `get_hitbox_hitchance()` | Per-hitbox hitchance |
| `rage.override_hitbox_hitchance(n)` | Override hitchance |

### 2.3 Minimum Damage / Hitchance / Priority
| Function | Description |
|---|---|
| `rage.set_min_damage(n)` / `get_min_damage()` | Global min damage |
| `rage.set_min_dmg(n)` / `get_min_dmg()` | Alias |
| `rage.override_min_damage(n)` / `override_min_dmg(n)` | Override min dmg |
| `rage.set_hitchance(n)` / `get_hitchance()` | Global hitchance |
| `rage.override_hitchance(n)` | Override hitchance |
| `rage.set_priority(n)` / `get_priority()` | Global priority |
| `rage.override_priority(n)` | Override priority |

### 2.4 Target Selection
| Function | Description |
|---|---|
| `rage.set_target_selection(mode)` / `get_target_selection()` | Set/get mode |
| `rage.override_target_selection(mode)` | Override mode |
| `rage.get_target()` | Current target entity |
| `rage.get_target_info()` | Detailed target info table |

### 2.5 Resolver
| Function | Description |
|---|---|
| `rage.set_resolver_enabled(bool)` / `is_resolver_enabled()` | Toggle |
| `rage.get_resolver_enabled()` | Alias |
| `rage.override_resolver_enabled(bool)` | Override |
| `rage.set_resolver_mode(mode)` / `get_resolver_mode()` | Set/get mode |
| `rage.override_resolver_mode(mode)` | Override mode |
| `rage.set_resolver_miss_limit(n)` / `get_resolver_miss_limit()` | Miss limit |
| `rage.set_resolver_baim_after_misses(n)` / `get_resolver_baim_after_misses()` | BAIM after N misses |
| `rage.reset_resolver()` | Reset resolver |
| `rage.reset_resolver_state()` | Reset resolver state |

### 2.6 Double Tap
| Function | Description |
|---|---|
| `rage.is_double_tap()` / `is_double_tap_active()` | DT active? |
| `rage.is_double_tap_charged()` / `is_dt_charged()` | DT charged? |
| `rage.get_dt_charge()` / `get_double_tap_charge()` | Current charge |

### 2.7 Force BAIM / Safe Point
| Function | Description |
|---|---|
| `rage.set_force_baim(bool)` / `is_force_baim()` | Toggle |
| `rage.force_baim()` | Trigger once |
| `rage.override_force_baim(bool)` | Override |
| `rage.set_force_safe_point(bool)` / `is_force_safe_point()` | Force safe point |
| `rage.force_safe_point()` | Trigger once |
| `rage.override_force_safe_point(bool)` | Override |

### 2.8 Multi-points / Shifting
| Function | Description |
|---|---|
| `rage.override_multipoints(tbl)` | Override multipoint config |
| `rage.force_shift()` | Force shift |
| `rage.shift_ticks(n)` | Shift N ticks |

### 2.9 Player Yaw / Pitch (rage-side)
| Function | Description |
|---|---|
| `rage.set_player_yaw(n)` / `override_player_yaw(n)` | Set/override yaw |
| `rage.set_player_yaw_offset(n)` / `override_player_yaw_offset(n)` | Yaw offset |
| `rage.reset_player_yaw()` | Reset yaw |
| `rage.set_player_pitch(n)` / `override_player_pitch(n)` | Set/override pitch |
| `rage.reset_player()` | Reset player state |

---

## 3. aa

Anti-aim / fake-lag / desync.

### 3.1 Yaw
| Function | Description |
|---|---|
| `aa.set_yaw(n)` / `get_yaw()` | Base yaw |
| `aa.override_yaw(n)` | Override yaw |
| `aa.is_yaw_overridden()` | Is overridden |
| `aa.set_yaw_base(mode)` / `get_yaw_base()` | Yaw base mode |
| `aa.override_yaw_base(mode)` | Override base |
| `aa.set_yaw_offset(n)` / `get_yaw_offset()` | Yaw offset |
| `aa.override_yaw_offset(n)` | Override offset |
| `aa.set_yaw_modifier(n)` / `get_yaw_modifier()` | Yaw modifier |
| `aa.override_yaw_modifier(n)` | Override modifier |

### 3.2 Pitch
| Function | Description |
|---|---|
| `aa.set_pitch(n)` / `get_pitch()` | Pitch |
| `aa.override_pitch(n)` | Override pitch |
| `aa.is_pitch_overridden()` | Is overridden |

### 3.3 Body Yaw / LBY / Feet Yaw
| Function | Description |
|---|---|
| `aa.set_body_yaw(n)` / `get_body_yaw()` | Body yaw |
| `aa.override_body_yaw(n)` | Override body yaw |
| `aa.is_body_yaw()` | Body yaw active? |
| `aa.get_lby()` / `get_lby_deg()` | Lower body yaw |
| `aa.get_lower_body_yaw()` / `get_lower_body_yaw_deg()` | Alias |
| `aa.get_feet_yaw()` / `get_feet_yaw_deg()` | Feet yaw |
| `aa.get_fake_lby()` / `get_fake_lby_deg()` | Fake LBY |
| `aa.get_fake_lower_body_yaw()` / `get_fake_lower_body_yaw_deg()` | Fake LB yaw |
| `aa.get_fake_feet_yaw()` / `get_fake_feet_yaw_deg()` | Fake feet yaw |
| `aa.get_twist()` / `get_twist_deg()` | Twist |
| `aa.get_fake_twist()` / `get_fake_twist_deg()` | Fake twist |

### 3.4 Desync
| Function | Description |
|---|---|
| `aa.set_desync(n)` / `get_desync()` | Desync amount |
| `aa.override_desync(n)` | Override desync |
| `aa.set_desync_limits(lim)` / `get_desync_limits()` | Limits |
| `aa.override_desync_limits(lim)` | Override limits |
| `aa.override_inverter(bool)` | Override inverter |
| `aa.set_inverter(bool)` / `get_inverter()` | Inverter state |

### 3.5 Fake Lag / Choke
| Function | Description |
|---|---|
| `aa.set_fake_lag(n)` / `get_fake_lag()` | Fake lag amount |
| `aa.override_fake_lag(n)` | Override fake lag |
| `aa.set_fake_lag_limit(n)` / `get_fake_lag_limit()` | Limit |
| `aa.override_fake_lag_limit(n)` | Override limit |
| `aa.set_fake_lag_variability(n)` / `get_fake_lag_variability()` | Variability |
| `aa.override_fake_lag_variability(n)` | Override variability |
| `aa.is_choking()` / `is_fake_lag_choking()` | Is choking? |
| `aa.get_choked()` / `get_choked_ticks()` / `get_choke_ticks()` | Choked ticks |
| `aa.get_choke_target()` | Choke target |
| `aa.unchoke()` | Release choke |
| `aa.flush_choked()` | Flush choked ticks |

### 3.6 Misc
| Function | Description |
|---|---|
| `aa.set_show_fake(bool)` / `get_show_fake()` | Show fake |
| `aa.is_show_fake()` / `override_show_fake(bool)` | Alias / override |
| `aa.get_turn_state()` | Turn state |
| `aa.get_display_feet_yaw()` | Display feet yaw |
| `aa.force_teleport()` | Force teleport |
| `aa.reset()` / `aa.clear()` | Reset / clear |

---

## 4. entity

Player / entity information.

### 4.1 Local Player
| Function | Description |
|---|---|
| `entity.get_local_player()` | Local player |
| `entity.get_local_state()` | Local state table |
| `entity.get_local_position()` | Local position |
| `entity.get_local_speed()` | Local speed |
| `entity.get_local_velocity()` | Local velocity |

### 4.2 Players & Enemies
| Function | Description |
|---|---|
| `entity.get_player()` | Player object |
| `entity.get_character()` | Character |
| `entity.get_players()` / `get_all_players()` | All players |
| `entity.get_enemies()` | Enemy list |
| `entity.get_teammates()` | Teammate list |
| `entity.get_player_team()` / `get_team()` | Team |

### 4.3 Health / Armor / Info
| Function | Description |
|---|---|
| `entity.get_health()` / `get_player_health()` | Health |
| `entity.get_armor()` / `get_player_armor()` | Armor |
| `entity.get_player_name()` | Name |
| `entity.get_player_info()` / `get_info()` | Info table |
| `entity.get_steam64()` | Steam64 |
| `entity.get_prop(name)` | Get property |

### 4.4 Positions & Angles
| Function | Description |
|---|---|
| `entity.get_position()` / `get_origin()` / `get_player_origin()` | Position |
| `entity.get_player_position()` | Player position |
| `entity.get_player_head_position()` / `get_head_position()` | Head |
| `entity.get_player_torso_position()` / `get_torso_position()` | Torso |
| `entity.get_hitbox_position()` / `get_player_hitbox_position()` | Hitbox |
| `entity.get_angles()` / `get_angles_deg()` | Angles |
| `entity.get_player_angles()` / `get_player_angles_deg()` | Player angles |
| `entity.get_bounding_box()` | Bounding box |
| `entity.get_delta()` | Delta |

### 4.5 Velocity / Speed
| Function | Description |
|---|---|
| `entity.get_velocity()` / `get_player_velocity()` | Velocity |
| `entity.get_speed()` / `get_player_speed()` | Speed |

### 4.6 Anim State / Yaw
| Function | Description |
|---|---|
| `entity.get_animstate()` / `get_player_animstate()` / `get_anim_state()` / `get_player_anim_state()` | Anim state |
| `entity.get_state()` / `get_player_state()` | State |
| `entity.get_lby()` / `get_lby_deg()` | LBY |
| `entity.get_lower_body_yaw()` / `get_lower_body_yaw_deg()` | LB yaw |
| `entity.get_player_lower_body_yaw()` / `get_player_lower_body_yaw_deg()` | Player LB yaw |
| `entity.get_feet_yaw()` / `get_feet_yaw_deg()` | Feet yaw |
| `entity.get_twist()` / `get_twist_deg()` | Twist |
| `entity.get_player_twist()` / `get_player_twist_deg()` | Player twist |
| `entity.get_desync()` / `get_desync_delta()` | Desync |

### 4.7 Weapon / Scope
| Function | Description |
|---|---|
| `entity.get_weapon()` / `get_weapon_name()` | Weapon |
| `entity.get_player_weapon()` / `get_player_weapon_name()` | Player weapon |
| `entity.get_scope_state()` / `get_player_scope_state()` | Scope state |
| `entity.get_scope_level()` / `get_player_scope_level()` | Scope level |

---

## 5. ui

Menu / element API.

### 5.1 Menu
| Function | Description |
|---|---|
| `ui.is_menu_open()` | Menu open? |
| `ui.set_visible(bool)` | Show/hide menu |
| `ui.menu_size()` | Menu size |
| `ui.menu_position()` | Menu position |
| `ui.mouse_position()` | Cursor position |

### 5.2 Element Creation
| Function | Signature |
|---|---|
| `ui.new_checkbox(tab, group, name)` | Checkbox |
| `ui.new_slider(tab, group, name, min, max, default)` | Slider |
| `ui.new_combobox(tab, group, name, items)` | Combo box |
| `ui.new_multiselect(tab, group, name, items)` | Multi-select |
| `ui.new_textbox(tab, group, name)` | Text box |
| `ui.new_label(tab, group, name)` | Label |
| `ui.new_listbox(tab, group, name)` | List box |
| `ui.new_button(tab, group, name, cb)` | Button |
| `ui.new_hotkey(tab, group, name)` | Hotkey |
| `ui.new_color_picker(tab, group, name, r,g,b,a)` | Color picker |

### 5.3 Element Access
| Function | Description |
|---|---|
| `ui.reference(tab, name)` | Get element by tab + name |
| `ui.set(name, value)` | ⚠️ **No-op** (accepts, doesn't apply) |
| `ui.set(tab, name, value)` | ⚠️ **No-op** |
| `ui.set(elem, value)` | ❌ **stack overflow** |
| `ui.get(elem)` | ❌ **stack overflow** |
| `ui.set_callback(elem, cb)` | Set callback |

### 5.4 Element Object (returned by `ui.reference`)
| Field | Type | Read | Write |
|---|---|---|---|
| `_type` | `"reference"` | ✅ | — |
| `_id` | string | ✅ | — |
| `_value` | any | ✅ | ⚠️ overwritten next tick |
| `_visible` | bool | ✅ | ⚠️ overwritten next tick |
| `_r`,`_g`,`_b`,`_a` | number | ✅ | ⚠️ overwritten next tick |
| `get()` | function | ❌ stack overflow | — |
| `set(v)` | function | ❌ stack overflow | — |
| `override(v)` | function | ❌ stack overflow | — |
| `set_visible(bool)` | function | ⚠️ untested | — |
| `depend(other)` | function | ✅ no error | — |
| `set_callback(fn)` | function | ✅ registers (fires only from engine) | — |

### 5.5 Notifications
| Function | Description |
|---|---|
| `ui.notify(text)` / `ui.notification(text)` | Toast notification |

---

## 6. Core Globals

### 6.1 Timing
| Function | Description |
|---|---|
| `tick()` / `time()` | Engine time |
| `wait(n)` | Yield N seconds |
| `delay(n, fn)` | Deferred call |
| `spawn(fn)` | Spawn coroutine |
| `task.wait(n)` / `task.spawn(fn)` / `task.delay(n, fn)` | Task API |

### 6.2 Environment
| Function | Description |
|---|---|
| `getfenv(level)` | Get env table (levels 0–3) |
| `getgenv()` | Get global env (same as `_G`) |
| `getrenv()` | Get Roblox env |
| `setfenv(level, tbl)` | Set env |

### 6.3 Utilities
| Function | Description |
|---|---|
| `json.encode(t)` / `json.decode(s)` | JSON |
| `bit.*` / `bit32.*` | Bit operations |
| `buffer.*` | Buffer API |
| `utf8.*` | UTF-8 |
| `math.*`, `string.*`, `table.*`, `os.*`, `coroutine.*` | Standard Lua |
| `vector(x,y,z)` | Vector helper |
| `color(r,g,b,a)` | Color helper |
| `notify(text)` | Short notification alias |

---

## 7. Usage Notes & Known Issues

1. **All module calls use dot notation:** `rage.set_enabled(true)`, not `rage:set_enabled(true)`.
2. **Element getters/setters are broken** in this build (`Environment:1591/1592/1696`). Read fields directly (`elem._value`), never call `elem:get()` / `elem:set()` / `ui.get()` / `ui.set(elem, v)`.
3. **UI elements are read-only from scripts** — `ui.set(name, v)` accepts but does not apply. Change menu state via the UI itself.
4. **`Interpreter` is inaccessible** — no `require`, `.Source` blocked, no public fields. Use public API modules.
5. **`game` / `workspace` / `script` are emulated** — `:FindFirstChild()` works, but returned objects may not support Roblox methods like `.Source`.
6. **`aa.get_yaw()` may return `nil`** when AA is inactive — normal.
7. **`entity.get_players()` returns local player** — in solo it's `1`.

---

## 8. Quick Examples

### Enable rage with defaults
```lua
rage.set_enabled(true)
rage.set_min_damage(70)
rage.set_hitchance(60)
rage.set_hitboxes({ "head", "upper_chest", "stomach" })
```

### Configure anti-aim
```lua
aa.set_yaw(180)
aa.set_pitch(89)
aa.set_desync(60)
aa.set_fake_lag(10)
```

### Loop: local player info
```lua
task.spawn(function()
    while true do
        local lp = entity.get_local_player()
        local hp = entity.get_health()
        print(string.format("[LP] %s | HP: %s", tostring(lp), tostring(hp)))
        task.wait(1)
    end
end)
```

### Loop: enemy list
```lua
task.spawn(function()
    while true do
        local enemies = entity.get_enemies()
        for i, e in ipairs(enemies) do
            print("[ENEMY]", i, entity.get_player_name(), entity.get_health())
        end
        task.wait(0.5)
    end
end)
```

### Add a menu checkbox (creation works; setting it from script does not)
```lua
local cb = ui.new_checkbox("Rage", "Main", "Auto DT")
ui.set_callback(cb, function(state)
    rage.set_enabled(state)
end)
```

### Read a UI element safely
```lua
local ok, elem = pcall(ui.reference, "visuals", "enemies")
if ok and type(elem) == "table" then
    print("enemies enabled:", elem._value)     -- ✅ read
    print("element id:", elem._id)             -- ✅ read
    -- elem:set(false)                         -- ❌ stack overflow
end
```

---

**Summary:** Penablox Recode exposes `rage`, `aa`, `entity`, `ui` plus core globals. `rage`/`aa`/`entity` are fully functional (verified). `ui` element access is **read-only** from scripts due to a recursion bug in `Environment` — read `_value`/`_r/_g/_b/_a`/`_visible` directly, never call get/set/override. `Interpreter` is closed and cannot be inspected.

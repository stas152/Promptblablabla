# Penablox Recode — API Reference (verified)

> Documented from live testing (250/250 calls executed, 0 errors). Where behavior differs from the inferred signature, the caveat is noted.

---

## Table of Contents

1. [Conventions](#conventions)
2. [Global Utilities](#global-utilities)
3. [`rage` — Aimbot / Ragebot](#rage--aimbot--ragebot)
4. [`aa` — Anti-Aim](#aa--anti-aim)
5. [`entity` — Player & World](#entity--player--world)
6. [`ui` — Interface Elements](#ui--interface-elements)
7. [Known Missing / Stub Functions](#known-missing--stub-functions)

---

## Conventions

- **Do not use dot-syntax for anything other than what's listed here** — the engine resolves `aa.set_yaw` etc. through a metatable and `require`-style lookup, not Lua libraries.
- `set_*` / `get_*` / `override_*` triplets share the same value semantics: `set_*` writes the config, `override_*` forces the value at runtime, `get_*` reads the current effective value.
- `is_*` — boolean shorthand for `get_*`.
- Angles are stored in **radians** internally. `*_deg` variants return degrees but **are not normalized to ±180** — apply `value % 360` yourself if needed.
- `pcall` **cannot detect stub getters** (they return `nil` silently). See [Stub Functions](#known-missing--stub-functions).

---

## Global Utilities

### JSON

The global `json` table exists but **is empty**. Use Roblox's `HttpService` instead:

```lua
local HttpService = game:GetService("HttpService")
HttpService:JSONEncode({a = 1, b = 2})  -->  '{"a":1,"b":2}'
HttpService:JSONDecode('{"a":1}')       -->  table
```

### Available Globals

| Name | Return | Notes |
|---|---|---|
| `tick()` | number | Unix epoch seconds |
| `time()` | number | Seconds since game start |
| `wait(n)` | number | Yields `n` seconds, returns actual elapsed |
| `getgenv()` | table | Script-side globals |
| `getrenv()` | table | Engine/realm-side globals |
| `getfenv(0)` | table | Same as `getgenv()` in script context |
| `vector(x, y, z)` | vector | Luau `vector` type |
| `color(r, g, b, a)` | color | Returns `r/255, g/255, b/255` normalized |
| `notify(msg)` | nil | Native toast notification |
| `bit.band(a, b)` | number | Bitwise AND (Luau builtin) |
| `bit32.band(a, b)` | number | Bitwise AND (Lua 5.2 compat) |
| `utf8.char(cp)` | string | UTF-8 codepoint → string |

---

## `rage` — Aimbot / Ragebot

### Enable / State

| Function | Args | Returns | Notes |
|---|---|---|---|
| `rage.set_enabled(bool)` | boolean | nil | |
| `rage.get_enabled()` | — | boolean | |
| `rage.is_enabled()` | — | boolean | Same as `get_enabled` |
| `rage.override_enabled(bool)` | boolean | nil | Forces value; resets on `rage.reset()` |
| `rage.clear()` | — | nil | Clears current target state |
| `rage.reset()` | — | nil | Full reset (clears all overrides) |

### Hitboxes

| Function | Args | Returns | Notes |
|---|---|---|---|
| `rage.set_hitboxes(list)` | `{string}` | nil | e.g. `{"head","stomach"}` |
| `rage.get_hitboxes()` | — | `{string}` | |
| `rage.override_hitboxes(list)` | `{string}` | nil | |
| `rage.set_hitbox_config(tbl)` | table | nil | `{min_damage = 50}` |
| `rage.set_hitparts(list)` | `{string}` | nil | |
| `rage.get_hitparts()` | — | `{string}` | |
| `rage.override_hitparts(list)` | `{string}` | nil | |
| `rage.set_hitbox_min_dmg(n)` | number | nil | |
| `rage.get_hitbox_min_dmg()` | — | **nil** | *stub — see note below* |
| `rage.set_hitbox_min_damage(n)` | number | nil | Alias of `set_hitbox_min_dmg` |
| `rage.get_hitbox_min_damage()` | — | **nil** | *stub* |
| `rage.override_hitbox_min_dmg(n)` | number | nil | |
| `rage.override_hitbox_min_damage(n)` | number | nil | Alias |
| `rage.set_hitbox_priority(n)` | number | nil | |
| `rage.get_hitbox_priority()` | — | **nil** | *stub* |
| `rage.override_hitbox_priority(tbl)` | **table** | nil | ⚠️ Takes `{head=1, chest=2}`, **not** a number |
| `rage.set_hitbox_hitchance(n)` | number | nil | |
| `rage.get_hitbox_hitchance()` | — | **nil** | *stub* |
| `rage.override_hitbox_hitchance(n)` | number | nil | |

> **Getters returning `nil`**: `get_hitbox_min_dmg`, `get_hitbox_min_damage`, `get_hitbox_priority`, `get_hitbox_hitchance` are stubs. Read actual per-hitbox config through `rage.get_hitboxes()`.

### Min Damage / Hitchance / Priority

| Function | Args | Returns | Notes |
|---|---|---|---|
| `rage.set_min_damage(n)` | number | nil | ✅ verified `set(70)` → `get()` = 70 |
| `rage.get_min_damage()` | — | number | |
| `rage.set_min_dmg(n)` | number | nil | Alias |
| `rage.get_min_dmg()` | — | number | Alias |
| `rage.override_min_damage(n)` | number | nil | |
| `rage.override_min_dmg(n)` | number | nil | Alias |
| `rage.set_hitchance(n)` | number | nil | ✅ verified `set(60)` → `get()` = 60 |
| `rage.get_hitchance()` | — | number | |
| `rage.override_hitchance(n)` | number | nil | |
| `rage.override_priority(tbl)` | **table** | nil | ⚠️ Takes `{head=1, chest=2}`, **not** a number |

> ❌ **`rage.set_priority` and `rage.get_priority` do not exist.** Use `override_priority({...})`.

### Target Selection

| Function | Args | Returns | Notes |
|---|---|---|---|
| `rage.set_target_selection(s)` | string | nil | `"crosshair"` \| `"distance"` \| `"fov"` |
| `rage.get_target_selection()` | — | string | Default: `"crosshair"` |
| `rage.override_target_selection(s)` | string | nil | |
| `rage.get_target()` | — | userdata \| nil | `nil` if no target |
| `rage.get_target_info()` | — | table \| nil | `nil` if no target |

### Resolver

| Function | Args | Returns | Notes |
|---|---|---|---|
| `rage.set_resolver_enabled(bool)` | boolean | nil | |
| `rage.is_resolver_enabled()` | — | boolean | Default: `true` |
| `rage.get_resolver_enabled()` | — | boolean | |
| `rage.override_resolver_enabled(bool)` | boolean | nil | |
| `rage.set_resolver_mode(s)` | string | nil | `"default"` \| others |
| `rage.get_resolver_mode()` | — | string | Default: `"default"` |
| `rage.override_resolver_mode(s)` | string | nil | |
| `rage.set_resolver_miss_limit(n)` | number | nil | |
| `rage.get_resolver_miss_limit()` | — | number | Default: `3` |
| `rage.set_resolver_baim_after_misses(n)` | number | nil | |
| `rage.get_resolver_baim_after_misses()` | — | number | Default: `2` |
| `rage.reset_resolver()` | — | nil | |
| `rage.reset_resolver_state()` | — | nil | |

### Double Tap

| Function | Returns | Notes |
|---|---|---|
| `rage.is_double_tap()` | boolean | |
| `rage.is_double_tap_active()` | boolean | |
| `rage.is_double_tap_charged()` | boolean | |
| `rage.is_dt_charged()` | boolean | Alias |
| `rage.get_dt_charge()` | number | 0–1 charge ratio |
| `rage.get_double_tap_charge()` | number | Alias |

### Force BAIM / Safe Point

| Function | Args | Returns |
|---|---|---|
| `rage.set_force_baim(bool)` | boolean | nil |
| `rage.is_force_baim()` | — | boolean |
| `rage.force_baim()` | — | nil — one-shot force |
| `rage.override_force_baim(bool)` | boolean | nil |
| `rage.set_force_safe_point(bool)` | boolean | nil |
| `rage.is_force_safe_point()` | — | boolean |
| `rage.force_safe_point()` | — | nil |
| `rage.override_force_safe_point(bool)` | boolean | nil |

### Multi-points / Shifting

| Function | Args | Returns | Notes |
|---|---|---|---|
| `rage.override_multipoints(n)` | **number** | nil | ⚠️ Takes a number, **not** a table |
| `rage.force_shift()` | — | nil | |
| `rage.shift_ticks(n)` | number | nil | |

### Player Yaw / Pitch

| Function | Args | Returns |
|---|---|---|
| `rage.set_player_yaw(deg)` | number | nil |
| `rage.override_player_yaw(deg)` | number | nil |
| `rage.set_player_yaw_offset(deg)` | number | nil |
| `rage.override_player_yaw_offset(deg)` | number | nil |
| `rage.reset_player_yaw()` | — | nil |
| `rage.set_player_pitch(deg)` | number | nil |
| `rage.override_player_pitch(deg)` | number | nil |
| `rage.reset_player()` | — | nil |

---

## `aa` — Anti-Aim

### Yaw

| Function | Args | Returns | Notes |
|---|---|---|---|
| `aa.set_yaw(deg)` | number | nil | |
| `aa.get_yaw()` | — | number \| nil | `nil` if inactive |
| `aa.override_yaw(deg)` | number | nil | |
| `aa.is_yaw_overridden()` | — | boolean | |
| `aa.set_yaw_base(s)` | string | nil | `"local"` \| `"camera"` |
| `aa.get_yaw_base()` | — | string | Default: `"camera"` |
| `aa.override_yaw_base(s)` | string | nil | |
| `aa.set_yaw_offset(deg)` | number | nil | |
| `aa.get_yaw_offset()` | — | number | |
| `aa.override_yaw_offset(deg)` | number | nil | |
| `aa.set_yaw_modifier(n)` | number | nil | |
| `aa.override_yaw_modifier(n)` | number | nil | |

> ❌ **`aa.get_yaw_modifier` does not exist.**

### Pitch

| Function | Args | Returns | Notes |
|---|---|---|---|
| `aa.set_pitch(deg)` | number | nil | ⚠️ clamped — `set(89)` → `get()` = 70 |
| `aa.get_pitch()` | — | number | Default: `70` |
| `aa.override_pitch(deg)` | number | nil | |
| `aa.is_pitch_overridden()` | — | boolean | |

### Body Yaw / LBY / Feet / Twist

| Function | Returns | Notes |
|---|---|---|
| `aa.set_body_yaw(deg)` | nil | |
| `aa.get_body_yaw()` | number | |
| `aa.override_body_yaw(deg)` | nil | |
| `aa.is_body_yaw()` | boolean | |
| `aa.get_lby()` | number (radians) | |
| `aa.get_lby_deg()` | number | ⚠️ Not normalized to ±180 |
| `aa.get_lower_body_yaw()` | number | Alias of `get_lby` |
| `aa.get_lower_body_yaw_deg()` | number | Alias |
| `aa.get_feet_yaw()` | number | |
| `aa.get_feet_yaw_deg()` | number | |
| `aa.get_fake_lby()` | number | |
| `aa.get_fake_lby_deg()` | number | |
| `aa.get_fake_lower_body_yaw()` | number | |
| `aa.get_fake_lower_body_yaw_deg()` | number | |
| `aa.get_fake_feet_yaw()` | number | |
| `aa.get_fake_feet_yaw_deg()` | number | |
| `aa.get_twist()` | number (radians) | |
| `aa.get_twist_deg()` | number | |
| `aa.get_fake_twist()` | number (radians) | |

> ❌ **`aa.get_fake_twist_deg` does not exist.** Compute: `aa.get_fake_twist() * 180 / math.pi`.

### Desync

| Function | Args | Returns | Notes |
|---|---|---|---|
| `aa.set_desync(n)` | number | nil | |
| `aa.override_desync(n)` | number | nil | |
| `aa.set_desync_limits(n)` | number | nil | |
| `aa.get_desync_limits()` | — | `number, number` | Returns **two** values: `(current, max)` |
| `aa.override_desync_limits(n)` | number | nil | |
| `aa.set_inverter(bool)` | boolean | nil | |
| `aa.get_inverter()` | — | boolean | |
| `aa.override_inverter(bool)` | boolean | nil | |

> ❌ **`aa.get_desync` does not exist.** Use `entity.get_desync()`.

### Fake Lag / Choke

| Function | Args | Returns | Notes |
|---|---|---|---|
| `aa.set_fake_lag(n)` | number | nil | |
| `aa.get_fake_lag()` | — | number \| boolean | `false` if disabled |
| `aa.override_fake_lag(n)` | number | nil | |
| `aa.set_fake_lag_limit(n)` | number | nil | |
| `aa.get_fake_lag_limit()` | — | number | |
| `aa.override_fake_lag_limit(n)` | number | nil | |
| `aa.set_fake_lag_variability(n)` | number | nil | |
| `aa.get_fake_lag_variability()` | — | number | |
| `aa.override_fake_lag_variability(n)` | number | nil | |
| `aa.is_choking()` | — | boolean | |
| `aa.is_fake_lag_choking()` | — | boolean | |
| `aa.get_choked()` | — | number | |
| `aa.get_choked_ticks()` | — | number | |
| `aa.get_choke_ticks()` | — | number | Alias |
| `aa.get_choke_target()` | — | number | Tick count, not entity ID |
| `aa.unchoke()` | — | nil | |
| `aa.flush_choked()` | — | nil | |

### Misc

| Function | Args | Returns |
|---|---|---|
| `aa.set_show_fake(bool)` | boolean | nil |
| `aa.get_show_fake()` | — | boolean |
| `aa.is_show_fake()` | — | boolean |
| `aa.override_show_fake(bool)` | boolean | nil |
| `aa.get_turn_state()` | — | boolean |
| `aa.get_display_feet_yaw()` | — | number |
| `aa.force_teleport()` | — | nil |
| `aa.reset()` | — | nil |
| `aa.clear()` | — | nil |

---

## `entity` — Player & World

### Local Player

| Function | Returns |
|---|---|
| `entity.get_local_player()` | string \| userdata |
| `entity.get_local_state()` | string (`"Stand"`, `"Crouch"`, …) |
| `entity.get_local_position()` | vector |
| `entity.get_local_speed()` | number |
| `entity.get_local_velocity()` | vector |

### Players

| Function | Returns |
|---|---|
| `entity.get_player()` | userdata (local) |
| `entity.get_character()` | userdata (local) |
| `entity.get_players()` | table |
| `entity.get_all_players()` | table |
| `entity.get_enemies()` | table |
| `entity.get_teammates()` | table |
| `entity.get_player_team()` | string (`"CT"`, `"T"`) |
| `entity.get_team()` | string |

### Health / Info

| Function | Returns |
|---|---|
| `entity.get_health()` | number |
| `entity.get_player_health()` | number |
| `entity.get_armor()` | number |
| `entity.get_player_armor()` | number |
| `entity.get_player_name()` | string |
| `entity.get_player_info()` | table |
| `entity.get_info()` | table |
| `entity.get_steam64()` | number |

### Positions

| Function | Returns |
|---|---|
| `entity.get_player_origin()` | vector |
| `entity.get_player_position()` | vector |
| `entity.get_player_head_position()` | vector |
| `entity.get_head_position()` | vector |
| `entity.get_player_torso_position()` | vector |
| `entity.get_torso_position()` | vector |
| `entity.get_hitbox_position()` | vector |
| `entity.get_player_hitbox_position()` | vector |
| `entity.get_angles()` | number (radians) |
| `entity.get_angles_deg()` | number |
| `entity.get_player_angles()` | number |
| `entity.get_player_angles_deg()` | number |
| `entity.get_bounding_box()` | number (volume, not extent) |
| `entity.get_delta()` | number |

### Velocity / Speed

| Function | Returns |
|---|---|
| `entity.get_velocity()` | vector |
| `entity.get_player_velocity()` | vector |
| `entity.get_speed()` | number |
| `entity.get_player_speed()` | number |

### Anim State / Yaw

| Function | Returns |
|---|---|
| `entity.get_animstate()` | table |
| `entity.get_player_animstate()` | table |
| `entity.get_anim_state()` | table |
| `entity.get_player_anim_state()` | table |
| `entity.get_state()` | string |
| `entity.get_player_state()` | string |
| `entity.get_lby()` | number (radians) |
| `entity.get_lby_deg()` | number |
| `entity.get_lower_body_yaw()` | number |
| `entity.get_lower_body_yaw_deg()` | number |
| `entity.get_player_lower_body_yaw()` | number |
| `entity.get_player_lower_body_yaw_deg()` | number |
| `entity.get_feet_yaw()` | number |
| `entity.get_feet_yaw_deg()` | number |
| `entity.get_twist()` | number |
| `entity.get_twist_deg()` | number |
| `entity.get_player_twist()` | number |
| `entity.get_player_twist_deg()` | number |
| `entity.get_desync()` | number |
| `entity.get_desync_delta()` | number |

> **`*_deg` values are not normalized.** Example: `get_lby_deg()` returned `-1263.37`, which is `-1263.37 % 360 ≈ 176.63°`.

### Weapon / Scope

| Function | Returns |
|---|---|
| `entity.get_weapon()` | string |
| `entity.get_weapon_name()` | string |
| `entity.get_player_weapon()` | string |
| `entity.get_player_weapon_name()` | string |
| `entity.get_scope_state()` | number |
| `entity.get_player_scope_state()` | number |
| `entity.get_scope_level()` | number |
| `entity.get_player_scope_level()` | number |

---

## `ui` — Interface Elements

### Menu / Globals

| Function | Returns |
|---|---|
| `ui.is_menu_open()` | boolean |
| `ui.set_visible(bool)` | nil |
| `ui.menu_size()` | number |
| `ui.menu_position()` | number |
| `ui.mouse_position()` | number |

### Element Creation

All constructors return an element userdata. Elements are auto-registered on the tab.

| Function | Signature | Notes |
|---|---|---|
| `ui.new_checkbox(tab, group, name)` | → element | |
| `ui.new_slider(tab, group, name, min, max, default)` | → element | |
| `ui.new_combobox(tab, group, name, items)` | → element | |
| `ui.new_multiselect(tab, group, name, items)` | → element | |
| `ui.new_textbox(tab, group, name)` | → element | |
| `ui.new_label(tab, group, name)` | → element | |
| `ui.new_listbox(tab, group, name)` | → element | |
| `ui.new_button(tab, group, name, callback)` | → element | |
| `ui.new_hotkey(tab, group, name)` | → element | |
| `ui.new_color_picker(tab, group, name, r, g, b, a)` | → element | |

### Element Access

| Function | Returns | Notes |
|---|---|---|
| `ui.reference(tab, group, name)` | element \| nil | Finds an existing element |
| `elem._value` | any | Raw value (safe to read) |
| `elem._id` | string | Internal ID, e.g. `"ui_9"` |
| `ui.set_callback(elem, fn)` | nil | Registers callback (fires when UI changes) |
| `ui.set(name, value)` | nil | Sets by name — **may not apply immediately** |

> ⚠️ **Do not call `elem:get()`, `elem:set()`, or `elem:override()` directly** — these cause a **stack overflow** in the current build. Read `elem._value` instead, and use `ui.set(name, v)` or the module-specific setters (`rage.set_min_damage`, etc.).

---

## Known Missing / Stub Functions

These appear in some external docs or were inferred, but **are not implemented** in the current build:

### Missing (call raises "Attempt to call a nil value")

| Function | Replacement |
|---|---|
| `aa.get_yaw_modifier` | none — read via `ui.reference("AA", "Yaw", "Modifier")._value` if that element exists |
| `aa.get_desync` | `entity.get_desync()` |
| `aa.get_fake_twist_deg` | `aa.get_fake_twist() * 180 / math.pi` |
| `rage.set_priority` | `rage.override_priority({...})` |
| `rage.get_priority` | none |
| `json.encode` / `json.decode` | `HttpService:JSONEncode` / `HttpService:JSONDecode` |

### Stubs (callable, but return `nil`)

| Function | Workaround |
|---|---|
| `rage.get_hitbox_min_dmg` | read from `rage.get_hitboxes()` |
| `rage.get_hitbox_min_damage` | same |
| `rage.get_hitbox_priority` | same |
| `rage.get_hitbox_hitchance` | same |

### Type Mismatches (signature corrected)

| Function | Wrong call | Correct call |
|---|---|---|
| `rage.override_hitbox_priority` | `(2)` → throws `ipairs(table expected)` | `({head=1, chest=2})` |
| `rage.override_priority` | `(1)` → throws | `({head=1, chest=2})` |
| `rage.override_multipoints` | `({head=2})` → throws `clamp(number expected)` | `(2)` |

---

## Test Status

Verified against Penablox Recode (build `22:53:28`):

```
=== DONE : 250 OK, 0 FAIL ===
```

All 250 documented calls execute without runtime errors. Semantic caveats above (`*_deg` normalization, pitch clamping, stub getters) are **behavioral**, not failures.

# PenaBlox HvH — шпаргалка по API

Собрано из дампов `_G` в F9. `(?)` — видели в таблице, но не проверяли.
`(!)` — ломает `RageEngine` / не работает.

## Глобалы
print, warn, error, assert, type, typeof, tostring, tonumber, pairs, ipairs,
next, select, unpack, pcall, xpcall, spawn, delay, wait, tick, time,
_VERSION, _G, bit, bit32, string, table, math, os, coroutine, task

## aa — анти-аим
pitch: set_pitch (рад), get_pitch (град, кламп ±70)
yaw: set_yaw_offset, get_yaw_offset, set_yaw, get_yaw, set_yaw_base,
  get_yaw_base, override_yaw_base, override_yaw, override_yaw_offset,
  set_yaw_modifier, override_yaw_modifier, is_yaw_overridden
ноги/lby/twist: get_feet_yaw (рад), get_feet_yaw_deg, get_lby_deg, get_twist,
  get_twist_deg, get_lower_body_yaw, get_lower_body_yaw_deg,
  get_display_feet_yaw, get_fake_feet_yaw, get_fake_feet_yaw_deg,
  get_fake_lby, get_fake_lby_deg, get_fake_twist,
  get_fake_lower_body_yaw, get_fake_lower_body_yaw_deg
body yaw (!): set_body_yaw, get_body_yaw, override_body_yaw, is_body_yaw
fake lag: set_fake_lag, get_fake_lag, set_fake_lag_limit, get_fake_lag_limit,
  set_fake_lag_variability, get_fake_lag_variability, override_fake_lag,
  override_fake_lag_limit, override_fake_lag_variability
desync (!): set_desync_limits, override_desync_limits, override_desync
choke: get_choked, get_turn_state, get_choke_target, flush_choked, unchoke
прочее: set_inverter, get_inverter, override_show_fake, get_show_fake,
  reset, clear, force_teleport (эффекта нет)

## rage — аимбот
вкл: set_enabled, override_enabled, get_enabled
точность: override_hitchance, set_hitbox_hitchance, override_hitbox_hitchance,
  get_hitbox_hitchance, override_multipoints (число!)
урон: set_min_dmg, override_min_dmg, override_min_damage,
  set_hitbox_min_dmg, set_hitbox_min_damage, override_hitbox_min_dmg,
  override_hitbox_min_damage, get_hitbox_min_dmg, get_hitbox_min_damage,
  get_min_damage
хитбоксы: set_hitboxes, get_hitboxes, override_hitboxes,
  set_hitbox_priority, get_hitbox_priority, override_priority,
  override_hitbox_priority, override_hitparts, get_hitparts, set_hitparts
цель: get_target_selection, set_target_selection, override_target_selection,
  get_target, get_target_info, set_force_baim, is_force_baim,
  override_force_baim, set_force_safe_point, is_force_safe_point, force_safe_point
резолвер: get_resolver_mode, set_resolver_mode, override_resolver_mode,
  is_resolver_enabled, override_resolver_enabled, reset_resolver_state,
  reset_player, reset_player_yaw, get_resolver_miss_limit,
  set_resolver_baim_after_misses
double tap: is_double_tap, is_double_tap_active, is_double_tap_charged,
  is_dt_charged, get_dt_charge, get_double_tap_charge, shift_ticks
yaw игрока: set_player_yaw, set_player_yaw_offset, override_player_yaw

## entity
игроки: get_local_player, get_players, get_all_players, get_teammates,
  is_enemy, is_dormant, get_player_info, get_player_name, get_delta
позиция: get_player_origin (x,y,z), get_player_angles (pitch,yaw рад),
  get_player_angles_deg, get_player_position, get_player_head_position,
  get_head_position, get_player_torso_position, get_torso_position,
  get_hitbox_position, get_player_hitbox_position, get_character
скорость: get_player_speed, get_local_speed, get_local_velocity,
  get_player_velocity, get_velocity
углы: get_lby, get_lby_deg, get_player_lby, get_player_lby_deg, get_twist,
  get_twist_deg, get_player_twist, get_feet_yaw, get_lower_body_yaw,
  get_lower_body_yaw_deg, get_player_lower_body_yaw,
  get_player_lower_body_yaw_deg, get_desync_delta, get_angles,
  get_anim_state, get_player_anim_state
состояние: is_in_air, is_moving, is_standing, is_crouching, is_slow_walking,
  is_alive, get_player_state, get_local_state, get_player_health,
  get_player_armor, get_player_team, is_team
оружие: get_weapon, get_weapon_name, get_player_weapon, get_player_weapon_name
снайперка: is_player_sniper, get_scope_state, get_player_scope_state,
  get_scope_level, get_player_scope_level

## client
get_angles, get_local_player, get_players, get_local_velocity,
get_local_speed, get_weapon_name, is_enemy, get_teammates, is_dormant,
get_character, get_player_origin, get_player_head_position,
get_player_health, get_player_name, get_player_lby, get_lby_deg,
get_feet_yaw_deg, get_player_torso_position, get_player_info, get_delta,
get_esp_data

## events
on_tick, get_tick, get_tick_count, on, remove_listener, at_tick,
every_ticks, delay_ticks

## task / os
task.spawn, task.delay, task.defer, task.cancel, task.wait
os.clock, os.time, os.difftime, os.date

## menu
new_tab, get_or_create_section, new_checkbox, new_combobox, new_slider,
new_hotkey, find, get, set, set_callback, set_visible

## input
on_key_press, on_input_began, on_input_ended, lock_cursor, unlock_cursor,
toggle_cursor, set_cursor_unlocked

## renderer / Drawing
draw_rect, rectangle_filled, rect_outline, rect_gradient, draw_gradient,
gradient, line, create_quad, create_triangle, cindicator, measure_text,
draw_image, load_image, image, texture, clear, is_cursor_unlocked,
toggle_cursor

## color / Color3
fromHex, fromHSV, toHSV, fromRGB, new

## Vector2 / Vector3 / vector
new, fromAxis, FromAxis, fromNormalId, FromNormalId, max, min, zero, one,
xAxis, yAxis, zAxis

## sounds
set_killsound_override, set_hitsound_override, play, play_killsound,
play_hitsound, override_hitsound, reset_killsound, presets

## cheat
on, register_callback

## данные
json, base64, plist, database

## string
split, match, gmatch, upper, gsub, format, lower, sub, pack, find, char,
packsize, reverse, byte, unpack, rep, len

## Что НЕ работает клиентски
- nospread, rapid fire, КД — сервер.
- Регистрация попаданий — сервер (фейк-модель не влияет).
- force_teleport — вызов проходит, эффекта нет.
- get_player_health — только чтение, изменить HP нельзя.

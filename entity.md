# Документация модуля `entity` — Penablox Recode

## Общее

Модуль `entity` — API игроков/сущностей в Penablox Recode. Доступен через **`getgenv().entity`**. Работает с **`Instance`** игроков (не строками).

```lua
local entity = getgenv().entity
```

### Ключевое поведение

- **Без аргумента** геттеры читают **локального игрока**.
- **С аргументом-`Instance`** — указанного игрока.
- **`get_players()`** возвращает массив **`Instance`**, не строк.
- **`get_info(player)`** и **`get_animstate(player)`** возвращают **градусы** (`yaw`, `pitch`, `lby`, `desync`), а геттеры `get_player_angles` / `get_player_lby` — **радианы**.

### Соглашения

- **`get_*`** — читает значение.
- **`is_*`** — булев шорткат.
- **`get_*_deg`** — градусы, **не нормализованы** к ±180 (применяйте `% 360` сами).
- **`set_prop`** — сеттер (не проверен).
- **Стабы** — функции, которые всегда возвращают `nil` (см. раздел «Не работает»).

## Списки игроков

| Функция | Возвращает | Примечание |
|---|---|---|
| `get_players()` | `{Instance, ...}` | динамический список (13–14 игроков) |
| `get_all_players()` | `{Instance, ...}` | алиас `get_players` |
| `get_enemies()` | `{Instance, ...}` | подмножество |
| `get_teammates()` | `{Instance, ...}` | подмножество |
| `get_local_player()` | `Instance` | локальный игрок |
| `get_player()` | `Instance` | локальный (без аргумента) |

## Данные игрока

Геттеры принимают `Instance`; без аргумента — локальный игрок.

| Функция | Возвращает | Примечание |
|---|---|---|
| `get_player_name(p)` | string | имя |
| `get_player_health(p)` | number | здоровье |
| `get_player_armor(p)` | number | броня |
| `get_player_team(p)` | string | команда (пустая в этой игре) |
| `get_player_weapon(p)` | table | структура оружия (12 полей) |
| `get_player_weapon_name(p)` | string | `"Scout"`, `"Knife"` |
| `get_player_origin(p)` | Vector3 | позиция (origin) |
| `get_player_position(p)` | Vector3 / `nil` | **`nil` для dormant**, `(0,-10000,0)` для мёртвых |
| `get_player_head_position(p)` | Vector3 | голова (Y +1.5 от position) |
| `get_player_torso_position(p)` | Vector3 | торс (= position) |
| `get_player_speed(p)` | number | скорость |
| `get_player_velocity(p)` | Vector3 | вектор скорости |
| `get_player_state(p)` | string | `"Stand"`, `"Falling"` |
| `get_player_angles(p)` | number | yaw (радианы) |
| `get_player_angles_deg(p)` | number | yaw (градусы) |
| `get_player_lby(p)` | number | LBY (радианы) |
| `get_player_lby_deg(p)` | number | LBY (градусы) |
| `get_player_lower_body_yaw(p)` | number | lower body yaw (радианы) |
| `get_player_lower_body_yaw_deg(p)` | number | градусы |
| `get_player_feet_yaw(p)` | number | feet yaw (радианы) |
| `get_player_feet_yaw_deg(p)` | number | градусы |
| `get_player_twist(p)` | number | twist (радианы) |
| `get_player_twist_deg(p)` | number | градусы |
| `get_player_scope_state(p)` | number | состояние прицела |
| `get_player_scope_level(p)` | number | уровень прицела |
| `get_player_animstate(p)` | table | animstate (28 полей) |
| `get_player_anim_state(p)` | table | алиас |
| `get_player_info(p)` | table | полная информация (22 поля) |
| `get_player_hitbox_position(p, idx)` | Vector3 / `nil` | хитбокс (см. ниже) |
| `get_player_state(p)` | string | состояние |
| `get_steam64(p)` | string | Steam64 ID |
| `get_character(p)` | Instance | персонаж |

## Универсальные геттеры (без префикса `player_`)

Эквивалентны версиям с `player_`:

| Без префикса | С префиксом |
|---|---|
| `get_health` | `get_player_health` |
| `get_armor` | `get_player_armor` |
| `get_team` | `get_player_team` |
| `get_weapon` | `get_player_weapon` |
| `get_weapon_name` | `get_player_weapon_name` |
| `get_origin` | `get_player_origin` |
| `get_position` | `get_player_position` |
| `get_head_position` | `get_player_head_position` |
| `get_torso_position` | `get_player_torso_position` |
| `get_speed` | `get_player_speed` |
| `get_velocity` | `get_player_velocity` |
| `get_state` | `get_player_state` |
| `get_angles` / `_deg` | `get_player_angles` / `_deg` |
| `get_lby` / `_deg` | `get_player_lby` / `_deg` |
| `get_lower_body_yaw` / `_deg` | `get_player_lower_body_yaw` / `_deg` |
| `get_feet_yaw` / `_deg` | `get_player_feet_yaw` / `_deg` |
| `get_twist` / `_deg` | `get_player_twist` / `_deg` |
| `get_animstate` / `get_anim_state` | `get_player_animstate` / `get_player_anim_state` |
| `get_info` | `get_player_info` |
| `get_scope_state` / `get_scope_level` | `get_player_scope_state` / `_level` |

## Локальный игрок

| Функция | Возвращает |
|---|---|
| `get_local_player()` | `Instance` |
| `get_local_state()` | string (`"Falling"`) |
| `get_local_position()` | Vector3 |
| `get_local_velocity()` | Vector3 |
| `get_local_speed()` | number |

## Флаги (`is_*`)

Принимают `Instance`; без аргумента — локальный.

| Функция | Возвращает | Примечание |
|---|---|---|
| `is_alive(p)` | boolean | жив |
| `is_enemy(p)` | boolean | враг |
| `is_dormant(p)` | boolean | dormant |
| `is_crouching(p)` / `is_ducking(p)` | boolean | присел |
| `is_standing(p)` | boolean | стоит |
| `is_moving(p)` | boolean | движется |
| `is_in_air(p)` / `in_air(p)` | boolean | в воздухе |
| `is_slow_walking(p)` | boolean | медленно идёт |
| `is_scoped(p)` / `is_player_scoped(p)` | boolean | в прицеле |
| `is_sniper(p)` / `is_player_sniper(p)` | boolean | снайпер |

## Хитбоксы

| Функция | Сигнатура | Возвращает | Примечание |
|---|---|---|---|
| `get_hitbox_position(p, index)` | `(player, number)` | Vector3 / `nil` | индексы `0`, `8`, `9`, `10` → **торс**; `1–7` → `nil` |
| `get_player_hitbox_position(p, index)` | `(player, number)` | Vector3 / `nil` | то же |
| `hitbox_position(p, index)` | `(player, number)` | Vector3 / `nil` | **алиас** `get_hitbox_position` |

> **Важно:** `get_hitbox_position` **не даёт голову**. Для наведения в голову используйте **`get_player_head_position(p)`**. Функция возвращает `nil`, если хитбоксы не загружены (dormant/мёртв).

## ESP

| Функция | Сигнатура | Возвращает | Примечание |
|---|---|---|---|
| `get_esp_data(p)` | `(player)` | table | `{x1, y1, x2, y2, alpha}` — экранный bounding box; `alpha = 0` — не на экране |

## Структуры таблиц

### `get_info(player)` — 22 поля (градусы!)

| Поле | Тип | Пример |
|---|---|---|
| `name` | string | `"C_Kast"` |
| `player` | Instance | `C_Kast` |
| `character` | Instance | `C_Kast` |
| `health` | number | `100` |
| `armor` | number | `41` |
| `position` | Vector3 | `(-230.5, 840.5, -542.9)` |
| `velocity` | Vector3 | `(0,0,0)` |
| `speed` | number | `0` |
| `yaw` | number | `114.63` (**градусы**) |
| `pitch` | number | `-70.16` (**градусы**) |
| `lby` | number | `31.01` (**градусы**) |
| `roll` | number | `0` |
| `desync_delta` | number | `83.62` (**градусы**) |
| `state` | string | `"Stand"` |
| `weapon` | table | `Knife` |
| `weapon_name` | string | `"Knife"` |
| `is_enemy` | boolean | `false` |
| `is_alive` | boolean | `true` |
| `is_standing` | boolean | `true` |
| `is_moving` | boolean | `false` |
| `is_crouching` | boolean | `false` |
| `is_ducking` | boolean | `false` |
| `is_in_air` | boolean | `false` |
| `is_scoped` | boolean | `false` |

### `get_animstate(player)` — 28 полей (градусы!)

| Поле | Тип | Пример |
|---|---|---|
| `yaw` | number | `101.39` |
| `pitch` | number | `-70.16` |
| `eye_yaw` | number | `101.39` |
| `eye_pitch` | number | `-70.16` |
| `eye_angles_x` | number | `-70.16` |
| `eye_angles_y` | number | `101.39` |
| `lby` | number | `17.77` |
| `goal_feet_yaw` | number | `17.77` |
| `current_feet_yaw` | number | `17.77` |
| `feet_yaw` | number | `17.77` |
| `torso_yaw` | number | `17.77` |
| `twist` | number | `83.62` |
| `desync` | number | `83.62` |
| `desync_delta` | number | `83.62` |
| `duck_amount` | number | `1` |
| `velocity` | Vector3 | `(0,0,0)` |
| `velocity_x` | number | `0` |
| `velocity_y` | number | `0` |
| `velocity_z` | number | `0` |
| `m_velocity` | number | `0` |
| `speed` | number | `0` |
| `stop_to_full_running_fraction` | number | `0` |
| `feet_speed_forwards_or_sideways` | number | `0` |
| `on_ground` | boolean | `true` |
| `is_in_air` | boolean | `false` |
| `is_ducking` | boolean | `true` |
| `is_crouching` | boolean | `true` |

### `get_weapon(player)` — 12 полей

| Поле | Тип | Пример |
|---|---|---|
| `id` | string | `"Scout"` |
| `name` | string | `"SSG 08"` |
| `type` | string | `"SniperRifle"` |
| `category` | string | `"Primary"` |
| `slot` | number | `1` |
| `ammo` | number | `10` |
| `reserve` | number | `90` |
| `scope_level` | number | `0` |
| `is_sniper` | boolean | `true` |
| `is_scoped` | boolean | `false` |
| `is_knife` | boolean | `false` |
| `is_grenade` | boolean | `false` |

## Состояния игрока

| Состояние | `health` | `position` | Что делать |
|---|---|---|---|
| Живой, в мире | `> 0` | `Vector3(x, y, z)` | использовать |
| Мёртвый | `0` | `Vector3(0, -10000, 0)` | **пропустить** (заглушка) |
| Dormant (живой) | `100` | **`nil`** | **пропустить** (нет позиции) |

## Не работает / не использовать

| Функция | Статус | Замена |
|---|---|---|
| `get_prop(name)` | всегда `nil` | `get_info(p)` |
| `get_bounding_box(p)` | возвращает `0` | `get_esp_data(p)` |
| `animstate(p)` / `anim_state(p)` | не алиасы `get_animstate` | `get_animstate(p)` |
| `get_hitbox_position` (голова) | голова не возвращается (только торс) | `get_player_head_position(p)` |

## Пример использования

```lua
local entity = getgenv().entity

-- Фильтр валидных целей (живых, в мире, врагов)
local function get_valid_targets(only_enemies)
    local result = {}
    for _, p in ipairs(entity.get_players()) do
        if not entity.is_alive(p) then continue end
        if entity.get_player_health(p) <= 0 then continue end
        local pos = entity.get_player_position(p)
        if not pos then continue end
        if pos.Y < -1000 then continue end
        if only_enemies and not entity.is_enemy(p) then continue end
        result[#result + 1] = { player = p, position = pos }
    end
    return result
end

-- Наведение в голову
for _, t in ipairs(get_valid_targets(true)) do
    local p = t.player
    local name = entity.get_player_name(p)
    local head = entity.get_player_head_position(p)   -- голова
    local info = entity.get_info(p)                   -- 22 поля, градусы
    local anim = entity.get_animstate(p)              -- 28 полей, градусы
    local weapon = entity.get_weapon(p)               -- 12 полей
    print(name, head, info.health, anim.yaw, weapon.name)
end

-- Локальный игрок
print("local:", entity.get_local_player())
print("local hp:", entity.get_health())               -- без аргумента
print("local pos:", entity.get_local_position())
print("local state:", entity.get_local_state())       -- "Falling"

-- ESP (только если игрок на экране)
for _, t in ipairs(get_valid_targets(false)) do
    local esp = entity.get_esp_data(t.player)
    if esp and esp.alpha > 0 then
        local w, h = esp.x2 - esp.x1, esp.y2 - esp.y1
        print("ESP box:", esp.x1, esp.y1, w, h, "alpha:", esp.alpha)
    end
end
```

## Ограничения

- `get_hitbox_position` возвращает только торс (`0/8/9/10`); головы нет.
- `get_prop` и `get_bounding_box` не работают.
- `animstate` / `anim_state` — не алиасы `get_animstate` (не используйте).
- `get_info` и `get_animstate` возвращают **градусы**, геттеры `get_player_angles` / `get_player_lby` — **радианы**.
- `get_player_position` возвращает `nil` для dormant и `(0, -10000, 0)` для мёртвых.
- `get_team` возвращает пустую строку (команды не используются).
- `debug.info` / `getmetatable` / `rawget` недоступны.
- Каждый скрипт — в отдельном окружении; переменные не сохраняются между запусками.

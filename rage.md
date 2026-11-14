# Документация модуля `rage` — Penablox Recode

## Общее

Модуль `rage` — API аимбота (aimbot/ragebot) в Penablox Recode. Доступен через **`getgenv().rage`** (не через `getrenv()`). Резолвится через метатаблицу, но фактически является **плоской таблицей функций** — полностью перечислим обычным `pairs`.

```lua
local rage = getgenv().rage
```

### Соглашения API

- **`set_*`** — пишет значение в конфиг (сохраняется).
- **`override_*`** — форсит значение только в рантайме (не сохраняется).
- **`get_*`** — читает текущее эффективное значение.
- **`is_*`** — булев шорткат для `get_*` (возвращает `true`/`false`).
- **Алиасы** (`min_dmg`/`min_damage`, `dt_charge`/`double_tap_charge`) — взаимозаменяемы.
- **Углы** — в **радианах**; `*_deg`-суффикса в модуле нет. Нормализация к ±180 — вручную:
  ```lua
  local function norm_rad(a) return ((a + math.pi) % (2*math.pi)) - math.pi end
  ```
- **Стаб-геттеры** — существуют, вызываются без ошибки, но **всегда возвращают `nil`**. `pcall` их не отлавливает.

## Статусы проверки

| Статус | Значение |
|---|---|
| **OK** | геттер рабочий, вернул осмысленное значение |
| **STUB** | геттер-заглушка, всегда `nil` |
| **STUB/нет цели** | `nil` по смыслу (нет цели), не стаб |
| **present** | функция присутствует, но не вызывалась (сеттер/оверрайд/служебная) |

## Полная таблица API

| Категория | Функция | Тип | Статус проверки | Результат | Примечание |
|---|---|---|---|---|---|
| **Состояние** | `set_enabled` | setter | present | — | пишет в конфиг |
| | `get_enabled` | getter | OK | `false` (boolean) | рабочий |
| | `is_enabled` | is_shortcut | OK | `false` (boolean) | рабочий |
| | `override_enabled` | override | present | — | форс в рантайме |
| | `reset` | service | present | — | общий сброс |
| | `clear` | service | present | — | очистка |
| **Цель** | `set_target_selection` | setter | present | — | пишет в конфиг |
| | `get_target_selection` | getter | OK | `"Highest damage"` (string) | рабочий |
| | `override_target_selection` | override | present | — | форс в рантайме |
| | `get_target` | getter | STUB/нет цели | `nil` | `nil` = нет цели, не стаб |
| | `get_target_info` | getter | STUB/нет цели | `nil` | `nil` = нет цели, не стаб |
| **Урон** | `set_min_damage` / `set_min_dmg` | setter | present | — | алиасы |
| | `get_min_damage` / `get_min_dmg` | getter | OK | `0` (number) | рабочий |
| | `override_min_damage` / `override_min_dmg` | override | present | — | алиасы |
| **Хитшанс** | `set_hitchance` | setter | present | — | пишет в конфиг |
| | `get_hitchance` | getter | OK | `0` (number) | рабочий |
| | `override_hitchance` | override | present | — | форс в рантайме |
| **Хитбоксы: хитшанс** | `set_hitbox_hitchance` | setter | present | — | пишет в конфиг |
| | `get_hitbox_hitchance` | getter | STUB | `nil` | стаб |
| | `override_hitbox_hitchance` | override | present | — | форс в рантайме |
| **Хитбоксы: мин. урон** | `set_hitbox_min_damage` / `set_hitbox_min_dmg` | setter | present | — | алиасы |
| | `get_hitbox_min_damage` / `get_hitbox_min_dmg` | getter | STUB | `nil` | стаб |
| | `override_hitbox_min_damage` / `override_hitbox_min_dmg` | override | present | — | алиасы |
| **Хитбоксы: приоритет** | `set_hitbox_priority` | setter | present | — | пишет в конфиг |
| | `get_hitbox_priority` | getter | STUB | `nil` | стаб |
| | `override_hitbox_priority` / `override_priority` | override | present | — | алиасы |
| **Хитбоксы: список** | `set_hitboxes` | setter | present | — | пишет в конфиг |
| | `get_hitboxes` | getter | STUB | `nil` | стаб |
| | `override_hitboxes` | override | present | — | форс в рантайме |
| **Хитбоксы: конфиг** | `set_hitbox_config` | setter | present | — | зонтичный сеттер |
| **Хитпарты** | `set_hitparts` | setter | present | — | пишет в конфиг |
| | `get_hitparts` | getter | STUB | `nil` | стаб |
| | `override_hitparts` | override | present | — | форс в рантайме |
| **Мультипоинты** | `override_multipoints` | override | present | — | форс в рантайме |
| **Резолвер: вкл** | `set_resolver_enabled` | setter | present | — | пишет в конфиг |
| | `get_resolver_enabled` | getter | OK | `true` (boolean) | рабочий |
| | `is_resolver_enabled` | is_shortcut | OK | `true` (boolean) | рабочий |
| | `override_resolver_enabled` | override | present | — | форс в рантайме |
| **Резолвер: режим** | `set_resolver_mode` | setter | present | — | пишет в конфиг |
| | `get_resolver_mode` | getter | OK | `"Freestanding"` (string) | рабочий |
| | `override_resolver_mode` | override | present | — | форс в рантайме |
| **Резолвер: лимит промахов** | `set_resolver_miss_limit` | setter | present | — | пишет в конфиг |
| | `get_resolver_miss_limit` | getter | OK | `1` (number) | рабочий |
| **Резолвер: BAIM после промахов** | `set_resolver_baim_after_misses` | setter | present | — | пишет в конфиг |
| | `get_resolver_baim_after_misses` | getter | OK | `0` (number) | рабочий |
| **Резолвер: сброс** | `reset_resolver` | service | present | — | сброс резолвера |
| | `reset_resolver_state` | service | present | — | сброс состояния |
| **Double Tap** | `is_double_tap` | is_shortcut | OK | `false` (boolean) | рабочий |
| | `is_double_tap_active` | is_shortcut | OK | `false` (boolean) | рабочий |
| | `is_double_tap_charged` | is_shortcut | OK | `false` (boolean) | рабочий |
| | `is_dt_charged` | is_shortcut | OK | `false` (boolean) | алиас |
| | `get_dt_charge` / `get_double_tap_charge` | getter | OK | `0` (number) | алиасы, рабочий |
| **Shift** | `force_shift` | service | present | — | форс шифта |
| | `shift_ticks` | service | present | — | тики шифта |
| **BAIM** | `set_force_baim` | setter | present | — | пишет в конфиг |
| | `is_force_baim` | is_shortcut | OK | `false` (boolean) | рабочий |
| | `override_force_baim` | override | present | — | форс в рантайме |
| | `force_baim` | service | present | — | форс BAIM |
| **Safe Point** | `set_force_safe_point` | setter | present | — | пишет в конфиг |
| | `is_force_safe_point` | is_shortcut | OK | `false` (boolean) | рабочий |
| | `override_force_safe_point` | override | present | — | форс в рантайме |
| | `force_safe_point` | service | present | — | форс safe point |
| **Углы: yaw** | `set_player_yaw` | setter | present | — | радианы |
| | `override_player_yaw` | override | present | — | радианы |
| | `reset_player_yaw` | service | present | — | сброс yaw |
| **Углы: yaw offset** | `set_player_yaw_offset` | setter | present | — | радианы |
| | `override_player_yaw_offset` | override | present | — | радианы |
| **Углы: pitch** | `set_player_pitch` | setter | present | — | радианы |
| | `override_player_pitch` | override | present | — | радианы |
| **Сброс игрока** | `reset_player` | service | present | — | сброс состояния игрока |

## Сводка по статусам

### Рабочие геттеры (OK)

```
get_enabled              -> false (boolean)
get_min_damage / _min_dmg -> 0 (number)
get_hitchance            -> 0 (number)
get_target_selection     -> "Highest damage" (string)
get_resolver_enabled     -> true (boolean)
get_resolver_mode        -> "Freestanding" (string)
get_resolver_miss_limit  -> 1 (number)
get_resolver_baim_after_misses -> 0 (number)
get_dt_charge / get_double_tap_charge -> 0 (number)
```

### Рабочие `is_*` (все OK)

```
is_enabled
is_resolver_enabled
is_double_tap / is_double_tap_active / is_double_tap_charged / is_dt_charged
is_force_baim
is_force_safe_point
```

### Стабы (всегда `nil`)

```
get_hitbox_hitchance
get_hitbox_min_damage / get_hitbox_min_dmg
get_hitbox_priority
get_hitboxes
get_hitparts
```

### Не стабы, а рабочее `nil`

```
get_target        -- нет цели
get_target_info   -- нет цели
```

## Практические рекомендации

1. **Не читайте стаб-геттеры** (`get_hitbox_*`, `get_hitboxes`, `get_hitparts`) — они всегда `nil`. Для чтения состояния хитбоксов ведите **свой конфиг**.
2. **`get_target` / `get_target_info`** проверяйте на `nil` — это «нет цели», а не стаб.
3. **Углы — радианы.** Нормализация:
   ```lua
   local function norm_rad(a) return ((a + math.pi) % (2*math.pi)) - math.pi end
   ```
4. **`set_*` vs `override_*`:** `set_*` пишет в конфиг (сохраняется), `override_*` — только на рантайм. Для временного вмешательства используйте `override_*`, затем `reset_*`.
5. **Стаб-геттеры `pcall` не отлавливает** — проверяйте результат на `nil` вручную.

## Пример использования

```lua
local rage = getgenv().rage

-- Свой конфиг хитбоксов (геттеры-стабы не помогут)
local Hitbox = {
    hitchance = 60,
    min_damage = 20,
    priority = "Head",
    hitboxes = {"Head", "UpperTorso"},
    hitparts = {"Head"},
}

-- Применение конфига (сеттеры присутствуют)
rage.set_hitbox_hitchance(Hitbox.hitchance)
rage.set_hitbox_min_damage(Hitbox.min_damage)
rage.set_hitbox_priority(Hitbox.priority)
rage.set_hitboxes(Hitbox.hitboxes)
rage.set_hitparts(Hitbox.hitparts)

-- Общие настройки аимбота
rage.set_enabled(true)
rage.set_min_damage(25)
rage.set_hitchance(70)
rage.set_target_selection("Highest damage")

-- Резолвер
rage.set_resolver_enabled(true)
rage.set_resolver_mode("Freestanding")
rage.set_resolver_miss_limit(2)
rage.set_resolver_baim_after_misses(3)

-- Чтение рабочих геттеров
print("enabled:", rage.get_enabled())               -- true
print("min dmg:", rage.get_min_damage())            -- 25
print("hitchance:", rage.get_hitchance())           -- 70
print("resolver:", rage.get_resolver_mode())        -- "Freestanding"
print("target:", rage.get_target())                 -- nil или цель

-- Форс BAIM на рантайм (не сохраняется)
rage.override_force_baim(true)
-- ... действие ...
rage.reset_player()  -- сброс состояния
```

## Ограничения и известные проблемы

- `getmetatable` **недоступен** в окружении Penablox Recode — не используйте его для рефлексии `rage`.
- Метатаблица модуля ничего не добавляет: все ключи видны через `pairs`.
- Каждый скрипт выполняется в **отдельном окружении** — переменные не сохраняются между запусками.
- `json` в глобалах пуст — используйте `HttpService:JSONEncode` / `:JSONDecode`.
- Стаб-геттеры неотличимы от рабочего `nil` без контекста (см. `get_target`).

# Документация модуля `aa` — Penablox Recode

## Общее

Модуль `aa` — API anti-aim в Penablox Recode. Доступен через **`getgenv().aa`** (не через `getrenv()`). Является плоской таблицей функций — перечислим обычным `pairs`.

```lua
local aa = getgenv().aa
```

### Соглашения API

- **`set_*`** — пишет значение в конфиг (сохраняется).
- **`override_*`** — форсит значение только в рантайме (не сохраняется).
- **`get_*`** — читает текущее эффективное значение.
- **`is_*`** — булев шорткат для `get_*` (возвращает `true`/`false`).
- **`*_deg`** — возвращают градусы, но не нормализуют к ±180 (применяйте `% 360` сами).
- **Углы без `_deg`** — в радианах.
- **Стаб-геттеры** — существуют, вызываются без ошибки, но всегда возвращают `nil`. `pcall` их не отлавливает.

### Отличия `aa` от `rage`

| Особенность | `rage` | `aa` |
|---|---|---|
| `set_enabled` / `get_enabled` / `is_enabled` | есть | нет |
| `*_deg`-геттеры | нет | есть |
| Стабы | хитбоксы (`get_hitbox_*`, `get_hitboxes`, `get_hitparts`) | основные yaw/pitch (`get_yaw`, `get_pitch`, `get_yaw_offset`) |
| Строковые геттеры | `get_target_selection`, `get_resolver_mode` | `get_yaw_base` |

У `aa` нет явного вкл/выкл. Anti-aim либо всегда активен, либо управляется через `set_yaw`/`set_pitch` (например, `set_yaw(nil)` = выкл) или `reset`.

## Полная таблица API

| Категория | Функция | Тип | Примечание |
|---|---|---|---|
| **Служебные** | `reset` | service | общий сброс |
| | `clear` | service | очистка |
| **Yaw (основной)** | `set_yaw` | setter | радианы |
| | `get_yaw` | getter | стаб |
| | `override_yaw` | override | радианы |
| | `is_yaw_overridden` | is_shortcut | — |
| **Yaw base** | `set_yaw_base` | setter | строка-режим |
| | `get_yaw_base` | getter | строка-режим (не угол) |
| | `override_yaw_base` | override | форс в рантайме |
| **Yaw offset** | `set_yaw_offset` | setter | радианы |
| | `get_yaw_offset` | getter | стаб |
| | `override_yaw_offset` | override | радианы |
| **Yaw modifier** | `set_yaw_modifier` | setter | пишет в конфиг |
| | `override_yaw_modifier` | override | форс в рантайме |
| | *get_yaw_modifier* | — | геттера нет |
| **Pitch** | `set_pitch` | setter | радианы |
| | `get_pitch` | getter | стаб |
| | `override_pitch` | override | радианы |
| | `is_pitch_overridden` | is_shortcut | — |
| **Body yaw** | `set_body_yaw` | setter | пишет в конфиг |
| | `get_body_yaw` | getter | флаг (не угол) |
| | `override_body_yaw` | override | форс в рантайме |
| | `is_body_yaw` | is_shortcut | — |
| **Inverter** | `set_inverter` | setter | пишет в конфиг |
| | `get_inverter` | getter | — |
| | `override_inverter` | override | форс в рантайме |
| **Desync** | `set_desync` | setter | пишет в конфиг |
| | `override_desync` | override | форс в рантайме |
| | *get_desync* | — | геттера нет |
| **Desync limits** | `set_desync_limits` | setter | пишет в конфиг |
| | `get_desync_limits` | getter | — |
| | `override_desync_limits` | override | форс в рантайме |
| **LBY** | `get_lby` | getter | радианы |
| | `get_lby_deg` | getter | градусы |
| | `get_lower_body_yaw` | getter | радианы |
| | `get_lower_body_yaw_deg` | getter | градусы |
| **Fake LBY** | `get_fake_lby` | getter | радианы |
| | `get_fake_lby_deg` | getter | градусы, не нормализованы |
| | `get_fake_lower_body_yaw` | getter | радианы |
| | `get_fake_lower_body_yaw_deg` | getter | градусы, не нормализованы |
| **Feet yaw** | `get_feet_yaw` | getter | радианы |
| | `get_feet_yaw_deg` | getter | градусы |
| | `get_display_feet_yaw` | getter | радианы |
| **Fake feet yaw** | `get_fake_feet_yaw` | getter | радианы |
| | `get_fake_feet_yaw_deg` | getter | градусы, не нормализованы |
| **Twist** | `get_twist` | getter | радианы |
| | `get_twist_deg` | getter | градусы |
| | `get_fake_twist` | getter | радианы |
| **Choke** | `get_choked` | getter | текущий choke |
| | `get_choked_ticks` | getter | тики |
| | `get_choke_ticks` | getter | алиас `get_choked_ticks` |
| | `get_choke_target` | getter | целевые тики |
| | `is_choking` | is_shortcut | — |
| | `unchoke` | service | снять choke |
| | `flush_choked` | service | сбросить choke |
| **Fake lag** | `set_fake_lag` | setter | пишет в конфиг |
| | `get_fake_lag` | getter | — |
| | `override_fake_lag` | override | форс в рантайме |
| | `set_fake_lag_limit` | setter | пишет в конфиг |
| | `get_fake_lag_limit` | getter | — |
| | `override_fake_lag_limit` | override | форс в рантайме |
| | `set_fake_lag_variability` | setter | пишет в конфиг |
| | `get_fake_lag_variability` | getter | — |
| | `override_fake_lag_variability` | override | форс в рантайме |
| | `is_fake_lag_choking` | is_shortcut | — |
| **Show fake** | `set_show_fake` | setter | пишет в конфиг |
| | `get_show_fake` | getter | — |
| | `override_show_fake` | override | форс в рантайме |
| | `is_show_fake` | is_shortcut | — |
| **Turn state** | `get_turn_state` | getter | — |
| **Телепорт** | `force_teleport` | service | форс телепорта |

## Стаб-геттеры (всегда `nil`)

```
get_yaw
get_pitch
get_yaw_offset
```

Итоговый угол anti-aim намеренно скрыт; производные (LBY, feet, twist) доступны для чтения.

## Отсутствующие геттеры

```
get_yaw_modifier   -- есть только set_/override_
get_desync         -- есть только set_/override_ + get_desync_limits
```

## Строковые геттеры

| Функция | Значение | Примечание |
|---|---|---|
| `get_yaw_base` | `"camera"` | режим базы yaw, не угол |
| `get_body_yaw` | `true` / `false` | флаг, не угол |

## Аномалии

1. **`get_body_yaw()` → boolean, не угол.** Это флаг «body yaw включён». Для угла тела используйте `get_lby` / `get_lower_body_yaw`.
2. **`get_yaw_base()` → строка `"camera"`, не угол.** База yaw — режим (`"camera"`, вероятно также `"local"`, `"target"`, `"movement"`).
3. **`*_deg` не нормализованы.** Пример: `get_fake_lby_deg()` → `-146.05` (вне ±180). Нормализуйте сами:
   ```lua
   local function norm_deg(d) return ((d + 180) % 360) - 180 end
   ```
4. **`get_desync` отсутствует.** Читается только `get_desync_limits()` (лимит, не текущее значение).
5. **`get_yaw_modifier` отсутствует.** Только `set_yaw_modifier` / `override_yaw_modifier`.

## Практические рекомендации

1. **Не читайте `get_yaw` / `get_pitch` / `get_yaw_offset`** — стабы, всегда `nil`.
2. **Для чтения углов используйте производные:**
   - `get_lby()` / `get_lby_deg()` — нижняя часть тела;
   - `get_feet_yaw()` / `get_feet_yaw_deg()` — ступни;
   - `get_twist()` / `get_twist_deg()` — скручивание;
   - `get_fake_lby()` / `get_fake_lby_deg()` — фейковый LBY;
   - `get_fake_feet_yaw()` / `get_fake_feet_yaw_deg()` — фейковые ступни.
3. **`get_body_yaw()` — флаг, не угол.** Для угла тела — `get_lby`.
4. **`get_yaw_base()` — строка-режим**, не угол.
5. **`*_deg` нормализуйте сами:** `((d + 180) % 360) - 180`.
6. **Для записи** используйте `set_yaw` / `set_pitch` / `set_yaw_base` / `set_yaw_offset` (все `present`).
7. **`get_desync` нет** — читайте лимиты через `get_desync_limits()`.
8. **`get_yaw_modifier` нет** — только `set_` и `override_`.
9. **`set_enabled` / `get_enabled` / `is_enabled` отсутствуют** — anti-aim управляется через `set_yaw`/`set_pitch` или `reset`.

## Пример использования

```lua
local aa = getgenv().aa

-- Нормализация градусов (для *_deg-геттеров)
local function norm_deg(d) return ((d + 180) % 360) - 180 end

-- Запись углов (радианы)
local yaw_rad   = math.rad(90)    -- 90 градусов в радианах
local pitch_rad = math.rad(-20)   -- -20 градусов в радианах

aa.set_yaw(yaw_rad)
aa.set_pitch(pitch_rad)
aa.set_yaw_base("camera")         -- строка-режим
aa.set_yaw_offset(math.rad(10))

-- Inverter
aa.set_inverter(true)

-- Desync
aa.set_desync(true)
aa.set_desync_limits(84)

-- Fake lag
aa.set_fake_lag(true)
aa.set_fake_lag_limit(14)
aa.set_fake_lag_variability(15)

-- Show fake
aa.set_show_fake(false)

-- Чтение рабочих геттеров
print("lby (rad):",   aa.get_lby())
print("lby (deg):",   norm_deg(aa.get_lby_deg()))
print("feet (deg):",  norm_deg(aa.get_feet_yaw_deg()))
print("twist (deg):", norm_deg(aa.get_twist_deg()))
print("fake lby:",    norm_deg(aa.get_fake_lby_deg()))
print("choked:",      aa.get_choked())
print("choke target:",aa.get_choke_target())
print("fake lag:",    aa.get_fake_lag())
print("inverter:",    aa.get_inverter())
print("yaw base:",    aa.get_yaw_base())

-- Стабы: всегда nil (не используйте для чтения)
-- aa.get_yaw() / aa.get_pitch() / aa.get_yaw_offset() -> nil

-- Служебные
aa.unchoke()        -- снять choke
aa.flush_choked()   -- сбросить choke
aa.reset()          -- общий сброс
```

## Ограничения

- **`getmetatable` недоступен** в окружении Penablox Recode.
- **`rawget` недоступен** — проверка «физического» наличия ключа невозможна; используйте обычное обращение.
- Метатаблица модуля ничего не добавляет: все ключи видны через `pairs`.
- Каждый скрипт выполняется в отдельном окружении — переменные не сохраняются между запусками.
- `json` в глобалах пуст — используйте `HttpService:JSONEncode` / `:JSONDecode`.
- Арность `set_*` / `override_*` не проверена (вызывать опасно). Для безопасной проверки используйте `debug.info`, если доступен:
  ```lua
  local ok, params = pcall(debug.info, aa.set_yaw, "a")
  ```
- Допустимые значения строковых режимов (`yaw_base`) не задокументированы — уточняйте через UI или экспериментально.

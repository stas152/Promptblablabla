# Документация модуля `ui` — Penablox Recode

## Общее

Модуль `ui` — API интерфейса в Penablox Recode. Доступен через **`getgenv().ui`**. Плоская таблица функций — перечислима обычным `pairs`.

```lua
local ui = getgenv().ui
```

### Соглашения

- **`new_*`** — фабрики элементов, возвращают объект-элемент (`table`).
- **`set_*` / `get_*`** — универсальные методы для работы с элементами.
- **`is_*`** — булев шорткат.
- **Геттеры меню** (`menu_size`, `menu_position`, `mouse_position`) возвращают **по два числа** (кортеж), не `Vector2`.
- **`notify`** дублирует глобальный `notify(msg)` из README.

### Сигнатуры фабрик

| Функция | Сигнатура | Возвращает | Примечание |
|---|---|---|---|
| `new_button` | `(group, name)` | объект | callback через `set_callback` |
| `new_checkbox` | `(group, name)` | объект | default через `ui.set(el, true)` |
| `new_slider` | `(group, name, default, min, max)` | объект | **`min <= max` обязательно** |
| `new_textbox` | `(group, name[, default])` | объект | — |
| `new_combobox` | `(group, name, default, options)` | объект | `default` — строка из `options` |
| `new_multiselect` | `(group, name, default, options)` | объект | `default` — таблица выбранных |
| `new_listbox` | `(group, name, options)` | объект | — |
| `new_label` | `(group, text)` | объект | — |
| `new_hotkey` | `(group, name[, key])` | объект | — |
| `new_color_picker` | `(group, name[, color])` | объект | `color` — `{r,g,b,a}` |

### Сигнатуры методов

| Функция | Сигнатура | Возвращает | Примечание |
|---|---|---|---|
| `reference` | `(group, name)` | объект | ссылка на существующий элемент |
| `set` | `(element, value)` | — | установить значение |
| `get` | `(element)` | значение | получить значение |
| `set_callback` | `(element, fn)` | — | колбэк на изменение |
| `notify` | `(text[, type])` | `nil` | уведомление |
| `notification` | `(text[, type])` | `nil` | вероятно, алиас `notify` |
| `set_visible` | `(visible: boolean)` | — | показать/скрыть меню |
| `is_menu_open` | `()` | `boolean` | открыто ли меню |
| `menu_size` | `()` | `width, height` | два числа |
| `menu_position` | `()` | `x, y` | два числа |
| `mouse_position` | `()` | `x, y` | два числа |

## Полная таблица API

| Категория | Функция | Тип | Возвращает / Примечание |
|---|---|---|---|
| **Состояние меню** | `is_menu_open` | is_shortcut | `boolean` (рабочий, `false`) |
| | `set_visible` | setter | `(visible: boolean)` |
| | `menu_size` | getter | `width, height` (два числа) |
| | `menu_position` | getter | `x, y` (два числа) |
| | `mouse_position` | getter | `x, y` (два числа) |
| **Уведомления** | `notify` | function | `(text[, type])` → `nil` |
| | `notification` | function | `(text[, type])` → `nil`, вероятно алиас |
| **Фабрики элементов** | `new_button` | factory | `(group, name)` |
| | `new_checkbox` | factory | `(group, name)` |
| | `new_slider` | factory | `(group, name, default, min, max)` |
| | `new_textbox` | factory | `(group, name[, default])` |
| | `new_combobox` | factory | `(group, name, default, options)` |
| | `new_multiselect` | factory | `(group, name, default, options)` |
| | `new_listbox` | factory | `(group, name, options)` |
| | `new_label` | factory | `(group, text)` |
| | `new_hotkey` | factory | `(group, name[, key])` |
| | `new_color_picker` | factory | `(group, name[, color])` |
| **Работа с элементами** | `reference` | function | `(group, name)` → объект |
| | `set` | setter | `(element, value)` |
| | `get` | getter | `(element)` → значение |
| | `set_callback` | setter | `(element, fn)` |

## Подтверждённые рабочие функции

```
is_menu_open()    -> false (boolean)
menu_size()       -> 650, 480 (width, height)
menu_position()   -> 200, 200 (x, y)
mouse_position()  -> 517, 153 (x, y)
reference(group, name) -> объект (table)
notify(text)      -> nil (побочный эффект — уведомление)
```

## Известные ошибки и ограничения

- **`new_slider`**: `invalid argument #3 to 'clamp' (max must be greater than or equal to min)` — при `min > max`. Сигнатура: `(group, name, default, min, max)`, где `min <= max` обязательно.
- **`new_checkbox`**: `attempt to concatenate string with boolean` — не принимает `default` третьим аргументом. Значение задаётся через `ui.set(el, true)`.
- **`new_button`**: `attempt to index function with 'gsub'` — не принимает callback третьим аргументом. Callback — через `ui.set_callback(el, fn)`.
- **`new_combobox`**: `attempt to concatenate string with table` — третий аргумент должен быть строкой (`default`), а `options` — четвёртым.
- **`menu_size()` / `menu_position()` / `mouse_position()`** возвращают **кортеж из двух чисел**, а не `Vector2`. Обращение `size.X` вызовет `attempt to index number with 'X'`.
- **`debug.info` недоступен** — арность функций нельзя проверить рефлексией.
- **`getmetatable` / `rawget` недоступны**.
- Каждый скрипт — в отдельном окружении; переменные не сохраняются между запусками.
- `json` в глобалах пуст — используйте `HttpService:JSONEncode` / `:JSONDecode`.
- **`new_*` создают элементы в меню.** Удалить их через `ui` нельзя — только перезапуск игры/скрипта.

## Пример использования

```lua
local ui = getgenv().ui

-- Состояние меню
print("menu open:", ui.is_menu_open())      -- false
local w, h = ui.menu_size()                 -- 650, 480
local x, y = ui.menu_position()             -- 200, 200
local mx, my = ui.mouse_position()          -- 517, 153
print("size:", w, h)
print("pos:", x, y)
print("mouse:", mx, my)

-- Уведомление
ui.notify("Hello from script")

-- Фабрики элементов
local cb = ui.new_checkbox("My Script", "Enable feature")
local sl = ui.new_slider("My Script", "Value", 50, 0, 100)  -- min <= max обязательно
local bt = ui.new_button("My Script", "Click me")
local tb = ui.new_textbox("My Script", "Text", "hello")
local combo = ui.new_combobox("My Script", "Mode", "A", {"A", "B", "C"})
local ms = ui.new_multiselect("My Script", "Options", {"A"}, {"A", "B", "C"})
local lb = ui.new_listbox("My Script", "List", {"A", "B", "C"})
local lbl = ui.new_label("My Script", "Some text")
local hk = ui.new_hotkey("My Script", "Key", "F")
local cp = ui.new_color_picker("My Script", "Color", {1, 0, 0, 1})

-- Чекбокс: default через set (не через аргумент)
ui.set(cb, true)

-- Колбэк (не через аргумент new_button)
ui.set_callback(cb, function(value)
    print("checkbox:", value)
end)

ui.set_callback(bt, function()
    print("button clicked")
end)

-- Чтение/запись
local val = ui.get(sl)
ui.set(sl, 75)

-- Ссылка на существующий элемент
local ref = ui.reference("My Script", "Value")
print("reference:", ref)

-- Управление видимостью меню
ui.set_visible(true)
```

## Отличия `ui` от `rage` / `aa`

| Особенность | `rage` | `aa` | `ui` |
|---|---|---|---|
| Фабрики `new_*` | нет | нет | есть (10 типов элементов) |
| Универсальные `set`/`get` | нет | нет | есть |
| Уведомления | нет | нет | `notify` / `notification` |
| Управление меню | нет | нет | `is_menu_open`, `set_visible`, `menu_*`, `mouse_position` |
| Стабы | хитбоксы | yaw/pitch | не обнаружены (кроме непроверенных) |
| `*_deg`-геттеры | нет | есть | нет |

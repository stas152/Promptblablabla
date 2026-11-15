# Penablox Recode — UI Module (verified)

> Сигнатуры подтверждены эмпирически. Отклонения от старой документации помечены ⚠️.

---

## Геттеры — возвращают **кортеж чисел**, не `Vector2`

```lua
ui.menu_size()        --> width, height    -- числа
ui.menu_position()    --> x, y             -- числа
ui.mouse_position()   --> x, y             -- числа
ui.is_menu_open()     --> boolean
```

⚠️ `ui.menu_size().X` **упадёт**. Только `local w, h = ui.menu_size()`.

---

## Создание элементов

```lua
ui.new_button(group, [name])                            --> element
ui.new_checkbox(group, [name])                          --> element
ui.new_label(group, [text])                             --> element
ui.new_textbox(group, [name], [default])                --> element
ui.new_combobox(group, [name], [default], [options])    --> element
ui.new_multiselect(group, [name], [default], [options]) --> element
ui.new_listbox(group, [name], [options])                --> element
ui.new_hotkey(group, [name], [key])                     --> element
ui.new_color_picker(group, [name], [color])             --> element
ui.new_slider(group, [name], [default], [min], [max])   --> element
```

**Слайдер:**
- `default` по умолчанию `0`, `min=0`, `max=100`.
- `default` автоматически клампится в `[min, max]`.
- ⚠️ `min > max` → **Lua-ошибка** (`clamp: max must be >= min`). Единственный конструктор, который реально бросает.

**Все `new_*`:** после `group` любые аргументы опциональны. Отсутствие не вызывает ошибку — подставляются дефолты/`nil`.

---

## Управление элементами

```lua
ui.reference(group, name)      --> element
ui.set(element, value)         --> nil
ui.get(element)                --> value | nil
ui.set_callback(element, fn)   --> nil
```

⚠️ **`reference` не проверяет существование** — возвращает таблицу-элемент даже для несуществующих `group`/`name`. Такой объект «мёртвый»: `set` уходит в никуда, `get` возвращает `nil`.

⚠️ **`set_callback` не принимает callback как аргумент `new_*`** — только отдельным вызовом после создания элемента.

---

## Уведомления и меню

```lua
ui.notify(text, [type])        --> nil
ui.notification(text, [type])  --> nil    -- отдельная функция, не алиас
ui.set_visible(visible)        --> nil
```

---

## Критично: silent-fail

Движок **не бросает ошибок** на неверные аргументы — молча возвращает `nil`:

```lua
ui.get(nil)             --> nil, без ошибки
ui.set(nil, true)       --> nil, без ошибки
ui.set({}, 1)           --> nil, без ошибки
ui.set_callback(nil,f)  --> nil, без ошибки
ui.new_button()         --> element (создан с пустой group!)
```

**Следствия:**
- `pcall` бесполезен для валидации (кроме `new_slider`).
- Опечатка в `group`/`name` → тихий баг без warning'а.
- Проверяй `element ~= nil` и `ui.get(el) ~= nil` вручную.

---

## Быстрые правила

| ❌ Неправильно | ✅ Правильно |
|---|---|
| `ui.menu_size().X` | `local w = ui.menu_size()` |
| `ui.new_button(g, "b", callback)` | `ui.set_callback(ui.new_button(g,"b"), callback)` |
| `ui.new_checkbox(g, "c", true)` | `ui.set(ui.new_checkbox(g,"c"), true)` |
| `ui.new_slider(g,"s",0,100,10)` | `ui.new_slider(g,"s",0,10,100)` |
| `if not pcall(ui.set, el, v) then` | `if el ~= nil then ui.set(el, v) end` |

---

## Хелпер-обёртка

```lua
local function set_if(el, v)
    if el == nil then return end
    ui.set(el, v)
end

local function ref_if(g, n)
    local el = ui.reference(g, n)
    if el == nil or ui.get(el) == nil then return nil end
    return el
end
```

---

## Пример

```lua
local group = "MyScript"

-- Слайдер с явным диапазоном
local speed = ui.new_slider(group, "Speed", 50, 1, 200)

ui.set_callback(speed, function(v)
    print("speed =", v)  -- v клампится в [1, 200]
end)

-- Чекбокс + колбэк
local cb = ui.new_checkbox(group, "Enabled")
ui.set(cb, true)
ui.set_callback(cb, function(on)
    ui.notify("Feature: " .. tostring(on))
end)

-- Кнопка
ui.set_callback(ui.new_button(group, "Reset"), function()
    ui.set(speed, 50)
end)
```

---

**Версия:** проверено на Penablox Recode, 2026-11. Все сигнатуры — по факту исполнения.

# Plant with Coding — Game Documentation

**Platform:** Roblox | **Status:** Beta | **Genre:** Simulation / Programming

## 1. Overview

Plant with Coding is a Roblox simulation game where players manage a farm through code rather than manual labor. Players program a high-tech drone to automate planting, watering, and harvesting crops. The game is inspired by titles such as *Grow a Garden* and *The Farmer Was Replaced*.

## 2. Core Gameplay

Players control a drone using **LAU**, a custom scripting language heavily inspired by Lua. The drone performs all agricultural tasks automatically once programmed. The primary loop involves purchasing seeds, planting them via drone commands, waiting for growth, and selling harvested produce for profit.

## 3. Programming Language: LAU

LAU is the in-game scripting language. Key characteristics:

| Feature | Description |
|---------|-------------|
| **Syntax** | Lua-like, but uses `varol` instead of `local` for variable declaration |
| **Block Termination** | Uses `then` / `end` (not curly braces) |
| **Reserved Words** | `varol`, `func`, `if`, `else`, `elseif`, `then`, `end`, `for`, `while`, `do`, `break`, `continue`, `return`, `req`, `true`, `false`, `null`, `inpairs`, `and`, `or`, `not` |
| **Character Restrictions** | Non-ASCII characters (including Cyrillic and emojis) are prohibited and will cause errors |
| **Permission Limitations** | The `task` module is restricted for regular players |

## 4. Drone API Reference

The `drone` object exposes the following methods:

### 4.1 Movement & Animation
- `drone.move(Enum.Direction)` — Moves the drone one unit in the specified direction (North, South, East, West)
- `drone.doFlip()` — Performs a backflip animation

### 4.2 Position & Coordinates
- `drone.getPosition()` — Returns both X and Z coordinates simultaneously (supports multiple assignment)
- `drone.getPositionX()` / `drone.getPositionZ()` — Returns the respective coordinate individually

### 4.3 Farming Operations
- `drone.plant(Enum.Seed)` — Plants the specified seed type into the soil beneath the drone
- `drone.useItem(Enum.Gear)` — Applies the specified gear (e.g., Watering_Can, Fertilizer, Sprinkler) to the current tile
- `drone.crop()` — Harvests root crops (wheat, potato, etc.) by cutting from the root
- `drone.harvest()` — Collects fruits from fruit-bearing trees

### 4.4 Smart Control Sensors
- `drone.canCrop()` — Returns true/false whether the crop beneath is ready for harvesting
- `drone.canHarvest()` — Checks if fruit is ready to be collected

### 4.5 Plant & Fruit Data Sensors
- `drone.getPlant()` — Returns an object containing plant data (Percent, Weight, HasFruit)
- `drone.getPlantPercent()` / `drone.getPlantWeight()` — Returns individual numeric values
- `drone.getPlantHasFruit()` — Returns boolean indicating if the plant produces fruit

### 4.6 Drone Status
- `drone.status()` — Returns `Enum.DroneStatus.Busy` or `Enum.DroneStatus.Sleep`

## 5. Enum Reference

| Enum | Values | Usage |
|------|--------|-------|
| `Enum.Direction` | North, South, East, West | Movement direction |
| `Enum.Seed` | Dynamic (e.g., Wheat, Potato, Carrot, Apple) | Seed type for planting |
| `Enum.Gear` | Dynamic (e.g., Watering_Can, Fertilizer, Sprinkler) | Gear/tool type for `useItem` |
| `Enum.DroneStatus` | Busy, Sleep | Drone operational state |

**Note:** Seed and Gear names are **dynamic** — they depend on the player's current inventory and game state. Use the exact name shown in the respective menu.

## 6. Pragmas (Compiler Directives)

Pragmas are special comments placed at the **top of the script** (within the first 10 lines) that modify engine behavior:

| Pragma | Effect |
|--------|--------|
| `--!ndrone` | Enables **Asynchronous mode**. Commands execute instantly without waiting for animations. **Risk:** Overlapping commands are ignored if the drone is busy. Must manually check `drone.status()` before issuing new commands. |
| `--!stats` | Generates a performance report after script execution, including total time, statement count, and hottest lines. |
| `--!native` | Runs script at maximum engine speed. **Restricted to developers/test servers only.** |
| `--!debug` | Opens a live debug window. **Restricted to approved testers only.** |

**Default behavior (no pragma):** Synchronous mode. Each command blocks until the drone finishes the previous action. No manual wait functions are required.

## 7. Recommended Farming Logic

The optimal decision sequence for each tile, as documented in-game:

1. **Check the tile** using plant-checking functions.
2. **Harvest first:** If `drone.canHarvest()` returns true, call `drone.harvest()`.
3. **Crop second:** Else if `drone.canCrop()` returns true, call `drone.crop()`.
4. **Plant if empty:** If neither harvest nor crop is possible, call `drone.plant(seedType)`.

## 8. Known Limitations

- `task.wait()` is not accessible to regular players (Missing Permission error)
- `local` keyword is not supported; use `varol` instead
- Non-English characters in code will produce syntax errors
- Seed and Gear Enum values must match the exact in-game inventory names; passing strings instead of Enums causes type errors
- The game is in Beta; APIs and behavior may change

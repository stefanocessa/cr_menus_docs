# CR Menus — v2.0.0

### Advanced Spell Menu, Ability System & Role Management for FiveM

---

## Table of Contents

1. [Description](#description)
2. [Features](#features)
3. [Dependencies](#dependencies)
4. [Installation](#installation)
5. [Configuration](#configuration)
6. [Commands](#commands)
7. [Exports (Client)](#exports-client)
8. [Exports (Shared — Spells API)](#exports-shared--spells-api)
9. [Usage Examples](#usage-examples)
10. [FAQ](#faq)

---

## Description

**CR Menus** is a complete FiveM resource for managing custom ability menus (spells/skills), assigning roles and grades to players, and handling cosmetic features such as emissive eyes, colored smoke, custom peds, and cloaks. It includes a persistent drag-and-drop **Spellbar** and **Hotbar**, a modern NUI interface, and a MySQL backend for persistent data storage.

The system is designed for **fantasy/roleplay** servers with classes, races, or factions (vampires, werewolves, guardians, etc.), but is completely generic and adaptable to any scenario.

> ⚠️ The resource name **must be exactly** `cr_menus`. An automatic check runs on startup — if the name is different, the server will shut down after 10 seconds.

> ℹ️ **Framework-agnostic:** CR Menus works with **ESX**, **QBCore**, or any custom framework. All framework interactions are abstracted through **`cr_lib`** — the free CR library available on Tebex. Configure it once for your framework and CR Menus will work out of the box.

---

## Features

| Feature                  | Description                                                                                                        |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| **Ability Menu (NUI)**   | Graphical panel (default F3) showing all the player's spells/abilities                                             |
| **Spellbar**             | Quick-access bar with up to 12 slots, customizable via drag & drop from the UI                                     |
| **Hotbar**               | Rotating bar (12 slots, groups of 4) for fast in-game spell access via keybinds                                    |
| **Menu + Grade System**  | Create menus (e.g. "Vampire") with multiple grades (e.g. "Fledgling", "Elder"), each with its own spells and stats |
| **Custom Spells**        | Define spells in Lua with cooldown, icon, description, `onUse` logic, and conditional lock                         |
| **Staff Assignment**     | Moderators can assign menus, grades, eyes, smoke, peds, and cloaks to any online player                            |
| **Player List**          | Staff panel to search, view, edit, and revoke any player's menu                                                    |
| **Transformation State** | Transformation state managed via StateBag with a customizable callback                                             |
| **Sensory System**       | Proximity radar with dynamic blips, configurable per grade                                                         |
| **Persistent KVP**       | Spellbar and hotbar are saved locally on the client between sessions                                               |
| **MySQL Database**       | All player data and created menus are stored in MySQL via oxmysql                                                  |
| **In-Game Menu Creator** | Moderators can create, edit, and delete menus and grades directly in-game                                          |

---

## Dependencies

Install and start **all** of the following resources before `cr_menus`:

| Resource                                             | Required | Notes                                                                                                                                                                    |
| ---------------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`oxmysql`](https://github.com/overextended/oxmysql) | ✅ Yes   | Async MySQL driver                                                                                                                                                       |
| [`ox_lib`](https://github.com/overextended/ox_lib)   | ✅ Yes   | UI library, callbacks, animations                                                                                                                                        |
| `cr_lib`                                             | ✅ Yes   | Free CR abstraction library — **download it for free on Tebex**. Supports ESX, QBCore, and custom frameworks. Configure it once and all CR resources work automatically. |

> **Database:** A MySQL/MariaDB database must be active and configured in `oxmysql`. Tables are created automatically on first resource start.

> **Framework:** ESX, QBCore, and custom frameworks are all supported via `cr_lib`. No direct dependency on any specific framework.

---

## Installation

### 1. Place the files

Extract and place the folder at:

```
resources/[scripts]/[cr]/cr_menus/
```

The folder name **must** be exactly `cr_menus`.

### 2. Add to server.cfg

```cfg
ensure cr_lib
ensure cr_menus
```

Make sure `cr_lib` is started **before** `cr_menus`.

### 3. Database configuration

Open `sv_config.lua` and verify the table names:

```lua
Sv_Config.DatabaseName            = "cr_menus"        -- player assignment table
Sv_Config.CreatedMenuDatabaseName = "cr_createdmenu"  -- created menus table
```

Names can be changed if needed. Tables are created automatically.

Also verify the users table name (used to resolve player identifiers):

```lua
Sv_Config.usersTable = "users"  -- change if your framework uses a different table name
```

### 4. Start the server

On the first start, `cr_menus` will automatically create the required MySQL tables. Check the server console for any database connection errors.

---

## Configuration

Everything is configurable in `config.lua`. Below are the main sections.

### Load Event

```lua
Config.LoadEvent = 'esx:playerLoaded'
```

Change this to match your framework's player-loaded event (e.g. `QBCore:Client:OnPlayerLoaded`).

### UI

```lua
Config.UI = {
    SpellsBarSize    = 12,         -- number of spellbar slots
    HotbarTotalSize  = 12,         -- total hotbar slots
    HotbarGroupSize  = 4,          -- slots per hotbar group (navigable with keybinds)
    DefaultBaseColor = "#9E0E16",  -- default menu base color (hex)
}
```

### Keybinds

Keybinds are registered automatically by `ox_lib`. Players can remap them in GTA V Settings > Key Bindings.

| Action                         | Default        |
| ------------------------------ | -------------- |
| Open spell list                | — (remappable) |
| Toggle spellbar                | —              |
| Use selected spell             | —              |
| Move selection left/right      | —              |
| Rotate hotbar forward/backward | —              |
| Clear KVP                      | —              |

### Defining a Spell

Add your spells to `Config.SpellsList` in `config.lua`:

```lua
Config.SpellsList = {
    ['spell_id'] = {
        name         = "Display Name",
        description  = "Spell description shown in the UI.",
        cooldown     = 10,              -- cooldown in seconds
        icon         = "https://...",   -- icon URL (png/webp)
        class        = "Optional Class",
        hideOnLocked = false,           -- hide from UI when locked

        -- Lock condition: return true to lock the spell
        lockSpell = function()
            return false
        end,

        -- Executed when the player uses the spell
        onUse = function()
            -- your spell logic here
            return true  -- return true to trigger the cooldown
        end,
    },
}
```

### Custom Icons per Menu

```lua
Config.MenuIcons = {
    menu_name = {
        ['spell_id'] = "https://example.com/icon.png",
    },
}
```

### Skins / Peds

```lua
Config.Skin = {
    base_male = { model = "mp_m_freemode_01", description = "Base male skin" },
    zombie    = { model = "u_m_y_zombie_01",  description = "Zombie" },
}

Config.Ped = {
    my_ped = { model = "model_name", description = "Description" },
}
```

### Cloaks

```lua
Config.Cloacks = {
    ['cloak_name'] = {
        white = {
            { componente = 11, drawable = 0, texture = 0 },
            -- add more components as needed
        },
    }
}
```

### Smoke Colors

The system includes 20+ predefined colors in `Config.Smoke`. Add more using the same `{ label, r, g, b, a }` pattern.

### Marker Color

```lua
Config.MarkersColor = { r = 158, g = 14, b = 22, a = 255 }
```

### Transformation

```lua
Config.TransformationIdentifier = "isTransformed"

Config.onTransformationChange = function(menuName, gradeName, menuData, gradeData, isTransformed)
    -- called every time a player enters or exits transformation
end
```

### Death Check

```lua
Config.CheckDeath = function(cID)
    -- customize how to check if the player is dead
    local sID = GetPlayerServerId(cID or PlayerId())
    return GetStateBagValue('player:' .. sID, 'isDead')
end
```

---

## Commands

All commands are configurable in the `Config.Commands` block in `config.lua`.

| Command         | Permission   | Description                                                          |
| --------------- | ------------ | -------------------------------------------------------------------- |
| `/menu`         | `user`       | Opens the player's spell panel (default key F3)                      |
| `/gestionemenu` | `moderatore` | Opens the in-game menu creator (create/edit/delete menus and grades) |
| `/playersmenu`  | `moderatore` | Opens the player list with their assigned menus                      |
| `/setmenu`      | `moderatore` | Opens the menu/grade assignment panel                                |
| `/seteye`       | `moderatore` | Assigns an emissive eye color to a player                            |
| `/seteyenumber` | `moderatore` | Sets the eye overlay number                                          |
| `/setpedmenu`   | `moderatore` | Assigns a custom ped to a player                                     |
| `/setsmoke`     | `moderatore` | Assigns a smoke color to a player                                    |
| `/setmanto`     | `moderatore` | Assigns a cloak to a player                                          |
| `/clearmenukvp` | `user`       | Clears the player's saved spellbar and hotbar data                   |

---

## Exports (Client)

These exports can be called from any other resource on the client side.

```lua
exports['cr_menus']:ExportName(params)
```

### Spells & Cooldowns

| Export              | Parameters                                 | Returns   | Description                                  |
| ------------------- | ------------------------------------------ | --------- | -------------------------------------------- |
| `GetActiveSpells`   | —                                          | `table`   | Returns the player's currently active spells |
| `IsSpellActive`     | `spellId: string`                          | `boolean` | Checks if a spell is active                  |
| `SetActiveSpell`    | `spellId: string`                          | —         | Marks a spell as active                      |
| `RemoveActiveSpell` | `spellId: string`                          | —         | Removes a spell from active state            |
| `IsOnCooldown`      | `spellId: string`                          | `boolean` | Checks if a spell is on cooldown             |
| `StartCooldown`     | `spellId: string, durationSeconds: number` | —         | Manually starts a cooldown                   |
| `RefreshSpells`     | —                                          | —         | Forces a recalculation of locked spells      |

### Spell Data

| Export               | Parameters        | Returns      | Description                                           |
| -------------------- | ----------------- | ------------ | ----------------------------------------------------- |
| `GetSpellData`       | `spellId: string` | `table\|nil` | Returns a spell's definition from `Config.SpellsList` |
| `GetSpellIcon`       | `spellId: string` | `string`     | Returns the icon URL of a spell                       |
| `GetAvailableSkills` | —                 | `table`      | Returns all available skills for the current player   |

### Spellbar / Hotbar

| Export              | Parameters    | Returns | Description                                |
| ------------------- | ------------- | ------- | ------------------------------------------ |
| `GetSpellbar`       | —             | `table` | Reads the saved spellbar (KVP)             |
| `SetSpellbar`       | `data: table` | —       | Saves the spellbar (KVP)                   |
| `GetHotbar`         | —             | `table` | Reads the saved hotbar (KVP)               |
| `SetHotbar`         | `data: table` | —       | Saves the hotbar (KVP)                     |
| `ClearAllSavedData` | —             | —       | Clears both spellbar and hotbar saved data |

### Menus

| Export                | Parameters            | Returns      | Description                                   |
| --------------------- | --------------------- | ------------ | --------------------------------------------- |
| `IsMenuReady`         | —                     | `boolean`    | Checks if the menu system is initialized      |
| `GetAllMenus`         | —                     | `table`      | Returns all created menus (server callback)   |
| `GetMenu`             | `menuName: string`    | `table\|nil` | Returns a single menu's data                  |
| `RebuildMenu`         | `menuName, gradeName` | `table`      | Recomputes and returns menu data              |
| `GetComputedMenuData` | `menuName, gradeName` | `table`      | Reads computed data from cache                |
| `RefreshMenus`        | —                     | `table`      | Refreshes the local menu cache and returns it |

### Menu CRUD

| Export                 | Parameters                              | Returns           | Description                         |
| ---------------------- | --------------------------------------- | ----------------- | ----------------------------------- |
| `CreateMenu`           | `menuLabel: string, baseColor: string`  | `boolean, string` | Creates a new menu                  |
| `DeleteMenu`           | `menuName: string`                      | `boolean`         | Deletes a menu                      |
| `UpdateMenuColors`     | `menuName, baseColor, fallbackIcon`     | `boolean`         | Updates menu color and default icon |
| `CreateGrade`          | `menuName, gradeLabel, spells, sensory` | `boolean, string` | Creates a grade inside a menu       |
| `UpdateGrade`          | `menuName, gradeName, newData`          | `boolean`         | Updates a grade's data              |
| `DeleteGrade`          | `menuName, gradeName`                   | `boolean`         | Deletes a grade                     |
| `AddSpellToGrade`      | `menuName, gradeName, spellName`        | `boolean`         | Adds a spell to a grade             |
| `RemoveSpellFromGrade` | `menuName, gradeName, spellName`        | `boolean`         | Removes a spell from a grade        |
| `GenerateMenuId`       | `label: string`                         | `string`          | Generates a clean ID from a label   |
| `GenerateGradeId`      | `label: string`                         | `string`          | Generates a clean ID from a label   |

### Staff Panel Openers

| Export                    | Description                            |
| ------------------------- | -------------------------------------- |
| `OpenAssignMainMenu`      | Opens the menu/grade assignment panel  |
| `OpenAssignEyeMenu`       | Opens the eye assignment panel         |
| `OpenAssignEyeNumberMenu` | Opens the eye number assignment panel  |
| `OpenAssignSmokeMenu`     | Opens the smoke color assignment panel |
| `OpenAssignPedMenu`       | Opens the ped assignment panel         |
| `OpenAssignMantoMenu`     | Opens the cloak assignment panel       |
| `OpenMenuCreator`         | Opens the menu creator                 |
| `OpenPlayersList`         | Opens the player list                  |

### Player Assignment Management

| Export                     | Parameters                      | Returns      | Description                                |
| -------------------------- | ------------------------------- | ------------ | ------------------------------------------ |
| `GetAllPlayerAssignments`  | —                               | `table`      | Returns all players with an assigned menu  |
| `GetPlayerAssignment`      | `identifier: string`            | `table\|nil` | Returns a specific player's menu data      |
| `UpdatePlayerAssignment`   | `identifier, newMenu, newGrade` | `boolean`    | Updates a player's menu and grade          |
| `RevokePlayerMenu`         | `identifier: string`            | `boolean`    | Revokes a player's menu                    |
| `ShowPlayerDetails`        | `identifier: string`            | `boolean`    | Opens the UI with a player's details       |
| `RefreshPlayerAssignments` | —                               | `table`      | Refreshes and returns the assignment cache |

### Utility

| Export      | Parameters    | Returns  | Description                   |
| ----------- | ------------- | -------- | ----------------------------- |
| `Translate` | `key: string` | `string` | Translates a localization key |

---

## Exports (Shared — Spells API)

The `Spells` library is available client-side from any resource:

```lua
local Spells = exports['cr_menus']:Spells()
```

### Ped Utilities

| Method                                              | Parameters        | Description                             |
| --------------------------------------------------- | ----------------- | --------------------------------------- |
| `Spells.setPedModel(model)`                         | `string\|number`  | Changes the player ped model            |
| `Spells.setPedHealth(amount, fill)`                 | `number, boolean` | Sets max HP (`fill=true` fills to max)  |
| `Spells.setPedArmor(amount)`                        | `number`          | Sets max and current armor              |
| `Spells.setComponents(list)`                        | `table`           | Applies component variations to the ped |
| `Spells.taskAnimation(dict, anim, flags, duration)` | —                 | Plays an animation on the ped           |

### Combat & Physics

| Method                                                      | Parameters | Description                                    |
| ----------------------------------------------------------- | ---------- | ---------------------------------------------- |
| `Spells.performPhysics(entity, dir, power)`                 | —          | Applies a directional force to an entity       |
| `Spells.spawnProiettile(info, tCoords, entityhit, cCoords)` | `table`    | Spawns a projectile with particles and physics |
| `Spells.toggleRun(bool)`                                    | `boolean`  | Toggles run speed multiplier (1.35x)           |
| `Spells.toggleJump(bool)`                                   | `boolean`  | Toggles super jump                             |
| `Spells.toggleStrenght(bool)`                               | `boolean`  | Toggles unarmed damage modifier                |
| `Spells.sendThunder()`                                      | —          | Triggers thunder FX around the player          |

### Targeting / Aiming

| Method                                          | Returns                                    | Description                          |
| ----------------------------------------------- | ------------------------------------------ | ------------------------------------ |
| `Spells.performAoeDrawMarker(toggle, canScale)` | `coords, scale, entity`                    | AoE marker with scroll-wheel scaling |
| `Spells.performTargetDrawMarker(toggle)`        | `entity, coords`                           | Entity targeting with a visual line  |
| `Spells.targetMarker(toggle, range, maxTime)`   | `entity\|'cancelled'\|'outofTime', coords` | Target marker with optional timeout  |
| `Spells.rayCastGamePlayCamera(distance)`        | `hit, coords, entity`                      | Raycast from the gameplay camera     |

### Proximity / Radar

| Method                                                                                | Parameters        | Returns            | Description                     |
| ------------------------------------------------------------------------------------- | ----------------- | ------------------ | ------------------------------- |
| `Spells.getClosestPed(coords, radius)`                                                | `vector3, number` | `ped, distance`    | Closest ped within radius       |
| `Spells.getClosestPlayer(radius, coords)`                                             | —                 | `player, distance` | Closest player within radius    |
| `Spells.getPlayersAround(radius, coords)`                                             | —                 | `table`            | List of players within radius   |
| `Spells.getPlayersInRange(range, coords)`                                             | —                 | `table`            | List of player IDs within range |
| `Spells.closestPeds(playerPed, radius)`                                               | —                 | `table`            | All peds within radius          |
| `Spells.getEquidistantCircleCoords(count, radius, zOffset, startAngle, snapToGround)` | —                 | `table[vector3]`   | Equidistant coords on a circle  |

### Player State

| Method                                       | Returns   | Description                                   |
| -------------------------------------------- | --------- | --------------------------------------------- |
| `Spells.getPlayerDeadStatus(cID)`            | `boolean` | Checks if a player is dead (StateBag)         |
| `Spells.getPlayerIsTransformed(cID)`         | `boolean` | Checks transformation state (StateBag)        |
| `Spells.performPlayerTransformation(toggle)` | —         | Sets transformation state via server callback |

### Sensory

| Method                                                        | Parameters | Description                                                           |
| ------------------------------------------------------------- | ---------- | --------------------------------------------------------------------- |
| `Spells.sensory(range)`                                       | `number`   | Loop emitting `cr_spells:sensoriale` every second with nearby players |
| `Spells.createSensoriale(coords, color, sprite, scale, name)` | —          | Creates a temporary blip (1.2s) on an entity                          |

### Math / Directions

| Method                                      | Description                                       |
| ------------------------------------------- | ------------------------------------------------- |
| `Spells.rotationToDirection(rotation)`      | Converts a rotation to a direction vector (table) |
| `Spells.getDirectionFromRotation(rotation)` | Converts a rotation to a `vector3`                |

---

## Usage Examples

### Spell that fires a projectile

```lua
Config.SpellsList['fireball'] = {
    name         = "Fireball",
    description  = "Launch a blazing sphere toward your target.",
    cooldown     = 8,
    icon         = "https://mycdn.com/fireball.png",
    class        = "Offensive",
    hideOnLocked = false,
    lockSpell    = function() return false end,

    onUse = function()
        local Spells = exports['cr_menus']:Spells()
        local entity, coords = Spells.targetMarker(true, 80.0, 10)
        if entity == 'cancelled' or entity == 'outofTime' then return false end

        Spells.spawnProiettile({
            model = "prop_golf_ball",
            speed = 80.0,
            dict  = "des_gas_station",
            name  = "ent_ray_paleto_gas_debris_trails",
            scale = 3.5,
            r = 1.0, g = 0.3, b = 0.0, a = 1.0
        }, coords, entity, nil)

        return true  -- triggers cooldown
    end,
}
```

### Read the current player's menu from another resource

```lua
-- Client-side
RegisterNetEvent('myresource:checkMenu', function()
    if exports['cr_menus']:IsMenuReady() then
        local pMenu  = LocalPlayer.state.menu   -- StateBag set by cr_menus
        local pGrade = LocalPlayer.state.grade
        print(("Menu: %s | Grade: %s"):format(pMenu, pGrade))
    end
end)
```

### Check and manually start a cooldown

```lua
local spellId = 'teleport'

if exports['cr_menus']:IsOnCooldown(spellId) then
    return  -- spell is on cooldown, do nothing
end

-- execute your spell logic...
exports['cr_menus']:StartCooldown(spellId, 15)  -- 15 second cooldown
```

### Programmatically assign a menu to a player

```lua
-- Open the staff UI:
exports['cr_menus']:OpenAssignMainMenu()

-- Or do it programmatically (client-side, requires staff permission):
local ok = exports['cr_menus']:UpdatePlayerAssignment("license:abc123", "vampire", "elder")
if ok then print("Menu updated!") end
```

### Create a menu from code

```lua
-- Create the menu
local ok, menuId = exports['cr_menus']:CreateMenu("Vampire", "#8B0000")
-- menuId will be "vampire" (auto-generated from the label)
if not ok then return end

-- Create a grade inside it
local gradeOk, gradeId = exports['cr_menus']:CreateGrade(menuId, "Fledgling", {
    "fireball", "teleport"
})

-- Update the menu color and fallback icon
exports['cr_menus']:UpdateMenuColors(menuId, "#6B0000", "https://mycdn.com/vampire.png")
```

---

## FAQ

**Q: What frameworks are supported?**
A: CR Menus is framework-agnostic. All framework interactions are handled by **`cr_lib`**, a free library available on Tebex. It supports **ESX**, **QBCore**, and custom frameworks. Configure `cr_lib` once and every CR resource will work without any further changes.

**Q: Are the database tables created automatically?**
A: Yes. On resource start, if the tables do not exist they are created automatically via `oxmysql`.

**Q: Can I rename the resource?**
A: No. The code in `server/menu-events.lua` checks that the resource name is exactly `cr_menus`. Renaming it will cause automatic server shutdown after 10 seconds.

**Q: How do I add translations for another language?**
A: In `config.lua`, inside `Config.Intl.translations`, add an `en = { ... }` block (or any other locale key) with the same keys as the existing `it` block, then set `Config.Intl.locale = 'en'`.

**Q: Can a player have multiple menus assigned at once?**
A: No. Each player has exactly one menu and one grade assigned at a time.

**Q: Are spells executed client-side or server-side?**
A: The `onUse` function runs client-side. For server-side actions (e.g. dealing damage to other players, database writes) use `TriggerServerEvent` inside `onUse`.

**Q: How do StateBags work with this resource?**
A: When a player is assigned a menu, the server automatically writes the values to `Player(source).state` (replicated to all clients). You can read any player's menu from any resource using `GetStateBagValue('player:' .. serverId, 'menu')`.

**Q: Can I trigger staff panels from my own resource?**
A: Yes. All staff panels are exposed as exports (e.g. `OpenAssignMainMenu`, `OpenMenuCreator`, `OpenPlayersList`) so you can open them from any client script.

---

_Documentation for CR Menus v2.0.0 — All rights reserved._

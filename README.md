# CR Menus — v2.0.0

### Advanced Skills, Menu & Role Management System for FiveM (ESX / QBCore)

---

## Index

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

**CR Menus** is a complete system for FiveM servers that allows you to manage custom skill/spell menus, assign roles with ranks to players, and handle cosmetic elements (emissive eyes, colored smoke, custom peds, cloaks).

It includes:

- A persistent draggable **Spellbar**
- A rotating **Hotbar**
- A modern **NUI interface**
- A MySQL backend for saving data

The system is designed primarily for **fantasy/roleplay** servers (classes, races, factions like vampires, werewolves, guardians, etc.), but it is fully generic and adaptable to any scenario.

It supports both **ESX and QBCore**, thanks to the internal framework abstraction provided by **`cr_lib`**, available for free on Tebex.  
`cr_lib` automatically bridges permissions, player data, and framework events.

> ⚠️ The resource name **must be exactly** `cr_menus`.  
If renamed, the server will automatically shut down after 10 seconds.

---

## Features

| Feature                     | Description                                                                 |
|------------------------------|----------------------------------------------------------------------------|
| Skill Menu (NUI)            | Graphical panel (default F3) showing all player spells/skills             |
| Spellbar                    | Quick-access bar with up to 12 slots, customizable via drag & drop        |
| Hotbar                      | Rotating bar (12 slots, groups of 4) for quick in-game access             |
| Menu + Rank System          | Create menus with multiple ranks, each with its own spells and stats      |
| Custom Spells               | Define spells in Lua with cooldown, icon, description and `onUse` logic   |
| Staff Assignment Panel      | Assign menus, ranks, eyes, smoke, peds, cloaks to players                 |
| Players List Panel          | Search, edit, and revoke player menus                                     |
| Transformation System       | Managed via StateBag with customizable callback                           |
| Sensory System              | Proximity radar with dynamic blips                                         |
| Persistent KVP              | Spellbar and hotbar saved locally between sessions                        |
| MySQL Database              | Player and menu data stored via `oxmysql`                                 |
| In-Game Menu Creator        | Create, edit and delete menus and ranks in-game                           |

---

## Dependencies

Install and start **all** of the following resources before `cr_menus`:

| Resource                                              | Required | Notes                                      |
|------------------------------------------------------|----------|--------------------------------------------|
| oxmysql                                              | ✅ Yes   | Async MySQL driver                         |
| ox_lib                                               | ✅ Yes   | UI, callbacks, animations                  |
| cr_lib                                               | ✅ Yes   | ESX/QBCore bridge & permission abstraction |

> A MySQL/MariaDB database must be configured in `oxmysql`.  
Tables are automatically created on resource startup.

---

## Installation

### 1. Place the resource

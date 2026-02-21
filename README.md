# CR Menus --- v2.0.0

### Advanced Skills, Menu & Role Management System for FiveM (ESX / QBCore)

------------------------------------------------------------------------

## Description

CR Menus is a complete system for FiveM servers that allows you to
manage custom skill/spell menus, assign roles with ranks to players, and
handle cosmetic elements such as emissive eyes, colored smoke, custom
peds, and cloaks.

It includes: - Persistent draggable Spellbar - Rotating Hotbar - Modern
NUI interface - MySQL backend (oxmysql)

Compatible with ESX and QBCore thanks to cr_lib (framework bridge
available on Tebex).

⚠️ The resource name must be exactly: cr_menus

------------------------------------------------------------------------

## Features

-   Skill Menu (NUI)
-   Spellbar (up to 12 slots)
-   Rotating Hotbar (12 total slots, groups of 4)
-   Menu + Rank System
-   Custom Lua Spells with cooldowns
-   Staff Assignment Panels
-   Player Menu Management
-   Transformation System (StateBag based)
-   Sensory Radar System
-   Persistent KVP storage
-   MySQL automatic table creation
-   In-game Menu Creator

------------------------------------------------------------------------

## Dependencies

-   oxmysql (required)
-   ox_lib (required)
-   cr_lib (required --- ESX/QBCore bridge)

Database tables are created automatically on first startup.

------------------------------------------------------------------------

## Installation

1.  Place the folder inside: resources/\[scripts\]/\[cr\]/cr_menus/

2.  Add to server.cfg: ensure cr_lib ensure cr_menus

Make sure cr_lib starts before cr_menus.

------------------------------------------------------------------------

## Framework Load Event

ESX: Config.LoadEvent = 'esx:playerLoaded'

QBCore: Config.LoadEvent = 'QBCore:Client:OnPlayerLoaded'

------------------------------------------------------------------------

## Basic Spell Example

Config.SpellsList = { \['spell_id'\] = { name = "Visible Name",
description = "Spell description", cooldown = 10, icon = "https://...",
class = "Optional Class", hideOnLocked = false, lockSpell = function()
return false end, onUse = function() return true end, }, }

------------------------------------------------------------------------

## FAQ

Q: Does it work only with ESX? A: No. It works with both ESX and QBCore
thanks to cr_lib.

Q: Are database tables created automatically? A: Yes, via oxmysql.

Q: Can I rename the resource? A: No. It must remain cr_menus.

------------------------------------------------------------------------

CR Menus v2.0.0 All rights reserved.

# CR Menus — v2.0.0

### Sistema avanzato di Abilità, Menù e Gestione Ruoli per FiveM (ESX)

---

## Indice

1. [Descrizione](#descrizione)
2. [Funzionalità](#funzionalità)
3. [Dipendenze](#dipendenze)
4. [Installazione](#installazione)
5. [Configurazione](#configurazione)
6. [Comandi](#comandi)
7. [Exports (Client)](#exports-client)
8. [Exports (Shared — Spells API)](#exports-shared--spells-api)
9. [Esempi di utilizzo](#esempi-di-utilizzo)
10. [FAQ](#faq)

---

## Descrizione

**CR Menus** è un sistema completo per server FiveM basati su ESX che permette di gestire menù di abilità personalizzati (spell/skill), assegnare ruoli con gradi ai giocatori, e gestire aspetti cosmetici (occhi emissivi, fumo colorato, ped personalizzati, mantelli). Include una **Spellbar** e una **Hotbar** persistenti trascinabili, un'interfaccia NUI moderna e un sistema backend con MySQL per salvare i dati.

Il sistema è pensato per server **fantasy/roleplay** con classi, razze o fazioni (vampiri, licantropi, guardiani, ecc.), ma è completamente generico e adattabile a qualsiasi scenario.

> ⚠️ Il nome della risorsa **deve essere esattamente** `cr_menus`. All'avvio viene eseguito un controllo automatico: se il nome è diverso, il server si spegne dopo 10 secondi.

---

## Funzionalità

| Funzionalità              | Descrizione                                                                                                 |
| ------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **Menù Abilità (NUI)**    | Pannello grafico apribile con tasto (default F3) che mostra tutte le spell/abilità del giocatore            |
| **Spellbar**              | Barra rapida con fino a 12 slot, personalizzabile tramite drag & drop dalla UI                              |
| **Hotbar**                | Barra rotante (12 slot, gruppi da 4) per accesso rapido agli spell in-game                                  |
| **Sistema Menù + Gradi**  | Crea menù (es. "Vampiro") con più gradi (es. "Progenie", "Anziano"), ognuno con spell e statistiche proprie |
| **Spell personalizzati**  | Definisci spell in Lua con cooldown, icona, descrizione, logica `onUse`, lock condizionale                  |
| **Assegnazione Staff**    | I moderatori possono assegnare menù, gradi, occhi, fumo, ped e mantelli a qualsiasi giocatore online        |
| **Lista Giocatori**       | Pannello staff per cercare, visualizzare, modificare e revocare il menù di ogni giocatore                   |
| **Trasformazione**        | Stato di trasformazione gestito via StateBag, con callback personalizzabile                                 |
| **Sensoriale**            | Sistema radar di prossimità con blip dinamici configurabile per grado                                       |
| **KVP persistente**       | Spellbar e hotbar vengono salvati localmente sul client tra le sessioni                                     |
| **Database MySQL**        | Tutti i dati dei giocatori e dei menù creati sono salvati su MySQL via oxmysql                              |
| **Creatore Menù In-Game** | I moderatori possono creare, modificare ed eliminare menù e gradi direttamente in-game                      |

---

## Dipendenze

Installa e avvia **tutte** le seguenti risorse prima di `cr_menus`:

| Risorsa                                              | Obbligatoria | Note                                        |
| ---------------------------------------------------- | ------------ | ------------------------------------------- |
| [`oxmysql`](https://github.com/overextended/oxmysql) | ✅ Sì        | Driver MySQL asincrono                      |
| [`ox_lib`](https://github.com/overextended/ox_lib)   | ✅ Sì        | Libreria UI, callback, animazioni           |
| `cr_lib`                                             | ✅ Sì        | Libreria interna CR (fornita nel pacchetto) |

> **Database:** È necessario un database MySQL/MariaDB attivo e configurato in `oxmysql`. Le tabelle vengono create automaticamente all'avvio della risorsa.

---

## Installazione

### 1. Scarica e posiziona i file

Posiziona la cartella `cr_menus` nel percorso:

```
resources/[scripts]/[cr]/cr_menus/
```

Assicurati che la cartella si chiami esattamente **`cr_menus`**.

### 2. Aggiungi al server.cfg

```cfg
ensure cr_lib
ensure cr_menus
```

Assicurati che `cr_lib` sia avviata **prima** di `cr_menus`.

### 3. Configurazione database

Apri `sv_config.lua` e verifica il nome del database:

```lua
Sv_Config.DatabaseName = "cr_menus"          -- tabella assegnazioni giocatori
Sv_Config.CreatedMenuDatabaseName = "cr_createdmenu"  -- tabella menù creati
```

I nomi possono essere cambiati se necessario. Le tabelle vengono create in automatico.

Verifica anche il nome della tabella utenti ESX:

```lua
Sv_Config.usersTable = "users"  -- cambia se la tua tabella ESX ha un nome diverso
```

### 4. Avvia il server

Al primo avvio, `cr_menus` creerà automaticamente le tabelle MySQL necessarie. Verifica i log per eventuali errori di connessione al database.

---

## Configurazione

Tutto è configurabile in `config.lua`. Di seguito i blocchi principali.

### Evento di caricamento

```lua
Config.LoadEvent = 'esx:playerLoaded'
```

Cambia questo evento se il tuo framework usa un evento diverso per il caricamento del giocatore.

### UI

```lua
Config.UI = {
    SpellsBarSize   = 12,   -- numero di slot nella spellbar
    HotbarTotalSize = 12,   -- numero totale di slot hotbar
    HotbarGroupSize = 4,    -- slot per gruppo hotbar (navigabile con tasti)
    DefaultBaseColor = "#9E0E16",  -- colore base dei menù (esadecimale)
}
```

### Tasti (keybind)

I tasti sono registrati automaticamente da `ox_lib`. I giocatori possono rimapparli dalle impostazioni di GTA V > Tasti di scelta rapida.

| Azione                           | Default              |
| -------------------------------- | -------------------- |
| Apri lista spell                 | — (personalizzabile) |
| Mostra/nascondi spellbar         | —                    |
| Usa spell selezionato            | —                    |
| Sposta selezione destra/sinistra | —                    |
| Ruota hotbar avanti/indietro     | —                    |
| Pulisci KVP                      | —                    |

### Definire uno Spell

In `config.lua`, aggiungi i tuoi spell in `Config.SpellsList`:

```lua
Config.SpellsList = {
    ['nome_spell'] = {
        name        = "Nome Visibile",
        description = "Descrizione dello spell",
        cooldown    = 10,              -- secondi di cooldown
        icon        = "https://...",   -- URL icona (png/webp)
        class       = "Classe Opzionale",
        hideOnLocked = false,          -- nascondi se bloccato (true/false)

        -- Condizione di blocco (ritorna true = bloccato)
        lockSpell = function()
            return false
        end,

        -- Logica eseguita quando il giocatore usa lo spell
        onUse = function()
            -- scrivi qui la logica del tuo spell
            return true  -- ritorna true per avviare il cooldown
        end,
    },
}
```

### Icone personalizzate per menù

```lua
Config.MenuIcons = {
    nome_menu = {
        ['nome_spell'] = "https://example.com/icona.png",
    },
}
```

### Skin / Ped

```lua
Config.Skin = {
    base_maschio = { model = "mp_m_freemode_01", description = "Skin base uomo" },
    zombie       = { model = "u_m_y_zombie_01",  description = "Skin zombie" },
}

Config.Ped = {
    nome_ped = { model = "nome_modello", description = "Descrizione" },
}
```

### Mantelli (Cloacks)

```lua
Config.Cloacks = {
    ['nome_manto'] = {
        white = {
            { componente = 11, drawable = 0, texture = 0 },
            -- aggiungi altri componenti
        },
    }
}
```

### Colori Fumo

Il sistema include 20+ colori predefiniti in `Config.Smoke`. Puoi aggiungerne altri seguendo lo stesso pattern `{ label, r, g, b, a }`.

### Colori Marker

```lua
Config.MarkersColor = { r = 158, g = 14, b = 22, a = 255 }
```

### Trasformazione

```lua
Config.TransformationIdentifier = "isTransformed"

Config.onTransformationChange = function(menuName, gradeName, menuData, gradeData, isTransformed)
    -- eseguito ogni volta che un giocatore entra/esce dalla trasformazione
end
```

### Morte

```lua
Config.CheckDeath = function(cID)
    -- personalizza come verificare se il giocatore è morto
    local sID = GetPlayerServerId(cID or PlayerId())
    return GetStateBagValue('player:' .. sID, 'isDead')
end
```

---

## Comandi

Tutti i comandi sono configurabili nel blocco `Config.Commands` in `config.lua`.

| Comando         | Permesso     | Descrizione                                                        |
| --------------- | ------------ | ------------------------------------------------------------------ |
| `/menu`         | `user`       | Apre il pannello spell del giocatore (tasto default F3)            |
| `/gestionemenu` | `moderatore` | Apre il creatore menù in-game (crea/modifica/elimina menù e gradi) |
| `/playersmenu`  | `moderatore` | Apre la lista giocatori con i loro menù assegnati                  |
| `/setmenu`      | `moderatore` | Apre il pannello assegnazione menù/grado a un giocatore            |
| `/seteye`       | `moderatore` | Assegna un colore occhi emissivo a un giocatore                    |
| `/seteyenumber` | `moderatore` | Imposta il numero overlay degli occhi                              |
| `/setpedmenu`   | `moderatore` | Assegna un ped personalizzato a un giocatore                       |
| `/setsmoke`     | `moderatore` | Assegna un colore fumo a un giocatore                              |
| `/setmanto`     | `moderatore` | Assegna un mantello a un giocatore                                 |
| `/clearmenukvp` | `user`       | Cancella i dati salvati di spellbar e hotbar del giocatore         |

---

## Exports (Client)

Questi exports possono essere chiamati da qualsiasi altra risorsa lato client.

```lua
exports['cr_menus']:NomeExport(parametri)
```

### Spell e Cooldown

| Export              | Parametri                                  | Ritorna   | Descrizione                                |
| ------------------- | ------------------------------------------ | --------- | ------------------------------------------ |
| `GetActiveSpells`   | —                                          | `table`   | Restituisce gli spell attivi del giocatore |
| `IsSpellActive`     | `spellId: string`                          | `boolean` | Verifica se uno spell è attivo             |
| `SetActiveSpell`    | `spellId: string`                          | —         | Attiva uno spell                           |
| `RemoveActiveSpell` | `spellId: string`                          | —         | Rimuove uno spell dagli attivi             |
| `IsOnCooldown`      | `spellId: string`                          | `boolean` | Verifica se uno spell è in cooldown        |
| `StartCooldown`     | `spellId: string, durationSeconds: number` | —         | Avvia manualmente il cooldown              |
| `RefreshSpells`     | —                                          | —         | Forza il ricalcolo degli spell bloccati    |

### Dati Spell

| Export               | Parametri         | Ritorna      | Descrizione                                                    |
| -------------------- | ----------------- | ------------ | -------------------------------------------------------------- |
| `GetSpellData`       | `spellId: string` | `table\|nil` | Restituisce la definizione di uno spell da `Config.SpellsList` |
| `GetSpellIcon`       | `spellId: string` | `string`     | Restituisce l'URL icona di uno spell                           |
| `GetAvailableSkills` | —                 | `table`      | Restituisce le skill disponibili per il giocatore corrente     |

### Spellbar / Hotbar

| Export              | Parametri     | Ritorna | Descrizione                        |
| ------------------- | ------------- | ------- | ---------------------------------- |
| `GetSpellbar`       | —             | `table` | Legge la spellbar salvata (KVP)    |
| `SetSpellbar`       | `data: table` | —       | Salva la spellbar (KVP)            |
| `GetHotbar`         | —             | `table` | Legge la hotbar salvata (KVP)      |
| `SetHotbar`         | `data: table` | —       | Salva la hotbar (KVP)              |
| `ClearAllSavedData` | —             | —       | Cancella spellbar e hotbar salvate |

### Menù

| Export                | Parametri             | Ritorna      | Descrizione                                        |
| --------------------- | --------------------- | ------------ | -------------------------------------------------- |
| `IsMenuReady`         | —                     | `boolean`    | Verifica se il sistema menù è inizializzato        |
| `GetAllMenus`         | —                     | `table`      | Restituisce tutti i menù creati (callback server)  |
| `GetMenu`             | `menuName: string`    | `table\|nil` | Restituisce i dati di un singolo menù              |
| `RebuildMenu`         | `menuName, gradeName` | `table`      | Ricalcola e restituisce i dati computati del menù  |
| `GetComputedMenuData` | `menuName, gradeName` | `table`      | Legge i dati computati dalla cache                 |
| `RefreshMenus`        | —                     | `table`      | Aggiorna la cache locale dei menù e la restituisce |

### Gestione Menù (CRUD)

| Export                 | Parametri                               | Ritorna           | Descrizione                      |
| ---------------------- | --------------------------------------- | ----------------- | -------------------------------- |
| `CreateMenu`           | `menuLabel: string, baseColor: string`  | `boolean, string` | Crea un nuovo menù               |
| `DeleteMenu`           | `menuName: string`                      | `boolean`         | Elimina un menù                  |
| `UpdateMenuColors`     | `menuName, baseColor, fallbackIcon`     | `boolean`         | Aggiorna colore e icona default  |
| `CreateGrade`          | `menuName, gradeLabel, spells, sensory` | `boolean, string` | Crea un grado in un menù         |
| `UpdateGrade`          | `menuName, gradeName, newData`          | `boolean`         | Aggiorna i dati di un grado      |
| `DeleteGrade`          | `menuName, gradeName`                   | `boolean`         | Elimina un grado                 |
| `AddSpellToGrade`      | `menuName, gradeName, spellName`        | `boolean`         | Aggiunge uno spell a un grado    |
| `RemoveSpellFromGrade` | `menuName, gradeName, spellName`        | `boolean`         | Rimuove uno spell da un grado    |
| `GenerateMenuId`       | `label: string`                         | `string`          | Genera un ID pulito da una label |
| `GenerateGradeId`      | `label: string`                         | `string`          | Genera un ID pulito da una label |

### Apertura Pannelli Staff

| Export                    | Parametri | Descrizione                                |
| ------------------------- | --------- | ------------------------------------------ |
| `OpenAssignMainMenu`      | —         | Apre il pannello assegnazione menù/grado   |
| `OpenAssignEyeMenu`       | —         | Apre il pannello assegnazione occhi        |
| `OpenAssignEyeNumberMenu` | —         | Apre il pannello assegnazione numero occhi |
| `OpenAssignSmokeMenu`     | —         | Apre il pannello assegnazione fumo         |
| `OpenAssignPedMenu`       | —         | Apre il pannello assegnazione ped          |
| `OpenAssignMantoMenu`     | —         | Apre il pannello assegnazione mantello     |
| `OpenMenuCreator`         | —         | Apre il creatore menù                      |
| `OpenPlayersList`         | —         | Apre la lista giocatori                    |

### Gestione Assegnazioni Giocatori

| Export                     | Parametri                       | Ritorna      | Descrizione                                      |
| -------------------------- | ------------------------------- | ------------ | ------------------------------------------------ |
| `GetAllPlayerAssignments`  | —                               | `table`      | Restituisce tutti i giocatori con menù assegnato |
| `GetPlayerAssignment`      | `identifier: string`            | `table\|nil` | Dati menù di un giocatore specifico              |
| `UpdatePlayerAssignment`   | `identifier, newMenu, newGrade` | `boolean`    | Aggiorna menù e grado di un giocatore            |
| `RevokePlayerMenu`         | `identifier: string`            | `boolean`    | Revoca il menù di un giocatore                   |
| `ShowPlayerDetails`        | `identifier: string`            | `boolean`    | Apre la UI con i dettagli del giocatore          |
| `RefreshPlayerAssignments` | —                               | `table`      | Aggiorna e restituisce la cache assegnazioni     |

### Utility

| Export      | Parametri     | Ritorna  | Descrizione                          |
| ----------- | ------------- | -------- | ------------------------------------ |
| `Translate` | `key: string` | `string` | Traduce una chiave di localizzazione |

---

## Exports (Shared — Spells API)

La libreria `Spells` è disponibile lato client da qualsiasi risorsa:

```lua
local Spells = exports['cr_menus']:Spells()
```

### Utility Ped

| Metodo                                              | Parametri         | Descrizione                                          |
| --------------------------------------------------- | ----------------- | ---------------------------------------------------- |
| `Spells.setPedModel(model)`                         | `string\|number`  | Cambia il modello del ped del giocatore              |
| `Spells.setPedHealth(amount, fill)`                 | `number, boolean` | Imposta HP massimi (fill=true riempie completamente) |
| `Spells.setPedArmor(amount)`                        | `number`          | Imposta corazza massima e attuale                    |
| `Spells.setComponents(list)`                        | `table`           | Applica variazioni di componenti al ped              |
| `Spells.taskAnimation(dict, anim, flags, duration)` | —                 | Esegui un'animazione sul ped                         |

### Combattimento e Fisica

| Metodo                                                      | Parametri | Descrizione                                       |
| ----------------------------------------------------------- | --------- | ------------------------------------------------- |
| `Spells.performPhysics(entity, dir, power)`                 | —         | Applica una forza a un'entità                     |
| `Spells.spawnProiettile(info, tCoords, entityhit, cCoords)` | `table`   | Spawna un proiettile con particelle e fisica      |
| `Spells.toggleRun(bool)`                                    | `boolean` | Attiva/disattiva moltiplicatore corsa (1.35x)     |
| `Spells.toggleJump(bool)`                                   | `boolean` | Attiva/disattiva super salto                      |
| `Spells.toggleStrenght(bool)`                               | `boolean` | Attiva/disattiva modificatore danno corpo a corpo |
| `Spells.sendThunder()`                                      | —         | Triggera effetti fulmine attorno al giocatore     |

### Targeting / Mirino

| Metodo                                          | Ritorna                                    | Descrizione                             |
| ----------------------------------------------- | ------------------------------------------ | --------------------------------------- |
| `Spells.performAoeDrawMarker(toggle, canScale)` | `coords, scale, entity`                    | Mirino ad area con scaling scroll mouse |
| `Spells.performTargetDrawMarker(toggle)`        | `entity, coords`                           | Mirino su entità con linea visiva       |
| `Spells.targetMarker(toggle, range, maxTime)`   | `entity\|'cancelled'\|'outofTime', coords` | Mirino con timeout opzionale            |
| `Spells.rayCastGamePlayCamera(distance)`        | `hit, coords, entity`                      | Raycast dalla camera di gameplay        |

### Prossimità / Radar

| Metodo                                                                                | Parametri         | Ritorna            | Descrizione                        |
| ------------------------------------------------------------------------------------- | ----------------- | ------------------ | ---------------------------------- |
| `Spells.getClosestPed(coords, radius)`                                                | `vector3, number` | `ped, distance`    | Ped più vicino nel raggio          |
| `Spells.getClosestPlayer(radius, coords)`                                             | —                 | `player, distance` | Giocatore più vicino nel raggio    |
| `Spells.getPlayersAround(radius, coords)`                                             | —                 | `table`            | Lista giocatori nel raggio         |
| `Spells.getPlayersInRange(range, coords)`                                             | —                 | `table`            | Lista player ID nel raggio         |
| `Spells.closestPeds(playerPed, radius)`                                               | —                 | `table`            | Lista ped nel raggio               |
| `Spells.getEquidistantCircleCoords(count, radius, zOffset, startAngle, snapToGround)` | —                 | `table[vector3]`   | Coordinate equidistanti su cerchio |

### Stato Giocatore

| Metodo                                       | Ritorna   | Descrizione                                         |
| -------------------------------------------- | --------- | --------------------------------------------------- |
| `Spells.getPlayerDeadStatus(cID)`            | `boolean` | Verifica se il giocatore è morto (StateBag)         |
| `Spells.getPlayerIsTransformed(cID)`         | `boolean` | Verifica lo stato trasformazione (StateBag)         |
| `Spells.performPlayerTransformation(toggle)` | —         | Imposta lo stato trasformazione via callback server |

### Sensoriale

| Metodo                                                        | Parametri | Descrizione                                                                         |
| ------------------------------------------------------------- | --------- | ----------------------------------------------------------------------------------- |
| `Spells.sensory(range)`                                       | `number`  | Loop che emette l'evento `cr_spells:sensoriale` ogni secondo con i giocatori vicini |
| `Spells.createSensoriale(coords, color, sprite, scale, name)` | —         | Crea un blip temporaneo (1.2s) su un'entità                                         |

### Matematica / Direzioni

| Metodo                                      | Descrizione                                  |
| ------------------------------------------- | -------------------------------------------- |
| `Spells.rotationToDirection(rotation)`      | Converte rotation → direction vector (table) |
| `Spells.getDirectionFromRotation(rotation)` | Converte rotation → `vector3`                |

---

## Esempi di utilizzo

### Creare uno spell che lancia un proiettile

```lua
Config.SpellsList['palla_di_fuoco'] = {
    name        = "Palla di Fuoco",
    description = "Lancia una sfera infuocata verso il nemico.",
    cooldown    = 8,
    icon        = "https://mycdn.com/fireball.png",
    class       = "Offensivo",
    hideOnLocked = false,
    lockSpell   = function() return false end,

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

        return true  -- avvia cooldown
    end,
}
```

### Leggere il menù corrente di un giocatore (da altra risorsa)

```lua
-- Lato client
RegisterNetEvent('myresource:checkMenu', function()
    if exports['cr_menus']:IsMenuReady() then
        local pState = LocalPlayer.state.menu   -- StateBag
        local pGrade = LocalPlayer.state.grade
        print(("Menu: %s | Grado: %s"):format(pState, pGrade))
    end
end)
```

### Verificare e avviare un cooldown manualmente

```lua
local spellId = 'teletrasporto'

if exports['cr_menus']:IsOnCooldown(spellId) then
    -- spell in cooldown, non fare nulla
    return
end

-- esegui la logica dello spell...
exports['cr_menus']:StartCooldown(spellId, 15)  -- 15 secondi di cooldown
```

### Assegnare un menù a un giocatore da un'altra risorsa (client → server)

```lua
-- Se hai già i permessi, puoi aprire il pannello direttamente:
exports['cr_menus']:OpenAssignMainMenu()

-- Oppure, programmaticamente (client):
local ok = exports['cr_menus']:UpdatePlayerAssignment("license:abc123", "vampiro", "anziano")
if ok then print("Menù aggiornato!") end
```

### Creare un menù da codice

```lua
-- Crea il menù
local ok, menuId = exports['cr_menus']:CreateMenu("Vampiro", "#8B0000")
if not ok then return end  -- menuId = "vampiro" (auto-generato)

-- Crea un grado
local gradeOk, gradeId = exports['cr_menus']:CreateGrade(menuId, "Progenie", {
    "palla_di_fuoco", "teletrasporto"
})

-- Aggiorna colore
exports['cr_menus']:UpdateMenuColors(menuId, "#6B0000", "https://mycdn.com/vamp.png")
```

---

## FAQ

**Q: Posso usare questo script senza ESX?**
A: No. Il sistema si appoggia a `es_extended` per i permessi (`grade`) e all'evento `esx:playerLoaded`. Sarebbe necessario un porting significativo.

**Q: Le tabelle vengono create automaticamente?**
A: Sì. All'avvio della risorsa, se le tabelle non esistono vengono create automaticamente tramite `oxmysql`.

**Q: Posso cambiare il nome della risorsa?**
A: No. Il codice in `server/menu-events.lua` verifica che il nome della risorsa sia esattamente `cr_menus`. Cambiarlo causerà lo spegnimento automatico del server dopo 10 secondi.

**Q: Come aggiungo la localizzazione in inglese?**
A: In `config.lua`, nel blocco `Config.Intl.translations` aggiungi una chiave `en = { ... }` con le stesse chiavi del blocco `it`, poi cambia `Config.Intl.locale = 'en'`.

**Q: Posso avere più menù attivi contemporaneamente per un giocatore?**
A: No. Ogni giocatore ha un solo menù e un solo grado assegnato alla volta.

**Q: Gli spell sono eseguiti lato client o server?**
A: La funzione `onUse` degli spell viene eseguita lato client. Per azioni server-side (es. danni ad altri giocatori, registrazioni DB) devi usare `TriggerServerEvent` dentro `onUse`.

**Q: Come funzionano gli StateBag?**
A: Quando a un giocatore viene assegnato un menù, il server scrive automaticamente i valori su `Player(source).state` (replicato a tutti i client). Puoi leggere il menù di qualsiasi giocatore con `GetStateBagValue('player:' .. serverId, 'menu')` da qualsiasi risorsa.

---

_Documentazione generata per CR Menus v2.0.0 — Tutti i diritti riservati._

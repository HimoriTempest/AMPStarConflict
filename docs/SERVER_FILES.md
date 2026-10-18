# Server files handbook

What every file of the Star Conflict server does, in English. Most of the comments in the game's own files are in Russian. Their translations are here.

It covers the game's server folder, `server/cloud/` as AMP's file manager shows it, from build 1.99.0.167293. The template's own scripts (`hub-wrapper.sh`, `amp-settings.sh` and so on) are described in the [README](../README.md).

| Part | What's in it |
|---|---|
| [How the server is put together](#how-the-server-is-put-together) | Services, start order, how settings reach a service |
| [Services](#services) | Every `.exe`: what it does, its type ID, config file and port |
| [Launch and config files](#launch-and-config-files) | `cloud.xml`, the `.cfg` files, `datasources*.txt`, `prepare_shared.bat` |
| [Game data files](#game-data-files) | Every file in `cloud/data/`, grouped by topic, with the services that read it |
| [MongoDB folder](#mongodb-folder) | The game's own MongoDB 2.6, which the template doesn't run |
| [Where to change what](#where-to-change-what) | Quick lookup from "I want to change X" to the file |
| [default_platform_config.lua](server-files/default_platform_config.md) | **Separate page.** Every setting (cvar) in the main config file, translated |

Markers used below:
- **AMP** means the template rewrites the file or part of it on every start, so edit it through the AMP setting instead. See [Settings](../README.md) in the README.
- *Generated* means the file was exported from the game's design data (`data/gamedata/`), not written by hand. Editing it works, but a server update overwrites it.

## How the server is put together

```
server/
├── cloud/                     the server: all .exe files, configs, data
│   ├── Hub.exe                supervisor: starts and watches every other service
│   ├── <Service>.exe          about 20 services (see Services)
│   ├── DedicatedServer.exe    one per battle or open-space zone, started on demand
│   ├── data/                  server configs and game data (this handbook)
│   ├── mongodb/               the game's own MongoDB 2.6 (unused by the template)
│   └── datasources*.txt       where services look for data
├── data/                      the game's assets, including every .pak archive (shared with the client)
├── sys_data/                  engine data (shared with the client)
└── server-logs/               logs and crash dumps (the template's -outputDir)
```

**Start order.**
1. `Hub.exe` reads `cloud/data/cloud.xml`, which lists the services to run on this machine (the "node").
2. Hub starts each service. Higher `priority` starts first; ScriptsServer has priority 0 and starts last.
3. Every service connects back to Hub (port 3850) and to MongoDB, so MongoDB must be running before Hub starts.
4. Services start `DedicatedServer.exe` on demand: one per arena or PvE battle, and one per open-space zone ("zone instance").

**How a service gets its settings.** The order matters, because a later step overwrites an earlier one:

1. `common.cfg`, which ends with `exec common_local.cfg`.
2. The service's own `.cfg` (for example `shard.cfg`), which ends with `exec shard_local.cfg`.
3. `default_platform_config.lua`, sent by **ConfigServer** to each service after it has read its `.cfg` files. The Lua file checks `service_type` and runs only that service's section.

So a cvar that `default_platform_config.lua` sets can't be changed in a `.cfg` file: the Lua value wins. Change the Lua file for those. A `.cfg` setting only takes effect for cvars the Lua file leaves alone. `*_local.cfg` files don't ship with the game. They exist so you can override settings without editing the original, and the template writes several of them (ports, addresses).

**Settings and console.** A setting is called a *cvar*. In the remote console, `help <cvar>` shows its description, `set <cvar> <value>` changes it until the next restart. See [CONSOLE.md](CONSOLE.md).

## Services

Type IDs come from the comment in `cloud.xml`. *Started* means Hub starts the service from the default `cloud.xml`.

| Executable | Type | Started | What it does | Config | Port |
|---|---:|---|---|---|---|
| `Hub.exe` | 5 | (is Hub) | Supervisor. Reads `cloud.xml` and starts the services; every service registers with it. Hosts the remote console. | `hub.cfg` | 3850 (`hub_listenTo`), 9001 status |
| `LoadBalancer.exe` | 1 | yes | First contact for clients: login queue, picks a shard, tells the client the shard and chat addresses. Holds the player limit. | `loadbalancer.cfg` | **3801** (`lb_listenTo`) |
| `Shard.exe` | 2 | yes | The game's core: accounts, sessions, hangar, inventory, shop, mail and trading, squads, premium, battle pass. | `shard.cfg` | **3802** (`shard_listenTo`) |
| `MatchMaker.exe` | 3 | yes | Battle queues (arena, coop, PvE, brawl, sector battles, raids) and battle rewards rates. | `matchmaker.cfg` | |
| `DedicatedServer.exe` | 4 | on demand | The battle server: runs one match, or one open-space zone (then it's a "zone instance", type 15). Also builds the server XML files (`prepare_shared.bat`). | `dediserver.cfg` | **35000+** (`net_gamePort`) |
| `GameRewardDispencer.exe` | 7 | yes | Pays out battle rewards: XP, credits, bonus GS, leaver penalties. | `game_reward_dispencer.cfg` | |
| `Authenticator.exe` | 8 | yes | Login and authentication. | `authenticator.cfg` | |
| `ChatServer.exe` | 9 | yes | Chat channels, anti-spam, word filter. | `chat_server.cfg` | **3815** (`chat_server_port`) |
| `ConfigServer.exe` | 11 | yes | Sends `default_platform_config.lua` to every service. | | |
| `Universe.exe` | 12 | yes | The galaxy map: sector conquest, corporation influence, corporation wars. | | |
| `QuestServer.exe` | 13 | yes | Quests, daily tasks, event quests, battle pass quests. | `quest_server.cfg` | |
| `ZoneCoordinator.exe` | 14 | yes | Open space: starts zone instances and moves players between zones. | `zone_coordinator.cfg` | |
| `LobbyServer.exe` | 16 | yes | Custom games ("user lobbies") and PvE lobbies. | `lobby_server.cfg` | |
| `AchievementServer.exe` | 17 | yes | Achievements and medals. | `achievement_server.cfg` | |
| `LeaderboardServer.exe` | 18 | yes | Leaderboards and their weekly rewards. | | |
| `ClanServer.exe` | 19 | yes | Corporations: members, emblems, upgrades, dreadnought building. | | |
| `Tournament.exe` | 20 | yes | Scheduled tournaments (`tournaments.xml`). | | |
| `EarlyMatchMaker.exe` | 21 | **no** | Not in `cloud.xml`. Unused. | | |
| `ScriptsServer.exe` | 22 | yes, last | Runs server-side Lua scripts: open-space global events (alien invasions and so on). Its scripts are inside the `.pak` archives in `data/`. | | |
| `AdventureServer.exe` | 23 | yes | Open-space "adventures" (mission givers). Its scripts are in the `.pak` archives. | | |
| `StatServer.exe` | | **no** | Performance statistics collector, reads `stat_server_cfg.xml`. Not in `cloud.xml`. | `statserver.cfg` | |
| `CrashReporter.exe` | | | Sends crash reports to the developers. | `crash-reporter.cfg` | |

Type 6 (admin console) and 10 (deprecated) are never started. The three `.dll` files (`libcurl`, `libssl-1_1_tg`, `libcrypto-1_1_tg`) are OpenSSL and curl, used by the services for HTTPS.

## Launch and config files

### cloud.xml (AMP)

Read by Hub. Lists the machines ("nodes") and the services on each. The server has one node.

```xml
<Cloud debug="0">
  <CommonCVars>
    <CVar name="dbg_disableLuaDataFormatChecking" value="1" />   <!-- set on every service -->
  </CommonCVars>
  <Node address="127.0.0.1:3850" dediServerHost="1" id="1" region="1">
    <Service type="1" />  ...  <Service type="22" priority="0" />  <Service type="23" />
  </Node>
</Cloud>
```

| Attribute | Meaning |
|---|---|
| `Node address` | This machine's address and Hub port. Hub only starts if the address is one of its own network addresses. Remote clients are sent to this host, so it must be the address players reach. The template sets it from Public Address or AMP's advertised address. |
| `dediServerHost="1"` | This node may run battle servers. |
| `region` | Region ID of the node. |
| `Service type` | A service to run (type IDs in [Services](#services)). |
| `Service priority` | Start order: 2 high, 1 medium (default), 0 low. Higher starts first; lower ones start only after all higher ones are up. |
| `Service addr`, `path` | (Not used here.) Host to start the service on, and the absolute path to its executable on that host. |
| `CommonCVars` | Cvars set on every service. |

### The .cfg files

Plain console commands, one per line: `set <cvar> <value>`, `exec <file>`, `bind`. `//` starts a comment. All of them are in `cloud/data/`.

| File | Read by | Contents (translated) |
|---|---|---|
| `common.cfg` | every service and battle server | `set cl_maxFps 60`, `bind alt+f4 "quit"`, commented-out Redis host and version type. Then `exec common_local.cfg`. **AMP** writes the External MongoDB address into `common_local.cfg`, in a marked block; the rest of that file is yours. |
| `hub.cfg` | Hub | `hub_listenTo "3850"`, `hub_detachChildProcesses 1` (presumably: services run as independent processes, not as Hub's children). **AMP**: `hub_local.cfg` sets the Hub port, status port and console. |
| `loadbalancer.cfg` | LoadBalancer | `lb_listenTo "3801"`. **AMP**: port in `loadbalancer_local.cfg`. |
| `shard.cfg` | Shard | `shard_listenTo "3802"`, `steam_useMicroTxnSandbox 1` (Steam purchases in sandbox mode), `sm_overloadCounterLimit 20`. **AMP**: `shard_local.cfg`. |
| `chat_server.cfg` | ChatServer | `chat_server_port "3815"`. **AMP**: `chat_server_local.cfg`. |
| `dediserver.cfg` | battle servers | See below. **AMP**: `dediserver_local.cfg` (battle port, public address). |
| `statserver.cfg` | StatServer | `ss_averagingPeriod "60.0"`. |
| `achievement_server.cfg`, `authenticator.cfg`, `game_reward_dispencer.cfg`, `lobby_server.cfg`, `matchmaker.cfg`, `quest_server.cfg`, `zone_coordinator.cfg` | the matching service | Only `exec <name>_local.cfg`. (`zone_coordinator.cfg`'s comment wrongly says "config for achievement server".) **AMP** writes `game_reward_dispencer_local.cfg`. |
| `crash-reporter.cfg` (in `cloud/`) | CrashReporter | Empty: "empty config to reset crash reporter to defaults". |

`dediserver.cfg`, translated:

```
// config for dedicated server

// The server should run at 30 fps (a multiple of sv_updateRate=15). It comes out slightly below
// cl_maxFps, so set the cvar a bit higher.
set cl_maxFps 32

// Lua is hardly used, so no significant garbage collection is needed
set script_gcWorkBudget 0

set s_masterVolume 0              // no sound
set script_initRemoteDebugger 0
set ship_controlForceMode -1      // don't force a ship control mode
set tm_enable 0                   // no async task manager needed
set ssh_port 2222                 // remote console port of battle servers
set ai_logSquads 5                // log everything about AI squads
set ai_convoyDefendRadius 300

// Loot pickup time
set sv_lootCrateScanTime 3
// Targeting distance of the various patrols
set ai_kiteDistance 5000

exec dediserver_local.cfg
```

`default_platform_config.lua` sets several of these again (`cl_maxFps 18`, `script_gcWorkBudget 1`, `sv_lootCrateScanTime 1`), and its values win.

### default_platform_config.lua (AMP)

The main settings file: about 300 settings for every service, matchmaking queues, loot rules, leaderboards, event dates. **Full translated reference: [server-files/default_platform_config.md](server-files/default_platform_config.md).**

### datasources*.txt

Search paths for data files, relative to `cloud/`. A service looks for a file in each folder in turn.

| File | Paths | Used by |
|---|---|---|
| `datasources.txt`, `datasources_cloud.txt` | `.\data`, `..\data`, `..\sys_data`, `..\local_data` | the services |
| `datasources_defs_build.txt` | `.\data`, `..\data`, `..\sys_data` | `prepare_shared.bat` |

`..\local_data` doesn't exist on the server. `..\data` holds the `.pak` archives, which is why the server needs all of `data/` even though it never draws anything: services load scripts such as `scripts/scripts_server/main.lua` from inside them.

### prepare_shared.bat

```bat
rem build xml's
DedicatedServer.exe -logFileName prepare_shared.log -nullCon -prepareSharedData -ds datasources_defs_build.txt
```

Rebuilds the server's XML files (shop, quests, battle pass and the other *generated* files below) from the game's design data in `data/gamedata/`. The developers ran it before each release. You'd only need it after changing game definitions, and it overwrites the generated files, including the template's edits (the next start reapplies those).

## Game data files

Everything in `cloud/data/` apart from the configs above. "Read by" lists the services whose executables contain the file name. `DedicatedServer` also appears for files it only *writes* through `prepare_shared`.

IDs that appear across files:
- **Race** (faction): 0 Empire, 1 Federation, 2 Jericho, 3 and 4 alien ships (Ze'Ta, Helicon), 5 any. Ship names count differently: `Ship_Race1` is Empire, `Ship_Race2` Jericho, `Ship_Race3` Federation.
- **Ranks**: comma-separated lists of 18 values are one value per ship rank, starting at rank 1.
- **Currencies**: `credits` = credits; `gold`/`goldCredits`/`premiumPrice` = Galactic Standards (GS); `tokenCredits`/`tokenPrice` = iridium; `eventCredits`/`eventPrice` = event currency (xenochips during the battle pass); `price` alone is credits.
- **defName**: internal item name, as in [ITEMS.md](ITEMS.md).
- Durations are in **hours** unless an attribute says seconds.

### Accounts and economy

| File | Read by | What it is |
|---|---|---|
| `starter_pack.xml` (AMP) | Shard | What a new account gets: `credits`, `goldCredits`, `tokenCredits` and the starting ship (`Ship_Race1_M_T1`, put in battle slot 1). The starting currency in `default_platform_config.lua` comes on top. |
| `special_values.xml` (AMP) | Shard and others | Prices and exchange rates for everything that isn't a shop item. See [special_values.xml](#special_valuesxml) below. |
| `aux_limits.xml` (AMP) | Shard and others | Caps, rank tables, experience curves, trading rules, raid schedule. See [aux_limits.xml](#aux_limitsxml) below. |
| `karma.xml` | AchievementServer | Open-space karma levels: 1000, 15000, 50000, 150000, 500000. |
| `faction_rep.xml` | (shared library) | Starting reputation with the five open-space factions (0, 0, 0, 0, −50000) and the maximum (50000). |
| `account_auras.xml` | Shard, QuestServer, GameRewardDispencer | 397 account "auras": timed account effects such as boosters (`credits_reward_mul`), discounts (`gold_credits_mul="-0.5"` = 50 % off GS), unlocks granted by bundles. `durationInHours`, `isProlongable` (buying again extends it), `keepExpired`, `strParam` (linked bundle), `AltDiscountDef` (bundle the discount applies to). |
| `offers.xml` | Shard | Personal shop offers (daily chest, rank-up packs, GS bonus offers), in groups, with conditions and a `cooldown`. Only the offer IDs listed in the cvar `sv_offerEnable` are active (`6,7,17,18,19`). `priority` presumably decides which offer of a group is shown. |
| `slot_machines.xml` | *(no reference found)* | Two slot machines ("SimpleSilver" and another): price, prize combinations and probabilities. Probably a removed feature. |
| `regions.xml` | Shard | Which countries belong to which region (RU, EU, US, AS), by two-letter country code. |
| `geo.gbin` | Shard, DedicatedServer | Binary GeoIP table: looks up a player's country from the IP address, which `regions.xml` then maps to a region. Not human-editable. |

### Shop

| File | Read by | What it is |
|---|---|---|
| `store.xml` (AMP, *generated*) | Shard and others | **The shop and item catalogue**: all 14,432 items, 15 MB. Every item the game knows is here, sold or not. See [store.xml](#storexml) below. The item lists in [ITEMS.md](ITEMS.md) are built from it. |
| `store_rules.xml` (*generated*) | Shard and others | 1,477 conversion rules applied to inventories, for items that were renamed, merged or removed: `<Rule type="def" defName="old" newDefName="new" credits="…" goldCredits="…" silent="1">`. A player who owns the old item gets the new one or the money instead; `silent` means without notifying them. |
| `defmod_rules.xml` | *(no reference found)* | An older, mostly commented-out version of the same rules. Its Russian template line translates to: `defName="[source def]" goldCredits="[GS to give]" credits="[credits to give]" newDefName="[replace with def]" silent="[do it silently]"`. |
| `defmod_rules_new.xml` | *(no reference found)* | The export the current rules were made from ("generated by Targem Export script from file 'export_subtypes.ods'"). Not valid XML (it starts with `--` comments); not loaded. |
| `stickers.xml` | Shard | 210 ship stickers (decals) and their prices: 24 h for credits, 7 days for GS, permanent (`duration="-1"`) for GS. |
| `vessel_custom_elements.xml` | Shard | 1,876 ship cosmetics (paints, decals, patterns): `type`, `group`, price per ship rank, `cantBeBought`, corporation-only elements (`<Clans>`). |
| `vessel_custom_elements_rules.xml` | Shard | Which cosmetic slot groups each ship (2,398 hulls) has: `<Slot index groupMask>`. |
| `titles.xml` | Shard | The 343 player titles: ID, internal name, duration (0 = permanent). The IDs are what `give <uid> title` takes. |
| `clan_emblems.xml` | ClanServer, Shard | 488 corporation emblems: GS and iridium price, and which corporations (`cid`) own each. Many are custom emblems made for one corporation of the live server. |

### Ships, modules and bots

| File | Read by | What it is |
|---|---|---|
| `vessel_presets.xml` (*generated*) | Shard and others | 361 ready-made ship loadouts (`<Module slot def>`, `<Munition slot def>`), for premium ships that come fitted, bots and tournaments. |
| `autogen.xml` | Shard, LobbyServer | Rules for randomly generated modules ("seed chips"): rarity classes, affixes, level costs, dismantle prices. The feature is off (`sv_autogenModulesEnabled = 0`). `autogen_special_values.xml` holds its version and element count. |
| `atlas_special_values.xml` | most services | Version and number of elements of the "atlas" (an internal data set; 1205 elements). |
| `skills.xml` | Shard | Ship talents (implants): three choices per ship rank (`TalentGroup requiredShipRank talent0..2`), and the GS cost to reset a set. |
| `league_equipment_lists.xml` | Shard, MatchMaker and others | Ships and module subtypes allowed in leagues, and exceptions. The additional banned list is `leagueForbiddenDefs` in `default_platform_config.lua`. |
| `bot_preset_exclusion_lists.xml` | Shard, MatchMaker and others | Ships and modules bots must never be given (tournament items, guardian ships, Apeiron). |
| `loot_tables.xml` | DedicatedServer | Three PvE loot tables (`PVE_T1_PassiveSet` …): shop item ID and drop chance. The main loot rules are in `default_platform_config.lua` (`DedicatedServcerServices_SetLootRules`). |

### Quests, achievements, events

| File | Read by | What it is |
|---|---|---|
| `quests.xml` (AMP, *generated*) | QuestServer, Shard | All 912 quests, 286 KB. See [quests.xml](#questsxml) below. The template multiplies the xenochip rewards in it. |
| `ship_quests.xml` | Shard | Daily per-ship tasks: how many per rank (`questsMax`), which ship roles each applies to. |
| `clan_quests.xml` | ClanServer | Corporation quests. Empty. |
| `achievements.xml` | AchievementServer | 250 achievements: conditions (`numericalParam`, `strEqParam`), ranks with points, GS and title rewards (`rewardTitle` = ID from `titles.xml`). |
| `medals.xml` | AchievementServer | 62 battle medals and their categories (multi-kill, spree, survive, special kills, ship class). |
| `battle_pass.xml` (AMP, *generated*) | Shard | The battle pass rewards: per stage, one `<reward bundle cost gold>` per level. `gold="true"` is the premium track. The dates are in `default_platform_config.lua`. See [BATTLE_PASS.md](BATTLE_PASS.md). |
| `tournaments.xml` | Tournament | Tournament schedule: `d` = weekday, `h`/`m` = start time, rank range, bot presets, GS rewards per number of wins, reward multipliers per round. Two tournaments, `d="0"` and `d="6"` at 17:00 and 16:00; if 0 is Monday as in the matchmaker's reward table, that's Monday and Sunday (untested). `registrationTime`, `roundTime` and the like are presumably minutes. |
| `universe.xml` | Universe and others | Sector conquest: conflict duration, civilian zone cost and duration, move cost factors, and the days of the New Year event stages. |
| `war.xml` | Universe and others | Corporation wars: cost to revive a corporation in the war (`warClanReviveCost`) and the base reward. |

### Open space and corporations

| File | Read by | What it is |
|---|---|---|
| `zones.xml` | almost every service | The 130 open-space and sector-map zones. See [zones.xml](#zonesxml) below. |
| `clan_ship.xml` | ClanServer and others | Dreadnought parts and modules: iridium price and build time (`duration`), and the dreadnought hulls per faction. |

### Chat

| File | Read by | What it is |
|---|---|---|
| `chat/stop-list.txt` | ChatServer | Word filter: roots of obscene words, one per line, lowercase. Empty here (only the header). Translated header: "List of obscene word roots (lowercase). Words containing one of these roots are *potentially* filtered. After this list, the words are checked against the exception list in white-list.txt. A word is filtered only if it contains a root from the stop list and NO root from the white list." The file must be UTF-8 with BOM. |
| `chat/white-list.txt` | ChatServer | Exceptions for the filter, one root per line. Empty. Translated header: "List of exception roots (lowercase). Words containing these roots are NOT filtered (see the obscene roots file stop-list.txt)." |

The anti-spam settings are in `default_platform_config.lua` ([Chat server](server-files/default_platform_config.md#chat-server)).

### Monitoring and leftovers

| File | Read by | What it is |
|---|---|---|
| `stat_server_cfg.xml` | StatServer | Which statistics StatServer collects from which service (`localhost:4000` LoadBalancer, `4001` Shard, `4002` MatchMaker, …). StatServer isn't started, so this does nothing. |
| `sstring.flog` | *(no reference found)* | A list of 380 MongoDB field and operator names (`$addToSet`, `accounts`, `acquiredStickers`, …). A leftover from the database layer's string table. |

### special_values.xml

Prices and rates that aren't shop items. The file's comments are empty except where noted.

| Element | Value | Meaning |
|---|---|---|
| `GoldExchangeRate` | 400 | Exchange rate between GS and credits. |
| `BonusBundle` | 10000 points, `Bundle_Chest_Gold_Reward` | A bonus chest and the points it takes (how the points are earned isn't documented). |
| `SalvageTokenConversion`, `SalvagePremiumTokenConversion`, `SalvagePremiumPriceConversion` | 0.1, 0.5, 0.00055 | Salvaging loot into iridium (normal and paid). The battle server uses its own cvars (`sv_salvage*`). |
| `BuybackCostModifier`, `VesselBuybackCostModifier` | 0.1, 0.25 | Price to buy back a sold module / ship, as a share of its price. |
| `DeconserveCostModifier` | 0.8 | Price to take a ship out of storage, as a share. |
| `MailTrading tax` | 0.1 | Tax on mail trades. |
| `DurabilityRestoreCostModifier` | 0.0508 | Repair cost factor. |
| `FactionChangeCostPerRank` | 30000 | Credits per rank to change faction. |
| `NickNameChangeCost` | 3000 | GS to change the nickname. |
| `CreditPacks` | | The six credit packs sold for GS: GS price, credits, bonus credits. |
| `InventoryExtensions` | base 50 | Hangar slot extensions: size and price. The 3000-slot step is commented out: "This extension was postponed until spring 2026 (if there are no complaints about the extension to 2750 slots, we'll open it)". |
| `AutogenInventoryExtensions` | base 10 | Same for seed-chip storage. |
| `FreeRepairs` | 3 × 259200 | Free repair charges, recharge time in seconds (72 h). |
| `PremiumTimePrices` | | Premium licences: hours and GS price. The `hidden` set is a discounted variant not shown in the shop. |
| `BattleSlotPrice` | 5000 GS, account rank 12 | A battle slot unlocks automatically at the account rank or can be bought with GS. |
| `ClanRegistrationCost` (AMP) | 1500 iridium | Founding a corporation. |
| `ClanChangeNameCost`, `ClanChangeTagCost`, `ClanChangeRaceCost` | 1000 iridium, 1000 iridium, 1,000,000 credits | Renaming, new tag, changing the corporation's faction. |
| `ClanUpgrades` | | Corporation upgrades: member capacity (13 levels, 20 to 500 members, as the comments say) and resource limit (5 levels, 1000 to 5000). |
| `ClanRecourceExchangeRate` | | Exchange rates for corporation iridium (from credits, GS, over-cap, for dreadnought building and repair). |
| `SocialRewards` | 24 h premium each | Reward for linking Facebook, VK, Steam. |
| `AutoPilotCosts` | all 0 | Open-space autopilot cost per rank. |
| `KarmaResetCost`, `FactionRepResetCost` | 500 GS, 2000 GS | Resetting karma or faction reputation. |
| `ArcZenExchangeRate` | 3.2 | GS per Zen (Arc/Perfect World's currency): `goldCredits = Zen × rate`. Unused here. |
| `TimeIntervalOffset` | 0 | Shift for daily resets. |
| `BattlePassPersonalOffers` | empty | "Personal discount offers for completing BattlePass stages". |

### aux_limits.xml

A collection of limits and tables, all at the top level of the file:

| Element | Meaning |
|---|---|
| `ReputationRank` | Faction reputation needed for each of the 18 ranks. |
| `VesselExpPoolCap` | Maximum free ship XP: 5,000,000. |
| `EventCreditsCap` (AMP) | Maximum event currency (xenochips): 300. |
| `CreditsCap`, `TokensCap` | Maximum credits (400,000,000) and iridium (10,000). |
| `VesselLvl`, `VesselMaxLvl`, `VesselMaxLevel` | Ship XP needed per level within each rank, and the highest level per rank. |
| `LobbySettings` | Which maps and game modes custom games may use (`forbiddenOnLive`). |
| `PVEMissions` | PvE missions: ship rank range, account rank, `disabled`, `raid`, difficulty. |
| `ReferralsSettings` | Rewards for recruiting players. |
| `TutorialRewards` | Rewards per tutorial stage. |
| `DailyAuras` | The daily first-win auras (normal and 2×). |
| `Teaching` | Mentor system: student max rank 6, teacher min rank 9 and 200 battles, max 5 students. |
| `RaidSchedule` | When which raid map is open: `start`/`finish` hours. |
| `LeagueTeamSettings`, `FootballSettings` | League and football team sizes and requirements. |
| `ClanUpgrades` | Corporation upgrade levels. |
| `ClanHighRank` (AMP) | Rank members need to count towards dreadnought building: 9. |
| `LockRank` | One list of 18 numbers per faction (meaning not confirmed; probably the rank requirement to unlock each ship rank). |
| `TradingLimits`, `TradingTax` | Trading: max 3 attachments, deals per day (0 = unlimited), offer expiry (seconds), max 5 adverts; tax brackets by amount. |
| `FreeSpace` | Price to jump to a zone. |
| `AccountRank`, `AccountExpGains` | Account XP per rank, XP per source. |
| `MilitaryRank` | Corporation-war military ranks (up to 15) and their rewards. |
| `UnlimPve` | Formulas for rewards in endless PvE. |

### store.xml

One `<Item>` per item, followed by its details. Item attributes:

| Attribute | Meaning |
|---|---|
| `id` | Shop ID, used by `give <uid> item`. |
| `defName` | Internal name. `localizedName` is the English display name. |
| `type` | 0 ship, 1 module or weapon, 2 ammo and missiles, 3 consumable, 4 bundle, 5 resource, 6 blueprint, 7 portrait, 8 taunt (motto), 9 loot (junk). |
| `price`, `premiumPrice`, `tokenPrice`, `eventPrice`, `tradePremiumPrice` | Current price in credits, GS, iridium, event currency, and GS in trading. 0 = not for that currency. |
| `base…Price` | The undiscounted price. |
| `requiredRank`, `race`, `requiredAccountAura` | Ship rank and faction (5 = any) needed, account aura needed. |
| `cantBeBought="1"` | Not sold. The template's settings remove it for some items. |
| `stacks`, `deleteFromInventory` | Stackable; removed from inventories. |
| `flags` | Shop flags; `IS_SHOW` makes the client show hidden bundles (template: DLC Packs In Shop). |

The child elements describe the item: `<Vessel>`, `<Module>` and so on with the item's data, `<Bundle>` with the bundle's `<Content>`, `<Blueprint>` with ingredients (`<Ingr>`), crafting and dismantling (`<Disassemble>`, `<NodePrice>`). A few display names contain a Cyrillic "С" in place of the Latin C (`Сorsair protocol "Apollo's gift"`), a typo in the original data.

### quests.xml

`<QuestGroups>` first: groups with offer and completion cooldowns (hours) and the quests in each. Then one `<Quest>` per quest:

| Attribute | Meaning |
|---|---|
| `id`, `name`, `group` | ID, internal name, group. |
| `unlocksOnAccountRank`, `locksOnAccountRank` | Account rank range. |
| `intervalStartHour`, `intervalLongHours`, `intervalLongEnableHours` | Repeating availability window: from the start date, every N hours, for M hours. |
| `maxCompletions`, `autoAccept`, `autoComplete`, `saveProgress`, `disabled`, `hidden` | Behaviour flags. |
| `rewardCredits`, `rewardGoldCredits`, `rewardTokens`, `rewardEventCredits`, `rewardVesselExp`, `rewardAccountAura`, `rewardRankUp` | Rewards: credits, GS, iridium, event currency (xenochips), ship XP, an aura, a rank-up. |
| `eventDayMin`/`Max`, `requiredAccountAura`, `acceptableDuring` | Event conditions. |

Each `<stage>` has a `gameEvent` (what counts, such as kills or wins) and `requiredValue`, with `<restriction param>` filters (map, game mode, ship type).

### zones.xml

The `<SpaceZones>` element names each faction's start zone and dreadnought zone. One `<Zone>` per location:

| Attribute | Meaning |
|---|---|
| `id`, `name`, `levelName` | Zone ID, internal name, map. |
| `minAccountRank`, `maxPlayers` | Entry rank and capacity per instance. |
| `civilianZone` | Marks a civilian zone (meaning not confirmed; `universe.xml` has a cost and duration for civilian zones). |
| `universeType`, `stellarSystem`, `tier`, `raceDefault` | Map type, star system, tier (1–5), owning faction. |
| `universeMinShipRank`, `universeMaxShipRank` | Ship ranks for sector battles there. |
| `factorPressure`, `conflictIntervalStart`, `conflictIntervalLong` | Sector conquest: influence factor, when conflicts happen. |
| `station`, `stationRace`, `clanShipRace`, `allowedJump`, `onlyForWing` | Station in the zone, dreadnought zone, jumping allowed, wings only. |

`<Link id>` lists the neighbouring zones, `<Reward>` what owning the zone pays (for example iridium monocrystals).

## MongoDB folder

`cloud/mongodb/` is the game's own MongoDB **2.6.12 for Windows**, which the single-player client starts on the player's PC. **The template doesn't run it.** It runs MongoDB 3.4 instead, and only uses `mongod.exe` here once, to migrate data from template versions before 2.0.

| File | What it is |
|---|---|
| `mongod.exe`, `mongo.exe`, `mongodump.exe`, `mongorestore.exe`, … | MongoDB 2.6 server and tools. |
| `config.cfg` | `dbpath = ./data`, `bind_ip = 127.0.0.1`. |
| `launch.bat` | Starts `mongod` with its data in `%LOCALAPPDATA%\Targem\StarConflict\cloud\mongodb\data`. |
| `admin.bat` | Opens a MongoDB shell on the game database: `mongo.exe 127.0.0.1/cosmosim`. |
| `mongo-r2.6.12.zip` | MongoDB's source code, kept for the AGPL licence. |
| `GNU-AGPL-3.0`, `README`, `THIRD-PARTY-NOTICES` | MongoDB's licence and notes. |

The game's databases are `cosmosim` (accounts, inventory, everything), `stats_cosmosim`, `achievements`, `logs`, `games`, `delivered` and `web_adm` (admin roles). Under Wine each name gets the user as a suffix (`cosmosim_amp`).

`cloud/Exceptions/` (if present) holds crash dumps from the client's local server. On the AMP server they go to `server-logs/cloud_exceptions/`.

## Where to change what

Check the template's [settings](../README.md) first. Most common changes are an AMP setting, and anything the template writes is overwritten on the next start.

| I want to change… | File | Where |
|---|---|---|
| Starting money and ship | `starter_pack.xml`, `default_platform_config.lua` | `credits_on_start`, `gold_on_start` (AMP: Starting Credits/GS) |
| XP and credit rates | `default_platform_config.lua` | `rate_exp`, `rate_cred`, `sv_expRate` (AMP: multipliers) |
| Shop prices of one item | `store.xml` | the item's `price`/`premiumPrice` |
| All ship/module/ammo prices | `default_platform_config.lua` | `store_*PriceMod` (AMP) |
| Premium, nickname, hangar slot prices | `special_values.xml` | `PremiumTimePrices`, `NickNameChangeCost`, `InventoryExtensions` |
| Currency caps | `aux_limits.xml` | `CreditsCap`, `TokensCap`, `EventCreditsCap` |
| Daily iridium limit | `default_platform_config.lua` | `sv_tokenCreditsDailyLimit` (AMP) |
| Bots in matches | `default_platform_config.lua` | `mm_botsCount`, queue `botsCount` (AMP: Ships In Arena Matches) |
| Maps in the queues | `default_platform_config.lua` | `levelsDesc` in the queue templates ([Matchmaker](server-files/default_platform_config.md#matchmaker)) |
| Brawl schedule | `default_platform_config.lua` | `brawlSchedule` ([Shard](server-files/default_platform_config.md#brawl-schedule)) |
| PvE mission rotation | `default_platform_config.lua` | `pveSchedule` |
| Loot drop rates | `default_platform_config.lua` | `sv_loot*`, `DedicatedServcerServices_SetLootRules` |
| AFK kick | `default_platform_config.lua` | `sv_afkCheckDuration` (AMP, battles); 3600 in open space |
| Leaver penalties | `default_platform_config.lua` | `sv_leave*`, `sv_leaver*` |
| Chat spam limits | `default_platform_config.lua` | `antispam_*` |
| Chat word filter | `chat/stop-list.txt`, `white-list.txt` | one root per line |
| Corporation costs | `special_values.xml` | `ClanRegistrationCost` (AMP) and the other `Clan*` entries |
| Dreadnought requirements | `default_platform_config.lua`, `aux_limits.xml` | `clan_shipBuildHighRanks`, `ClanHighRank` (AMP) |
| Tournament times | `tournaments.xml` | `d`, `h`, `m` |
| Holiday events | `default_platform_config.lua` | `event_*` cvars |
| Battle pass | AMP settings only | see [BATTLE_PASS.md](BATTLE_PASS.md) |
| Player limit, close the server | `default_platform_config.lua` | `lb_limitPlayersOnShardTo` (AMP: Max Players), `lb_disallowAccess` |
| Ports | AMP port settings | the template writes `*_local.cfg` |

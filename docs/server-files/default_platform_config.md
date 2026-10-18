# default_platform_config.lua

`server/cloud/data/default_platform_config.lua` is the server's main settings file. This page lists every setting in it, in the file's order, with the Russian comments translated. Part of the [server files handbook](../SERVER_FILES.md).

## How the file works

ConfigServer sends the file to every service after the service has read its `.cfg` files. A global variable, `service_type`, tells the file which service it is running in. The end of the file calls the matching function:

| Function | Service | Section |
|---|---|---|
| `ConfigureShard()` | Shard | [Shard](#shard) |
| `ConfigureMatchmaker()` | MatchMaker | [Matchmaker](#matchmaker) |
| `ConfigureScriptsServer()` | ScriptsServer | [Scripts server](#scripts-server) |
| `ConfigureDediServer()` | battle servers | [Battle servers](#battle-servers) |
| `ConfigureDediServerFlags()` | battle servers, per queue | [Queue flags](#queue-flags) |
| `ConfigureZoneInstance()` | open-space zone servers | [Open space](#open-space-zone-instances) |
| `ConfigureGRD()` | GameRewardDispencer | [Reward dispenser](#reward-dispenser) |
| `ConfigureLoadBalancer()` | LoadBalancer | [Load balancer](#load-balancer) |
| `ConfigureChatServer()` | ChatServer | [Chat server](#chat-server) |
| `ConfigureUniverse()` | Universe | [Universe](#universe-sector-conquest-and-corporation-wars) |
| `ConfigureQuestServer()` | QuestServer | [Quest server](#quest-server) |
| `ConfigureAchievementServer()` | AchievementServer | [Achievement server](#achievement-server) |
| `ConfigureLobbyServer()` | LobbyServer | [Lobby server](#lobby-server) |
| `ConfigureLeaderboardServer()` | LeaderboardServer | [Leaderboards](#leaderboards) |
| `ConfigureZoneCoordinator()` | ZoneCoordinator | [Zone coordinator](#zone-coordinator) |
| `ConfigureTournament()` | Tournament | [Tournament](#tournament) |
| `ConfigureClanServer()` | ClanServer | [Clan server](#clan-server) |
| `ConfigureAdventureServer()` | AdventureServer | [Adventure server](#adventure-server) |
| `ConfigureHub()` | Hub | [Hub](#hub) |

A setting is written `cvars.<name> = <value>`. Some settings appear in several sections because each service only sees its own section: `x3fw`, for example, is used by battle servers, open space and the load balancer.

Settings marked **AMP** are written by the template from an AMP setting (in a block appended to the end of the file, or edited in place). The values below are the game's originals. Edit the file outside the AMP block for anything else, and restart the server: the file is only read at start.

Types: *bool* is 0 or 1. *Rank list* is a string of 18 comma-separated values, one per ship rank starting at rank 1. Dates are ISO 8601 in UTC, `YYYY-MM-DDThh:mm:ssZ`.

## Top of the file

Shared values that the sections below refer to.

| Variable | Value | Meaning |
|---|---|---|
| `x3fw` | 3 | **AMP** (First Win Of The Day XP Multiplier). XP multiplier for the first win of the day. Used by battle servers and open space (`sv_dailyWin*ExpRewardCf`) and sent to clients in the login greeting. |
| `store_vesselExpExtractionCost` | 0.006666666 | Cost of converting ship XP into free XP. See [Shard](#shard). |
| `event_custom` | `2023-04-16T00:00:00Z` | Start of the "custom" event. Comment: "Start and end times must be changed in whole hours, otherwise the *imaginary* quest can behave unpredictably." |
| `creditsEventStart`, `creditsEventEnd` | `2023-06-09T05:00:00Z`, `2023-09-14T00:00:00Z` | **AMP** (Battle Pass dates). The event-currency period. Xenochips are only paid while it runs. |
| `battlePassStart` | `2023-06-09T05:00:00Z` | **AMP**. Battle pass start. |
| `battlePassEndTable` | stages 1–3, all ending `2023-09-14T00:00:00Z` | **AMP**. Active battle pass stages and their end times. |
| `leagueForbiddenDefs` | 24 defs | Ships and modules banned in leagues: the New Year weapons, several unique modules and the "Glass" event modules. |
| `botPresetForbiddenDefs` | `Weapon_PlasmaWebLaser_T5_*`, `Ship_Race5_M_T5_Apeiron` | Items bots' adaptive loadouts may never contain. |
| `numAdaptivePresets` | 32 | Number of adaptive bot loadouts kept in the database. |

The battle pass comment, translated:

> Activating the BattlePass event
> - `battlePassStart`: string, start of the event, format YYYY-MM-DDThh:mm:ssZ (ISO 8601).
> - `battlePassEndTable`: table of the active stages, `{ {stage, stageEndTime}, {stage, stageEndTime}, ... }`, where `stage` is the stage number (counting from 1) and `stageEndTime` its end time in the same format. `stageEndTime` may be an empty string; then the stage has no end and stays active until it is removed from the list.
> - Note: player progress is stored for one event only. Changing the start time loses it, because it counts as a different event. This doesn't apply to premium bought through DLC, which is kept for every event.
> - (On the table:) Make sure the end dates/times of stages 1 and 2 of one battle pass are identical!!! A 3rd stage was added (from summer 2023).

The template manages all of this: see [BATTLE_PASS.md](../BATTLE_PASS.md).

## Shard

`ConfigureShard()`.

### Accounts

| Cvar | Value | Meaning |
|---|---|---|
| `auth_allowInternalAutoCreate` | 1 | *bool.* Create an internal account automatically on first login. |
| `sm_sandboxAccess` | 4 | Access level to sandbox functions. |
| `auth_infinitePremiumForRoles` | `gm;support` | **AMP** (Infinite Premium For All Players). Account roles (`admRole`), separated by `;`, that have unlimited premium. |
| `gold_on_start` | 300 | **AMP** (Starting Galactic Standards). GS for a new account. |
| `premium_on_start` | 0 | **AMP** (Starting Premium Days). Premium for a new account, in **hours**. |
| `credits_on_start` | 0 | **AMP** (Starting Credits). Credits for a new account. |
| `premiumTimeForAccountRank` | 86400 | **AMP** (Premium Hours Per Account Rank). Premium per account rank-up, in seconds (default: one day). |
| `sv_tokenCreditsDailyLimit` | 1000 | **AMP** (Daily Token Credit Limit). Iridium a player can earn per day. |
| `sm_initialKeepAliveTime` | 120 | Seconds to wait for the client's first keep-alive after connecting; without it the client is disconnected. |
| `ctm_taskRecoveryInterval` | 5 | How often (seconds, 0.01–5) Shard checks the database for pending admin tasks. Lower runs them sooner but polls more. The template's account commands (`give`, `player`) are such tasks. |

### Shop and inventory

| Cvar | Value | Meaning |
|---|---|---|
| `store_allowAllItemsForBying` | 0 | **AMP** (Buy Everything). *bool.* Allow buying items that have the "can't be bought" flag. |
| `store_vesselPriceMod` | 1 | **AMP**. Ship price multiplier, for all currencies (credits, GS, iridium, event). Range 0.001–100. |
| `store_modulePriceMod` | 1 | **AMP**. Same for modules. |
| `store_munitionPriceMod` | 1 | **AMP**. Same for ammunition. |
| `store_prestigePerVssl` | 0.01 | Prestige per normal ship. Account prestige = normal ships × this + premium ships × `store_prestigePerPremVssl`. |
| `store_prestigePerPremVssl` | 0.01 | Prestige per premium ship. The higher, the more premium ships raise prestige. |
| `store_vesselExpExtractionCost` | `store_vesselExpExtractionCost` (0.0067) | Cost of extracting ship XP. |
| `store_vesselExpExtractionCostVisualBonus` | 0 | Display flag for the XP extraction price: 0 if the cost is 0.006 or more, otherwise 1 (shown as a discount). |
| `store_bonusBundleChancePercentageMax` | 0.2 | Highest chance (0–1) of a bonus bundle when buying a bundle. The chance is this × (prestige / max prestige). |
| `store_maxUniqueModules` | 3 | Maximum identical unique modules on one ship ("balance rule"). |
| `store_maxUniqueModulesOnPremium` | 2 | Same for premium ships. Comment: "#0098493: allow fitting experimental modules on premium and DLC ships, 1 each". |
| `store_maxTransactionSum` | 20000 | Largest amount in one transaction or mail (set twice, same value). |
| `shard_minAntibotCounterForMailSending` | 50 | Minimum anti-bot counter (0–100) a player needs to send mail (set twice, same value). |
| `store_creditsEventStart`, `store_creditsEventEnd` | from the top | **AMP**. The event-currency period. |
| `sv_offerEnable` | `6,7,17,18,19` | The `offerId`s from `offers.xml` that are active. Numbers separated by commas, spaces or tabs; an empty string turns all offers off. |
| `shipQuests_dbgLog` | 0 | *bool.* Log ship quests. |

### Squads and wings

| Cvar | Value | Meaning |
|---|---|---|
| `squad_maxNonPremiumSize` | 12 | Squad size that can queue without premium. |
| `squad_maxNonWingSize` | 4 | Largest squad that isn't a wing. Inviting more players makes it a wing. |
| `squad_maxWingSize` | 12 | Largest wing. No one can be added after that. |
| `mm_sectorMinWingSize` | 5 | Smallest wing allowed into sector battle queues. |
| `mm_sectorMaxWingSize` | 8 | Largest wing allowed into sector battle queues. |
| `mm_raidMinWingSize` | 10 | Smallest wing allowed into raid queues. |
| `mm_raidSquadFilter` | 0 | Who may queue for raids: 0 everyone (solo, squads, wings); 1 solo only; 2 solo and squads, no wings; 3 squads and wings, no solo; 4 wings only. Wing modes also need `mm_raidMinWingSize`. |
| `mm_enablePveRaids` | 1 | Enable PvE raids in the matchmaker. |
| `mm_notArcadeSquadRankDiff` | 17 | Allowed rank difference within a squad for non-arcade modes. |
| `raid_lootAuraInterval` | 1 | Days between grants of the raid loot auras. Checked at the player's first login of the day: 1 = daily, 3 = every three days. |

### Matchmaking value (MMV)

MMV estimates a player's skill so the matchmaker can build fair teams. These settings sit in the Shard section because Shard computes it.

| Cvar | Value | Meaning |
|---|---|---|
| `mm_squad1AsSquad` | 0 | Treat a solo player as a squad of one when matching. |
| `mm_totalMMVMdCap` | 2 | Cap on the total MMV adjustment from efficiency, kills per minute, wins and other factors. |
| `mm_effRatingMMVMdBase`, `mm_effRatingMMVMdStep`, `mm_effRatingMMVMdCap` | 0, 25, 50 | MMV from the efficiency rating: Base = threshold, Step = step, Cap = maximum. |
| `mm_kpmMMVBase`, `mm_kpmMMVStep`, `mm_kpmMMVCap` | 0, 0.0325, 50 | MMV from kills per minute, same scheme. |
| `mm_winLoseMMVMinBattles` | 2000 | The win/loss adjustment only starts after this many battles. |
| `mm_winLoseMMVBase`, `mm_winLoseMMVStep`, `mm_winLoseMMVCap` | 0.75, 0.012, 50 | MMV from the win/loss ratio: `clamp((winLose − Base) / Step, 0, Cap)`. |
| `mm_mmvMdBattles` | `200,…,100,0,…` | Rank list: number of battles per rank after which the floor `mm_mmvMin` applies. |
| `mm_mmvMdPenalty` | `30,…` | Rank list of MMV penalty values. |
| `mm_mmvMin` | `30,…` | Rank list: lowest MMV once the battle threshold is reached. Penalties can't push MMV below it. Must have as many values as `mm_mmvMdBattles`. |
| `mm_mmvMdUniqFingerprintClamp` | `30×7, 80×11` | Rank list: highest MMV for a player with a new PC fingerprint, until they reach the battle threshold. Limits how high a newcomer can be rated. |
| `mm_decisionLog` | 0 | Detailed log of why a player or squad was or wasn't queued: 0 off, any higher number on. |

### Brawl schedule

`brawlSchedule`, passed to `BrawlManager_SetupSchedule`. Ticket SC-112207. The comments, translated:

- The schedule covers 28 days, counted from the first Monday of 2018.
- Brawl game modes: `football`, `gtdm`, `hunger_games`, `one_against_all`, `sentinel`, `capture_the_base`, `atdm`, `zombie`, `close_distance`, `autogen_dm`, `three_teams_dm`, `april_control`.
- Intervals don't have to be sorted.
- A day's last interval may run past midnight, such as `{23,1}`.
- Intervals within one day must not overlap, and days must not repeat.

Active entry: every day 1–28, `three_teams_dm` from 1–2, 9–10, 15–16, 17–18, 19–20 and 21–22 o'clock. A commented-out rotation of football, one against all, GTDM, hunger games, sentinel and capture the base is still in the file.

```lua
-- one entry: these days of the 28-day cycle, these hours (from, to), this mode
{ days = { 1, 8, 15, 22 }, time_intervals = { {2,3}, {10,11}, {14,15}, {17,18}, {21,22} }, gameplay = "football" },
```

### Leagues, battle pass, bots

| Line | Value | Meaning |
|---|---|---|
| `Shard_SetupLeagueForbiddenEquipment(leagueForbiddenDefs)` | | Applies the league ban list from the top of the file. |
| `Shard_SetupBattlePassActivation(battlePassStart, battlePassEndTable)` | | **AMP**. Activates the battle pass. |
| `store_battlePassUnlockLvlBtnStart` | `2023-07-14T00:00:00Z` | **AMP**. From this date on, the "unlock battle pass level" button appears. |
| `sm_bflRoleInc` | 0.5 | Adaptive bot loadouts: how fast a loadout's rating rises (BFL = battle time for life time; SC-115316). |
| `sm_bflRoleDec` | 0.05 | How fast it falls. |
| `sm_numAdaptivePresetsToSaveDB` | 32 | Adaptive loadouts saved in the database (0–1000; 0 clears them). |

## Matchmaker

`ConfigureMatchmaker()`. The file starts with a list of all arena maps (as comments) and the bot loadout types: `bpt_none` 0, `bpt_mirror` 1, `bpt_adaptive` 2, `bpt_mirror_and_save` 3.

### Queues

The matchmaker has two sets of queues: `queuesConfig` for normal population and `queuesConfigLo` for low population. It switches by players online:

| Cvar | Value | Meaning |
|---|---|---|
| `mm_lowOnlineLo` | 250 | Switch to the low-population queues when fewer players than this are online. |
| `mm_lowOnlineHi` | 300 | Switch back at this many players or more. |

Unless 250 or more players are online, **only `queuesConfigLo` is used**.

Each queue is a Lua table:

| Field | Meaning |
|---|---|
| `maxPlayers`, `optimalPlayers`, `minPlayers` | Match size: the most players, the size it aims for, the fewest that start a match (bots fill the rest where the mode has bots). |
| `maxExpandTime` | Seconds over which the search widens. |
| `regionLocalizedTimeInQueue`, `maxTimeInQueue`, `maxSquadTimeInQueue` | How long to look within the region, and the longest wait (solo, squad). Sector queues override the region time with 1 ("default value is 120"). |
| `levelsDesc` | Maps and the game modes on each: `s1338_devilsjaw = {4,9,11}` means Devil's Jaw with modes 4, 9 and 11 (numbers in [Game modes](#game-modes)). |
| `query` | Which players the queue takes: the queue type (`mmParams.arcade` and so on), ship rank (`mmRank`), squad rank, and the MMV range (`mmValue`). |
| `special` | Mode options: `arcade`, `coop_vs_ai`, `pve`, `brawl`, `botsCount`, `gameMode`, `queueFlags` (for example `newbie = 1`). The template adds `botsCount` to the arena queues and `botsCount`/`botsInTeam1` to the coop queues. |
| `qid`, `squadFilter` | Fixed queue ID, squad filter (see `mm_raidSquadFilter`). |

`InstantiateQueueTemplate(template, minRank, maxRank, minSquadRank, maxSquadRank, minMMV, maxMMV, addMods)` copies a template with a different rank and MMV range, and merges `addMods` into it.

Low-population queues:

| Queue | Players | Maps and modes | Notes |
|---|---|---|---|
| Training | 1–10 | Dreadnought wreck, Anomaly, Northern mining site | practice, no bots |
| PvE | 1 | | `gameMode = 7` |
| Arena (6 MMV brackets) | 1–24 | Devil's Jaw, Northern mining site, Asteroid city, Barrier debris, Dreadnought wreck; modes 4, 9, 11 | Newbie bracket (MMV 0–20) limited to 16 players. MMV brackets 0–20, 19–24, 24–31, 31–39, 39–51, 51–99 |
| Coop vs AI (6 brackets) | 1–12 | same maps | Newbie bracket limited to 8 |
| Football | 2–6 | `football`, mode 21 | brawl |
| GTDM | 2 | `da_prototype`, mode 13 | brawl |
| ATDM | 2–8 | `paper_airplanes`, mode 26 | brawl (two queues) |
| Sentinel | 4–24 | Asteroid city, mode 9 | brawl |
| Capture the base | 4–24 | Devil's Jaw, mode 3 | brawl |
| Hunger games | 2–24 | `s1338_hg_pandora_anomaly`, mode 24 | |
| One against all | 2–24 | same map, mode 25 | 7 bots |
| Zombie | 5–24 | `da_deathstar`, mode 27 | |
| Close distance | 2–24 | `allidium_yard`, mode 28 | |
| Autogen DM | 2–24 | Anomaly, mode 29 | |
| Three teams DM | 3–24 | `s1340_thar_alienruins`, mode 30 | the active brawl |
| April control | 2–8 | `paper_airplanes`, mode 31 | |
| Sector battle (qid 501) | up to 20 | six dreadnought battle maps, mode 14 | wings of 5+, bots from `BotsBZS_T3`–`T5` |
| League (qid 502, #0108276) | up to 12 | ten arena maps, mode 3 | wings of 3+, no bots |
| PvE raid (#0046436) | 1–12 | | `pve_raid`; who may join is set by `mm_raidSquadFilter` |

`queuesConfig` (normal population) has the same queues with nine narrower MMV brackets, and only Devil's Jaw in the arena queues. Sector, league and raid queues are in both sets.

### Game modes

`mm_defaultGameplay` = 3: the default mode for a new match, which a queue can override. The values, as listed in the file ("not all of them work"):

| ID | Mode | ID | Mode |
|---:|---|---:|---|
| 0 | none | 16 | last man standing |
| 1 | space battle | 17 | transport escort |
| 2 | capture the flag | 18 | open space |
| 3 | capture the base | 19 | mission script |
| 4 | control | 20 | PvP mission |
| 5 | practice | 21 | football |
| 6 | tutorial | 22 | battle royale |
| 7 | PvE mission | 23 | capture the base 2 |
| 8 | free space | 24 | hunger games |
| 9 | sentinel | 25 | one against all |
| 10 | king of the hill | 26 | ATDM |
| 11 | bomb the base | 27 | zombie |
| 12 | TDM | 28 | close distance |
| 13 | GTDM | 29 | autogen DM |
| 14 | clanship (dreadnought battle) | 30 | three teams DM |
| 15 | capture the frag | 31 | april control |

### Bots, rates and logging

| Cvar | Value | Meaning |
|---|---|---|
| `mm_botsCount` | 6 | **AMP** (Ships In Arena Matches). Base number of bots (0–31); a queue can override it. In arena matches it is the total ship count, bots filling up to it. |
| `mm_timeInQueueAveragingCf` | 200 | Smoothing of the average queue time (2–10000): the higher, the slower it changes. "10 for dev/test, >100 for live". |
| `rate_exp` | `1,…` | **AMP** (XP Multiplier). Rank list of XP multipliers. |
| `rate_cred` | `1,…` | **AMP** (Credit Multiplier). Rank list of credit multipliers. |
| `mm_decisionLog` | 0 | Bit mask for the matchmaker's decision log. Add the numbers to combine: 1 basic PvP, 2 PvP join-in-progress, 4 PvP candidates, 8 coop, 16 coop join-in-progress, 32 PvE, 64 universe, 128 brawl, 256 brawl candidates, 512 hunger games, 1024 one against all, 2048 zombie, 4096 three teams, 8192 close distance, 16384 general. Example: 33 = basic PvP + PvE. All: 32767. |
| `mm_decisionDbg` | 0 | How the decision log is written: 0 collect and write as one block to the info log after each pass; 1 write every message to the debug log immediately (good for live debugging, but many lines). |

### Reward times

`reward_mult`, passed to `MatchMaker_SetRewardRates`: extra multipliers or bonus GS by weekday and hour. Days count from 0 = Monday.

```lua
[0] = {                                                    -- Monday
  { hourBegin = 0, hourEnd = 1, rateExp = "1,…", rateCred = "1,…" },
  { hourBegin = 0, hourEnd = 3, rateExp = "1,…", rateCred = "1,…" },
  { hourBegin = 2, hourEnd = 5, goldReward = 50 },         -- 50 GS per battle 02–05
},
[6] = { { hourBegin = 10, hourEnd = 13, goldReward = 100 } }, -- Sunday 10–13
```

The multipliers are all 1, so only the two bonus GS windows have an effect.

### PvE mission rotation

`pveSchedule`, passed to `MatchMaker_SetPveSchedule`: the PvE missions players can choose.

| Field | Value |
|---|---|
| `rotatingLevels` | `pve_frozen_station`, `s1550_pve_asteroid_building`, `s1550_pve_bigship_building`, `s1550_pve_bigship_building_2`, `s1550_pve_capture_repairbase`, `pve_desttown`, `pve_stealth` (others commented out) |
| `maxRotatingLevels` | 12 offered at once |
| `rotationTime` | 7200 seconds (2 hours) |
| `permanentLevels` | `pve_transit_emp_rock_frontier`, always available |

## Scripts server

`ConfigureScriptsServer()`.

| Line | Value | Meaning |
|---|---|---|
| `sv_enableCommunticationWithSS` | 1 | *bool.* Zone instances exchange data with the scripts server. |
| `dbg_disableLuaDataFormatChecking` | (commented out) | Skip checking Lua data (such as a zone's `playerLuaData`) against its schema when saving to the database. Only works on non-live builds. `cloud.xml` sets it to 1 for every service anyway. |
| `ScriptsServer:ConfigureGlobalEvents({...})` | empty | Forces open-space global events on start ("pass the scripts server default settings, for example 'the green war event at hard difficulty is running'"). All variants are commented out. |
| `ScriptsServer:SetTimerScale(1)` | 1 | Speed of event timers. A commented-out line uses 3600 ("1 second = 1 hour; not every parameter supports speeding up"). |

The commented-out event blocks describe the alien ("green") invasion. Translated, for anyone who wants to turn it on:

| Field | Meaning |
|---|---|
| `eventDefName` | Event type. Created if it doesn't exist, otherwise the given values are changed. Green (alien) events: 7 pre-event, 2 ownership, 1 invasion, 6 timer before the invasion (`GREEN_PRE_EXPANSION`), 8 counter-attack control. Empire: 9 preparing an invasion, 12 invading. |
| `eventLocation` | Zone ID (see `zones.xml`). Aliens' capitals: 213 Abandoned complex (behind Jericho), 11 Destroyed laboratory (Empire–Federation), 224 Northern mining station (Federation–Jericho). Also mentioned: 25 the Empire–Federation PvP zone, 231 the start zone, 104 Wasteland (Empire–Federation, next to 11). |
| `newEventState` | New state or timer: 1000, 500, `60*60*3` (3 hours), or −100501 for `TIMEOUT_EVENT`. |
| `disableCreate` | Don't create the event if it doesn't exist. |
| `removeEvent` | Remove the event if it exists. |
| `eventParams` | Parameters to overwrite. |

The variants in the file: "pre-event", "at launch", "at launch with counter-attacks (not ready for November)" and "my debug". A single-line example is also commented out: `ScriptsServer:ConfigureGlobalEvents({ forceEvents = { { eventDefName = 4, eventLocation = 247, newEventState = 998 } } })`.

## Battle servers

`ConfigureDediServer()`: arena, coop, PvE and brawl matches.

### General

| Cvar | Value | Meaning |
|---|---|---|
| `cl_maxFps` | 18 | Frame rate limit of the battle server (0 = unlimited). Overrides the 32 in `dediserver.cfg`. |
| `sv_dbgJoinInProgress` | 0 | *bool.* Debug joining a running match. |
| `cl_combatLog` | 0 | Combat events in the log; each level includes the previous: 0 off, 1 kills, deaths, rewards; 2 + modules, missiles, auras; 3 + all damage; 4 + gameplay aura coefficients. |
| `db_logMessages` | 0 | Log the database protocol: 1 short, 2 full ("to catch some nasty stuff"). |
| `sv_respawnTimePenalty` | −1 | Overrides the respawn delay penalty; −1 uses the game's own values. |
| `sv_afkCheckDuration` | 120 | **AMP** (AFK Kick Time). Seconds without (changing) input before an AFK player is kicked. A large value such as 3600 effectively disables it. |
| `ship_controlForceMode` | −1 | Force a ship control mode (−1 to 5): −1 don't force, 0 following, 1 combined, 2 local turret, 3 remote turret, 4 landed, 5 WOW. |
| `ship_ufoRotateX` | 1 | Special X-axis rotation for UFO ships ("UFO ship controls"). |
| `ship_aiThink` | 1 | *bool.* Ship AI on (debug switch). |
| `script_gcWorkBudget` | 1 | Lua garbage-collection time budget per step, in microseconds. |
| `sv_autorecordReplaysTo` | `""` | Folder to save match replays in; empty = off. |
| `event_custom` | from the top | Start of the custom event (whole hours only, see the top). |
| `sv_kickExcessiveBotsEnabled` | 1 | **AMP** (Kick Excessive Bots). Remove bots from a match once there are more bots than players. |

### Loot

| Cvar | Value | Meaning |
|---|---|---|
| `sv_lootEnabled` | 1 | *bool.* Loot after battle. |
| `sv_lootFailChance` | 0 | Chance (0–1) to find nothing. |
| `sv_lootScanTime` | 1 | Scan time, seconds. |
| `sv_lootDuration` | 120 | Length of the loot phase. |
| `sv_salvageTokenConversion` | 0.2 | Salvaging a found item into iridium: refund = item's `tokenPrice` × this. |
| `sv_salvagePremiumTokenConversion` | 0.6 | Paid (GS) salvage: refund = `tokenPrice` × this. |
| `sv_salvagePremiumPriceConversion` | 0.0008 | Price of paid salvage: item price × this. |
| `sv_rarePicksNumberWithFloatingChances` | 100 | Attempts at the rare table with floating chances ("rare_limit"). |
| `sv_epicPicksNumberWithFloatingChances` | 150 | Same for the epic table ("epic_limit"). |
| `sv_overlootedRarePickChance` | 0.004 | Base chance of picking the rare table. |
| `sv_overlootedEpicPickChance` | 0 | Base chance of picking the epic table. |
| `sv_lootCommonPickChance` | 0.2 | Base chance of picking the common table ("common_default_weight"). |
| `sv_lootMaxLevelRestriction` | 0 | *bool.* Rare and epic tables only for ships at maximum level. |
| `sv_lootRuleCfA`, `sv_lootRuleCfB`, `sv_lootRuleCfOffset` | 0.25, 0.25, −1 | Weight of the rare/epic table: `weight = max(0, A × (attempts − (it + Offset)) + B)`. `attempts` = tries without a rare/epic drop; `it` = the threshold from `iter` in the loot rule for the current drop number. |

`DedicatedServcerServices_SetLootRules` (the function name has the developers' typo) sets per ship rank which loot table is used, and from how many attempts on:

```lua
[8] = { common = { lt = "CommonRanks_7_8", iter = { 0 } },
        rare   = { lt = "RareT3Mono",      iter = { 10,15,14,15,15,14,15,15,26,40,50 } },
        epic   = { lt = "NeodiumDrop",     iter = { 5,7,10,12,18,24,36,36,36,36 } } },
```

`lt` is a loot table from the game data, `iter` the thresholds for the 1st, 2nd, … drop. An `iter` of 100000 means never. Ranks 8 and 11 drop the crafting materials (`NeodiumDrop`, `LezortiumDrop`) as their epic table.

### Rewards and events

| Cvar | Value | Meaning |
|---|---|---|
| `sv_universeWingVsWingRewardCf` | 2.0 | Reward multiplier for wing-against-wing sector battles (1–10). |
| `sv_achievementEnabled` | 1 | *bool.* Achievements on. |
| `sv_achievProgressTimer` | 3 | Seconds between progress updates to the achievement server. |
| `sv_freeSynergyRewardCf` | 0.05 | Free synergy reward multiplier. |
| `sv_freeSynergyRewardCfPremiumVessel` | 0.2 | Same for premium ships. |
| `sv_goldMunitionExpRewardCf` | 1.0 | Extra synergy XP for kills or assists with premium (gold) ammunition. |
| `sv_dailyWinPveExpRewardCf`, `sv_dailyWinPvpExpRewardCf`, `sv_dailyWinCoopExpRewardCf`, `sv_dailyWinBrawlExpRewardCf` | `x3fw` (3) | **AMP**. XP multiplier for the first PvE / PvP / coop / brawl win of the day. |
| `sv_pvpLabelChances` | `0,…` | Rank list (values 0–1): chance that a ship is marked as an event agent in PvP. |
| `sv_pveLabelChances` | `0,…` | Same for PvE missions and raids. |
| `event_cosmonauticsDay`, `event_gameAnniversary`, `event_february23`, `event_march8`, `event_botPrefix`, `event_claws`, `event_knowledgeDay`, `event_halloween`, `event_invasionPVP`, `event_valentinesDay` | 0 | Holiday events in battles: Cosmonautics Day (12 April), game anniversary, 23 February, 8 March, the bot-name-prefix event, Claws (1 April), Knowledge Day (1 September), Halloween, PvP invasion, Valentine's Day. |

### Bots and leavers

| Cvar | Value | Meaning |
|---|---|---|
| `DedicatedServerServices_SetupBotPresetForbidden(botPresetForbiddenDefs)` | | Applies the bot ban list. (The league ban list call is commented out: "uncomment and fill in if needed".) |
| `sv_numBestPlayerToSave` | 2 | How many of the best players' ships are saved as adaptive bot loadouts; −1 = every player and ship (SC-115316). |
| `sv_minEfficiencyVesselToSave` | 1500 | Minimum efficiency score for a ship to be saved. |
| `sv_numAdaptivePresetsInDB` | 32 | Saved loadouts per mode, rank and role; 0 clears them. |
| `sv_numUniqueAdaptivePresets` | 20 | Unique loadouts (0–100) the battle server requests from the database. |
| `sv_rangeRankAdaptivePresets` | 3 | Rank range for loadouts, down from the queue's highest rank: with max rank 12 and 3, ranks 9–12. |
| `sv_leaveFineRewardCoef` | 0.8 | Reward multiplier (0–1) for players who left earlier matches. Game default 1 (no penalty). |
| `sv_leaveCounterMaxDecay` | 4 | The most the leave counter can drop when a player gets a penalty (counter above 1). Default 3. |
| `sv_leaveCounterDecayWithoutFine` | 0.9 | How much the counter drops when the player gets no penalty (counter below 1). Default 0.1. |
| `sv_normalLeaveCounterAdd` | 0 | Added to the counter for a normal leave (the "Leave battle" button). Default 0.5. |
| `sv_abnormalLeaveCounterAdd` | 0 | Added for an abnormal leave (crash, lost connection). Default 0. |
| `sv_leaverReputationAdd` | 1 | Added to leaver reputation (0–100) when a player is banned for leaving. |
| `sv_leaverBanTime` | 0 | Battle ban for leavers, in seconds (0–1800). Default 300. |
| `sv_leaverReputationDecay` | 0.9 | How much leaver reputation drops when the player doesn't leave. Default 0.01. |

The leave counter's maximum is `reward_leaveCounterMax` in the [reward dispenser](#reward-dispenser).

## Queue flags

`ConfigureDediServerFlags(queueFlags)` runs per match with the queue's `queueFlags`:

| Flag | Effect |
|---|---|
| `newbie` | `sv_respawnTimePenalty = 0`: no respawn penalty in newbie queues. |
| `rank` | `sv_mmBattleRank = rank`: the match's battle rank, which the rank limits depend on. Sector and league queues set 15. |

## Open space (zone instances)

`ConfigureZoneInstance()`: the servers for open-space zones. The file says: "restart cloud after changing these cvars".

### General

| Cvar | Value | Meaning |
|---|---|---|
| `cl_maxFps` | 18 | Frame rate limit. |
| `ship_aiThink` | 1 | Ship AI on. |
| `cl_combatLog` | 0 | Combat log level (as for battle servers). |
| `sv_shipWeaponDamageCoef` | 1 | Weapon damage multiplier. |
| `sv_instanceLifeTime` | 720 | Seconds an empty zone instance stays open. |
| `sv_afkCheckDuration` | 3600 | AFK kick in open space: one hour. Not changed by the template. |
| `sv_spaceStationPortalTimer` | 3 | Portal timer to a station. |
| `sv_enableCommunticationWithSS` | 1 | Exchange data with the scripts server. |
| `sv_autorecordReplaysTo` | `""` | Replay folder. |
| `ship_ufoRotateX` | 1 | UFO ship controls. |
| `sv_autogenModulesEnabled` | 0 | Seed chips (randomly generated modules, SC-112454). |
| `sv_dbgAutogenDefs` | 0 | Where seed chips are shown: 1 in game only, 2 in game and hangar. |
| `ai_logSquads` | 5 | AI squad log: 1 spawn, 2 movement, 3 combat, 5 everything. |
| `ai_convoyDefendRadius` | 300 | Distance between the player and the despawn point. |
| `ai_kiteDistance` | 5000 | How far an AI squad may move from its start point while chasing an enemy. |
| `sv_lootCrateScanTime` | 1 | Time to scan a loot crate. |

### Events

| Cvar | Value | Meaning |
|---|---|---|
| `sv_eventObjectRespawnTimeMin`, `sv_eventObjectRespawnTimeMax` | 60, 420 | Range for the random delay before an event object appears, the first and each later time. |
| `sv_eventObjectEndTime` | `2026-04-10T05:00:00Z` | After this, event objects no longer spawn. |
| `event_claws` | 0 | The 1 April event. |
| `event_invasionWithBeacons` | 0 | The "invasion with beacons" event (#57052). |
| `sv_owLabelChances` | `0,…` | Rank list: chance of marking ships with labels in open space. |

### Rewards and XP

| Cvar | Value | Meaning |
|---|---|---|
| `sv_resurrectionCreditsPrice` | 100000 | **AMP** (Open Space Resurrection Cost). Credits to respawn in place. |
| `sv_pickedLootRateToClanResource` | 0.1 | Conversion of loot pickups into corporation iridium. |
| `sv_pickedLootUpdateTime` | 21600 | How often (seconds, 6 h) the iridium is sent to the corporation. |
| `sv_dailyWinPveExpRewardCf`, `sv_dailyWinPvpExpRewardCf`, `sv_dailyWinCoopExpRewardCf`, `sv_dailyWinBrawlExpRewardCf` | `x3fw` | **AMP**. First win of the day multipliers. |
| `sv_freeSynergyRewardCf`, `sv_freeSynergyRewardCfPremiumVessel` | 0.05, 0.20 | Free synergy rewards in open space. |
| `sv_freeSpaceExpMastery` | `1.95,…,4.1` | Rank list: ship mastery used for open-space XP. |
| `sv_expRate` | `1,…` | **AMP** (XP Multiplier). Rank list of XP multipliers in open space. |
| `sv_freeSpaceExpRewards` | 2 | Ship XP for fighting in open space: 0 nobody, 1 premium players only, 2 everyone. Also limited by the zone's rank and `sv_freeSpaceExpCap`. |
| `sv_expForDamageFreeSpace` | 0 | *bool.* XP for damage in open space ("fast synergy gain"). |
| `sv_randomLootDef` | `LootCrate_Craft_paradise_Weapon20` | Loot crate used by random open-space loot. Must be a `LootCrate` with `isRare`, otherwise random loot is off and the log shows an error. |

`DedicatedServerServices_SetIterativeLoot` assigns one loot crate per open-space map, from `LootCrate_Loot_4_ranks_2_3` (`wreck_generic_rock_03_r_ow`) up to `LootCrate_Loot_460_ranks_15_17` (`mininghub_jer_rock_frontier_e_ow`). The crate name gives the ship ranks.

### Rank scaling

| Cvar | Value | Meaning |
|---|---|---|
| `sv_ranksDiff` | `-5, -2, 0, 2, 5, 10` | Bands of player rank minus enemy rank. |
| `sv_ranksDiffCoeff` | `-0.5, 0.2, 0.5, 0.6, 0.7, 0.8, 1` | Damage scaling for each band. |
| `dbg_logRankDiffScaling` | 0 | Log the scaling. |

### Aggression and factions

| Cvar | Value | Meaning |
|---|---|---|
| `sv_aggressorMinNPCDamage` | 3000 | Damage to a non-player target after which a player becomes an aggressor. |
| `sv_aggressorTimer` | 30 | Maximum aggressor timer. |
| `sv_frendlyFactionDamageCoef` | 0.25 | Damage multiplier (0–1) against friendly faction NPCs. Only this one affects damage; the others only affect relations (SC-113076, pirate event). |
| `sv_enemyMinFactionNPCDamage` | 6 | Damage counter value at which a faction turns hostile. |
| `sv_factionBadRelationshipTime` | 45 | Seconds the faction stays hostile. |
| `sv_enemyMaxFactionNPCDamage` | 100 | Cap of the damage counter. |
| `sv_enemyMaxFactionNPCDamagePerSec` | 1 | Cap of the counter's growth per second. |
| `sv_factionNPCDamageDecay` | 0.2 | Counter decrease per second. |

## Reward dispenser

`ConfigureGRD()`: GameRewardDispencer.

| Cvar | Value | Meaning |
|---|---|---|
| `reward_bonusGoldForWinner` | 0 | **AMP** (Bonus GS Per Win). GS added for the winners of a match. |
| `reward_bonusGoldForLooser` | 0 | **AMP** (Bonus GS Per Loss). GS for the losers. |
| `reward_eventGoldTimePeriod` | `2025-09-02T12:00:00Z;2025-09-11T06:00:00Z` | Period of the "GS for the first and fifth win" event, start and end separated by `;`. |
| `reward_eventGoldTimeValue` | `0;0;0;0;50;…;200` | GS for the 1st, 2nd, … win during the event: 50 for the 5th, 100 for the 10th, 150 for the 15th, 200 for the 20th. |
| `cron_enabled` | 1 | Scheduler on. |
| `store_creditsEventStart`, `store_creditsEventEnd` | from the top | **AMP**. Event-currency period. |
| `sv_tokenCreditsDailyLimit` | 1000 | **AMP**. Daily iridium limit. |
| `reward_leaveCounterMax` | 12 | Maximum leave counter (SC-113769). |

## Load balancer

`ConfigureLoadBalancer()`.

| Cvar | Value | Meaning |
|---|---|---|
| `lb_disallowAccess` | 0 | 1 closes the server to logins. |
| `lb_limitPlayersOnShardTo` | 800 | **AMP** (Max Players). Players per shard. There is one shard, so this is the server's player limit. |
| `lb_avgTimeMinClamp` | 10 | The average login queue time shown never drops below this. |
| `lb_avgTimeAveragingCf` | 10 | Number of recent players the average queue time is computed from. |
| `lb_minNextPlayerDelay`, `lb_maxTaskQLenOnShard` | 0.05, 500 | Load limits: delay between logins, maximum task queue length on the shard. |
| `LoadBalancer_SetGreetingMessageCVars({...})` | | Settings sent to every client at login, so the client shows the right numbers: the four first-win multipliers (`x3fw`), the "player bought a ship" ad brackets and cooldown, `cl_showBiomorphsKilledCounter = 0`, `clan_shipPartsCanBuild = 4`, `clan_shipUpgradeCanBuild = 3`, `clan_shipBuildHighRanks = 3` (**AMP**: Dreadnought Members), `event_claws = 0`, `event_halloween = 0`. |

## Chat server

`ConfigureChatServer()`. The anti-spam system keeps an **awareness** score per player: it rises with every message, faster when messages come quickly or repeat, and drops over time.

| Cvar | Value | Meaning |
|---|---|---|
| `antispam_autobanSec` | 60 | Length of an automatic chat ban, seconds. |
| `antispam_autobanAwareness` | 500 | Awareness at which a player is banned automatically. |
| `antispam_notifyAwareness` | 450 | Awareness at which the player is warned to stop spamming. |
| `antispam_notifyExtended` | 0 | Send an extended warning. |
| `antispam_awarenessDecay` | 10 | Awareness lost per second. |
| `antispam_awarenessIntervalNumerator` | 40 | Increase by message frequency: `A / dt`, `dt` = seconds since the last message (at least 0.5). Higher A = faster rise. |
| `antispam_awarenessSimilarityMul` | 1 | Increase by repetition: `K × similarity ^ P`, added with each message. `similarity` = the share of the message that matches earlier messages (total length of matching substrings longer than 2 characters, divided by the message length). This is K. |
| `antispam_awarenessSimilarityPow` | 1 | P in the same formula. |
| `antispam_trade_autobanSec` | 60 | Trade chat: ban length. |
| `antispam_trade_autobanAwareness` | 200 | Trade chat: ban threshold. |
| `antispam_trade_notifyAwareness` | 150 | Trade chat: warning threshold. |
| `antispam_trade_notifyExtended` | 0 | Trade chat: extended warning. |
| `antispam_trade_awarenessDecay` | 10 | Trade chat: decay per second. |
| `antispam_trade_awarenessIntervalNumerator` | 100 | Trade chat: frequency factor A. |
| `antispam_trade_awarenessSimilarityMul`, `antispam_trade_awarenessSimilarityPow` | 1, 1 | Trade chat: repetition K and P. |

## Universe (sector conquest and corporation wars)

`ConfigureUniverse()`.

| Cvar | Value | Meaning |
|---|---|---|
| `universe_unid` | 2 | ID of the universe in use (1–65535). |
| `universe_newYearStartTime` | `2018-12-17T00:00:00Z` | Start of the New Year event ("NY2016 params"). |
| `universe_empireKillsCF`, `universe_federationKillsCF`, `universe_jerichoKillsCF` | 0 | Growth rate of each faction's kill counter. |
| `universe_factorOwnerInReward` | 0.4 | Share (0–1) of a zone's reward that goes to its owner; the rest is split among the rivals. |
| `universe_decayOwner` | 0.25 | Share of the owner's influence lost when a conflict starts. |
| `universe_decayRival` | 0.1 | Share of the rivals' influence lost when a conflict starts. |
| `universe_takeOwnerCoeffA`, `universe_takeOwnerCoeffB` | 2, 3 | Owner penalty after a conflict: each rival contributes `influence / (A × place + B)`, places counted from 0, so 1/3, 1/5, 1/7 … |
| `universe_decayIdle` | 0.5 | Share of influence lost after a conflict in which no new influence was gained (SC-111290). |
| `universe_allowExternalControl` | 1 | Allow an external service to control zone logic. With zone logic off, the server doesn't start or end conflicts itself and doesn't accept contributions (SC-112644). |
| `universe_raceFactorStellar` | 0.1 | Influence bonus for a corporation whose faction matches the star system's dominant faction: × (1 + value). |
| `universe_raceFactorZone` | 0.1 | Same for matching the zone's faction. |

Corporation war formulas: raw points = kill multiplier × target multiplier, then converted into military XP and corporation damage.

| Cvar | Value | Meaning |
|---|---|---|
| `war_coeffAssist` | 0.2 | Kill multiplier for assists; the killing blow gets 1. |
| `war_coeffRivalA`, `war_coeffRivalB` | 1, 2 | Target multiplier for an enemy: `A × victim's rank + B`. |
| `war_coeffNeutral` | 0.1 | Target multiplier when the victim's corporation isn't at war with the attacker's. |
| `war_coeffBot` | 0.05 | Target multiplier for a bot standing in for a player. |
| `war_damageRawRateToMilitaryExp` | 10 | Raw points → the player's military rank progress. |
| `war_damageRawRateToDamage` | 100 | Raw points → damage to the corporation on the galaxy map. |
| `war_decayCostBase` | 0.1 | Periodic reduction of the base war cost: × (1 − value). |

## Quest server

`ConfigureQuestServer()`.

| Cvar | Value | Meaning |
|---|---|---|
| `quests_enable` | 1 | *bool.* Quests on. |
| `event_newYear` | `2030-12-17T00:00:00Z` | Start of the New Year event (far in the future = off). |
| `quests_readWaveMaxUids` | 100 | Accounts loaded per database read (1–1024). |
| `quests_dbgLog` | 31 | Quest debug log, bit mask: 1 caching, 2 quest availability, 4 player state, 8 quest selection, 16 quest progress; 31 = everything. |
| `event_summer`, `event_summerShip`, `event_halloween` | 0 | Summer event, summer ship event, Halloween. |
| `event_custom` | from the top | Custom event start (whole hours only). |
| `quests_imaginaryRewardCoeffA` | 25 | Diminishing rewards for *imaginary* quests: `amount = floor((A − 1) × (1 − tanh(B × (completions − 1))) + 1)`, `completions` = how often the player has done the quest. A = the reward the first time; it falls towards 1 (SC-109926). |
| `quests_imaginaryRewardCoeffB` | 0.17 | B = how fast the reward falls. |
| `store_creditsEventStart`, `store_creditsEventEnd` | from the top | **AMP**. Event-currency period. |
| `QuestServer_SetupBattlePassActivation(...)` | | **AMP**. Battle pass dates for the quest server. |

## Achievement server

`ConfigureAchievementServer()`.

| Cvar | Value | Meaning |
|---|---|---|
| `achievements_enable` | 1 | Server-side achievements on. |
| `achievements_readWaveMaxUids` | 100 | Accounts per database read (1–256). |
| `achievements_dbgLogEnable` | 0 | Extra debug logging. |

## Lobby server

`ConfigureLobbyServer()`: custom games.

| Cvar | Value | Meaning |
|---|---|---|
| `lobby_enabled` | 1 | Custom games on. |
| `lobby_rewardRates` | 1 | Reward multiplier for custom games. |
| `lobby_reqPlayersPvEMin`, `lobby_reqPlayersPvEMax` | 3, 4 | Players needed for a PvE lobby. |
| `store_maxUniqueModules`, `store_maxUniqueModulesOnPremium` | 3, 2 | Unique module limits (#85547, #0098493), as on Shard. |
| `Lobby_SetupLeagueForbiddenEquipment(leagueForbiddenDefs)` | | League ban list. |

## Leaderboards

`ConfigureLeaderboardServer()` builds two lists: the leaderboards (`lbrds`) and the routes (`routes`) that feed game events into them.

Leaderboard fields: `name`; `entityType` (0 player, 1 corporation, 2 corporation member, 3 faction, 4 league team); `dir = 1` sorts ascending (lowest is best, for times); `decayInterval` (minutes) and `decayPower` (share lost each interval); `expiresAt` (Unix time in milliseconds), `renewalInterval` (minutes) and `expAction` (0 reset, 1 delete, 2 nothing); `rewards`.

Route fields: `event` (game event), `target` (leaderboard), `update`: 0 add (default), 1 set, 2 set if greater, 3 set if lower.

Rewards: `type = 0` pays currency (`tokenCredits`) or a title (`tid`), `type = 1` triggers a leaderboard event; `posMin`/`posMax` = places.

| Leaderboard | Fed by | Notes |
|---|---|---|
| **Players, permanent** | | |
| `player_prestige` | `prestige` (set) | Prestige |
| `player_pvp_effectiveness`, `player_pve_effectiveness` | `pvp_eff`, `pve_eff` (add) | Total efficiency, loses 10 % a day |
| `player_battles`, `player_wins`, `player_kills` | `gover`, `gwin`, `kills` | Battles, wins, kills |
| `player_karma_good`, `player_karma_bad` | `karma` (set) | Open-space karma, positive and negative |
| `player_goals` | `goals` | Football goals |
| `player_hunger` | `hunger_games_kills` | Hunger games kills |
| `player_pressure` | `pressure` | Influence points earned |
| `player_time_pve_raid…` (5 boards) | `time_pve_raid…` (set if lower) | Fastest raid times, per raid |
| `unlim_pve_level_planet_war_waves_T1` | (set if greater) | Endless PvE: highest level |
| `player_wavePve_wave` | (set if greater) | Wave PvE: highest wave |
| **Players, weekly with rewards** (renew every week from 28 Nov 2013) | | |
| `player_kills_weekly` | `kills_ht` | Places 1/2/3: 400/200/100 iridium |
| `player_pvp_eff_max_weekly`, `player_pve_eff_max_weekly` | `pvp_eff_ht`, `pve_eff` (set if greater) | Best PvP / PvE efficiency of the week; 400/200/100 iridium |
| `player_pve_win_weekly` | `pvegwin` | PvE wins of the week; 400/200/100 iridium |
| `player_eff_rating_weekly` | `eff_rating` (set) | Weekly rating; titles 31 (places 1–3), 32 (4–20), 33 (21–50) |
| **Corporation members** | | |
| `cm_pvp_effectiveness`, `cm_pve_effectiveness` | `pvp_eff`, `pve_eff` | Member's efficiency, loses 10 % a day |
| `cm_pressure`, `cm_military_rank`, `cm_war_damage`, `cm_war_losses` | | Influence, military rank, war damage, war losses |
| **Corporations** | | |
| `CLS_PVP_RATING`, `CLS_PVE_RATING` | `pvp_b`, `pve_eff` | PvP points and PvE efficiency, losing 1 % every hour |
| `CLS_WAR_DAMAGE`, `CLS_ZONE_CAPTURE` | `warDamage`, `zoneCapture` | War damage, zones captured |
| `CLS_WAR_RATING`, `CLS_WAR_RATING_MAX` | `warRating` | Current and highest war rating |
| **Events** | | |
| `event_green_war_empire`, `…federation`, `…jericho`, `…total` | `green_war_*` (set) | Alien war event scores |
| `player_april_win` | `atdmgwin` | April brawl wins |
| `player_cd_win` | `close_distance_kills` | Close distance kills |
| `player_kills_marathon` | `kills` | Kill marathon; expired 16 May 2022 ("Timestamp in milliseconds!!", converter: epochconverter.com) |

Commented out (not active): kills per battle, achievement points, kill rating, corporation influence, ELO, iridium, sectors, member count, league ratings, faction influence.

`leaderboard_logEvents` (commented out): log every incoming leaderboard event.

## Zone coordinator

`ConfigureZoneCoordinator()`.

| Cvar | Value | Meaning |
|---|---|---|
| `zc_enableZoneBlocks` | 0 | Block zones by the raid schedule: ZoneCoordinator passes the flag to new and running zone instances, the current raid zone is picked from the schedule, and jumping into it is blocked. |

## Tournament

`ConfigureTournament()`. The schedule itself is in `tournaments.xml`.

| Cvar | Value | Meaning |
|---|---|---|
| `tournament_idOffset` | 0 | Shift for team names: the client adds this to the name number and shows the string `single_tournament_name_N`. Changes the displayed names without changing team IDs. |
| `tournament_levels` | 9 maps | Tournament maps, separated by `;`. Comment: "`s1338_pandora_anomaly` is buggy" (left out). |
| `tournament_battleRank` | 17 | Battle rank of tournament matches, copied into `sv_mmBattleRank`. |
| `tournament_gamemode` | 12 | Game mode (12 = TDM). |
| `tournament_teamSize` | 8 | Largest team. |
| `tournament_gameInterval` | 2000 | Milliseconds between creating matches (50–10000). |

## Clan server

`ConfigureClanServer()`.

| Cvar | Value | Meaning |
|---|---|---|
| `clan_shipPartsCanBuild` | 4 | Dreadnought building stages available (0–4). |
| `clan_shipUpgradeCanBuild` | 3 | Dreadnought upgrade stages available (0–3: T3, T4, T5). |
| `clan_shipBuildHighRanks` | 3 | **AMP** (Dreadnought Members). Members of rank 9 needed to build a dreadnought. |
| `clans_historyMinResourceAddByZone` | 10 | Smallest iridium amount from a zone that is written to the corporation log ("don't spam the corporation log"). |
| `clan_warWinRewardAura` | `Bundle_ClanWar_Win` | Reward for winning a corporation war. |

## Adventure server

The file defines `ConfigureAdventureServer()` twice. The first sets `adventure_saveGiverOffers = 0` ("save adventure giver offers to the database: 1 saves, 0 doesn't"). The second, further down, is empty, and in Lua the later definition replaces the earlier one, so **`adventure_saveGiverOffers` is never set** and the game's built-in default applies. To set it, put it in the second function.

## Hub

`ConfigureHub()` sets `RegionWeightProportion`: how players from each region are weighted against the others, for example `eu = { ru = 0.2, eu = 0.7, us = 0.1 }`. Regions come from `regions.xml`. With one server it has no visible effect.

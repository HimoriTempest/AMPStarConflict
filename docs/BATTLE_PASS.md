# Battle pass

[← README](../README.md)

The battle pass is the game's xenochip event: players earn xenochips from battle pass tasks and unlock one reward
per level, stage by stage. The template can run any **event**, meaning a set of stages with their rewards, from a
start date until an end date. You pick it in the settings, and it takes effect on the next restart.

## Settings

All of them are under **Battle Pass** in the instance settings.

| Setting | What it does |
|---|---|
| Battle Pass | On: run the event. Off: the game's own `battle_pass.xml` and its original, expired 2023 dates are restored. |
| Battle Pass Event | The event's key, from `battle-pass/events.json` or `battle-pass/custom-events.json`. Default `precursors-2026`. |
| Battle Pass Stages | Active stages, e.g. `1` or `1,2,3`. Empty or `all` = every stage of the event. Default `1`. |
| Battle Pass Start | Start, in UTC. Player progress is stored per start time, so a new start date begins a new battle pass and existing progress is lost. |
| Battle Pass End | End of every active stage, in UTC. Empty = no end. |

On each start the console shows which event and stages are running and whether players need the client
file. A typo (unknown event, stage that doesn't exist, invalid date, broken custom event) leaves the battle
pass off, and the console says why.

## Events

The events live in `instance/battle-pass/` (AMP's file manager):

| File | What it is |
|---|---|
| `events.json` | Built-in events. Rewritten on every start, so don't edit it. Copy from it instead. |
| `custom-events.json` | Your events. Created once with an example, then never touched by the template. An event here with the same key as a built-in one replaces it. |
| `client/battle_pass.lua` | The client file for the running event (see below). Rewritten on every start. |

Built in:

| Key | Event | Stages |
|---|---|---|
| `precursors-2026` | Legacy of the Precursors, the game's current battle pass | 4 stages: 41, 41, 30 and 30 levels. Stages 1–2 give Overseer parts, stage 3 Guard parts; 23 of stage 4's rewards are premium. |

Only the current battle pass can be rebuilt. Earlier seasons reused the same bundle names
(`Bundle_Battle_Pass_1_1` …) with other contents, and the game files only hold the latest. A custom event
can rearrange those rewards and hand out anything else in the [reward list](#rewards) below.

The example in `custom-events.json`, `example-precursors-all-free`, is the same 4 stages with every reward
on the free track.

### Format

```json
{
  "events": {
    "my-event": {
      "name": "My event",
      "stages": [
        {
          "cost": 100,
          "rewards": [
            "Bundle_Battle_Pass_1_1",
            "Bundle_Battle_Pass_3_30",
            { "bundle": "Bundle_Wasp_Stuff", "cost": 500 },
            { "bundle": "Bundle_Battle_Pass_4_2", "gold": true }
          ]
        },
        {
          "cost": 150,
          "rewards": ["Bundle_Battle_Pass_2_1", "Bundle_Battle_Pass_2_2"]
        }
      ]
    }
  }
}
```

- **Key** (`my-event`): letters, digits, `.`, `_` and `-`. This is what goes into Battle Pass Event.
- **`stages`**: one entry per stage, in order. Stage 1 is the first entry.
- **`rewards`**: one entry per level, in order. A plain string is a bundle with the stage's `cost` (default
  100 xenochips) and `gold` false. An object can set `cost` and `gold` for that one level.
- **`gold`**: `true` marks a premium-track reward. Premium came with the paid battle pass (a Steam DLC), which
  can't be bought on a self-hosted server, so leave it `false` unless you unlock premium for players by hand
  (`dbg_testBattlePassUnlockPremium` in [CONSOLE.md](CONSOLE.md); untested).
- **`name`**, **`description`**: shown in the console and in the client file; the game itself doesn't show them.
- **`uiShip`**, **`uiShip2`** (optional): the ship whose progress the client's battle pass window shows for
  stages 1–2 and for stage 3. Defaults: `Ship_Race5_H_OVERSEER_Rank9` and `Ship_Race5_L_GUARD_Rank5`.
- Every reward must be a rewardable bundle on the server (all are listed under [Rewards](#rewards)).
  Otherwise Shard would reject the whole battle pass, so the template checks first and says which bundle
  is wrong.
- The game has used up to 4 stages and 41 levels per stage. More may or may not work; the console warns.

Stages are numbered by position, so the server gets every stage up to the highest active one. Battle Pass
Stages `3` sends stages 1–3 and activates only stage 3.

## The client file

The game client doesn't get the rewards from the server. It reads them from its own
`data/gamedata/shared/battle_pass.lua`, and the server only tells it the start time and which stages are
active. The unmodified client knows stage 1 of `precursors-2026` and nothing else.

- **`precursors-2026` with Battle Pass Stages `1`** (the default) works with every client, without changes.
- **Anything else** (more stages, a custom event) needs the matching client file on every player's PC.
  Otherwise the client shows the wrong rewards or none at all. The console says when this is the case.

To install it, copy `instance/battle-pass/client/battle_pass.lua` from the server over
`<game folder>/data/gamedata/shared/battle_pass.lua` (keep a backup of the original). Do this again whenever
the event or the active stages change. A Steam update or "Verify integrity of game files" puts the original
back.

*Not yet tested in the client:* stages 2–4 and custom events. The file has the same format as the game's
own, but whether the client picks up a replaced file and how it shows more than one stage hasn't been
checked yet.

## Rewards

Every rewardable bundle on the server, generated from `server/cloud/data/store.xml`. The shop ID and the
contents are the same as in [items/bundles.md](items/bundles.md); the IDs work with `give <uid> item` too.
"Track" is the original battle pass's premium marking.

### Stage 1

| Level | Bundle | Shop ID | Reward | Contents | Track |
|---:|---|---:|---|---|---|
| 1 | `Bundle_Battle_Pass_1_1` | 1176 | Portrait | Portrait 375 (item 130342) |  |
| 2 | `Bundle_Battle_Pass_1_2` | 8305 | Pattern | decal or paint BP17_Maze |  |
| 3 | `Bundle_Battle_Pass_1_3` | 9587 | Experience bonus | booster Battle Pass Exp15 |  |
| 4 | `Bundle_Battle_Pass_1_4` | 1074 | Paint | decal or paint BP16_Matt_Jacarta |  |
| 5 | `Bundle_Battle_Pass_1_5` | 7791 | Sticker pack | decal or paint battle_pass24_1, decal or paint battle_pass24_2, decal or paint battle_pass24_3 |  |
| 6 | `Bundle_Battle_Pass_1_6` | 2592 | Pattern | decal or paint BP17_Metal |  |
| 7 | `Bundle_Battle_Pass_1_7` | 8278 | Weapon | Protozan Blaster T5 Mk1 (item 1346) |  |
| 8 | `Bundle_Battle_Pass_1_8` | 2602 | Portrait | Portrait 376 (item 130343) |  |
| 9 | `Bundle_Battle_Pass_1_9` | 907 | Ellydium destroyer “Ze'Ta” parts | 30× Crystal neocortex, 30× Crystal archcortex |  |
| 10 | `Bundle_Battle_Pass_1_10` | 953 | Decor | decal or paint BP14_Cloud |  |
| 11 | `Bundle_Battle_Pass_1_11` | 9758 | Free experience | 5,250,000 XP |  |
| 12 | `Bundle_Battle_Pass_1_12` | 9852 | Large seed-chip pack | 15× rank 17 autogen modules, 10× rank 17 autogen modules, 5× rank 17 autogen modules, 5× rank 17 autogen modules |  |
| 13 | `Bundle_Battle_Pass_1_13` | 12265 | Ship part | Protazan cabin |  |
| 14 | `Bundle_Battle_Pass_1_14` | 8508 | Pattern | decal or paint BP17_Drone |  |
| 15 | `Bundle_Battle_Pass_1_15` | 9072 | Taunt | Taunt 345 (item 120346) |  |
| 16 | `Bundle_Battle_Pass_1_16` | 8338 | Access to special bounties | booster Golden Aura Battle Pass |  |
| 17 | `Bundle_Battle_Pass_1_17` | 4312 | Sticker pack | decal or paint battle_pass24_4, decal or paint battle_pass24_5, decal or paint battle_pass24_6 |  |
| 18 | `Bundle_Battle_Pass_1_18` | 682 | Paint | decal or paint BP16_Matt_Purplenavy |  |
| 19 | `Bundle_Battle_Pass_1_19` | 1592 | Weapon | Charged Blast T5 Mk1 (item 11033) |  |
| 20 | `Bundle_Battle_Pass_1_20` | 1163 | Pattern | decal or paint BP18_Nature |  |
| 21 | `Bundle_Battle_Pass_1_21` | 1915 | Decor | decal or paint BP24_Detector |  |
| 22 | `Bundle_Battle_Pass_1_22` | 10738 | Portrait | Portrait 377 (item 130344) |  |
| 23 | `Bundle_Battle_Pass_1_23` | 669 | Credit bonus | booster Battle Pass Credits15 |  |
| 24 | `Bundle_Battle_Pass_1_24` | 8990 | Taunt | Taunt 346 (item 120347) |  |
| 25 | `Bundle_Battle_Pass_1_25` | 9671 | Ship part | 3× Manual controls system |  |
| 26 | `Bundle_Battle_Pass_1_26` | 3370 | Pattern | decal or paint BP18_Pattern |  |
| 27 | `Bundle_Battle_Pass_1_27` | 5135 | WEAPON | Plasma Web Laser T5 Mk1 (item 10085) |  |
| 28 | `Bundle_Battle_Pass_1_28` | 5534 | Resource pack | 50× Enriched monocrystal, 50× Xenocrystal |  |
| 29 | `Bundle_Battle_Pass_1_29` | 1785 | Synthetic polycrystals | 10× Synthetic polycrystal |  |
| 30 | `Bundle_Battle_Pass_1_30` | 1630 | Paint | decal or paint BP16_Matt_Paleviolet |  |
| 31 | `Bundle_Battle_Pass_1_31` | 3143 | Decor | decal or paint BP17_Repairrobot |  |
| 32 | `Bundle_Battle_Pass_1_32` | 644 | Module | Protozan Camouflage T5 Mk1 (item 6272) |  |
| 33 | `Bundle_Battle_Pass_1_33` | 1132 | Rare earth metals | 25× Rare earth metals |  |
| 34 | `Bundle_Battle_Pass_1_34` | 2683 | Portrait | Portrait 378 (item 130345) |  |
| 35 | `Bundle_Battle_Pass_1_35` | 8869 | Premium ship of choice | booster Gold Ship For Free |  |
| 36 | `Bundle_Battle_Pass_1_36` | 4964 | Modules | Combo Restore T5 Mk1 (item 8660), Stealth T5 Mk1 (item 1535) |  |
| 37 | `Bundle_Battle_Pass_1_37` | 7485 | Experience and credit bonus | booster Battle Pass Credits Exp30 |  |
| 38 | `Bundle_Battle_Pass_1_38` | 7878 | Module | Drone Field T5 Mk1 (item 1597) |  |
| 39 | `Bundle_Battle_Pass_1_39` | 2582 | Pattern | decal or paint BP18_Void |  |
| 40 | `Bundle_Battle_Pass_1_40` | 3458 | “Archaeologist” bundle | 4× Special part of the ship “Albireo”, title “Battle_Pass_sum26_1” (title 341), decal or paint BP24_Mask |  |
| 41 | `Bundle_Battle_Pass_1_41` | 9070 | Special bundles | booster Battle Pass Aditional Cell SUM2026 |  |

### Stage 2

| Level | Bundle | Shop ID | Reward | Contents | Track |
|---:|---|---:|---|---|---|
| 1 | `Bundle_Battle_Pass_2_1` | 9400 | Portrait | Portrait 371 (item 130338) |  |
| 2 | `Bundle_Battle_Pass_2_2` | 9401 | Pattern | decal or paint BP23_Quartz |  |
| 3 | `Bundle_Battle_Pass_2_3` | 8794 | Experience bonus | booster Battle Pass Exp15 |  |
| 4 | `Bundle_Battle_Pass_2_4` | 877 | Paint | decal or paint BP23_Matt_23_Mint |  |
| 5 | `Bundle_Battle_Pass_2_5` | 6711 | Sticker pack | decal or paint battle_pass23_7, decal or paint battle_pass23_8, decal or paint battle_pass23_9 |  |
| 6 | `Bundle_Battle_Pass_2_6` | 2305 | Pattern | decal or paint BP23_Cloth |  |
| 7 | `Bundle_Battle_Pass_2_7` | 4084 | Weapon | Omega B T5 Mk1 (item 1681) |  |
| 8 | `Bundle_Battle_Pass_2_8` | 4282 | Portrait | Portrait 370 (item 130337) |  |
| 9 | `Bundle_Battle_Pass_2_9` | 1359 | Modules | 33× Crystal neocortex, 33× Crystal archcortex |  |
| 10 | `Bundle_Battle_Pass_2_10` | 1703 | Decor | decal or paint BP24_Sphere |  |
| 11 | `Bundle_Battle_Pass_2_11` | 8199 | Free experience | 5,250,000 XP |  |
| 12 | `Bundle_Battle_Pass_2_12` | 706 | Large seed-chip pack | 15× rank 17 autogen modules, 10× rank 17 autogen modules, 5× rank 17 autogen modules, 5× rank 17 autogen modules |  |
| 13 | `Bundle_Battle_Pass_2_13` | 1201 | Ship part | 4× Special part of the ship «Emperor» |  |
| 14 | `Bundle_Battle_Pass_2_14` | 11027 | Pattern | decal or paint BP23_Waves |  |
| 15 | `Bundle_Battle_Pass_2_15` | 12277 | Taunt | Taunt 347 (item 120348) |  |
| 16 | `Bundle_Battle_Pass_2_16` | 11916 | Special contracts | booster Golden Aura Battle Pass |  |
| 17 | `Bundle_Battle_Pass_2_17` | 7872 | Sticker pack | decal or paint battle_pass23_10, decal or paint battle_pass23_11, decal or paint battle_pass23_12 |  |
| 18 | `Bundle_Battle_Pass_2_18` | 1400 | Paint | decal or paint BP23_Matt_23_Lilac |  |
| 19 | `Bundle_Battle_Pass_2_19` | 2425 | Module | Relic Gun T5 Mk1 (item 9242) |  |
| 20 | `Bundle_Battle_Pass_2_20` | 1890 | Pattern | decal or paint BP23_Chaotic |  |
| 21 | `Bundle_Battle_Pass_2_21` | 4136 | Decor | decal or paint BP24_Eye |  |
| 22 | `Bundle_Battle_Pass_2_22` | 11764 | Portrait | Portrait 369 (item 130336) |  |
| 23 | `Bundle_Battle_Pass_2_23` | 1387 | Credit bonus | booster Battle Pass Credits15 |  |
| 24 | `Bundle_Battle_Pass_2_24` | 12110 | Taunt | Taunt 348 (item 120349) |  |
| 25 | `Bundle_Battle_Pass_2_25` | 674 | Ship part | 4× Special part of the ship “Relic” |  |
| 26 | `Bundle_Battle_Pass_2_26` | 6544 | Pattern | decal or paint BP23_Giraffe |  |
| 27 | `Bundle_Battle_Pass_2_27` | 9055 | Weapon | Turn Gun T5 Mk1 (item 685) |  |
| 28 | `Bundle_Battle_Pass_2_28` | 8452 | Resource pack | 50× Enriched monocrystal, 50× Xenocrystal |  |
| 29 | `Bundle_Battle_Pass_2_29` | 3138 | Resources | 10× Synthetic polycrystal |  |
| 30 | `Bundle_Battle_Pass_2_30` | 1282 | Paint | decal or paint BP23_Matt_23_Lemon |  |
| 31 | `Bundle_Battle_Pass_2_31` | 2132 | Decor | decal or paint Fly_Lantern |  |
| 32 | `Bundle_Battle_Pass_2_32` | 8631 | Blueprint | Emperors Robe T5 Mk1 (item 1521) |  |
| 33 | `Bundle_Battle_Pass_2_33` | 790 | Rare resource pack | 25× Rare earth metals |  |
| 34 | `Bundle_Battle_Pass_2_34` | 1958 | Portrait | Portrait 368 (item 130335) |  |
| 35 | `Bundle_Battle_Pass_2_35` | 10321 | Ship | booster Gold Ship For Free2 |  |
| 36 | `Bundle_Battle_Pass_2_36` | 7948 | Blueprint | Rocket Mine T5 Mk1 (item 7500) |  |
| 37 | `Bundle_Battle_Pass_2_37` | 5837 | Experience and credit bonus | booster Battle Pass Credits Exp30 |  |
| 38 | `Bundle_Battle_Pass_2_38` | 6175 | Blueprint | Shrike Heal All T5 Mk1 (item 11169), Adaptive Defense T5 Mk1 (item 5685), Shrike Crit T5 Mk1 (item 1922) |  |
| 39 | `Bundle_Battle_Pass_2_39` | 1784 | Pattern | decal or paint BP23_Evil |  |
| 40 | `Bundle_Battle_Pass_2_40` | 11363 | “Federation coordinator” bundle | 4× Part of the ship “Shrike”, title “Battle_Pass_sum26_2” (title 342), decal or paint Veteran_RClaw |  |
| 41 | `Bundle_Battle_Pass_2_41` | 8943 | Special bundles | booster Battle Pass Aditional Cell SUM2026 |  |

### Stage 3

| Level | Bundle | Shop ID | Reward | Contents | Track |
|---:|---|---:|---|---|---|
| 1 | `Bundle_Battle_Pass_3_1` | 9351 | Portrait | Portrait 374 (item 130341) |  |
| 2 | `Bundle_Battle_Pass_3_2` | 2904 | Module | Iapetus (item 1262) |  |
| 3 | `Bundle_Battle_Pass_3_3` | 4647 | Ammunition | 30× Weapon Mod Laser Huge Killer (item 2669) |  |
| 4 | `Bundle_Battle_Pass_3_4` | 9116 | Resources | 15× Enriched monocrystal |  |
| 5 | `Bundle_Battle_Pass_3_5` | 8966 | Ammunition | 30× Magnetic mine (item 11420) |  |
| 6 | `Bundle_Battle_Pass_3_6` | 11595 | Pattern | decal or paint BP15_Chlorophyll |  |
| 7 | `Bundle_Battle_Pass_3_7` | 843 | Resources | 15× Enriched monocrystal |  |
| 8 | `Bundle_Battle_Pass_3_8` | 3121 | Seed-chip bundle | 10× rank 17 autogen modules |  |
| 9 | `Bundle_Battle_Pass_3_9` | 8666 | Resources | 5× Synthetic polycrystal |  |
| 10 | `Bundle_Battle_Pass_3_10` | 8992 | Ship part | 25× Energy router package |  |
| 11 | `Bundle_Battle_Pass_3_11` | 3188 | Title | title “Event_Pass_sum26_3” (title 343) |  |
| 12 | `Bundle_Battle_Pass_3_12` | 647 | Module | Zeus (item 9837) |  |
| 13 | `Bundle_Battle_Pass_3_13` | 1144 | Seed-chip bundle | 10× rank 17 autogen modules |  |
| 14 | `Bundle_Battle_Pass_3_14` | 10746 | Ammunition | 30× Crystal drone (item 2591) |  |
| 15 | `Bundle_Battle_Pass_3_15` | 11941 | Module blueprint | Active module “Chaos” systems booster |  |
| 16 | `Bundle_Battle_Pass_3_16` | 5017 | Paint | decal or paint BP19_Matt_19_SpicyPink |  |
| 17 | `Bundle_Battle_Pass_3_17` | 7533 | Module | Nemesis (item 1348) |  |
| 18 | `Bundle_Battle_Pass_3_18` | 1333 | Resources | 15× Enriched monocrystal |  |
| 19 | `Bundle_Battle_Pass_3_19` | 8946 | Ammunition | 30× Weapon Mod Plasma Huge Killer (item 10429) |  |
| 20 | `Bundle_Battle_Pass_3_20` | 1825 | Ship part | 4× Part of the ship “Helios” |  |
| 21 | `Bundle_Battle_Pass_3_21` | 4011 | A set of stickers | decal or paint battle_pass12_18, decal or paint battle_pass12_19, decal or paint battle_pass12_20 |  |
| 22 | `Bundle_Battle_Pass_3_22` | 832 | Ammunition | 30× Crystal drone (item 1604) |  |
| 23 | `Bundle_Battle_Pass_3_23` | 10721 | Seed-chip bundle | 10× rank 17 autogen modules |  |
| 24 | `Bundle_Battle_Pass_3_24` | 11755 | Resources | 10× Synthetic polycrystal |  |
| 25 | `Bundle_Battle_Pass_3_25` | 609 | Weapon blueprint | “Small artillery” weapon |  |
| 26 | `Bundle_Battle_Pass_3_26` | 6010 | Decor | decal or paint BP17_Cat |  |
| 27 | `Bundle_Battle_Pass_3_27` | 8156 | Ammunition | 30× Weapon Mod Rail Huge Killer (item 3491) |  |
| 28 | `Bundle_Battle_Pass_3_28` | 9328 | Seed-chip bundle | 10× rank 17 autogen modules |  |
| 29 | `Bundle_Battle_Pass_3_29` | 2826 | Resources | 10× Synthetic polycrystal |  |
| 30 | `Bundle_Battle_Pass_3_30` | 2684 | Ship part | 4× Super dense crystals containment |  |

### Stage 4

| Level | Bundle | Shop ID | Reward | Contents | Track |
|---:|---|---:|---|---|---|
| 1 | `Bundle_Battle_Pass_4_1` | 12182 | Portrait | Portrait 267 (item 130234) |  |
| 2 | `Bundle_Battle_Pass_4_2` | 5224 | Ammunition | 30× Missile with a chemical container (item 1845) | premium |
| 3 | `Bundle_Battle_Pass_4_3` | 7790 | Ammunition | 30× Chain Rockets T5 Mk3 (item 4557) | premium |
| 4 | `Bundle_Battle_Pass_4_4` | 756 | Resources | 15× Enriched monocrystal | premium |
| 5 | `Bundle_Battle_Pass_4_5` | 6067 | Ammunition | 30× Repair rockets (item 1004) |  |
| 6 | `Bundle_Battle_Pass_4_6` | 8118 | Pattern | decal or paint BP18_Waves | premium |
| 7 | `Bundle_Battle_Pass_4_7` | 1442 | Resources | 15× Enriched monocrystal | premium |
| 8 | `Bundle_Battle_Pass_4_8` | 1529 | Seed-chip bundle | 10× rank 17 autogen modules | premium |
| 9 | `Bundle_Battle_Pass_4_9` | 10759 | Resources | 5× Synthetic polycrystal | premium |
| 10 | `Bundle_Battle_Pass_4_10` | 891 | Ship part | Special part of the ship “Cor Legibus” |  |
| 11 | `Bundle_Battle_Pass_4_11` | 1516 | Title | title “Event_Pass_ny26_4” (title 337) | premium |
| 12 | `Bundle_Battle_Pass_4_12` | 9490 | Ammunition | 30× Anchor mine (item 10456) | premium |
| 13 | `Bundle_Battle_Pass_4_13` | 11856 | Seed-chip bundle | 10× rank 17 autogen modules | premium |
| 14 | `Bundle_Battle_Pass_4_14` | 8058 | Ammunition | 30× “Radiation fog” missile (item 725) | premium |
| 15 | `Bundle_Battle_Pass_4_15` | 8565 | Module blueprint | Cor V2 TPAlly T5 Mk1 blueprint |  |
| 16 | `Bundle_Battle_Pass_4_16` | 2063 | Paint | decal or paint BP18_Matt_18_Lightmauve | premium |
| 17 | `Bundle_Battle_Pass_4_17` | 3815 | Ammunition | 30× Cluster rockets (item 3952) | premium |
| 18 | `Bundle_Battle_Pass_4_18` | 607 | Resources | 15× Enriched monocrystal | premium |
| 19 | `Bundle_Battle_Pass_4_19` | 1372 | Ammunition | 30× Anomalous torpedoes (item 3083) | premium |
| 20 | `Bundle_Battle_Pass_4_20` | 11272 | Ship part | Special part of the ship “Cor Legibus” |  |
| 21 | `Bundle_Battle_Pass_4_21` | 1727 | A set of stickers | decal or paint battle_pass15_9, decal or paint battle_pass16_3, decal or paint battle_pass16_7 | premium |
| 22 | `Bundle_Battle_Pass_4_22` | 10446 | Ammunition | 30× “Attractor” missile (item 1586) | premium |
| 23 | `Bundle_Battle_Pass_4_23` | 596 | Seed-chip bundle | 10× rank 17 autogen modules | premium |
| 24 | `Bundle_Battle_Pass_4_24` | 9181 | Resources | 10× Synthetic polycrystal | premium |
| 25 | `Bundle_Battle_Pass_4_25` | 9216 | Weapon blueprint | Cor V2 Smart RL T5 Mk1 blueprint |  |
| 26 | `Bundle_Battle_Pass_4_26` | 2503 | Decor | decal or paint BP20_Epichero | premium |
| 27 | `Bundle_Battle_Pass_4_27` | 4280 | Blueprints | Long-distance minelayer Mk.5, “Anchor” slowing weapon Mk.5, “Ion-cyclotron discharger” weapon Mk.5, “Guardian-1” blaster Mk.5 | premium |
| 28 | `Bundle_Battle_Pass_4_28` | 8006 | Seed-chip bundle | 10× rank 17 autogen modules | premium |
| 29 | `Bundle_Battle_Pass_4_29` | 1585 | Resources | 10× Synthetic polycrystal | premium |
| 30 | `Bundle_Battle_Pass_4_30` | 4043 | Ship part | Special part of the ship “Cor Legibus” |  |

### Other rewardable bundles

Not part of any battle pass, but allowed as rewards.

| Bundle | Shop ID | Reward | Contents |
|---|---:|---|---|
| `Bundle_AsteroidPremiumLoot` | 210724 | Asteroid conqueror | booster Asteroid Premium Loot10 |
| `Bundle_Avatar_117` | 210726 | Portrait | Portrait 117 (item 130084) |
| `Bundle_Avatar_120` | 210719 | Portrait | Portrait 120 (item 130087) |
| `Bundle_Avatar_133` | 210720 | Portrait | Portrait 133 (item 130100) |
| `Bundle_Avatar_134` | 210722 | Portrait | Portrait 134 (item 130101) |
| `Bundle_Avatar_135` | 210723 | Portrait | Portrait 135 (item 130102) |
| `Bundle_Avatar_136` | 210725 | Portrait | Portrait 148 (item 130115) |
| `Bundle_Avatar_150` | 210728 | Portrait “Empire destroyer commander” | Portrait 150 (item 130117) |
| `Bundle_Avatar_151` | 210729 | Portrait “Jimmy the Shark” | Portrait 151 (item 130118) |
| `Bundle_Avatar_152` | 210730 | Portrait “Selena Oliveira” | Portrait 152 (item 130119) |
| `Bundle_Avatar_157` | 210731 | Portrait “Leif Erikson” | Portrait 157 (item 130124) |
| `Bundle_Avatar_158` | 210732 | Portrait “Commander Jocund” | Portrait 158 (item 130125) |
| `Bundle_Avatar_165` | 210733 | Portrait “Lieutenant Getz Bayer” | Portrait 165 (item 130132) |
| `Bundle_Avatar_171` | 210753 | Ship pilot portrait | Portrait 171 (item 130138) |
| `Bundle_Avatar_173` | 210734 | Portrait | Portrait 173 (item 130140) |
| `Bundle_Avatar_174` | 210735 | Portrait | Portrait 174 (item 130141) |
| `Bundle_Avatar_176` | 210755 | Ship pilot portrait | Portrait 176 (item 130143) |
| `Bundle_Avatar_177` | 210757 | Ship pilot portrait | Portrait 177 (item 130144) |
| `Bundle_Avatar_178` | 210759 | Ship pilot portrait | Portrait 178 (item 130145) |
| `Bundle_Avatar_179` | 210761 | Ship pilot portrait | Portrait 179 (item 130146) |
| `Bundle_Avatar_180` | 210765 | Ship pilot portrait | Portrait 180 (item 130147) |
| `Bundle_Avatar_181` | 210736 | Portrait | Portrait 181 (item 130148) |
| `Bundle_Avatar_183` | 210738 | Portrait | Portrait 183 (item 130150) |
| `Bundle_Avatar_184` | 210739 | Portrait | Portrait 184 (item 130151) |
| `Bundle_Avatar_185` | 210767 | Ship pilot portrait | Portrait 185 (item 130152) |
| `Bundle_Avatar_195` | 210763 | Ship pilot portrait | Portrait 195 (item 130162) |
| `Bundle_Avatar_196` | 210740 | Portrait | Portrait 196 (item 130163) |
| `Bundle_Avatar_198` | 210741 | Portrait | Portrait 198 (item 130165) |
| `Bundle_Avatar_200` | 210769 | Ship pilot portrait | Portrait 200 (item 130167) |
| `Bundle_Avatar_209` | 210742 | Portrait | Portrait 209 (item 130176) |
| `Bundle_Avatar_210` | 210743 | Portrait | Portrait 210 (item 130177) |
| `Bundle_Avatar_216` | 210744 | Portrait | Portrait 216 (item 130183) |
| `Bundle_Avatar_217` | 210745 | Portrait | Portrait 217 (item 130184) |
| `Bundle_Avatar_221` | 210779 | Ship pilot portrait | Portrait 221 (item 130188) |
| `Bundle_Avatar_223` | 210746 | Portrait | Portrait 223 (item 130190) |
| `Bundle_Avatar_229` | 210747 | Kolhis | Portrait 229 (item 130196) |
| `Bundle_Avatar_230` | 210748 | Ship pilot portrait | Portrait 230 (item 130197) |
| `Bundle_Avatar_231` | 210750 | Jimmy Ryder | Portrait 231 (item 130198) |
| `Bundle_Avatar_237` | 210752 | Ship pilot portrait | Portrait 237 (item 130204) |
| `Bundle_Avatar_238` | 210771 | Ship pilot portrait | Portrait 238 (item 130205) |
| `Bundle_Avatar_239` | 210772 | Lazarus the Preacher | Portrait 239 (item 130206) |
| `Bundle_Avatar_249` | 210774 | Ship pilot portrait | Portrait 249 (item 130216) |
| `Bundle_Avatar_254` | 210775 | Ship pilot portrait | Portrait 254 (item 130221) |
| `Bundle_Avatar_255` | 210776 | Ship pilot portrait | Portrait 255 (item 130222) |
| `Bundle_Avatar_256` | 210777 | Ship pilot portrait | Portrait 256 (item 130223) |
| `Bundle_Avatar_261` | 210780 | Ship pilot portrait | Portrait 261 (item 130228) |
| `Bundle_Avatar_262` | 210781 | Ship pilot portrait | Portrait 262 (item 130229) |
| `Bundle_Avatar_263` | 210782 | Ship pilot portrait | Portrait 263 (item 130230) |
| `Bundle_Avatar_268` | 210783 | Ship pilot portrait | Portrait 268 (item 130235) |
| `Bundle_Avatar_273` | 210784 | Stormbringer | Portrait 273 (item 130240) |
| `Bundle_Avatar_278` | 210785 | Ship pilot portrait | Portrait 278 (item 130245) |
| `Bundle_Avatar_279` | 210786 | Ship pilot portrait | Portrait 279 (item 130246) |
| `Bundle_Avatar_280` | 210787 | Ship pilot portrait | Portrait 280 (item 130247) |
| `Bundle_Avatar_286` | 210788 | Ship pilot portrait | Portrait 286 (item 130253) |
| `Bundle_Avatar_291` | 210789 | Ship pilot portrait | Portrait 291 (item 130258) |
| `Bundle_Avatar_292` | 210790 | Ship pilot portrait | Portrait 292 (item 130259) |
| `Bundle_Avatar_297` | 210791 | Ship pilot portrait | Portrait 297 (item 130264) |
| `Bundle_Avatar_298` | 210792 | Ship pilot portrait | Portrait 298 (item 130265) |
| `Bundle_Avatar_307` | 210793 | Ship pilot portrait | Portrait 307 (item 130274) |
| `Bundle_Avatar_308` | 210794 | Ship pilot portrait | Portrait 308 (item 130275) |
| `Bundle_Avatar_317` | 210795 | Ship pilot portrait | Portrait 317 (item 130284) |
| `Bundle_Avatar_322` | 210796 | Ship pilot portrait | Portrait 322 (item 130289) |
| `Bundle_Avatar_327` | 210798 | Ship pilot portrait | Portrait 327 (item 130294) |
| `Bundle_Avatar_328` | 210797 | Ship pilot portrait | Portrait 328 (item 130295) |
| `Bundle_Avatar_337` | 210799 | Ship pilot portrait | Portrait 337 (item 130304) |
| `Bundle_Avatar_338` | 210800 | Ship pilot portrait | Portrait 338 (item 130305) |
| `Bundle_Avatar_347` | 210801 | Ship pilot portrait | Portrait 347 (item 130314) |
| `Bundle_Avatar_352` | 210802 | Ship pilot portrait | Portrait 352 (item 130319) |
| `Bundle_Avatar_357` | 210803 | Ship pilot portrait | Portrait 357 (item 130324) |
| `Bundle_Avatar_362` | 210804 | Ship pilot portrait | Portrait 362 (item 130329) |
| `Bundle_Avatar_363` | 210805 | Valentina Vega | Portrait 363 (item 130330) |
| `Bundle_Avatar_372` | 210806 | Ship pilot portrait | Portrait 372 (item 130339) |
| `Bundle_Avatar_373` | 210807 | Ship pilot portrait | Portrait 373 (item 130340) |
| `Bundle_Caltrop_Stuff` | 210766 | Taunt | title “CALTROP_1” (title 253) |
| `Bundle_Chest_Gold_Reward` | 210037 | Complimentary container | 2 random rewards, e.g. one of Pirate 'Orion' Targeting Complex blueprint R11-15, 'Pirate Engine Overcharge' blueprint R13-15, 'Orion' Targeting Complex blueprint R11-15, 'Valkyrie System' blueprint R11-15, 'Augmented cooler' blueprint R12-15, 'Hybrid missile' blueprint R12-15, … (27 in total) |
| `Bundle_Confessor_Stuff` | 210773 | Taunt | Taunt 246 (item 120244) |
| `Bundle_Cyning_Stuff` | 210754 | Taunt | title “CYNING_1” (title 247) |
| `Bundle_DLC_Seed_Balanced_01` | 210717 | Seed-chips | 30 days of premium, 15× rank 17 autogen modules, 5× rank 17 autogen modules, rank 17 autogen module; One random reward of booster DLCCred Exp60 |
| `Bundle_DLC_Seed_Special_01` | 210718 | Seed-chips | 30 days of premium, 20× rank 17 autogen modules, 2× rank 17 autogen modules; One random reward of booster DLCCred Exp120 |
| `Bundle_DailyChest_T1` | 210017 | Ciphered container | One random reward of 5,000 XP, 20,000 XP, booster Daily Synergy25, booster Daily Synergy35, booster Daily Credits25, booster Daily Credits35; shop price 1,000,000,000 credits |
| `Bundle_DragonflyStuff` | 210749 | Taunts and titles | title “DRAGONFLY_1” (title 230), title “DRAGONFLY_2” (title 231), title “DRAGONFLY_3” (title 232), Taunt 231 (item 120232), Taunt 232 (item 120233), Taunt 233 (item 120234) |
| `Bundle_Elite_Donater` | 210778 | 300 xenochips in the next phase | booster Elite Donater BP23 2 |
| `Bundle_EventCred_BattlePass` | 210727 | Special license “Legacy of the Precursors” | booster Battle Pass Event Cred50 NY23, booster DLCCred Exp30 |
| `Bundle_EventCredits` | 210721 | *Event Credits* | One random reward of 100 event currency |
| `Bundle_Iridium_Overseer_T3` | 210092 | *Iridium Overseer T3* | booster Bundle Daily Pvp open, booster Bundle Daily Raid open, booster Bundle Daily Pve open; One random reward of booster Bundle Iridium Overseer T3 1 open, booster Bundle Iridium Overseer T3 2 open |
| `Bundle_Iridium_Overseer_T4` | 210093 | *Iridium Overseer T4* | booster Bundle Daily Pvp open, booster Bundle Daily Raid open, booster Bundle Daily Pve open; 2 random rewards, e.g. one of booster Bundle Iridium Overseer T3 1 open, booster Bundle Iridium Overseer T4 1 open, booster Bundle Iridium Overseer T4 3 open |
| `Bundle_Iridium_Overseer_T5` | 210094 | *Iridium Overseer T5* | booster Bundle Daily Pvp open, booster Bundle Daily Raid open, booster Bundle Daily Pve open; 3 random rewards, e.g. one of booster Bundle Iridium Overseer T3 1 open, booster Bundle Iridium Overseer T4 1 open, booster Bundle Iridium Overseer T5 1 open, booster Bundle Iridium Overseer T5 2 open |
| `Bundle_Jaguar_Stuff` | 210762 | Taunt | title “JAGUAR_1” (title 251) |
| `Bundle_Kraken_Stuff` | 210758 | Taunt | title “KRAKEN_1” (title 249) |
| `Bundle_Mjolnir_Stuff` | 210756 | Taunt | title “MJOLNIR_1” (title 248) |
| `Bundle_Octopus_Stuff` | 210764 | Taunt | title “OCTOPUS_1” (title 252) |
| `Bundle_Peregrine_Stuff` | 210760 | Taunt | title “PEREGRINE_1” (title 250) |
| `Bundle_Pikabu` | 210737 | Pikabu | Portrait 182 (item 130149), Taunt 161 (item 120162), Taunt 162 (item 120163), Taunt 163 (item 120164), decal or paint Pikabu_01, decal or paint pikabu |
| `Bundle_Ronin_Stuff` | 210770 | Taunt | title “RONIN_1” (title 255) |
| `Bundle_Saw_One_Stuff` | 210768 | Taunt | title “SAWONE_1” (title 254) |
| `Bundle_Secret_Chest_T5` | 210016 | Resource container | One random reward of 5× Xenocrystal, 5× Enriched monocrystal, 20× Alien composite block; shop price 1,000,000,000 credits |
| `Bundle_Wasp_Stuff` | 210751 | Taunts and titles | title “WASP_1” (title 235), title “WASP_2” (title 236), title “WASP_3” (title 237), Taunt 238 (item 120249), Taunt 239 (item 120250), Taunt 240 (item 120251) |

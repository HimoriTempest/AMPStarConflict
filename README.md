# AMP Generic Module template: Star Conflict

Runs a self-hosted Star Conflict server in [CubeCoders AMP](https://cubecoders.com/AMP): Hub with all its
services and battle servers, plus a bundled MongoDB. The game's server is a Windows build, so the template
always runs in AMP's `cubecoders/ampbase:wine-stable` container, where it runs under Wine. That works on
Linux and Windows hosts.

## Docs

| Document | What's in it |
|---|---|
| This README | Installing and running the server: requirements, ports, settings, console, database |
| [docs/CONSOLE.md](docs/CONSOLE.md) | Every console command: the template's account commands and the game's own Hub and Shard commands |
| [docs/ITEMS.md](docs/ITEMS.md) | Item, ship and title IDs for the `give` commands, split into lists in [docs/items/](docs/items/) |

## Requirements

- AMP 2.6 or newer, with Docker available to AMP (the template only runs in a container).
- About 15 GB of disk space for the server files and the database.
- The server files, either:
  - a **Steam account** with Star Conflict in its library (free to play; add it once in the Steam client).
    A separate account for the server is recommended.
  - or a **server tarball**: a direct `.tar.gz` link with `./cloud`, `./data` and `./sys_data` at the top.

## Installation

1. In ADS go to **Configuration → Instance Deployment → Configuration Repositories**, add
   `HimoriTempest/AMPStarConflict:main` and click **Fetch latest**.
2. Create an instance: *Generic*, then *Star Conflict*.
3. Pick the **Server Files Source** (Updates settings), then **Update** the instance:
   - *SteamCMD* (default) installs Steam app 212070, about 12 GB. AMP asks for the Steam login and
     Steam Guard code during the update. Anonymous download doesn't work.
   - *Tarball URL* downloads and extracts the file from **Server Tarball URL**, about 10 GB. For a
     Nextcloud share, use `https://<host>/public.php/dav/files/<token>` or append `/download` to the share link.
4. Set the address players connect to (see below), then start the instance.

The instance shows as **Ready** once the load balancer has registered. Later updates keep the database
and your own `*_local.cfg` files.

## Folder structure

AMP's file manager (and AMP's backups) start at `instance/`:

```
star-conflict/
├── instance/                  file manager root, included in backups
│   ├── server/                game files (from SteamCMD or the tarball)
│   │   ├── cloud/             Hub and services; config in cloud/data/
│   │   ├── data/  sys_data/   game data
│   │   └── server-logs/       all logs and crash dumps
│   └── database/              bundled MongoDB 3.4
│       ├── mongodb/           binaries
│       ├── data/              the database
│       └── mongod.log
├── .wine/                     Wine prefix
└── hub-wrapper.sh, mongodb.sh, amp-settings.sh   written by the template on every start
```

Updates never touch `database/` or `server/server-logs/`. Files you may edit are in
`server/cloud/data/` (see [Your own cvars](#your-own-cvars)); everything the template writes is
regenerated on every start.

## Server address

Players connect to the address in the server's `cloud.xml`. The template sets it on every start, from:

1. **Public Address** (Network settings), if set;
2. otherwise AMP's **Advertised Connection Address** (instance settings → Network and Ports).

The console shows which one was used (`Cloud node address: … (source)`).

## Ports

Forward these to players:

| Port | Proto | Service |
|---|---|---|
| 3801 | TCP | Load balancer (players enter this port) |
| 3802 | TCP | Shard |
| 3815 | TCP | Chat |
| 35000–35049 | TCP+UDP | Battle servers |

The instance also uses **internal ports** (Hub, Hub status, MongoDB, and the two remote console
ports). **Never forward these.** The game listens on every address and can't be bound to one IP, so keep
them outside your forwarded ranges and, ideally, firewall them to loopback only.

Keep all ports **below 32768**. Ports in Linux's ephemeral range can be taken by outgoing connections,
and the service then fails with `Failed to listen on port …`.

### Several servers on one host

Every port comes from AMP's port allocation, so several instances can run side by side. Give each
instance its own set of ports below 32768. For example, use the defaults for the first, and
4801/4802/4815, 4850, 10001, 28017 and 36000–36049 for a second. Players choose a server by its load
balancer port.

## Connecting with the client

On the client's login screen, enter the server's address and the **load balancer port** (not the battle port).

Steam starts the client with `-launchLocalServer`, which also starts a local server on the player's
machine. To prevent that, empty the `executable` attributes of `<Mongo>` and `<Hub>` in
`win64\local_server.xml`. On Linux/Proton, use this launch option instead:

```
bash -c 'exec "${@/-launchLocalServer/}"' -- %command%
```

## Settings

Every setting is reversible: the default is the game's original value, and setting it back restores
the original behaviour on the next start. Each setting's description in AMP names the file it edits.

| Group | Settings | Notes |
|---|---|---|
| Accounts | Starting Credits, Starting Galactic Standards, Starting Premium Days, Premium Hours Per Account Rank, Infinite Premium For All Players, Starter Pack (Credits / GS / Token Credits) | Starting values and the starter pack only apply to accounts created afterwards |
| Economy | XP / Credit / First Win Of The Day XP Multiplier, Bonus GS Per Win / Loss, Daily Token Credit Limit, Ship / Module / Ammo Price Multiplier, Buy Everything | Buy Everything allows obsolete and dev items, which can break an account |
| Battles | AFK Kick Time, Open Space Resurrection Cost | |
| Matchmaking | Ships In Arena Matches, Kick Excessive Bots, Coop vs AI Team Size (experimental) | Ships In Arena Matches is the total per match, filled with bots (max. 16 in the newbie queue, 24 otherwise) |
| Battle Pass | Battle Pass, Start, End | Dates are taken as UTC. Changing the start begins a new battle pass, and existing progress is lost |
| Network | Public Address, Max Players | |
| Database | MongoDB (Bundled / External), External MongoDB Hosts, MongoDB Cache Size (GB) | See [External MongoDB](#external-mongodb) |
| Shop (experimental) | DLC Packs In Shop, Destroyer Pack Items In Shop | Sells the former DLC and destroyer pack contents for GS. Not yet confirmed in the client |
| Administration | Admin Account IDs, Moderator Account IDs, Remote Console Password | See below |

Steam DLC purchases can't reach a self-hosted server. The shop settings above are the way to make that
content obtainable.

### Your own cvars

Put your own `.cfg` settings in `instance/server/cloud/data/common_local.cfg`. The template only
manages the block between `// >>> AMP managed` and `// <<< AMP managed` there and leaves the rest alone.
This only works for cvars that `default_platform_config.lua` doesn't set. For those, edit that file
outside its AMP block.

Don't edit the other `*_local.cfg` files. The template rewrites them on every start.

## Admins and moderators

List account IDs (uids) in **Admin Account IDs** or **Moderator Account IDs**. The uid is in the console
line `[AMP] Player joined: name (uid N)`, so an account exists after its first login. Changes apply on
the next start.

| Role | Can |
|---|---|
| Admin | Everything: mute and ban, GM tools (player info, renaming), resetting stats and accounts, logging in during maintenance |
| Moderator | Mute and ban (temporary or permanent) and lifting them, logging in during maintenance, observing games |

In game, right-click a player in chat and choose **Punish**.

An account has only one role. Listing an account with open-world access ("Free Space") as admin or
moderator replaces that access.

## Console

### Account commands

The template adds its own commands to AMP's console. They work without a Remote Console Password,
and also while the player is online (the player may need to relog to see the change).

| Command | What it does |
|---|---|
| `player <uid or name>` | Shows an account: uid, nickname, role, online status, GS, credits, tokens, premium, ban and mute, ships, items and titles. Names are matched case-insensitively. |
| `give <uid> gs <amount>` | Adds Galactic Standards. A negative amount takes them away. |
| `give <uid> credits <amount>` | Adds (or takes) credits. |
| `give <uid> tokens <amount>` | Adds (or takes) token credits. |
| `give <uid> premium <days>` | Adds premium time, from now or on top of running premium. |
| `give <uid> item <shop id> [amount]` | Adds an item by its shop id, including items that aren't sold. |
| `give <uid> ship <shop id>` | Adds a ship by its shop id. |
| `give <uid> title <title id>` | Adds a title. |
| `find <text>` | Searches the shop and the titles (at least 3 characters) and shows id, type, name and GS price. |

Example: `find Thor` lists `item 5784 ship Ship_Race1_M_T5_Dlc Thor, 3500 GS`, then `give 3 ship 5784`
gives account 3 the Thor. All item, ship and title IDs are in [ITEMS.md](docs/ITEMS.md).

Decals, paints and open-world access can't be given.

### Game console

With a **Remote Console Password** set, the console also runs the game's own commands: type a command
to run it on Hub, or put `shard ` in front to run it on Shard. All commands are listed in
[CONSOLE.md](docs/CONSOLE.md). Commands take a few seconds. Don't use quotes or backslashes in the password.

The console also reports players joining and leaving (AMP's user list works), and once a minute the
registered accounts, total CPU and total memory of the whole server, as metrics.

## External MongoDB

By default the instance runs its own MongoDB, stored in `instance/database/`. To use a separate one
instead, set **MongoDB** to *External* and **External MongoDB Hosts** to `host:port` (or
`host1:port,host2:port,…` for a replica set).

- **It must be MongoDB 3.4.** The game's database driver doesn't work with newer versions. The
  [AMPStarConflictDB](https://git.himori.dev/Himori/AMPStarConflictDB) template runs a suitable one.
- **No authentication.** The game can't log in to MongoDB, so keep the database on a private network
  or behind a firewall.
- **One database per server.** Two instances pointed at the same MongoDB share all their data.
- **Data isn't moved automatically.** Before switching, stop the instance and copy the contents of
  `instance/database/data/` into the new database's data folder, or dump and restore it with the 3.4
  tools in `instance/database/mongodb/bin/`. Switching back to *Bundled* uses the old data again.

## Backups and upgrades

AMP's backups cover `instance/`, which includes the game files and the database. The database is only
running while the instance runs, so a backup of a stopped instance is always consistent.

When you upgrade from a version before 2.0, the first start moves everything into the new layout and
migrates the old database once. Delete the old folders once the server runs fine. The console says which ones.

## Logs

| What | Where (in the file manager) |
|---|---|
| Hub, services and battles | `server/server-logs/logs/` |
| Crash dumps | `server/server-logs/cloud_exceptions/` |
| Database | `database/mongod.log` |

## Known limitations

- The shop settings and Coop vs AI Team Size are experimental and not confirmed in the client yet.
- The **Punish** menu has been verified server-side, but not yet in the client.
- If an instance stays on "Starting" while the server works, the ready message has changed. Report it.

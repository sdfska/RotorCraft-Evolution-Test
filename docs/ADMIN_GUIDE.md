# RotorCraft Evolution: server administrator's guide

## 1. Requirements

RotorCraft Evolution by FCE Flight Mods adds flyable helicopters to **DayZ 1.30**. Install **Community Framework (CF)** and RotorCraft Evolution on the dedicated server and every client. Load CF first. Client and server must use the same game version and the same mod build. A server-only installation is not sufficient: clients need the models, sounds, inputs and scripts.

Release packages use `@RotorCraftEvolution`. Aircraft classes are `RotorCraft_*`; settings use `RotorCraft_*.json`. The PBO prefix is `RotorCraftEvolution`. Normal and Dev builds use `@RotorCraftEvolutionTest` and `@RotorCraftEvolutionDev`. Load only one copy. An admin-tools mod is optional.

## 2. Install

Stop the server. Extract the signed package and place its mod folder beside the server executable. Locate the folder containing `mod.cpp`, `addons` and `keys`; do not leave an extra archive directory between that folder and its contents.

```text
DayZServer/
  @CF/
  @RotorCraftEvolution/
    mod.cpp
    addons/
      RotorCraftEvolution.pbo
      RotorCraftEvolution.pbo.rotorcraftevolution.bisign
    keys/
      rotorcraftevolution.bikey
  keys/
    rotorcraftevolution.bikey
    <CF public key>
  mpmissions/
  profiles/
  serverDZ.cfg
```

`mod.cpp` displays **RotorCraft Evolution** and **FCE Flight Mods**. Keep the PBO and matching signature together. Copy `@RotorCraftEvolution/keys/rotorcraftevolution.bikey` and CF's supplied `.bikey` into the server's top-level `keys` directory.

Use these mod arguments on the server:

```text
"-mod=@CF;@RotorCraftEvolution"
```

Do not put CF or RotorCraft Evolution in `-serverMod`. If unrelated server-only mods are installed, list only those there:

```text
"-serverMod=@YourServerOnlyMod"
```

Omit `-serverMod` when none are needed. Keep `verifySignatures = 2;` in `serverDZ.cfg`. The `-profiles` argument selects the settings and log directory. See Bohemia's [server configuration reference](https://community.bohemia.net/wiki/DayZ:Server_Configuration) for these options.

**Windows:** copy the folders and keys, then add the mod arguments to the existing startup command or hosting panel. From the server directory, a minimal PowerShell launch is:

```powershell
.\DayZServer_x64.exe -config=serverDZ.cfg -profiles=profiles "-mod=@CF;@RotorCraftEvolution" -dologs
```

**Linux:** copy the same package and keys. Match path case exactly in the launch command, and keep the files named as shipped. Do not unpack or modify the signed PBO. Give the server account read access to the mod and write access to its profiles and mission storage. See Bohemia's [mod installation guidance](https://community.bistudio.com/wiki/DayZ:Modding_Basics).

```sh
./DayZServer -config=serverDZ.cfg -profiles=profiles '-mod=@CF;@RotorCraftEvolution' -dologs
```

Keep the rest of the server's normal launch options and mission configuration. On clients, load CF and the same RotorCraft Evolution package through the DayZ Launcher, in that order. A manually supplied package can be added as a local mod.

## 3. Put aircraft in the world

Use `$profile:RotorCraft_Spawns.json` for automatic placement without mission editing. The server writes it when missing and checks its positions once, about 20 seconds after mission start, allowing saved vehicles to load first. On Nasdara it supplies the four apron positions below; other maps start with an empty list and a log message asking for positions.

| Aircraft class | Seats | Tank / fresh fuel | Standard starting parts |
|---|---:|---:|---|
| `RotorCraft_Mosquito` | 1 | 40 / 10 L | `CarBattery`, two `SparkPlug` items |
| `RotorCraft_MH6` | 4 | 240 / 60 L | `TruckBattery`, `GlowPlug` |
| `RotorCraft_Gazelle` | 4 | 445 / 111.25 L | `TruckBattery`, `GlowPlug` |
| `RotorCraft_BlackHawk` | 12 | 1360 / 340 L | `TruckBattery`, `GlowPlug` |
| `RotorCraft_BlackHawkOpen` | 12 | 1360 / 340 L | Same as Black Hawk; open-cabin model |

The open Black Hawk uses the Black Hawk settings rows and permits seat entry only while grounded. The supplied CE fragments cover the first four classes only. Do not spawn base classes or test/probe classes.

Place aircraft on clear, level terrain, with space for the entire hull and rotor. Avoid buildings, roofs, rocks, steep slopes and overlapping aircraft. A fresh aircraft with no attachments receives its standard electrical parts and a blowtorch with gas after about one second. Pre-attaching anything skips that entire equipment step. No belt or blade item is supplied. Fuel starts at 25% independently of equipment. Mosquito's second spark plug uses slot `RotorCraft_SparkPlug2`.

### Editing the JSON

Stop the server, open `RotorCraft_Spawns.json` in the profile directory selected by `-profiles`, save valid JSON, then restart. A rented host can expose this directory through its file manager. No `init.c` or economy XML changes are needed for this method. Keep normal vehicle persistence enabled.

Nasdara defaults:

```json
{
    "enabled": 1,
    "entries": [
        {"className": "RotorCraft_Mosquito", "x": 11945, "z": 10845, "yaw": 0},
        {"className": "RotorCraft_MH6", "x": 11858, "z": 10912, "yaw": 0},
        {"className": "RotorCraft_Gazelle", "x": 11876, "z": 10912, "yaw": 0},
        {"className": "RotorCraft_BlackHawk", "x": 12012, "z": 10910, "yaw": 0}
    ]
}
```

Use only `enabled` and `entries` at the top level, without duplicate fields or JSON comments. Set `enabled` to **0** to disable placement; this does not remove saved aircraft. Both top-level fields are required in an existing file. For other maps, replace the empty `entries` list with surveyed positions; do not copy Nasdara coordinates.

The complete disabled file can be one line: `{ "enabled": 0, "entries": [] }`. Save as UTF-8, with or without a final newline.

Each entry uses an exact class name from the table above, x/z map coordinates in metres, and yaw in degrees from **-360 to 360**. Both coordinates must be greater than 0 and less than the map size. Missing coordinates read as 0 and are rejected; omitted yaw reads as 0. The open Black Hawk spawns only when explicitly listed. Base, test and unrelated classes are rejected. A bad entry logs its row number and does not block later rows. Broken JSON structure or invalid top-level fields stop the pass and leave the file unchanged. A failure to write a missing file also stops placement.

Any `RotorCraft_HeliBase` within **14 m** suppresses placement, regardless of class or condition. This includes saved aircraft, wrecks and aircraft placed by earlier entries. It is a position check, not a global fleet limit: moving an aircraft away allows another at that position on the next restart. Keep spawn positions well separated and use only one spawning system for each population.

Placement uses each aircraft's rest height plus 0.05 m to settle. Conservative clearance checks reject water, slopes over 20 degrees, terrain varying by more than 0.5 m across the checked area, and obstructing geometry. Clearance extends 5 / 7 / 10 / 16 m each side of the origin for Mosquito / MH-6 / Gazelle / Black Hawk, rotated with yaw, and checks 50 m overhead for roofs. It can reject a tight location where an aircraft might fit. Inspect every new position in a client. `[RCESPAWN]` logs `placed`, `already there`, `skipped` with a reason, or `bad entry`.

**Respawn is at server start only:** an aircraft that is destroyed and removed returns at the next startup if its configured position is clear. A wreck still within 14 m blocks replacement. There is no timed respawn or retry during that session.

### Manual placement and economy files (advanced)

An admin spawner can create the exact class as a persistent vehicle. Leave automatic attachments off for the standard equipment above. Disable JSON placement for populations managed by CE or another recurring spawner.

Manual spawning does not require an aircraft `types.xml` entry for the script's lifetime renewal. For explicit economy configuration, merge [server-config/types.xml](server-config/types.xml) into the existing `<types>` root of `mpmissions/<mission>/db/types.xml`. These entries have nominal/min **0**, lifetime **3888000**, restock **0**, quantity limits **-1**, cost **100**, and count only in the map. They do not spawn ordinary loot or create vehicle events.

For automatic spawning, also merge vehicle entries into `db/events.xml` and matching positions into the mission root's `cfgeventspawns.xml`. Event names must match exactly; child types must be aircraft class names. Keep vehicle persistence enabled in `db/economy.xml` (`init`, `load`, `respawn`, `save` all `1`) and the existing `CarScript` root with `act="car"` in `cfgeconomycore.xml`. No extra root or aircraft `cfgspawnabletypes.xml` entry is needed for bare spawning.

The supplied [CE examples](server-config/README.md) are map-specific fragments, not an automatic map-wide distribution. Survey positions for your map. Keep events disabled for retained manually spawned aircraft until their counts are reconciled. Copying XML into the mod folder does not register it. For `RotorCraft_BlackHawkOpen`, add a separate type entry using the same lifetime/count fields and a corresponding event only if wanted.

Part loot uses [types_custom_items.xml](types_custom_items.xml) and [cfgspawnabletypes_custom_items.xml](cfgspawnabletypes_custom_items.xml), merged into the corresponding mission files. These do not attach parts to aircraft. Named blade sets have nominal 3/min 1 each, independent of the parts settings; universal blades and keys default to nominal/min 0. Include blade loot even when `partsBladesRequired=0`. Add `RotorCraft_RotorBlades_Mosquito`, `RotorCraft_RotorBlades_MH6`, `RotorCraft_RotorBlades_Gazelle` and `RotorCraft_RotorBlades_BlackHawk`. Without matching blade sets, a ruined rotor means a wrecked aircraft: a blowtorch cannot recover it.

### Lifetime and restarts

Healthy aircraft set their lifetime and maximum lifetime to **3,888,000 seconds (45 days)**, then renew every **600 seconds** on the server. This is not a 45-day inactivity deletion rule. Ruined aircraft instead use `wreckLifetime`, default **1800 seconds**; the script updates the remaining lifetime every 20 seconds and leaves final removal to CE. With `wreckLifetime=0`, CE's ruined-object cleanup applies. JSON placement replaces missing aircraft at startup as described above; it does not restore a wreck or its inventory.

Keep normal mission world storage and vehicle saving enabled. Aircraft save their fuel litres and key state; vanilla storage carries inventory and damage. Loaded aircraft receive no replacement equipment or new key. Current storage reads its versioned format and the earlier tagged fuel format; it rejects unknown or truncated framing. This does not guarantee compatibility with every older save or a rollback. Back up world storage and profiles together. Land and stop aircraft before a planned restart.

## 4. Settings

`$profile:` means the server profile directory selected by `-profiles`, not the mod folder. Files are loaded once per process when first needed; missing files are written then, usually when aircraft are created. An empty world may not create all files immediately. Stop the server before editing, keep a backup, use valid JSON, and restart. There is no live reload.

| File | Where it is read | Missing or unreadable file |
|---|---|---|
| `$profile:RotorCraft_Aircraft.json` | Dedicated server; handling and relevant flags are sent to clients | Missing: writes defaults. Unreadable: uses defaults, leaves file intact. Listen servers use defaults. |
| `$profile:RotorCraft_Damage.json` | Damage decisions run on server; resulting state is synchronized | Missing: writes defaults. Unreadable: uses defaults, including damage, rotor damage and explosions on. |
| `$profile:RotorCraft_Keys.json` | Server; lock settings/state are synchronized | Missing: writes defaults. Unreadable: keys remain disabled. |
| `$profile:RotorCraft_Spawns.json` | Server, about 20 seconds after mission start | Missing: writes enabled Nasdara positions or an empty list for other maps. Unreadable: skips placement and leaves the file intact. |

These are the four administrator JSON files. Clients do not need copies of the server profile files. Flight runs on the server and controlling client; editing only a client's local settings is not a server configuration method.

### Damage presets

Set `difficulty` in `RotorCraft_Damage.json` to one of these names (case-insensitive):

| Preset | toughness | crewScale | bleedEnabled | bleedScale | legBreakSeverity |
|---|---:|---:|---:|---:|---:|
| `hardcore` | 1 | 1 | 1 | 1.5 | 0.6 |
| `realistic` | 1.5 | 1 | 1 | 1 | 0.8 |
| `standard` (default) | 2 | 0.8 | 1 | 0.7 | 0.9 |
| `forgiving` | 3 | 0.5 | 0 | 0.7 | 0 |

The generated file writes **-1** for these five numeric fields. A negative value selects the preset; a nonnegative value overrides it, subject to the ranges below. To change difficulty as a group, keep all five at -1. An unknown name uses standard and logs a warning. `toughness=0` is invalid and falls back to standard's 2.

`toughness` multiplies sink, hit, slide and rollover speed thresholds, not angles. Higher values require harder impacts. `crewScale=0` removes scaled nonfatal health/shock loss; it does not prevent death at fatal severity or disable bleeding, leg injuries or explosions.

### RotorCraft_Damage.json

Ranges are inclusive. Invalid numeric values fall back to defaults, rather than being clamped. Use 0/1 for switches; damage switches generally treat any nonzero value as on and do not validate it as a Boolean.

| Setting | Default in new file | Valid value / effect |
|---|---:|---|
| `enabled` | 1 | 0/1; master switch for scripted impacts, rotor strikes/kills, explosions, water and degradation |
| `difficulty` | `"standard"` | Names above |
| `toughness` | -1 (effective 2) | Negative: preset; otherwise 0.1–10 |
| `crewScale` | -1 (effective 0.8) | Negative: preset; otherwise 0–100 |
| `airframeScale` | 1 | 0–100; structural damage multiplier |
| `crewHealthMin` | 2 | 0–100; health loss just above a safe threshold, before scale |
| `crewHealthMax` | 90 | 0–100; health loss just below fatal, before scale |
| `crewShockMin` | 20 | 0–200; minimum shock loss before scale |
| `crewShockMax` | 150 | 0–200; maximum shock loss before scale |
| `windowRepairRuined` | 1 | 0/1; special blowtorch window repair |
| `rotorEnabled` | 1 | 0/1; crash, tip and obstacle damage to rotors |
| `rotorKills` | 0 | 0/1; turning rotor kills bodies it reaches; also needs `enabled` and `rotorEnabled` |
| `rotorMinSpeed` | 0.2 | 0.01–1; fraction of full rotor speed required for strikes |
| `rotorHardShare` | 0.5 | 0–1; rotor health fraction lost per hard strike at full speed |
| `rotorTreeShare` | 0.3 | 0–1; same, for trees |
| `tipStoppedShare` | 0.25 | 0–1; main-rotor health fraction lost in a stopped tip-over |
| `explodeEnabled` | 1 | 0/1; explosion on destruction with sufficient fuel; also enables outright destruction at `explodeSeverity` |
| `explodeAmmo` | `"LandFuelFeed_Ammo"` | Default, or a valid explosive `CfgAmmo` class with `DamageApplied` and positive `indirectHitRange`; identifier max 128 characters |
| `explodeMinFuel` | 5 | 0–100000 L; minimum fuel for explosion |
| `explodeSeverity` | 1.4 | 0.1–100; impact severity for outright destruction (1 is fatal to crew) |
| `burnEnabled` | 1 | 0/1; a ruined fuel tank or Engine with fuel starts a burn while `enabled=1` |
| `burnSeconds` | 60 | 1–3600 s; burn duration before destruction, unless further damage finishes it sooner |
| `waterEnabled` | 0 | 0/1; water stops engine and eventually destroys aircraft |
| `waterSeconds` | 90 | 1–100000 s in water before destruction |
| `waterSink` | 0 | 0–10; additional downward force as a fraction of weight |
| `wreckLifetime` | 1800 | 0–3888000 s; 0 leaves cleanup to CE |
| `bleedEnabled` | -1 (effective 1) | Negative: preset; otherwise 0/1 |
| `bleedFrom` | 0.5 | 0–1 severity; bleeding starts here |
| `bleedScale` | -1 (effective 0.7) | Negative: preset; otherwise 0–10; wound-count multiplier |
| `legBreakSeverity` | -1 (effective 0.9) | Negative: preset; otherwise 0–1; 0 disables leg breaks |
| `degradeEnabled` | 0 | 0/1; engine weakness, rotor shake, tank leaks and extra hull drag |
| `degradeFrom` | 0.6 | 0.01–1; zone health fraction below which degradation starts |
| `engineWeakest` | 0.6 | 0–1; thrust fraction at worst engine damage |
| `rotorShake` | 0.4 | 0–1 rad/s; worst rotor shake |
| `tankLeakWorst` | 0.1 | 0–10 tank capacities per minute at worst tank damage |
| `hullDragWorst` | 0.5 | 0–1; extra drag relative to normal forward drag |
| `tankRuinsAircraft` | 0 | Legacy field; ignored |
| `aircraft` | Four rows below | Array of per-class limits |
| `help` | Generated explanatory text | Informational string; no gameplay effect |

`enabled=0` does not make aircraft invulnerable: vanilla gunfire/zone damage still applies. Wreck lifetime, special window repair and rotor-health-to-flight-power synchronization remain active. A ruined tank prevents starting with the normal failed-start sound; boarding never ruins the engine. This rule applies with any damage settings.

Rotor damage and explosions are **on by default**. To disable the scripted rotor strikes and crash/tip rotor damage, set `rotorEnabled=0`; to disable blasts, set `explodeEnabled=0`. To enable rotor kills, set `rotorKills=1` while keeping `enabled=1` and `rotorEnabled=1`. Seated crew and boarding/exiting crew have exclusions. The Gazelle's shrouded tail rotor uses a narrower body-contact band; it is not harmless. Rotor health affects lift and pedal authority even with degradation off. Zero working share in either rotor stops the engine once on the server, regardless of these switches. Tail failure preserves main-rotor autorotation; main failure provides no lift. Fit a non-ruined matching set in `RotorBlades` to restore destroyed rotors.

An explosion kills living occupants, consumes the remaining fuel and uses the configured blast ammo. The fuel threshold applies to the blast, not to whether the aircraft can be ruined. `waterSink=0` means no extra scripted sinking force, not flotation. Window repair restores ruined glass to 40% health, then another pass restores it fully. Other ordinary repairs stop at worn.

A burning aircraft loses global health over `burnSeconds`, with flames and heavy smoke visible to late joiners. Fire itself does not damage crew or bystanders; the final blast uses the existing explosion settings and fuel minimum. With no fuel, ruined tanks and engines only smoke. Emptying the tank stops the burn. The burn does not consume extra fuel; normal fuel use and configured leaks still apply. A restart starts a new countdown from saved health and fuel. The separate 60-second fire after an explosion is unchanged. Body paint stays normal until global destruction; glass stages and repair limits are unchanged.

An existing damage file that omits `burnEnabled` reads it as **0**, so files written before this setting keep burning off. Omitted or invalid `burnSeconds` falls back to 60. Regenerate settings after updating as described below, then reapply intended changes.

Each damage `aircraft` row has these fields. Defaults below are **before toughness** and assume default handling:

| Field | Mosquito | MH-6 | Gazelle | Black Hawk | Valid range |
|---|---:|---:|---:|---:|---|
| `sinkSafe` | 4.578 | 5.7225 | 5.7225 | 6.75 | 1–50 m/s |
| `sinkFatal` | 9 | 10.5 | 11 | 13 | 1.5–200 m/s |
| `hitSafe` | 4 | 5 | 5 | 6 | 1–50 m/s lost against an object |
| `hitFatal` | 14 | 16 | 17 | 20 | 1.5–200 m/s |
| `slideSafe` | 18 | 20 | 20 | 22 | 1–100 m/s lost on ground/floor |
| `slideFatal` | 40 | 45 | 45 | 50 | 1.5–300 m/s |
| `tiltSafe` | 30 | 30 | 30 | 35 | 5–90 degrees from upright |
| `tiltRollover` | 60 | 60 | 60 | 65 | 10–170 degrees |
| `tiltFatal` | 150 | 150 | 150 | 150 | 15–180 degrees |
| `rollSafe` | 6 | 6 | 6 | 7 | 0.5–100 m/s rollover motion |
| `rollFatal` | 14 | 16 | 17 | 20 | 1–300 m/s |
| `airframeAtFatal` | 1 | 0.8 | 0.8 | 0.6 | 0–5; zone damage fraction before scale |
| `mainRadius` | 3.33 | 4.04 | 5.25 | 8.24 | 0.5–30 m |
| `tailRadius` | 0.50 | 0.80 | 0.35 | 1.35 | 0.1–10 m |
| `className` | `RotorCraft_Mosquito` | `RotorCraft_MH6` | `RotorCraft_Gazelle` | `RotorCraft_BlackHawk` | Exact aircraft class |
| `why` | Explanatory text | Explanatory text | Explanatory text | Explanatory text | Informational string |

Fatal speed limits must exceed their safe limits; angles must rise `tiltSafe < tiltRollover < tiltFatal`. Invalid ordering resets the affected group to defaults. For the first three aircraft, default `sinkSafe` is the greater of the base value (4 / 4.5 / 4.5) and `1.05 * 0.981 / (sinkDamping * 0.9)`, capped at `sinkFatal - 2`. Nonpositive `sinkDamping` uses the base value. Existing saved JSON values do not recalculate. At standard difficulty the table's speed limits double.

### RotorCraft_Aircraft.json

Keep complete aircraft rows unless deliberately changing handling. Top-level settings:

| Setting | Default | Valid value / effect |
|---|---|---|
| `partsBeltRequired` | 0 | 0/1; only 1 requires a healthy Mosquito drive belt |
| `partsBladesRequired` | 0 | 0/1; only 1 requires a healthy matching blade set |
| `partsUniversalBlades` | 0 | 0/1; only 1 permits the universal blade item |
| `debugLogging` | false | JSON `true`/`false`; diagnostic logs |
| `aircraft` | Four rows below | Recognized class names only; missing whole rows retain defaults |
| `help` | Generated explanatory text | Informational string |

The following are each row's defaults and accepted ranges. Out-of-range values revert to that aircraft's default.

| Field | Mosquito | MH-6 | Gazelle | Black Hawk | Valid range / units |
|---|---:|---:|---:|---:|---|
| `bodyMass` | 230 | 976 | 1700 | 8800 | 0–100000 kg; 0 keeps model mass |
| `ceiling` | 2100 | 5319 | 5000 | 5800 | 500–15000 m ASL |
| `rotorDiameter` | 5.95 | 5.6 | 10.5 | 16.36 | 1–40 m; ground-effect scale |
| `climbAccel` | 6 | 8 | 7.2 | 3.8 | 0.5–30 m/s² |
| `heaveDamping` | 1.2 | 0.8 | 0.85 | 0.45 | 0.05–5 /s |
| `sinkDamping` | 0.25 | 0.2 | 0.2 | 0.12 | 0–5 /s |
| `dragForward` | 0.0016 | 0.0003 | 0.00016 | 0.00025 | 0–0.1; acceleration per speed squared |
| `dragBack` | 0.02 | 0.0132 | 0.0132 | 0.0132 | 0–0.1; same units |
| `dragSide` | 0.007 | 0.0025 | 0.0015 | 0.0012 | 0–0.1; same units |
| `dragTop` | 0.004 | 0.0008 | 0.0006 | 0.0006 | 0–0.1; same units |
| `dragDisc` | 0.004 | 0.00232 | 0.002 | 0.0016 | 0–0.1; same units |
| `neverExceed` | 40 | 78 | 86 | 99 | 5–300 m/s; increasing rotor drag above this |
| `pitchRate` | 0.95 | 0.70 | 0.64 | 0.32 | 0.05–5 rad/s |
| `rollRate` | 1.15 | 0.87 | 0.82 | 0.42 | 0.05–5 rad/s |
| `yawRate` | 1.35 | 1.05 | 1 | 0.55 | 0.05–5 rad/s |
| `rateResponse` | 7 | 4 | 3.4 | 1.1 | 0.1–30 /s |
| `maxTiltSin` | 0 | 0 | 0 | 0 | 0–0.95; sine of tilt limit, 0 unlimited |
| `levelGain` | 3 | 2 | 2 | 1 | 0.1–10 /s; leveling response |
| `rotorInertia` | 5 | 8 | 9 | 12 | 0.5–60 s |
| `autoSink` | 6.3 | 10.2 | 9 | 13 | 1–40 m/s; autorotation airflow reference |
| `canPush` | 1 | 1 | 0 | 0 | 0/1 |
| `pushText` | `Push helicopter` | `Push helicopter` | `Push helicopter` | `Push helicopter` | 1–128 characters; no pipe or line breaks |
| `pushCoef` | 0.35 | 1 | 1 | 1 | 0–20 times vanilla shove |
| `burnPerHour` | 20 | 120 | 180 | 550 | 0–100000 L/h before pacing; 0 disables scripted burn |
| `recoveryEnabled` | 1 | 1 | 1 | 1 | 0/1; powered recovery |
| `recoveryCooldown` | 30 | 30 | 30 | 30 | 0–86400 s |
| `cameraDistance` | 7 | 9 | 11 | 16 | 1–40 m |
| `cameraHeight` | 1.5 | 1.5 | 2 | 2.5 | -5–15 m from origin |
| `cameraAft` | 0 | 0 | 2 | 5 | -5–20 m aft of origin |
| `canFlip` | 1 | 1 | 1 | 1 | 0/1 |
| `flipSeconds` | 5 | 8 | 10 | 15 | 0.5–600 s; synchronized as an integer, use whole seconds |
| `flipNeedsTool` | 1 | 1 | 1 | 1 | 0/1 |
| `knockoutBleed` | 1 | 1 | 1 | 1 | 0.1–20 collective notches/s while unconscious airborne |
| `copilotTakeover` | 0 | 0 | 0 | 0 | 0/1; no effect on single-seat Mosquito |
| `className` | `RotorCraft_Mosquito` | `RotorCraft_MH6` | `RotorCraft_Gazelle` | `RotorCraft_BlackHawk` | Exact recognized class |
| `source` | Explanatory text | Explanatory text | Explanatory text | Explanatory text | Informational string |

Fuel burn uses a fixed 3× pace: full lever burns 60 / 360 / 540 / 1650 L/h respectively; idle is one third of that. Fresh fuel is fixed at 25%. Engine shutdown after 120 seconds without a living flight pilot is also fixed. These are not JSON settings.

**Flip:** hold a nonruined `Wrench` or `LugWrench`, aim at the aircraft and hold the displayed action. A successful flip costs 8 tool health. It requires at least 60° tilt, speed at most 0.5 m/s, no living occupant, a nonruined aircraft, and either engine off or rotor speed at most 2%. Dead occupants may remain. Ground pitch/roll must each be at most 20°, with space for the upright hull. Bushes and small vegetation are ignored; trees, rocks and solid obstacles can block it. `flipNeedsTool=0` offers an empty-hands interaction instead. Flipping does not repair damage.

**Required parts:** `RotorCraft_DriveBelt` fits only the Mosquito. Named blade sets are `RotorCraft_RotorBlades_Mosquito`, `_MH6`, `_Gazelle` and `_BlackHawk`; each fits its own aircraft, including the open Black Hawk for `_BlackHawk`. `partsUniversalBlades=1` additionally permits `RotorCraft_RotorBlades` on all aircraft. A universal set does nothing on a server with this setting off; the default remains 0. Named sets are not interchangeable. Fit parts outside with the engine stopped; blade sets also require stopped rotors. Blade sets are carried in hands and cannot go in cargo. Missing, wrong or ruined required parts prevent starting; the optional requirements do not stop an engine already in flight. Belts and blade attachments have no running wear, but rotor destruction ruins a fitted blade set.

**Rotor replacement:** stop the engine and wait for the blades to stop. Remove the ruined set if present, then drag a non-ruined matching set from the ground or hands into `RotorBlades`. The server restores every rotor at zero working share to full health; one set covers both and stays fitted. Above-zero working share is unchanged and keeps ordinary torch repair. Zero working share ruins the fitted set once, with a `[RCEROTOR]` log; replacement logs each restored zone. Loading a saved attachment never repairs rotors: a dead rotor with a healthy saved set leaves that set ruined. Condition rows and working percentages synchronize.

With `partsBladesRequired=0`, the slot accepts a set only while at least one rotor is destroyed, and healthy fitted sets cannot be removed. With `partsBladesRequired=1`, healthy sets can be fitted and removed; the aircraft cannot start without one. Ruined sets can always be removed once the engine and rotors stop. These rules apply on client and server. Drive belts and other attachments retain their existing rules. Wrenches and lug wrenches retain the flip action.

**Co-pilot:** `copilotTakeover=1` enables automatic seat-1 control on MH-6, Gazelle and Black Hawk only while a dead body remains in seat 0. An empty, disconnected-but-living or unconscious pilot does not trigger takeover. A connected, living co-pilot must remain attached to the aircraft; no seat swap occurs. Removing the dead pilot ends the takeover condition. A living co-pilot prevents the empty-pilot shutdown. This option is experimental; leave it at 0 unless needed.

### RotorCraft_Keys.json

| Setting | Default | Valid value / effect |
|---|---:|---|
| `keysEnabled` | 0 | 0/1; only 1 enables ignition locks |
| `keyMode` | 0 | 0: vanilla colour groups; 1: unique aircraft key |
| `keySpawnInCargo` | 1 | 0/1; only 1 creates a matching key on fresh unique-mode aircraft |

Locks block ignition, not boarding or cargo access. Unlocked aircraft need no key to start. Hold a healthy matching key and use the two-second lock/unlock action while outside a grounded, stopped aircraft.

Colour mode uses `ShippingContainerKeys_Blue`, `_Yellow`, `_Orange`, `_Red` and `_Green`. Any healthy key can lock an unlocked aircraft to its colour; any key of that colour can unlock it. Unlocking applies 6 absolute item damage. No colour key is automatically supplied.

Unique mode uses `RotorCraft_AircraftKey`. Fresh aircraft pair to their persistent ID, start unlocked and receive one matching cargo key if enabled. Each successful lock or unlock consumes 1% of maximum key health and increments saved wear. A key that ruins while locking can leave the aircraft locked. Blank, ruined and wrong keys do not work. There is no key cutting, copying or lost-key recovery action.

Restored aircraft get no replacement key. Old aircraft without a saved key ID remain unpaired and cannot start in unique mode; use colour mode or disable keys for those vehicles. Full cargo prevents key creation without a retry or ground drop. Failure to obtain a persistent ID also blocks unique ignition. Disabling keys preserves existing IDs and lock states.

### Older settings files

A missing numeric field in an existing JSON file reads as **0**, not necessarily the initializer shown in the source. Do not assume omission means “use the default.” After an update, stop the server, **keep a copy**, remove or rename the old settings files, let the new build generate complete files, then stop it again and reapply intended changes. Keep generated rows complete.

| File / fields | Omitted-field behavior |
|---|---|
| Aircraft: missing whole class row | Keeps that class's full defaults |
| Aircraft: fields whose table range starts above 0 | Zero fails validation and falls back; includes `knockoutBleed` and `flipSeconds` |
| Aircraft: flip switches | If `flipSeconds <= 0`, both `canFlip` and `flipNeedsTool` revert to defaults; if a valid time exists but a switch is omitted, that switch remains 0 |
| Aircraft: `bodyMass`, `sinkDamping`, all five drag fields, `maxTiltSin`, `pushCoef`, `burnPerHour`, `recoveryCooldown`, `cameraHeight`, `cameraAft`, `canPush`, `recoveryEnabled` | Zero is accepted; several differ materially from defaults |
| Aircraft: `copilotTakeover`, three top-level part switches, `debugLogging` | Zero/false matches their disabled default |
| Aircraft: empty `pushText` / `source` | Replaced by default text |
| Damage: positive-minimum fields | Zero fails validation: `toughness`, `rotorMinSpeed`, `explodeSeverity`, `burnSeconds`, `waterSeconds`, `degradeFrom`, and per-aircraft limits/radii except `airframeAtFatal` |
| Damage: `burnEnabled` | Omitted zero keeps burning off, including files written before this setting; new files default to 1 |
| Damage: switches, zero-allowed scales/thresholds, `wreckLifetime`, `airframeAtFatal` | Zero is accepted; it can disable effects or change cleanup/injury behavior |
| Damage: preset-controlled fields | Omitted zero does not select a preset. Explicitly use -1 for all five; omitted `toughness` alone falls back to standard's 2 |
| Damage: missing `difficulty` / `explodeAmmo` | Empty difficulty uses standard; empty explosion ammo fails validation and skips the blast |
| Keys: all fields | Zero; keys off, colour mode, no cargo key. The last differs from the new-file default |

Missing damage rows retain default limits; a missing/null damage array also restores default limits, but does not restore omitted top-level settings. A missing/null aircraft array retains default handling while still applying top-level part/log settings. Neither loader rewrites a readable old file to add new fields.

## 5. Manage the server

Before updating, stop the server and back up the active mission, world storage and profiles. Replace the complete mod package, including signatures and its public key if changed. Update every client to the same package. Remove obsolete duplicate mod folders from launch arguments. Regenerate old settings as described above, then check startup logs, joining, spawning and a save/restart before reopening normal play.

After a DayZ patch, match the server and client game builds and use CF and RotorCraft Evolution versions compatible with that patch. The supported target here is 1.30; do not assume another game version is compatible. Validate on a separate copy of the mission and storage before updating a live world. A rollback needs the matching pre-update storage backup as well as the old mod files.

Read the server profile's `script_*.log`, RPT and crash logs; use the client's script/RPT logs for prompts, inputs and rendering. Client logs normally live under `%LOCALAPPDATA%\DayZ` on Windows. CE errors also appear in server economy/RPT output.

| Prefix | Useful evidence |
|---|---|
| `[RCEDATA]` | Aircraft file loading, rejected fields and handling overrides |
| `[RCESPAWN]` | Startup placement, nearby aircraft, rejected entries and obstructed positions |
| `[RCEDMG]` | Preset resolution, effective limits, contacts and crew/airframe injuries |
| `[RCELIFE]` | Creation/load, minute heartbeat, pilot departure, kill and deletion; position and lifetime |
| `[RCESTORE]` | Save/load rejection and framing problems |
| `[RCEENGINE]`, `[RCEPARTS]` | Engine events, missing/ruined parts and empty-pilot shutdown |
| `[RCEFLIP]` | Prompt checks, action start/end and refusal reasons on server/client |
| `[RCEKEY]` | Settings, missing persistent ID or no room for a matching key |
| `[RCEROTOR]`, `[RCEHIT]`, `[RCEZONE]` | Rotor strikes, projectile hits and damage-zone changes |
| `[RCEWRECK]`, `[RCEDEGRADE]` | Destruction, explosion decision, wreck retention and degradation |
| `[RCELOG]`, `[RCEGAUGE]`, `[RCEPFD]`, `[RCEINPUT]` | Flight, instruments and controls; verbose diagnostic lines are gated |
| `[RCEWATER]`, `[RCECOPILOT]`, `[RCESEAT]`, `[RCETAB]` | Water, takeover and boarding diagnostics; gated |

To enable diagnostics, set top-level `"debugLogging": true` in `RotorCraft_Aircraft.json` and restart. Alternatively add `-flyDebugLog` to the server launch command; it overrides a false setting or unreadable file. To turn them off, set false **and remove that flag**, then restart. Clients receive the flag from the server. Lifetime, errors and many event logs remain enabled. See [LOGGING_IDLE.md](LOGGING_IDLE.md) for log details.

Use normal orderly server restarts. Ask pilots to land and stop first. Keep the same mission storage path; neither a restart nor settings regeneration should require a world wipe. Do not re-enable one-time spawn blocks after restarting.

## 6. Troubleshooting

Flight bindings are under **RotorCraft Evolution**: Up arrow sets lever +10, Right arrow sets 0, Down arrow sets -10, and Left arrow activates recovery. Existing player presets may retain X; rebind recovery in Controls. These are client bindings; the server reads the pilot's synced input. No server key configuration is needed.

During ground spin-up the lever stops at centre until 100% rotor speed, with **ROTOR SPINNING UP** on the HUD. Airborne flight and scripted flight-test profiles are exempt. A zero-working-share rotor refuses starting and displays **MAIN ROTOR DESTROYED: FIT BLADE SET** or **TAIL ROTOR DESTROYED: FIT BLADE SET**; its condition row reads Ruined and requires replacement even when zone health is above zero. Battery or plug failure gives one click; fuel failure keeps the existing failed-start sound. Verify these paths with a client after updating.


| Symptom | Check / action | Log or message to look for |
|---|---|---|
| “Client is a newer build than the server” | Compare DayZ executable versions first, including stable/experimental channel. Update the actual server installation being launched. If the message names a mod/PBO, synchronize that package instead. Restart after updating. | `0x00020013 - BUILD_GREATER` in Bohemia's [error-code reference](https://community.bistudio.com/wiki/DayZ:Error_Codes); version lines in both RPTs |
| Signature or mod mismatch kick | Check the top-level server `keys`, matching PBO/signature pair, client/server package versions and duplicate loaded folders. Keep signature verification on. Check Linux case. | Kick text containing `signature`, the named `.pbo` / `.bisign`, or missing-mod message |
| No aircraft spawn | Wait at least 20 seconds after mission start. Check `RotorCraft_Spawns.json`, `enabled`, map positions, nearby aircraft and clearance. Other maps have no default positions. For CE, check active events, counts and player avoidance. | `[RCESPAWN]` gives placement or refusal reasons; `[RCELIFE] ... created` confirms creation; CE/RPT errors identify invalid classes/XML |
| Flip prompt appears briefly, then stops | Stand upright, stop moving, keep a healthy wrench in hand and remain within reach. Check tilt, speed, live occupants, ground slope and solid obstructions. The server rechecks the flip at completion. | `[RCEFLIP] ... check:`, `checks:`, `setup:`, `server hold started`, `refused:` and the stated reason, such as `still moving`, `no room` or `ground too steep` |
| Aircraft disappears while unattended | Determine whether it was ruined, cleaned up, moved or failed to reload. Inspect lifetime and position before deletion. Confirm the intended build is loaded and storage paths are unchanged. Do not wipe storage to diagnose it. | `[RCELIFE] ... still here`, `killed by`, `deleted`; `[RCEWRECK] ... stays`; `[RCESTORE] ... rejected saved data`; `Scripted variables corrupted upon RotorCraft_...` |
| Server will not start | Read the first script/config error, check package completeness and CF/load order, then check mission syntax. Restore the last compatible package and its matching backup if needed. A malformed settings JSON normally logs a fallback, not a successful custom configuration. | `Can't compile` with module/file/line, `Undefined function`, `Cannot open file`, missing-addon/config errors; `[RCEDATA] can't read` or `[RCEDMG] settings: can't read` for JSON |

## 7. Known limits

Aircraft use seated vehicle animations. Some seat changes and exits can use car-style movement or clip the model. There is no standing or walking inside an aircraft in flight; use a seat. Getting out in flight is not a safe transport method.

Seat 0 normally flies. Optional co-pilot takeover has the narrow dead-pilot condition above. Ground seat changes allow empty seats; airborne changes exclude the pilot seat, and MH-6 seat changes are ground-only. See [PLAYER_GUIDE.md](PLAYER_GUIDE.md) and [KEYBINDINGS.md](KEYBINDINGS.md) for basic operation; the settings and defaults in this administrator guide take precedence over older feature summaries there.

Prototype features such as custom paint/logo packs, spray-can repainting, door guns and key cutting/copying are not included. They have no supported install or settings steps in this package.

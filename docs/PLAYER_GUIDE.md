# RotorCraft Evolution: player guide

DayZ 1.30. Server settings can change handling and damage. Default keys are in [KEYBINDINGS.md](KEYBINDINGS.md).

## Start and fly

1. Aim at a seat and hold **F**. Only seat 0 controls flight.
2. Hold **left mouse** to start. Allow about 25 seconds for full rotor speed. **ROTOR SPINNING UP** means the lever cannot rise above centre on the ground; lowering still works.
3. Use the wheel to adjust the lever, or the **Right arrow** for 0, **Up arrow** for +10 and **Down arrow** for -10. Raise above centre after rotor speed reaches 100% to lift off.
4. **W/S** pitches, **A/D** banks, **Q/E** turns the nose. Pitch and bank stay when released.
5. The **Left arrow** levels, centres the lever and brakes drift and descent for about two seconds. It needs engine power and enough rotor speed. Default cooldown: 30 seconds; during cooldown it only levels and centres.

The lever has 21 positions. Centre is approximately a level hover; altitude, tilt and airflow change that. There is no default attitude limit. Steep banks and inverted flight lose height. Recovery also needs height.

## Aircraft

| Aircraft | Seats | Required parts | Tank | Fresh fuel | Cargo | Push by default |
|---|---:|---|---:|---:|---|---|
| Mosquito XE | 1 | Car battery, two spark plugs | 40 L | 10 L | 4 x 4 | Yes |
| MH-6 Little Bird | 4 | Truck battery, glow plug | 240 L | 60 L | 10 x 10 | Yes |
| SA342M Gazelle | 4 | Truck battery, glow plug | 445 L | 111.25 L | 10 x 10 | No |
| UH-60M Black Hawk | 12 | Truck battery, glow plug | 1360 L | 340 L | 10 x 60 | No |

A fresh bare aircraft receives parts and a blowtorch with gas. Admin spawners that attach parts first can skip that equipment step. All four start with a quarter tank in script. Fit parts from outside with the engine off. No radiator, drive belt or aircraft key is required.

A destroyed main or tail rotor stops the engine and blocks starting. The HUD reads **MAIN ROTOR DESTROYED: FIT BLADE SET** or **TAIL ROTOR DESTROYED: FIT BLADE SET**; its condition row reads **Ruined**. A blowtorch cannot repair it. Missing or ruined plugs, or a missing, ruined or discharged battery, give one starter click without spin-up. Fuel failures retain the failed-start sound.

## Land and stop

Slow, level, and lower the lever a little. Watch descent and rotor speed; a fast powered descent can lose lift. Stop after landing with **left click**. The rotor coasts down over roughly 55 seconds.

The stop prompt was absent in earlier airborne client tests. However, vanilla's action checks speed (at most 8 km/h), not ground contact. In-flight stopping during a slow hover is unconfirmed. Avoid left click in flight.

Holding **F** can exit in flight. An empty pilot seat leaves the engine running for up to two minutes. There is no co-pilot takeover.

## Engine out

Autorotation is implemented and has earlier server-only landing tests. Lower the lever promptly to preserve rotor speed, maintain forward speed, then flare and raise the lever near touchdown. Pulling early spends the rotor's energy. Engine-out, the **Left arrow** only levels; it does not stop the fall or centre the lever. The ground spin-up restriction does not apply in flight, including airborne restarts. Hand-flown engine-out landings remain unconfirmed.

A destroyed tail rotor leaves the main rotor available for autorotation, with no tail-rotor authority. A destroyed main rotor cannot provide lift or recover by autorotation.

## Seats, fuel and lights

On the ground, any seat can move to any empty seat. In the air, nobody moves into or out of the pilot seat through the change-seat action. Gazelle and Black Hawk passengers can change non-pilot seats; MH-6 seat changes are ground-only. Aim at the destination seat. Earlier client seat changes passed; poses and every exit still need release checks.

Refuel with gasoline in a held container, aimed at the fuel cap from within 2.5 m. A gas can is client-confirmed. Pots, cauldrons, bottles and canteens have server checks; a carried closed barrel needs a client check. A barrel on the ground is unsupported. A full tank still offers the action, stops after about a second and should consume no fuel.

Default full-lever burn is 60 / 360 / 540 / 1650 L/h (Mosquito / MH-6 / Gazelle / Black Hawk); idle is about one third, centre about two thirds. These are code-derived rates, with 3x game pacing. Cargo does not change burn.

**L** switches exterior lights. The landing light is active below 60 m when lights are on. Dim panel illumination remains with exterior lights off. Earlier night tests passed; verify on the release build.

## Gauges and damage

Pilot HUD: speed, height above ground, altitude above sea level, lever, rotor speed, fuel and horizon. Cockpit needles work; the MH-6 has no moving horizon. Black Hawk digital screens show attitude, speed, altitude, heading, NR and TRQ; the centre stack is still static. Altimeter face units are assumed metres. Use HUD fuel percent where the dial scale differs from tank size.

Crashes and hard rollovers can damage the aircraft and injure or kill occupants. Use a blowtorch on repairable zones. Ordinary repairs stop at worn; ruined structural parts are not repairable. Window repair has a special rule enabled in source: ruined to cracked, then whole; full visual restoration needs a client check. Aim at a dead occupant for **Pull out body**.

To replace ruined rotors, stop the engine and wait for the blades to stop. Open the aircraft inventory from outside, remove the ruined set if present, and drag a non-ruined matching set from the ground or your hands into **RotorBlades**. One set restores both ruined rotors to full health and stays fitted. Worn or damaged rotors with working share above zero are unchanged and still take the blowtorch. A rotor reaching zero working share ruins the fitted set. Blade sets are carried in hands and cannot go in cargo.

With `partsBladesRequired` off, the slot accepts a set only for a destroyed rotor; a healthy fitted set stays attached until ruined. With it on, healthy sets can be fitted or removed, but a missing or ruined set prevents starting. Fitting and removal require the engine off and blades stopped. Named sets fit only their aircraft; the Black Hawk set also fits the open version. A universal set does nothing on a server with `partsUniversalBlades` off, which is the default.

## Unconfirmed and coming soon

Two real clients, CE spawning, updates from old saves, key rebinding, gunfire through glass, and maps other than Nasdara still need checks. Gazelle leg clipping, Black Hawk exits and MH-6 seat-change animation need review.

Coming soon, untested: hardcore / realistic / standard (default) / forgiving difficulty levels, flip with a wrench, revised knockout behaviour, 30-minute wreck lifetime, crash bleeding and broken legs. Rotor strikes, explosions, water sinking and degraded flight are experimental settings, off by default.

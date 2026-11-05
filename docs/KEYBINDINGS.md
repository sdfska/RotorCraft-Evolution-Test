# RotorCraft Evolution: key bindings

Defaults from `RotorCraftEvolution/modded_inputs.xml`, under **RotorCraft Evolution**. Existing saved presets may retain X for recovery; rebind it to the Left arrow. Rebinding still needs a client check.

| Control | Input | Default |
|---|---|---|
| Nose down / up | `UARCECyclicForward` / `UARCECyclicBack` | W / S |
| Bank left / right | `UARCECyclicLeft` / `UARCECyclicRight` | A / D |
| Pedal left / right | `UARCEPedalLeft` / `UARCEPedalRight` | Q / E |
| Lever up / down | `UARCECollectiveUp` / `UARCECollectiveDown` | Wheel up / down |
| Lever to +10 | `UARCECollectiveMax` | Up arrow |
| Lever to 0 | `UARCECollectiveCenter` | Right arrow |
| Lever to -10 | `UARCECollectiveMin` | Down arrow |
| Recovery / auto-level | `UARCELevel` | Left arrow |
| Start / stop engine | `UADefaultAction` | Hold left mouse / left click |
| Get in / out | `UAAction` | Hold F |
| Change seat | Vanilla action shown on prompt | Follow prompt |
| Refuel / repair | `UADefaultAction` | Hold left mouse |
| Lights | `UAToggleVehicleLights` | L |
| Inventory | `UAGear` | Tab |
| Look around / camera | `UALookAround` / `UAPersonView` | Left Alt / V |

Only seat 0 flies. Powered recovery centres the lever and levels; braking recovery has a default 30-second cooldown. Engine-out recovery only levels. The lever still works during autorotation. Ground seat changes allow any empty seat; airborne changes exclude the pilot, and the MH-6 has no airborne seat changes.

The Up, Right and Down arrows set the lever instantly. While at the flight controls, the vanilla voice range on the Up and Down arrows is off; menus and passengers keep it. Home and End are avoided because VPP Admin Tools and Community Online Tools use them. During ground spin-up, wheel up and the Up arrow stop at centre until rotor speed reaches 100%; the HUD shows **ROTOR SPINNING UP**. Lowering remains available. Press Page Up again after spin-up to select +10. Airborne restarts and engine-out flight have no spin-up restriction.

Flight input is suppressed while a menu is open; confirm inventory, chat, map and pause in a real client. The pilot's action display is hidden with the engine running, but action keys still work. Vanilla stop-engine checks speed at most 8 km/h, not ground contact: slow-hover left click needs a test. Get-out can work in flight.

WASD car controls are zeroed; Q/E gear inputs are disabled while seated and restored on leaving. Logout while seated still needs a restoration check. Wheel input also overlaps action selection and zoom. Recovery now defaults to End to avoid X's toggle-weapons binding. These conflicts have not been fully checked. L is the exterior-light master switch; dim panel glow remains.

Moving flight controls to other keys leaves the old WASD car inputs suppressed. Held incremental lever inputs repeat every six physics ticks; the three instant settings act once per press. Cyclic and pedals read analog values; incremental lever inputs use a threshold above 0.5. No gamepad defaults are supplied; see [CONTROLLER_NOTES.md](CONTROLLER_NOTES.md) for the proposed layout. Test a saved preset after reconnecting and after changing the mod list. Vanilla horn H may remain available; unconfirmed.

Wrench flipping and revised knockout controls are coming soon / unconfirmed, even where source hooks exist. They are not additional key bindings.

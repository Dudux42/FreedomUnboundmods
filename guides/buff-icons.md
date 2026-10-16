# Buff Icons 0.1.0

Author: DuduX42. Game: MHFU USA (`ULUS10391`) in the updated native launcher.

## Installation and behavior

Drop `build/mods/buff-icons-0.1.0.zip` onto the launcher's **Mods** page,
install it and enable **Buff Icons**. The desktop shortcut uses the rebuilt
native executable, which includes the provider required by this ZIP.

Small colored icons form one row immediately to the right of the sharpness
gauge, aligned with its center. There are no numeric timers or labels.

- More than 30 seconds remaining: normal icon.
- 30 seconds or less: gently pulsating icon.
- 10 seconds or less: faster pulse, alternating between its own color and red.
- Expired or removed effect: icon disappears immediately. All following icons
  move left one slot, leaving no holes.

The row has a stable effect order; refreshing a buff does not shuffle it.
Pulsing changes icon size within its slot without pushing its neighbors.
Coordinates follow the native HUD when the window changes size or aspect.

## Supported effects

Timed item effects appear first:

| Effect | Icon |
|---|---|
| Temporary attack, including seeds/pills/powders | Coral sword |
| Temporary defense, including seeds/pills/powders | Blue shield |
| Power/Mega Juice stamina effect | Yellow lightning |
| Immunizer and fish recovery effect | Green cross |
| Hot Drink/Hot Meat cold protection | Orange flame |
| Cool Drink/Cold Meat heat protection | Cyan snowflake |
| Psychoserum | Purple eye |

The local hunter's 21 native Hunting Horn effect slots are also supported,
following the item icons. Shared-effect melodies use a matching silhouette
with a small note mark; other melodies use colored musical notes. Separate
item and melody durations remain separate icons, so an expiring seed cannot
hide an active attack melody.

Lasting effects such as Demondrug/Armorskin, food/armor skills and negative
statuses are not timed item buffs and do not create these icons. Normal weapon
mechanics such as the Long Sword spirit gauge remain native HUD elements.

Icons clear when the hunter is carted, outside quest phases, on the reward
screen, when disabled, or when the observed data is stale. The game removes
the relevant effects on cart; this mod does not alter that behavior.

## Implementation

`host/buff_icons.cpp` reads the local hunter pointer at `0x099998F4`, validates
the quest overlay and native decrement instructions, and reads these timers:

| Effect | Hunter-relative timer | Native decrement/store |
|---|---|---|
| Attack | `0x3D6` | `0x09A693DC` / `0x09A693E0` |
| Defense | `0x3DA` | `0x09A69400` / `0x09A69404` |
| Stamina | `0x622` | `0x09A6907C` / `0x09A69080` |
| Recovery | `0x628` | `0x09A692C4` / `0x09A692C8` |
| Hot Drink | `0x626` | `0x09A69248` / `0x09A6924C` |
| Cool Drink | `0x624` | `0x09A69160` / `0x09A69164` |
| Psychoserum | `0x62A` | `0x09A69340` / `0x09A69344` |
| Melodies | 21 signed 16-bit values at `0x6A0` | `0x09A69EBC` / `0x09A69EC0` |

The native item handlers at `0x09AAB310` onward establish the item-to-counter
mapping. Item names were cross-checked against the owner's local string table.
Attack/defense additionally require a positive modifier at `0x3D4`/`0x3D8`.
Melodies require their matching active-effect byte at `0x5F7 + index`.
The native melody loop bound at `0x09A6A008` is 21.

The counters tick at native 30 Hz. Hot Drink can consume two ticks per frame
in severe cold; consecutive native samples detect that rate for the warning
thresholds. Paused values do not advance a host-side countdown. Wall time
animates icons and rejects stale snapshots only.

`scripts/instrument-buff-icons.py` adds a read-only observer at the existing
camera-frame return `0x0886941C`, chaining after the established damage/HP and
return-timer instrumentation in a build copy. None of those mods needs to be
enabled for buff tracking to work. Original generated game sources are untouched.

`host/buff_icons_ui.cpp` draws original vector glyphs through the existing
ImGui overlay renderer. The provider never writes guest memory, changes buff
durations, consumes items or modifies a save. The package contains only a
manifest and this guide; native support is compiled into the launcher.

## Verification

`mhfu_buff_icons_tests` covers the exact 30/10-second boundaries, animation
states, middle-slot expiry/repacking, reapplication, all 21 melody slots,
severe-cold double depletion, frozen counters, negative/lasting modifiers,
invalid pointers/signatures, cart/reward states and disabling the provider.
`mhfu_buff_icons_instrumentation_tests` checks the observer and preservation
of the existing return and hooks.

With the owner's local files, the optional native probe runs actual decrement
and store instructions in private RAM for all seven item counters and all 21
melody slots. No game instructions or assets are included in the package.

The Windows native playthrough verified alignment beside sharpness, normal and
warning rendering, alternating red/original colors, middle-slot expiry with
the green recovery icon moving left after yellow stamina expired, and an empty
row after all effects expired. A separate attack melody remained alongside
the item effects. All 19 selected mod regression checks passed.

That visual run used shortened native-counter fixtures applied externally to
one newly launched process with an isolated data directory, after quest
loading finished. The production provider contains no fixture writes. Copied
save hashes were unchanged. This verifies native countdown/renderer integration;
individual item consumption, every melody type, severe-cold gameplay and real
multiplayer still need normal playtesting. The severe-cold warning adjustment
and all 21 melody slots have automated coverage.

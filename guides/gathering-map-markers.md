# Gathering Map Markers 0.1.0

Author: DuduX42. MHFU USA (ULUS10391), updated Freedom Unbound native launcher.

Install `gathering-map-markers-0.1.0.zip` through the launcher's Mods page and
enable Gathering Map Markers. The desktop shortcut uses the updated executable.
This is independent of Gathering Nodes: either mod can be enabled on its own.

Small markers follow the game's quest map projection, including its zoomed
view. Blue pickaxes identify mining spots, yellow bugs identify insect spots,
and green leaves identify ordinary gathering spots. Markers cover usable
nodes in the hunter's current area only. Changing areas switches the markers
to that area's nodes. Depleted, disabled and invalid spots disappear;
the native remaining-use counter controls their availability. Markers remain
useful with a drawn weapon and do not trigger gathering or consume tools.

There are no guest-memory or save writes. Node records and map data come from
the owner's running game; no game artwork, map textures or tables ship in the
package. The icons are original vector drawing commands.

## Implementation

`host/gathering_map.cpp` reads the quest's area-to-zone mapping, bounded
24-byte node lists, remaining uses and zone origins. It observes the native
local-hunter map arrow and uses the same affine scale/origin, including map
padding and zoom panning. It clips markers to that map view and collapses
duplicate native entries at identical locations.

`scripts/instrument-gathering-map.py` adds read-only observations at the normal
and zoomed native map-arrow call sites in build copies of main units 15/16.
Existing timing, gathering, HP and other observers remain composed. Unsupported
instrumentation inputs fail the build. `host/gathering_map_ui.cpp` draws the
original icons; its `gathering_map_markers` provider is selected by the ZIP.

## Validation

Native-state regression tests cover multiple zones, coordinate origins,
duplicate records, node depletion, invalid offsets, quest/rewards gating,
drawn-weapon visibility, zoom clipping and disabling. Instrumentation tests
verify both observation sites and preserve the original arrow instructions.

The native build passed 11 selected regression tests covering this mod and the
existing gathering, camera, HP and wishlist integrations. A live Jungle test
verified normal and zoomed map alignment, current-area filtering, depletion,
restoration and an empty area. This used a controlled node fixture in an isolated
process; temporary test memory changes are not part of the shipped mod. The mod
itself only reads game state.

Full natural exploration across every map and day/night variant remains for
player testing. Automated tests cover switching the current area.

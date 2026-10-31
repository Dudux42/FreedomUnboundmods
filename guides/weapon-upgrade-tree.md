# Weapon Upgrade Tree

Author: DuduX42. Version: 0.3.0.

Highlight a weapon at the smithy and press **F8** to open its upgrade chart.
Each weapon has its original class icon, tinted with the game's rarity color,
and its name. The highlighted smithy weapon has a green card.

Follow the connecting lines to see each possible upgrade. The chart includes
related branches and future upgrades, even when they are not unlocked yet.
Scroll horizontally and vertically for large families. Select a card with the
mouse or controller confirm button to show its information in a separate panel
beside the tree. Use the D-pad or arrow keys to move between cards. The panel
shows Power, element/status and the normal sharpness bar. Sharpness does not apply to bows
or bowguns. Bowguns also show recoil, reload, base deviation and compact
supported-ammo lists with magazine sizes. Light Bowguns also show which ammo
types support rapid fire. Bows show supported coatings and
the shot type/level for every charge; extra charges are marked Load Up. Equipment skills, buffs and bowgun attachments are not included.
Weapons already in the item box have a green check. A gold marker means the
weapon is the next upgrade from an owned weapon and its required materials are
available. The side panel legend explains the markers and branch lines.

Press F8, Esc, controller Back, or Close to return to the smithy. Browsing the
chart cannot craft a weapon or spend materials. Mouse/controller input belongs
to the chart while it is open. The game continues running behind it.

Install `weapon-upgrade-tree-0.3.0.zip` through the launcher's Mods page and
enable Weapon Upgrade Tree. This mod needs the updated native launcher.
Equipment Wishlist and the other mods are optional.

Version 0.3.0 adds the separate details panel, item-box ownership markers,
upgrade-ready markers, and clearer controller navigation. Version 0.2.0 adds
the Light Bowgun rapid-fire ammo list. Version 0.1.1 adds
F8 support on Improve Weapon's equipment grid, before
choosing an upgrade. Version 0.1.0 only observed the recipe/upgrade lists.

## Implementation notes

The mod independently reads the player's running USA ULUS10391 game. No game
artwork, weapon names, statistics, or upgrade tables are distributed in the ZIP.
The archive only selects the native provider and includes this guide.

The existing native smithy observer at 09A838E0/09A83CD8 and common page
observer at 09A81180 supply the highlighted row. It is observed even when
Equipment Wishlist is disabled. Menu +32 must identify the smithy; row +10
is the equipment type and +12 the weapon ID. Shops and armor are excluded.

The initial Improve Weapon grid uses a separate selection format. Its Info
prompt returns to 09A82140 with the menu in s1; phase 6 and operation 1 identify
that screen. Menu +64 is the absolute equipment-box slot. The native routines
09AA4828/09AA4930 resolve it through the hunter pointer at 089CC558, then
offset 0548 plus slot times 12; type is byte +1 and equipment ID is u16 +2.
The existing text observer reads this before the Wishlist enabled check.

Melee records start at 0895A0EC with a 24-byte stride (1150 entries);
ranged records start at 08960CBC with a 28-byte stride (354 entries).
The model-to-class table is 089785A8. Native upgrade lists have five u16
children at +16 in 28-byte records, based at 0894FAE4 for melee and 089578AC
for ranged weapons. These were verified against the original smithy loop
09AA3DA4..09AA3E84. Graph construction bounds IDs, removes duplicate edges,
includes ancestors and sibling branches, and excludes malformed cycles.

Power uses the native class multiplier at 0893ACDC divided by 100, as in
0881F124..0881F17C. Melee raw Power is +10, ranged +8. Melee elements are
+14/+15 and ailments +16/+17; the native dual-attribute exception table at
08979878 overrides those fields for fifteen weapons. Bow element is +17/+18. Displayed effect values multiply
by ten. Normal sharpness follows 088DAAA0 and 088DABB0: class pointer table
0897935C, profile index at +8, seven cumulative u16 thresholds per profile,
and maximum 150 + 50 * tier (+9). Skills such as Sharpness +1 are excluded.

Bowguns also show recoil (+11), reload (+19), base horizontal deviation from
the mod-level-zero record (pointer table 08979568, variant +3, float +16),
and supported ammo from the +24 mask. Magazine capacities come from the
32-byte configuration at 08978818 indexed by +16. Stats exclude power upgrades,
attachments and skills; deviation is shown in the game's stored scale. Rapid-fire
ammo is mapped by the in-game Light Bowgun name because the bowgun record has no
rapid-fire field; the mapping is checked against the [MHFU Light Bowgun roster](https://monsterhunter.fandom.com/wiki/MHFU:_Light_Bowgun_List).
Bows show four charge patterns from 08978F58 (+16 configuration, four bytes),
with the normal charge count at +11. Charges beyond that count need Load Up.
Codes 0..4 are Rapid, 5..9 Spread, 10..14 Pierce, each Lv1..5, verified against
the native shot-name table at 089ACAFC. Coatings use the native +24 masks.

Format cross-check: [Silvris's original format research](https://github.com/Silvris/MH-Tools-and-Scripts/blob/master/MHFU%20Weapons-Armor.bt)
and [extractor field ordering](https://github.com/Silvris/MH-Tools-and-Scripts/blob/master/MHFUWeaponDataExtract.1sc).
Only format facts were used as research leads; no third-party implementation
was copied or translated. Addresses and field behavior were checked against
the owner's executable. The references' regional file offsets are not used.

Icons and rarity colors use the launcher's existing native equipment reader.

F8 uses an optional framework-owned tool panel callback with the existing F9
input capture. F8 is reserved from gameplay bindings; alternate mappings are
preserved when an old F8 binding is removed. Guest memory is read on the game
thread and copied under a mutex; drawing never follows guest pointers.

## Validation

Regression tests cover branching, merged paths, duplicate links, cycles,
sharpness clipping, selection, smithy-only availability and disable behavior.
The optional original-executable check reads all eleven weapon classes from
the owner's ELF. Run:

```powershell
cmake --build build/windows --target FreedomUniteNative mhfu_weapon_tree_tests --parallel 2
ctest --test-dir build/windows -R "weapon_tree|wishlist|input" --output-on-failure
build/windows/mhfu_weapon_tree_tests.exe game/EBOOT.ELF
python scripts/package-weapon-tree.py
```

The five targeted regression suites passed. Reading the owner's executable
validated 1,502 weapons, 958 upgrade links and all eleven weapon classes,
including every bow charge pattern and bowgun magazine configuration.

Scripted native-game runs navigated naturally from a copied save to the smithy,
with only this mod enabled. Inspected screenshots verified the hammer upgrade
branches, rarity icons, sharpness, controller focus and bowgun stats/ammo.
F8 reopening, Esc closing and switching back to F9 were exercised. Both runs
exited normally through the test script, and the copied saves were unchanged.
Evidence is in `build/windows/weapon-tree-live/` and
`build/windows/weapon-tree-ranged-live/`.

For 0.1.1, a copied-save run reproduced the old failure on the initial Improve
Weapon grid. The fixed run opened Bone Kris's chart directly from that grid,
exercised controller focus, closed it with F8, selected Hunter's Bow I and
reopened the matching chart. Inspected screenshots verified its bow charge
shots/coatings, and Esc/F9 still worked. The copied save remained unchanged.
Evidence is in `build/windows/weapon-tree-grid-fixed-live/`. Regression tests
also cover absolute equipment slots beyond the first page, empty slots and
rejecting other menus/callers. Physical controller hardware remains to be
tested. No user save files are modified by this mod.

# Equipment Wishlist 0.2.0

Author: DuduX42. MHFU USA (`ULUS10391`), updated Freedom Unbound native launcher.

## Install and controls

Drop `build/mods/releases/equipment-wishlist-0.2.0.zip` onto the launcher **Mods** page,
install it and enable **Equipment Wishlist**. The rebuilt native launcher
contains its provider; the ZIP carries configuration and this guide.

Press **F7** in a village, quest or gathering hall to open the wishlist window.
Click a wishlisted item, or focus it and press the controller's configured
**Confirm** button, to expand its materials. The list shows how many you have,
how many are required and how many are still missing. Press F7, Esc, controller
Back or Close to leave the window. F7 is reserved for this panel.

At the smithy, highlight a craftable equipment recipe and its materials:

- **ZR / RT / R2:** pin that equipment recipe's material requirements.
- **ZL / LT / L2:** remove that recipe's pin.

The extra triggers no longer fall back to PSP L/R. The physical shoulder
bumpers and keyboard L/R bindings retain their normal behavior. Modern Camera continues
to use these triggers for its quest target controls outside the smithy.

The smithy shows the two control hints side-by-side at the bottom-right, below
the native menu panels, with no enclosing background box. Hints stay visible on
all smithy recipe detail pages. Pins appear beside wishlisted equipment names
in the smithy list/details and beside each pinned material name. The wishlist
window uses the same owned-material counts as quest readiness, combining the
item box and 24-slot pouch. Armor creation, weapon creation and weapon upgrade recipes are
distinct entries; decoration menus are excluded.

Pinning or removing a pin shows a three-second confirmation in the top-left
margin, above the HUD/menu panels. Rapid changes replace the previous confirmation.
Successfully crafting or upgrading a pinned recipe automatically removes its pin,
persists the updated wishlist and displays **Wishlisted equipment crafted** at the
middle-left for five seconds. Cancelling or failing a craft leaves it pinned.
Village readiness reminders wait until the village world is actually rendered
and loading/fade covers have cleared, then receive their full five-second display.

## Quest notifications

Acquiring a material used by a pinned recipe shows **Wishlist item acquired**,
the material name and acquired quantity at the middle-left. When a recipe's
requirements become satisfied, it shows **Materials for [equipment] acquired**.
Carving during the successful-quest return countdown also counts.

Each notification lasts five visible seconds, including its smooth entry and
exit. Both motions travel upward: entry fades from transparent to opaque, and
exit fades back to transparent. New cards arrive individually, pushing older
ones upward. Expired cards are removed and the remaining cards move smoothly
into their places. At most three are visible; additional cards wait in order.

Owned materials are checked independently for every pinned recipe. Materials
are never reserved or deducted by this mod. If two recipes each need three of
an item and you own three, both can announce readiness. Crafting one normally
uses those materials; the other recipe becomes unready and can announce again
when you replenish the missing stock on a later quest. Extra acquisitions do
not repeat an already-ready equipment notification. Pinning an already-ready
recipe or entering a quest does not manufacture pickup notifications.

Pending quest notifications wait through rewards and resume in the village.
Every village return and hunter save load also announces each pinned recipe
whose materials are currently ready. Repeated idle frames do not repeat these
reminders. No reward-screen inventory transfers create pickup notifications.
Launcher menus pause the visible notification animation.

## Persistence and native data

Pins are stored separately from game saves in the launcher's data directory:
`wishlist/<hunter-key>.txt`. The key combines the selected character slot and
native hunter name, so equipment changes do not lose pins and characters with
the same name in different slots remain separate. Renaming a hunter starts a
new list. Files are validated and staged before replacing the previous file.
Malformed lists are ignored and a recoverable `.invalid` copy is retained.

The mod reads the native highlighted smithy row and its 26-byte creation record
or target-indexed 28-byte weapon-upgrade material record.
It copies only the selected requirements into the personal pin file. No game
recipe tables, executable instructions, fonts or artwork ship in the ZIP.
Inventory observations and pin/text hooks never write guest memory or saves.

Pressing controller Confirm on a highlighted wishlist item expands or collapses
its material list. Mouse clicks do the same. Closing the window restores normal
game input. The F7 panel uses the launcher's interactive in-game panel, so mouse
and controller actions are captured while it is open.

The native source adapters are in `host/equipment_wishlist.cpp`; checklist and
notification behavior are in `host/equipment_wishlist_model.cpp`. Drawing and
controller hints are in `host/equipment_wishlist_ui.cpp`.
`scripts/instrument-equipment-wishlist.py` observes build copies of the smithy
recipe renderer, equipment-commit return, material rows and common text-rendering entries, retaining existing text,
cheat and key-quest hooks. The camera-frame observer in `host/save_portraits.cpp`
supplies the game-frame inventory updates and routes the extra triggers.

## Controller presentation extension

The Controller Prompts mod now calls
`wishlist::set_control_hint_provider(ControlHintProvider)` on the UI thread.
The callback returns `ControlHints` for pin and unpin. Each `ControlHint` can
provide a label, or a `draw(x, y, size)` glyph callback and its width. The
provider can inspect its connected controller and return the corresponding
Switch, Xbox or PlayStation artwork. Unregister by passing `nullptr` before
unloading the provider. Without a provider, the controls show the equivalent
ZR/RT/R2 and ZL/LT/L2 labels. The `controller_hint_provider` capability is exposed
by the registered native provider.

## Validation

Version 0.1.2 reads the native recipe row's equipment type as one byte, keeping
the adjacent availability flags separate. Regression cases cover flagged armor
and weapon rows, successful creation/upgrade automatic removal, failed insertion,
mismatched equipment, and persistence after automatic removal. The craft observer
runs at the native equipment-commit return and validates the resulting box slot.
A Windows visual run pinned Bone Blade, completed creation and declined immediate
equipping. Its equipment/material pins disappeared and the persisted wishlist
contained zero entries afterward; copied game-save hashes remained unchanged.
Live upgrade removal remains covered by native-state tests rather than a full
upgrade playthrough.

Version 0.1.1 additionally verifies the three-second pin/unpin confirmation,
equipment-name pins, and deferred five-second reminders. Native Windows
screenshots confirm the bottom-right controls outside the smithy panels,
pins on equipment names and materials, and top-left pin/unpin confirmations
that disappear while the pins persist. Physical bumper inputs still switch
the native requirements view; extra trigger inputs pin/unpin.

A save-load visual check confirms no readiness cards during the village
fade-in, followed by cards in the fully visible village. Timer tests confirm
the queue starts at presentation and grants the full five seconds. Both
isolated native runs preserved every copied game-save hash.

Automated checks cover independent overlapping recipes, spending/reacquisition,
quest-entry baselines, duplicate materials, pin/unpin, validated persistence,
per-slot separation, native read-only observations, post-success carving,
village-return and save-load readiness reminders, weapon-upgrade materials,
five-second animation curves, staggered entry and the three-card queue limit.
Instrumentation checks reject unsupported inputs and preserve earlier hooks.

An additional local probe runs the owner's native recipe-address instructions
and compares the bridge against 200 actual armor/weapon creation records.

Windows native visual checks verified smithy pin/unpin/repin with a scripted
controller, a real Bone Kris+ upgrade and its two material pins, readable control hints, and village
save-load reminders. The queue test preloaded five legitimate native recipes
against the isolated hunter's existing stock; no inventory or save edits were
needed. Copied game-save hashes remained unchanged.

Quest pickups, carving, spent-stock rearming and village-return transitions
are covered by native-state regression fixtures. A complete quest/craft/reload
playthrough and physical Switch/Xbox/PlayStation controllers still need a
player test. The connected-controller prompt provider is now available as
the separate Controller Prompts mod; see `CONTROLLER_PROMPTS_MOD_GUIDE.md`.

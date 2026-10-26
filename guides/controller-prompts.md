# Adaptive Button Prompts 0.4.0

Author: **DuduX42**. Game: MHFU USA (`ULUS10391`).

Install `adaptive-button-prompts-0.4.0.zip` using the launcher's mod manager and
enable Adaptive Button Prompts. Requires the updated Freedom Unbound native build.

Vanilla Circle, Cross, Triangle, Square, L/R, START/SELECT, analog-stick and
D-pad sprites on the
recognized common-menu/heading sheets are replaced with their current input
assignments. Controller prompts follow `input.pad.*`, including Confirm/Back
swaps; keyboard prompts follow `input.bind.*`. Compact native icons show the first
assigned binding; the Controls screen shows all assignments. Mouse buttons use
ML/MR/MM/M4/M5 and modifier keys use short labels such as LS for Left Shift.
Analog and D-pad caps show the four corresponding bindings in a compact cross;
the controller's analog stick uses LS. The symbols retain their native slot,
tint, alpha, clipping and draw order.
Disabling the mod restores the game's original sprites on subsequent draws. HD texture
packs remain compatible: recognition uses the original native sprite silhouettes.

The smithy's Equipment Material Pin controls use individual button caps:

| Input | Remove pin / face target | Pin / next target |
|---|---|---|
| Switch Pro / Joy-Con | ZL | ZR |
| PlayStation | L2 | R2 |
| Xbox / generic gamepad | LT | RT |
| Keyboard defaults | F6 | F7 |

Prompts follow the last input device used. Keyboard presses, mouse clicks or
pointer movement select keyboard prompts; gamepad buttons or deliberate axis movement select
gamepad prompts. Disconnecting the controller falls back to keyboard prompts.
Connection alone does not override the last keyboard input. SDL identifies the
controller exposed to the launcher: Steam Input or another translation layer
may expose an Xbox controller even when the physical controller is different.

In Controls, remap **Extra left trigger (mod action)** and **Extra right trigger
(mod action)** under keyboard bindings. Both slots and mouse bindings are
supported, and prompts read the current assignments each frame. These extra
actions reach the same smithy/target-camera handlers as analog triggers. They
do not press PSP L/R. Existing controller triggers and D-pad mappings are kept.
The keyboard shortcuts work whenever the corresponding gameplay mod is enabled;
Controller Prompts controls their visual presentation.

With Modern Camera, a small top-right legend above the minimap displays Face target and Next
target while a monster is selected. It also works with Monster HP enabled.
Smithy prompts require Equipment Material Pin. Standalone button sprites on the
recognized sheets are supported. Literal written instructions such as "SELECT
button", tutorial diagrams and artwork on other sheets are not rewritten.

Implementation: `host/controller_prompts.*` exposes a shared UI-thread action
label/drawing API and fills the existing wishlist control-hint provider. Button
art follows the detected controller: Switch face buttons use round letter caps,
PlayStation face buttons use the Cross/Circle/Square/Triangle symbols, and Xbox
face buttons use their colored A/B/X/Y labels. Keyboard bindings use raised,
light keycaps with dark labels. Framework input adds two
generic, configurable extra actions and merges them with physical triggers
before dispatch, preserving edge-triggered pin/target actions and menu capture.
Native replacement uses a generic profile draw-transform hook. Only verified
button cells are replaced; unrelated sprites in a shared draw retain their
original order and texture. New caps and compact letterforms are original vector
geometry. No original game images, game data or save modifications are included.

Nine targeted tests passed: native sprite selection/order/geometry, controller labels and input routing, keyboard/mouse
bindings, settings, wishlist native/model/instrumentation, and modern-camera
native/instrumentation checks. The native Windows build compiled successfully.

An isolated smithy run verified controller caps, F8/F9 rebound caps, keyboard
pin/unpin, controller repin, and switching back to keyboard. A native quest run
with Monster HP enabled verified F7 target cycling, F6 camera snapping, and
switching to controller prompts/actions. Final screenshots verified the two-row
legend above the minimap, clear of health, stamina and the item-name strip.
Camera preview fixtures supplied sample
targets; no fake monsters were inserted into game memory. Copied save hashes
remained unchanged. Physical Switch/PlayStation/Xbox hardware and Bluetooth
reconnects still need player testing; SDL family labeling is covered by code and
label tests, while the native input runs used SDL's virtual gamepad.

Version 0.2 native runs verified Cross rebound to G versus controller A, analog
rotation prompts switching between WASD and LS, Square rebound to H versus
controller Y, and the quest HUD's L prompt switching between Q and LB. Native
sprite tests also cover exact shoulder widths, START/SELECT and D-pad cells,
mixed-sprite ordering, unrelated art, finite geometry and fade alpha. These
checks use isolated settings and copied saves; game-save hashes were unchanged.
An additional native menu run loaded FU retextured 0.8.1 (489 texture mappings)
and visually verified the new caps alongside its HD panels and text. Its copied
save hashes were unchanged as well.

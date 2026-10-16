# Quest Return Timer 0.1.0

Author: DuduX42. Game: MHFU USA (`ULUS10391`) in the updated native launcher.

## Install and behavior

Drop `build/mods/return-timer-0.1.0.zip` onto the launcher's **Mods** page,
install it and enable **Quest Return Timer**. The desktop shortcut uses the
rebuilt native executable; the configuration ZIP requires that provider.

A compact top-center panel shows **RETURN IN** and the remaining time as
`MM:SS` during the return countdown after a successful quest. The final ten
seconds change to a warm red color. It sits below the top HUD, away from the bottom-left HP bars
and the carving/item-notification area.

It follows the game's actual return counter, including the native 20- and
60-second durations. No host stopwatch advances it. The success announcement
before that countdown is not mistaken for five seconds remaining. The panel
is hidden during normal hunts, Village/Hall menus, failed/abandoned quests,
return transitions and rewards. Launcher menus hide gameplay overlays normally.
It changes no quest timing, rewards or save data.

## Implementation

The native quest manager's success-return state is mode 1, substage 2. Its
counter at quest object +`0x710` is decremented by the native routine at
`0x0886B494`, with a store at `0x0886B49C`. Initialization chooses 1800 or
600 ticks; the native 30-Hz logic gives 60 or 20 seconds respectively.

`host/return_timer.cpp` validates instruction signatures, the loaded gameplay
overlay, current quest/state pointers, success phase and return substage.
Only positive counters up to the supported 60-second maximum are displayed.
Seconds round upward so the final fraction remains `00:01` until the game
leaves that state. Invalid contexts immediately clear the snapshot.

`scripts/instrument-return-timer.py` adds a read-only observer at the existing
camera-frame return `0x0886941C`, after the established damage/HP instrumentation.
This runs independently of whether those mods are enabled. It modifies a build
copy, leaving the original generated corpus unchanged. The observer reads live
native time and publishes a synchronized snapshot; host time is used only to
reject stale snapshots. No guest memory or registers are written.

`host/return_timer_ui.cpp` registers the provider and draws the panel using the
existing overlay renderer. `scripts/package-return-timer.py` creates the
deterministic configuration-and-guide ZIP. No game code or assets ship in it.

## Checks

CTest names: `mhfu_return_timer_tests` and
`mhfu_return_timer_instrumentation_tests`. Tests cover rounding, supported
durations, frozen native values, success announcements, hunting/failure/reward
states, invalid pointers/signatures and disabling the provider.

With the owner's local ELF, `mhfu_return_timer_tests.exe game/EBOOT.ELF` also
executes the real decrement/branch/store instructions in private RAM and
compares the displayed seconds throughout both countdown lengths. It does not
touch a user save. `MHFU_RETURN_TIMER_TRACE=1` enables diagnostic native-counter
logs for explicit QA runs; normal desktop launches leave it off.

The Windows native playthrough verified the 20-second countdown, its red final
ten seconds, hiding during the success announcement, pause/resume behavior and
clearing during the return transition. The existing completion cheat was used
only in an isolated QA profile to trigger success; the copied save hashes were
unchanged. Both 20- and 60-second counters passed the private native-instruction
probe, and all 17 selected mod regression tests passed. Real multiplayer and
later quests have not been playtested for this mod.

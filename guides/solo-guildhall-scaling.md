# Solo Guildhall Scaling

Version 0.1.0 — Freedom Unbound, MHFU USA (ULUS10391).

Large monsters have **35% less HP** in ordinary Offline Gathering Hall quests.
A monster that normally has 10,000 HP starts with 6,500 HP instead.

Online Gathering Hall quests keep their normal HP, even if you play alone.
Village quests, small monsters, attack damage, rewards, status resistance and
part/stagger thresholds are unchanged.

## Installation

Install `solo-guildhall-scaling-0.1.0.zip` through the launcher's Mods page and
enable it before departing on a quest. It requires the updated native launcher
with the `offline_hall_hp` hook; the ZIP does not contain a standalone plugin DLL.
Monster Health Bars is optional.

Change the enabled setting between quests. It affects native HP calculations;
it does not rescale monsters already fighting when the setting is changed.

## Coverage and verification

The first release targets normal Hall quest IDs 1000–3999. Training, Village and
quests outside that range are excluded. Siege, repel and carried-over elder
dragon HP need gameplay verification; the game retains its native encounter
logic and this mod does not rewrite saved damage or quest tables.

The hook adjusts the result of the native monster HP calculator once per call,
after the game's own quest calculation. It does not repeatedly multiply live
HP, alter player health, or change network data. Unknown profile signatures,
invalid pointers, non-HP stat selectors and online mode leave the result alone.

Automated checks cover offline/online eligibility, quest exclusions, invalid
inputs, small-monster exclusion and rounding across the signed 16-bit HP range.
In-game validation remains pending: compare the same ordinary quest offline and
online, then check a Village quest and a second offline quest for state leakage.

# INCONTINENTIA'S UNDERCOVER / CIVILIAN RECRUITMENT - INCON UNDERCOVER — Community Update
### Based on Incontinentia's Undercover / Incognito Simulation Script for Arma 3

---

## Overview

Incon Undercover is a comprehensive, performance-friendly undercover and incognito simulation for Arma 3. Work as a guerrilla cell, go undercover as a civilian or enemy soldier, recruit comrades, and cause mayhem — all without a single gamey checkbox in sight.

This community update builds on the original script by Incontinentia, incorporating bug fixes, stability improvements, ALiVE and ACE compatibility work, and an expanded civilian recruitment system developed and validated on active multiplayer servers.

Compatible with Singleplayer, Coop, and Dedicated Server.

---

## Credits

**Original script:** Incontinentia
**Detection system basis:** Grumpy Old Man, Tajin, sarogahtyp
**Original additional functions:** Spyderblack723
**Optimisation contributions:** das attorney, davidoss, Bad Benson, Tankbuster, dedmen, fn_Quiksilver, marceldev89, baermitumlaut, Duda123, Jebediah, Jmaster, ScottyZ
**Original testing:** accuracythruvolume
**Optimization testing:** Silver Squadron
**Community update:** Reschke — with thanks to the one and only Incontinentia for the original work and permission to continue development as needed.

---

## Requirements

- **CBA_A3** (required)
- **ACE3** (optional — ACE interact menu integration included)
- **ALiVE** (optional — civilian recruitment and persistence integration included)

---

## What's New in This Version

### Bug Fixes
- Fixed a critical side-switching bug caused by engine behaviour where `setCaptive` could silently switch a unit's side to civilian rather than simply setting captive state. A `captiveCheck` system now wraps every captive state change to detect and correct side mismatches before they occur. This was the root cause of the "unresponsive enemies" bug on longer or multi-session missions when running ALiVE and ACE simultaneously.
- Fixed `fn_initUcrVars` nil check that was incorrectly overwriting `_asymEnySide` instead of `_regEnySide` when `_regEnySide` was undefined, causing unpredictable side assignment on certain faction configurations.
- Fixed vehicle compromise flag logic that used `&&` (AND) instead of `||` (OR) when checking whether either enemy side had spotted the unit's vehicle. Previously the flag would only set when both sides simultaneously knew about the vehicle — which almost never happens in single-enemy-faction missions.
- Fixed trespass and high security zone checks that used `true` as the default variable value, causing both checks to be skipped entirely on the first execution pass.
- Fixed `fn_recruitHandler` `makeCivNormal` operation which called `enableAI "CHECKVISIBLE"` twice in sequence. Second call corrected to `enableAI "COVER"`.
- Fixed `_carryAllWeaponsOpenly` switch in `fn_recruitHandler` which had the `addWeapon` and `addCarryWeapon` cases reversed, causing civilians to always conceal weapons when the flag was true and always carry openly when false — the opposite of the intended behaviour.
- Fixed `fn_initUcrVars` which previously used `exitWith` on a unit side mismatch, aborting the entire initialisation script. Now logs a warning and continues, preventing hard init failures when ALiVE or ACE temporarily shuffles unit sides during setup.
- Corrected copy-paste error in `fn_groupsWithPID` function header which incorrectly described the function as `IsKnownExact`.

### Expanded Civilian Recruitment System
The civilian gear initialisation system has been significantly expanded with new configurable options (see `UCR_setup.sqf`):

- Tiered weapon rarity system — civilians can now be assigned common, rare, and super rare weapons from separate configurable arrays with weighted probability
- Pistols can be automatically hidden in uniform rather than carried visibly, controlled by `_hideAllPistols`
- Fine-grained control over overt vs concealed weapon carry via `_canCarryOpenly` and `_carryAllWeaponsOpenly`
- Configurable backpack and vest assignment chance via `_civPackPercentage` and `_civVestPercentage`
- Configurable maximum magazine count per civilian via `_maxCivMags`
- Civilians of the same side as the undercover unit now receive a recruitment chance bonus
- Recruited civilians now have ACE medical, EOD and engineer flags set automatically

### ALiVE Compatibility
- `makeCivNormal` operation in `fn_recruitHandler` now correctly unregisters recruited civilians from ALiVE agent tasking and disables ALiVE civilian animations before handing control to the undercover system
- Side mismatch handling changed from hard abort to soft warning to prevent init failures in ALiVE-managed faction environments

---

## Installation

1. Copy the `INC_undercover` folder into your mission folder.
2. If you already have a `description.ext`, add the `#include` line to the existing `cfgFunctions` class rather than replacing the file. If you already have an `initPlayerLocal.sqf` or `postInitXEH.sqf`, add the relevant lines to your existing files rather than replacing them.
3. Configure your settings in `INC_undercover\UCR_setup.sqf`. Read every setting carefully — one wrong value can cause unexpected behaviour.
4. For each trespass / out-of-bounds area, place a map marker with `INC_tre` somewhere in the marker name (e.g. `INC_tre_airbase` or `myMarker_INC_tre_01`). The script will find them automatically.
5. For each high security zone, place a map marker with `INC_highSec` somewhere in the marker name.
6. For each playable undercover unit, add the following to their unit init in the editor:

```sqf
this setVariable ["isSneaky",true,true];
```

AI units in the undercover player's group do not need this — the script handles them automatically on mission start.

---

## Configuration — UCR_setup.sqf

All settings live in `INC_undercover\UCR_setup.sqf`. Do **not** comment out any lines — this will break the script.

### General Settings

| Variable | Type | Default | Description |
|---|---|---|---|
| `_undercoverUnitSide` | Side | `west` | The side that undercover units belong to. Only one side supported. |
| `_debug` | Bool | `false` | Enables debug hints and log output. |
| `_fullAIfunctionality` | Bool | `true` | Runs all checks on AI group members. May slightly affect performance with 15+ unit groups. |
| `_easyMode` | Bool | `true` | Check Disguise action also tells the player whether their disguise is working. |
| `_racism` | Bool | `true` | Enemies notice if the unit's face doesn't match the faction they are impersonating. |
| `_racProfFacCiv` | Number | `1` | Multiplier for racial profiling by civilians. Lower to simulate a more multicultural population. |
| `_racProfFacEny` | Number | `1` | Multiplier for racial profiling by enemies. Lower to simulate more multicultural enemy forces. |
| `_globalSuspicionModifier` | Number | `1` | Global suspicion scaler. 2 = twice as hard to stay undercover. 0.5 = half as hard. |

### Enemy Sides

| Variable | Type | Default | Description |
|---|---|---|---|
| `_regEnySide` | Side | `east` | Regular enemy side. Shares detected unit identity across the entire map. Use `sideEmpty` if not needed. |
| `_regBarbaric` | Bool | `false` | Regular side may lash out at civilians after taking casualties if attacker is unknown. |
| `_regDetectRadius` | Number | `10` | Base detection radius for regular troops in metres. Expands and contracts based on behaviour, weather, and time of day. |
| `_asymEnySide` | Side | `sideEmpty` | Asymmetric enemy side. Better at spotting imposters but only shares identity locally. Use `sideEmpty` if not needed. |
| `_asymBarbaric` | Bool | `true` | Asymmetric side may lash out at civilians after taking casualties if attacker is unknown. |
| `_asymDetectRadius` | Number | `15` | Base detection radius for asymmetric troops in metres. |

### Civilian Disguise

| Variable | Type | Description |
|---|---|---|
| `_civFactions` | Array | Faction classnames whose gear is automatically considered safe for civilian disguise. |
| `_civilianVests` | Array | Additional safe vest classnames on top of faction auto-detection. |
| `_civilianUniforms` | Array | Additional safe uniform classnames. |
| `_civilianHeadgear` | Array | Additional safe headgear classnames. |
| `_civilianBackpacks` | Array | Additional safe backpack classnames. |
| `_civilianVehicleArray` | Array | Additional safe vehicle classnames. |
| `_HMDallowed` | Bool | Whether HMDs (NVGs etc.) are safe to wear as a civilian. |
| `_noOffRoad` | Bool | Civilian vehicles driving more than 50m off-road are immediately considered hostile. |

### Enemy Disguise

| Variable | Type | Description |
|---|---|---|
| `_incogFactions` | Array | Enemy faction classnames whose gear and vehicles allow impersonation. |
| `_trespassMarkers` | Array | Additional trespass marker names (markers with `INC_tre` in the name are auto-detected). |
| `_incognitoVests` | Array | Additional safe vests for enemy disguise. |
| `_incognitoHeadgear` | Array | Additional safe headgear for enemy disguise. |
| `_incognitoBackpacks` | Array | Additional safe backpacks for enemy disguise. |
| `_incognitoUniforms` | Array | Additional safe uniforms for enemy disguise. |
| `_incogVehArray` | Array | Additional incognito vehicles beyond faction auto-detection. |

### High Security Zones

| Variable | Type | Description |
|---|---|---|
| `_highSecMarkers` | Array | Additional high security marker names (markers with `INC_highSec` are auto-detected). |
| `_highSecInstantHostile` | Bool | If true, wrong uniform in high security area = instantly hostile. If false, highly suspicious. |
| `_highSecVehicles` | Array | Vehicles that can enter high security areas without raising suspicion. |
| `_highSecurityUniforms` | Array | Uniforms that permit entry into high security areas. |
| `_highSecItemCheck` | Bool | Check for disallowed items in high security areas. Each non-permitted item adds suspicion. |
| `_highSecItems` | Array | Items permitted in high security areas without raising suspicion. |
| `_hsItChkOutside` | Bool | Apply high security item checks even outside high security zones when wearing a high security uniform. |
| `_hsMustBeUnarmed` | Bool | Carrying any weapon in a high security area is treated as hostile. |
| `_highSecItemCheckScalar` | Number | Multiplies suspicion caused by each disallowed item in high security areas. |

### Civilian Recruitment

| Variable | Type | Default | Description |
|---|---|---|---|
| `_civRecruitEnabled` | Bool | `true` | Enable or disable civilian recruitment entirely. |
| `_armedCivPercentage` | Number | `70` | Percentage of recruitable civilians who will be armed. |
| `_civPackPercentage` | Number | `30` | Percentage of civilians who will be given a backpack. |
| `_civVestPercentage` | Number | `10` | Percentage of civilians who will be given a vest. |
| `_hideAllPistols` | Bool | `true` | Pistols are placed in the uniform container rather than carried visibly. |
| `_canCarryOpenly` | Bool | `true` | If a weapon cannot fit in uniform or backpack, carry it openly instead of discarding. |
| `_carryAllWeaponsOpenly` | Bool | `false` | All armed civilians carry weapons openly regardless of inventory space. Overrides `_canCarryOpenly`. |
| `_rareWeaponPercentage` | Number | `20` | Chance a civilian receives a weapon from `_rareWeaponArray` instead of `_civWpnArray`. |
| `_superRareWeaponPercentage` | Number | `5` | Chance a civilian receives a weapon from `_superRareWeaponArray`. |
| `_maxCivMags` | Number | `10` | Maximum number of magazines a civilian will carry. |
| `_civWpnArray` | Array | — | Common civilian weapon classnames. |
| `_rareWeaponArray` | Array | `[]` | Rare weapon classnames. Falls back to `_civWpnArray` if empty. |
| `_superRareWeaponArray` | Array | `[]` | Super rare weapon classnames. Falls back to `_civWpnArray` if empty. |
| `_civItemArray` | Array | — | Miscellaneous items civilians may carry. |
| `_civPackArray` | Array | — | Backpack classnames for civilian backpack assignment. |

---

## Known Limitations

- Only one side can have undercover units at a time.
- Only one side can be defined as regular and one as asymmetric — both must be hostile to the undercover side.
- In three-way conflicts, incognito units (dressed as one enemy faction) will be seen as friendly by the third faction due to an engine limitation.
- Factions that use randomisation scripts for gear (e.g. some mod factions) may not be fully auto-detected. Use the manual classname arrays in `UCR_setup.sqf` to cover missing items.
- In multiplayer, set respawn timers to at least 5 seconds to allow the script to correctly detect unit death and reset variables.

---

## How It Works

Behaviours fall into three categories:

**Suspicious** — immediately makes enemies see the unit as hostile if witnessed. Two minor suspicious behaviours together (e.g. being armed AND trespassing while dressed as a civilian) or one major one (firing a weapon) will compromise the unit.

**Weird** — does not immediately blow cover, but each additional weird behaviour increases the chance that nearby enemies will challenge or follow the unit. Accumulate enough weirdness and enemies will blow your cover outright.

**Attention-drawing** — expands the radius at which enemies start paying attention to the unit. Running, wearing the wrong kit, or driving fast all draw attention from further away, making it more likely that any weird behaviour will be noticed.

Once compromised, the unit must eliminate everyone who knows about them before the information spreads, then change disguise out of sight of any remaining enemies to go undercover again. Each time a unit is fully compromised, enemies become more suspicious of them even after a disguise change.

---

## License

GPL-3.0 — see LICENSE file. Original work by Incontinentia.

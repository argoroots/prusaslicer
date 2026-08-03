# PrusaSlicer: CORE One INDX — 0.25 nozzle on tool 8 + Flexfill TPU

PrusaSlicer 2.9.6. Prusa ships no 0.25 or flex profiles for INDX, so these were derived from Core One(+) system profiles. Only the listed values were changed; everything else stays inherited from the base preset.

## Prerequisites — system profiles to install first

Configuration → Configuration Wizard → Prusa Research, tick:

- **Prusa CORE One INDX 8T** (HF0.4) — base for the printer profile; also install its filament profiles
- **Prusa CORE One+ — 0.25 nozzle variant** — parent of the print profile and the PLA/PETG copies
- **Prusa CORE One+ — HF0.4 nozzle variant** — parent of the Flexfill copy (source of the `@COREONE+` flex profile)

In the wizard's Filaments tab make sure Prusament PLA, Prusament PETG, and the Flexfill/FLEX profiles are ticked for those printers. Without these parents the custom presets fail to load (`inherits` can't resolve) — required on every machine before importing the bundle.

## Printer

### `Prusa CORE One INDX 7T HF0.4 + 1T 0.25 nozzle`

Copy of system printer profile: **Prusa CORE One INDX 8T HF0.4 nozzle**

| Setting | From | To | Why |
|---|---|---|---|
| Extruder 8 → Nozzle diameter | 0.4 | 0.25 | physical nozzle on tool 8 |
| Extruder 8 → High flow | on | off | the 0.25 is a standard-flow nozzle |

Also set tool 8 to 0.25 / standard in the printer's own tool settings (firmware nozzle check).

## Print settings

### `0.12mm STRUCTURAL @INDX 0.25`

Copy of system print profile: **0.12mm STRUCTURAL @COREONE 0.25**

| Setting | From | To | Why |
|---|---|---|---|
| Compatible printers condition | `printer_model=~/(COREONE\|COREONEOAK\|COREONEMMU3)/ and nozzle_diameter[0]==0.25 and printer_notes!~/.*HT400.*/` | `printer_model=~/COREONE_INDX.*/ and nozzle_diameter[7]==0.25` | original matched neither INDX models nor extruder 1 = 0.4; new one pins to INDX with 0.25 on tool 8 |

## Filaments

### `Fillamentum Flexfill 98A @INDX`

Copy of system filament profile: **Fillamentum Flexfill 98A** (Core One+ HF0.4 nozzle)  
Used on the HF 0.4 tools.

| Setting | From | To | Why |
|---|---|---|---|
| Compatible printers condition | Core One model list variant | `printer_model=~/COREONE_INDX.*/ and nozzle_diameter[0]>=0.4` | INDX printers, 0.4+ nozzles only (HF or standard) — excludes the 0.25 tool, where flex is impractical |
| Start G-code | `M900 K0` + `M142 S36` | `M572 S0 ; Pressure advance off`<br>`M573 R` | INDX dialect; PA off for flex |

### `Prusament PETG @INDX 0.25`

Copy of system filament profile: **Prusament PETG** (Core One+ 0.25 nozzle)

| Setting | From | To | Why |
|---|---|---|---|
| Compatible printers condition | Core One model list + `!=0.6/0.8` + `! nozzle_high_flow[0]` + `!HT400` | `printer_model=~/COREONE_INDX.*/ and nozzle_diameter[0]==0.25 and ! nozzle_high_flow[0]` | show only on the std-flow 0.25 tool of INDX printers |
| Max volumetric speed | 12 | 4 | inherited value is for 0.4; real ceiling for std 0.25 |
| Start G-code | legacy/modern branch with full nozzle table + `M142 S36` | `M572 S{if nozzle_diameter[filament_extruder_id]==0.25}0.18{else}0.053{endif} ; Pressure advance`<br>`M573 R` | INDX uses `M572`, not `M900`; `M142` is Nextruder-only. PA values 0.18/0.053 taken unchanged from the original table; `M573 R` added |

### `Prusament PLA @INDX 0.25`

Copy of system filament profile: **Prusament PLA** (Core One+ 0.25 nozzle)

| Setting | From | To | Why |
|---|---|---|---|
| Compatible printers condition | Core One model list + `!=0.6/0.8` + `! nozzle_high_flow[0]` + `!HT400` | `printer_model=~/COREONE_INDX.*/ and nozzle_diameter[0]==0.25 and ! nozzle_high_flow[0]` | show only on the std-flow 0.25 tool of INDX printers (filament conditions evaluate per tool → `[0]` = current tool) |
| Max volumetric speed | 15 | 4 | inherited value is for 0.4; real ceiling for std 0.25 |
| Start G-code | legacy/modern branch with full nozzle table + `M142 S36` | `M572 S{if nozzle_diameter[filament_extruder_id]==0.25}0.12{else}0.036{endif} ; Pressure advance`<br>`M573 R` | INDX uses `M572`, not `M900`; `M142` is Nextruder-only. PA values 0.12/0.036 taken unchanged from the original table, other nozzle branches dropped; `M573 R` added to match Prusa's INDX profiles |

## Adding another filament for the 0.25 tool

1. **Get the right base profile.** Switch the active printer to the plain `CORE One+ 0.25 nozzle` (if it's not in the printer list, tick it in the Configuration Wizard first). In the filament dropdown, select the filament you want — with this printer active, PrusaSlicer shows the 0.25-tuned system variant if Prusa ships one. If there is no 0.25 variant, the standard one works too; the steps below fix everything nozzle-specific anyway.

2. **Save a copy.** In Filament Settings, save it under a new name following the existing convention, e.g. `<Filament name> @INDX 0.25`.

3. **Set the compatibility condition.** In *Dependencies → Compatible printers condition*, replace the inherited condition with:
   `printer_model=~/COREONE_INDX.*/ and nozzle_diameter[0]==0.25 and ! nozzle_high_flow[0]`
   This makes the preset visible only on the standard-flow 0.25 tool of INDX printers and hides it on the HF 0.4 tools. Also check that the *Compatible printers* picker (the explicit list) is empty — a non-empty list overrides the condition.

4. **Set an explicit max volumetric speed.** In *Advanced → Max volumetric speed*, enter a realistic ceiling for a standard-flow 0.25 nozzle — around 4 mm³/s for PLA/PETG-class materials, lower for flex or high-viscosity materials. Do not keep the inherited value: even the 0.25 system variants inherit the 0.4 nozzle's much higher number and rely on slow print profiles instead of a real cap.

5. **Rewrite the start G-code.** The inherited G-code contains a legacy/modern branch, a PA table for all nozzle sizes, and the Nextruder-only `M142 S36` line. Replace the whole block with two lines:
   `M572 S{if nozzle_diameter[filament_extruder_id]==0.25}<0.25 value>{else}<0.4 value>{endif} ; Pressure advance`
   `M573 R`
   Take both PA values from the original G-code's own `M572` table (the entries for 0.25 and 0.4) — they are Prusa's calibrated numbers for this material. Keep the conditional even for a 0.25-only preset; it costs nothing and stays correct if the preset is ever used on a 0.4 tool.

6. **Verify.** Switch the printer back to the INDX profile. The new preset must appear in tool 8's filament dropdown and be absent from tools 1–7. Slice a small test and check the G-code contains the expected `M572 S<0.25 value>` line.

## Transfer

File → Export → Export Config Bundle. On the target: run Configuration Wizard first (Core One+ 0.25 and HF0.4 variants + INDX printers, needed for `inherits` to resolve), then Import Config Bundle.

## After PrusaSlicer / system profile updates

Open each custom preset and review fields marked as modified (orange) against the change tables above:

- Marked fields **listed in the tables** are our deliberate changes — re-check that the values still make sense against the updated system profile.
- Marked fields **not listed in the tables** are Prusa's updates that our snapshot is missing — use the revert arrow to adopt them.
- If a parent profile was renamed/removed, the preset fails to load — fix the `inherits` line in the preset .ini to the new parent name.
# Flashforge Creator Pro 2 + Octopus + Klipper

This repository documents my **FFCP2+OK** conversion: a Flashforge Creator Pro 2
running Klipper on a BIGTREETECH Octopus Pro v1.1 H723 controller.

I use this printer primarily for functional parts in **Polymaker PolyLite ASA**,
and the calibration and tuning reflects that. This conversion adds chamber heater
help manage temperatures for ASA/ABS printing. The independent dual extruders
(**IDEX**) support dissolvable supports (HIPS) as well as two-color prints. In
addition, it can print two identical parts at once in COPY mode, or a mirrored
pair in MIRROR mode. Either head can also to emulate a single-tool printer. This
conversion is intended to maintain the FFCP2's mechanics while moving beyond the
stock FlashPrint workflow to Klipper macros and OrcaSlicer.

## Conversion at a Glance

- Octopus controller, six TMC2209 drivers, replacement hotend thermistors and
  heater block, and a Linux touchscreen host.
- Chamber heating and monitoring, filament sensors, NeoPixel lighting, a USB
  camera for monitoring the print, and a magnetic print-bed with a CryoGrip
  build-plate.
- Five print modes: IDEX, T0, T1, COPY, and MIRROR modes.  Selectable at job-start.

## Documentation

Read the guides in order:

| Guide                                                | What it covers                                                                       |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------ |
| [0. Design Decisions](doc/0-Design_Decisions.md)        | Goals, hardware choices, print modes, thermal-control tradeoffs, and future directions. |
| [1. Bill of Materials](doc/1-BOM.md)                   | Purchased parts, printed adapters, and workshop supplies.                             |
| [1a. Printed Parts — BOM Supplement](doc/1a-BOM-Printed-Parts-Supplement.md) | Part purposes, installation locations, photo placeholders, and printing considerations. |
| [2. Physical Conversion](doc/2-Physical_Conversion.md) | Controller installation, wiring references, safety, and staged first-power-on checks. |
| [3. Slicer Setup](doc/3-Slicer.md)                     | OrcaSlicer, the five print modes, printable areas, and start G-code.                   |
| [4. Calibration](doc/4-Calibration.md)                 | Heater PID, bed leveling, nozzle alignment, XY offsets, filament shrinkage, and skew.  |
| [9. Klipper G-Code Style Guide](doc/9-GCODE-Style-Guide.md) | Voron-informed formatting, macro-scoped documentation, state handling, and review standards. |

Guide 9 is a maintainer reference for writing and reviewing configuration changes.

**Repository layout:**

- [`config/`](config/) — Klipper configuration and macros. See the
  [configuration layout](#configuration-layout) for topic files and migration notes.
- [`cad/`](cad/) — CAD models and printable-part designs.
- [`doc/`](doc/) — Design, BOM, printed parts, conversion, slicer, calibration, and G-Code style guides.
- [`bin/`](bin/) — Host display-orientation scripts.

> **Build status and safety:** The physical-conversion guide is a rough draft,
> not a complete wiring manual. This is an advanced 3D printing project, and the
> reader should be comfortable with electrical safety and electrical fabrication
> techniques. Verify wiring and machine-specific calibration; do not copy saved
> PID or Z-endstop values blindly. The 120 V AC chamber heater means you need to
> be comfortable working with mains voltages as part of this project.

## Configuration Layout

[`config/printer.cfg`](config/printer.cfg) is the entry point. It directly
includes the hardware configuration and nine macro topic files, all in the same
`config/` directory. The former `Macros.cfg` has been removed; there is no
`config/macros/` subdirectory.

| File | Contents |
| --- | --- |
| [`printer.cfg`](config/printer.cfg) | Explicit includes, acknowledgements, machine tunables, shared print-mode state, tool-offset macros, and saved calibration. |
| [`Components.cfg`](config/Components.cfg) | Hardware definitions, motion limits, heaters, fans, lights, and filament sensors. |
| [`Motion.cfg`](config/Motion.cfg) | Homing, parking, and shared positioning helpers. |
| [`Toolheads.cfg`](config/Toolheads.cfg) | `T0`, `T1`, `TCOPY`, and `TMIRROR` tool/mode selection. |
| [`Temperature.cfg`](config/Temperature.cfg) | Nozzle-temperature helper, `M140`, `M141`, `M191`, and `CONTROLLED_COOLDOWN`. |
| [`Fans.cfg`](config/Fans.cfg) | Active-tool part cooling, `M106`, and `M107`. |
| [`Lighting.cfg`](config/Lighting.cfg) | Startup lighting and the continuous chamber LED status template. |
| [`Filament.cfg`](config/Filament.cfg) | Filament loading/unloading, nozzle cleaning, and priming. |
| [`Calibration.cfg`](config/Calibration.cfg) | The `Z_ENDSTOP_CALIBRATE` override. |
| [`Print_Start.cfg`](config/Print_Start.cfg) | All `START_PRINT*` entry points and their shared startup helper. |
| [`Print_Control.cfg`](config/Print_Control.cfg) | Pause, resume, idle timeout, print completion, and cancellation. |

Capitalized hardware/topic filenames distinguish repository-managed files from
local integration files in `printer_data/config`.  I symlink from
`printer_data/config` into my repo to keep everything consistent. Includes are
explicit rather than wildcard-based, so unrelated `.cfg` files are not loaded
automatically. I'm gradually migrating these files toward the
[Klipper G-Code style guide](doc/9-GCODE-Style-Guide.md), which draws on published
Voron configurations and community documentation while identifying local choices,
including macro-scoped comment blocks. I'm using LLM-based tools to deep-lint my
Klipper G-Code and to ensure a consistent style, while applying conventional
testing and review practices tailored for a homelab-style side-project.

**Updating an existing installation:** Deploy all repository `.cfg` files in
`config/` together, replacing the old `[include Macros.cfg]` with the direct
includes shown in `printer.cfg`. Preserve your machine-specific tunables and
`SAVE_CONFIG` calibration block rather than overwriting them with this build's
values. Keep the locally supplied `mainsail.cfg` and any optional integrations;
they are not provided by this repository. Macro names, slicer calls, and printer
behavior are unchanged by this reorganization.

## License

Copyright (C) 2025–2026 Luke Scharf and respective contributors.

This project's original configuration, macros, scripts, slicer profiles,
documentation, and CAD designs are licensed under the **GNU General Public
License, version 3 only** (`GPL-3.0-only`). You may redistribute and modify them
under those terms; see [LICENSE](LICENSE) for the full license text.

The project is provided **without warranty**, including any implied warranty of
merchantability or fitness for a particular purpose. The license is not a safety
certification; follow the build warnings and verify your own hardware.

Third-party material retains its applicable copyright notices and license terms.
This license does not relicense externally linked models, documentation, or other
resources. Preserve upstream notices when redistributing adapted material.

## Major Change Log

- **2026-11-17 — Topic-based configuration reorganization.** Split the former
  `Macros.cfg` into nine topic files directly under `config/` and moved their
  explicit includes and acknowledgements into `printer.cfg`. Applied Voron
  community-style formatting and macro descriptions to `printer.cfg`; retained
  machine settings, saved calibration, macro interfaces, and behavior. Updated
  documentation links and added the [configuration layout and migration
  notes](#configuration-layout).
- **2026-11-12 — Chamber temperature waits and heater duct.** Updated `M141`
  to wait for heating and conditionally for cooling, parking the active head
  and restoring G-code and dual-carriage state afterward. Cooling waits are
  skipped for targets below 30°C or more than 10°C above the measured bed
  temperature; `S0` turns the heater off without waiting. Added the 400 W PTC
  heater-duct FreeCAD model in [`cad/`](cad/).
- **2026-11-08–11 — Conversion documentation overhaul.** Expanded and reviewed
  the [BOM](doc/1-BOM.md), [physical-conversion draft](doc/2-Physical_Conversion.md),
  [OrcaSlicer setup](doc/3-Slicer.md), and [calibration guide](doc/4-Calibration.md).
  Clarified electrical-safety gaps, build-plate selection, the shared coordinate
  reference, and PolyLite ASA tuning, including filament-shrinkage profiling.
  Corrected the documented MIRROR width to 94.5 mm to match the macros and
  40 mm carriage-separation limit.
- **2026-11-05 — Startup priming lines.** Added configurable `PRIME_LINE`
  and called it after heating in T0, T1, COPY, and MIRROR startup. COPY/MIRROR
  prime both nozzles through synchronized motion; IDEX startup remains unchanged.
- **2026-11-04–11 — Pause and filament-runout handling.** Improved `PAUSE` /
  `RESUME` to preserve G-code and dual-carriage state, report the pause reason,
  and avoid overwriting saved state on repeated pause requests. Refined filament
  presence/motion notifications and preserved pause messages during sensor events.
- **2026-10-17–29 — Parking and bed-temperature transitions.** Tuned X travel
  and parking to keep priming stripes within bounds. Replaced the short-lived
  `LAYER_CHANGE` temperature checks with a blocking `M140` that parks the active
  head during bed-temperature waits to avoid heat damage to the model. November
  refinements added dual-carriage-state restoration around those waits.
- **2026-02-15–24 — Touchscreen mounting and service access.** Added and refined
  the parametric tablet mount and TPU service back-feet. Kept steppers enabled
  after `END_PRINT`, leaving motor shutdown to the idle timeout instead.
- **2026-02-01–24 — AC chamber heating and controlled cooldown.** Replaced the
  initial DC heater configuration with a 200 W AC design, then moved to a
  **400 W, 120 V AC PTC heater** on February 7. Added and refined `M141`, `M191`,
  chamber PID tuning, and `CONTROLLED_COOLDOWN` for a gradual bed/chamber
  temperature ramp. Refactored NeoPixel lighting to indicate heating and print
  status. See the [chamber-heater safety notes](doc/2-Physical_Conversion.md#120-v-ac-chamber-heater-wiring)
  before attempting the mains-powered heater installation.
- **2026-01-17–28 — Enclosure and host additions.** Added front-panel enclosure
  fan CAD, a chamber-thermistor mount reference, and display-rotation scripts for
  the Dell Chromebook touchscreen host. Introduced the first chamber-heater
  configuration: an 80 W, 24 V DC PTC design limited to protect the PSU budget
  (superseded by the AC heater).
- **2025-12-26–2026-01-15 — Five print modes and calibration.** Developed
  separate `START_PRINT_*` entry points for IDEX, T0, T1, COPY, and MIRROR, with
  corresponding OrcaSlicer areas and origins. Added `TCOPY`, `TMIRROR`, and a
  mode lock for single-tool and synchronized prints. Documented physical nozzle
  height matching with shims, Z-endstop and IDEX XY-offset calibration, and skew
  checks; refined homing and filament loading/unloading.
- **2025-12-18–27 — Configuration layout and peripheral mounts.** Separated
  hardware definitions (`Components.cfg`), macros (`Macros.cfg`), and machine
  tunables/saved calibration (`printer.cfg`). Added the combined NeoPixel /
  USB-camera adapter and replacement screen-panel plug CAD.
- **2025-12-03–11 — Initial Octopus/Klipper configuration.** Checked in the
  printer configuration and macros, plus Octopus mounting-rail CAD. Refined
  filament-runout sensing, extrusion and travel calibration, and nozzle
  cleaning/priming behavior.

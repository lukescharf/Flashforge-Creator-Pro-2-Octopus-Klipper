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
| [1. Bill of Materials](doc/1-BOM.md)                   | Purchased parts, printed adapters, and workshop supplies.                             |
| [2. Physical Conversion](doc/2-Physical_Conversion.md) | Controller installation, wiring references, safety, and staged first-power-on checks. |
| [3. Slicer Setup](doc/3-Slicer.md)                     | OrcaSlicer, the five print modes, printable areas, and start G-code.                   |
| [4. Calibration](doc/4-Calibration.md)                 | Heater PID, bed leveling, nozzle alignment, XY offsets, filament shrinkage, and skew.  |

Klipper configuration and macros are in [`config/`](config/); printable-part
designs are in [`cad/`](cad/).

> **Build status and safety:** The physical-conversion guide is a rough draft,
> not a complete wiring manual. This is an advanced 3D printing project, and the
> reader should be comfortable with electrical safety and electrical fabrication
> techniques. Verify wiring and machine-specific calibration; do not copy saved
> PID or Z-endstop values blindly. The 120 V AC chamber heater requires qualified
> electrical work and independent overtemperature protection; do not energize it
> from this draft alone.

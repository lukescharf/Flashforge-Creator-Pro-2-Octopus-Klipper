# BOM for Flashforge Creator Pro 2 → FFCP2+OK Conversion

Parts list for converting the Flashforge Creator Pro 2 into the modernized
Klipper-based FFCP2+OK.

## Parts Required

Printed parts are called out next to the purchased parts they support. Printable
files are in [`../cad/`](../cad/). Print rigid brackets in ABS or ASA; PETG is a
reasonable substitute for external mounts and cable clips. The service-mode foot
is designed for TPU.

### Mainboard

- [BIGTREETECH Octopus Pro v1.1 H723][octopus-pro] (qty: 1).
  - Mounting Rails (printed):
    `FFCP2 Octopus Pro v1.1 Mounting Rails-Body.step` (qty: 2). Adapter rails
    that mate the Octopus Pro v1.1 to the existing FFCP2 case holes. Requires
    4× M3 heat-set inserts (2 per rail), 4× M3 screws to attach the rails to
    the case, and 4× M3 screws to attach the board to the rails.
- [BIGTREETECH TMC2209 V1.3 Stepper Motor Driver][tmc2209] (qty: 6).
- [M3 Cable Mount P-Clips, 10 mm][p-clips] (printed; qty: as needed). For
  securing the new wiring with M3 screws.

### Klipper Pad

- Dell Chromebook 11.6" 2-in-1 Touch Laptop, Celeron, 4 GB RAM, 32 GB SSD,
  running Debian 13. Nearly any Linux computer with a touchscreen and
  Raspberry Pi 3B (or better) specs should work. Used touchscreen Chromebooks
  modified with the MrChromebook BIOS are an under-appreciated option as
  Klipper Pads go.
- Tablet Mount (printed): `FFCP2 Tablet Mount.FCStd` (qty: 1). Attaches to the
  existing 150 mm-spaced bracket holes. Edit the `params` spreadsheet for your
  tablet and export an STL; no STEP file is included.

### Hotend Compatibility with Octopus Pro v1.1

- [Micro Swiss Plated Copper High Temperature MK10 Heater Block Upgrade
  (M7 Threads)][heater-block] (qty: 2).
- [Ender-3-style thermistors][thermistors] (qty: 2).

### Lighting Compatibility with Octopus Pro v1.1

The existing LEDs can work with Octopus, but use up three fan PWM ports.
Replacing them with a NeoPixel device is just easier.

- [Adafruit Industries NeoPixel Stick — 8 × 5050 RGBW LEDs][neopixel] (qty: 2).
- [DuPont-style Header Pins, 2.54 mm][header-pins] (requires some soldering).
- NeoPixel + Camera Mounting Adapter (printed):
  `FFCP2 Neopixel + Camera Mounting Adapter-Body.step` (qty: 1). Mounts both
  NeoPixel sticks and the camera. Requires 6× M2 screws for the lights and
  camera and 2× M3 screws to attach the bracket to the printer.

### FFCP2 Screen Removal

- Screen Panel Plug (printed): `FFCP2 Screen Panel Plug-Body.step` (qty: 1).
  Fills the opening left by the removed stock screen; includes
  "Octopus Pro V1.1" lettering and needs no additional hardware.

### Camera

- [SVPRO 8MP USB Camera Module][camera] (qty: 1).
- Mounting Bracket (printed): uses the
  `FFCP2 Neopixel + Camera Mounting Adapter-Body.step` listed under
  [Lighting Compatibility with Octopus Pro v1.1][lighting-section].

### Filament Sensors

- [BIGTREETECH Smart Filament Detector V2.0][filament-sensor] (qty: 2).
- [PC6 PTFE Tubing, 6 mm OD × 4 mm ID][ptfe-tubing] (qty: 2 m).
- [PC6-M6 Pneumatic Fitting][pneumatic-fitting] (qty: 2).

### Chamber Heater and Monitoring

- [120 V AC PTC chamber heater][chamber-heater]
  (qty: 1).
- [Solid-state relay (SSR)][chamber-ssr] (qty: 1). The relay used in this
  build to switch the 120 V AC PTC heater. Verify the actual unit's control-input
  requirements, AC load rating, derating and heatsink requirements before wiring.
- [Chamber thermistor (NTC 100K 3950)][thermistors] (qty: 1).
- [Voron Chamber Thermistor Mount][thermistor-mount] (printed; qty: 1). Mounts
  the chamber thermistor to the printer's 2020 extrusion; the
  `voron_champber_thermister.link` file in `../cad/` points to the model.

### PEI Print Bed Upgrade

- [230 × 150 mm PEI Build Plate with magnetic base][pei-plate] (qty: 1).
- [BIQU CryoGrip Glacier, 220 × 220 mm][cryogrip] (for Adventurer 5M, Ender 3,
  Ender 3 V2, Ender 3 Pro, Ender 3S, CR20, Ender 5, and Sovol SV05). This fits
  the FFCP2 reasonably well and is a great surface to print on.

## Cable Fabrication and Workshop Supplies

> **Note:** I keep these in stock by my electronics workbench and didn't need to
> purchase them for this project, but these parts are necessary for completing
> the project.

- JST-XH Connector Kit, 2.54 mm, 2/3/4/5/6-pin housings.
- DuPont-style Connector Kit, 2.54 mm.
- DuPont-style Locking Connector Kit, 2.54 mm.
- Standard Crimp-On Ring Terminals (qty: as needed). Used for the 120 V AC
  heater wiring; select terminals rated for the circuit's current, wire gauge,
  stud size and insulation requirements, and use the appropriate crimping tool.
- M3 Heat-Set Inserts (brass, knurled), assortment.
- M3 Screws, socket head cap assortment with nuts and washers.
- M3 Shim Washers, 6 mm OD, assorted thicknesses including 0.2 mm (qty: as
  needed). For physically matching nozzle heights during
  [IDEX Z-Offset Calibration](4-Calibration.md#idex-z-offset-calibration);
  ordinary washers are not a substitute for precision shims.
- Zip Ties, assorted.

## Service and Maintenance

- `FFCP2 Service Mode Back-Foot.3mf` (printed; qty: 4): TPU feet that let the
  printer rest safely on its back for service. The 3MF contains one foot, so
  print four; it is sliced at 0.2 mm layers, 3 walls, 25% infill, and a
  7 mm brim.

[lighting-section]: #lighting-compatibility-with-octopus-pro-v11
[octopus-pro]: https://biqu.equipment/products/bigtreetech-octopus-pro-v1-0-chip-f446
[tmc2209]: https://biqu.equipment/products/btt-tmc2209-stepper-driver
[p-clips]: https://www.printables.com/model/1117426-m3-cable-mount-p-clip-10-sizes-2-10mm
[heater-block]: https://store.micro-swiss.com/products/micro-swiss-plated-copper-high-temperature-mk10-heater-block-upgrade
[thermistors]: https://www.amazon.com/dp/B09WDR1M5Q
[neopixel]: https://www.adafruit.com/product/2869
[header-pins]: https://www.amazon.com/dp/B07PKKY8BX
[camera]: https://www.amazon.com/dp/B0DCF8WW6V
[filament-sensor]: https://biqu.equipment/products/btt-sfs-v2-0-smart-filament-sensor
[ptfe-tubing]: https://www.amazon.com/dp/B0DKBLJDX1
[pneumatic-fitting]: https://www.amazon.com/dp/B0C6GKKQKL
[chamber-heater]: https://www.amazon.com/dp/B0F62STF3M
[chamber-ssr]: https://www.amazon.com/dp/B08GPJ1V2J
[thermistor-mount]: https://www.printables.com/model/549305-voron-chamber-thermistor-mount
[pei-plate]: https://www.amazon.com/dp/B09MLNJR8D
[cryogrip]: https://biqu.equipment/products/biqu-cryogrip-pro-glacier-build-plate

BOM for Flashforge Creator Pro 2 -> FFCP2+OK Conversion
-------------------------------------------------------
Parts-list for converting the Flashforge Creator Pro 2 into the the modernized Klipper-based FFCP2+OK.

Parts Required
========================
Printed parts are called out next to the purchased parts they support.  Printable files are in [`../cad/`](../cad/).  Print rigid brackets in ABS or ASA; PETG is a reasonable substitute for external mounts and cable clips.  The service-mode foot is designed for TPU.

+ Mainboard:
  + BIGTREETECH Octopus Pro v1.1 H723: https://biqu.equipment/products/bigtreetech-octopus-pro-v1-0-chip-f446 (qty: 1)
    + Mounting Rails (printed): `FFCP2 Octopus Pro v1.1 Mounting Rails-Body.step` (qty: 2).  Adapter rails that mate the Octopus Pro v1.1 to the existing FFCP2 case holes.  Requires 4x M3 heat-set inserts (2 per rail), 4x M3 screws to attach the rails to the case, and 4x M3 screws to attach the board to the rails.
  + BIGTREETECH TMC2209 V1.3 Stepper Motor Driver: https://biqu.equipment/products/btt-tmc2209-stepper-driver (qty: 6)
  + M3 Cable Mount P-Clips, 10mm (printed; qty: as needed): https://www.printables.com/model/1117426-m3-cable-mount-p-clip-10-sizes-2-10mm.  For securing the new wiring with M3 screws.
+ Klipper Pad:
  + Dell Chromebook 11.6" 2-in-1 Touch Laptop Celeron 4GB Ram 32GB SSD running Debian 13, but nearly any Linux computer with a touchscreen and Raspberry Pi 3B (or better) specs should work.  Used touchscreen Chromebooks modified with the MrChromebook BIOS are an under-appreciated option as Klipper Pads go.
  + Tablet Mount (printed): `FFCP2 Tablet Mount.FCStd` (qty: 1).  Attaches to the existing 150mm-spaced bracket holes.  Edit the `params` spreadsheet for your tablet and export an STL; no STEP file is included.
+ Hotend compatibility with Octopus Pro v1.1:
  + Microswiss Plated Copper High Temperature MK10 Heater Block Upgrade (M7 Threads): https://store.micro-swiss.com/products/micro-swiss-plated-copper-high-temperature-mk10-heater-block-upgrade (qty: 2)
  + Ender-3-syle thermisters: https://www.amazon.com/dp/B09WDR1M5Q (qty: 2)
+ Lighting Compatibility with Octopus Pro V1.1:
  The existing LEDs can work with Octopus, but it uses up three fan PWM ports.  Replacing it with a Neopixel device is just easier.
  + Adafruit Industries NeoPixel Stick - 8 x 5050 RGBW LEDs: https://www.adafruit.com/product/2869 (qty: 2)
  + DuPont-style Header Pins 2.54mm: https://www.amazon.com/dp/B07PKKY8BX (requires some soldering)
  + NeoPixel + Camera Mounting Adapter (printed): `FFCP2 Neopixel + Camera Mounting Adapter-Body.step` (qty: 1).  Mounts both NeoPixel sticks and the camera.  Requires 6x M2 screws for the lights and camera and 2x M3 screws to attach the bracket to the printer.
+ FFCP2 Screen Removal:
  + Screen Panel Plug (printed): `FFCP2 Screen Panel Plug-Body.step` (qty: 1).  Fills the opening left by the removed stock screen; includes "Octopus Pro V1.1" lettering and needs no additional hardware.
+ Camera:
  + SVPRO 8MP USB Camera Module: https://www.amazon.com/dp/B0DCF8WW6V (qty: 1)
  + Mounting Bracket (printed): uses the `FFCP2 Neopixel + Camera Mounting Adapter-Body.step` listed under Lighting Compatibility.
+ Filament Sensors:
  + BIGTREETECH Smart Filament Detector V2.0: https://biqu.equipment/products/btt-sfs-v2-0-smart-filament-sensor (qty: 2)
  + PC6 PTFE Tubing 6mm OD x 4mm ID: https://www.amazon.com/dp/B0DKBLJDX1 (qty: 2m)
  + PC6-M6 Pneumatic Fitting: https://www.amazon.com/dp/B0C6GKKQKL (qty: 2)
+ Chamber Heater & Monitoring:
  + Chamber heater (3D printer enclosure/chamber heater): https://www.amazon.com/dp/B0F62STF3M (qty: 1)
  + Chamber thermistor (NTC 100K 3950) (qty: 1): https://www.amazon.com/dp/B09WDR1M5Q
  + Voron Chamber Thermistor Mount (printed; qty: 1): https://www.printables.com/model/549305-voron-chamber-thermistor-mount.  Mounts the chamber thermistor to the printer's 2020 extrusion; the `voron_champber_thermister.link` file in `../cad/` points to the model.
+ PEI Print Bed Upgrade:
  + 230x150mm PEI Build Plate with magnetic base: https://www.amazon.com/dp/B09MLNJR8D (qty: 1)
  + BIQU CryoGrip Glacier 220mm*220mm (for Adventurer 5M/Ender 3/V2/Pro/Ender 3S/CR20/Ender 5/Sovol SV05): https://biqu.equipment/products/biqu-cryogrip-pro-glacier-build-plate.  This fits the FFCP2 reasonably well, and is a great surface to print on.

Cable Fabrication & Workshop Supplies:  
  *Note: I keep these in stock by my electronics workbench and didn't need to purchase them for this project, but these parts are necessary for completing the project.*
  + JST-XH Connector Kit 2.54mm 2/3/4/5/6 Pin Housing
  + DuPont-style Connector Kit 2.54mm
  + DuPont-style Locking Connector Kit 2.54mm
  + M3 Heat-Set Inserts (brass, knurled), assortment
  + M3 Screws, socket head cap assortment with nuts and washers
  + Zip Ties, assorted
  

+ Service & Maintenance:
  + FFCP2 Service Mode Back-Foot.3mf (printed; qty: 2): TPU feet that let the printer rest safely on its back for service.  The 3MF contains one foot, so print a pair; it is sliced at 0.2mm layers, 3 walls, 25% infill, and a 7mm brim.

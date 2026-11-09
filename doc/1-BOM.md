BOM for Flashforge Creator Pro 2 -> FFCP2+OK Conversion
-------------------------------------------------------
Parts-list for converting the Flashforge Creator Pro 2 into the the modernized Klipper-based FFCP2+OK.

Parts Required
========================
Printed parts are called out next to the purchased parts they support.  Printable files are in [`../cad/`](../cad/).  Print rigid brackets in ABS or ASA; PETG is a reasonable substitute for external mounts and cable clips.  The service-mode foot is designed for TPU.

+ Flashforge Creator Pro 2 Mainboard Upgrade:
  + BIGTREETECH Octopus Pro v1.1 H723: https://amzn.to/4b8VYAA (qty: 1)
    + Mounting Rails (printed): `FFCP2 Octopus Pro v1.1 Mounting Rails-Body.step` (qty: 2).  Adapter rails that mate the Octopus Pro v1.1 to the existing FFCP2 case holes.  Requires 4x M3 heat-set inserts (2 per rail), 4x M3 screws to attach the rails to the case, and 4x M3 screws to attach the board to the rails.
  + BIGTREETECH TMC2209 V1.3 Stepper Motor Driver: https://amzn.to/4971j8M (qty: 6)
  + Klipper Pad: Dell Chromebook 11.6" 2-in-1 Touch Laptop Celeron 4GB Ram 32GB SSD running Debian 13, but nearly any Linux computer with a touchscreen and Raspberry Pi 3B (or better) specs should work.  Used touchscreen Chromebooks modified with the MrChromebook BIOS are an under-appreciated option as Klipper Pads go.  https://www.ebay.com/itm/236058833748
    + Tablet Mount (printed): `FFCP2 Tablet Mount.FCStd` (qty: 1).  Attaches to the existing 150mm-spaced bracket holes.  Edit the `params` spreadsheet for your tablet and export an STL; no STEP file is included.
  + Hotend compatibility with Octopus Pro v1.1:
    + Microswiss Plated Copper High Temperature MK10 Heater Block Upgrade (M7 Threads): https://amzn.to/4skQPvj (qty: 2)
    + Ender-3-syle thermisters: https://amzn.to/49eT4HY (qty: 2)
  + Lighting Compatibility with Octopus Pro V1.1:
    The existing LEDs can work with Octopus, but it uses up three fan PWM ports.  Replacing it with a Neopixel device is just easier.
    + Adafruit Industries NeoPixel Stick - 8 x 5050 RGBW LEDs: https://amzn.to/4pYJRul (qty: 2)
    + DuPont-style Header Pins 2.54mm: https://amzn.to/494a0lO (requires some soldering)
    + NeoPixel + Camera Mounting Adapter (printed): `FFCP2 Neopixel + Camera Mounting Adapter-Body.step` (qty: 1).  Mounts both NeoPixel sticks and the camera.  Requires 6x M2 screws for the lights and camera and 2x M3 screws to attach the bracket to the printer.
  + Screen Panel:
    + Choose one screen-opening cover (printed):
      + `FFCP2 Screen Panel Plug-Body.step` (qty: 1): fills the opening left by the removed stock screen; includes "Octopus Pro V1.1" lettering and needs no additional hardware.
      + `FFCP2 Screen Panel Enclosure Fan-Body.step` (qty: 1): alternative to the plug with two ducted enclosure fans; requires M3 and M4 heat-set inserts (4 of each) plus the fans.
  + Camera:
    + SVPRO 8MP USB Camera Module: https://amzn.to/4slLW5e (qty: 1)
    + Mounting Bracket (printed): uses the `FFCP2 Neopixel + Camera Mounting Adapter-Body.step` listed under Lighting Compatibility.
  + Filament Sensors:
    + BIGTREETECH Smart Filament Detector V2.0: https://amzn.to/4jinaih (qty: 2)
    + PC6 PTFE Tubing 6mm OD x 4mm ID: https://amzn.to/44Txdoa (qty: 2m)
    + PC6-M6 Pneumatic Fitting: https://amzn.to/3LnfDCm (qty: 2)
  + PEI Print Bed Upgrade:
    + 230x150mm PEI Build Plate with magnetic base: https://amzn.to/4sCAeUf (qty: 1)
    + 230x150mm holographic build-plate: https://amzn.to/4qACtFz (cheap and good, despite looking like a novelty) 
    + BIQU CryoGrip Glacier 220mm*220mm(For Adventurer 5M/Ender 3/V2/Pro/Ender 3S/CR20/Ender 5/Sovol SV05).
      This fits the FFCP2 reasonably well, and is a great surface to print on.

Cable Fabrication & Workshop Supplies:  
  *Note: I keep these in stock by my electronics workbench and didn't need to purchase them for this project, but these parts are necessary for completing the project.*
  + JST-XH Connector Kit 2.54mm 2/3/4/5/6 Pin Housing: https://amzn.to/4ssbjCH
  + DuPont-style Connector Kit 2.54mm: https://amzn.to/3L6rbKf
  + DuPont-style Locking Connector Kit 2.54mm: https://amzn.to/3KV4wR7
  + Zip Ties
  + M3 Cable Mount P-Clips, 10mm (printed; qty: as needed): https://www.printables.com/model/1117426-m3-cable-mount-p-clip-10-sizes-2-10mm.  For securing the new wiring with M3 screws; the `m3-pclip-10mm.link` file in `../cad/` points to the model.

+ Optional Chamber Monitoring:
  + Voron Chamber Thermistor Mount (printed; qty: 1): https://www.printables.com/model/549305-voron-chamber-thermistor-mount.  Mounts the chamber thermistor to the printer's 2020 extrusion; the `voron_champber_thermister.link` file in `../cad/` points to the model.

+ Service & Maintenance:
  + FFCP2 Service Mode Back-Foot.3mf (printed; qty: 2): TPU feet that let the printer rest safely on its back for service.  The 3MF contains one foot, so print a pair; it is sliced at 0.2mm layers, 3 walls, 25% infill, and a 7mm brim.

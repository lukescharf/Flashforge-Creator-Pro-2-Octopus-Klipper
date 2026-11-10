Physical Conversion: Flashforge Creator Pro 2 -> FFCP2+OK
=========================================================

This is a **rough build draft**, not a pin-by-pin harness drawing. It describes the physical changes behind the configuration in [`../config/`](../config/) and the parts in the [BOM](1-BOM.md). This build keeps the Creator Pro 2's IDEX mechanics (two independent X carriages), Y gantry, Z bed, and much of its wiring, but replaces the stock controller with a BIGTREETECH Octopus Pro **v1.1, STM32H723**, running Klipper. T0 is the **left** head and T1 is the **right** head throughout this repository. Do not assume another FFCP2 or Octopus revision has the same connector order, voltage, or wire colors.

This conversion and writeup build on **[Evil Azrael's Flashforge Creator Pro 2 documentation](https://wiki.evilazrael.de/en/flashforge-creator-pro-2)** and his FFCP2 Klipper configuration. His reverse-engineering and explanations were an important reference during this project. Other sources are credited where used and in [Acknowledgements and references](#acknowledgements-and-references) below.

**BTT board references:** [Octopus Pro v1.1 pinout](https://github.com/bigtreetech/BIGTREETECH-OCTOPUS-Pro/blob/master/Hardware/BIGTREETECH%20Octopus%20Pro%20V1.1-Pin.jpg), [BTT Octopus Pro product manual (English PDF)](https://github.com/bigtreetech/BIGTREETECH-OCTOPUS-Pro/blob/master/BTT_Octopus_pro_EN.pdf), and [BTT's v1.1 revision notes](https://github.com/bigtreetech/BIGTREETECH-OCTOPUS-Pro#octopus-pro-v11). **The PDF describes v1.0**, not v1.1; use the v1.1 pinout and revision notes for actual v1.1 wiring and pin assignments. BTT does not appear to publish a separate v1.1 English PDF manual in that repository.

> **Electrical safety:** Unplug mains and USB, wait for the power supply to discharge, and verify the printer is de-energized before opening it. Photograph and label every stock connector *before* unplugging it. Use the board manufacturer's **v1.1** pinout and a meter to identify each wire; never use connector shape or color as proof of function. Keep protective earth bonded to the chassis and keep mains wiring segregated and enclosed. Have a qualified person handle any mains, PSU, or chamber-heater/SSR work. Do not power heaters during the first board/firmware bring-up.

## Prerequisites: Before taking the printer apart

1. Read the [BOM](1-BOM.md), inventory the printed adapters in [`../cad/`](../cad/), and print the mounts while the stock printer still works. Print four TPU `FFCP2 Service Mode Back-Foot.3mf` feet if you plan to rest the machine on its back; support it so the gantry, lid, and cables cannot take the load.
2. Take photos of the stock board, supply/earth wiring, toolhead cable routing, and every intermediate connector. Draw a harness map from each signal to its eventual Octopus port. Save the original electronics and screws in labeled bags; this conversion should not depend on a guess about which identical-looking plug belonged to T0.

## Background: Configured travel and thermal limits

These are the values in [`../config/Components.cfg`](../config/), documented here to help check the physical conversion. They are **configuration values for this particular build**, not measurements or safe operating limits for every FFCP2. Verify endstops, travel, head alignment, head separation, temperature sensors, and component ratings on your own machine before relying on them. The slicer's mode-dependent printable areas are smaller or differently positioned than these individual axis ranges; see [Slicer Setup](3-Slicer.md).

### Background: Motion and travel

The two X carriages do not have identical reach over the bed: there is an overlapping region both nozzles can print in, plus regions reachable by only one nozzle. [Evil Azrael's FFCP2 bed-overlap diagram](https://wiki.evilazrael.de/ffcp2/print_regions.png) illustrates this geometry; see his [accompanying explanation](https://wiki.evilazrael.de/en/flashforge-creator-pro-2) and [Frank Paynter's FFCP2 Klipper writeup](https://www.fpaynter.com/2024/10/klipperizing-my-flashforge-creator-pro-2-3d-printer/) for related conversion background.

For this conversion, I put the XY origin **(0,0) at the lower-left (front-left) corner of the area that both T0 and T1 can reach**, not at the physical build-plate corner or either carriage's homing position. All five print modes adapt to this shared reference. I use the slicer and **five separate logical printer profiles** to handle the different reachable regions and head-clearance constraints: **T0 only, T1 only, IDEX/toolswapping, COPY, and MIRROR**. Each profile pairs the appropriate printable area and origin offset with the corresponding start macro, rather than treating the entire carriage travel as printable bed area. [Slicer Setup — Five Print Modes](3-Slicer.md#five-print-modes) describes these profiles and their dimensions. In particular, IDEX needs the common reachable area, COPY needs room for two offset copies, and MIRROR must keep the heads clear as they approach one another.

| Axis / carriage | Configured range | Endstop position | Homing speed |
| --- | --- | --- | --- |
| T0 / left X (`stepper_x`) | -43 to 200 mm | -38 mm | 50 mm/s; 7 mm retract |
| T1 / right X (`dual_carriage`) | -3 to 242 mm | 237 mm | 50 mm/s; 7 mm retract |
| Y | -13 to 150 mm | -10 mm | 50 mm/s |
| Z | -5 to 170 mm | Calibrated per machine; `printer.cfg` currently has a saved example of -4.030 mm | 15 mm/s |

The `dual_carriage` section sets `safe_distance: 40` mm between the carriages. It is a software constraint, not a substitute for checking actual physical clearance, including the heads, cables, and parked positions. The X ranges are **individual carriage limits**, not a promise that both heads can reach every coordinate simultaneously. Negative X/Y coordinates are measured from the shared reachable area's origin; depending on the carriage and mode, they can include bed area outside the overlap as well as parking or homing travel. A negative coordinate alone does not establish whether a location is printable.

Overall motion limits are `max_velocity: 200` mm/s and `max_accel: 1500` mm/s², with Z separately limited to `max_z_velocity: 30` mm/s and `max_z_accel: 100` mm/s². The configured homing speeds above may be too fast for initial commissioning; verify direction and endstop operation first and use conservative settings while testing. Calibrate Z on the converted machine rather than copying the saved example.

### Background: Temperature limits

| Heater / sensor | Sensor type in config | `min_temp` | `max_temp` |
| --- | --- | --- | --- |
| T0 / left hotend (`extruder`) | Generic 3950 | 0°C | 275°C |
| T1 / right hotend (`extruder1`) | Generic 3950 | 0°C | 275°C |
| Heated bed | ATC Semitec 104GT-2 | 0°C | 130°C |
| Chamber heater sensor | Generic 3950 | 0°C | 65°C |

Klipper uses these bounds to check sensor readings and restrict heater targets; exceeding a configured sensor bound can trigger shutdown. They are **not recommended print temperatures** or proof that a particular thermistor, heater block, cartridge, wiring, bed surface, chamber component, or printed mount is rated to those temperatures. Select targets within the lowest applicable component rating, and verify that each sensor type matches the installed sensor. The hotend heatbreak fans are configured to turn on at 50°C. The `mcu_fan` also monitors MCU temperature with configured bounds of 0–70°C and a 45°C target; it is not a chamber or heater safety cutoff.

Do not rely on Klipper temperature limits alone for chamber-heater safety: the independent protection and electrical design described below still need to be established.


## Fit the controller and prepare the hotends

1. With power disconnected, open the electronics bay and remove the stock screen and controller only after labeling and documenting their connections. Preserve the stock PSU and mains inlet arrangement unless you have independently verified a change is necessary. Do not trap wiring beneath the new board.
2. Fit four M3 heat-set inserts into the two printed `FFCP2 Octopus Pro v1.1 Mounting Rails-Body.step` rails (two per rail). Attach the rails to the existing case holes with four M3 screws, then fasten the Octopus Pro with another four. Check standoff height, underside clearance, screw length, airflow, and that no conductive surface touches the board. Reinstall an electronics-bay fan with a clear air path to the drivers; `Components.cfg` controls an MCU fan from board temperature.
3. Fit six TMC2209 drivers in the sockets used below, in the orientation indicated by the **board and driver** markings. Set each socket for the **main board supply**, not the Octopus Pro's high-voltage MOTOR_POWER input: ordinary TMC2209 modules are not high-voltage drivers. Set UART jumpers for TMC2209 operation and leave sensorless-homing DIAG jumpers off when using physical endstops. Verify the exact jumper layout against the BTT manual and v1.1 pinout linked above before powering it.
4. The BOM replaces the two MK10 heater blocks with Micro Swiss M7-thread blocks and the stock hotend thermocouples with two Generic 3950-style thermistors. Assemble each hotend according to its component instructions; seat and retain each thermistor securely without crushing or shorting its leads, and protect heater/thermistor wires from the block and moving carriage. Confirm the actual heater cartridge voltage and resistance before reusing it. The stock bed sensor is configured separately as `ATC Semitec 104GT-2`; don't silently swap it to the hotend sensor type.
5. Fit the printed `FFCP2 Screen Panel Plug-Body.step` in the vacated front display opening. Mount the touchscreen host with `FFCP2 Tablet Mount.FCStd` if desired: its FreeCAD `params` spreadsheet needs adapting to your device before exporting a printable mesh. The host is not powered from an unverified board header.

## Rebuild the low-voltage harness

The MCU-pin tables are **configuration cross-references**, not physical plug-orientation diagrams. The breakout connector table below is a separate, photo-derived reference with its own orientation convention. `../config/Components.cfg` gives MCU pin names; the BTT **v1.1 H723** pinout identifies the corresponding sockets and connector polarity. T0/left is Klipper `extruder`; T1/right is `extruder1`. Route each harness with strain relief, insulation, and enough slack for all travel; keep thermistor/signal leads clear of heater and motor wiring where practical. Re-terminate with appropriate housings and crimped contacts rather than forcing a stock plug into a different pin order.

### Print-head breakout boards and thermistors

The conversion **reuses the temperature-sensor connections on each stock print-head breakout board** for the replacement thermistors. The original hotend sensors were **thermocouples**, as described in [Evil Azrael's writeup](https://wiki.evilazrael.de/en/flashforge-creator-pro-2); the reused connector labels `T+` and `T−` come from that stock arrangement. Each replacement sensor is carried through its breakout and harness to the corresponding Octopus temperature input: T0/left to T0 (PF4 and GND), T1/right to T1 (PF5 and GND). These are Octopus MCU pin names, **not** pin numbers on the Flashforge breakout. The end-to-end harness details below still need to be recorded at the workbench; do not use the placeholders as wiring instructions.

#### Stock eight-pin breakout connector

The following labels are readable in Partsbuilt 3D's product photos of the [left board](https://partsbuilt.com/extruder-pcb-board-left-side-creator-pro-2-flashforge/) and [right board](https://partsbuilt.com/extruder-pcb-board-right-side-creator-pro-2-flashforge/). Both show the same sequence when counted from the end marked **Left Extruder** or **Right Extruder** toward the green heater screw terminal. 3D Prima's [high-resolution right-board photo](https://cdn03.plentyone.com/ioseuwg7moqp/item/images/25686/full/Flashforge-Creator-Pro-2-Right-Extruder-Board-30-001560002-25686.png) also identifies the small sockets as **24V FAN**, **5V FAN**, and **TEMP**, with TEMP nearest the green heater terminal.

**Orientation:** View the board from the **component side** and follow the row of eight harness pins from the end marked Left/Right Extruder toward the green heater terminal. Positions below are **indices assigned for this guide**, not verified Flashforge pin numbers and not a view into the mating cable plug. Check the markings and continuity on your actual board before wiring it.

| Position along row | Printed label | Function |
| --- | --- | --- |
| 1 | `24+` | 24 V fan positive |
| 2 | `24−` | 24 V fan negative/control return |
| 3 | `5+` | 5 V fan positive |
| 4 | `5−` | 5 V fan negative/control return |
| 5 | `T+` | Temperature-sensor conductor |
| 6 | `T−` | Temperature-sensor conductor |
| 7–8 | `HEAT` across both contacts | Heater cartridge pair; individual polarity is not specified by this label |

**Do not assume the fan minus contacts are permanent ground:** they may be switched returns. Preserve each fan's separate circuit rather than tying its minus contact to sensor ground. The photo-derived labels do not establish hidden shared traces, cable wire colors, or the pin order at the controller end.

For component-rating context, [Evil Azrael's component list](https://wiki.evilazrael.de/en/flashforge-creator-pro-2) identifies a **24 V, 0.08 A** hotend fan and a **5 V, 0.25 A** part fan. [Spool3D lists the FFCP2 replacement heater cartridge](https://spool3d.ca/flashforge-creator-pro-2-heater-cartridge/) as **24 V, 40 W** (nominal calculated resistance 14.4 Ω), and [3D Print Beginner's teardown](https://3dprintbeginner.com/flashforge-creator-pro-2-review/) identifies a **24 V, 320 W Delta DPS-320AB-1 A** PSU on its review machine. These are published references, not measurements of this conversion: verify the actual components and power budget at the workbench.

With mains and USB disconnected and power verified off, unplug the relevant connectors and trace both sensor conductors from the thermistor connector, through the breakout and harness, to the Octopus input using a multimeter. Document the connector latch/viewing direction and check both heads separately. Verify the sensor circuits are isolated from heater and fan power, and check for shorts or damaged insulation. Before heating, confirm both sensors read plausibly near room temperature and that warming each sensor changes only its corresponding Klipper reading. A 3950 thermistor itself is non-polarized, but the board and harness may have shared connections: verify the full circuit before connecting it to the Octopus.

**FIXME:** Add photos of both sides of each breakout and both ends of its harness, connector orientation and pin numbering, and any shared traces or differences between left and right. Also map each breakout's heater and both fan connections, including polarity and measured fan voltages, before documenting or changing those circuits.

### T0: left carriage and print head

| Function | Klipper section / pin(s) in `Components.cfg` | Build check |
| --- | --- | --- |
| Left X motor / driver | `stepper_x`: PF13, !PF12, !PF14; UART PC4 | Left carriage, physical X endstop PG6 |
| T0/left extruder motor / driver | `extruder`: PG4, PC1, !PA2; UART PC7 | MOTOR3 in this build |
| T0 hotend heater | `extruder` PA0 (HE0) | Match to T0 sensor; leave load disconnected until firmware and sensing are verified |
| T0 hotend thermistor | PF4 (T0) | Configured `Generic 3950`; verify actual replacement probe |
| T0 heatbreak fan | PA8 | `heater_fan` output, on above 50°C; verify fan voltage/polarity |
| T0 part-cooling fan | PE5 | `fan_generic` output; verify fan voltage/polarity |
| T0 filament presence / motion | PG12 / PG13 | Verify SFS V2.0 supply, signal wiring and both functions |

**T0 thermistor path — workbench record:**

| Connection to document | Pin or observation |
| --- | --- |
| Breakout sensor connector, conductor A | **FIXME:** connector and pin |
| Breakout sensor connector, conductor B | **FIXME:** connector and pin |
| Breakout main-harness connector, conductor A | `T+`, position 5 in the photo-derived table; **FIXME:** verify continuity on this board |
| Breakout main-harness connector, conductor B | `T−`, position 6 in the photo-derived table; **FIXME:** verify continuity on this board |
| Harness controller end, conductor A | **FIXME:** connector, pin and wire color |
| Harness controller end, conductor B | **FIXME:** connector, pin and wire color |
| Octopus temperature input and return | T0 connector: PF4 and GND, per BTT v1.1 pinout; **FIXME:** record which harness conductor reaches each contact |

### T1: right carriage and print head

| Function | Klipper section / pin(s) in `Components.cfg` | Build check |
| --- | --- | --- |
| Right X motor / driver | `dual_carriage`: PC13, PF0, !PF1; UART PE4 | Right carriage, physical X endstop PG11; keep both heads apart during tests |
| T1/right extruder motor / driver | `extruder1`: PF9, !PF10, !PG2; UART PF2 | MOTOR4 in this build |
| T1 hotend heater | `extruder1` PA3 (HE1) | Match to T1 sensor; leave load disconnected until firmware and sensing are verified |
| T1 hotend thermistor | PF5 (T1) | Configured `Generic 3950`; verify actual replacement probe |
| T1 heatbreak fan | PD12 | `heater_fan` output, on above 50°C; verify fan voltage/polarity |
| T1 part-cooling fan | PD13 | `fan_generic` output; verify fan voltage/polarity |
| T1 filament presence / motion | PG14 / PG15 | Verify SFS V2.0 supply, signal wiring and both functions |

**T1 thermistor path — workbench record:**

| Connection to document | Pin or observation |
| --- | --- |
| Breakout sensor connector, conductor A | **FIXME:** connector and pin |
| Breakout sensor connector, conductor B | **FIXME:** connector and pin |
| Breakout main-harness connector, conductor A | `T+`, position 5 in the photo-derived table; **FIXME:** verify continuity on this board |
| Breakout main-harness connector, conductor B | `T−`, position 6 in the photo-derived table; **FIXME:** verify continuity on this board |
| Harness controller end, conductor A | **FIXME:** connector, pin and wire color |
| Harness controller end, conductor B | **FIXME:** connector, pin and wire color |
| Octopus temperature input and return | T1 connector: PF5 and GND, per BTT v1.1 pinout; **FIXME:** record which harness conductor reaches each contact |

### Shared motion and electronics

| Function | Klipper section / pin(s) in `Components.cfg` | Build check |
| --- | --- | --- |
| Y motor / driver | `stepper_y`: PG0, !PG1, !PF15; UART PD11 | Physical endstop !PG9 |
| Z motor / driver | `stepper_z`: PF11, !PG3, !PG5; UART PC6 | Physical endstop !PG10; confirm bed moves away from nozzle when commanded to positive Z |
| Bed heater / thermistor | PA1; PF3 | Confirm bed load, PSU and board/output current ratings before connection; sensor configured `ATC Semitec 104GT-2` |
| Electronics cooling | PD14 | `temperature_fan mcu_fan`; verify voltage, airflow and clearance |
| RGBW chamber lights | PB10 | `neopixel chamber_led`, 16 pixels, `GRBW`; use a suitable 5 V supply and common ground, not an arbitrary fan output |
| Chamber sensor / heater control | PF6; PB0 | `Generic 3950` chamber thermistor; PB0 is a **control output**, not permission to connect a mains heater to the board |

**Board revision matters:** BTT documents a v1.0-to-v1.1 change of HE0 from PA2 to **PA0**, HE2 from PB10 to **PB0**, MOTOR3 enable from PA0 to **PA2**, and RGB from PB0 to **PB10**. A v1.0 wiring diagram or example config will be misleading for this build. Set each selectable fan-voltage jumper for the measured fan voltage; check polarity and available regulator capacity. Confirm the USB 5 V power-selection jumper arrangement before linking a separately powered host and board to avoid backfeeding. Never move jumpers with power applied.

## Peripherals

* **Lighting/camera (required):** Replace the original lights with two 8-LED RGBW NeoPixel sticks and mount them together with the USB camera on `FFCP2 Neopixel + Camera Mounting Adapter-Body.step` (six M2 screws for devices, two M3 for the bracket). Solder the header pins, insulate exposed joints, and connect the 16 pixels as one data chain (controller data -> first stick DIN; first DOUT -> second DIN). Check the arrow/data direction, 5 V power budget, shared ground, and the actual `GRBW` color order before enabling full brightness. Route the USB camera to the host, not to a random MCU pin.
* **Filament monitoring:** The BOM uses two BTT Smart Filament Sensor V2.0 units and PC6 PTFE guides/fittings. Mount one in each filament path with gentle bends and accessible service loops. Each sensor provides a presence switch and a motion signal: T0 uses PG12/PG13; T1 uses PG14/PG15. Follow the sensor's own connector pinout and supply-voltage specification, and test presence and motion separately. The configured `detection_length: 8.65` assumes the sensors are on the spool side of the path; tune it to your feed path after installation.
* **Bed surface:** If fitting the BOM's magnetic base and PEI or CryoGrip plate, clean and align the existing bed, check clearance to clips and parked heads, and recheck Z endstop/offset after the added thickness. The listed surfaces are alternatives, not two plates to stack by default.
* **Chamber monitoring/heating (required):** A Voron-style thermistor mount attaches to the 2020 extrusion (the CAD directory has a link to the external model). Secure it away from the heater's direct airflow so it represents chamber temperature. The config has `heater_generic chamber_heater` on PB0, with sensor PF6 and an SSR-oriented PWM cycle time. This build uses a **120 V AC PTC heater** switched by the [SSR listed in the BOM](https://www.amazon.com/dp/B08GPJ1V2J); see the wiring notes below. A software temperature limit (`max_temp: 65`) is not a substitute for independent protection.

### 120 V AC chamber-heater wiring

The completed build's heater wiring was fabricated with **standard crimp-on ring terminals** rated for the circuit's amperage, as listed in the [BOM](1-BOM.md). Select terminals suitable for the actual conductor size/type, terminal studs, voltage, current and operating temperature; use a compatible crimping tool and check each termination mechanically. Keep mains connections enclosed and strain-relieved, separated from low-voltage wiring, and maintain the required protective-earth bonds. Do not use the DuPont or JST signal connectors in the workshop supplies list for mains wiring.

PB0 (HE2 on the Octopus Pro v1.1) controls the SSR's input circuit; the **120 V AC heater load must not connect to the Octopus board**. The MCU pin name is not a physical wiring instruction: verify the HE2 output circuit, the SSR's control-input voltage/current and polarity, and the exact terminal assignments before connecting them. Confirm that the actual SSR is suitable for switching the AC heater load, including load current, derating, mounting and any required heatsink. The product link identifies the relay used, not a verified terminal diagram or rating for every replacement sold under that listing.

**The exact mains harness, wire gauge, SSR control-terminal mapping, fusing, independent thermal cutoff and complete power/control design still need to be documented. Do not energize the chamber heater from this draft alone.** Have a qualified person verify the installation against the heater and SSR documentation, including independent overtemperature protection and safe disconnection. PTC behavior and Klipper's temperature limit do not replace those protections.

## First power-on and staged checks

1. With all heaters, including the chamber heater, **physically disconnected**, inspect PSU voltage/polarity, protective earth, fusing, driver orientation and jumpers, cable insulation, strain relief, and board clearance. Power the controller and host; stop immediately for unexpected heat, smoke, or fan behavior. Flash Klipper for the **STM32H723** per the [BTT H723 firmware instructions](https://neo.bttwiki.com/en/docs/board-docs/octopus-series/octopus-firmware-h723) and [Klipper installation guide](https://www.klipper3d.org/Installation.html). Use the *actual* `/dev/serial/by-id/` path in `[mcu]`, not the example machine-specific serial string committed in `Components.cfg`.
2. At ambient temperature, check readings for both hotends, bed and chamber sensor. A missing, swapped, implausible or unstable reading is a stop condition. Test `M112` emergency stop before enabling heaters. For motion, follow [Klipper's configuration checks](https://www.klipper3d.org/Config_checks.html): query and manually actuate each endstop with `QUERY_ENDSTOPS`, test motor identity/direction using `STEPPER_BUZZ` with clear travel, and check each carriage independently before any `G28`. In particular, never home two unverified X heads into each other.
3. Once the sensors and heater connections have been verified, test **one heater at a time** with a modest target while watching the corresponding temperature rise, then turn it off and check cooling. Verify T0 heater heats T0 sensor, T1 heats T1, and bed heats bed. If the wrong channel rises, or temperature rises with the heater commanded off, cut power and fix the wiring. Check each hotend fan comes on as expected and each part fan responds independently. Leave the chamber heater disabled until its separate safety design and installation have been verified; it is a required component, but must not be energized before these checks are complete.
4. Confirm safe X/Y/Z directions, both physical X endstops, and head-to-head/bed clearance at the limits *before* trusting the configured travel, `safe_distance: 40`, and copy/mirror modes. Start with slow single-carriage movements and only then home. The repo's bed/origin and mode-dependent printable areas are described in [Slicer Setup](3-Slicer.md).
5. After basic motion and temperature checks, follow [Calibration](4-Calibration.md) for bed screws, PID, both nozzle heights (physical shims for copy/mirror), Z endstop offset, and IDEX XY offsets. Do not copy the saved PID or Z-endstop numbers from somebody else's machine into yours. Test T0, then T1, then IDEX tool changes, and only after clearance checks try COPY and MIRROR.

## Still to document from the actual build

* Photos of stock and converted electronics, labeled connectors, and the exact FFCP2 harness-to-Octopus **v1.1** socket/ground/polarity map.
* Measured PSU, bed, heater cartridge and fan ratings; power budget; the exact fan/driver/USB jumper positions used; and whether any stock connectors must be repinned.
* Mounting and cable-routing photos for the rails, host, lights/camera, filament sensors and chamber sensor; the exact SSR installation, connector/terminal variants, wire gauge, mains and control-terminal map, and complete circuit and independent safety details for the required chamber-heater installation.

## Acknowledgements and references

* **Evil Azrael:** [Flashforge Creator Pro 2 writeup](https://wiki.evilazrael.de/en/flashforge-creator-pro-2) and [FFCP2 Klipper configuration](https://github.com/eazrael/klipper/blob/ffcp2/config/printer-flashforge-creator-pro2.cfg). Foundational reference for this project's conversion and documentation, including stock hardware identification, temperature sensing, travel geometry and IDEX behavior. His stock-board configuration uses different tool numbering and MCU pins from this Octopus build; do not copy those assignments without translating them.
* **Frank Paynter, Paynter's Palace:** [Klipperizing my Flashforge Creator Pro 2 3D Printer](https://www.fpaynter.com/2024/10/klipperizing-my-flashforge-creator-pro-2-3d-printer/). FFCP2 conversion/build-log reference, with stock-mainboard photos and a schematic supplied to the author by Flashforge. It documents the stock-board approach, not this Octopus harness pinout.
* **Partsbuilt 3D:** [Left breakout board](https://partsbuilt.com/extruder-pcb-board-left-side-creator-pro-2-flashforge/) and [right breakout board](https://partsbuilt.com/extruder-pcb-board-right-side-creator-pro-2-flashforge/) product photos, the source of the eight-pin silkscreen transcription above. Their [left harness](https://partsbuilt.com/cable-for-left-extruder-pcb-creator-pro-2-creator-max-2-flashforge/) and [right harness](https://partsbuilt.com/cable-for-right-extruder-pcb-creator-pro-2-creator-max-2-flashforge/) listings provide additional identification photos, not a verified end-to-end wiring map.
* **3D Prima:** [FFCP2 right extruder board](https://www.3dprima.com/parts/spare-parts/flashforge/flashforge-creator-pro-2-right-extruder-board_25686_6784) and its high-resolution product photos, used to identify the small fan and TEMP sockets.
* **Spool3D:** [Flashforge Creator Pro 2 heater cartridge](https://spool3d.ca/flashforge-creator-pro-2-heater-cartridge/), source of the published 24 V / 40 W replacement-cartridge rating.
* **3D Print Beginner:** [Flashforge Creator Pro 2 Review: Enclosed IDEX 3D Printer](https://3dprintbeginner.com/flashforge-creator-pro-2-review/), including teardown photos and identification of the review machine's Delta power supply.
* **BIGTREETECH / BTT:** [Octopus Pro v1.1 pinout](https://github.com/bigtreetech/BIGTREETECH-OCTOPUS-Pro/blob/master/Hardware/BIGTREETECH%20Octopus%20Pro%20V1.1-Pin.jpg), [English product manual](https://github.com/bigtreetech/BIGTREETECH-OCTOPUS-Pro/blob/master/BTT_Octopus_pro_EN.pdf), [v1.1 revision notes](https://github.com/bigtreetech/BIGTREETECH-OCTOPUS-Pro#octopus-pro-v11), and [H723 firmware instructions](https://neo.bttwiki.com/en/docs/board-docs/octopus-series/octopus-firmware-h723). Primary board references; the PDF describes v1.0 and must be read alongside the v1.1-specific material.
* **Klipper contributors:** [Installation](https://www.klipper3d.org/Installation.html) and [Configuration checks](https://www.klipper3d.org/Config_checks.html), the basis for firmware bring-up and staged temperature, endstop, motor and heater checks.

Product photos and other builders' observations are references rather than confirmation of this machine's wiring. The remaining FIXMEs deliberately separate those sources from details still to be verified on the actual conversion.

# Calibration Steps

## Standard Klipper Calibration Steps

1. `PID_CALIBRATE HEATER=extruder TARGET=220` — Use 240 if primarily printing ABS.
2. `PID_CALIBRATE HEATER=extruder1 TARGET=220`
3. `PID_CALIBRATE HEATER=heater_bed TARGET=55` — Use 100 if primarily printing ABS.
4. `PID_CALIBRATE HEATER=chamber_heater TARGET=55` — Use your intended chamber
   operating temperature, within component ratings and the configured 65°C limit.
   Only run this after the chamber wiring, sensor and independent overtemperature
   protection have been verified; see the
   [chamber-heater safety notes](2-Physical_Conversion.md#120-v-ac-chamber-heater-wiring).
5. Bed leveling using bed-screws. The FFCP2 bed is very flat, but extreme
   precision here pays off when performing the IDEX Z-Offset Calibration below.
6. Standard Klipper Z-offset calibration (perform after bed leveling and IDEX
   Z-Offset Calibration).

## IDEX Z-Offset Calibration

**Prerequisite:** Bed leveling to at least a 0.05mm precision.

### Z-Offset Calibration Notes

- The typical layer-height is around 0.2mm, and we need to be level to within a
  fraction of that height to get a consistent first layer in MIRROR or COPY
  modes.
- A metric feeler gauge set is the standard tool. A Post-it note is a practical
  substitute for a 0.15mm–0.20mm feeler gauge when comparing nozzle clearances;
  paper thickness varies, so use it as a relative reference rather than a
  calibrated measurement.
- Revisit this calibration after changing nozzles, or any mechanical changes to
  the hotend.

### Z-Offset Calibration Procedure

1. Preheat each nozzle and clean the tip with a paper towel to remove any
   excess filament around the orifice and the cone, since excess filament can
   throw off the feeler gauge readings. The nozzles can be cool after this point.
2. Home the axes: `G28`.
3. Position the heads: `_POSITION_CENTER`.
4. Adjust the bed position using the motion controls and a 0.15mm (or 0.20mm)
   feeler gauge, or a Post-it note, passed below both nozzles to position the
   lower nozzle at an appropriate Z-height above the bed.
5. Use the feeler gauges to measure each nozzle's clearance from the bed. If
   using a Post-it note, compare the drag beneath both nozzles instead.
6. Find the difference between the two feeler gauge measurements. On the first
   printer I converted, the right nozzle was 0.18mm lower than the left nozzle.
   If using a Post-it note, select shims iteratively until both nozzles produce
   similar drag rather than calculating a precise difference from the paper
   thickness.
7. Position the heads for service: `_POSITION_SERVICE`.
8. Remove the lower toolhead from its X-carriage using a hex driver.
9. Insert M3x6mm shim washers in the height closest to the Z-difference that you
   calculated in Step 6. In my case, I used M3x6x0.2mm shim washers to raise the
   lower toolhead by 0.2mm, which is about right for that printer. Note that you
   can mix and match different thicknesses of shims front & back or side-to-side
   if you need a 0.05mm Z-height resolution for your particular printer.
   **This step is the main point of this procedure.**
10. Re-attach the toolhead to its X-carriage using a hex driver.
11. Repeat the procedure from Step 3, until you're satisfied that the Z-height
    is as close together as physically possible.
12. Perform the standard Klipper Z-offset calibration to set your actual
    Z-offset.

## IDEX XY-Offset Calibration

1. Print an [XY-Offset Calibration test][xy-offset-test] in IDEX/TOOLCHANGER mode.
2. Adjust the `t1_nozzle_adjust_*` variables in your printer's `printer.cfg`
   based on what you learn.
3. Repeat starting from Step 1 until the results meet the precision required
   from your multi-material prints.

## Gross Skew Calibration

**Prerequisites:** N/A.

### Gross Skew Calibration Notes

- This is usually only necessary if the printer has been serviced, stored or
  transported on its back. With the steppers off, the gantry can strike the back
  of the printer and cause a belt to skip teeth on one side, leading to severe
  skew in printed objects.
- Don't do this too often. It will eventually damage the belts and the toothed
  pulley.

### Gross Skew Calibration Procedure

1. Turn off the steppers.
2. With one hand on each side of the gantry, pull the gantry forward until it
   hits the forward end-stop.
3. Keep pulling. The belt may skip teeth until it's aligned.

[xy-offset-test]: https://www.printables.com/model/129617-offset-xy-dual-extruder-idex-calibration

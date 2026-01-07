Calibration Steps
-----------------

Standard Klipper Calibration Steps
===================================================
1. PID_CALIBRATE HEATER=extruder0 TARGET=220  # Use 240 if primarily printing ABS
2. PID_CALIBRATE HEATER=extruder1 TARGET=220
3. PID_CALIBRATE HEATER=heater_bed TARGET=55 # Use 100 if primarily printing ABS
4. Bed leveling using bed-screws.  The FFCP2 bed is very flat, but extreme precision here pays off when performing the IDEX Z-Offset Calibration (nelow).
5. Standard Klipper Z-offset calibration (perform after bed leveling and IDEX Z-Offset Calibration).


IDEX Z-Offset Calibration
===============================
Prerequisite: Bed Leveling, to at least a 0.05mm precision.

Notes:
+ The typical layer-height is around 0.2mm, and we need to be level to within a fraction of that height to get a consistent first layer in MIRROR or COPY modes.
+ Requires a metric feeler gauge set.
+ Revisit this calibration after changing nozzles, or any mechanical changes to the hotend.

Procedure:
0. Preheat each nozzle and clean the tip with a paper towel to remove any excess filament around the orfice and the cone.  Allow the nozzles to cool.
1. Home the Axes: `G28`
2. Position the heads: `_POSITION_CENTER`
3. Adjust the bed position using the motion controls and a 0.15mm (or 0.20mm) feeler gauge passed below both nozzles to position the lower nozzle at an apporpirate Z-height above the bed.
4. Use the feeler gauge to determine the distance between the higher nozzle and the lower nozzle.
5. Find the differece between the two feeler gauge measurements.  On the first printer I converted, the right nozzle was 0.18mm lower than the left nozzle.
6. Position the heads for service: `_POSITION_SERVICE`
7. Remove the lower toolhead from its X-carriage using a hex driver.
8. Insert M3x6mm shim washers in the height closest to the Z-difference that you measured in Step 5.  In my case, I used M3x6x0.2mm shim washers to raise the lower toolhead by 0.2mm, which is about right for that printer.  Note that you can mix and match different thicknesses of shims front & back or side-to-side if you need a 0.05mm Z-height resolution for your particular printer.  **<-- This step is the main point of this procedure.**
9. Re-attach the toolhead to its X-carriage using a hex driver.
10. Repeat the procedure from Step 3, until you're satisfied that the Z-height is as close together as physically possible.
11. Perform the standard Klipper Z-offset calibration to set your actual Z-offset.


IDEX XY-Offset Calibration:
=================================
1. Print an XY-Offset Calibration test, like this one: https://www.printables.com/model/129617-offset-xy-dual-extruder-idex-calibration
2. Adjust the `t1_nozzle_adjust_*` variables in your printer `printer.cfg` based on what you learn.
3. Repeat starting from Step 1 until the results meet the precision required from your multi-material prints.


Gross Skew Calibration
======================
Prerequisites: N/A

Notes:
+ This is usually only necessary if the printer has been serviced, stored or transported on its back.   With the steppers off, the gantry can crash into the back of the printer, and there is only and endstop on one side at the back of the printer.  The gantry crashing into the printer this way can cause the belt to skip teeth on one side, leading to severe skew in printed objects.
+ Don't do this too often.  It will eventially damage the belts and the toothed pulley.

Procedure:
1. Turn off the steppers
2. With one hand on each side of the gantry, pull the gantry forward until it hits the forward end-stop
3. Keep pulling.  The belt may skip teeth until it's aligned.

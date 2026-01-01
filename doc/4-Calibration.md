Calibration Steps
-----------------

Bed Leveling
============
FIXME: Klipper has screw-leveling coordinates, need to write the procedure.


Multi-Tool Z-Offset Calibration
===============================

Prerequisite: Bed Leveling, to at least a 0.05mm precision.

Notes:
+ The typical layer-height is around 0.2mm, and we need to be level to within a fraction of that height to get a consistent first layer in MIRROR or COPY modes.
+ Requires a 0.15mm feeler gauge

Procedure:
+ Move heads over the bed.  FIXME: Implement Klipper Z_ENDSTOP_CALIBRATE inheritance
+ Adjust Z-offset via the screen so that the lowest head is 0.15mm away from the bed.
+ Loosen the higher nozzle with a wrench until it's at the same level as the lower nozzle and matches with the feeler gauge.  The PTFE tube will keep plastic from ozzing into the gap that you can see in your mind's eye.
+ FIXME: Apply threadlock to the higher nozzle?
+ Run the standard Klipper Z-Offset calibation procedure.


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

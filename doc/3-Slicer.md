Slicer Setup for FFCP2+OK IDEX 3D printer
-----------------------------------------

IDEX Printer
============
FIXME: Explain what an IDEX printer is, to help contextualize what I say next.

Slicer Choice
=============
I did most of the configuration and testing of this printer using Orca Slicer.  As a result, the slicer setup is oriented around Orca Slicer's quirks.  It should work OK with SuperSlicer and Prusa Slicer, but I haven't built and tested configurations for those printers.  FlashPrint 5, the printer's native slicer is unlikely to work because I didn't test my  macros for compatibility with its quirks -- the point of this upgrade is to bring this printer into the Klipper/Voron era.

Print Bed Dimensions
====================
Due to the risk of collissions between the tools, the different print-modes all have different usable areas.  A great writeup is provided by Evil Azreal's Wiki: https://wiki.evilazrael.de/en/flashforge-creator-pro-2

Key differences between his implementation and mine are that I chose to use the left print-head as T0, and I put the origin of my printable area (0,0,0) at the front-left corner of the area that both printheads can reach.  In other words, my (0,0,0) is in the lower-left hand corner of the region the right extruder can reach in Evil Azreal's diagram (https://wiki.evilazrael.de/ffcp2/print_regions.png), and the extent of the logical build-plate in TOOLSWAPPER/IDEX mode goes to the front-right corner of the area that the left roolhead can reach.

What makes this complicated is that other print-modes (especialy COPY and MIRROR) have different constraints.  The print-heads in COPY mode never get close to each other, and the print-heads in MIRROR mode always get close to each other.  Each of these allows a different portion of the print-bed to be available.

Five Print Modes
================
I decided to set up Klipper to support each of the following modes, each with its own unique START_PRINT macro.  In OrcaSclicer, I configured five logical printer since adjusting both the G-code macros and the bed dimensions every time would be error-prone.  Having five different logical printers, each with the START_PRINT_* macro for the mode and the coordinate system adjusted to accommodate the quirks of that mode, make it quick and easy to switch between the modes.

The five modes are as follows:

  + START_PRINT_TOOLSWAPPER:
    Native mode, also known as IDEX mode.  Slicer support required for multicolor and multimaterial prints.  Toolswapping commands (T0 and T1) are honored.
    
    In my OrcaSlicer configuration, this is the base setup from which the other configurations are derived.

    Prinable Volume, Origin-offset: 200x150x 170mm, (0,0,0)
    
    Machine start code:
    ```
    START_PRINT_IDEX INITIAL_EXTRUDER={initial_extruder} BED_TEMP={first_layer_bed_temperature[initial_extruder]} EXTRUDER_TEMP={first_layer_temperature[0]}     EXTRUDER1_TEMP={first_layer_temperature[1]}
    ```
    
  + START_PRINT_MIRROR:
    Emulates a single-tool printer, but T1 mirrors T0's moves.  Toolswapping commands (T0 and T1) are ignored.

    Prinable Volume, Origin-offset: 95x150x 170mm, (15,0,0)

    Machine start code:
    ```
    START_PRINT_MIRROR BED_TEMP={first_layer_bed_temperature[initial_extruder]} EXTRUDER_TEMP={first_layer_temperature[0]}
    ```
    
  + START_PRINT_COPY:
    Emulates a single-tool printer with a bed that's roughly half the size of the full print-bed.  T1 follows T0 and it perform the exact same motions - so it prints two identical copies of your object.  Toolswapping commands (T0 and T1) are ignored.

    Prinable Volume, Origin-offset: 115x150x 170mm, (15,0,0)

    Machine start code:
    ```
    START_PRINT_COPY BED_TEMP={first_layer_bed_temperature[initial_extruder]} EXTRUDER_TEMP={first_layer_temperature[0]}
    ```
    
  + START_PRINT_T0:
    Emulates a single-tool printer with the T0 (left) toolhead selected.  Toolswapping commands (T0 and T1) are ignored.

    Prinable Volume, Origin-offset: 215x150x 170mm, (15,0,0)
    
    Machine start code:
    ```
    START_PRINT_T0 BED_TEMP={first_layer_bed_temperature[initial_extruder]} EXTRUDER_TEMP={first_layer_temperature[0]}
    ```
    
  + START_PRINT_T1:
    Emulates a single-tool printer with the T1 (right) toolhead selected.  Toolswapping commands (T0 and T1) are ignored.

    Prinable Volume, Origin-offset: 215x150x 170mm, (0,0,0)

    Machine start code:
    ```
    START_PRINT_T1 BED_TEMP={first_layer_bed_temperature[initial_extruder]} EXTRUDER_TEMP={first_layer_temperature[0]}
    ```

Detailed Setup
==============
Files with the detailed setup of this printer for OrcaSlicer are available here:
[https://github.com/lukescharf/orca-profiles](https://github.com/lukescharf/orca-profiles)






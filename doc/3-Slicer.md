Slicer Setup for FFCP2+OK
-------------------------

IDEX Printer
============
FIXME: Explain what an IDEX printer is, to help contextualize what I say next.

Slicer Choice
=============
I did most of the configuration and testing of this printer using Orca Slicer.  As a result, the slicer setup is oriented around Orca Slicer's quirks.  It should work OK with SuperSlicer and Prusa Slicer, but I haven't built and tested configurations for those printers.  FlashPrint 5, the printer's native slicer is unlikely to work because I didn't test my  macros for compatibility with its quirks -- the point of this upgrade is to bring this printer into the Klipper/Voron era.

Five Printing Modes
===================
I devided to set up the printer to support each of the following modes, each with its own unique START_PRINT macro:

  + START_PRINT_T0:
    Emulates a single-tool printer with the T0 toolhead selected.  Toolswapping commands (T0 and T1) are ignored.
  + START_PRINT_T1:
    Emulates a single-tool printer with the T1 toolhead selected.  Toolswapping commands (T0 and T1) are ignored.
  + START_PRINT_COPY:
    Emulates a single-tool printer with a bed that's roughly half the size of the full print-bed.  T1 follows T0 and it perform the exact same motions - so it prints two identical copies of your object.  Toolswapping commands (T0 and T1) are ignored.
  + START_PRINT_MIRROR:
    Emulates a single-tool printer, but T1 mirrors T0's moves.  Toolswapping commands (T0 and T1) are ignored.
  + START_PRINT_TOOLSWAPPER:
    Native mode.  Slicer support required for multicolor and multimaterial prints.  Toolswapping commands (T0 and T1) are honored.

In order to support these modes chosen to configure 5 different logical printers in my slicer, each with the attributes suitable to each mode.





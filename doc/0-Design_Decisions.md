# Design Decisions: Flashforge Creator Pro 2 → FFCP2+OK

This document explains the goals and tradeoffs behind this conversion, rather
than providing assembly instructions. The [BOM](1-BOM.md),
[physical-conversion draft](2-Physical_Conversion.md),
[slicer setup](3-Slicer.md), and [calibration guide](4-Calibration.md)
cover the implementation. The current source of truth for behavior is
the topic files in [`config/`](../config/), loaded through
[`config/printer.cfg`](../config/printer.cfg).

## 0. Goal: Functional ASA/ABS Parts with Dissolvable Supports

The goal was to build a good **ASA/ABS printer for functional parts**, including
parts that need dissolvable supports. Dimensional accuracy, reliable extrusion,
and controlled temperatures matter more here than maximum print speed or
preserving the stock FlashPrint workflow.

The FFCP2 is a useful starting point: it already has an enclosed build space and
independent dual extruders. I wanted to retain those mechanics while moving the
electronics and workflow into the Klipper ecosystem. Independent heads let the
model and support materials use separate nozzles and temperatures, and let the
unused head park away from the part. HIPS is the intended dissolvable-support
option for this ASA/ABS-oriented workflow; any other material pairing still
needs its own adhesion, temperature, and support-removal testing.

Most of this printer's calibration and tuning has been done with **Polymaker
PolyLite ASA**. Those settings are not universal ABS or support-material
profiles, but they do provide a good starting point for other filament-types.

## 1. Controller and Drivers: Octopus Pro v1.1 + TMC2209

I selected the **BIGTREETECH Octopus Pro v1.1 with an STM32H723** as the
mainboard for this printer, with **TMC2209 stepper-driver modules**. The
TMC2209s were selected due to a combination of reasonable cost and precise/quit
operation; the stock motors are not pushing their limits in this printer and
don't need to be upgraded.

This IDEX printer needs six independently driven motors: left X, right X, Y, Z,
and two extruders. The Octopus Pro's eight driver sockets provide that capacity
with room left over. Its heater, temperature-sensor, fan, and peripheral
connections also suit two hotends, a heated bed, chamber control, filament
monitoring, and lighting without trying to preserve the stock controller's
interfaces.

TMC2209s provide UART-configurable motor current and quiet motion in a
well-supported Klipper configuration. The current configuration uses StealthChop
for the motion axes and SpreadCycle for the extruders. Physical endstops are
retained; using these drivers does not imply sensorless homing. Driver current
and cooling still need to suit the actual motors and enclosure conditions.

The board revision matters: BTT's documents show that the heater, the
motor-enable, and the RGB pin assignments were all changed between v1.0 and
v1.1. Use the [manufacturer's v1.1 references][octopus-pro], not an untranslated
v1.0 pin map.  Since v1.0 and v1.1 board documentation are shared in the same
repo, it's easy to pick the wrong one.

Several other Klipper-compatible boards could work in this application. If
you choose one of those, you'll need to revise the pin assignments and
work out the physical mounting yourself.

## 2. Five Print Modes, Selected at Job Start

I chose to implement the different print modes as five separate logical printers
in OrcaSlicer. Each profile selects a mode through its **Machine start G-code**
and supplies the matching printable volume and origin, which vary substantially
by print mode. The G-code start macros defined in
[`config/Print_Start.cfg`](../config/Print_Start.cfg) are:

| Mode | Start macro | When to choose it |
| --- | --- | --- |
| IDEX / toolswapping | `START_PRINT_IDEX` | Model and support material, or two colors; the slicer selects each head as needed. |
| T0 only | `START_PRINT_T0` | Ordinary single-material printing with the left head. |
| T1 only | `START_PRINT_T1` | Ordinary single-material printing with the right head. |
| COPY | `START_PRINT_COPY` | Two identical parts printed simultaneously from one toolpath. |
| MIRROR | `START_PRINT_MIRROR` | A left/right mirrored pair printed simultaneously from one toolpath. |

`START_PRINT` is a convenience alias for `START_PRINT_T0`, not a sixth mode.
`_START_PRINT_SINGLETOOL` is the shared implementation behind the two
single-head entry points, not the intended slicer-facing interface.

When using `START_PRINT_IDEX`, the `TOOL=` parameter selects the initial head,
and `T0_TEMP` and `T1_TEMP` specify independent nozzle targets, and subsequent
slicer-generated `T0` / `T1` commands perform tool changes.  The `START_PRINT_IDEX` is designed around the parameters that OrcaSlicer provides in the job-start maco.

The other four modes take `EXTRUDER_TEMP` and lock tool selection after startup.
This prevents incidental tool commands from the slicer from undoing the selected
single-head or synchronized mode. `TCOPY` and `TMIRROR` establish the
synchronized relationships; in these modes the T0 printhead drives the toolpath and T1 follows its motion and extrusion.

See [Slicer Setup](3-Slicer.md) for the actual start-code expressions, origins,
printable volumes, and MIRROR clearance calculation.

The startup macros wait for their specified nozzle and bed temperatures, but
**do not automatically set or wait for chamber temperature**. Chamber preheating
must be requested separately through the chamber commands described below.  I
chose this approach based on the quirks of the G-Code that Orca Slicer actually
generates, and some details may need to be adjusted in the Klipper-side G-Code macros to accommodate other slicers.

## 3. Left Head as the Primary Tool, T0

I arbitrarily chose the tool numbering, and I did not follow FlashForge's
convention.  When I had the machine torn apart with FlashForge's
mainboard/firmware/screen removed, I realzied I could this machine up in the way
I think is intuitive.  I chose **left = T0** and **right = T1** because
left-to-right numbering felt the most intuitive to me.

This convention is carried consistently throughout every level of the
conversion:

- T0 is Klipper's `extruder` and primary X carriage, `stepper_x`.
- T1 is `extruder1` and `dual_carriage`.
- The generic `START_PRINT` defaults to the left head.
- COPY and MIRROR use T0 as the motion/extrusion leader.
- T1's calibrated nozzle offsets in Klipper are relative to the T0 reference.

**This differs from the stock workflow's default primary head, and also from the
other conversion guides that I referenced.** The [Flashforge Creator Pro 2 User
Guide][ffcp2-manual], section 3.3, printed page 45 (PDF page 47), states that
the left extruder is used for support material by default, while the right
extruder prints the model. It also says that a monochrome model uses the right
extruder automatically unless the user changes the selection. That passage
establishes the stock default roles, not literal `T0` / `T1` G-code numbering;
the numbering above is this conversion's explicit mapping. Neither head is
permanently restricted to model or support material.

In software terms, this is a one-off Klipper-based IDEX printer, so applying
profiles from other FFCP2 variations needs to be done with care.

## 4. Heater Blocks and Octopus-Compatible Thermistors

The stock hotend sensors are **thermocouples**. This build instead uses
thermistors on the Octopus's ordinary thermistor inputs, with both
hotend-sensors configured as `Generic 3950` (generic Ender3-style thermisters).
A thermocouple is not a drop-in electrical replacement for a thermistor on those
inputs, and thermisters are not expensive.

The two heater blocks were replaced with **Micro Swiss plated-copper,
high-temperature MK10 blocks with M7 threads** so the selected replacement
thermistors could be fitted and retained appropriately while retaining physical
compatibilty with the FFCPK's extruder and nozzles. For this parts selection,
the block change was a compatibility necessity, not simply a cosmetic hotend
upgrade. It also avoids improvising an insecure sensor mounting in a hot block.

Swapping in standard thermistors and matching blocks was the most straightforward
and cost-effective way to make this work using off-the-shelf parts. The heated
bed retains the sensor that Flashforge installed, since it is compatible with
the Octopus Pro v1.1.

Sensor type, retention, electrical isolation, and heater/sensor pairing must be
verified before heating. Replacing blocks and sensors also requires fresh PID
tuning and nozzle-height/Z calibration. Use thermal paste only where specified
by the component manufacturer. The configured 275°C hotend limit is my estimate,
not a verified safe hardware rating. I've experienced frequent clogging at
temperatures above 250°C with the Flashforge extruder.

## 5. Active Chamber Heating to Reduce ASA/ABS Warping

The stock FFCP2 enclosure reduces drafts and keeps the print warmer than it
would otherwise be, but I still experienced frequent warping with the enclosure
in place. Active chamber heating, the blocking `M140` and `M141` commands, and
the `CONTROLLED_COOLDOWN` post-print macro are all intended to reduce temperature
gradients and uneven contraction in ASA/ABS parts. With all of these methods
applied, I get minimal warping and keep elephant's foot under control when
printing with ASA and ABS.

The chamber heater design evolved from an **80 W, 24 V DC PTC prototype** (which
was constrained by the printer's PSU power supply), through a 200 W AC-heater &
SSR design, to the current **400 W, 120 V AC PTC heater switched by an SSR**
with a printed duct to direct airflow and provide a mounting pracked. Using a
mains-powered AC heater avoids overloading the printer's low-voltage
power-budget, but introduces mains-voltage installation and safety requirements
and using a 400W PTC heater alongside a 300W DC power supply means we have to
start budgeting gainst the 1800W limitation of a standard 15-amp 120-volt North
American power outlets.

A separate chamber thermistor enables closed-loop temperature control in
Klipper. It is mounted near the top rear of the chamber at roughly same height
as the print nozzle. Mounting the thermister on the back wall is a compromise to
maintain mechanical clearances and doesn't seem to throw off the temperature
reading too much while reading the temperature that the model is subjected to
about as well as we can given the mechanical constraints. The configured chamber
maximum is 65°C, but the highest chamber temperature at which I've been able to
print reliably without constant clogs is about 60°C. The thermal limits of the
wiring, fans, electronics, mounts, and other components exposed to the chamber
also matter. More chamber heat is not automatically better.

## 6. Blocking Temperature Changes for Print Quality

I implemented blocking bed-temperature and chamber-temperature changes to reduce
warping, elephant's foot, and other distortions. The build plates I've tested
require a bed temperature of aboout 90°C for ASA to adhere consistently, but
that temperature is basically the glass-transition temperature for ASA and using
the build-plate at this temperature can  leave the base of the print soft enough
to deform under the normal stresses of being printed. I noticed that the bottom
5–7 mm of a print would be distorted, while the areas above it on tall prints
were clearly unaffected, and realized that the built-plate was keeping the print
too hot. I tried setting the bed temperature to 90°C for the first layer and
then to  75°C for subsequent layers and that  reduced (but did not eliminate)
the effect. When I modified the bed-temperature macro (`M140`) block until the
bed cooled to the operating temperature I requested, I found that my print had
both good bed adhesion and low deformation!

In addition to blocking while the temperatures change, I discovered that M140
needed to park the toolhead during the operation to keep a stationary hot-end
from melting the model and depositing ooze on the print during the minutes it
takes to equalize the temperature. Since this is an IDEX printer, parking hot
printheads and managing ooze are design requirements when running in
tool-swapping mode, so I was able to leverage those capabilities here. In other words, the usual usual nonblocking semantics for `M140` and `M141` just doesn't work very well.

Here's a breakdown of the cooling/heating behavior, since there are a number of exceptions:

| Command | Current waiting policy |
| --- | --- |
| `M140 S…` | Sets the bed target and, outside ±3°C, waits for heating. For cooling, waits only when the target is more than 5°C above the greater of measured chamber temperature and chamber target. `S0` does not wait for cooling. |
| `M141 S…` | Sets the chamber target and, outside ±3°C, waits for heating. For cooling, waits only for targets at least 30°C and no more than 10°C above the measured bed temperature. `S0` turns the heater off without waiting. |
| `M191 S…` | Explicit blocking chamber-temperature command; for a positive target, waits within ±7%. It can temporarily use an otherwise-off bed to assist chamber heating. `S0` turns the chamber heater off. |

When `M140` or `M141` requires a temperature wait, it saves G-code and
dual-carriage state, parks the active head, waits, then restores the carriage
relationship and print position. This matters for COPY/MIRROR as well as
single-head printing. The cooling conditions are evaluated when the macro is
invoked and avoid selected waits for targets that may be impractical under the
current thermal conditions; they do not guarantee every cooling request will
finish quickly.

The `START_PRINT_*` macros use direct heater commands to begin nozzle and bed
heating in parallel, followed by explicit temperature waits. Separately, `M191`
can use an otherwise-off heated bed to assist the chamber heater during chamber
preheating. Calling `SET_HEATER_TEMPERATURE` directly elsewhere also bypasses
the `M140` / `M141` parking-and-wait policy. Slicer start code and temperature
transitions must be reviewed with these semantics in mind.

`CONTROLLED_COOLDOWN S=<target> R=<degrees_per_minute>` complements this policy
with a gradual bed-target ramp, maintaining the chamber target until the ramp
drops below it, then lowering both together. It relies on natural cooling and
temperature waits rather than active refrigeration. It must be requested
explicitly: `END_PRINT` currently turns the heaters off and does **not** call
`CONTROLLED_COOLDOWN` automatically.

The tradeoff for this quality-oriented thermal management is that the printer
pauses frequently to manage bed and chamber temperatures, in exchange for more
predictable thermal conditions and higher-quality prints.

This printer is optimized for making functional parts and has not been optimized
for speed. This printer spends a fair amount of print-job time on thermal
management, but I get high-quality prints out of it -- so I'll take it.

## 7. Future Direction: Orbiter V3.5 Printheads

I am seeing both mechanical and thermal limitations in the Flashforge extruder
gripper/feed arrangement and hotend. Replacing the controller and controlling
chamber temperature improve the platform, but did not remove those toolhead
limits.  I expect higher speeds, higher temperatures, and improved quality with
Orbiter V3.5 printheads, but I'll have to try it to see.

My next FFCP2 conversion will likely **fork this project and replace the
printheads with Orbiter V3.5 printheads and Revo nozzles**. I hope this will reduce
slack in the filament path, improve filament-motion detection, and enable higher
printing speeds and chamber temperatures.

## References

- [Flashforge Creator Pro 2 User Guide][ffcp2-manual], section 3.3, printed
  page 45: stock model/support head defaults.
- [BTT Octopus Pro repository][octopus-pro]: v1.1 revision notes, pinouts, and
  driver-supply warnings.
- [`Components.cfg`](../config/Components.cfg),
  [macro topic files](../config/), and
  [`printer.cfg`](../config/printer.cfg): implemented hardware configuration,
  mode/temperature behavior, and machine-specific tunables.
- [Major Change Log](../README.md#major-change-log): dated milestones,
  including the chamber-heater revisions.

[ffcp2-manual]: https://en.fss.flashforge.com/10000/software/899daf8848060aa85e584cda439514c0.pdf
[octopus-pro]: https://github.com/bigtreetech/BIGTREETECH-OCTOPUS-Pro

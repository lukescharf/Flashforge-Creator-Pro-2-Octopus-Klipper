# 3. Printed Parts

This guide supplements the [Bill of Materials](02-BOM.md) with details of the
printed parts in [`cad/`](../cad/). The lighting and camera entries share a
single bracket. See [Physical Conversion](04-Physical-Conversion.md) for
assembly and wiring.

## Print Inventory and File Formats

| Part | Quantity | Source |
| --- | --- | --- |
| [Octopus mounting rails](#octopus-mounting-rails) | 2 | FreeCAD |
| [Tablet mount](#tablet-mount) | 1 | Parametric FreeCAD |
| [NeoPixel and camera adapter](#neopixel-and-camera-adapter) | 1 | FreeCAD |
| [Screen-panel plug](#screen-panel-plug) | 1 for the plain-panel arrangement | FreeCAD |
| [Screen-panel enclosure-fan insert](#screen-panel-enclosure-fan-insert) | Unfinished concept; not required for the current build | FreeCAD |
| [PTC heater duct](#ptc-heater-duct) | 1 for the documented 400 W heater arrangement | FreeCAD |
| [Chamber thermistor mount](#chamber-thermistor-mount) | 1 | External model link |
| [M3 cable P-clips](#m3-cable-p-clips) | As needed | External model link |
| [Service back-feet](#service-back-feet) | 4 | 3MF project containing one foot |

The unfinished fan insert is intended to use the same stock display opening as
the plain plug. Development has stalled while the focus remains on ASA printing;
its fan and hardware selection are not yet documented in the BOM. The heater
duct is present in CAD and described in [Design
Decisions](01-Design-Decisions.md#4-chamber-heating-hardware).

`.FCStd` files are editable FreeCAD documents. Recompute after changes and
export the finished Body as an STL or another format supported by your slicer.
`.link` files contain external download URLs, not printable geometry.

## General Printing Notes

The BOM recommends **Polymaker PolyLite ASA** for most rigid parts. Temporary
PLA parts may help bootstrap the conversion in cool locations, but are not a
general substitute for parts exposed to chamber heat or heater discharge. Use
**TPU** for the service feet.

Except for the service-foot project, the repository does not record validated
layer heights, wall counts, infill, support settings, or build-plate
orientations. The notes below are printing recommendations rather than tested
slicer presets. Inspect overhangs and layer adhesion, preserve hole spacing, and
test-fit before assembly. Avoid scaling a whole part to correct one tight hole:
that also changes its mounting geometry. For ASA, control warping and check long
parts for flatness.

## Octopus Mounting Rails

### What the part is and does

Two adapter rails connect the Octopus Pro v1.1 mounting pattern to the existing
FFCP2 case holes, with standoffs that hold the board above the case.

File: `cad/FFCP2 Octopus Pro v1.1 Mounting Rails.FCStd`.

### Installation location

Inside the bottom-access electronics bay, between the case and replacement
controller. The stock printer has a removable louvered bottom panel; I've left
it unistalled during most of the conversion work, and I plan to reinstall it
when the project is "done". Fit two M3 heat-set inserts per rail (four total).
The BOM specifies four M3 screws for rail-to-case attachment and another four
for board-to-rail attachment. Check screw length and clearance beneath the
board.

> **Photo placeholder:** Electronics bay showing both rails, case attachment
> points, heat-set inserts, and the installed Octopus board.

### Printing considerations

Print two copies. Keep the rails straight and the standoff faces level so the
board is not twisted when tightened. Provide sufficient walls around the insert
pockets; check the actual inserts' dimensions before fitting them. Install the
inserts with controlled heat after printing, with the board removed.

## Tablet Mount

### What the part is and does

A parametric holder supports the touchscreen Klipper host. The BOM's host is a
Dell Chromebook 11.6-inch convertible, but the model is adjustable for other
devices.

File: `cad/FFCP2 Tablet Mount.FCStd`.

### Installation location

On the outside of the machine's front top rail, using the same attachment points
that secure the plastic front enclosure panel to the metal frame. The mounting
screw centers are 150 mm apart. Replace the original enclosure-front screws with
longer **M4 screws** to accommodate the tablet mount's added thickness while
retaining the front panel. Allow access to the host's ports, cooling vents, and
power button.

> **Photo placeholder:** Holder on the outside front top rail, showing the
> longer M4 screws shared with the plastic front panel, device retention, and
> cable routing.

### Printing considerations

Edit FreeCAD's `params` spreadsheet before exporting the finished Body.
Parameters include tablet thickness and width, front/back holder heights, rake
angle, rear clearance, wall thickness, and bracket geometry. Retain the 150 mm
mounting-hole spacing unless adapting to a different attachment point.

Print one holder. Choose an orientation that gives the bracket-to-holder
junction good strength under the host's weight; inspect any required supports at
the holder's overhangs. Check the customized model against your printer's build
volume and test the device fit before loading the mount.

## NeoPixel and Camera Adapter

### What the part is and does

One combined bracket carries two Adafruit eight-pixel RGBW NeoPixel sticks and
the SVPRO USB camera. Its raised mounting features position the circuit boards
and camera on a common beam. The camera listing in the BOM refers to this same
part, not a second bracket.

File: `cad/FFCP2 Neopixel + Camera Mounting Adapter.FCStd`.

### Installation location

This is mounted on the inside of the top-front rail, using existing mounting
studs.  It is aimed toward the build area. The BOM specifies two M3 screws to
attach the adapter to the printer and six M2 screws for the lights and camera.
The exact frame attachment points and viewing direction still need to be
identified in the installation photo. Check clearance to both carriages and
their cables throughout travel.

> **Photo placeholder:** Installed bracket showing both NeoPixel sticks, camera
> orientation, the two printer attachment screws, and cable exits.

### Printing considerations

Print one. Preserve the small mounting holes and standoffs; remove support
material without breaking them. Preview the orientation so supports do not spoil
the circuit-board seating surfaces. Check that the long beam remains straight.
Use material suitable for the heated chamber rather than relying on temporary
PLA here.

## Screen-Panel Plug

### What the part is and does

A plain replacement insert fills the opening left by removing the original
screen. It includes recessed “Octopus Pro V1.1” lettering and retaining
features; the BOM lists no additional hardware.

File: `cad/FFCP2 Screen Panel Plug.FCStd`.

### Installation location

In the vacated stock display opening on the front panel. This is the plain-panel
option; the enclosure-fan insert below uses the same opening.

> **Photo placeholder:** Front view of the fitted plug and a rear/detail view of
> its retention features.

### Printing considerations

Print one if using this arrangement. Preview the lettering at your chosen layer
height and nozzle size. Account for elephant's foot at the panel-fitting edges
and avoid support scars on the visible face or latch surfaces. Test-fit gently;
do not force warped or oversized retaining features into the bezel.

## Screen-Panel Enclosure-Fan Insert

### What the part is and does

This unfinished fan/duct concept is intended to let the printer switch between
**PLA and ASA/ABS printing without physical reconfiguration**, such as removing
the top-hat. The proposed enclosure airflow path uses the old screen opening. My
focus has been on making a great ASA printer, so development of this feature has
stalled. It is not a completed feature of the current build.

The CAD includes mirrored left/right ducts, a cable passthrough, and features
labeled for M3 and M4 heat-set inserts. Fan models, insert quantities, screw
lengths, and operating airflow direction still need to be documented; they are
not established by the filename alone.

File: `cad/FFCP2 Screen Panel Enclosure Fan.FCStd`.

### Installation location

The intended location is the front panel's former stock-screen opening,
replacing the plain plug. Fan and cable clearances, airflow direction, and
control behavior remain to be worked out as development resumes.

> **Photo placeholder (future prototype):** Exterior and interior views of the
> fan panel, with fan sizes, mounting hardware, cable passthrough, and airflow
> arrows identified.

### Printing considerations

Treat any print as a development prototype. Inspect the internal ducts and
overhangs in the slicer; any supports must be removable through the openings.
Keep air passages and the cable passthrough clear. Check insert-pocket
dimensions against the selected hardware and leave enough surrounding walls for
heat-setting. Preserve the panel's retaining features during support removal and
test-fitting.

## PTC Heater Duct

### What the part is and does

The printed duct directs airflow and provides a mounting interface for the 400 W
PTC chamber-heater arrangement described in Design Decisions. The model includes
a heater interface, hollow duct, faceplate, and bent outlet.

File: `cad/PTCYIDU 80mm PTC Fan Heater 400W Heater Duct.FCStd`.

### Installation location

At the chamber heater, with the outlet delivering air into the build chamber.
I've installed this in the bottom-stage-left portion of the build-chamber,
attached with double-sided mounting tape.  The fan wires fit through existing
holes in the metal chassis (ring-terminals were installed after the wire was
routed).

> **Photo placeholder:** Heater and duct installed, showing chassis attachment,
> heater interface, outlet direction, and clearance to the bed and wiring.

### Printing considerations

Export the finished FreeCAD Body; no slicer project is supplied. Inspect the
hollow passage and outlet bend for overhangs and support-removal access.
Maintain continuous walls and check the heater-interface fit without distorting
the duct.

Material selection needs particular attention: discharge and heater-interface
temperatures can exceed the measured chamber temperature. Neither the general
ASA recommendation nor the configured 65°C chamber limit establishes a suitable
material rating for this duct. Do not use the temporary-PLA recommendation here;
the actual material and local temperature suitability remain to be documented.
See the [chamber-heater installation
notes](04-Physical-Conversion.md#120-v-ac-chamber-heater-wiring).

## Chamber Thermistor Mount

### What the part is and does

The external [Voron Chamber Thermistor Mount][thermistor-model] holds the
chamber temperature sensor on 2020 extrusion. The repository stores the model
URL in `cad/voron_champber_thermister.link` (filename spelling retained).

### Installation location

Near the top rear of the chamber, roughly at nozzle height and attached to the
chamber wall with double-sided servo-tape. The BOM specifies attachment to the
printer's 2020 extrusion. Position the sensing element away from direct heater
airflow and retain its leads without pinching them, so it measures
representative chamber air temperature.

> **Photo placeholder:** Rear chamber sensor location, with extrusion
> attachment, sensor retention, and its position relative to the heater outlet
> identified.

### Printing considerations

Print one from the linked project's files and follow its instructions for the
chosen variant and mounting hardware. Check fit against the actual sensor and
extrusion. Use chamber-temperature-appropriate material and remove strings or
support debris around the sensor opening without enlarging its retention
features.

## M3 Cable P-Clips

### What the part is and does

Printed P-clips secure the replacement wiring using M3 screws. The external [M3
Cable Mount P-Clip project][pclip-model] supplies multiple cable sizes; the
repository's `cad/m3-pclip-10mm.link` identifies the 10 mm selection.

### Installation location

At suitable M3 mounting studs in the electronics bay and along the frame rails,
as needed. Choose positions that keep wiring away from moving parts and allow
service loops; the exact clip count and locations depend on the finished
harness.

> **Photo placeholder:** Representative installed clips and an overview of the
> harness route, with attachment points and moving-cable slack shown.

### Printing considerations

Download the linked model and print as many clips as the wiring needs. Choose a
size that retains the bundle without crushing insulation; change the model size
rather than scaling the M3 mounting hole. Follow the source's orientation
guidance and check layer strength where the clip flexes. Select material for
each clip's actual location, particularly inside the heated chamber.

## Service Back-Feet

### What the part is and does

Four TPU feet support the printer on its back during service, providing
compliant contact points instead of resting it on vulnerable rear components.

File: `cad/FFCP2 Service Mode Back-Foot.3mf`.

### Installation location

These are attached to the printer's back panel using double-sided mounting tape
at the corners.  Since the back-panel of the printer is not planar, these feet
provide stability for when you need to work in the electronics bay for an
extended period of time.

> **Photo placeholder:** Rear view showing all four foot positions and a side
> view of the printer resting on them for service.

### Printing considerations

Use **TPU** and print **four copies**; the 3MF contains only one foot. The
recorded project settings are **0.2 mm layers, three walls, 25% infill, and a 7
mm brim**. Review the imported printer and filament settings and re-slice for
your machine. Remove the brim cleanly and inspect layer bonding before loading
the feet.

[thermistor-model]:
    https://www.printables.com/model/549305-voron-chamber-thermistor-mount
[pclip-model]:
    https://www.printables.com/model/1117426-m3-cable-mount-p-clip-10-sizes-2-10mm

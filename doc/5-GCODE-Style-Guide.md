# 5. Klipper G-Code Style Guide

This style guide was created by and for the LLM-based linting tools that I'm
using to refine the machine-specific Klipper G-Code for my FFCP2+OK printer
project.

This guide defines how to write and review the Klipper configuration and macros
in this repository. It follows common practice in published Voron configurations
and community documentation, with explicit project choices where examples differ.
It also adopts the macro-scoped documentation blocks used in
[`config/Temperature.cfg`](../config/Temperature.cfg).
## Basis and Scope

There is no single mandatory, Voron-wide formatting standard established by the
sources below. The [Voron macro documentation][voron-macros], its
[community beginner's guide][voron-beginners], and the
[official VORON2 Octopus configuration][voron-config] provide the baseline:
recognizable macro names, indented command bodies, logical sections, explanatory
comments, and deliberate state management. Published examples vary in indentation,
comment markers, and expression spacing.

This guide distinguishes three kinds of rules:

- **Klipper requirements:** syntax and execution behavior defined by Klipper.
- **Voron practice:** conventions illustrated by Voron documentation and examples.
- **Project choices:** consistent local rules, including four-space indentation
  and the detailed macro documentation format below.

Klipper's [command templates][klipper-templates],
[configuration reference][klipper-config], and [G-code reference][klipper-gcodes]
take precedence over examples when determining what commands actually do. Voron
examples are not drop-in hardware configurations for this Cartesian IDEX printer.

Apply this guide to new or substantially edited code. Existing deviations are
not, by themselves, permission to reformat unrelated files or change behavior.

## Quick Reference

| Item | Convention | Basis |
| --- | --- | --- |
| Public macros and commands | `CONTROLLED_COOLDOWN`, `SET_HEATER_TEMPERATURE` | Voron practice |
| Internal helpers | Leading underscore: `_MOTORS_ACTIVE_BARRIER` | Voron practice; not access control |
| Command parameter names | Uppercase: `HEATER=heater_bed`, `params.TARGET` | Voron practice; Klipper exposes parameter names uppercase |
| Configuration keys | Lowercase with `: `, such as `description: ...` | Voron practice; separator choice is local |
| Jinja local variables | Lowercase snake_case: `cooling_per_second` | Voron practice |
| Macro state variables | Lowercase: `variable_bed_temp_target` | Klipper requirement against uppercase variable names |
| Indentation | Four spaces per level; no tabs | Project choice consistent with many Voron examples |
| Comments in `.cfg` files | `# ` for prose and disabled code | Project choice; Voron examples also use `;` |
| Macro documentation | Section-local purpose, usage, parameters, behavior, and safety | Project extension of Voron's documentation guidance |

## File and Section Organization

- Keep `config/printer.cfg` as the entry point with explicit includes. Keep
  hardware definitions in `Components.cfg` and macros in their existing topic
  files. See the [configuration layout](../README.md#configuration-layout).
- Preserve the repository's capitalized topic filenames. They distinguish
  repository-managed files from locally supplied integrations; this is not a
  universal Voron naming rule.
- Use a Voron-style banner for a file or major functional group, not before every
  short helper:

  ```ini
  #####################################################################
  #   Heating and Cooling
  #####################################################################
  ```

- Group related settings and keep one setting or executable command per line.
  Avoid duplicate options and sections used as accidental overrides. Document
  any intentional integration override and its include-order dependency.
- Leave one blank line between logical steps and two between macro sections.
- Preserve the generated `SAVE_CONFIG` block. Its `#*#` lines contain active
  saved configuration; they are not ordinary commented-out code.

## Indentation, Casing, and Expressions

Configuration section headers and keys start at column zero. Everything in a
multiline `gcode:` value must be indented. Use four spaces for the body and four
more for each nested Jinja block. Jinja nesting is determined by its tags, not
by Python-style indentation, but the indentation must show that nesting clearly.

```ini
[gcode_macro EXAMPLE_REPORT_BED]
description: Report the current bed temperature
gcode:
    {% set bed_temp = printer.heater_bed.temperature | float %}
    {% if bed_temp > 30 %}
        RESPOND TYPE=echo MSG="Bed temperature is {bed_temp | round(1)} C."
    {% else %}
        RESPOND TYPE=echo MSG="Bed is at or below 30 C."
    {% endif %}
```

- Uppercase command names and parameter **keys**, not object names or values.
  Write `SET_HEATER_TEMPERATURE HEATER=chamber_heater TARGET=50`, not
  `HEATER=CHAMBER_HEATER`. Preserve configured names and case-sensitive values.
- Use `key: value` for configuration and `KEY=value` for extended command
  parameters. Traditional commands use their own syntax: `G1 X20 F1200`,
  `M140 S30`, and `G28 X`. Do not mechanically convert those to `X=20` or `S=30`.
- Use Klipper expressions `{value}` and Jinja statements `{% ... %}`. Do not
  substitute ordinary Jinja's `{{ value }}` expression delimiters.
- Put spaces around assignment, arithmetic, comparison, and filter operators:

  ```jinja
  {% set rate = params.R | default(5) | float %}
  {% set cooling_per_second = rate / 60 %}
  {% set remaining_minutes = ((seconds - i) / 60) | int %}
  ```

- Use readable names and explicit parentheses where precedence is not obvious.
  Extract a long calculation into a local variable instead of repeating it.
- Use lowercase Jinja literals such as `true`, `false`, and `none`. Configuration
  macro-variable values and `SET_GCODE_VARIABLE VALUE=...` are **Python
  literals**, so use values such as `False`, `0.0`, or a correctly quoted string
  there instead.
- Aim for roughly 80-character prose and comment lines. Do not split one
  executable G-code command across lines just to meet a line-length target;
  indentation is not a general G-code line-continuation mechanism.
- Remove trailing whitespace and end files with a newline.

Macro names are case-insensitive to Klipper, but consistent uppercase improves
readability. Follow Klipper's documented naming constraints, including keeping
digits at the end of a macro name. Do not rename existing slicer/UI entry points
merely to match another project's preferences: this repository uses
`START_PRINT_*` and `END_PRINT`, even though Voron examples often use
`PRINT_START` and `PRINT_END`.

## Macro-Scoped Documentation

**Project standard:** place the comment block immediately after
`[gcode_macro NAME]`, before `description:`, variables, and `gcode:`, as in
`Temperature.cfg`. This makes the documentation visibly belong to that macro.
Add a concise `description:` for Klipper's `HELP` and frontend discovery; it
does not replace the detailed block.

For a nontrivial macro, document:

1. **Purpose:** what it does, not merely what its name expands to.
2. **Usage:** actual command syntax and a representative invocation.
3. **Parameters:** names, units, defaults, accepted ranges, required values, and
   special meanings such as `S0` or `MOVE=0`.
4. **Behavior:** ordering, blocking waits, heating versus cooling, and whether a
   final temperature is maintained or the heaters are turned off.
5. **Preconditions and safety:** homing, coordinate mode, clearance, extrusion
   temperature, valid print modes, and whether a model may be on the bed.
6. **State and side effects:** tool/carriage selection, offsets, fans, sensors,
   heater targets, movement, and exactly what is restored or intentionally left
   changed.
7. **Limitations and dependencies:** required sections/helpers, evaluation-time
   assumptions, unreachable targets, and any lack of a timeout.

Use only the headings that are relevant; a short alias does not need a large
block. Prefer complete explanations over duplicated boilerplate.

The following complete example illustrates the format. It is **not installed**
by this guide and is not a replacement for `CONTROLLED_COOLDOWN`. Review any
example before deploying it to a printer.

```ini
[gcode_macro EXAMPLE_WAIT_FOR_BED_COOLDOWN]
# Wait for passive bed cooling without moving axes or changing heater targets.
#
# Usage: EXAMPLE_WAIT_FOR_BED_COOLDOWN TARGET=30 TOLERANCE=3
#
# Parameters:
#   TARGET: Final temperature in C; must be positive. Default: 30.
#   TOLERANCE: Allowed temperature above TARGET in C; must be >= 0. Default: 3.
#
# Behavior:
#   Block until the bed measures <= TARGET + TOLERANCE.
#   No homing, carriage selection, extrusion, or heater changes are performed.
#
# Preconditions and safety:
#   The bed heater must already be off. Requires [respond] and [heater_bed].
#   Choose an achievable threshold above ambient; the wait has no timeout.
#   G-code modes, tool selection, and heater targets are left unchanged.
description: Wait for passive bed cooling without motion
gcode:
    {% set target = params.TARGET | default(30) | float(none) %}
    {% set tolerance = params.TOLERANCE | default(3) | float(none) %}

    {% if target is none or target <= 0 %}
        {action_raise_error("TARGET must be a positive temperature in C.")}
    {% endif %}
    {% if tolerance is none or tolerance < 0 %}
        {action_raise_error("TOLERANCE must be a nonnegative temperature in C.")}
    {% endif %}
    {% if printer.heater_bed.target | float > 0 %}
        {action_raise_error("Turn off the bed heater before waiting for cooldown.")}
    {% endif %}

    {% set maximum = target + tolerance %}
    RESPOND TYPE=echo MSG="Waiting for bed temperature <= {maximum | round(1)} C."
    TEMPERATURE_WAIT SENSOR=heater_bed MAXIMUM={maximum}
    RESPOND TYPE=echo MSG="Bed cooldown wait finished."
```

### Writing Accurate Comments

- Explain **why** a command or guard exists. Avoid comments that just repeat its
  spelling. Put multi-line explanations above the affected code.
- Use `# ` consistently in `.cfg` files. Existing semicolon comments are valid;
  prefer `;` for comments in standalone `.gcode` examples or slicer output.
  This is a local consistency choice, not a claim that Voron forbids either form.
- Do not rely on quotes to protect a literal `#` from the configuration reader.
  Keep configuration comments separate from text intended for display.
- Label intentionally disabled code, its purpose, and the consequence of
  enabling it. Remove abandoned experiments rather than preserving unexplained
  commented-out command piles. Preserve useful, clearly labeled diagnostics.
- Describe actual behavior, including known bugs and assumptions. “Restores
  state” must not imply restoration of heaters, active extruders, or carriage
  synchronization when only G-code parsing state is restored.
- Update comments in the same change as behavior. If a wait is disabled, remove
  claims that execution waits for measured temperature. If a variable is used as
  a proxy for hardware state, name that assumption explicitly.

## Parameters, Variables, and Overrides

Convert `params` strings before arithmetic. Apply `default(...)` before the
numeric conversion, and validate before division, movement, heating, or waiting.
Numeric filters may substitute fallback values for malformed input; conversion
alone is not validation. Where invalid input matters, use an explicit failure
path, as in the example above, and check applicable bounds.

Keep these three concepts distinct:

- **Parameters:** caller-supplied strings, accessed as `params.TARGET`.
- **Jinja locals:** render-time values created with `{% set ... %}`.
- **Macro variables:** `variable_<name>` values updated by
  `SET_GCODE_VARIABLE`; retained in memory between calls, not automatically
  persisted across restarts. Disk persistence is a separate Klipper feature.

A no-op macro used to hold shared settings, such as `_FFCP2`, is an established
Voron community pattern, not dead code. A leading underscore marks a helper by
convention and may hide it in frontend macro lists; it does not prevent callers
from invoking it.

For command overrides, document changes from stock semantics, the renamed base
command, and parameter forwarding. Preserve supported arguments with
`{rawparams}` when appropriate; intentionally unsupported arguments must be
documented rather than silently advertised as supported. Avoid accidental
recursion and collisions with integration-provided overrides.

## Evaluation Timing and State Contracts

Klipper renders an entire macro before executing its generated commands. A
`printer` value read later in the same template does not observe an earlier
`G28`, `ACTIVATE_EXTRUDER`, or `SET_GCODE_VARIABLE` command in that macro.
See [Klipper's template timing explanation][klipper-templates].

- Use a called helper macro when a decision needs state updated by preceding
  commands; that helper renders when execution reaches its call.
- Do not describe a Jinja loop over temperature snapshots as live polling.
  Use `TEMPERATURE_WAIT` for a blocking sensor wait, or an intentionally designed
  `delayed_gcode` loop for repeated later evaluations.
- Establish coordinate and extrusion modes before issuing moves, and give the
  first move an explicit feed rate. Preserve caller state when a utility macro
  should be transparent; document deliberate state changes in job-start macros.
- Use macro-specific state names, such as `prime_line_state`, to prevent nested
  helpers from overwriting one another's saved state.
- Write `MOVE=0` or `MOVE=1` explicitly when restoring state. G-code restoration
  normally does not move unless requested; dual-carriage restoration normally
  does. A return move needs its own clearance and collision review.
- `SAVE_GCODE_STATE` does not save active extruder selection, heater targets,
  fan state, filament sensor enables, extrusion synchronization, or `_FFCP2`
  variables. Dual-carriage state is separate again. Restore each required
  subsystem deliberately, especially in IDEX, COPY, and MIRROR modes.
- Do not add automatic homing to a heater-off or end-of-print path as a stylistic
  cleanup. Explain whether a macro requires homing, homes conditionally, or
  deliberately rehomes. Never assume homing is safe with a model on the bed.

## Units, Waits, and Status Messages

Use names and comments that distinguish temperatures (C), temperature tolerances
(C), rates (C/min or C/s), durations (s), and feed rates (`G1 F` in mm/min).
Klipper configuration velocity settings and command options such as `MOVE_SPEED`
typically use mm/s; check the specific command instead of assuming matching units.

Keep a temperature threshold independent of a cooling rate. For example,
`MAXIMUM={target + tolerance}` expresses a temperature limit; multiplying the
target by `cooling_per_second` does not. Test defaults as carefully as explicit
arguments: a passive cooldown wait near 0 C can remain blocked at room temperature.

Explain whether a wait has a timeout, and distinguish the **programmed ramp
duration** from measured cooling completion. Report the actual wait threshold,
or state its tolerance, so the displayed status agrees with the generated command.
Log transitions and periodic progress rather than flooding the console every
second. Preserve pause reasons instead of replacing them with routine status.

## Review and Validation Checklist

- [ ] Comments, defaults, units, prerequisites, and side effects match the code.
- [ ] Includes and helper dependencies resolve in the deployed configuration.
      Account for the external `mainsail.cfg`; it is not tracked here.
- [ ] The configuration parses and templates compile with Klipper's Jinja
      environment, including its single-brace expression delimiters.
- [ ] Rendered commands are checked for missing, malformed, or stray prose lines.
- [ ] Defaults, invalid parameters, boundary values, already-complete operations,
      and relevant tool/carriage modes have been tested offline.
- [ ] Formatting-only edits preserve rendered command order and meaning,
      including intentionally disabled operations and existing error behavior.
- [ ] No duplicate settings, unused state, or unreachable branches were added.
      A macro with no internal callers may still be a slicer/UI entry point.
- [ ] `git diff --check` passes, and unrelated user edits remain untouched.
- [ ] Any hardware validation is separate, deliberate, and performed only with
      appropriate clearance and thermal supervision. Never execute documentation
      examples on a live printer merely to validate formatting.

Passing parsing and template tests does **not** establish safe physical behavior.
Keep bug fixes distinct from mechanical reformatting in review and commit messages;
describe a bug and its consequences plainly instead of calling it style cleanup.

## Sources

Reviewed 2026-11-22. These are the sources for the conventions above, not an
assertion of a formal community vote or endorsement of this repository.

1. [Voron Documentation: Custom Macros][voron-macros] — macro structure,
   indentation, common entry points, and the request to document contributions.
2. [Voron community: Writing Klipper Macros, by voidtrance][voron-beginners] —
   uppercase naming conventions, helpers, shared-variable macros, topic files,
   evaluation timing, and state management. Community tutorial examples are
   illustrative; use upstream Klipper references for exact syntax and behavior.
3. [VoronDesign VORON2 Octopus configuration][voron-config] — section banners,
   grouped settings, four-space macro bodies, uppercase commands, lowercase
   Jinja locals, and explicit `MOVE=0` in the end-of-print state restoration.
4. [Voron community: mjonuschat macros][voron-community-example] — macro
   descriptions, underscore-prefixed helpers, and typed parameter defaults.
5. [Klipper: Command Templates][klipper-templates] — naming, descriptions,
   indentation, typed parameters, render timing, and state-saving guidance.
6. [Klipper: Configuration Reference, `gcode_macro`][klipper-config] — macro
   options, lowercase variable names, Python literals, and command overrides.
7. [Klipper: G-Codes][klipper-gcodes] — exact command semantics, including
   temperature waits and G-code versus dual-carriage state restoration.

[voron-macros]: https://docs.vorondesign.com/community/macros/
[voron-beginners]: https://docs.vorondesign.com/community/howto/voidtrance/Klipper_Macros_Beginners_Guide.html
[voron-config]: https://github.com/VoronDesign/Voron-2/blob/Voron2.4/firmware/klipper_configurations/Octopus/Voron2_Octopus_Config.cfg
[voron-community-example]: https://docs.vorondesign.com/community/macros/macros/mjonuschat.html
[klipper-templates]: https://www.klipper3d.org/Command_Templates.html
[klipper-config]: https://www.klipper3d.org/Config_Reference.html#gcode_macro
[klipper-gcodes]: https://www.klipper3d.org/G-Codes.html

# 3D Printing G-code Configuration Repository

## Repository Purpose

Custom G-code start sequences for Bambu Lab 3D printers, specifically optimized for the H2D machine with G10/Garolite build plates.

## G-code File Architecture

### Variable-Driven Configuration Pattern

All G-code files use **top-level variables** for material-specific parameters to enable easy customization without searching through the entire file:

```gcode
{global_variable_3 pla_standby_temp=140}
{global_variable_3 pla_soak_time=60}
```

**Critical Pattern**: When adding new filament support or modifying heating behavior:

1. Define variables at the top of the file (lines 5-40)
2. Reference them in conditional blocks using `[variable_name]` syntax
3. Never hardcode temperatures or times in the execution sections

### Optimization Strategy: "Heat-Level-Soak" Sequence

The start G-code implements a specific optimization sequence (see `OPTIMIZED` comment blocks):

1. **Bed heating first** (line ~130) - Heat bed to target while nozzle is at standby temp
2. **Bed leveling at standby temps** (line ~377) - Perform G29 leveling with nozzle at safe material-specific standby temperature
3. **Chamber heating and material soak** (line ~433) - Only after leveling, soak at standby temp for material-specific duration
4. **Final heating** - Rise to print temperature

**Why this matters**: Traditional sequences heat to print temp before leveling, wasting time and energy. This approach minimizes nozzle oozing during leveling while keeping total preheat time optimal.

### Filament Type Conditional Blocks

Material-specific behavior uses nested conditionals checking `filament_type[initial_no_support_extruder]`:

```gcode
{if filament_type[initial_no_support_extruder]=="PLA"}
M104 S[pla_standby_temp] A
{endif}
```

**Pattern**: 16 supported filament types (PLA, PETG, TPU, ABS, ASA, PC, PA, PA-CF, PA6-GF, PA6-CF, PAHT-CF, PET-CF, PPA-CF, PPS-CF, PVA, Support) each requiring:

- Standby temperature definition
- Soak time definition
- Conditional block for standby temp assignment
- Conditional block for soak time execution

## File Organization

```
bambu_studio/
  machine_start_gcode/
    darkmoon_g10_garolite.gcode  # H2D printer with G10 plate configuration
```

**Naming Convention**: `{machine}_{plate_type}.gcode` - Files are named for the specific printer model and build surface material combination.

## Editing Guidelines

### Adding New Filament Type

1. Add `{global_variable_3 newmat_standby_temp=XXX}` in variables section (lines 5-40)
2. Add `{global_variable_3 newmat_soak_time=XXX}` in variables section
3. Add conditional block at line ~135 (standby temp assignment)
4. Add conditional block at line ~445 (soak time execution)

### Modifying Heating Behavior

- **Standby temps**: Adjust variable values at top (typically print_temp - 50-80°C)
- **Soak times**: Based on material thermal mass needs (PLA: 60s, high-temp engineering: 300s+)
- Do not modify the sequence order without understanding the optimization strategy

### Bambu Studio Template Variables

The G-code uses Bambu Studio's templating system:

- `{filament_type[initial_no_support_extruder]}` - Current filament material
- `{bed_temperature_initial_layer[initial_no_support_extruder]}` - Target bed temp from slicer
- `{nozzle_temperature_initial_layer[initial_no_support_extruder]}` - Target nozzle temp from slicer
- `{overall_chamber_temperature}` - Chamber heating target
- `{curr_bed_type}` - Current build plate type

**These are runtime-substituted by Bambu Studio** - don't treat them as editable variables.

## Testing & Validation

- Changes affect print start sequence which runs before every print
- Test with a small calibration print to verify heating timing
- Monitor for nozzle oozing during leveling (indicates standby temp too high)
- Check total preheat time hasn't increased significantly

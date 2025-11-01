# 3D Printing G-code Configuration Repository

## Repository Purpose

Custom G-code start sequences for Bambu Lab 3D printers, specifically optimized for the H2D machine with different build plate surfaces (G10/Garolite and CFX Carbon Fiber).

## G-code File Architecture

### Variable-Driven Configuration Pattern

All G-code files use **top-level variables** for material-specific parameters to enable easy customization without searching through the entire file:

```gcode
{global_variable_3 pla_standby_temp=140}
{global_variable_3 pla_soak_time=60}
```

**Critical Pattern**: When adding new filament support or modifying heating behavior:

1. Define variables at the top of the file (lines 5-45)
2. Reference them in conditional blocks using `[variable_name]` syntax
3. Never hardcode temperatures or times in the execution sections

### Build Plate Specific Configurations

Each build plate has unique thermal properties requiring different soak times:

- **G10/Garolite** (`darkmoon_g10_garolite.gcode`): Standard soak times for glass-reinforced epoxy
- **CFX Carbon Fiber** (`darkmoon_cfx_carbonfiber.gcode`): Adjusted soak times based on [official user guide](https://darkmoon3d.com/pages/darkmoon-cfx-carbon-fiber-build-plate-user-guide)
  - Carbon fiber is an excellent thermal insulator requiring plate-specific preheat strategies
  - Some materials require NO preheat (PLA, TPU, PPS-CF: 0s)
  - Some materials ALWAYS preheat (PETG: 180s, PC: 300s)
  - Engineering materials use moderate times (PA/PET-CF: 120-240s)

**When creating new plate configs**: Copy an existing file and adjust soak times based on manufacturer specifications. Standby temps remain the same across plates.

### Optimization Strategy: "Heat-Level-Soak" Sequence

The start G-code implements a specific optimization sequence (see `OPTIMIZED` comment blocks):

1. **Bed heating first** (line ~135) - Heat bed to target while nozzle is at standby temp
2. **Bed leveling at standby temps** (line ~380) - Perform G29 leveling with nozzle at safe material-specific standby temperature
3. **Chamber heating and material soak** (line ~440) - Only after leveling, soak at standby temp for material-specific duration
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

- Standby temperature definition (same across all plates)
- Soak time definition (plate-specific)
- Conditional block for standby temp assignment (line ~140)
- Conditional block for soak time execution (line ~445)

## File Organization

```
bambu_studio/
  machine_start_gcode/
    darkmoon_g10_garolite.gcode     # H2D with G10/Garolite plate
    darkmoon_cfx_carbonfiber.gcode  # H2D with CFX Carbon Fiber plate
```

**Naming Convention**: `darkmoon_{plate_type}.gcode` - Files are named for the build surface material. Machine type (H2D) is documented in header comments.

## Editing Guidelines

### Adding New Filament Type

1. Add `{global_variable_3 newmat_standby_temp=XXX}` in variables section (lines 9-24)
2. Add `{global_variable_3 newmat_soak_time=XXX}` in variables section (lines 27-43)
3. Add conditional block at line ~140 (standby temp assignment)
4. Add conditional block at line ~450 (soak time execution)
5. **Repeat for ALL plate configurations** - each plate may have different soak times

### Adding New Build Plate Configuration

1. Copy the most similar existing plate config file
2. Update header comments: date, build plate name, user guide link (if available)
3. Adjust soak time variables based on plate's thermal properties and manufacturer specs
4. Keep standby temperatures unchanged (material property, not plate property)
5. Test with representative materials to verify heating timing

### Testing Soak Times for Plates Without Manufacturer Guidance

When manufacturer specifications aren't available, use this methodology to determine optimal soak times:

1. **Establish baseline**: Start with G10/Garolite times as reference
2. **Consider plate material thermal properties**:
   - High thermal conductivity (metals, graphite): reduce soak by 20-30%
   - High thermal insulation (carbon fiber, FR4): increase soak by 30-50%
   - Standard surfaces (PEI, glass): use baseline times
3. **Test protocol** (per material category):
   - Print a 20x20mm adhesion test square
   - Start with estimated time, adjust ±30-60s based on:
     - First layer adhesion quality (too little = poor adhesion)
     - Visible temperature stability (infrared thermometer helpful)
     - Warping behavior on large parts
4. **Document findings**: Add inline comments with rationale for chosen times
5. **Iterate**: Test with materials at thermal extremes (PLA, PA, PC)

### Machine-Specific Considerations

When adapting G-code for different Bambu Lab machines:

- **H2D** (current focus): Large 350mm bed, dual extruders, requires longer heating times
- **X1C/P1**: Standard 256mm bed, faster heating, reduce soak times by ~20%
- **A1/A1 Mini**: Smaller bed (184mm mini), much faster heating, reduce soak by ~40%

**Machine-specific variables to consider**:

- Bed size affects thermal mass (larger = more soak time)
- Heated chamber presence (X1C has chamber, affects ABS/ASA strategy)
- Extruder configuration (single vs dual affects toolhead calibration sequence)

**Naming convention for machine variants**: `{machine}_{plate_type}.gcode`

- Example: `x1c_cfx_carbonfiber.gcode`, `a1mini_g10_garolite.gcode`

### Build Plate Thermal Property Reference

For future plate additions, common build surfaces and their characteristics:

| Surface Type          | Thermal Property    | Soak Time Adjustment | Notes                             |
| --------------------- | ------------------- | -------------------- | --------------------------------- |
| G10/Garolite          | Moderate insulator  | Baseline (reference) | Glass-reinforced epoxy            |
| CFX Carbon Fiber      | Excellent insulator | +50-80% vs baseline  | Requires extended preheat per mfg |
| PEI (smooth/textured) | Good conductor      | -10-20% vs baseline  | Quick heat transfer               |
| Borosilicate Glass    | Good conductor      | -15-25% vs baseline  | Even heat distribution            |
| FR4 PCB Material      | Moderate insulator  | Baseline to +20%     | Similar to G10                    |
| Anodized Aluminum     | Excellent conductor | -30-40% vs baseline  | Fastest heat transfer             |
| Polypropylene Sheet   | Poor conductor      | +100%+ vs baseline   | Extreme insulation                |

Use this as starting point, then apply testing methodology above to refine.

### Modifying Heating Behavior

- **Standby temps**: Adjust variable values at top (typically print_temp - 50-80°C)
- **Soak times**: Based on material thermal mass AND plate thermal properties
  - Reference manufacturer documentation when available
  - Consider plate material (glass, carbon fiber, PEI) as thermal insulator/conductor
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
- For new plate configs: test with materials at extremes (PLA, PETG, PA, PC) to verify behavior

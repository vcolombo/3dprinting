# 3D Printing G-code Configuration Repository

## Repository Purpose

Custom G-code start sequences for Bambu Lab 3D printers, specifically optimized for the H2D machine with different build plate surfaces (G10/Garolite, CFX Carbon Fiber, and Satin Modified PEI).

## G-code File Architecture

### Hardcoded Configuration Pattern

**CRITICAL**: Bambu Studio does NOT support custom variable definitions (`{global_variable_3 ...}` syntax). All material-specific parameters are **hardcoded directly in conditional blocks**.

```gcode
{if filament_type[initial_no_support_extruder]=="PLA"}
M104 S140 A          ; PLA: standby temp 140C
{endif}
```

**When adding new filament support or modifying heating behavior**:

1. Locate the material-specific conditional blocks (~line 100 for standby temps, ~line 410 for soak times)
2. Add new conditional with hardcoded values
3. **Repeat for ALL plate configurations** - each may have different soak times but same standby temps

### Build Plate Specific Configurations

Each build plate has unique thermal properties requiring different soak times:

- **G10/Garolite** (`darkmoon_g10_garolite.gcode`): Standard soak times for glass-reinforced epoxy (baseline configuration)
- **CFX Carbon Fiber** (`darkmoon_cfx_carbonfiber.gcode`): Adjusted soak times for excellent thermal insulator
  - Carbon fiber requires plate-specific preheat strategies per manufacturer specs
  - Some materials require NO preheat (PLA, TPU, PPS-CF: 0s)
  - Some materials ALWAYS preheat (PETG: 180s, PC: 300s)
  - Engineering materials use moderate times (PA/PET-CF: 120-240s)
- **Satin Modified PEI** (`darkmoon_satin.gcode`): Reduced soak times for good thermal conductor
  - Modified PEI polymer with excellent thermal conductivity
  - Shorter preheat times (PLA/TPU/PVA: 0s, PETG: 60s, most engineering: 180s)

**When creating new plate configs**: Copy an existing file and adjust soak times in conditional blocks based on manufacturer specifications. Standby temps (120-240°C) remain the same across all plates.

### Special Cases: Material-Specific Adhesion Behavior

#### PPS-CF on CFX Carbon Fiber: Zero Soak Despite High Standby Temperature

PPS-CF is unique - it uses **0s soak time** on CFX despite having the highest standby temperature (240°C). This is intentional:

- **Chemical bonding (van der Waals forces)** occurs instantly at 240°C standby temperature
- Extended thermal soak does NOT improve adhesion (unlike PC, which is thermally-dependent)
- Longer soak actually risks thermal drift/warping before printing starts
- This is manufacturer-tested and validated behavior

When adding materials with chemical adhesion properties, research the adhesion mechanism before assuming thermal soak helps.

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
M104 S140 A          ; PLA: standby temp 140C
{endif}
```

**Pattern**: 18 supported filament types (PLA, PLA-CF, PETG, PETG-CF, TPU, ABS, ASA, PC, PA, PA-CF, PA6-GF, PA6-CF, PAHT-CF, PET-CF, PPA-CF, PPS-CF, PVA, Support) each requiring:

- Standby temperature hardcoded in conditional (same across all plates: 120-240°C)
- Soak time hardcoded in conditional (plate-specific: 0-330s)
- Conditional block for standby temp assignment (~line 100)
- Conditional block for soak time execution (~line 410)

## File Organization

```
bambu_studio/
  H2D/
    machine_start_gcode/
      darkmoon_g10_garolite.gcode     # H2D with G10/Garolite plate
      darkmoon_cfx_carbonfiber.gcode  # H2D with CFX Carbon Fiber plate
      darkmoon_satin.gcode            # H2D with Satin Modified PEI plate
```

**Naming Convention**: `darkmoon_{plate_type}.gcode` - Files are named for the build surface material. Machine type (H2D) is documented in header comments.

## Editing Guidelines

### Adding New Filament Type

1. Add conditional block at ~line 100 (standby temp assignment):

```gcode
{if filament_type[initial_no_support_extruder]=="NEWMAT"}
M104 S180 A          ; NEWMAT: standby temp 180C
{endif}
```

2. Add conditional block at ~line 410 (soak time execution):

```gcode
{if filament_type[initial_no_support_extruder]=="NEWMAT"}
G4 S120              ; NEWMAT: soak 120s
{endif}
```

3. **Repeat for ALL plate configurations** - each plate may have different soak times but same standby temps

### Adding New Build Plate Configuration

1. Copy the most similar existing plate config file
2. Update header comments: date, build plate name, user guide link (if available)
3. Adjust soak time values in conditional blocks (~line 410) based on plate's thermal properties and manufacturer specs
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

- **Standby temps**: Adjust hardcoded values in conditional blocks at ~line 100 (typically print_temp - 50-80°C)
- **Soak times**: Adjust hardcoded values in conditional blocks at ~line 410
  - Based on material thermal mass AND plate thermal properties
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

## Version Control Workflow

### Branch Strategy

- `main` - Stable, tested configurations ready for production use
- Feature branches - Use descriptive names for changes:
  - `add-{material-name}` - Adding new filament type support
  - `add-{plate-name}` - Adding new build plate configuration
  - `fix-{issue-description}` - Bug fixes or timing adjustments
  - `optimize-{improvement}` - Performance or sequence optimizations

### Commit Message Format

```
<type>: <short description>

<detailed explanation if needed>

Affected files:
- darkmoon_g10_garolite.gcode
- darkmoon_cfx_carbonfiber.gcode
- darkmoon_satin.gcode
```

**Types**:

- `feat`: New filament type or plate configuration
- `fix`: Correction to temps, times, or sequence
- `refactor`: Code reorganization without behavior change
- `docs`: Documentation updates only
- `test`: Testing methodology or validation changes

**Examples**:

```
feat: add ASA support to all plate configs

Added ASA with 170C standby temp and plate-specific soak times:
- G10: 240s (similar to ABS)
- CFX: 240s (manufacturer recommended)
- Satin: 180s (good conductor)

Affected files:
- darkmoon_g10_garolite.gcode
- darkmoon_cfx_carbonfiber.gcode
- darkmoon_satin.gcode
```

```
fix: reduce PETG soak time on Satin from 90s to 60s

Testing showed 90s was excessive for Satin's high thermal conductivity.
First layer adhesion remained excellent at 60s with faster start time.

Affected files:
- darkmoon_satin.gcode
```

### Before Committing Changes

1. **Verify syntax**: Check all three files if adding new material
2. **Test in Bambu Studio**: Ensure G-code parses without errors
3. **Update line number references**: If adding/removing large blocks, update documentation
4. **Document rationale**: Add inline comments explaining non-obvious values (especially special cases like PPS-CF)

### Pull Request Guidelines

When creating PRs for collaborative work:

1. Title should match commit message format
2. Include test results (which materials tested, any issues observed)
3. Reference manufacturer documentation if applicable
4. Note any deviations from baseline values with justification

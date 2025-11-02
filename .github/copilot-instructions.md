# 3D Printing G-code Configuration Repository

## Repository Purpose

Custom G-code start sequences for Bambu Lab 3D printers, specifically optimized for the H2D machine with three build plate surfaces: G10/Garolite, CFX Carbon Fiber, and Darkmoon Satin (Modified PEI).

## G-code File Architecture

### CRITICAL: Hardcoded Configuration Pattern

**Bambu Studio does NOT support custom variable definitions**. All G-code files use hardcoded material-specific values directly in conditional blocks:

```gcode
{if filament_type[initial_no_support_extruder]=="PLA"}
M104 S140 A          ; PLA: standby temp 140C
{endif}
```

**Why this matters**: The `{global_variable_3 variable_name=value}` syntax causes parsing errors. Bambu Studio only supports:

- Built-in placeholders: `{bed_temperature_initial_layer[initial_no_support_extruder]}`, `{nozzle_temperature_initial_layer[initial_no_support_extruder]}`, etc.
- Conditionals: `{if ...}{endif}`
- Variable references to built-in placeholders only

**When adding new filament support**:

1. Add conditional block with hardcoded standby temp in the standby temperature section (~line 100)
2. Add conditional block with hardcoded soak time in the material soak section (~line 420)
3. Add conditional block with hardcoded temp in Z offset calibration section (~line 395)
4. **Repeat for ALL three plate configurations** - soak times vary by plate thermal properties

### Build Plate Specific Configurations

Each build plate has unique thermal properties requiring different soak times:

- **G10/Garolite** (`darkmoon_g10_garolite.gcode`):

  - Moderate insulator (glass-reinforced epoxy)
  - Standard soak times: PLA 30s, PETG 90s, Engineering 240-330s
  - Product page: https://darkmoon3d.com/products/bambu-lab-g10-build-plate

- **CFX Carbon Fiber** (`darkmoon_cfx_carbonfiber.gcode`):

  - Excellent insulator requiring extended preheat per [manufacturer guide](https://darkmoon3d.com/pages/darkmoon-cfx-carbon-fiber-build-plate-user-guide)
  - Material-specific strategy: NO preheat (PLA/TPU/PPS-CF: 0s), ALWAYS preheat (PETG: 180s, PC: 300s), moderate (PA/PET-CF: 120-240s)
  - **PPS-CF Special Case**: 0s soak despite highest temp (240°C) - chemical adhesion is instant, thermal soak risks drift

- **Darkmoon Satin** (`darkmoon_satin.gcode`):
  - Good conductor (modified PEI polymer blend)
  - Reduced soak times: PLA 0s, PETG 60-90s, Engineering 150-180s
  - User guide: https://darkmoon3d.com/pages/darkmoon-satin-build-plate-user-guide

**Standby temperatures** (same across all plates, material property not plate property):

- PLA: 140°C, PLA-CF: 145°C, TPU: 120°C
- PETG: 160°C, PETG-CF: 165°C
- ABS/ASA: 170°C
- PA/PA-CF/PA6-GF/PA6-CF: 190°C
- PC/PAHT-CF: 200°C, PET-CF: 210°C, PPA-CF: 220°C, PPS-CF: 240°C
- PVA/Support: 140°C

### Optimization Strategy: "Heat-Level-Soak" Sequence

The start G-code implements a specific optimization to minimize print start time while preventing nozzle oozing:

1. **Bed heating first** (~line 95): Heat bed to target, nozzle stays cold
2. **Standby temp heating** (~line 100): Set nozzle to material-specific standby temp (NOT print temp)
3. **Bed leveling at standby temps** (~line 350): Perform G29 leveling without oozing
4. **Z offset calibration** (~line 390): Uses same material-specific standby temps
5. **Chamber heating and material soak** (~line 410): Dwell at standby temp for plate-specific duration
6. **Final heating** (~line 575): Rise to print temperature just before printing

**Why this matters**: Traditional sequences heat to print temp before leveling, causing oozing and wasting energy. This approach keeps nozzle at safe temps until needed.

**Status Messages**: Added `M1002 gcode_claim_action` commands throughout to display progress in Bambu Studio:

- Action 1: Bed leveling
- Action 2: Bed heating
- Action 3: Mechanical mode calibration
- Action 4: AMS operations
- Action 8: Extrusion calibration
- Action 10: Hotend heating
- Action 13: First homing
- Action 14: Nozzle wiping
- Action 18: Material soak phase (heating bed & hotend)
- Action 29: Chamber temperature (cooling)
- Action 39: XY offset calibration
- Action 49: Chamber heating

**Available for future use**: Actions 0 (idle/complete), 5-7, 9, 11-12, 15-17, 19-28, 30-38, 40-48, 50+

### Filament Type Conditional Blocks

Material-specific behavior uses 18 conditionals checking `filament_type[initial_no_support_extruder]`:

```gcode
{if filament_type[initial_no_support_extruder]=="PETG"}
M104 S160 A          ; PETG: standby temp 160C
{endif}
```

**Supported materials**: PLA, PLA-CF, PETG, PETG-CF, TPU, ABS, ASA, PC, PA, PA-CF, PA6-GF, PA6-CF, PAHT-CF, PET-CF, PPA-CF, PPS-CF, PVA, Support

**Three sections require updates** when adding materials:

1. Standby temperature assignment (~line 100-160)
2. Material soak time (~line 420-480)
3. Z offset calibration temperature (~line 395-450)

## File Organization

```
bambu_studio/
  H2D/
    machine_start_gcode/
      darkmoon_g10_garolite.gcode     # H2D with G10/Garolite plate (702 lines)
      darkmoon_cfx_carbonfiber.gcode  # H2D with CFX Carbon Fiber plate (702 lines)
      darkmoon_satin.gcode            # H2D with Satin Modified PEI plate (702 lines)
```

**Naming Convention**: `darkmoon_{plate_type}.gcode` - Named for build surface. Machine type (H2D) in header comments.

**File Structure** (all three files follow same pattern):

- Lines 1-8: Header with plate info and optimization notes
- Lines 15-95: Machine initialization and airduct setup
- Lines 95-160: Bed + standby temp heating with material conditionals
- Lines 160-350: Homing, detection, material prep, extrusion calibration
- Lines 350-390: Bed leveling at standby temps
- Lines 390-450: Z offset calibration with material-specific temps
- Lines 410-480: Chamber heating and material soak conditionals
- Lines 480-650: Mech calibration, XY offset, final heating, purge line

## Editing Guidelines

### Adding New Filament Type

Must update **three locations in each of three files** (9 edits total):

1. **Standby temp section** (~line 100):

   ```gcode
   {if filament_type[initial_no_support_extruder]=="NEWMAT"}
   M104 S170 A          ; NEWMAT: standby temp 170C
   {endif}
   ```

2. **Z offset calibration** (~line 395):

   ```gcode
   {if filament_type[initial_no_support_extruder]=="NEWMAT"}
   G383 O0 M2 T170
   {endif}
   ```

3. **Material soak** (~line 420, **plate-specific times**):
   ```gcode
   {if filament_type[initial_no_support_extruder]=="NEWMAT"}
   G4 S120              ; NEWMAT: soak 120s (adjust per plate)
   {endif}
   ```

**Determining standby temp**: Print temp - 50-80°C (enough to prevent oozing, not too cold)

**Determining soak time**: Use thermal property table below as baseline, test with adhesion squares

### Adding New Build Plate Configuration

1. Copy the most similar existing plate config file (based on thermal properties)
2. Update header comments (lines 1-7): date, build plate name, user guide link (if available)
3. Adjust all 18 material soak times (~line 420-480) based on plate thermal properties
4. **DO NOT change standby temperatures** - those are material properties, not plate properties
5. Test with materials at thermal extremes (PLA at low end, PC/PPS-CF at high end)

**Naming**: `darkmoon_{plate_surface}.gcode` in `bambu_studio/H2D/machine_start_gcode/`

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

- **H2D** (current): 350mm bed, dual extruders, ~702 line G-code, full feature set
- **X1C/P1**: 256mm bed, reduce soak times by ~20%, faster heating
- **A1/A1 Mini**: 184mm bed (mini), reduce soak by ~40%, much simpler startup

**Key variables affected**:

- Bed thermal mass (larger bed = longer soak)
- Chamber heating (X1C yes, A1 no - affects ABS/ASA strategy)
- Dual vs single extruder (affects toolhead calibration blocks)

**Naming**: `{machine}_{plate_type}.gcode` (e.g., `x1c_cfx_carbonfiber.gcode`)

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

- **Standby temps**: Typically print_temp - 50-80°C to prevent oozing without being too cold
- **Soak times**: Based on BOTH material thermal mass AND plate thermal properties
  - Always reference manufacturer documentation when available
  - Consider plate thermal characteristics (insulator vs conductor)
- **DO NOT modify sequence order** without understanding the Heat-Level-Soak optimization strategy

### Bambu Studio Template Variables

Built-in placeholders (runtime-substituted by Bambu Studio):

- `{filament_type[initial_no_support_extruder]}` - Current filament material
- `{bed_temperature_initial_layer[initial_no_support_extruder]}` - Target bed temp
- `{nozzle_temperature_initial_layer[initial_no_support_extruder]}` - Target nozzle temp
- `{overall_chamber_temperature}` - Chamber target
- `{curr_bed_type}` - Build plate type

**Important**: Custom variables like `{global_variable_3 ...}` are NOT supported and cause parsing errors

## Testing & Validation

- Changes affect print start sequence which runs before every print
- Test with a small calibration print to verify heating timing
- Monitor for nozzle oozing during leveling (indicates standby temp too high)
- Check total preheat time hasn't increased significantly
- For new plate configs: test with materials at extremes (PLA, PETG, PA, PC) to verify behavior

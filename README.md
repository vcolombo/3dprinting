# 3D Printing G-code Configurations

Custom G-code start sequences for Bambu Lab 3D printers, optimized for different build plate surfaces.

## Overview

This repository contains machine-specific G-code configurations that implement an optimized "Heat-Level-Soak" sequence to minimize print start time while preventing nozzle oozing during bed leveling. Each configuration is tailored to specific build plate thermal properties.

## Structure

```text
bambu_studio/
  H2D/
    machine_start_gcode/
      darkmoon_g10_garolite.gcode     # G10/Garolite plate
      darkmoon_cfx_carbonfiber.gcode  # CFX Carbon Fiber plate
      darkmoon_satin.gcode            # Satin (Modified PEI) plate
```

## Build Plate Configurations

### Darkmoon G10/Garolite

- **Thermal Property**: Moderate insulator (glass-reinforced epoxy)
- **Use Case**: Baseline configuration, reliable for most materials
- **Soak Times**: Standard (PLA: 60s, PETG: 90s, Engineering: 240-330s)

### Darkmoon CFX Carbon Fiber

- **Thermal Property**: Excellent insulator (requires extended preheat)
- **Use Case**: Engineering filaments (Nylon, PC, PET-CF, PPS-CF)
- **Soak Times**: Extended for PETG (180s) and PC (300s); none for PLA/TPU/PPS-CF
- **User Guide**: [CFX Build Plate Guide](https://darkmoon3d.com/pages/darkmoon-cfx-carbon-fiber-build-plate-user-guide)

### Darkmoon Satin (Modified PEI)

- **Thermal Property**: Good conductor (modified PEI polymer blend)
- **Use Case**: Superior adhesion with matte bottom finish
- **Soak Times**: Reduced (most materials 0-180s, faster heating)
- **User Guide**: [Satin Build Plate Guide](https://darkmoon3d.com/pages/darkmoon-satin-build-plate-user-guide)

## Features

- **Variable-Driven Configuration**: All material-specific parameters defined at top of file for easy customization
- **16 Filament Types Supported**: PLA, PETG, TPU, ABS, ASA, PC, PA, PA-CF, PA6-GF, PA6-CF, PAHT-CF, PET-CF, PPA-CF, PPS-CF, PVA, Support
- **Optimized Heating Sequence**:
  1. Heat bed first
  2. Bed leveling at material-specific standby temps
  3. Chamber heating and soak (plate-specific times)
  4. Final heating to print temperature

## Usage

1. Choose the G-code file matching your build plate
2. Import into Bambu Studio/Orca Slicer as machine start G-code
3. Adjust soak times in variables section if needed for your environment

## Contributing

See [`.github/copilot-instructions.md`](.github/copilot-instructions.md) for detailed editing guidelines and patterns.

## License

See [LICENSE](LICENSE) file for details.

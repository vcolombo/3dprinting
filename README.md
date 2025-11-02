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
- **Soak Times**: Standard (PLA: 30s, PETG: 90s, Engineering: 240-330s)
- **Product Page**: [G10 Build Plate](https://darkmoon3d.com/products/bambu-lab-g10-build-plate)

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

- **Material-Specific Configuration**: Hardcoded material-specific standby temperatures and soak times optimized for each build plate
- **18 Filament Types Supported**: PLA, PLA-CF, PETG, PETG-CF, TPU, ABS, ASA, PC, PA, PA-CF, PA6-GF, PA6-CF, PAHT-CF, PET-CF, PPA-CF, PPS-CF, PVA, Support
- **Optimized Heating Sequence**:
  1. Heat bed first with status display
  2. Bed leveling at material-specific standby temps (no oozing)
  3. Chamber heating and material soak (plate-specific times)
  4. Final heating to print temperature
- **Enhanced Status Messages**: Continuous status updates throughout the entire start sequence for better visibility in Bambu Studio

## Usage

1. Choose the G-code file matching your build plate
2. Import into Bambu Studio/Orca Slicer as machine start G-code
3. Adjust soak times in variables section if needed for your environment

## Contributing

See [`.github/copilot-instructions.md`](.github/copilot-instructions.md) for detailed editing guidelines and patterns.

## License

See [LICENSE](LICENSE) file for details.

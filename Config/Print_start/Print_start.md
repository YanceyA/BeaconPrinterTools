# Print Start Macro
This is a full featured print start macro from Beacon 3d probe users. 

# Steps
1. Copy file or contents into your printer config [Print Start Macro](bacon_print_start.cfg)
2. Review macro contents and enable any features required
3. Add in the "Start g-code" for your slicer of choice.

## :warning: Required change in your slicer :warning:
You need to update your "Start g-code" in your slicer by adding a few lines of code. This will send data about your print temps and chamber temp to klipper for each print.

### SuperSlicer
In Superslicer go to "Printer settings" -> "Custom g-code" -> "Start G-code" and update it to:

```
M104 S0 ; Stops SuperSlicer from sending temp waits separately
M140 S0
print_start EXTRUDER=[first_layer_temperature] BED=[first_layer_bed_temperature] CHAMBER=[chamber_temperature]
```

### PrusaSlicer

:warning: PrusaSlicer doesn't give you the option to set a specific chambertemp. Therefor you it will fallback to the standard chambertemp of 40c. 

In PrusaSlicer go to "Printer settings" -> "Custom g-code" -> "Start G-code" and update it to:

```
M104 S0 ; Stops PrusaSlicer from sending temp waits separately
M140 S0
print_start EXTRUDER=[first_layer_temperature[initial_extruder]] BED=[first_layer_bed_temperature]
```

### Cura

In Cura go to "Settings" -> "Printer" -> "Manage printers" -> "Machine settings" -> "Start G-code" and update it to:

```
print_start EXTRUDER={material_print_temperature_layer_0} BED={material_bed_temperature_layer_0} CHAMBER={build_volume_temperature}
```

### OrcaSlicer
In OrcaSlicer go to "Printer settings" -> "Machine start g-code" and update it to:

```
M104 S0 ; Stops OrcaSlicer from sending temp waits separately
M140 S0
print_start EXTRUDER=[first_layer_temperature] BED=[first_layer_bed_temperature] CHAMBER=[chamber_temperature]
```

# Macro Outline
- Conditional G28 home
- Move to bed center
- Clear bed mesh and z offsets (if any)
- Turns on part cooling fans and nevermore if present
- Positions bed for heat soak
- Waits for chamber temperature sensor if configured
- Wipes nozzle
- G28 Z contact + calibrate
- Heat nozzle to contact temp
- Tilt or QGL
- Bed mesh (adaptive or not)
- Wipe Nozzle
- Center of bed move
- Part Cool fan off and heat to print temp
- Set temp expansion compensation
- Apply z_adjust if required
- Purge line if configured

Shoutouts:
Brazuka on discord for the fast_QGL macro!
Jontek2 for the better print_start macro started with. https://github.com/jontek2/A-better-print_start-macro

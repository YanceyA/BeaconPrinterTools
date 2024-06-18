# Print Start Macro
This is a full featured print start macro from Beacon 3d probe users. 

# Steps
1. Copy file or contents into your printer config [Print Start Macro](bacon_print_start.cfg)
2. Review macro contents and enable any features required

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

(Shoutout to Brazuka on discord for the fast_QGL macro!)
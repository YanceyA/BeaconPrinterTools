> [!IMPORTANT]
> This macro is in development and performance is not guarenteed. Please provide feedback on any issues or make PRs for changes.

# Beacon Thermal Expansion Compensation
This is a macro set that will calibrate and apply a GCODE Z offset to compensate for thermal expansion of the hotend. This is the thermal expansion that occuring as the system heats from standard probe temperatures (~150-180C) to printing temperatures (~220-350C). The macro calibrates an expansion coefficent and then calculates a specific z-offset based on the desired print temperature.

# Calibration Routine Description
1. Home all axis and perform a QGL/Tilt. 
2. 10 poke operations to settle down the printer mechanics. 
3. Heat to 150C, wait for the temperature to stabilize for 60s, and perform a probe operation.
4. Heat to 250C, wait for the temperature to stabilize for 60s, and perform a probe operation.
5. Repeat steps 3 and 4 again to ensure the results are repeatable.
6. The hotend is moved back to a central position and the nozzle expansion data saved to the variables file. 

Depending on how fast the hotend heats and cools down this could take 5-10min to complete both probe cycles.

A seperate macro called in print start will use the nozzle expansion coefficent and extruder print temperature to calculate and apply the appropriate z_offset automatically at print time.

# Pre-Reqs
- Beacon sensor updated to latest firmware version
- Contact mode setup and enabled on the beacon. See [https://docs.beacon3d.com/contact/]
- [save_variables] configured. An example given below:
```
[save_variables]
filename: ~/printer_data/config/variables.txt
```
- Ooze mitigation if 250C nozzle temp will lead to filament ooze. Best practice would be to perform this without filament to ensure no ooze is present.
- Max nozzle contact temperature set to >250C. The default maximum probe temperature is set to 180C and will cause an error during 250C probing.
```
[beacon]
BEACON_CALIBRATE_NOZZLE_TEMP_OFFSET: 275
```

> [!CAUTION]
> Probing at higher temperatures, 250C, can cause damage to some build plates. Ensure that your setup can handle 250C contact probing without damage. Refer to the beacon contant documentation for additional details. 

# Setup Steps
1. Copy macro block into your config
2. Review/adjust the default variables in BEACON (reccomended to use defaults right now)
3. Add lines to PRINT_START and PRINT_END macros
4. Restart klipper
5. Run BEACON_CALIBRATE_NOZZLE_TEMP_OFFSET to perform the nozzle expansion calibration.
6. Review the variables file to ensure there is a nozzle_expansion_coefficient entry.
  ```
  [Variables]
  nozzle_expansion_coefficient = 0.04749999999994525
  ```
7. Run a print and monitor the first layer.

# Fine Tuning

If additional tuning is required for the first layer squish the following two methods are avaliable, in order of preference:

## Adjust Expansion Multiplier
The BEACON macro has a variable beacon_contact_expansion_multiplier that can be adjusted for fine tuning. This multiplier keeps the expansion compensation aligned to temperature and has emperically been found to be the best method to fine tune expansion compensation to print quality. A value of 1.1 produces 10% less first layer squish and 0.9 produces 10% more first layer squish.
```
variable_beacon_contact_expansion_multiplier: 1.0 
```

## Gcode Offset
A gcode offset could be added to the PRINT_START macro. This could be hard coded or passed via the slicer as {OFFSET}. Note that this type of offset is not temperature dependent like the expansion multiplier.
```
SET_GCODE_OFFSET Z_ADJUST={OFFSET}
```

# Print Start
Apply this to the print start macro to calculate and apply a GCODE offset Z_adjust to the print. Note that this replaces "SET_GCODE_OFFSET" gcode lines as reccomended in the Beacon Contact Docs.

> [!Note]
> Nozzle expansion calibration should be stable and only run if a mechanical change is made. BEACON_CALIBRATE_NOZZLE_TEMP_OFFSET should not need to be called in the PRINT_START macro.

```
# set nozzle thermal expansion offset
{% if printer.configfile.settings.beacon is defined %}
    _BEACON_SET_NOZZLE_TEMP_OFFSET 
{% endif %}
```

# Print End
This removes the applied offset at the end of the print.

```
# reset nozzle thermal expansion offset
{% if printer.configfile.settings.beacon is defined %}
    _BEACON_REMOVE_NOZZLE_TEMP_OFFSET
    _BEACON_SET_NOZZLE_TEMP_OFFSET RESET=True
{% endif %}
```

# Additional Configurations
Some printers may require specific settings to ensure that the probe operations are completed with best accuracy. Specifically the probe speeds up/down and the probe retract distance. If these are not specified in [beacon] then the probe defaults are used.

```
[PROBE PARAMETER NAME] -> [BEACON CONFIG NAME]
PROBE_SPEED            -> autocal_speed
LIFT_SPEED             -> autocal_retract_speed
SAMPLE_RETRACT_DIST    -> autocal_retract_dist
```

# Credits

Credit to RatOS team {Helge Keck and Mikkel Schmidt} for developing this macro. You can see their work in Beacon.cfg: https://github.com/Rat-OS/RatOS-configuration/blob/v2.1.x/z-probe/beacon.cfg


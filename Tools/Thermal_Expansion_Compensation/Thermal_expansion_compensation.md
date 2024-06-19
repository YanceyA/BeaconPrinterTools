# Beacon Thermal Expansion Compensation
This is a macro set that will calibrate and apply a GCODE Z offset to compensate for thermal expansion of the hotend. This is the thermal expansion that occurs as the system heats from standard probe temperatures (~150-180C) to printing temperatures (~220-350C). The macro calibrates an expansion coefficent and then at print time a specific z-offset is calculated using the desired print temperature.

# Thermal Expansion Calibration Macro Description
1. Conditionally homes all axis and perform a QGL/Tilt. 
2. 10 poke operations to settle down the printer mechanics. 
3. Heat to 150C, wait for the temperature to stabilize for 60s, and perform a probe operation.
4. Heat to 250C, wait for the temperature to stabilize for 60s, and perform a probe operation.
5. Repeat steps 3 and 4 again to ensure the results are repeatable.
6. The hotend is moved back to a central position and the nozzle expansion data saved to the variables file. 

Depending on how fast the hotend heats and cools down this could take 5-10min to complete both probe cycles.

A seperate macro called in print start will use the nozzle expansion coefficent and extruder print temperature to calculate and apply the appropriate z_offset automatically at print time.

# Pre-Reqs
- Beacon sensor updated to latest firmware version and contact features enabled and validated. See [https://docs.beacon3d.com/contact/]
- [save_variables] is configured to allow macros to save variables over power cycles.
```
[save_variables]
filename: ~/printer_data/config/variables.txt
```
- [respond] is present in the printer config to enable status messages.
- Ooze mitigation if 250C nozzle temp will lead to filament ooze. Best practice would be to perform this without filament to ensure no ooze is present.
- Max nozzle contact temperature set to >250C. The default maximum probe temperature is set to 180C and will cause an error during 250C probing.
```
[beacon]
BEACON_CALIBRATE_NOZZLE_TEMP_OFFSET: 275
```

> [!CAUTION]
> Probing at higher temperatures, 250C, can cause damage to some build plates. Ensure that your setup can handle 250C contact probing without damage. Refer to the beacon contant documentation for additional details. 

# Setup Steps
1. Copy macro block into your config as appropriate for your setup: [macro_thermal_expansion_compensation.cfg](macro_thermal_expansion_compensation.cfg)
2. Review/adjust the default variables in BEACON section (defaults reccomended)
3. Add lines to PRINT_START and PRINT_END macros to enable compensation
4. Restart klipper
5. Pre-heat chamber for best accuracy
6. Run BEACON_CALIBRATE_NOZZLE_TEMP_OFFSET to perform the thermal expansion calibration.
7. Review the variables file to ensure there is a nozzle_expansion_coefficient entry.
  ```
  [Variables]
  nozzle_expansion_coefficient = 0.04749999999994525
  ```
8. Run a print and monitor the first layer.

# Fine Tuning

If additional tuning is required for the first layer squish the following two methods are avaliable, in order of preference:

## Adjust Expansion Multiplier
The macro has a variable beacon_contact_expansion_multiplier that can be adjusted to fine tune squish.  A value of 1.1 produces 10% less first layer squish and 0.9 produces 10% more first layer squish. This multiplier has emperically been found to be the best method to fine tune expansion compensation for print quality as it adjusts along with different print temperatures.
```
variable_beacon_contact_expansion_multiplier: 1.0 
```

## Gcode Offset
A gcode offset could be added to the PRINT_START macro directly or passed via the slicer. Note that this type of offset is not temperature dependent like the expansion multiplier and may not work well for a variety of print temperatures.
```
SET_GCODE_OFFSET Z_ADJUST={OFFSET}
```

# Print Start
Apply this code to the print start macro. It calculates and applies a GCODE offset Z_adjust to the print. Note that this replaces the "SET_GCODE_OFFSET" gcode lines as reccomended in the Beacon Contact Docs.

> [!Note]
> Nozzle expansion calibration should be stable and only run if a mechanical change is made. BEACON_CALIBRATE_NOZZLE_TEMP_OFFSET should not be called in the PRINT_START macro.

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
Some printers may require specific settings to ensure that the probing operations are completed with best accuracy. Specifically the probe speed up/down and the probe retract distance. If these parameters are not specified in [beacon] then the probe operation uses the defaults.

The following list gives the equivalent probe parameter and beacon config names.

```
[PROBE PARAMETER NAME] -> [BEACON CONFIG NAME]
PROBE_SPEED            -> autocal_speed
LIFT_SPEED             -> autocal_retract_speed
SAMPLE_RETRACT_DIST    -> autocal_retract_dist
```

# Credits

Credit to RatOS team {Helge Keck and Mikkel Schmidt} for developing this macro. You can see their work in Beacon.cfg: https://github.com/Rat-OS/RatOS-configuration/blob/v2.1.x/z-probe/beacon.cfg
Also thanks to Burgo for the porting this macro for generic Klipper machine use and to Michele Fattoruso and Rolf Schäuble for PR's on the previous repo.


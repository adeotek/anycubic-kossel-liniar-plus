# GCode Reference

[https://marlinfw.org/docs/gcode/G000-G001.html]

- Standard Codes
G0/G1 - Coordinated Movement X Y Z E
G28 - Home all axis or named axis.
G33 - Measure distortion map
G33 R0 - delete distortion map
G33 L0 - List distortion map
G92 - Set current position to coordinates given

- RepRap M Codes
M104 Sxx - Set extruder target temp
M105 - Read current temp
M112 - Emergency stop.
M114 S1 - Display current position, S1 = also write position in steps
M140 Sxx - Set bed target temp


- Repetier Custom M Codes
M205 - Output EEPROM settings
M206 - Set EEPROM value



---More nifty pause and resume
[https://docs.octoprint.org/en/master/features/gcode_scripts.html#bundled-scripts]
If you do not have a multi-extruder setup, aren’t printing from SD and have “Log position on pause” enabled under Settings > Serial Connection > Behaviour > Pausing, the following afterPrintPaused and beforePrintResumed scripts might be interesting for you. With something like them in place, OctoPrint will move your print head out of the way to a safe rest position (here G1 X0 Y0, you might want to adjust that) on pause and move it back to the persisted pause position on resume, making sure to also reset the extruder and feedrate.

Listing 6 afterPrintPaused script
{% if pause_position.x is not none %}
; relative XYZE
G91
M83

; retract filament, move Z slightly upwards
G1 Z+5 E-5 F4500

; absolute XYZE
M82
G90

; move to a safe rest position, adjust as necessary
G1 X0 Y0
{% endif %}
Listing 7 beforePrintResumed script
{% if pause_position.x is not none %}
; relative extruder
M83

; prime nozzle
G1 E-5 F4500
G1 E5 F4500
G1 E5 F4500

; absolute E
M82

; absolute XYZ
G90

; reset E
G92 E{{ pause_position.e }}

; move back to pause position XYZ
G1 X{{ pause_position.x }} Y{{ pause_position.y }} Z{{ pause_position.z }} F4500

; reset to feed rate before pause if available
{% if pause_position.f is not none %}G1 F{{ pause_position.f }}{% endif %}
{% endif %}
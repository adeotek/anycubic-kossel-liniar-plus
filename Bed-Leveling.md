# Bed Leveling

## Using LCD Screen
1.	Install Z Probe & pre-heat bed to 50 degree C
2.	Select Configuration / Advanced Settings / Initialize EEPROM
3.	Select Configuration / Store Settings
4.	Select Configuration / Delta Calibration / Auto Calibrate
I try to shoot for an SD of .03 or below. If you don't get it on the first try perform steps 2 - 4 again to see if it gets lower.
5.	Select Configuration / Store Settings
6.	Select Motion / Level Bed
7.	Select Configuration / Store Settings
8.	Remove Z Probe
9.	Select Motion / Move Axis / Move Z
-	Move 10mm
-	Move down slowly until less than 10 mm from bed
-	Place sheet of paper on bed
-	Move 1mm
-	Move down till less than 1
-	Move 0.1mm
-	Move down slowly to zero.
-	Paper should still be movable but snagged by nozzle.
10.	If paper is too tight or too loose then do the following:
-	Select Configuration / Probe Z Offset
-	If too loose turn dial to left (counter clockwise).
-	If too tight turn dial to right (clockwise)
11.	Select Configuration / Store Settings
12.	Select Motion / Auto Home
13.	Perform step 9 - 10 to verify.
If any changes save with Configuration / Store Settings

---

## Using IDE Terminal
1.	Install Z Probe
2.	M502
3.	M500
4.	G33
I try to shoot for an SD of .03 or below. If you don't get it on the first try perform steps 2 - 4 again to see if it gets lower.
5.	M500
6.	G29
7.	M500
8.	Remove Z Probe
9.	G28
10.	Place sheet of paper on bed
11.	G1 Z20
12.	Issue G1 commands to lower nozzle to zero while checking paper movement.
-	As an example
-	G1 Z1
-	G1 Z.5
-	G1 Z.3
-	G1 Z.1
-	G1 Z0
-	Paper should still be movable but snagged by nozzle.
13.	If paper is too tight or too loose then do the following:
On the LCD Screen
-	Select Configuration / Probe Z Offset
-	If too loose turn dial to left (counter clockwise).
-	If too tight turn dial to right (clockwise)
14.	M500
15.	G28
16.	Perform 11 - 13 to check. If any changes save with an M500.

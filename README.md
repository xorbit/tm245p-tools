# tm245p-tools
Here I will collection tools I found and created for the Neoden TM245P pick and place

## TM245P_mount.ulp

When you run this ULP from Eagle CAD, it will ask for a filename for placements on top
and a separate filename for placements on the bottom. (I have not yet verified if
coordinates for placements on the bottom are correct.)

The generated files look something like this:
```
#/path/to/file/Test.brd TOP
#Stack,X1,Y1,X2,Y2,Pick Depth,Place Depth,Pick Delay,Place Delay,SizeX,SizeY,Rate,Speed,Torque,Vacuum 1,Vacuum 2,Vacuum Off,Calibration,Skip,,,,
#Stack_S,X1,Y1,X2,Y2,Pick Depth,Place Depth,Pick Delay,Place Delay,SizeX,SizeY,Rate,Speed,Torque,Vacuum 1,Vacuum 2,Vacuum Off,Calibration,Skip,Chip X,Chip Y,Loop X,Loop Y
#Circuit,OffsetX,OffsetY,Skip,,,,,,,,,,,,,,,,,,,
10201,0,0,0,,
#Component,Nozzle,Stack,X,Y,Angle,Skip,,,,,,,,,,,,,,,,
1,1,,25910,9220,-180,0,C1,0.1uF-C0603
2,1,,8380,1440,90,0,R1,1K-R0603
3,1,,6500,4300,-180,0,R2,1K-R0603
```

This defines a simple board with 2 resistors and 1 capacitor.

### Adding stack definitions

The comment line starting with `#Stack` describes how to define a source of components, such as a reel.
The comment line starting with `#Stack_S` shows an alternate format that deals with components in trays.
You can add some stack definitions such as:
```
#Stack,X1,Y1,X2,Y2,Pick Depth,Place Depth,Pick Delay,Place Delay,SizeX,SizeY,Rate,Speed,Torque,Vacuum 1,Vacuum 2,Vacuum Off,Calibration,Skip,,,,
10026,180,80,230,80,0,0,0,0,0,0,4000,90,80,-60,-60,0,0,0,,,,
10027,180,120,230,120,0,0,0,0,0,0,4000,90,80,-60,-60,0,0,0,,,,
```

The first number is 10000 (stack definition) + the stack number you're defining.  The X1/Y1 and X2/Y2
coordinates allow you to fine-tune the pick positions for head 1 and head 2 respectively.  Rate is the
feed rate for the reel in microns (4 mm for 0603 reel).

### Adding stack references

Once stack definitions are added, you can now reference them from component placements.  If for instance
we loaded the capacitor reel in stack 26 and the resistor reel in stack 27, we could change the placement
statements to:
```
#Component,Nozzle,Stack,X,Y,Angle,Skip,,,,,,,,,,,,,,,,
1,1,26,25910,9220,-180,0,C1,0.1uF-C0603
2,1,27,8380,1440,90,0,R1,1K-R0603
3,1,27,6500,4300,-180,0,R2,1K-R0603
```

Since there are two identical resistors though, we can optimize this to use both heads of the TM245P to
load both resistors at once and save travel time.  We could change the statements to:
```
#Component,Nozzle,Stack,X,Y,Angle,Skip,,,,,,,,,,,,,,,,
1,1,26,25910,9220,-180,0,C1,0.1uF-C0603
2,1,27,8380,1440,90,0,R1,1K-R0603
3,2,27,6500,4300,-180,0,R2,1K-R0603
```

R2 is now loaded from head 2.

### Adjusting the origin

The ULP positions the board at (0, 0) by default (the middle of the workspace).  If you want to use a
different position such as the fixed mount, you need to find out the offset and enter it in the CSV.

To do this, go into "Config" on the TM245P once the board is loaded, select a component to move the
laser crosshairs to the component's position and then keep adjusting the X and Y offset values until
the crosshairs are centered on the selected component.  Take note of the X and Y offsets and adjust
the `#Circuit` line in the CSV accordingly.
```
#Circuit,OffsetX,OffsetY,Skip,,,,,,,,,,,,,,,,,,,
10201,-170940,78300,0,
```

### Multiple boards on a panel

You can add multiple `#Circuit` lines if you want to stuff a panel of identical boards.  You will
need to add lines starting with 10200 + board number, one for each board on the panel.  For
instance, if we had a 2x2 panel, spaced 50 mm in X and 30 mm in Y direction, we could write:
```
#Circuit,OffsetX,OffsetY,Skip,,,,,,,,,,,,,,,,,,,
10201,-170940,78300,0,
10202,-120940,78300,0,
10203,-170940,108300,0,
10204,-120940,108300,0,
```

### Final CSV placement file

The final CSV file to be put on the SD card could look like this:
```
#/path/to/file/Test.brd TOP
#Stack,X1,Y1,X2,Y2,Pick Depth,Place Depth,Pick Delay,Place Delay,SizeX,SizeY,Rate,Speed,Torque,Vacuum 1,Vacuum 2,Vacuum Off,Calibration,Skip,,,,
10026,180,80,230,80,0,0,0,0,0,0,4000,90,80,-60,-60,0,0,0,,,,
10027,180,120,230,120,0,0,0,0,0,0,4000,90,80,-60,-60,0,0,0,,,,
#Circuit,OffsetX,OffsetY,Skip,,,,,,,,,,,,,,,,,,,
10201,-170940,78300,0,
10202,-120940,78300,0,
10203,-170940,108300,0,
10204,-120940,108300,0,
#Component,Nozzle,Stack,X,Y,Angle,Skip,,,,,,,,,,,,,,,,
1,1,26,25910,9220,-180,0,C1,0.1uF-C0603
2,1,27,8380,1440,90,0,R1,1K-R0603
3,2,27,6500,4300,-180,0,R2,1K-R0603
```

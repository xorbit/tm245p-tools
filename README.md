# tm245p-tools
Here I collect some tools I created for the Neoden TM245P Pick-and-Place machine

## TM245P_mount.ulp

When you run this ULP from Eagle CAD, it will ask for a filename for placements on top
and a separate filename for placements on the bottom. (I have not yet verified if
coordinates for placements on the bottom are correct.)

The generated files look something like this:
```
#/path/to/file/Test.brd TOP
#Stack,X1,Y1,X2,Y2,Pick Depth,Place Depth,Pick Delay,Place Delay,SizeX,SizeY,Rate,Speed,Torque,Vacuum 1,Vacuum 2,Vacuum Off,Calibration,Skip,,,,
#Stack_S,X1,Y1,X2,Y2,Pick Depth,Place Depth,Pick Delay,Place Delay,SizeX,SizeY,Rate,Speed,Torque,Vacuum 1,Vacuum 2,Vacuum Off,Calibration,Skip,Chip X,Chip Y,Loop X,Loop Y
#Board,OffsetX,OffsetY,Skip
10201,0,0,0
#Part,Nozzle,Stack,X,Y,Angle,Skip,Name,Value
1,1,,25910,9220,180,0,C1,0.1uF-C0603
2,1,,8380,1440,90,0,R1,1K 1%-R0603
3,1,,6500,4300,180,0,R2,1K-R0603
```

This defines a simple board with 2 resistors and 1 capacitor.  For the sake of the example, let's
assume R1 needs to be a 1% resistor, R2 does not, but we're loading both parts from the same 1K
1% reel.  Let's also assume that our resistor was defined in Eagle part definition at the same angle
as it gets picked from the reel, but the capacitor needs to be turned 90 degrees after it is picked
to match the part definition in Eagle.

### Adding stack definitions

The comment line starting with `#Stack` describes how to define a source of parts, such as a reel.
The comment line starting with `#Stack_S` shows an alternate format that deals with parts in trays.
You can add some stack definitions such as:
```
#Stack,X1,Y1,X2,Y2,Pick Depth,Place Depth,Pick Delay,Place Delay,SizeX,SizeY,Rate,Speed,Torque,Vacuum 1,Vacuum 2,Vacuum Off,Calibration,Skip,,,,
10026,180,80,230,80,0,0,0,0,0,0,4000,90,30,-60,-60,0,0,0,,,,
10027,180,120,230,120,0,0,0,0,0,0,4000,90,30,-60,-60,0,0,0,,,,
```

The first number is 10000 (stack definition) + the stack number you're defining.  The X1/Y1 and X2/Y2
coordinates allow you to fine-tune the pick positions (in microns) for nozzle 1 and nozzle 2 respectively.
Rate is the feed rate for the reel in microns (4 mm for 0603 reel).  Speed limits the speed of the machine
when dealing with parts from this reel.  Torque defines how much the cover tape will be pulled while the
needle feeds the part.  If this value is too low the cover tape may not get pulled off correctly.  If
this value is too high, it may cause the needle to bind while feeding the part.  Vacuum 1 and 2 define
the vacuum level each nozzle must reach to detect a part was picked up correctly.  Vacuum Off can be set
to 1 to disable this check.  Setting Calibration to 1 will cause the machine to center the part on the
nozzle using the calibration box before it is placed.  If this feature is used, the part size in microns
must be defined using SizeX and SizeY.  When Skip is set to 1, parts that reference this reel are skipped.

### Adding stack references

Once stack definitions are added, you can now reference them from part placements.  If for instance
we loaded the capacitor reel in stack 26 and the resistor reel in stack 27, we could change the placement
statements to:
```
#Part,Nozzle,Stack,X,Y,Angle,Skip,Name,Value
1,1,26,25910,9220,180,0,C1,0.1uF-C0603
2,1,27,8380,1440,90,0,R1,1K 1%-R0603
3,1,27,6500,4300,180,0,R2,1K-R0603
```

Since there are two identical resistors though, we can optimize this to use both nozzles of the TM245P to
load both resistors at once and save travel time.  We could change the statements to:
```
#Part,Nozzle,Stack,X,Y,Angle,Skip,Name,Value
1,1,26,25910,9220,180,0,C1,0.1uF-C0603
2,1,27,8380,1440,90,0,R1,1K 1%-R0603
3,2,27,6500,4300,180,0,R2,1K-R0603
```

R2 is now loaded using nozzle 2.

Since the capacitor package was defined differently from the way it is actually picked up from the reel,
we need to adjust the angle:
```
#Part,Nozzle,Stack,X,Y,Angle,Skip,Name,Value
1,1,26,25910,9220,-90,0,C1,0.1uF-C0603
2,1,27,8380,1440,90,0,R1,1K 1%-R0603
3,2,27,6500,4300,180,0,R2,1K-R0603
```

### Adjusting the origin

The ULP positions the board at (0, 0) by default (the middle of the workspace).  If you want to use a
different location such as the fixed mount in the top left corner, you need to change the board offset
to match your actual board location.

To do this, "Edit" the file on the TM245P, select the board definition line, again "Edit", check
"Location lock" and move the head using the touch screen buttons until the laser crosshairs are
centered on the first part defined in the file (C1 in our example).  Hit "OK" to save this location.
```
#Board,OffsetX,OffsetY,Skip
10201,-170940,78300,0
```

### Multiple boards on a panel

You can add multiple `#Board` lines if you want to stuff a panel of identical boards.  You will
need to add lines starting with 10200 + board number, one for each board on the panel.  For
instance, if we had a 2x2 panel, spaced 50 mm in X and 30 mm in Y direction, we could write:
```
#Board,OffsetX,OffsetY,Skip
10201,-170940,78300,0
10202,-120940,78300,0
10203,-170940,48300,0
10204,-120940,48300,0
```

### Final CSV placement file

The final CSV file to be put on the SD card could look like this:
```
#/path/to/file/Test.brd TOP
#Stack,X1,Y1,X2,Y2,Pick Depth,Place Depth,Pick Delay,Place Delay,SizeX,SizeY,Rate,Speed,Torque,Vacuum 1,Vacuum 2,Vacuum Off,Calibration,Skip,,,,
10026,180,80,230,80,0,0,0,0,0,0,4000,90,30,-60,-60,0,0,0,,,,
10027,180,120,230,120,0,0,0,0,0,0,4000,90,30,-60,-60,0,0,0,,,,
#Board,OffsetX,OffsetY,Skip
10201,-170940,78300,0
10202,-120940,78300,0
10203,-170940,48300,0
10204,-120940,48300,0
#Part,Nozzle,Stack,X,Y,Angle,Skip,Name,Value
1,1,26,25910,9220,-90,0,C1,0.1uF-C0603
2,1,27,8380,1440,90,0,R1,1K 1%-R0603
3,2,27,6500,4300,180,0,R2,1K-R0603
```

## tm245p-merge

`tm245p-merge` is a simple Python script I wrote to automate a lot of the manual work described above.
(NOTE: It's still under development and missing some crucial error checking.  It you feed it weird stuff,
it will crash.)

To enable the script to take care of a lot of grunt work, I extended the format of the stack definitions
in the CSV file with extra fields to make it possible to:
* Automatically assign stack references to part definitions
* Specify an angle in the stack definition if a part was defined at a different angle in the CAD program
  than it is picked from the reel
* Automatically assign the nozzle to use for the part, or leave it up to the script to optimize the use
  of both nozzles

To extended stack definitions for the above example could look like this:
```
#Stack,X1,Y1,X2,Y2,Pick Depth,Place Depth,Pick Delay,Place Delay,SizeX,SizeY,Rate,Speed,Torque,Vacuum 1,Vacuum 2,Vacuum Off,Calibration,Skip,Chip X,Chip Y,Loop X,Loop Y,Angle,Nozzle,Value Matches
10026,180,80,230,80,0,0,0,0,0,0,4000,90,30,-60,-60,0,0,0,,,,,90,1,0.1uF-C0603
10027,180,120,230,120,0,0,0,0,0,0,4000,90,30,-60,-60,0,0,0,,,,,0,,1K 1%-R0603,1K-R0603
```

New fields added are Angle, Nozzle and a list of Value Matches.
The example above defines that stack 26 holds the 0.1uF capacitor, we want to use nozzle 1 to pick
and place it, and it needs to be turned 90 degrees for the picked part to match the Eagle part
definition.
Stack 27 holds the 1K resistors.  Since the values output from Eagle are different for each resistor
(1K 1%-R0603 for R1 and 1K-R0603 for R2), we specify that both part values will be
loaded from stack 27.  We also specify no rotation and leave it up to the script which nozzle to use,
so the script can interleave them for improved pick and place efficiency with less travel.

Given a file called "stack.txt" which contains stack and board definitions, and a file "parts.txt" that
holds the parts as generated using the TM245P_mount.ulp script, we can run:
```
./tm245p-merge stack.txt parts.txt out.txt
```

The following shows the out.txt file:
```
# out.txt generated by tm245p-merge
# Stacks/boards from stack.txt and parts from parts.txt)
#Stack,X1,Y1,X2,Y2,Pick Depth,Place Depth,Pick Delay,Place Delay,SizeX,SizeY,Rate,Speed,Torque,Vacuum 1,Vacuum 2,Vacuum Off,Calibration,Skip,Chip X,Chip Y,Loop X,Loop Y,Angle,Nozzle,Value Matches
10026,180,80,230,80,0,0,0,0,0,0,4000,90,30,-60,-60,0,0,0,,,,,90,1,0.1uF-C0603,
10027,180,120,230,120,0,0,0,0,0,0,4000,90,30,-60,-60,0,0,0,,,,,0,,1K 1%-R0603,1K-R0603,
#Board,OffsetX,OffsetY,Skip
10201,-170940,78300,0
10202,-120940,78300,0
10203,-170940,48300,0
10204,-120940,48300,0
#Part,Nozzle,Stack,X,Y,Angle,Skip,Name,Value
1,1,26,25910,9220,-90,0,C1,0.1uF-C0603
2,2,27,8380,1440,90,0,R1,1K 1%-R0603
3,1,27,6500,4300,180,0,R2,1K-R0603
```

As can be seen, the script automatically inserted stack refences based on the matched values for each part.
It also turned the C1 capacitor by 90 degrees.  Plus it made use of both nozzle 1 and 2 to optimize pick
and place speed.

Something not obvious in this example is that the script also throws out parts for which no stack definition
matches (it warns the user) and stack definitions not used by any part, plus it reorders the parts by value
and name, making it so that the interleaving of nozzles has the most chance of improving efficiency by doing
all the parts from the same reel in a row.

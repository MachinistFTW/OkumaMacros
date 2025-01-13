# OkumaMacros
 This repository is a collection of CNC machine code macros that can be used with most Okuma machines. Some macros may have the need for additional options or be specific to some machines. 

 ## G116-NEW.LIB
 **Works with all Okuma Mill machine of U100, E100, & P100 - P500 type**
 The G116 macro is used in place of M6 for tool changes and performs a few different functions. First, it allows T0 to be called which will empty the tool pot. This cannot be done with M6 as it will designate a no tool found alarm and would instead require the use of M64 followed by M6. Second, it allows the next to be different than the tool currently staged. For example, if you run a program that stages tool 2 and, instead of calling tool 2, you reset the program to rerun tool one, the machine will alarm out at the 'T1 M6' line since tool 2 is currently staged for the tool change and will need to be returned before staging a different tool. 
 ### Changes from the old 'G116' to the new 'G116'
 The macro now uses VTLCN(Active tool number) instead of VATOL(Actual Tool Number) and VTLNN (Next tool number) instead of VNTOL(Next tool number). This is because both VATOL and VNTOL use a bit matrix of two-byte data type which includes tool number and, tool weight, and tool kind, limiting the max tool number to 999. With the old G116 if a user describe a tool as T1000 or greater, a B level alarm will occur. VTLCN and VTLNN describe the tool number as a whole number whose limit is 16-bit.
 ### Syntax
 'G116 T5' 
 Note that the T number is after the G116 command. In addition, the G116 will need to be set in the parameter G and M CODE MACRO as OTC.

 ## OPHAN.SSB  
 **Works with Okuma Lathe and Mill machines**
 The OPHAN is an example of checking the value of an Input / Output signal for a machine and generating an alarm. In this program, the customer had a concern regarding a crash that was caused by the Pulse Handle Shift function being active and the user being unaware. On a newer machine, this function has it's own variable and an interlocking function. However, on older machines, neither one of these function were present. As such, I created a macro that would look at the current state of the light for the pulse handle shift function on the control panel. Each machine may be different, so the value 'VORD[0024]' may need to be adjusted accordingly. Since the output for opPHAL was 514 bit 4, I the value of VORD will need to be 0024. where 002 is the 514 offset -512 and the value of 4 is hexadecimal form of the bit. In the case that the bit value was 12, one would use 002C instead as C is the hexadecimal value of 12. 
 In addition, the alarm generated specifies Alarm B 'P. HANDLE ON' and will stop the operator from running if the value of opPHAL is on.
### Usage Syntax
Since this is a SSB program, the syntax will be 'CALL OPHAN'
## G205-MULTUS.LIB
**Designed for Multus-B, Multus-U, and VTM-YB Machines**
This is a customer macro that was created to clear the machine and prepare it for run. It is intended to be ran in MDI and will perform the following functions.
- Turn off all coolants that might be active 'M9 M144 M174 M262'
- Send machine to positive X limit
- Send machine to home position 4
- Turn off Probe
- Air Blow spindle for 4 seconds
- Index to T1000 position (tool pointed towards spindle)

## OPULL.SSB
**Designed for turret lathe with turret mounted bar puller**
OPULL is a simplified bar pull program that uses the work counter to stop the program and block skip for single cycles.

### Syntax
'CALL OPULL PL=.75'
This code example would pull the bar a total of .75" assuming that the Work counter has not exceeded the total count aloud.

## OPOP.SSB
**Designed for turret lathe with turret mounted bar puller**
OPOP is a bar pull that works similarly to OPULL but draws the part out for a second cut. PL is the incremental value of the pull and PR is the Z pull position. Turret position may need to be modified for correct bar puller.
### Syntax
'CALL OPOP PR=-.75 PL=1.0'
This will cause the bar puller to position it's self at -0.75" then connect, then pull 1.0" then disconnect. Part counter is not modified.

## SET-MASTER-Z.MIN
**Designed for Okuma Mill machines with mechanical touch setter (example Renishaw RTS, OTS, or Okuma Contact Tool Setter)**
This program sets the Z offset for the tool setter using a gauge bar. PLI value will need to be modified according to gauge bar with known tool length actively in the spindle.

## SET-MASTER-XY.MIN
**Designed for Okuma Mill machines with mechanical touch setter (example Renishaw RTS, OTS, or Okuma Contact Tool Setter)**
This program saves the current location of X and Y for the touch setter.

## OROTA.SSB
**Designed for Okuma Mill machines with a 4th axis set as the A axis.**
I haven't done a lot of testing with this one. I have several changes I hope to make, like a center line shift. Currently the Z and Y axis must be the centerline of the A axis. 
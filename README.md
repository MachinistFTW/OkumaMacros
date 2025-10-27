# OkumaMacros
 This repository is a collection of CNC machine code macros that can be used with most Okuma machines. Some macros may have the need for additional options or be specific to some machines. 
 ## Contents
- [Warning](#warning)
- [Installation](#installation)
- [List of Macros](#User-Task-List)
## Warning
 ⚠ **Warning:** When using custom macros (user tasks) on Okuma machines, ensure that you fully understand their functionality. These macros can execute machine motions and modify values, potentially leading to unexpected results or **significant damage** to machine components. If you are unsure, do not use them. Please ask questions on Stack Overflow using the `[okuma]` tag.
## Installation
The usage files come in two types: **LIB** and **SSB** files. 
> An SSB file is designed to be called from within a program. It cannot be called with a G or M code but only with the word `CALL xxxx`.  
> A LIB file can be called with a G or M code on machine that have the G/M Code Macro Option but needs to be registered in the LIBRARY file stack.  
> The biggest advantage of SSB over LIB is that an LIB file takes up available program size from your regular MIN file, whereas an SSB does not.  
> If you want to use an M or G code to call the routine, you must use a LIB. Otherwise, SSB is the better choice.

**Using LIB Method**
For using .LIB files, please reference this youtube video from Gosiger West Application [LIB file registration on an Okuma P200 or P300 control](https://www.youtube.com/watch?v=UmauXHBuxa8)
**Using SSB Method**
For using .SSB files, add these files to your MD1 folder on the Okuma. Add the required call statements to your current program such as `CALL OXXXX` where `OXXXX` is the name of the program such as OPULL.
 ## User Task List
- [G116 Tool Change](#g116-newlib)
- [VORD/VIRD Check](#ophan)
- [Multus Reset](#g205-multus)
- [Lathe Bar Pull W/ Counter](#opull)
- [Lathe Bar Pull for 2 OP](#opop)
- [Z Axis Touch Setter Calibration](#set-master-z)
- [XY Location for Touch Setter](#set-master-xy)
- [Center of Rotation Calculation Mill A axis](#orota)
## G116-NEW
 **Works with all Okuma Mill machine of P100 through P500 type**
 The G116 macro is used in place of M6 for tool changes and performs a few different functions. First, it allows T0 to be called which will empty the tool pot. This cannot be done with M6 as it will designate a no tool found alarm and would instead require the use of M64 followed by M6. Second, it allows the next to be different than the tool currently staged. For example, if you run a program that stages tool 2 and, instead of calling tool 2, you reset the program to rerun tool one, the machine will alarm out at the 'T1 M6' line since tool 2 is currently staged for the tool change and will need to be returned before staging a different tool. 
 ### Changes from the old 'G116' to the new 'G116'
 The macro now uses VTLCN(Active tool number) instead of VATOL(Actual Tool Number) and VTLNN (Next tool number) instead of VNTOL(Next tool number). This is because both VATOL and VNTOL use a bit matrix of two-byte data type which includes tool number and, tool weight, and tool kind, limiting the max tool number to 999. With the old G116 if a user describe a tool as T1000 or greater, a B level alarm will occur. VTLCN and VTLNN describe the tool number as a whole number whose limit is 16-bit.
 ### Function
 `G116 T5` 
 Note that the T number is after the G116 command. In addition, the G116 will need to be set in the parameter G and M CODE MACRO as OTC.
## OPHAN  
 **Works with Okuma Lathe and Mill machines**
 The OPHAN is an example of checking the value of an Input / Output signal for a machine and generating an alarm. In this program, the customer had a concern regarding a crash that was caused by the Pulse Handle Shift function being active and the user being unaware. On a newer machine, this function has it's own variable and an interlocking function. However, on older machines, neither one of these function were present. As such, I created a macro that would look at the current state of the light for the pulse handle shift function on the control panel. Each machine may be different, so the value VORD[0024] may need to be adjusted accordingly. Since the output for opPHAL was 514 bit 4, I the value of VORD will need to be 0024. where 002 is the 514 offset -512 and the value of 4 is hexadecimal form of the bit. In the case that the bit value was 12, one would use 002C instead as C is the hexadecimal value of 12. 
 In addition, the alarm generated specifies Alarm B 'P. HANDLE ON' and will stop the operator from running if the value of opPHAL is on.
### Usage Function
Since this is a SSB program, the syntax will be 'CALL OPHAN'
## HOME-MULTUS
**Designed for Multus-B, Multus-U, and VTM-YB Machines**
This is a macro that was created to clear the machine and prepare it for run. It is intended to be ran in MDI and will perform the following functions.
- Turn off all coolants that might be active 'M9 M144 M174 M262'
- Send machine to positive X limit
- Send machine to home position 4
- Turn off Probe
- Air Blow spindle for 4 seconds
- Index to T1000 position (tool pointed towards spindle)
## OPULL
**Designed for turret lathe with turret mounted bar puller**
OPULL is a simplified bar pull program that uses the work counter to stop the program and block skip for single cycles.

### Function
'CALL OPULL PL=.75'
This code example would pull the bar a total of .75" assuming that the Work counter has not exceeded the total count aloud.
## OPOP
**Designed for turret lathe with turret mounted bar puller**
OPOP is a bar pull that works similarly to OPULL but draws the part out for a second cut. PL is the incremental value of the pull and PR is the Z pull position. Turret position may need to be modified for correct bar puller.
### Function
'CALL OPOP PR=-.75 PL=1.0'
This will cause the bar puller to position it's self at -0.75" then connect, then pull 1.0" then disconnect. Part counter is not modified.
## SET-MASTER-Z
**Designed for Okuma Mill machines with mechanical touch setter (example Renishaw RTS, OTS, or Okuma Contact Tool Setter)**
This program sets the Z offset for the tool setter using a gauge bar. PLI value will need to be modified according to gauge bar with known tool length actively in the spindle. It's important to note that this does not affect Renishaw gauging cycles.
## SET-MASTER-XY
**Designed for Okuma Mill machines with mechanical touch setter (example Renishaw RTS, OTS, or Okuma Contact Tool Setter)**
This program saves the current location of X and Y for the touch setter. It's important to note that this does not affect Renishaw gauging cycles.
## OROTA
**This macro is designed for Okuma Mill machines with a 4th axis configured as the A axis.**
'CALL OROTA PX=0 PY=0 PZ=0 PA=0 PSA=0'
-PX, PY, PZ: These parameters serve as a parallel shift function. They are generally not required unless the relative program zero changes.
    -Example: If at A0, the program zero is at the top-left corner with Z zero on the face, and at A90, the zero is again at the top-left corner but with Z zero on the newly visible face, then the relative zero point has shifted.
    -In such cases, the operator must either set additional work offsets or use the parallel shift function to adjust for this change.
-PA= (Absolute A-Axis Rotation Solution): This value represents the absolute A-axis rotation. In most cases, it will match the A-axis value directly.
-PSA= (Absolute A-Axis Rotation Zero Solution): This parameter is typically not needed, but it becomes useful for updating zero points located at positions other than A0.
    -Example: If a programmed part references a bore positioned at A-20 relative to the programmed zero, the operator can set PA=0 PSA=-20. This adjustment ensures the new offset accounts for the rotary axis shift.
⚠ Important Consideration:
When rotating planes using PSA, all three linear axes (X, Y, and Z) must be updated before applying the PSA value. If any of these values are incorrect, the reference plane will not calculate correctly.

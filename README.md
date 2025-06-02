OkumaMacros
===========

***A collection of CNC machine code macros specific to Okuma machines.***

 # Contents
* [Warning](#warning)
* [Installation](#installation)
- [COMMON:  VORD/VIRD Check](#ophan)
- [MILL:    G116 Tool Change](#g116-newlib)
- [MILL:    Center of Rotation Calculation Mill A axis](#orota)
- [MILL:    Z Axis Touch Setter Calibration](#set-master-z)
- [MILL:    XY Location for Touch Setter](#set-master-xy)
- [LATHE:   Bar Pull W/ Counter](#opull)
- [LATHE:   Bar Pull for 2 OP](#opop)
- [LATHE:   Multus/VTM Reset](#g205-multus) 

## Warning
 ⚠ **Warning:** When using custom macros (user tasks) on Okuma machines, ensure that you fully understand their functionality. These macros can execute machine motions and modify values, potentially leading to unexpected results or **significant damage** to machine components. If you are unsure, do not use them. Please ask questions on Stack Overflow using the `[okuma]` tag.
## Installation
The usage files come in two types: **LIB** and **SSB** files. 
> SSB files are designed to be called from within another program. It cannot be called with a G or M code. It can only be used with the code `CALL`.  
> LIB files can be called with a G or M code on machine that have the G/M Code Macro Option but needs to be registered in the LIBRARY file stack.  
> The biggest advantage of SSB over LIB is that an LIB file takes up available program size from your regular MIN file, whereas an SSB does not.  
> If you want to use an M or G code to call the routine or be able to use the CALL function in MDI mode, you must use a LIB. Otherwise, SSB is the better choice.
**Reporting Issues and requesting enhancements**
As users continue to use these macros, we ask that you please provide feedback and report issues by using the issues function on the Github repo. This will help us keep track of known issues, update fixes, and add functions that users come up with. If you are unfamiliar with Github, I strongly recommend creating an account and adding this repo to your watch list. This video will show you how to create issues once you have an account. [Workflow: How to create issue on GitHub] (https://www.youtube.com/watch?v=U_mLcJ0d6xQ)
**Using LIB Method**
For using .LIB files, please reference this youtube video from Gosiger West Application [LIB file registration on an Okuma P200 or P300 control](https://www.youtube.com/watch?v=UmauXHBuxa8)
**Using SSB Method**
For using .SSB files, add these files to your MD1 folder on the Okuma. Add the required call statements to your current program such as `CALL OXXXX` where `OXXXX` is the name of the program such as OPULL.

## Additional Information
There are a few useful videos available for those looking to learn more about Okuma Macros.
[Gosiger West - Okuma Macro Variables explained](https://www.youtube.com/watch?v=ZKhFpNQgRug)
[Gosiger West - Okuma horizontal work coordinate macro](https://www.youtube.com/watch?v=h1MvFW2YDIU)
[Gosiger West - TLM MACRO](https://www.youtube.com/watch?v=1BZwu-c6JQA)


# COMMON
The following macros can be used for mill or lathe.

## OPHAN  
 The OPHAN is an example of checking the value of an Input / Output signal for a machine and generating an alarm. In this program, the customer had a concern regarding a crash that was caused by the Pulse Handle Shift function being active and the user being unaware. On a newer machine, this function has it's own variable and an interlocking function. However, on older machines, neither one of these function were present. As such, I created a macro that would look at the current state of the light for the pulse handle shift function on the control panel. Each machine may be different, so the value 'VORD[0024]' may need to be adjusted accordingly. Since the output for opPHAL was 514 bit 4, I the value of VORD will need to be 0024. where 002 is the 514 offset -512 and the value of 4 is hexadecimal form of the bit. In the case that the bit value was 12, one would use 002C instead as C is the hexadeci mal value of 12. 
 In addition, the alarm generated specifies Alarm B 'P. HANDLE ON' and will stop the operator from running if the value of opPHAL is on.

# MILL
The following macros are intended for Mill type controllers.

## G116-NEW
 **Works with all Okuma Mill machine of P100 through P500 type**
 The G116 macro is used in place of M6 for tool changes and performs a few different functions. First, it allows T0 to be called which will empty the spindle. This cannot be done with M6 as it will designate a no tool found alarm and would instead require the use of M64 followed by M6 instead. Second, it allows the next to be different than the tool currently staged. For example, if you run a program that stages tool 2 and, instead of calling tool 2, you reset the program to rerun tool one, the machine will alarm out at the 'T1 M6' line since tool 2 is currently staged for the tool change and will need to be returned before staging a different tool. Lastly, if the tool you call is already in the spindle, the machine will not alarm out. If you have a T1 in the spindle and issue a T1 M6 it will alarm out with a same tool command alarm. G116 will fix this.  

 ### Changes from the old 'G116' to the new 'G116'
 The macro now uses VTLCN(Active tool number) instead of VATOL(Actual Tool Number) and VTLNN (Next tool number) instead of VNTOL(Next tool number). This is because both VATOL and VNTOL use a bit matrix of two-byte data type which includes tool number and, tool weight, and tool kind. With out additional bitwise operations this limited the max tool number to 512. With the old G116 if a user describe a tool as T1000 or greater, a B level alarm will occur. VTLCN and VTLNN describe the tool number as a whole number whose limit is 16-bit. 
 
 In addition, one of the original goals of the G116 was to allow users to tool change to a tool regardless of what tool was staged. Previously the logic checked the staged tool to see if it matched the requested tool. If it did not, it unstaged the tool and allowed the tool change to continue with the requested tool. However, during tool life management functions, this would unstage sister tools (tools of the same group number) since they did not match the tool requested. This has been fixed by checking the group number for the requested tool here:'IF [VTLD1[PT] EQ VTLD1[VTLNN]] NTC6'

 Previously, calling the same tool would not check the condition of the tool before running it. For example, if the tool was in the NG state because of tool life expired, it would continue running the tool. Now, G116 will check the state of the tool and force a tool change if the tool in NG. This will call up a different tool or give an alarm if the tool in NG. 

 Okuma noted an issue related to the use of "MC USER PARAMETER ATC TOOL ECHANGE - SEQUENCE RESTART TOOL NO. CHECK IS MADE EFFECTIVE" which, in the previous versions of tool change macros, the system would pull in the wrong tool offset during restart due to a logic error. This has been addressed in this macro.

 Note on update: This G116 is a stand alone .LIB file. If your machine currently has a G116 type macro (or anything called OG116 as the program name), it will load whichever one it finds first on power up. It's important that, if you are updating the G116 file, that you remove the old G116 type program (which might be inside another .LIB file with additional programs)
 
 ### Function
 `G116 T5` 

 Notes: 
 - The T number is after the G116 command.
 - The G116 will need to be set in the parameter G and M CODE MACRO as OG116.

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

### Usage Function
Since this is a SSB program, the syntax will be 'CALL OPHAN'

## SET-MASTER-Z
**Designed for Okuma Mill machines with mechanical touch setter (example Renishaw RTS, OTS, or Okuma Contact Tool Setter)**
This program sets the Z offset for the tool setter using a gauge bar. PLI value will need to be modified according to gauge bar with known tool length actively in the spindle. It's important to note that this does not affect Renishaw gauging cycles.
## SET-MASTER-XY
**Designed for Okuma Mill machines with mechanical touch setter (example Renishaw RTS, OTS, or Okuma Contact Tool Setter)**
This program saves the current location of X and Y for the touch setter. It's important to note that this does not affect Renishaw gauging cycles.

# LATHE
The following macros are intended for lathe type controllers.
## HOME-MULTUS
**P300 TD mode controllers only**
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

---
title: Generation 2
has_children: true
nav_order: 2
---

# Panels System

Author: [Michael Reiser](https://github.com/mbreiser)

This is a rather new system and still has a very small user community. As we get more users, it is very important to make use of peoples' experiences, suggestions, and improvements. I am especially eager to have people offer to improve the documentation. Please send any updates, improvements, troubleshooting suggestions, etc. to the mailing list (after joining it, of course). This page documents the technical details of the system. A [user guide](https://github.com/reiserlab/Panel-G3-Software/blob/master/assets/flight_simulator_user_guide3.doc), written by Mark Frye, Michael Dickinson, and Michael Reiser, introduces new users to this system (and its use as a flight arena for flies).


# System Overview

The system has been designed to allow for rapid development of behavioral (visual) stimuli. There are three components to the system:

- **The panels** - the actual display. These are 8x8 dot matrix displays of LEDs with some electronics to drive the display.
- **The controller** - a custom designed microprocessor circuit that communicates with a PC via the serial port and to the panels using a rapid serial interface (TWI). Note that TWI is Atmel's name for their own implementation of the I2C bus, a standard developed by Phillips.
- **PC software** - written in Matlab, there are tools for generating patterns and sending commands to the controller.

In all the documentation that follows, we will try to use this terminology consistently:

- **The system** - refers to all of the components: controllers, panels, etc.
- **Setup** - refers to your particular configuration. This is mostly used in the context of the particular way in which you have the panels connected to the controller.
- **The Controller** - refers to the controller circuit board that is described in Section controller. This can be somewhat confusing, because the panels and controller boards both have microcontrollers on them, which both act as controllers. Furthermore, the Matlab program is also controlling the controller. However, *the controller* will always refer to the controller circuit. 
- **A Panel** - refers to the module that is composed of both the circuit board and the LED display.
- **PControl** - is the name of the Matlab gui. 

# Installation and Prerequisites

There are some basic tools needed to use the system:

- A PC running a recent version of Windows. The PC must have at least 1 serial port, although 2 (or even 3) would be useful. To easily add serial ports to your computer, consider a USB to Serial converter. [Here are some](http://www.usbgear.com/USB-Serial.html).
- [MATLAB](https://www.mathworks.com ) installed on this PC - the code should work on all Matlab releases, but is currently only tested on releases from 2009 onwards.
- A Programming device for the Atmel microcontrollers. Either the [AVR ISP mkII](http://www.atmel.com/dyn/products/tools_card.asp?tool_id=3808&category_id=163&family_id=607&subfamily_id=760) or the [AVR STK500](http://www.atmel.com/dyn/products/tools_card.asp?tool_id=2735). The AVR ISP [Digikey p/n ATAVRISP2-ND](https://www.digikey.com/products/en?keywords=ATAVRISP2-ND) is a stand alone programmer that connects to a PC via USB on one end, and has the 6 pin programming interface which is used to program the controller and the panels. The [AVR STK500](https://www.digikey.com/short/z9zd3h) has the programming interface, but is also a starter kit and development system for Atmel microcontrollers.
- AVR studio or other set of AVR tools - this is useful if you plan to make any modifications to the software running on either the controller or the panels. The AVR studio has a nice GUI for programming Atmel microcontrollers through either the AVR ISP or AVR STK500. More information at www.atmel.com and www.avrfreaks.net. 
- Power supplies - see section [Controller Hardware](#controller-hardware) for details.
- Compact Flash (CF) card and reader. Any USB-based CF (or multi-format) reader will work. Even large patterns only require space measured in kilobytes, so any available CF card will be large enough, but it is advisable to get a high speed card. The fastest card we measured was a 128 MB Dane-Elec 45X card. Note: currently the system seems not to work with cards that are larger than 128 MB. This is certainly a solvable problem, but buying 128 MB cards is the best option currently. These cards are increasingly harder to find, one reason that PDC v3 uses SD cards. One vendor that still carries these is http://www.flash-memory-store.com/.
- Many experiments can be made simpler by generating signals from Matlab that can be used to encode some information about the current trial (this signal is then acquired along with any other experimental data). An inexpensive and reliable product for doing this is one of the USB DAQ modules from hhttp://www.mccdaq.com/. We have been using the 1208 series that has 2 analog output signals and is supported by the Matlab DAQ toolbox. 


# Download Code & PCB files

This code is no longer officially supported or maintained, but has been used for years by at least 10 labs. The code is distributed “as is”; it works for me and I hope it works for you. **Michael Reiser and the Dickinson Lab are not liable for any damage caused by using (or misusing) this code.** Remember, programming microcontrollers is not as safe as PC programming, extreme caution must be taken to ensure that all connections are correct, and that the circuit boards are functioning – if anything smells like it is burning or gets hot – turn off immediately and recheck everything. 

- [Panel Files Release 2](https://github.com/reiserlab/Panel-G3-Software/blob/master/assets/Panels_R2.zip)

There are many changes that are included in the second release:

- New control modes, numeric position control. This uses the function generator to numerically specify the current position. Implemented as a (signed) offset to the position specified by `Panel_com(‘set_position’, [X,Y])`.
- System can now support much larger patterns – previously a frame was limited to one block of the CF memory – 512 bytes. Now multi-block frames are supported. Tested with 48 panel gscale patterns (1152 bytes per frame). 
- CF reading is slightly faster, no longer reads entire block, but stops once the number of bytes in the frame have been read. 
- Row compression: for patterns that consist of identical data for all rows of the pattern. RC is enabled by setting pattern.row_compression = 1; in  the pattern-making script. An RC pattern will just consist of one row instead of 8, resulting is a speedup factor of at least 5. This feature required modification to the code on the main and CF controllers, the panel code, and a few of the MATLAB pattern making and displaying routines. 
- Identity compression: For patterns that contain large swaths of pixels that are simply on or off. The controller can check to see if this is the case, and then just send the row-compressed version of the panel data for a panel that corresponds to a pattern piece that is all one value (works for grayscale too!). This feature is not used at pattern making time, but rather while the pattern is running. Identity compression can be enabled by invoking: Panel_com('ident_compress_on') from MATLAB. 
- New (simpler) serial port interface from Matlab. Fixes previous problem sending certain values (byte values above 127) over the serial port.
- Two new digital outputs: pulse to trigger a laser depending on pattern position, and an adjustable clock signal to trigger a camera. These are supported by commands in the panel_com function. 
- Most of the C code (controller programs and the G3 panel code) is now compatible with newest version of GCC compiler. This required including a legacy.h file. 
- Now releasing code for 3 generations of panels. The code in the previous release is now considered G2 code. G3 code is only slightly modified from G2 to support a new chip – the Atmega168 (slightly faster, bigger flash), using the same panel PCB. G1 code is included for the convenience of Dickinson Lab members who are still using the original design. All controller code is compatible with all 3 generations of panels. 
- The distribution includes PCB files for a 12 panel ring, the typical configuration of a flight arena. 

All files are being distributed in a single .zip file. This file contains 3 folders:

- c code - code for the panels and the controller board. Each subfolder also contains the .hex file needed to program the individual microcontrollers. If no modification will be made to the c code, then the compiled .hex files are sufficient to build the system. To edit and compile the AVR c code, you will need to install a GCC compiler for the Atmel AVRs. We recommend the [WinAVR](http://winavr.sourceforge.net/ ) package, but there are others out there. Some of the modules used in the AVR c code, are taken from the Procyon AVRlib (link broken) package written by Pascal Stang. The code from the AVRlib is included in the distributed .zip file above, so you will not need to download this package, but you are encouraged to do so if you plan to do any serious Atmel programming.
- schematics –schematic and PCB files for the panels and the controller. These files were created in Protel 99 SE. Each folder also contains a subfolder with all of the CAD files that are needed to order the PCBs. 
- Matlab panels code – contains a folder called `MatlabRoot` that contains a series of subfolders. To set this up on a new system, the folder should be copied to the `C:\` drive (you want the subfolders to reside under `C:\matlabroot`). It is important to add these new folders and all subfolders to the Matlab path. There is a file called `panel_control_paths.m` that should be updated with the appropriate paths for the specified files, so that the installation will function on your PC. You will also need to the file [dd.exe](http://www.chrysocome.net/dd) (*Note: the original documentation linked to http://uranus.it.swin.edu.au/~jn/linux/rawwrite/dd-0.3.zip which is broken. The file is included in the zip, but should you require a different version, the above link might help you to get started*), but this is included in the folder of other files. This file is necessary to burn images onto the CompactFlash cards (from Windows). In `MatlabRoot\Panels\IO_tools\` there is a file called `get_CF_drive.m`, this file contains a hard-coded drive letter for the CF drive - you should update this letter to reflect your system. 

# Panels

The panels are the display devices in the system. They are individually addressed and communicated with over the TWI bus. Each panel runs a small program which receives updated pattern information and refreshes the display.

## Panel Hardware

The panels are a small package that contains a circuit board with a microprocessor and line driver, and an 8x8 LED dot matrix display. A full parts list and schematic is found in [Panel Parts](#Panel-Parts). Each Panel contains an Atmel Atmega8 microcontroller. If you plan to program the Atmel chip, it is a good idea to read the Atmega8 spec sheet. The other chip on the circuit board is an 8-channel darlington driver which acts as a current sink. Current is sourced from the Atmega8's outputs and then passes through a current limiting resistor and one column of the 8x8 LED display. An individual LED is turned on only if the corresponding row line is enabled on the 8-channel current sink, allowing current to flow. In this way it is possible to scan the entire 8x8 array
and thus 64 LEDs can be individually controlled with 16 lines. Of course, for this to happen no more than 1 row of LEDs can actually be on at any one time, and so the LEDs must be scanned quickly to prevent the perception of flicker. Currently the entire display is refreshed at 4 kHz, and every LED that is on, is only on 12.5% of the time. 

- [Panel v2.1 PCB (pdf)](https://github.com/reiserlab/Panel-G3-Hardware/blob/master/assets/Panel_PCB_v21.pdf)
- [v2.1 Panel schematic (pdf)](https://github.com/reiserlab/Panel-G3-Hardware/blob/master/assets/Panel_schematic_v21.pdf)

## Panel Software

The software running on each Panel's Atmega8 microcontroller is written in C code, and compiled using the GCC tools for the AVR (WinAVR distribution). Packets are sent to each panel over the TWI bus, the TWI unit on the Atmega8 will only accept a packet if it is addressed either with a 0 (general call) or the address of the panel. In this way specific panels can be targeted. The EEPROM of the Atmega8 is loaded with some patterns, so that the SRAM on the chip is available for pattern buffers. The EEPROM contains the character library for 0 - 9 and the patterns for the error codes. The EEPROM also has one byte designated for the panel ID, this is set to 0 during the programming.

### Support for Gray Scale

Multiple levels of //greenscale// patterns are supported by the panels. This is accomplished by further dividing the 4 kHz refresh of the panels into either 3 or 7 pieces. When divided into 3 time segments, 4 greenscale levels are possible - that is each LED can be on for 0, 1, 2, or 3 of the 3 possible time slices. Similarly, when divided into 7 time segments, 8 greenscale levels are possible - each LED can be on for 0,1,...,7 of the 7 total time slices. When measured with a photomoter, this scheme is remarkably linear, e.g. the luminance measured for greenlevel 6 is almost exactly 3 times the luminance measured for greenlevel 2. This does reduce the minimum refresh rate to about 570 Hz for the 7 levels (so-called `Gray_Scale = 7`), and to about 1333 Hz for the 3 levels (so-called `Gray_Scale = 3`). The simplest implementation of this would use 7 copies of the 64-bit pattern map, each bit specifying whether each LED should be on during each time slice. However, we take advantage of binary decoding - so we need twice the bits - 16 bytes for each pattern frame at `Gray_Scale = 3`, and 24 bytes for each panels piece of a `Gray_Scale = 7` pattern. The actual way this is done is just a lot of messy, uninteresting boolean algebra, please read the code if you are interested.

### Panel Address

The address of each panel can be any integer from 1 - 127. The 0 address is reserved for a general call - all panels receive packets sent to address 0. The address displays when the panels turn on (and after a reset). Also immediately after a rest or power on, the panels enter a 0.5 second pause, and will be unresponsive to commands and patterns. Only two characters are shown (00 -
99), and if the value is above 100, the top row of the panel is also  illuminated. It is necessary to give the panels addresses in an incremental manner. If 5 panels, all with address 0 are connected to the controller, and a change address command is issued, all panels with the same address will respond. Read the user guide for instructions on addressing panels. 

## Programming the Panels

Because each panel has a microprocessor on-board, each panel needs to be programmed individually. The programming interface (or ISP, in system programming) lines only connect to the microprocessor when the panel is in the left-most position of the controller board, or the plug on the case-mount portion of the controller PCB. The procedure described here is for programming G2 panels (the fuse setting are different for G3 panels). In some eventual future, programming will be done directly from the Matlab interface. As of right now the procedure is as follows:

1. Once the panel is in place, connect the programming device to the 6 pin header, labelled **Panel ISP**.
2. Open AVR Studio and start the AVR programming tool STK500/AVRISP from the tools menu.
3. Test the connection by selecting the Atmega8 device from the drop-down menu, and then in the advanced tab try to read signature. If these match, move on, otherwise check the connections, restart, etc.
4. Program the fuses - this is done from the **Fuses** tab – these options should be set: **Preserve EEPROM memory through chip erase cycle** to on, and  set clock rate to the last choice: **Ext. Crystal/Resonator High Freq.: Start-up time 16K CK + 64 ms**. It is also a good idea to set **Brown-out detection level at VCC=4.0V** and **Brown-out detection enabled**.
5. Program the EEPROM on the Atmega8 by selecting the **Program** tab and electing the input hex file as: `panel.eep`, and then programming the chip.
6. Program the flash on the Atmega8 by selecting the **Program** tab and selecting the input hex file as: `panel.hex`, and then programming the chip.
7. If this is all done correctly the LED should display 00 and the display should be bright with no flicker. If it appears dull or flickery - the clock was probably not set correctly. It is always a good idea to verify both the program and the fuses to make sure these are set correctly.

For programming many panels this process can be automated by using the **Auto** tab and selecting program FLASH, program EEPROM, and program fuses. Once a panel is programmed it needs to be addressed, this is done using the PControl software.

# Controller

The Controller reads in pattern data from the CF card memory, accepts commands from the PC control program, and sends pattern data to the panels over the TWI bus.

## Controller Hardware

The controller board contains 2 Atmega128 microcontrollers, one is dedicated to reading the pattern data from the CompactFlash card, and the other communicates with the PC and the panels display. The PCB design has been modified, so that connections with a case are rather simple.

* [Controller v2.2 PCB (pdf)](https://github.com/reiserlab/Panel-G3-Hardware/blob/master/assets/Controller_PCB_v22.pdf)
* [Controller v2.2 schematic (pdf)](https://github.com/reiserlab/Panel-G3-Hardware/blob/master/assets/Controller_schematic_v22.pdf)

### Power supplies

There are 2 options for supplying power to the controller board. An unregulated supply can be connected to the 3-pin header. This power is regulated down to 5 volts and used by the circuitry on the board. An inexpensive DC wall transformer is a good choice, at least 9V and 1-2 A, should be adequate. Use this connector when no panels (or just 1 or 2) are on the controller. The three pins are [GND V+ GND], the suggested connector to use is a three pin female: [GND V+ Blank]. This way there is no possibility of switching power and ground. The other option is a 5-Pin DIN connector regulated supply. Looking head on at the supply connector, should have (Left to Right): Ground, no connect, Ground, no connect, + 5V. The two pins are not connected, so if a pre-built supply has something on
these pins, it will be fine to use this supply, otherwise, modify the connector to this pinout. The 5V, 10 A power supply given in [Controller Parts](#Controller-Parts) works very well.

## Controller Software

The software running on the Controller is essentially the intermediary between the commands sent from the PControl program detailed in section ..., and the panels. The following section explains how and what the code does, read this if you are curious or plan on modifying the code. Hopefully most users can use the system directly from Matlab and remain blissfully ignorant of these details. The software on the controller has been designed with one goal - making sure patterns are streamed to the panels as quickly as possible, keep this in mind if some of this seems peculiar.

### Programming the Controller

Programming the controller is similar to the process for programming each panel. There is a 6-pin ISP interface for both of the controller board’s Atmega128 microprocessors. Let’s program the main controller first:

1. Power the Controller board if using the AVR ISP, otherwise power the STK 500, and connect the programming device to the 6 pin Controller ISP header (labeled J7 on the PCB).
2. Open AVR Studio and start the AVR programming tool STK500/AVRISP from the tools menu.
3. Test the connection by selecting the Atmega128 device from the drop-down menu, and then in the advanced tab try to read signature. If these match, move on, otherwise check the connections (rotate connector), restart, etc.
4. Program the fuses - this is done from the **Fuses** tab - a few options should be set: **ATMega103 compatibility mode** must be set to OFF, **JTAG interface enable** must be set to OFF, and clock rate must be set to the last choice: **Ext. Crystal/Resonator High Freq.: Start-up time 16K CK + 64 ms**. It is also a good idea to set **Brown-out detection level at VCC=4.0V** and **Brown-out detection enabled**.
5. There is no EEPROM file needed for this application.
6. Program the flash on the Atmega128 by selecting the **Program** tab and selecting the input hex file as: `mainctrl.hex`, and then programming the chip.
7. It is always a good idea to verify both the program and the fuses to make sure these are set correctly.
8. To program the CompactFlash controller, just repeat steps 1-7, except for:
    1. Unplug the CF card if there is one in the socket. Also plug the 6 pin ISP cable into the 6 pin ISP header for the CF controller (labeled J8 on the PCB). 
9. program with the file `cfctrl.hex`.

# To-dos & Wish List

As presented the system is fully functional and has been in regular use for more than a two years. However, there are many features that will improve the system. Here is a list of suggested projects. Some of these will be completed by MBR (many will certainly not be), but if anyone is interested in tackling these or suggesting other projects, please send email to the [mailing list](#mailing-list)

- programming of the panels from the Matlab GUI, maybe even address them during programming. This would be done using the command line interface to the AVR Studio, or similar. **Possible but not yet implemented in PDC v3.**
- A small box that converts ±5 V to two distinct 0-5 V lines, mapping positive voltage to one and negative to the other. This would really simplify using certain signals with controller. Ideally, put 2 of these in one box (will need independent power). **Not necessary with PDC v3, since inputs and outputs can go ±5 V.**
- program panels when they are plugged in, so they don’t need to be programmed independently. Apparently this is possible, especially since the panels are set up to communicate with the TWI interface. Look at **Boot Loader Support** section in the AVR documentation. **PDC v3 and more recent software development supports this.**
- re-mapable greenscales – the idea here is to be able to use binary, 2 level, patters and then be able to arbitrarily map the 0 and 1 values to any of the 8 greenscale levels. This scheme has basically been worked out (is in the distributed panels code, but commented out). This system needs to be tested, etc.
- more efficient CF reading – that would start moving the bytes from the FIFO before the pattern is completely dumped – this should provide at least 10% faster frame rates. **Superseded by faster SD card reading in PDC v3**
- more user-friendly installation that checks the CF card location, makes a temp folder if necessary, etc. 
- Some buttons on the PControl gui for quickly linking to experiment scripts.
- Move storage of internal function generator functions onto the CF card, maybe increase the temporal resolution (100 Hz seems reasonable), and allow the function buffer length to vary. **Implemented in PDC v3**

# Parts Lists

Prices given are approximate (and circa 2006), are for 1 item of each, and apply to small quantity orders.

## Controller Parts

These parts are required to make 1 controller: 

| quantity | part description               | source & p/n |
| -------- | ------------------------------ | ------------ |
|2         | [Atmega128](http://www.atmel.com/dyn/resources/prod_documents/doc2467.pdf) microcontroller, 64 TQFP package  | Digikey, ATMEGA128-16AI-ND ($10)  |
|1       | [CY7C4251V-15AC FIFO](http://rocky.digikey.com/WebLib/Cypress/Web%20Data/CY7C4201V,11V,21V,31V,41V,51V,4421V.pdf), 32 TQFP package |      Digikey, 428-1229-ND ($18) |           
|2       |  [DAC 8571](http://focus.ti.com/lit/ds/symlink/dac8571.pdf), I2C 16 bit, 8 MSOP package |     Digikey, 296-14307-1-ND ($5)|
|1       | [Maxim RS-232 driver](http://pdfserv.maxim-ic.com/en/ds/MAX220-MAX249.pdf) |     Digikey, MAX233ACPP-ND ($8) |
|5       | 150 Ohm 1/4 W resistor, 1206 package | Digikey, 311-150ECT-ND ($0.02) |          
|9       | 10K Ohm 1/4 W resistor, 1206 package | Digikey, 311-10KECT-ND ($0.02) |          
|2      |   16 MHz crystal oscillator   |   Digikey, X192-ND ($0.70)|
|14     |   BNC connector, vertical PCB, mount  |  Digikey, A24504-ND ($1.50)       |
|1      |   CF connector                   |Digikey, 478-2012-ND ($3.00)|
|2      |   10-pin ribbon connector        |Digikey, M3AAA-1006R-ND ($2.50)|
|2      |   6-pin male double-row headers  |Digikey, |
|1      |   7805T, 5 V regulator           |Jameco, 51262 ($0.30) – optional!|
|1      |   10 uH RF choke (inductor)      |Jameco, 208135 ($0.60)|
|1      |   10 uF capacitors, tantalum     |Jameco, 33689 ($0.60)|
|8      |   0.1 uF coupling caps, tant.    |Jameco, 332110 ($0.20)|
|2      |   20-pin DIP socket              |Jameco, 38607CL ($0.10)|
|5      |   rt. Angle, PCB mount LEDs      |Jameco, 104256(gr.), 104248(red) ($.25)|
|2      |   push button switches           |Jameco,    |
|4      |   22 pF (20%) capacitors         |Jameco, 15405 ($0.05)|
|3      |   9-pin fem. rt. angle D-sub     |Jameco, 104951 ($0.50)|
|1      |   5 pin fem. DIN connector       |Jameco, 29399 ($0.50)|
|1-2    |   male single row “0.1 header    |Jameco, 160881 ($0.40)|
|4      |   fem. single row header, 8 pin  |Jameco, 70754 ($0.40)|
|2      |   TWI resistors – std. 1k Ohm    |Jameco, 29663 (peanuts)|
|1      |   SPST toggle switch             |Jameco, 76523 ($1.00)|
|1      |   power supply, 5V, min. 2A	   |  Jameco, 221487 (w DIN conn.) ($35)	|
|1      |   Controller PCB		   |  ordered from Advanced Circuits |

Notes: male single row headers are needed in sizes of 2 and 3 - easiest thing to do is cut these from a larger strip. Female single row headers are needed in 8 row size, so easiest thing is to order these as a pre-sized piece from Jameco (also needed for flight arenas).

## Panel Parts

These parts are required to make 1 panel. Part descriptions for major components are linked to spec sheets: 

| quantity | part description               | source & p/n |
| -------- | ------------------------------ | ------------ |
|1         |  8*8 green [LED display](https://github.com/reiserlab/Panel-G3-Software/blob/master/assets/Green%20Panels%20BM-10288MD.pdf)    | American Bright, BM-10288MI ($2.50)|
|1         |  8 Pin rt angle male header  | Major League, LTSHR-108-S-02-A-T ($0.23)|
|1         |  8 Pin rt angle fem. header  | Major League, SSHR-108-S-02-L-T ($0.38)|
|8         |   82 Ohm resistor, 1/10W, 1%, 0603 package | Digikey, 311-82.0HTR-ND ($0.005)|
|1         |   1K Ohm resistor, 1/10W, 5%, 0603 package | Digikey, 311-10KGCT-ND ($0.02)|
|1         |   0.33 uF capacitor, ceramic, 1206 package  | Digikey, PCC1889CT-ND ($0.09)|          
|1         |   16 MHz ceramic resonator w. capacitors    | Digikey, X908-ND ($0.30)|
|1         |   8CH line driver ULN2804A 8-SOL package (update link)| Digikey, ULN2804AFW-ND ($0.35)|
|1         |  [Atmega8](http://www.atmel.com/dyn/resources/prod_documents/doc2486.pdf) microcontroller, 32 TQFP package| Digikey, ATMEGA8-16AI-ND ($2.25)|
|1         |    Panel PCB			|   ordered from Advanced Circuits		|

Optional: 2 8-Pin straight female headers, Jameco p/n 70754 ($0.23), can be used so that the LED displays are socketed, this is convenient, but also will not guarantee ideal contact between circuit and display. 

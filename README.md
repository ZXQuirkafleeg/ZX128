# ZX128 - ZX Spectrum 128 RC2014 Card

This is an ZX Spectrum implementation implemented in the popular RC2014 modular format (https://rc2014.co.uk).  The primary purpose is to provide a flexible spectrum hardware development environment – and save an aging spectrum from damage when developing hardware.  

The base board borrows heavily from the design on Superfo’s excellent ZX Max [(link)](https://speccy.pl/wiki/index.php?title=ZX_Max_48).  It uses an Altera EPM7128SLC CPLD to replace the 6C/7C ULA.  It is not intended to a perfect replication of a Spectrum 48 or 128 but works well for most hardware applications and has played all the software tried so far.

Notable features are:

*	Spectrum 48 and 128K support
*	PS2 and spectrum membrane keyboard interface
*	Video output suitable for SCART or 15.7KHz VGA display, or GBS200 source
*	Readily available source of functional and prototyping cards
*	Single 5V supply (<500 mA)
*	Speaker and AY Sound output is not fully implemented as there is a separate sound card

Three cards are needed to make a working ZX Spectrum system:

1.	ZX128 Spectrum card (this card)
2.	RC2014 Z80 CPU card (any standard RC2014 card)
3.	Passive backplane (any standard RC2014 backplane)

![ZX128 Development system](https://github.com/ZXQuirkafleeg/ZX128/blob/main/Images/System.jpg)

All of the cards, except the motherboard, are 100mm by 100mm or smaller so can be cheaply fabricated by the usual off-shore PCB manufacturing companies.

## Video Output

Video output in via an 8 pin mini DIN connector which has RGB, vertical sync, combined sync and audio.  RGB plus combined sync should drive a SCART lead (or HDMI via a converter box) and RGB plus both syncs will drive a wide input range (15.7 KHz) VGA monitor or std VGA monitor via a GBS-8200 video converter. Please note a std VGA monitor will not work directly - it must support 15.7 Khz.

Cable details are:

Signal          | 8 Pin mini DIN	| DB15 VGA	| SCART
:---:           | :---:           | :---:     | :---:
Gnd	            | 1		
Vertical sync	  | 2		
Vcc (via 270R)  |	3		
Green	          | 4		
Audio	          | 5		
Red 	          | 6		
Blue	          | 7		
Composite Sync	| 8		

## Sound Output

Speaker and AY Sound output is not fully implemented as there is a separate sound card; however, J8 and J6 provide ULA sound and AY port decoding respectively.  

J8 – ULA Sound:

*	Pin 1 – GND
*	Pin 2 – Mono ULA audio – Line out
*	Pin 3 – (same as Pin 2)
*	Pin 4 – CN2 Pin 5 (audio out)

Pins 1-3 can be used to connect to an amplifier line input.  Shorting pins 3&74 provides the ULA audio output to pin 5 of the video connector.

J6 – AY8910/12:

*	Pin 1 – GND
*	Pin 2 – AY_CLOCK (~1.7 MHz)
*	Pin 3 – AY_BDIR
*	Pin 4 – AY_BC1

If needed these decoded signals can be used to drive an AY PSG.  Alternatively, a standalone 128 Compatible PSG sound board could be used (e.g. https://github.com/electrified/rc2014-ym2149) or the i/o pins could be repurposed in the CPLD VHDL code.

## Keyboard

The board supports a PS2 or standard spectrum keyboard.  For the standard spectrum keyboard, D1-D8, RN2, JP4 and 5 FPC connectors should be populated.  You may need an extender cable to make the keyboard practical depending on its position.

The PS2 interface uses an ATMega8A with the Avray V5.5 keyboard firmware [(link)](https://www.avray.ru/ru/zx-spectrum-ps2-keyboard/) and U6, X2, CN1, R29, R30 and R41 should be populated.  The keyboard emulator has some issues, for example E-mode FORMAT and CAT keys are moved right by one key, but generally works well.

## RC2014 Bus

There are some differences between the RC2014 bus and the Spectrum edge connector signals.  Notably the CPU clock is inverted on the edge connector.  Spectrum peripherals also use ROMCS, NMI and WAIT (e.g. DIVMMC, Multiface and Inferface 1), so these are brought out to the user pins on the RC2014 bus:

*	Pin 37 – User 1 – ROMCS
*	Pin 38 – User 2 – NMI 
*	Pin 39 – User 3 – WAIT 
*	Pin 40 – User 4 – (unallocated)
 
## Build
All components are though-hole and standard density.  A BOM can be found [here](https://github.com/ZXQuirkafleeg/ZX128/blob/main/PCB/ZX128%20V1.1%20BOM.csv).

### PCBs

#### ZX128 Board

For the PCB, EasyEDA (V6.5.22) design files can be found [here](https://github.com/ZXQuirkafleeg/ZX128/tree/main/PCB) and gerbers [here](https://github.com/ZXQuirkafleeg/ZX128/blob/main/PCB/Gerber_PCB-%20RC2014-%20128K%20ZX%20Spectrum-%20V1.1.zip).  The PCB is a standard two layer board approx. 100mm by 100mm.  Bus isolation and reset resistors are placed under the IC sockets for U3, U4 and U6 to tidy the layout and maintain the small footprint.

![Front of ZX128 board V1.1 with PS2 keyboard](https://github.com/ZXQuirkafleeg/ZX128/blob/main/Images/ZX128%20-%20Front.jpg)

#### CPU Board

Any standard RC2014 Z80 module should work as long as the CPU can be driven by the bus (ULA) clock (and not a standalone oscillator).   Karl Brokstad’s version [(link)](https://www.z80.no/modules1/21d.html) works well and the Gerbers are [here](https://www.z80.no/resources/21d-CPU+CLK/Gerber_21d_CPU_and_Clock_20190130195607.zip).   The standalone oscillator circuit (U1, C2-C4, R6, R7 and X1) is not needed.

![Front of RC2014 CPU board](https://github.com/ZXQuirkafleeg/ZX128/blob/main/Images/CPU%20-%20Front.jpg)

Other boards, such as the disk interface, use NMI and WAIT which are not available on the standard 40 Pin RC2014 bus.  For simplicity, NMI is connected to U2 and WAIT to U3 using wirewrap as shown below.  Neither connection is needed to get the basic spectrum working.

![Rear of RC2014 CPU board showing changes](https://github.com/ZXQuirkafleeg/ZX128/blob/main/Images/CPU%20-%20Rear.jpg)

#### Backplane

Any passive backplane should work.  Again, Karl’s Blackplane 8 [(link)](https://www.z80.no/backplanes/pb8b.html) works well and only a SIL connector is needed for each board.  Gerbers are [here](https://www.z80.no/resources/BackPlanes-revB/08/08b_Gerber.zip).  This board is 130mm by 150mm through there may be smaller versions available, with less slots, which fit within the JLC/Pcbway cheap PCB fabrication size limit.

![RC2014 Passive backplane](https://github.com/ZXQuirkafleeg/ZX128/blob/main/Images/Backplane.jpg)

### Device Programming	

#### EPM7128SLC (ULA)

Firmware for the CPLD implementation is here and a POF here.  The implementation is the same as for the ZX Max 128 with two changes:

1.	IORQULA now includes RD as ATMega keyboard requires IORQ RD xFE to enable outputs
2.	VSync is now connected to a CPLD VGA support

Programming the CPLD can be done by the Programmer app in Quartus 13.1.  Connect a USB Blaster dongle (or clone) to J7 and connect the 5V supply to the ZX128 board.
 
#### ATMega8A (PS2 Keyboard)

Keyboard firmware is located here and the 20Mhz ATMega version (KBD13_M8_nw_MODIFIEDv5_5_20MHz.hex) was used.  Firmware can programmed using an TL866II, Arduino as an ISP or anything else appropriate.  

#### 27C512 (ROM)

An example system ROM can be found [here](https://github.com/ZXQuirkafleeg/ZX128/blob/main/ROM/Combined%20ROM%20-%20ZX128%20-%20D106.bin) and the memory map is:

*	0K - 16K	ZX Spectrum 128 Editor ROM
*	16K - 32K	ZX Spectrum 128 System ROM
*	32K - 48K	Retroleum test ROM V1.41 - [link](https://blog.retroleum.co.uk/electronics-articles/a-diagnostic-rom-image-for-the-zx-spectrum/)
*	48K - 64K	Brendan Alford’s diagnostic ROM V0.37 - [link](https://github.com/brendanalford/zx-diagnostics/wiki)

Again, an TL866II or similar can be used.  

ROM A14 and A15 are usually driven by the CPLD.  A14 switches between 128K editor and system ROMs and A15 is intended for a future +2A/+3 implementation.  Either signal can be overridden by populating J3.  For example, shorting pins 3&5 forces a direct boot to the 48K ROM, and shorting pins 1&3 and 4&6 forces a boot into the Retroleum test ROM.

### Power Supply

PSU, as for the RC2014, should be a regulated 5V supply.  The base design requires around 250 mA.  Connector U3 on the backplane is centre positive.

### Jumpers and Connectors

* J1 – RC2014 Bus connector
* J2 – ROMCS jumper – used connect ROMCS to U1 line of RC2014 bus
* J3 – ROM A14 and A15 select jumper
* J4 – ZX Spectrum membrane Keyboard row FPC connector
* J5 – ZX Spectrum membrane Keyboard column FPC connector
* J6 – AY-3-8910 control and clock signals for external PSG
* J7 – Altera USB Blaster ICSP connector
* J8 – ULA Ear, Mic and Speaker sound output
* CN1 – PS2 Keyboard connector – 5 pin mini DIN
* CN2 – RGB Video output – 8 pin mini DIN
* CN3 – Ear/Mic connector – 3.5mm stereo plug

## System Test

No jumper is needed on J2 (ROMCS) and, with the example system ROM, J3 can remain un-jumpered to boot into Spectrum 128 mode.   Retroleum test ROM can be enabled by shorting J3 pins 1&3 and 4&6.

Install the CPU and ZX128 boards in the backplane and apply power.  Current draw is around 250 mA with ATMega keyboard controller fitted.



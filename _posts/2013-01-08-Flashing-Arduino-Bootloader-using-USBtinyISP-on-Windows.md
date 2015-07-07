---
layout: post
Title: Flashing Arduino Bootloader using USBtinyISP on Windows
Date: 2013-01-08 19:00  
tags: [Electronics, Open Source, Programing]
---

So the bootloader on my arduino got corrupted somehow, lets try and fix it! This method will work with any 'blank' atmega328p.

Tools
-----

1.  USBtinyISP from [Adafruit](http://www.ladyada.net/make/usbtinyisp/)
2.  Arduino dev board
3.  WinAVR aka avrdude; [Dowload](http://sourceforge.net/projects/winavr/), [Setup](http://www.ladyada.net/learn/avr/setup-win.html)
4.  Arduino IDE

Setup
-----

First make sure you have all the software/drivers setup for the above. Connect the programer to the ICSP port on the arduino make sure you get the right port and get it the right way round!

![Connected up](https://dl.dropbox.com/u/78443198/apps/scriptogram/icsp.jpg)

You need to power up both the dev board and the programmer. Now test that AVR dude can communicate with the board be running the following by reading the fuses of the chip

	C:\Users\Phil>avrdude -B 25 -c usbtiny -p m328p

-B 25 specifies the speed -c usbtiny specifies the device we are using (USBtinyISP) and -p indicates the target chip is a atmel atmega328p.

You should see this result:

	avrdude: AVR device initialized and ready to accept instructions

	Reading | ################################################## | 100% 0.02s

	avrdude: Device signature = 0x1e950f

	avrdude: safemode: Fuses OK

	avrdude done.  Thank you.

Fuses
-----

lets setup the fuses. First its worth checking them so lets reconnect but this time in terminal mode.

	C:\Users\Phil>avrdude -B 25 -c usbtiny -p m328p -t

You will recive the same output as before but end up with an "avrdude>" prompt. There are 3 fuses we are intrested in hfuse, lfuse and efuse. To check fuse values you can use the dump command.

	avrdude> dump hfuse
	>>> dump hfuse
	0000  de

For our the arduino bootloader the defualts are hfuse:0xDA lfuse:0xFF and efuse:0x05

to exit terminal mode type "quit"
	
	avrdude> quit
	>>> quit

	avrdude: safemode: Fuses OK

	avrdude done.  Thank you.

If any of our fuses where incorrect we can set them using the commands below

	avrdude -B 25 -c usbtiny -p m328p -u -U hfuse:w:0xDA:m

	avrdude -B 25 -c usbtiny -p m328p -u -U efuse:w:0x05:m

	avrdude -B 25 -c usbtiny -p m328p -u -U lfuse:w:0xFF:m

Bootloader
----------

You can find the bootloader in the place you installed the arduino IDE. The path should be somthing like:

	arduino-x.x.x\hardware\arduino\bootloaders

The file we want to burn is "ATmegaBOOT_168_atmega328.hex" so to burn we run:

	avrdude -B 1 -c usbtiny -p m328p -u -U flash:w:ATmegaBOOT_168_atmega328.hex

This will take a little bit of time the output should look somthing like:

	avrdude: AVR device initialized and ready to accept instructions

	Reading | ################################################## | 100% 0.01s

	avrdude: Device signature = 0x1e950f
	avrdude: NOTE: FLASH memory has been specified, an erase cycle will be performed
	         To disable this feature, specify the -D option.
	avrdude: erasing chip
	avrdude: reading input file "ATmegaBOOT_168_atmega328.hex"
	avrdude: input file ATmegaBOOT_168_atmega328.hex auto detected as Intel Hex
	avrdude: writing flash (32670 bytes):

	Writing | ################################################## | 100% 18.33s



	avrdude: 32670 bytes of flash written
	avrdude: verifying flash memory against ATmegaBOOT_168_atmega328.hex:
	avrdude: load data flash data from input file ATmegaBOOT_168_atmega328.hex:
	avrdude: input file ATmegaBOOT_168_atmega328.hex auto detected as Intel Hex
	avrdude: input file ATmegaBOOT_168_atmega328.hex contains 32670 bytes
	avrdude: reading on-chip flash data:

	Reading | ################################################## | 100% 12.98s



	avrdude: verifying ...
	avrdude: 32670 bytes of flash verified

	avrdude done.  Thank you.


Thats it! Its a good idea at this point to verify the bootloader installed ok by loading a small program onto the chip, happy programming.
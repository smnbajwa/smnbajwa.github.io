---
layout: post
title:  "From Arduino to AVR"
---

To make the ATMega328p MCU work without the Arduino Uno board, we need two things.

1. A programmer(ISP)
2. Change fuse bits

**Making a programmer (ISP)**  
This is a piece of hardware that lets us program the 328p via USB. Instead of buying a programmer from the market, I am going to use an Arduono Nano. To make things easy, Arduino IDE comes with an example sketch called 'ArduinoISP'.

Once this sketch is uploaded to the Nano, my ISP is ready.

**Changing fuse bits**  
Uno has a 16 MHz oscillator on the board and 328p is configured at Arduino factory to use that. Once we remove it from the board, it won't have a clock. Luckily, it comes with an onboard 8 MHz oscillator and we will use that.

The way to do that is by changing something called 'fuse bits'. These are locations in memory where MCU configuration is stored.

But before we do that, let's wire up our ISP and Uno board.

As per the [datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/ATmega48A-PA-88A-PA-168A-PA-328-P-DS-DS40002061B.pdf), ISP needs three wires connected to the target (our Uno) as shown in the picture below  
![Serial](/assets/img/serial_programming.png "Programming pins")
![pinMapping](/assets/img/serial_pinmapping.png "Pin mapping")  

Looking at the Uno pin mapping below, these are just digital pins 11, 12, 13.  

![mapping](https://docs.arduino.cc/static/47180601da55c458736e09d26b8bfab5/d0d8c/Atmega168PinMap2.png "Arduino Pin Mapping")

Therefore, pin connections look something like this:

ISP (Nano)  ---->   Uno  
13 ----> 13  
12 ----> 12  
11 ----> 11  
10 ----> RESET (As per ArduinoISP sketch)  
5V ----> 5V  
GND ----> GND  

![circuit](/assets/img/setup.jpg "Circuit")

  
To access the fuse bits, we need to use a tool called AVRdude. We can install it ourselves but Arduino IDE comes with it already.  Looking at the verbose output from Arduino, I discover that avr binary is located in: 
```
/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17/bin/avrdude
```
I cd into `6.3.0-arduino17` directory and run this command:
```
./bin/avrdude -C ./etc/avrdude.conf -v -p atmega328p -c stk500v1 -P /dev/ttyUSB0 -b 19200 -U lfuse:r:/tmp/lfusesetting:h
```
All this command does is this:
1. Specify avrdude.conf file using -C
2. Print verbose output using -v
3. Specify target mcu atmega328p using -p
4. Programmer I am using '-c stk500v1' (I got this from Arduino log)
5. Port where my device is connected (-P /dev/ttyUSB0)
6. Baud rate (-b 19200)
7. -U specifies a memory operation. This needs exaplanation.  
    ```
    lfuse:r:/tmp/lfusesetting:h  
    'lfuse' --> access lfuse byte  
    'r' --> read the memory and write to specified file  
    '/tmp/lfusesetting' --> specified file  
    'h' --> format of the file, hex in this case
    ```

Check [this](https://www.nongnu.org/avrdude/user-manual/avrdude_3.html#Option-Descriptions) link for more information about avrdude.

You should see this output.
```
saman@saman-pc:~/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17$ sudo ./bin/avrdude -C ./etc/avrdude.conf -v -p atmega328p -c stk500v1 -P /dev/ttyUSB0 -b 19200 -U lfuse:r:/tmp/lfusesetting:h

avrdude: Version 6.3-20190619
         Copyright (c) 2000-2005 Brian Dean, http://www.bdmicro.com/
         Copyright (c) 2007-2014 Joerg Wunsch

         System wide configuration file is "./etc/avrdude.conf"
         User configuration file is "/root/.avrduderc"
         User configuration file does not exist or is not a regular file, skipping

         Using Port                    : /dev/ttyUSB0
         Using Programmer              : arduino
         Overriding Baud Rate          : 19200
         AVR Part                      : ATmega328P
         Chip Erase delay              : 9000 us
         PAGEL                         : PD7
         BS2                           : PC2
         RESET disposition             : dedicated
         RETRY pulse                   : SCK
         serial program mode           : yes
         parallel program mode         : yes
         Timeout                       : 200
         StabDelay                     : 100
         CmdexeDelay                   : 25
         SyncLoops                     : 32
         ByteDelay                     : 0
         PollIndex                     : 3
         PollValue                     : 0x53
         Memory Detail                 :

                                  Block Poll               Page                       Polled
           Memory Type Mode Delay Size  Indx Paged  Size   Size #Pages MinW  MaxW   ReadBack
           ----------- ---- ----- ----- ---- ------ ------ ---- ------ ----- ----- ---------
           eeprom        65    20     4    0 no       1024    4      0  3600  3600 0xff 0xff
           flash         65     6   128    0 yes     32768  128    256  4500  4500 0xff 0xff
           lfuse          0     0     0    0 no          1    0      0  4500  4500 0x00 0x00
           hfuse          0     0     0    0 no          1    0      0  4500  4500 0x00 0x00
           efuse          0     0     0    0 no          1    0      0  4500  4500 0x00 0x00
           lock           0     0     0    0 no          1    0      0  4500  4500 0x00 0x00
           calibration    0     0     0    0 no          1    0      0     0     0 0x00 0x00
           signature      0     0     0    0 no          3    0      0     0     0 0x00 0x00

         Programmer Type : Arduino
         Description     : Arduino
         Hardware Version: 2
         Firmware Version: 1.18
         Topcard         : Unknown
         Vtarget         : 0.0 V
         Varef           : 0.0 V
         Oscillator      : Off
         SCK period      : 0.1 us

avrdude: AVR device initialized and ready to accept instructions

Reading | ################################################## | 100% 0.01s

avrdude: Device signature = 0x1e950f (probably m328p)
avrdude: safemode: lfuse reads as FF
avrdude: safemode: hfuse reads as DE
avrdude: safemode: efuse reads as FD
avrdude: reading lfuse memory:

Reading | ################################################## | 100% 0.01s

avrdude: writing output file "/tmp/lfusesetting"

avrdude: safemode: lfuse reads as FF
avrdude: safemode: hfuse reads as DE
avrdude: safemode: efuse reads as FD
avrdude: safemode: Fuses OK (E:FD, H:DE, L:FF)

avrdude done.  Thank you.
```

If you see the following error, try adding a 100uF capacitor between RST and GND pin of your ISP.
```
avrdude: stk500_initialize(): (b) protocol error, expect=0x10, resp=0x01
```

Now, the lfuse bits should be saved in `/tmp/lfusesetting` so I open that file using 'sudo nano' and it shows '0xff'

As per the datasheet, to use the Internal clock, bits CKSEL3 to CKSEL0 must be set to '0010'.
![CKSEL_settings](/assets/img/cksel_settings.png "CKSEL bits")
![CKSEL](/assets/img/cksel.png "CKSEL bits")

We will leave bits 4-7 unchanged which makes our desired lfuse byte 0xF2.

To do that, run this command, which is similar to the one we used before, except that instead of reading and saving lfuse to a file, we are writing to it.
```
saman@saman-pc:~/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17$ sudo ./bin/avrdude -C ./etc/avrdude.conf -v -p atmega328p -c stk500v1 -P /dev/ttyUSB0 -b 19200 -U lfuse:w:0xF2:m
```
You should see the following output confirming that lfuse has been set to F2
```
saman@saman-pc:~/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17$ sudo ./bin/avrdude -C ./etc/avrdude.conf -v -p atmega328p -c stk500v1 -P /dev/ttyUSB0 -b 19200 -U lfuse:w:0xF2:m

avrdude: Version 6.3-20190619
         Copyright (c) 2000-2005 Brian Dean, http://www.bdmicro.com/
         Copyright (c) 2007-2014 Joerg Wunsch

         System wide configuration file is "./etc/avrdude.conf"
         User configuration file is "/root/.avrduderc"
         User configuration file does not exist or is not a regular file, skipping

         Using Port                    : /dev/ttyUSB0
         Using Programmer              : stk500v1
         Overriding Baud Rate          : 19200
         AVR Part                      : ATmega328P
         Chip Erase delay              : 9000 us
         PAGEL                         : PD7
         BS2                           : PC2
         RESET disposition             : dedicated
         RETRY pulse                   : SCK
         serial program mode           : yes
         parallel program mode         : yes
         Timeout                       : 200
         StabDelay                     : 100
         CmdexeDelay                   : 25
         SyncLoops                     : 32
         ByteDelay                     : 0
         PollIndex                     : 3
         PollValue                     : 0x53
         Memory Detail                 :

                                  Block Poll               Page                       Polled
           Memory Type Mode Delay Size  Indx Paged  Size   Size #Pages MinW  MaxW   ReadBack
           ----------- ---- ----- ----- ---- ------ ------ ---- ------ ----- ----- ---------
           eeprom        65    20     4    0 no       1024    4      0  3600  3600 0xff 0xff
           flash         65     6   128    0 yes     32768  128    256  4500  4500 0xff 0xff
           lfuse          0     0     0    0 no          1    0      0  4500  4500 0x00 0x00
           hfuse          0     0     0    0 no          1    0      0  4500  4500 0x00 0x00
           efuse          0     0     0    0 no          1    0      0  4500  4500 0x00 0x00
           lock           0     0     0    0 no          1    0      0  4500  4500 0x00 0x00
           calibration    0     0     0    0 no          1    0      0     0     0 0x00 0x00
           signature      0     0     0    0 no          3    0      0     0     0 0x00 0x00

         Programmer Type : STK500
         Description     : Atmel STK500 Version 1.x firmware
         Hardware Version: 2
         Firmware Version: 1.18
         Topcard         : Unknown
         Vtarget         : 0.0 V
         Varef           : 0.0 V
         Oscillator      : Off
         SCK period      : 0.1 us

avrdude: AVR device initialized and ready to accept instructions

Reading | ################################################## | 100% 0.02s

avrdude: Device signature = 0x1e950f (probably m328p)
avrdude: safemode: lfuse reads as FF
avrdude: safemode: hfuse reads as DE
avrdude: safemode: efuse reads as FD
avrdude: reading input file "0xF2"
avrdude: writing lfuse (1 bytes):

Writing | ################################################## | 100% 0.02s

avrdude: 1 bytes of lfuse written
avrdude: verifying lfuse memory against 0xF2:
avrdude: load data lfuse data from input file 0xF2:
avrdude: input file 0xF2 contains 1 bytes
avrdude: reading on-chip lfuse data:

Reading | ################################################## | 100% 0.01s

avrdude: verifying ...
avrdude: 1 bytes of lfuse verified

avrdude: safemode: lfuse reads as F2
avrdude: safemode: hfuse reads as DE
avrdude: safemode: efuse reads as FD
avrdude: safemode: Fuses OK (E:FD, H:DE, L:F2)

avrdude done.  Thank you.
```

And we are done.  
Go ahead and pull the 328p from the Uno board. Use a small flat-head screwdrivers and be careful not to bend any of the pins.

![ATMEGA](/assets/img/atmega_removed.jpg "ATMega328p removed")

**Conclusion**  
It might be a lot to take in if you are just getting started but really there are only two steps to this. First, we made a programmer out of an Arduino Nano. Second, we changed lfuse on the MCU to use its internal oscillator.

In the following posts, I will be programming 328p directly in C using VS Code without relying on Arduino libraries. 

Also, remember that I am new to embedded systems as well. This blog is just my attempt to better understand what I am learning by teaching it to others.

**Resources**  
Danielle Thurow's blog:  
[https://daniellethurow.com/blog/2021/5/25/directly-programming-an-atmega328p-from-an-arduino-uno](https://daniellethurow.com/blog/2021/5/25/directly-programming-an-atmega328p-from-an-arduino-uno)

Mitch Davis' playlist on bare metal programming:  
[https://www.youtube.com/watch?v=tBq3sO1Z-7o&list=PLNyfXcjhOAwOF-7S-ZoW2wuQ6Y-4hfjMR&pp=iAQB](https://www.youtube.com/watch?v=tBq3sO1Z-7o&list=PLNyfXcjhOAwOF-7S-ZoW2wuQ6Y-4hfjMR&pp=iAQB)



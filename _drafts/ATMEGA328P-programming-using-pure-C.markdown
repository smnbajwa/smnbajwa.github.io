---
layout: post
title:  "Program ATMega328p in pure C [Part 1]"
---

Arduino board and IDE is helpful when you are just starting but I had always wondered how it all worked under the hood.

Arduino Uno is based on ATMega328P microprocessor. It belongs to a class of MCUs(Microcontroller unit) called AVR. What the guys at Arduino did was grab this chip, put it on a board with a few other components, and built a library of functions to help beginners get started without worrying about low level details.

My goal is to be able to program a MCU directly using C programming language without relying on Arduino Library. I don't know how to do this yet. I am learning and I will post my journey here.

To that end, I will start with the Blink example that comes with Arduino and start removing 'training wheels' one by one.

Let's get started.

** Programmer **

ATMega328p on Uno board has 28 pins and no USB port. So how are we going to program it?
<DIP pinout>
We need something called an ISP (In-system Programmer). They are available in the market like the one here <AVR ISP>. 
We can also use an Arduino board as a programmer. I am going to use Arduino Nano for this purpose.
This step is pretty straightforward. Upload the example sketch called ArduinoISP that comes with Arduino IDE to the Nano and you have your own programmer ready. I am going to refer to this Nano as ISP and ATMega328p as '328p'.

** Fuse bits **

Uno has a 16 MHz oscillator on the board and 328p is configured at Arduino factory to use that. Once we remove it from the board, it won't have a clock. Luckily, it comes with an onboard 8 MHz and we are going to use that.

The way to do that is by changing something called 'fuse bits'. These are locations in memory where MCU configuration is stored.


Taking a look at the Blink.ino, you will see functions such as setup(), loop(), pinMode(), etc.
These are not standard C library functions. They come from Arduino library and our first step is to rewrite this program without using these functions.

<Blink.ino example>

From a couple of quick searches, I have learned that it comes down to these steps:

* Get a piece of hardware called a programmer to program the chip 
* Change fuse bits before removing ATMega328p from Arduino Uno board (explanation coming later)
* Wire everything up and program the chip

** Programmer **
It is possible to use another Arduino board as a programmer. I will be using an Arduino Nano but feel free to use other boards.
Going forward, I will refer to Nano board as ISP (In-system programmer).

This step is pretty straightforward. Arduino IDE comes with an example sketch called ArduinoISP. Upload it to Nano and you have your programmer.






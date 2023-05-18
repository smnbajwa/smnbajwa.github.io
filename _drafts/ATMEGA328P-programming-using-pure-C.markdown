---
layout: post
title:  "Program ATMega328p in pure C [Part 1]"
---

Arduino board and IDE is helpful when you are just starting but I had always wondered how it all worked under the hood.

Arduino Uno is based on ATMega328P microprocessor. It belongs to a class of MCUs(Microcontroller unit) called AVR. What the guys at Arduino did was grab this chip, put it on a board with a few other components, and built a library of functions to help beginners get started without worrying about low level details.

My goal is to be able to program a MCU directly using C programming language without relying on Arduino Library. I don't know how to do this yet. I am learning and I will post my journey here.

To that end, I will start with the Blink example that comes with Arduino and start removing 'training wheels' one by one.

Let's get started.

I am using Arduino Uno board and I have it wired it up as shown.

<picture of blink circuit>
  
 I am going to open the blink example sketch but before we do that, let's turn on the verbose output
 from File > Preferences. This will allow us to see what's going on under the hood.
  
 Now, go ahead and upload blink.ino. The LED should be blinking.
  
 The code for the blink.ino looks like this:
``` c
    // the setup function runs once when you press reset or power the board
    void setup() {
        // initialize digital pin LED_BUILTIN as an output.
        pinMode(LED_BUILTIN, OUTPUT);
    }

    // the loop function runs over and over again forever
    void loop() {
        digitalWrite(LED_BUILTIN, HIGH);   // turn the LED on (HIGH is the voltage level)
        delay(1000);                       // wait for a second
        digitalWrite(LED_BUILTIN, LOW);    // turn the LED off by making the voltage LOW
        delay(1000);                       // wait for a second
    } 
```

 






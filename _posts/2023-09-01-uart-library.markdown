---
layout: post
title: Writing a simple UART library
---

# Objective
In a previous [post](https://smnbajwa.github.io/2023/08/03/making-avr-talk-with-pc-using-UART.html), I explained how to make AVR communicate with my PC using UART. Since I will be using UART in other projects, I wanted to write a simple library for my own use.

[Link to github repo](https://github.com/smnbajwa/myUART)

What functions should this library have? To begin with:

1. Initialize UART
2. Transit a byte to a receiver
3. Receive a byte from a transmitter
4. Print a string
5. Print a byte as a number

I will primarily use this library to print to a console for debugging purposes, so these functions should suffice. I will add more when the need arises.

# Code


My code is broken down into myUART.c and myUART.h  

1. 
```c
// myUART.c
void initUART(int baud) {
	// Calculate UBBR
	uint16_t baud_setting = (F_CPU / 16 / baud) - 1;

	// Set BAUD
	UBRR0H = baud_setting >> 8;
	UBRR0L = baud_setting;

	// Set Transmitter/Receiver enable
	UCSR0B |= (1 << TXEN0) | (1 << RXEN0);
	
	// Set 8 data bits and 1 stop bit
	UCSR0C |= (1 << UCSZ00) | (1 << UCSZ01);
}
```
First, we set Baudrate using `F_CPU` and `baud` using the formula from the datasheet.
Macro F_CPU holds the clock frequency and needs to be defined for this to work.
Then we enable Transmitter/Receiver, and set data format. 


2. 
```c
void transmitByte(uint8_t ch) {
    while (!(UCSR0A & (1 << UDRE0)));

    UDR0 = ch;
}
```
Run a blocking loop until Data Registery Empty `UDRE0` flag bit is set. Then, write the byte to Data Register `UDR0`.

3. 
```c
uint8_t receiveByte(void) {
	while (!(UCSR0A & (1 << RXC0)));

	return UDR0;
}
```
Run a blocking loop until Receive complete `RXC0` bit is set. Then, return the byte from Data Register `UDR0`.

4. 
```c
void printString(const char myString[]) {
    for (int i = 0; myString[i]; i++)
    {
        transmitByte(myString[i]);
    }
}
```
Loop over the string and use `transmitByte()` to send one byte at a time.

5. 
```c
void printByte(uint8_t ch) {
	transmitByte('0' + (ch / 100));
	transmitByte('0' + ((ch / 10) % 10));
	transmitByte('0' + (ch % 10));
}
```

Transmit ASCII code of each decimal digit of the byte.
   
   
  
# Complete Code

```c
#include <avr/io.h>
#include "myUART.h"


// Initialization
void initUART(int baud) {
	// Calculate UBBR
	uint16_t baud_setting = (F_CPU / 16 / baud) - 1;

	// Set BAUD
	UBRR0H = baud_setting >> 8;
	UBRR0L = baud_setting;

	// Set Transmitter/Receiver enable
	UCSR0B |= (1 << TXEN0) | (1 << RXEN0);
	
	// Set 8 data bits and 1 stop bit
	UCSR0C |= (1 << UCSZ00) | (1 << UCSZ01);
}


// Transmitting functions
void transmitByte(uint8_t ch) {
    while (!(UCSR0A & (1 << UDRE0)));

    UDR0 = ch;
}


// Receiving functions
uint8_t receiveByte(void) {
	while (!(UCSR0A & (1 << RXC0)));

	return UDR0;
}


// Printing functions
void printString(const char myString[]) {
    for (int i = 0; myString[i]; i++)
    {
        transmitByte(myString[i]);
    }
}


void printByte(uint8_t ch) {
	transmitByte('0' + (ch / 100));
	transmitByte('0' + ((ch / 10) % 10));
	transmitByte('0' + (ch % 10));
}
```
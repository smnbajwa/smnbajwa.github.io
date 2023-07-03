---
layout: post
title: Breaking free from Arduino[Part 2] - Reading input and AVR library 
---

**Objective**  
* To read input from a pin triggered by a push button  
* Simplify code using `io.h`  

I am going to use an example Arduino sketch from the [booklet](https://www.eitkw.com/wp-content/uploads/2020/03/Arduino_Projects_Book.pdf) called 'Project 02'

```c
int switchState = 0;

void setup() {
  // put your setup code here, to run once:
  pinMode(2, INPUT);
  pinMode(3, OUTPUT);
  pinMode(4, OUTPUT);
  pinMode(5, OUTPUT);
}

void loop() {
  // put your main code here, to run repeatedly:
  switchState = digitalRead(2);

  if (switchState == LOW)
  {
    digitalWrite(3, HIGH);
    digitalWrite(4, LOW);
    digitalWrite(5, LOW);
  }

  else
  {
    digitalWrite(3, LOW);
    digitalWrite(4, LOW);
    digitalWrite(5, HIGH);

    delay(250);

    digitalWrite(4, HIGH);
    digitalWrite(5, LOW);
    delay(250);
  }
}
```
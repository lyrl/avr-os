# avr-os

[![Build Status](https://travis-ci.org/chrismoos/avr-os.png?branch=master)](https://travis-ci.org/chrismoos/avr-os)

avr-os is a library that provides a very basic rutime that enables your program to multitask.

The library uses pre-emptive multitasking to switch tasks and each task has its own stack that is restored when a task is resumed. An AVR timer is used to provide ticks and this interrupt is used to switch tasks.

## Adding library to Arduino

    git clone git://github.com/chrismoos/avr-os.git ~/Documents/Arduino/libraries/avros

## Building

You can create a static library for avr-os by issuing the following command:

    make DEVICE=arduino_uno

## Supported devices

* arduino_uno
* arduino_mega
* arduino_mega2560

## Arduino/AVR specific

### Choosing a timer

The following timers are supported by *avr-os* to use for multitasking:

* TIMER0 (8-bit)
* TIMER1 (16-bit)
* TIMER2 (8-bit)

When building use the **CONFIG_AVR_TIMER** flag. For example, to specify TIMER2 to be used:

    make CONFIG_AVR_TIMER=2 DEVICE=arduino_uno

*Note*: TIMER1 is the default timer if not specified.

## License

Copyright 2012 Chris Moos

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

## Sample sketch

This sketch has two tasks that update the LCD.

![example](https://raw.github.com/chrismoos/avr-os/master/example.png)

```cpp
#include <Arduino.h>
#include <LiquidCrystal.h>

#include <util/delay.h>

extern "C" {
    #include <os.h>
}

LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

spinlock_t testLock;

void kernel_task(void *arg) {
    while(1) {
        spinlock_acquire(&testLock);
        lcd.setCursor(0, 0);
        lcd.print("kernel: " + String((long)os_get_uptime()));
        spinlock_release(&testLock);
        os_sleep(1000);
    }
}

void user_task(void *arg) {
    int x = 0;
    while(1) {
        spinlock_acquire(&testLock);
        lcd.setCursor(0, 1);
        lcd.print("user_task: " + String(x++));
        spinlock_release(&testLock);
        os_sleep(5000);
    }
}

void setup() {
    os_init();
    lcd.begin(16, 2);
    lcd.print("Starting up...");
}

void loop() {
    spinlock_init(&testLock);

    os_schedule_task(kernel_task, NULL, 0);
    os_schedule_task(user_task, NULL, 0);
    lcd.clear();
    os_loop(); 
}
```

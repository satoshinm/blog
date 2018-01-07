# Constructing and experimenting with the STM32-O-Scope

by snm, January 6th, 2018

A brief follow-up to *[Building an Amazing $10 Oscilloscope with an STM32 blue pill, LCD touchscreen, and STM32-O-Scope software](https://satoshinm.github.io/blog/180105_stm32scope_building_an_amazing_10_oscilloscope_with_an_stm32_blue_pill_lcd_touchscreen_and_stm32-o-scope_software.html)*, where I built the scope on a breadboard. Time to try it out and make some improvements.

As a test, I'd like to observe a PS/2 mouse's kilohertz clock with an oscilloscope. However, there is a problem. PS/2 uses TTL voltage levels, that is, 5V. The STM32-O-Scope runs on 3.3V (besides the 5V input from USB). Wouldn't want to directly connect a 5V signal to the scope input. With a 1 MΩ resistor in series, I barely got a signal.

## Experiments with optical isolation

So I thought I'd try an optocoupler, as seen in *[The smart bulb / smart switch dilemma: smartening up a dumb wall switch](https://satoshinm.github.io/blog/171209_wallswitch_the_smart_bulb_smart_switch_dilemma_smartening_up_a_dumb_wall_switch.html)*. The same [Fairchild FOD817C 4-pin DIP phototransistor optocoupler](http://www.onsemi.com/PowerSolutions/product.do?id=FOD817), wired in the same way: 220 Ω current-limiting resistor on the LED input, and 10 kΩ pull-up resistor to 3.3V on the transistor output. Both cathode and emitter are wired to common ground. Connect the PB1 test signal to the LED input in series with the current-limiting resistor, and the PB0 analog oscilloscope input on the transistor collector. Also hooked up a 3-wire voltmeter to the input, powered by 5V. The wiring is getting messy:

![Optocoupler oscilloscope wiring](https://user-images.githubusercontent.com/26856618/34645089-a4d09da8-f2f9-11e7-840d-1cffc770fe13.png)

but it works. The downside is signal distortion, this was a square wave before it passed through the optocoupler:

![Square wave through optocoupler](https://user-images.githubusercontent.com/26856618/34645096-cae93694-f2f9-11e7-8a39-b91d41635a5e.png)

could probably be improved (note the breadboard I prototyped it on has some capacitance), this particular optocoupler in its data sheet says it is ideal for "digital logic inputs". The 500 Hz square wave signal is passable, but when I tried to measure the 5V PS/2 clock, there was a nearly fixed 0.05V voltage, and the input voltage dropped considerably. 

## Level shifter for 5V to 3.3V

So instead I tried using a 5V to 3.3V level shifter on the clock. When connected through 1 MΩ to the scope input, something is visible, either signal or noise, but when connected directly, the scope display stops. 

What do we expect? 1/(10 kHz) = 100 µs, 1/(16.7 kHz) = 59.9 µs. This is plenty slow for the 0.58 µS/sample theoretical limit of the STM32-O-Scope. 0.58 µs * 6144 samples = 3563 µs. /100 µs = 36 clock ticks per screen, /59.9 µs = 59.5 clock ticks per screen. That's a lot, is this the actual PS/2 clock signal?

![PS/2 clock through scope](https://user-images.githubusercontent.com/26856618/34645925-08f3ad88-f30f-11e7-8f45-7353cb673f00.png)

Not really clear, but it's time to move on. Tore up the optocoupler and level shifter circuits, try something else.

## Moving the circuit off the breadboard

To free up the breadboard and jumpers for other projects, I'd like to move the oscilloscope circuit into something more permanent. We could mount the microcontroler module and display module into a perforated circuit board, then solder wires between them. I selected this board, which seemed to fit, oriented something like this (shown with two blue pills as an example):

![Showing off the perfboard](https://user-images.githubusercontent.com/26856618/34646794-81d279de-f326-11e7-8bfd-8a8ab54dada6.png)

but having the USB cable on the top may get in the way. This is more like it:

![Reoriented perfboard](https://user-images.githubusercontent.com/26856618/34646800-adf1e86a-f326-11e7-8e1a-f40376866db1.png)

The blue pill will be oriented horizontally and the display module's pins oriented vertically, as so:

![Perfboard populated](https://user-images.githubusercontent.com/26856618/34646815-071236a2-f327-11e7-9639-2428e22529df.png)

The perfboard is conveniently numbered and lettered. Due to how they are positioned, the blue pill's pins will be numbered and the display module's lettered (using the letters on the *back* of the board, not the front, since they differ for some reason). Then wiring is simply a matter of matching up the numbers to letters. The blue pill, first row:

| # | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 |
| - | - | - | - | - | - | - | - | - | - | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
| From | - | G | 3.3 | - | - | - | B1 | B0 | A7 | A6 | A5 | - | A3 | A2 | A1 | A0 | - | - | - | - |
| To | - | GND | VCC | - | - | - | (test) | (probe) | SDI | SDO | SCK | LED | RESET | CS | D/C | - | - | - | - |

second row (only showing 5 of the 20 pins, as that's all I've used):

| # | 1 | 2 | 3 | 5 | 
| - | - | - | - | - |
| From | B12 | B13 | B14 | B15 | A8 |
| To | T_CLK | T_CS | T_DIN | T_DO | T_IRQ |

The display module connections, oriented vertically, using the letters from the back, connecting as "to" above:

| # | Signal |
| - | ------ |
| N | VCC |
| M | GND |
| L | CS |
| K | RESET |
| J | D/C |
| I | SDI(MOSI |
| H | SCK |
| G | LED |
| F | SDO(MISO) |
| E | T_CLK |
| D | T_CS |
| C | T_DIN |
| B | T_DO |
| A | T_IRQ |

If you want to critique my soldering job, here it is (note this is before cleaning off flux):

![Soldered reverse](https://user-images.githubusercontent.com/26856618/34646889-7e5c6d8a-f328-11e7-9541-82be6eb14e5c.png)

Had some difficulties wiring, be sure to double-check the connections! But after fixing mistakingly swapping all of the display wires, and then power and ground, fortunately nothing was permanently damaged, the oscilloscope continued to operate properly as it did on the breadboard. I paired it with a 1A USB wall adapter, now we have a standalone oscilloscope unit for further experimentation with other circuits. Here it is demonstrating the 500 Hz square wave test signal:

![Final scope](https://user-images.githubusercontent.com/26856618/34646907-ebdcddfe-f328-11e7-840a-562a7a310c90.png)

## Conclusions

This oscilloscope isn't perfect, in fact there are a lot of problems with it (besides the inherent limitations). Even though it is currently unused/unsupported, the display module has an SD card slot on the bottom, but the way I mounted it onto the perfboard it is inaccessible. It would be better flipped upside-down so one could insert an SD card. There is also no 1 MΩ attentuator onboard (and no optocoupler or level shifters either, both I only experimented with and decided against; or the voltmeter, which would need to be mounted somewhere), although it does work well with the built-in test signal. Enclosing the boards in a case would be nice, too (pingumacpenguin, the creator of STM32-O-Scope, used a toy pig so he called it a pigscope). But I've already spent enough time on this project today, any further improvements will have to wait until later. For now, this is now a decently usable standalone low-bandwidth oscilloscope.
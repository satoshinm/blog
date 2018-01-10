# AT&T 1739 Digital Answering Machine teardown and resuscitation attempt

by snm, January 9th, 2018

* auto-gen TOC:
{:toc}

Would you buy this for a dollar? Apparently no one did, so this Digital Answering System by at&t ended up like this:

![Dirty case](https://user-images.githubusercontent.com/26856618/34647897-2b55b1ac-f344-11e7-8245-befb2bc78661.png)

The front case consists of a grill for the speaker, and tactile buttons for play/stop, setup, clock, volume up/down, delete, on/off, memo/repeat, annc/skip, and the number 1739. The reverse side of the case is even filthier, can't make out the compliance notices, can barely read the instructions "To set the clock: press and hold [CLOCK] until you hear the day of the week. To change day/hour/minutes and year: use [MEMO/REPEAT] and [ANNC/SKIP]. Press [CLOCK] after each is set:"

![Back of case](https://user-images.githubusercontent.com/26856618/34647909-79aae4da-f344-11e7-90f3-d1332c8f5708.png)

Model 1739, SN E5??3804887. I found the manual here: [AT&T 1739 User Manual](https://www.manualslib.com/manual/10392/AtAndt-1739.html). Yes, that's it:

![AT&T 1739 manual intro](https://user-images.githubusercontent.com/26856618/34647948-376c9a18-f345-11e7-968c-f654dae4517a.png)

This is how you're supposed to hook it up:

![Answering machine hookup instructions](https://user-images.githubusercontent.com/26856618/34647958-5d25100a-f345-11e7-8885-b1bc6a11a407.png)

The power input is 6VAC, 350 mA...crucially, AC not DC. Didn't get a wall adapter with it. Assuming 120VAC input, 6VAC would be a 20:1 step-down. Absent a compatible power supply, I decided to take it apart.

## Opening it up

Ripping out its guts, the circuitry is in pristine shape, betraying its outside condition:

![Circuit board front](https://user-images.githubusercontent.com/26856618/34647993-0c190a58-f346-11e7-86bf-fab9a4c5bffb.png)

The silkscreen reads KR_DS_DS-1107A and 35-007381-001-100 >PF-LP<, major visible components include the speaker (8 Ω, 0.4 W), microphone, modular telephone jack, barrel-style power connector, two large electrolytic capacitors (both 4700 µF 16V), 3.579545/0B crystal, a header in the corner labeled "T&R" for tip and ring (it connected to a modular telephone connector plug) and a moderate number of other discrete components. Flipping it over, we see a 7-segment LED display, a couple of chips, and an epoxyed "glob top" semiconductor module where the magic happens:

![Circuit board reverse](https://user-images.githubusercontent.com/26856618/34648050-ff711358-f346-11e7-919b-831dc74d7811.png)

Undoubtedly a proprietary application-specific integrated circuit. Taking a closer look, we see the silkscreen text "BS COB(MASK)" on the printed circuit board on top of the module, and "35-006657-001-100" beneath it:

![Glob top closer](https://user-images.githubusercontent.com/26856618/34648061-4041924a-f347-11e7-8941-be747bf872fc.png)

Moving in closer, on the epoxy itself, "30664-OP-100 B98/C B43/D/C/A-10 06/05/10":

![Glob top closeup](https://user-images.githubusercontent.com/26856618/34648070-742acaae-f347-11e7-97a4-5ef6a9d749d9.png)

All I can make out from this label is the date code, appears to be June 5th, 2010. This seems about right, despite being part of the aging [Plain old telephone service](https://en.wikipedia.org/wiki/Plain_old_telephone_service) (POTS) system, this is a relatively modern device. Unlike [answering machines](https://en.wikipedia.org/wiki/Answering_machine) of yore, there is no tape. Where are the messages stored? I'd wager in flash memory (or even RAM?), in the glob top. Found it on Amazon: [AT&T 1739 Corded Digital Answering System, White](https://www.amazon.com/AT-1739-Corded-Digital-Answering/dp/B000VWJ210), for sale for $9.60, and reviews as latest as two years ago:

> 	Mochni (Talking Bird) 2.0 out of 5 stars

> Handy, up to a point, but not as advertised.

> June 6, 2016 Verified Purchase

> OK, except it loses all messages when power goes out, despite the product description. I have never had a memory-featured item "wear out" like this one.

Presumably superseded by the 1740 model, sold on [telephones.att.com](https://telephones.att.com/) for $17.95: [Digital Answering System with Time/Day Stamp](https://telephones.att.com/pd/1887/1740-White-Digital-Answering-System-with-Time-Day-Stamp), but it doesn't look much different.

## Powering it up

How can we get 6VAC? Well, I got a line transformer which ouptuts 21VAC and 10.5VAC from the *[Kinyo 1-way VHS Rewinder UV-428 teardown](https://satoshinm.github.io/blog/180105_vhsrewind_a_blast_from_the_past_kinyo_1_way_vhs_rewinder_uv_428_teardown.html)*. Stepping down these to 6VAC could be done with a 3.5:1 or 1.75:1 ratio transformer. It's a start, I'll use the ~6:1 line transformer first, to work with a safer voltage, then try to step it down further. Desolder the DC motor which was wired to the rectifier board's output, leaving us with the AC wall plug connected to the transformer (red wires), the stepped-down AC output and rectifier board:

![Line transformer and rectifier](https://user-images.githubusercontent.com/26856618/34760290-10153cd0-f594-11e7-8778-eda03a4fe5b8.png)

### Adding a power indicator LED to the Kinyo supply

It would be handy to have an indicator when this power supply is energized, so I soldered a yellow LED to the DC output, in series wiht a current-limiting resistor. At about 13 VDC output, minus about 1.8 V drop across the LED, divided by 20 milliamperes maximum, computes to a 560 Ω resistor. Picked the next closest, a 680 Ω, which will limit the current slightly more. Soldered it up, minding the LED polarity, now we have a simple power indicator to see when it is on:

![Adding power LED to rectifier board](https://user-images.githubusercontent.com/26856618/34760373-7bf2d85e-f594-11e7-9d45-1ecf481521c4.png)

The capacitor stores charge, so the LED slowly dims when the power is removed, takes about a second. Note, I used a 1/4 watt resistor, but this is probably too low, the resistor becomes noticeably warm. Use a higher-wattage resistor if the circuit needs to be powered for extended periods of time, or better yet, step down or buck the voltage to a more appropriate level. But for now, this'll do.

### Stepping down to 6 VAC with a variac

Why does this answering machine need AC, anyways? It feeds the input voltage, across a [spark gap](https://en.wikipedia.org/wiki/Spark_gap), directly into a 4-diode [full-wave bridge rectifier](https://en.wikipedia.org/wiki/Diode_bridge). Much of the circuit is DC, but it may still use AC for some purpose.

Searching found a post, [AC output plugpack design needed](http://www.eevblog.com/forum/projects/ac-output-plugpack-design-needed/msg1387466/#msg1387466), someone in a similar situation needing to replace a 6VAC power adapter (but for a sentimental lamp, not a junk answering machine), and this transformer was suggested: [M7252 • 10VA 6+6V PCB Transformer](http://www.altronics.com.au/p/m7252-powertran-10va-6+6v-pcb-transformer/). At $16.50 I'll pass, but it looks like it would work.

[Is it possible to lower AC voltage without a transformer?](https://forum.allaboutcircuits.com/threads/how-to-lower-ac-voltage.28367/). Wendy replied "Sure, you can use a low ohmage resistor, a coil, or a capacitor in series, depending on the load." Triac dimmers are another option. [Another thread](https://forum.allaboutcircuits.com/threads/how-to-lower-the-ac-voltage-without-using-a-transformer.70385/) suggests using a resistive [voltage divider](https://en.wikipedia.org/wiki/Voltage_divider). But what would be most immediately testable is a transformer where we could adjust the ratio to get the right voltage, this exists and is known as a [variac](https://en.wikipedia.org/wiki/Autotransformer#Variable_autotransformers) or variable autotransformer. 

Wire up the 21 VAC output across the coil of the variac, then take the stepped-down voltage one side of the coil, as well as the knob, wiring this output to the answering machine's barrel connector power input. The complete contraption:

![Rigging a 6VAC PSU with the variac](https://user-images.githubusercontent.com/26856618/34760596-556dd714-f595-11e7-8853-12d623d9fcc3.png)

The moment of truth...

### Success, then failure

When I first powered it up, a click was heard from the speaker the "answering" LED started blinking! It seemed to be working!

But then I powered it off, and back on, and only the speaker clicking remained. No more blinking answering LED. Instead, the 7-segment display briefly turned on, then slowly off. Best picture I could capture of this phenomena:

![Powered on, sorta](https://user-images.githubusercontent.com/26856618/34760711-c486c174-f595-11e7-8891-dbf3e7cd463f.png)

What happened? My best guess is we just witnessed the downside of unregulated power supplies. When the makeshift 6 VAC power supply is plugged in, the output voltage, as measured by multimeter, spikes to overload, before the scale adjusts to the expected value. [Inrush current](https://en.wikipedia.org/wiki/Inrush_current)? Furthermore, there are no regulators anywhere in this supply. The voltage comes in from the mains, then is stepped down from a fixed ratio, and then from a variable ratio, but there is no circuitry to ensure the voltage never exceeds a maximum threshold or drops below a minimum. Whatever comes out of the wall, divided down, is what the answering machine circuitry will get.

Tried fiddling with the variac adjustment, tried turning it off and on again, but try as I might, nothing seemed to resucitate this answering machine. Was a component fried? The fuse, at least, was still good. Couldn't get it to work. Oh well, had to give up. At least it briefly may have worked. Next time, maybe make a regulated AC power supply.

Since I considered it dead, next I figured I'd increase the voltage and see what happens. Seeing as it appears to briefly turn "on" when first plugged in, assuming a voltage spike. At 21 VAC, the speaker makes a constant clicking noise, and the 7-segment display illuminates all the segments. No blinking answering LED, in any case. Not sure what is going on here, but it is time to write it off.

## Tearing to pieces

All that is left now to do is pick away at the parts, taking what can possibly be useful elsewhere. Not much to this board, but here is what I desoldered, at least for the time being:

![Salvaged parts](https://user-images.githubusercontent.com/26856618/34760984-d20c3b84-f596-11e7-9342-3fe1ad3c6a6b.png)

* 2 x 4700 µF 16V electrolytic capacitors, and one 1000 µF
* Speaker, 8Ω 1/4W
* Microphone
* Barrel jack connector
* Modular telephone connector
* Fuse
* LED, red, clear
* 7-segment LED display
* Crystal, "3.579545/0B"

The LEDs still work (both the 7-segment and singular LED), the fuse seems still good not measuring an open, the speaker measures about 8Ω, so all the parts I tested seemed fine, except I couldn't get the crystal to oscillate for unknown reasons. 
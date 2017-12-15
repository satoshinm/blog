# Assembling a HX3208 DIY FM radio kit

by snm, December 14th, 2017

Bought a DIY FM radio kit on Aliexpress for $2.51: [FM Micro SMD Radio DIY Kits FM Frequency Modulation Radio Electronic Production Training Suite](https://www.aliexpress.com/item/FM-Micro-SMD-Radio-DIY-Kits-FM-Frequency-Modulation-Radio-Electronic-Production-Training-Suite/32752016113.html). Labeled in Chinese and with the model number HX3208, this is an inexpensive yet fun kit to make, herein a chronological my build progress from start to finish.

## Unboxing

The kit arrived in an envelope containing plastic bags:

![Wrapped bags](https://user-images.githubusercontent.com/26856618/34026260-92ea1530-e109-11e7-9e68-9bcdfe69cbc2.png)

Break it open, there is the plastic case, a microphone, and another bag of all the components:

![Three pieces](https://user-images.githubusercontent.com/26856618/34026293-ca9e0c48-e109-11e7-8eca-a2ffa308a304.png)

Opening up the component bag and carefully sorting each piece, we see what is ahead of us:

![All components](https://user-images.githubusercontent.com/26856618/34026305-e6a5dcfe-e109-11e7-94f9-9eadaa34a873.png)

On the left are capacitors of various values (labeled with a sticker on the back), next to resistors, then two transistors, and miscellaneous other components we'll see in depth soon.

## Construction

Instructions are not included in the package, but they can be found in the [seller feedback](https://www.aliexpress.com/item/FM-Micro-SMD-Radio-DIY-Kits-FM-Frequency-Modulation-Radio-Electronic-Production-Training-Suite/32752016113.html). I used these images as a guide:

![Schematic](https://user-images.githubusercontent.com/26856618/33975915-a21ba6ea-e046-11e7-8ce9-ce0a9a174485.jpg)

![Another schematic](https://user-images.githubusercontent.com/26856618/33975916-a22e1988-e046-11e7-9f59-9b50dce25924.png)

![PCB layout](https://user-images.githubusercontent.com/26856618/33975917-a2412410-e046-11e7-84f3-cba7d717993d.png)

One annoyance is this guide has the text oriented upside-down from the silkscreen on the actual printed circuit board. Rotate the board using the potentiometer hole to orient, and read it upside-down.

The general principle when assembling these kits is to start with the smallest components first, so I started with the capacitors.

### Capacitors

![First capacitor](https://user-images.githubusercontent.com/26856618/34026355-48e42ace-e10a-11e7-9228-ea8523aa0523.png)

The capacitors are ([0603](https://en.wikipedia.org/wiki/Surface-mount_technology#Packages)?) surface-mount, with no identifying markers. To help pick the right ones, they come in a tape with a code on the back (although the color may correlate to the capacitance, the code should be used to identify the components). Match up the designator with the capacitance on the schematic. If you haven't dealt with SMD before, [EEVblog #997 - How To Solder Surface Mount Components](https://www.youtube.com/watch?v=hoLf8gvvXXU) is a great introduction. All the caps finished:

![All capacitors](https://user-images.githubusercontent.com/26856618/34026484-47c44e48-e10b-11e7-861d-054fe636a830.png)

Note that I haven't cleaned off the solder flux yet, which is why it looks ugly. Will clean it after soldering the other components. There are three through-hole capacitors, but will be done after all the surface-mount on this side:

![Soldered SMD caps plus waste and thru-hole caps](https://user-images.githubusercontent.com/26856618/34026513-769dcbd6-e10b-11e7-90e3-969d0820a5d9.png)

### Resistors

![First resistor](https://user-images.githubusercontent.com/26856618/34026562-d837e34a-e10b-11e7-9c28-1fb8acadae25.png)

There are only a few resistors, but they are labeled unlike the caps. Except the label doesn't always match the schematic. It does for R2, where 154 (150 kΩ) is both on the component (shown above) and the schematic. However R1 is "15K" on the schematic, which means 15 kΩ. See [Resistor SMD code](http://www.resistorguide.com/resistor-smd-code/) for more information, but for all the surface-mount resistors in this kit, there are three digits, the last being the exponent, so 153 is 15 x 10^3 ohms = 15 k. Use the "153" resistor in R1. All of the resistors soldered:

![All resistors](https://user-images.githubusercontent.com/26856618/34026745-f144cea6-e10c-11e7-9d9f-b9bde31f4b25.png)

### SC1088 radio IC

There is one SOIC integrated circuit in this kit, a SC1088. Align it based on the dot and notch then solder each pin:

![Soldered IC](https://user-images.githubusercontent.com/26856618/34026763-1210392c-e10d-11e7-99f0-72865c40f8c8.png)

What is the SC1088? Petervis explains in [SC1088 FM Radio Chip](https://www.petervis.com/electronics%20guides/SC1088/SC1088.html):

> The SC1088 is an AM / FM radio IC manufactured by Silan Semiconductors, a subsidiary of the Hangzhou based Silan Microelectronics Corporation. This single chip radio IC is very similar to the Philips TDA7088T, and usually found in portable radios. The IC contains all the necessary building blocks for building a radio, and contains a frequency locked loop (FLL), intermediate frequency (IF) blocks centred at 70 kHz, and all-pass demodulator blocks. Operating from as little as 1.8 V at 4.2 mA, this IC requires very little current making it an ideal solution for portable radio applications.

### Transistors

![Transistors schematic](https://user-images.githubusercontent.com/26856618/34026869-cc7d2784-e10d-11e7-8e58-7ab34fb610df.png)

V3 is a 9013 NPN transistor, V4 is a 9012 PNP, per the schematic. But now we have a problem, how to tell which is the 9013 and which is the 9012? Lookup the [SMD code](https://alltransistors.com/smd-search.php?search=j6), J6 is the 9014 NPN. Find the [data sheet](http://www.datasheetcafe.com/wp-content/uploads/2016/01/S9014-datasheet.pdf):

![J6 is NPN](https://user-images.githubusercontent.com/26856618/34026946-6e7ddd08-e10e-11e7-9d15-3be845a2d364.png)

it confirms "Markings: J6". This means V3 is the J6 transistor, V4 is the other labeled 2T1. Go ahead and solder:

![Transistors soldered](https://user-images.githubusercontent.com/26856618/34026845-9ce73604-e10d-11e7-99f3-a3e99f1465c6.png)

The board so far:

![Progress](https://user-images.githubusercontent.com/26856618/34027004-b3de3db6-e10e-11e7-87f2-13fdb16fce89.png)

and the tape used up:

![Trash progress](https://user-images.githubusercontent.com/26856618/34027014-d05b1e0a-e10e-11e7-818a-99b491bafabb.png)

### Electrolytic and through-hole ceramic capacitors

Flip the board over, begin to solder the through-hole components. C18 is 100 µF, but as a polarized capacitor, the polarity is crucial. There is a subtle "+" sign in the diagram, use it to the orient the device properly (the negative side is marked). The positive side should be be facing to the edge of the board. Using a [helping hands](https://www.amazon.com/Neiko-01902-Adjustable-Magnifying-Alligator/dp/B000P42O3C) is helpful here to hold up the board:

![Helping hands electrolytic cap](https://user-images.githubusercontent.com/26856618/34027098-51334818-e10f-11e7-8d5c-d49dfaad5da6.png)

I soldered the two ceramics too:

![Ceramic caps](https://user-images.githubusercontent.com/26856618/34027118-71820456-e10f-11e7-8dc7-15b4a4016648.png)

but all of these three caps are assembled wrongly. There is not enough vertical clearance to fit in the case, I later discovered. Usually caps sit straight up, but the electrolytic needs to lie on its side, with the leads bending at 90º. The ceramic capacitor leads need to be long enough to reach but bent around to be compact as possible.

### Variable capacitance diode

Now this is an interesting component. Labeled V1 in the schematic, with a diode symbol adjacent to a capacitor symbol. Comes in a what looks like a [TO-92](https://en.wikipedia.org/wiki/TO-92) package, but the middle lead is cut off. Labeled "BB910 SRS". Searching for the [data sheet](http://www.alldatasheet.com/datasheet-pdf/pdf/16081/PHILIPS/BB910.html) reveals it is a "VHF variable capacitance diode". Here's an excerpt of the data sheet, except for a different package:

![BB910 data](https://user-images.githubusercontent.com/26856618/34027245-5fb09c96-e110-11e7-8f05-e838c9a0d842.png)

> The BB910 is a variable capacitance diode, fabricated in planar technology, and encapsulated in the hermetically sealed leaded glass SOD68 (DO-34) package.

Wikipedia calls it a [varicap](https://en.wikipedia.org/wiki/Varicap):

> In electronics, a varicap diode, varactor diode, variable capacitance diode, variable reactance diode or tuning diode is a type of diode designed to exploit the voltage-dependent capacitance of a reversed-biased p–n junction.

Solder it:

![BB910 SRS soldered](https://user-images.githubusercontent.com/26856618/34027177-ce4cadc6-e10f-11e7-87b6-c358fc76addd.png)

### Pushbuttons, LED, audio jack

Fleshing out the front of the board:

![Pushbuttons, LED, audio jack](https://user-images.githubusercontent.com/26856618/34027345-df9a17d4-e110-11e7-820c-38d38fc59368.png)

Make sure the LED polarity is correct. Couldn't get the pushbuttons aligned up too straight, but it is workable. Also solder on the battery wires, + to red and - to blue.

## Potentiometer

The potentiometer is used for volume control (not tuning). Stick it through the hole:

![Pot unaligned](https://user-images.githubusercontent.com/26856618/34027420-696559b0-e111-11e7-9d10-52d122225f0e.png)

The tabs don't line up with the board holes too well, but we'll manage. Do your best to align it, and solder:

![Pot soldered](https://user-images.githubusercontent.com/26856618/34027394-3f8ffdca-e111-11e7-9f61-d56804e3698f.png)

On the right the device leans off the board a bit, so I bent it inward. 

### Inductors

Another problem. The inductors L1, L2, L3, and L4 on the schematic are labeled something in Chinese, which I can't read, and there are no markings on the inductors themselves:

![Mysterious inductors](https://user-images.githubusercontent.com/26856618/34027461-bd65a6f0-e111-11e7-9828-9bc6206cf6ee.png)

How to tell them apart? This is where a fourth instructional guide image became essential, found on the Aliexpress comments:

![Assembled example](https://user-images.githubusercontent.com/26856618/33975914-a2069fb6-e046-11e7-88d9-36babfbf79d4.jpg)

Follow their example. Incidentally, this is where I learned I needed to lie the caps on their side, so I fixed that too. Solder the inductors with the small and large coiled wires, radial package (looks like a resistor, but it's green. Also solder the through-hole resistor R5)), and the inductor with magnet wire around a ferrous choke. It's almost done:

![Inductors soldered and more](https://user-images.githubusercontent.com/26856618/34027524-128f034c-e112-11e7-80de-9834d36cfd96.png)

## Jumpers

There are two connections which need to be made using jumper wire. I used left over component leads, and the assembly picture as reference, J1 and J2:

![Jumper wires](https://user-images.githubusercontent.com/26856618/34027524-128f034c-e112-11e7-80de-9834d36cfd96.png)


## Cleaning

With the soldering finished (almost), time to clean off the flux. Before:

![Before cleaning](https://user-images.githubusercontent.com/26856618/34027602-95a643e4-e112-11e7-9f24-3f50712d8899.png)

Spray with isopropyl alcohol, rub with a toothbrush, then pat dry. After cleaning it's a little better:

![IPA cleaned board](https://user-images.githubusercontent.com/26856618/34027618-b7ad0bee-e112-11e7-860f-504e40c56def.png)

## Battery case assembly

Pop the soldered board into the plastic case:

![Board in case](https://user-images.githubusercontent.com/26856618/34027643-e91bd9d0-e112-11e7-9d5f-92445343739f.png)

Insert the battery springs and tabs, then solder the positive and negative wires. Plug in two AAA batteries, and the provided headphones:

![Battery powered](https://user-images.githubusercontent.com/26856618/34027674-1d23a744-e113-11e7-9c9c-a832b8662c45.png)

The power LED lights up! 

![Powered on front](https://user-images.githubusercontent.com/26856618/34027715-5828673a-e113-11e7-9a64-dc12b38733a2.png)

But there is a problem... no audio. What could be wrong?

## Debugging the circuit

Double-check the wiring, poke around while the circuit is live, and I found the problem: a poor connection with the ferrous ferrous core inductor. It is wrapped in [magnet wire](https://en.wikipedia.org/wiki/Magnet_wire) aka "enameled wire", a very thin wire coated with an insulator, which you have to carefully thread through the circuit board hole and solder. But I soldered to the insulated coating:

![Badly soldered inductor](https://user-images.githubusercontent.com/26856618/34027761-a7c6310a-e113-11e7-82e4-69945dcf767a.png)

To make a proper connection, unless the insulation is stripped, the wire has to be soldered on the exposed tip. After making this fix, the FM radio audio comes through!

## Screws

The two identical screws are used to affix the circuit board to the case. But not from the top, this is wrong:

![Wrong](https://user-images.githubusercontent.com/26856618/34027891-9abcf59c-e114-11e7-890f-6022f09a24dd.png)

They are screwed in from the back:

![Board case screws](https://user-images.githubusercontent.com/26856618/34027907-b4a3e2b8-e114-11e7-99f0-5c9c3c63e4ad.png)

The smaller fine-threaded screw is used to hold the potentiometer knob, something like this:

![Pot screw rotated](https://user-images.githubusercontent.com/26856618/34027932-dd91639e-e114-11e7-8753-3420c17f6484.png)

but that's rotated 180 degrees. Insert the knob holder so that the notch points towards the power LED when turned off, like this:

![Pot screw correct](https://user-images.githubusercontent.com/26856618/34027960-fb4ad9b0-e114-11e7-96e5-8787844ca357.png)

Now you can attach the knob, it pushes on securely:

![Pot knob](https://user-images.githubusercontent.com/26856618/34027977-20252bb4-e115-11e7-8e4b-28608bbc076d.png)

There is one extra screw I haven't found the use for:

![Extra screw](https://user-images.githubusercontent.com/26856618/34028003-378f4da2-e115-11e7-8fd0-693976375e8b.png)

## Operation

This completes the circuit construction. To turn off the radio, turn the potentiometer far to the left, to turn it on or louder turn it to the right. The radio is surprisingly loud using fresh batteries, I had to turn it almost all the way down to not go deaf. Press "SCAN" to change the channel or RESET. FM channels come in clearly when the radio is in my pocket and the headphones are in my ears. 

Overall I'm quite satisfied with this kit. For less than $3 you can practice surface-mount soldering and get a decent working portable FM radio out of it. I'd recommend it to anyone interested in a moderate circuit building challenge or receiving FM radio.
# Custom laptop upgrades: internal breadboard, power indicator, and larger case

by snm, December 2nd, 2017

Have you ever wanted to make your own laptop? I did, and this was my first attempt: *[Building a small custom Raspberry Pi Zero laptop in a cardboard box](https://satoshinm.github.io/blog/171112_building_a_small_custom_raspberry_pi_zero_laptop_in_a_cardboard_box.html)*. This is a continuation of that article, making several minor improvements including moving it to a larger case, adding a breadboard for prototyping, and an improved power indicator LED on the front panel from the USB hub. Codename: ztop2, for Raspberry Pi **Z**ero lap**top**, generation **2**.

## Breadboard

Having soldered-on jumper wires on the OLED display module, supported only by sticking the pins through cardboard, was a sore point in the old laptop design. A 20-pin female adapter could plug into the display module, but I didn't have one on hand, and would like to improve the expandability of this device as well, so I opted to plug it into an 830-tie breadboard. The springs in the breadboard holes provide structural support:

![OLED display module in breadboard](https://user-images.githubusercontent.com/26856618/33519006-fbb84156-d752-11e7-9d5f-0ae4267dae0a.png)

Now the jumper wires can be desoldered and plugged into the breadboard. But you may recall one of the wires pins was broken (second from the right), and I was low on jumper wires anyways, so I ordered these replacements: [Dupont line 120pcs 10cm male to male + male to female +female to female jumper wire Dupont cable for arduino](https://www.aliexpress.com/item/Dupont-line-120pcs-10cm-male-to-male-male-to-female-female-to-female-jumper-wire-Dupont/32316585867.html). 40 each of jumpers with female-female, male-female, and male-male connectors:

![New jumper wires](https://user-images.githubusercontent.com/26856618/33519473-1cef1cb0-d75c-11e7-8e38-584039404fcd.png)

One disadvantage is they are a bit shorter (10 cm) than the others I have, but it'll do for prototyping (brown wire on right):

![OLED module plugged into breadboard](https://user-images.githubusercontent.com/26856618/33519490-7ee3076a-d75c-11e7-9ccd-c79c4b152232.png)

## GPIO cable

Even though these jumpers wire get the job done, they aren't as elegant as I'd like. To remedy this, ordered a GPIO cable and breakout board: $2.67 from Aliexpress [Raspberry Pi 3 40 Pin Extension Board Adapter +40 Pin GPIO GPIO Cable Line For Banana Pi M3 /Raspberry Pi 2/ For Orange Pi PC](https://www.aliexpress.com/item/Orange-Pi-40-Pin-GPIO-Cable-Line-Adapter-40-Pin-GPIO-Extension-Board-For-Banana-Pi/32618788035.html)

There are other alternatives, Adafruit sells what they call a [Pi T-Cobbler](https://www.adafruit.com/product/20280) for $7.95 and Sparkfun has a [Pi Wedge](https://www.sparkfun.com/products/13717) with a serial connector on the side, for $9.95. The board I got from Aliexpress is labeled "PI GPIO-T Plus V2.4".

For the wiring, see *[Monochrome 2.7” and 2.42” 128x64 OLED displays on a Raspberry Pi Zero](https://satoshinm.github.io/blog/171110monochrome_2.7_and_2.42_128x64_oled_displays_on_a_raspberry_pi_zero.html)*, copying here with the jumper wire colors to assist in my rewiring:

| OLED Module Pin | Pi Zero Pin | GPIO-T | Jumper wires| Remarks |
| ------------- | ----------- | ------- |
| 1 GND         | P01-20 GND | GND | black/black | GND, ground |
| 2 +3V         | P01-17 3V3 | 3.3V | brown/white | 3.3V, power |
| 4 DC          | P01-18 GPIO 24 | P24 | red/green | data/command, the fourth wire of "4-wire SPI" (sic) | 
| 7 SCLK        | P01-23 GPIO 11 (SCLK) | P11(SCLK) | yellow/yellow | serial clock
| 8 DIN         | P01-19 GPIO 10 (MOSI) | P10(MOSI) | blue/blue | MOSI, master out slave in (MISO is not connected, no slave output) |
| 15 /CS        | P01-24 GPIO 8 (CE0) | (CE0)P08 | yellow/white | Chip enable/select 0 |
| 16 /RES       | P01-22 GPIO 25 | P25 | n bgreen/green | reset |

The socket on the GPIO-T board is keyed, but how do we know what way to plug in the ribbon cable into the Pi? I guessed the wrong way and the Pi drew excessive current, but reversed it and it worked: the ribbon cable is supposed to be plugged in facing outwards, the key *towards* the inside of the board. 

Wire up with jumper wires again:

![OLED to GPIO-T](https://user-images.githubusercontent.com/26856618/33520610-4fc78b38-d773-11e7-9af1-4021be3891b6.png)

Then using breadboard wires, tidying it all up:

![OLED to GPIO-T breadboard](https://user-images.githubusercontent.com/26856618/33520614-5d7e725a-d773-11e7-848d-8441c39dcb79.png)

## USB power meter

To monitor the power usage, bought a "USB safety tester": [Digital Display 3V-30V USB Tester Current Voltage Charger Capacity Doctor qc2.0/3.0 quick charge power bank meter voltmeter](https://www.aliexpress.com/item/Digital-Dispay-3V-30V-Mini-Current-Voltage-Charger-Capacity-Tester-USB-Doctor-qc2-0-3-0/32686482158.html). Similar to a charge doctor. It can be plugged in inline, monitoring the power draw from the battery:

![USB power](https://user-images.githubusercontent.com/26856618/33520971-ec8c1866-d779-11e7-95ed-08d065b6feea.png)

When idle but fully powered up, the laptop draws about 500 - 600 mA at 5 volts.

## Bigger box

The small cardboard box was just enough to fit the keyboard and other components, but very cramped, and cannot comfortably fit the 840-tie breadboard. To allow room for expansion, switched to another cardboard shipping Sparkfun box, but substantially higher:

|     | Old | New
| --- | --- | ---
| Height | 1 3/4" | 4 1/4"
| Width | 6 1/4" | 9 3/4"
| Depth | 5" | 6"

![New box](https://user-images.githubusercontent.com/26856618/33518983-b321645e-d752-11e7-830a-3d0a29639444.png)

Check out how roomy it is inside:

![New box inside](https://user-images.githubusercontent.com/26856618/33518994-d1c0a7e4-d752-11e7-8ee2-31a666cd3ea3.png)

Shoved in all the components:

![Components in box](https://user-images.githubusercontent.com/26856618/33520977-12a6da54-d77a-11e7-9483-b8c155015c63.png)

Now to build it.

## Construction

First some tweaks so it fits better.

### Power input: USB to GPIO

The micro USB plug on the "PWR IN" connector is unnecessarily bulky. But it is actually not needed, the Pi can be powered through the [5 volt and ground pins on the GPIO](https://raspberrypi.stackexchange.com/questions/246/powering-without-using-the-micro-usb#581) header. To save my micro USB cable, I sacrificed an unused mini USB cable (who uses mini USB anymore?), split it open:

![mini USB cable open](https://user-images.githubusercontent.com/26856618/33521124-e4ab5176-d77d-11e7-9953-a362ee132777.png)

The black and red wires are V- and V+ respectively as you might expect, the white and green are D+/D-. Is the shielding ground? Turned to [USB device cable shield connection - grounding it or not?](https://forum.allaboutcircuits.com/threads/usb-device-cable-shield-connection-grounding-it-or-not.58811/) for insight: in this cable, the shielding connects to the connector shield on both sides, but is separate from the V- (ground) wire. It doesn't have to be, but I connected the shield to ground. 

Measuring with a multimeter confirms the red wire is +5 volts. The wires are stranded, not easily breadboardable, to fix this I soldered them onto a 2-pin 0.1" header which can plug into the breadboard power rails (using the upper rails for 3.3V, lower rails for 5V):

![Powering from GPIO header 5V](https://user-images.githubusercontent.com/26856618/33521193-0b24eb44-d780-11e7-9d64-faf261e901cd.png)

### USB hub + power LED

The external USB ports in
*[Building a small custom Raspberry Pi Zero laptop in a cardboard box](https://satoshinm.github.io/blog/171112_building_a_small_custom_raspberry_pi_zero_laptop_in_a_cardboard_box.html)* were provided by a Monoprice Aquamini 4-port hub, which I turned upside-down to hide the annoying red power indicator LED. For this upgrade, I took it apart, prying apart the plastic shell with nothing plugged in:

![Monoprice Aquamini 4-port USB hub disassembled](https://user-images.githubusercontent.com/26856618/33521597-957b79b0-d78b-11e7-842a-e70248eeba3d.png)

There's that pesky bright red LED. Desolder (back of PCB labeled "HUB-107-2", two holes above it are for the indicator):

![Aquamini hub annoying LED removed](https://user-images.githubusercontent.com/26856618/33521618-4108f42e-d78c-11e7-9964-b54e46b78f30.png)

How about replacing it with a soft diffused green?

![USB hub mod green LED](https://user-images.githubusercontent.com/26856618/33521601-d0268c26-d78b-11e7-828d-3214e149eda6.png)

But I'd like to see the light outside of the case. Fortunately I had a 2-pin cable, with hammer headers (not unlike [these](https://www.adafruit.com/product/3413)), so I soldered it in the LED's place, and folded the LED's leads back on themselves to snugly fit into the plug:

![USB hub green LED cabled](https://user-images.githubusercontent.com/26856618/33521605-f04f465a-d78b-11e7-89c3-6145d2796d0e.png)

Cut a hole on the front of the case and push the LED through. Sadly, while moving the Pi from the older version of this laptop the activity LED lifted from the board, along with the bulkier LED I soldered onto it (too much stress):

![Rip Pi indicator LED](https://user-images.githubusercontent.com/26856618/33521642-ca812302-d78c-11e7-81ca-4e85ebd0dcfa.png)

This is worth fixing, but for now, we have a front LED showing the USB hub power. Note that the USB hub is powered even when the Pi is "off!". A [phantom load](https://en.wikipedia.org/wiki/Standby_power), slowly draining the battery unless it is disconnected. So this external USB hub power indicator serves as a harder indicator of if there is any power:

![Front panel USB power LED](https://user-images.githubusercontent.com/26856618/33521645-f2943ff0-d78c-11e7-9238-4780e69176fe.png)

Finally, left the hub's plastic shell off (primarily because I couldn't thread the LED cable through its small hole, but also to reduce bulkiness), then cut holes in the cardboard case for the two external USB ports:

![Two external USB ports in case](https://user-images.githubusercontent.com/26856618/33521667-77e32da6-d78d-11e7-81bc-8ac675579c6c.png)

![USB ports side corner view](https://user-images.githubusercontent.com/26856618/33521752-e0f6f370-d78f-11e7-9aa3-9d62d677fb23.png)

The laptop is starting to come together!

### Ethernet port

I had some trouble with this one, the USB-to-Ethernet adapter has a longer cord to allow a straight shot to the back, and not long enough to allow exposing the port on another side. Settled on looping the cord through:

![USB-to-Ethernet adapter in laptop](https://user-images.githubusercontent.com/26856618/33521676-a89b52f2-d78d-11e7-9f06-d89a795339b6.png)

### Video output

Considered removing the DVI-to-HDMI adapter, and exposing HDMI directly (through the HDMI-to-mini-HDMI adapter), but the micro USB data cable on the back of the Pi Zero sticks out too far to fit the HDMI port through the back of the laptop. This may be solvable using a right-angle USB plug, but not any USB cable will do, since it allows plugging USB-A devices into it (expanded via a hub), a special "on-the-go" (OTG) cable is required, I have this one: [USB OTG Host Cable - MicroB OTG male to A female](https://www.adafruit.com/product/1099). This is an area of improvement (maybe go with a [Ethernet Hub and USB Hub w/ Micro USB OTG Connector](https://www.adafruit.com/product/2992) while we're at it?), but for now, I stuck with the DVI port so it moves the Pi far away enough from the back of the case:

![DVI port](https://user-images.githubusercontent.com/26856618/33521695-e55908d8-d78d-11e7-960d-99863ec15884.png)

Another downfall is the port and Pi are raised up from the bottom of the case, because of the short length of the GPIO ribbon cable. Could be improved by adding a false bottom, but for now, it hovers in midair.

Back view of the DVI port and Ethernet port:

![Video and Ethernet rear](ps://user-images.githubusercontent.com/26856618/33521746-b176e4f2-d78f-11e7-8723-dc36f5b3c310.png)


### Battery

Again there's a lot to be improved on here, but for now I taped the battery in the corner, plugged the USB safety tester (power usage meter) into it, then the USB-A cable which connects to the headers for +5 V GPIO on the breadboard.

![Battery stuff](https://user-images.githubusercontent.com/26856618/33521735-4cadd45e-d78f-11e7-8da8-fcc4034f1b66.png)

A power on/off switch and/or a selector to allow external power would be really nice, not this time but now that the power supply lines are on the breadboard, a future improvement could be more easily made. There's always a lot more to make better.

## Final product

Here we go:

![Laptop turned on, front view](https://user-images.githubusercontent.com/26856618/33521634-8e2f8aa6-d78c-11e7-9dd7-a7c7aa85ea61.png)

This new upgraded laptop looks about the same as in *[Building a small custom Raspberry Pi Zero laptop in a cardboard box](https://satoshinm.github.io/blog/171112_building_a_small_custom_raspberry_pi_zero_laptop_in_a_cardboard_box.html)*, but has much more breathing room internally. Especially with the internal breadboard, easing future prototyping experiments. And when the laptop is closed, you can't even tell it is more than a box:

![Closed box](https://user-images.githubusercontent.com/26856618/33521760-17d5bcc8-d790-11e7-882a-5ab41e8c12f5.png)

---

**[Comments?](https://www.reddit.com/r/raspberry_pi/comments/7h79lb/custom_laptop_upgrades_internal_breadboard_power/)**

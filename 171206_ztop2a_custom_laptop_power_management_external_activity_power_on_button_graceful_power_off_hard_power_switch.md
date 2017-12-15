# Custom laptop power management: external activity, power on button, graceful power off, hard power switch

by snm, December 6th, 2017

* auto-gen TOC:
{:toc}

## Activity LED

In the [previous update](https://satoshinm.github.io/blog/171202_ztop2_custom_laptop_upgrades_internal_breadboard_power_indicator_and_larger_case.html) to my [custom Raspberry Pi Zero-based cardboard laptop](https://satoshinm.github.io/blog/171112_building_a_small_custom_raspberry_pi_zero_laptop_in_a_cardboard_box.html), the onboard surface-mount activity LED on the Pi Zero, along with the external LED I soldered on top of it, accidentally lifted off the board. This is an opportunity to fix it. My plan was to use stranded wire for strain relief, not solid core as in the first laptop. And a red LED for activity, to contrast with the green LED for power. Here's what we have to work with:

![Lifted pad of status activity LED](https://user-images.githubusercontent.com/26856618/33644021-4bbc27a2-d9f7-11e7-9b63-1e15b80b4280.png)

Let's take a closer look:

![Closeup of lifted pad of activity LED](https://user-images.githubusercontent.com/26856618/33644041-6889bf3e-d9f7-11e7-87ef-3f0bd3afc7cb.png)

It's not looking good. The pads are completely gone. The LEDs still are operational, off the board:

![Status LEDs ripped off still working](https://user-images.githubusercontent.com/26856618/33644068-8c020f66-d9f7-11e7-966e-80e1bb94f9ca.png)

but that doesn't help us much off the Pi. So I tried applying solder to where the LED was, but to no avail. Can we solder somewhere else on the board? Refer to the official [Raspberry Pi schematics](https://www.raspberrypi.org/documentation/hardware/raspberrypi/schematics/README.md), for the Raspberry Pi Zero revision 1.3:

![Schematic status "ACT" LED](https://user-images.githubusercontent.com/26856618/33644136-eb578496-d9f7-11e7-8c57-cfc508c9e2fe.png)

This is somewhat helpful, it reveals the small resistor to the upper left (R20) is 470 Ω, connecting to 3.3 V. Measuring the resistor with a multimeter confirms it is 469.9 Ω, within spec. But unfortunately it connects to power, not to the signal. The other side, the cathode (negative), connects to the `STATUS_LED` signal. And where may this `STATUS_LED` signal come from? The "reduced schematics" don't say. But there are a bunch of exposed pads on the reverse side, labeled with "PP":

![Reverse of Raspberry Pi Zero](https://user-images.githubusercontent.com/26856618/33644234-6be39258-d9f8-11e7-8f92-43ca48dc3697.png)

These are test pads. Promising? Found a spreadsheet labeling what they are for: [Raspberry Bplus / 0 Test Pads](https://docs.google.com/spreadsheets/d/15kHcow993d_2CS6rZxQMq0wAWlgWNaHU_go1Hyf08ck/), screenshotted here in its entirety:

![Test pads reference](https://user-images.githubusercontent.com/26856618/33644365-1fc0fb9e-d9f9-11e7-8606-0cd0025be127.png)

PP13 is for the ACT activity/status LED on the Raspberry Pi B+, but I can't find it on the Pi Zero. The Zero has much fewer test pads, numbered out of order. Listing by their location, left to right, grouped logically:

* PP1: USB +5V input, PP6: GND
* PP40: SDA0? Spreadsheet has comment "not on version 1.3 with camera", PP35: GND
* PP22: USB+, PP23: USB D-
* PP9: 1.8V
* PP8: 3.3V
* PP5: GND
* PP16: SD DAT0, PP17: SD DAT1, PP18: SD DAT2, PP19: SD CD, PP15: SD CMD
* PP39: SCL0?, PP36: "GPIO6 of LAN9514" on RPi B+ but the Zero has no Ethernet
* PP20: "H5V" on B+, ? on Zero
* PP38: CAM_GPIO1, PP37: CAM_GPIO2

There are plenty of test points for the SD card, surely it is useful during factory testing with [pogo pins](https://en.wikipedia.org/wiki/Pogo_pin). But no activity LED test point. Possible to hijack an SD card data or clock line? Maybe, but it won't give proper functionality, such as blinking during an error or on power off. We need something better.

What is the activity LED signal anyways? It's just a GPIO output, controlled by the kernel. Or rather, it used to be. Found a useful thread here: [Act LED GPIO](https://www.raspberrypi.org/forums/viewtopic.php?f=29&t=100219), wherein /boot/config.txt is edited to add `dtparam=act_led_gpio=18`, driving GPIO18 on pin 12...but this was on a Pi B+ in 2015, a followup comment the following year is the bearer of bad news:

> Are you using a Pi 3B? On the Pi 3B the activity LED is no longer driven by a GPIO, it's driven by a port expander which is driven by the VideoCore. I'm not sure if it's possible to control the activity LED on a Pi 3B.

There is even a detailed wiki page on elinux: [Assign GPIO as ACT-led](https://elinux.org/Assign_GPIO_as_ACT-led), last updated 2014, explaining how to recompile the kernel, since obsoleted by the `act_led_gpio` parameter. This is turn is obsoleted, but there is a replacement developed in 2016 for the Raspberry Pi 3: [raspberrypi/linux #1363 Cannot move ACT LED functionality to GPIO pin](https://github.com/raspberrypi/linux/issues/1363):

```
dtoverlay=pi3-act-led,gpio=18
```

and there's more information in StackOverflow: [How can I control the red LED again](https://raspberrypi.stackexchange.com/questions/44168/how-can-i-control-the-red-led-again). But what about on the Raspberry Pi Zero? This thread [PWR/ACT LED ported to a GPIO on RPi Zero](https://www.raspberrypi.org/forums/viewtopic.php?f=28&t=146455) claims `dtparam=act_led_gpio=22` (GPIO/BCM 22, physical pin 15) works great. Time to try it.

It works! Was that easy, no pi3-act-led needed on the Raspberry Pi Zero. I tested with a red LED wired in series with a 470 Ω resistor to ground, moved to GPIO 21 for board placement convenience, tested with `gpio -g blink 21` first, then added to `/boot/config.txt`:

```
dtparam=act_led_gpio=21
```

reboot, power back on, then enjoy the beautiful activity LED. First prototype on the internal breadboard I conveniently stuck on the lid of my custom "laptop" Pi box: 

![Custom ACT LED on breadboard](https://user-images.githubusercontent.com/26856618/33645190-6d09ac48-d9fe-11e7-8ee9-73485927d200.png)

then bring it out on the front panel, wiring a cable through the edges of the box, to a red LED inserted into a hole cut in the front:

![New activity LED wiring cable](https://user-images.githubusercontent.com/26856618/33646073-f9ec2d08-da02-11e7-899a-7b597e5da223.png)

it lights up when activity occurs, the red LED next to the green (USB) power LED:

![Front panel activity LED on, by green USB power LED](https://user-images.githubusercontent.com/26856618/33646090-132d8f0a-da03-11e7-886d-c3a138244a12.png)

## Power on pushbutton (P6 RUN header)

When the system is in standby mode, including after being turned "off" by `sudo shutdown now`, it is not completely off. Sometimes called [soft off](https://answers.yahoo.com/question/index?qid=20080413122638AAQR0Nm). The board is still powered, and for example powers the USB ports, so the green power LED I added [last time](https://satoshinm.github.io/blog/171112_building_a_small_custom_raspberry_pi_zero_laptop_in_a_cardboard_box.html) brought out from the USB hub lights up. There is actually a header for turning it back on, labeled "RUN". Short the two pads together to turn it on when in standby. The Pi doesn't come with one, but all you need is any pushbutton, I happen to have this one:

![Micro switch](https://user-images.githubusercontent.com/26856618/33646318-c8f40b02-da03-11e7-98a5-494bb5592872.png)

it fits almost perfectly in the RUN unpopulated header, here is it soldered viewed from above, in the bottom right-hand corner adjacent to the GPIO header:

![RUN header soldered top](https://user-images.githubusercontent.com/26856618/33646380-0d7e61be-da04-11e7-88d6-21891b8c99a5.png)

I had to slightly bend the pins, and cut off part of the metal housing to avoid shorting out the other GPIO pins, but it works, hanging off the bottom of the Pi:

![RUN header pushbutton switch below](https://user-images.githubusercontent.com/26856618/33646417-3beff9d6-da04-11e7-83c8-ae68ed087085.png)

This power on button could be brought out to the front of the laptop, but I decided to leave it internal for now, so you have to open the box to turn it on, making accidental power-ons less likely. Usually the power button would be on the keyboard, compare to the [IBM Thinkpad](https://en.wikipedia.org/wiki/ThinkPad) keyboard, where the power button is on the top middle:

![Thinkpad keyboard](https://user-images.githubusercontent.com/26856618/33646555-03d45122-da05-11e7-8177-6171d2bf7370.JPG)

or other laptops where the power button is somewhere else near the keyboard, but with the detachable foldable keyboard in this custom laptop, adding a power button on the keyboard isn't so easy. Maybe a separate microcontroller (or a simpler circuit?) could listen for activity on the USB keyboard, then electronically short the RUN header to wake up the Pi. The USB hub remains powered on when in standby, so circuit would only need to monitor the USB keyboard connection, somehow. Is a full USB host mode supporting microcontroller required? Most PCs support this feature, "Power On By Keyboard", seen in [How to turn on computer via USB Keyboard (closed)](https://superuser.com/questions/596758/how-to-turn-on-computer-via-usb-keyboard):

![CMOS settings Power on by Keyboard](https://user-images.githubusercontent.com/26856618/33646825-828f9e30-da06-11e7-8f58-0a68a875b250.png)

leaving "power on by keyboard" unresolved for now. I'll have to push the pushbutton soldered underneath the Pi, no big deal.

## Graceful power off button

Shorting the P6 RUN header when the Pi is off will turn it on, which is good, but doing the same while it is off while abruptly [reset the BCM2835](https://elinux.org/RPi_Low-level_peripherals#P6_Pinout), which is bad. Filesystem corruption is possible. To [shutdown properly](https://raspberrypi.stackexchange.com/questions/381/how-do-i-turn-off-my-raspberry-pi), you need to run `sudo shutdown now`. For some reason, most of those StackExchange responses say to use `shutdown -h now`, where `-h` is equivalent to `--poweroff`, which is the default (unless you also pass `--halt` or `-H`) so it is unnecessary:

```sh
man shutdown
       -P, --poweroff
           Power-off the machine (the default).

       -r, --reboot
           Reboot the machine.

       -h
           Equivalent to --poweroff, unless --halt is specified.
```

A confusing mix of flags... furthermore, equivalent to `halt -p` or `poweroff`, but both of those commands are "legacy commands available for compatibility only", so I'm going to use `shutdown now`. Now that the command is settled, all we need is the hardware to trigger it, and software to listen and execute said command.

[SpaceBlogs: Shut down your Raspberry Pi on button press and add reset function](http://spaceblogs.org/2013/06/03/shut-down-your-raspberry-pi-on-button-press-and-add-reset-function/) links to [shutdown.py](http://flipdot.org/blog/uploads/shutdown.py.txt), copied here in its entirety:

```python
# This script will wait for a button to be pressed and then shutdown
# the Raspberry Pi.
# The button is to be connected on header 5 between pins 6 and 8.

# http://kampis-elektroecke.de/?page_id=3740
# http://raspi.tv/2013/how-to-use-interrupts-with-python-on-the-raspberry-pi-and-rpi-gpio
# https://pypi.python.org/pypi/RPi.GPIO

import RPi.GPIO as GPIO
import time
import os

# we will use the pin numbering of the SoC, so our pin numbers in the code are 
# the same as the pin numbers on the gpio headers
GPIO.setmode(GPIO.BCM)  

# Pin 31 (Header 5) will be input and will have his pull up resistor activated
# so we only need to connect a button to ground
GPIO.setup(31, GPIO.IN, pull_up_down = GPIO.PUD_UP)  

# ISR: if our button is pressed, we will have a falling edge on pin 31
# this will trigger this interrupt:
def Int_shutdown(channel):  
	# shutdown our Raspberry Pi
	os.system("sudo shutdown -h now")
	
# Now we are programming pin 31 as an interrupt input
# it will react on a falling edge and call our interrupt routine "Int_shutdown"
GPIO.add_event_detect(31, GPIO.FALLING, callback = Int_shutdown, bouncetime = 2000)   

# do nothing while waiting for button to be pressed
while 1:
        time.sleep(1)
```

They connect pins 6 (GPIO31) and 8 (GND) of...[header P5](https://elinux.org/RPi_Low-level_peripherals#P5_header)? This header was added in [Raspberry Pi Revision 2.0](https://www.raspberrypi-spy.co.uk/2012/09/raspberry-pi-p5-header/), intended to be populated on the reverse side of the board, providing power, ground, and four GPIOs: 28, 29, 30, 31. But the Pi Zero doesn't have this header or GPIOs brought out, the [schematic](https://www.raspberrypi.org/documentation/hardware/raspberrypi/schematics/Raspberry-Pi-Zero-V1.3-Schematics.pdf) only shows up to GPIO27, oh well. No matter, any GPIO can be used. For wiring convenience, I picked GPIO 17:

```python
#!/usr/bin/python
import RPi.GPIO as GPIO
import signal
import os

pin = 17

GPIO.setmode(GPIO.BCM)
GPIO.setup(pin, GPIO.IN, pull_up_down = GPIO.PUD_UP)
GPIO.add_event_detect(pin, GPIO.FALLING, callback=lambda channel: os.system("shutdown now"), bouncetime = 2000)

signal.pause()
```

Save this script to `/home/pi/shutdown.py`, then run it on startup by appending to `/etc/rc.local` (no need to use crontab):

```sh
python /home/pi/shutdown.py &
```

Wire a pushbutton from GPIO 17 to GND:

![Graceful power down pushbutton](https://user-images.githubusercontent.com/26856618/33699329-52760434-dac7-11e7-8e6e-80ee396b23ba.png)

Test by pressing the black button, confirming the system shuts down gracefully. Then power it back on by pressing the red pushbutton added earlier, and watch it power on. 

Admittedly, having two buttons is a clunky design. Could we use the same pushbutton to both power on and off? This question is left as an exercise to the reader.

## Hard power switch

The graceful power-off button only puts the Pi into "standby" mode, it still powers USB and can be awoken, therefore, it remains drawing power. The battery will slowly drain due to this phantom power. The solution? Yet another switch, duh!

I happened to have an old rocker switch for power, inserted it between the +5 V power rail from USB and +5 V on the GPIO header:

![Hard power switch 5 V](https://user-images.githubusercontent.com/26856618/33699836-6494faa0-daca-11e7-8fe6-5cd441920284.png)

Now it can be flipped to either force a shutdown while the system is on (not recommended), or completely turn it off after a graceful shutdown into standby mode, to conserve power. Tap the black pushbutton to initiate a shutdown, wait until the activity LEDs stop blinking, then the user can flip this switch and watch the green power LED go dark.

For now I used a single-pole single-throw (SPST) switch, but a future upgrade could be to use a double-throw (SPDT) switch to select between the built-in battery in this laptop, or an external USB charging port. To this end I purchased a few micro USB breakout boards: [5Pcs CJMCU Micro USB Board Power Adapter 5V Breakout Switch Interface Module For Arduino](https://www.aliexpress.com/item/5Pcs-CJMCU-Micro-USB-Board-Power-Adapter-5V-Breakout-Switch-Interface-Module-For-Arduino/32838024725.html) for $0.97/5, when they arrive this laptop could grow a micro USB charging port.

That's all for now, there's a lot more that could be done to improve power management in this Raspberry Pi Zero-based custom laptop, but this preliminary effort goes to show the complexity and intricacies of dealing with power in a mobile computing system.


---

**[Comments?](https://www.reddit.com/r/raspberry_pi/comments/7i4fim/custom_laptop_power_management_adding_an_external/)**

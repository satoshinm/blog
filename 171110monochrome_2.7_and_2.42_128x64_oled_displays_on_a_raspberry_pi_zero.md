# Monochrome 2.7" and 2.42" 128x64 OLED displays on a Raspberry Pi Zero

by snm, November 10th, 2017

[OLED](https://en.wikipedia.org/wiki/OLED) (organic light-emitting diode) displays are renowned for their lack of backlight requirement, allowing for deeper blacks and higher contrast. Used from watches to television sets, but to experiment with using OLEDs, I purchased a few monochrome 128x64 OLED display modules from Adafruit:

* [Monochrome 2.7" 128x64 OLED Graphic Display Module Kit](https://www.adafruit.com/product/2674)
* [Monochrome 2.42" 128x64 OLED Graphic Display Module Kit](https://www.adafruit.com/product/2719)

## Interfacing mode for SSD1325: SPI

![2.7" module unboxing](https://user-images.githubusercontent.com/26856618/32684064-f4f4b7e8-c634-11e7-982f-45af22b6e55d.png)

Solder on the included header:

![Soldering pins](https://user-images.githubusercontent.com/26856618/32684107-6419fe44-c635-11e7-97f1-afab9fe92966.png)

and plug into a breadboard:

![Breadboarded](https://user-images.githubusercontent.com/26856618/32684119-86b238ae-c635-11e7-814f-28f1aafd9161.png)

From here we have a choice, the SSD1325 controller chip supports three different interfacing modes (and other OLED controllers support various other modes as well):

![Pinout with interfacing modes](https://user-images.githubusercontent.com/26856618/32410903-3e03af6c-c189-11e7-965e-b950056c8c85.png)

Decided to go with SPI due to the low pin count. The reverse side of the board shows the BS1 and BS2 jumpers were already configured as "00", meaning SPI:

![SSD1325 module reverse](https://user-images.githubusercontent.com/26856618/32683191-4537e9b6-c62e-11e7-9f27-466475ce15ea.png)

Curiously, this board (labeled "TW28642270B03") looks a lot different than [Adafruit's 2.7"](https://www.adafruit.com/product/2674) pictures:

![Adafruit website OLED module](https://user-images.githubusercontent.com/26856618/32476706-f4bec50c-c32d-11e7-9fa5-8eaa76fbf7f6.jpg)

newer/prototype version?

Now that the header is soldered, wire it up to match the documented [pinout]((https://learn.adafruit.com/2-7-monochrome-128x64-oled-display-module)), I connected it to a Raspberry Pi Zero as follows (using [pinout.xyz](https://pinout.xyz) for reference):

| OLED Module Pin | Pi Zero Pin | Remarks |
| ------------- | ----------- | ------- |
| 1 GND         | P01-20 GND | GND, ground |
| 2 +3V         | P01-17 3V3 | 3.3V, power |
| 4 DC          | P01-18 GPIO 24 | data/command, the fourth wire of "4-wire SPI" (sic) | 
| 7 SCLK        | P01-23 GPIO 11 (SCLK) | serial clock
| 8 DIN         | P01-19 GPIO 10 (MOSI) | MOSI, master out slave in (MISO is not connected, no slave output) |
| 15 /CS        | P01-24 GPIO 8 (CE0) | Chip enable/select 0 |
| 16 /RES       | P01-22 GPIO 25 | reset |

![2.7" connected](https://user-images.githubusercontent.com/26856618/32684089-2a553c1e-c635-11e7-9bbb-dd80da7db14d.png)

## luma.oled on Pi Zero

Run `sudo raspi-config` then enable SPI in "4 Interfacing Options" then "P5 I2C". The [luma-oled guide](https://luma-oled.readthedocs.io/en/latest/hardware.html#identifying-your-serial-interface) said to enable in "> Advanced Options > A6 SPI", I guess they moved it for some reason in Debian Stretch. After enabling there are character devices for SPI:

```sh
pi@pizero:~ $ ls -l /dev/spi*
crw-rw---- 1 root spi 153, 0 Nov  7 01:39 /dev/spidev0.0
crw-rw---- 1 root spi 153, 1 Nov  7 01:39 /dev/spidev0.1
```

then give the `pi` user access:

```sh
sudo usermod -a -G spi,gpio pi
```

and follow the [luma.oled installation steps](https://luma-oled.readthedocs.io/en/latest/install.html). 

```sh
pi@pizero:~ $ sudo apt-get install python-dev python-pip libfreetype6-dev libjpeg-dev build-essential
...
pi@pizero:~ $ sudo -H pip install --upgrade pip
Requirement already up-to-date: pip in /usr/lib/python2.7/dist-packages
pi@pizero:~ $ sudo apt-get purge python-pip
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  libpython-all-dev python-all python-all-dev python-keyring python-keyrings.alt python-secretstorage python-wheel
Use 'sudo apt autoremove' to remove them.
The following packages will be REMOVED:
  python-pip*
0 upgraded, 0 newly installed, 1 to remove and 1 not upgraded.
After this operation, 671 kB disk space will be freed.
Do you want to continue? [Y/n] 
(Reading database ... 122781 files and directories currently installed.)
Removing python-pip (9.0.1-2+rpt1) ...
Processing triggers for man-db (2.7.6.1-2) ...
pi@pizero:~ $ sudo -H pip install --upgrade luma.oled
sudo: pip: command not found
```

What was the point of removing pip, causing the last command to fail?  This is a known issue in the luma.core documentation on Stretch: https://github.com/rm-hull/luma.core/issues/106 - fix is to reinstall with `sudo apt-get install python-pip` then  install luma.oled as normal with `sudo -H pip install --upgrade luma.oled`. This succeeded. Now to try the examples, found in http://github.com/rm-hull/luma.examples - run an example:

```sh
pi@pizero:~/luma.examples/examples$ python clock.py --display ssd1325 --interface spi
Display: ssd1325
Interface: spi
Dimensions: 128 x 64
----------------------------------------
```


![Clock example on 2.7](https://user-images.githubusercontent.com/26856618/32683299-0bb4bc04-c62f-11e7-9b8f-847cb2655c13.png)

It works! The OLED module shows the clock example. A sigh of relief, since I actually first ordered this same module [from Amazon, Adafruit's Monochrome 2.7" 128x64 OLED Graphic Display Module Kit](https://www.amazon.com/gp/product/B013W22SUW), but the device was defective. Adafruit tests each module before shipment so I had more confidence their product would operate correctly. Here's another module:

## 2.42" display with SSD1305

![2.42" module unboxing](https://user-images.githubusercontent.com/26856618/32684026-9a868c28-c634-11e7-8606-dd5100ea9cab.png)

The [2.42" display](https://www.adafruit.com/product/2719) is also 128x64 but slightly smaller than the 2.7". It also has a different controller chip: SSD1305 instead of SSD1325. The SSD1305 supports 8-bit and SPI, just like the SSD1325, but also I2C. Solder on the header and plug it into the breadboard; fortunately, it has the same pinout so and supports the same interfacing mode as the other module so no wiring changes are needed, it can be swapped it interchangeably.

This board came configured for 8-bit interfacing mode (R18=R20=1):

![R18=R20=1](https://user-images.githubusercontent.com/26856618/32683993-536eeec0-c634-11e7-8514-a4112b528391.png)

necessitating desoldering the jumpers and moving them to R19 and R21, selecting SPI:

![R19 and R21 SPI](https://user-images.githubusercontent.com/26856618/32684012-800c67d2-c634-11e7-98c4-9212e5ab3714.png)

luma.oled doesn't recognize "ssd1305" but it does support ssd1306 and selecting that display type works well:

```sh
pi@pizero:~/luma.examples/examples $ python clock.py --display ssd1306 --interface spi
Display: ssd1306
Interface: spi
Dimensions: 128 x 64
----------------------------------------
```

![2.42" clock](https://user-images.githubusercontent.com/26856618/32684042-d49cd9bc-c634-11e7-8f39-5ce08b39b46e.png)

## Both modules

![Two OLED modules at the same time](https://user-images.githubusercontent.com/26856618/32684995-71224f3e-c63e-11e7-99bf-8e3d501cb0b7.png)

Here's how the two modules (2.7" and 2.42") compare:

![2.7" vs 2.42"](https://user-images.githubusercontent.com/26856618/32684141-b1317cac-c635-11e7-8074-ccc3fa8eefd5.png)

Would be nice to connect both, but how? The SPI clock/data lines can be shared, and there are two chip enables (CE0 and CE1) so the CE1 can be used for a second SPI device. DC and /RES require two more GPIOs, arbitrarily selected:

| OLED Module Pin | Pi Zero Pin | Remarks |
| ------------- | ----------- | ------- |
| 1 GND         | P01-20 GND | GND, ground |
| 2 +3V         | P01-17 3V3 | 3.3V, power |
| 4 DC          | **P01-16 GPIO 23** | data/command | 
| 7 SCLK        | P01-23 GPIO 11 (SCLK) | (shared) serial clock
| 8 DIN         | P01-19 GPIO 10 (MOSI) | (shared) MOSI, master out slave in |
| 15 /CS        | **P01-26 GPIO 7 (CE1)** | Chip enable/select 1 |
| 16 /RES       | **P01-36 GPIO 16 | reset |

Now to drive this second display with luma.oled, we need to specify SPI device 1, as well as the data/command and reset pins:

```sh
pi@pizero:~/luma.examples/examples $ python clock.py --display ssd1325 --interface spi --spi-device 1 --gpio-data-command 23 --gpio-reset 16 
```

running the other command simultaneously, driving both devices:

```sh
pi@pizero:~/luma.examples/examples $ python clock.py --display ssd1306 --interface spi
```

![Both displays at the same time](https://user-images.githubusercontent.com/26856618/32684862-6f932f9a-c63d-11e7-9eeb-6fae6e1114a2.png)

The refresh rate of the 2.7" SSD1325 is slower, so my camera picks it up mid-frame, but it looks fine in person, albeit dimmer than the 2.42" SSD1305 (which doesn't have grayscale). Now with two clocks, we can never tell what time it is, as the saying goes. Fortunately, they both update simultaneously, so they never disagree.

What good are two OLED displays on a Pi? luma.oled has some nifty examples, including game simulators, here is invaders.py and jetset_willy.py:

```sh
python invaders.py --display ssd1325 --interface spi --spi-device 1 --gpio-data-command 23 --gpio-reset 16 
python jetset_willy.py --display ssd1306 --interface spi
```

![Invaders and jetset willy](https://user-images.githubusercontent.com/26856618/32685264-b5c41948-c642-11e7-859d-55fc7db14579.png)

but a practical use for these displays will have to wait for another time. This concludes this experiment, we have seen how to successfully setup two OLED display modules, a 2.7" SSD1325-based board and a 2.42" SSD1305-based board, via SPI, to a Raspberry Pi Zero.

![sys_info and starfield](https://user-images.githubusercontent.com/26856618/32685337-e7e4bcc4-c643-11e7-96e0-5e8fbf961a6b.png)

---

**[Comments?](https://www.reddit.com/r/raspberry_pi/comments/7ckils/building_a_small_custom_raspberry_pi_zero_laptop/)**

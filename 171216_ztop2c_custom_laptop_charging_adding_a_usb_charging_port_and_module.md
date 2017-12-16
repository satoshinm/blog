# Custom laptop charging: adding a USB charging port and module

by snm, December 16th, 2017

* auto-gen TOC:
{:toc}

Continuing from *[Custom laptop power management: external activity, power on button, graceful power off, hard power switch](https://satoshinm.github.io/blog/171206_ztop2a_custom_laptop_power_management_external_activity_power_on_button_graceful_power_off_hard_power_switch.html)*.

## micro USB cable Y splitter for external charging port

The Morphie juice pack I am using for this laptop battery has a female micro USB port for charging, and a female USB-A port for powering. The power port has to plug into the Raspberry Pi board to run it, but then the charging port isn't accessible on the outside of the laptop cardboard box. So I purchased this micro USB cable:

* [AT 1PCS Micro USB 2.0 Splitter Y 1 Female to 2 Male Data Charge Cable Extension Cord For phone High Quality sync data cable](https://www.aliexpress.com/item/AT-1PCS-Micro-USB-2-0-Splitter-Y-1-Female-to-2-Male-Data-Charge-Cable/32825633032.html)

It arrived after some time. One end of the cable is female micro USB, and it splits into two cables: the longer cable is for data and charging, the shorter is for charging only. I tested by plugging the female side into a charger, then each of the male ends into the battery pack: both succeeded in charging it. And I was able to power two devices at the same time, thanks to this splitter.

![Y splitter cable](https://user-images.githubusercontent.com/26856618/34074022-a5eff1f6-e25b-11e7-9642-2e5aacc98827.png)

Cut a hole for the female port, to allow docking the laptop to charge the battery externally:

![Cut micro USB female charging port hole](https://user-images.githubusercontent.com/26856618/34074043-0e961f82-e25c-11e7-8e66-f3937db496a6.png)

## Lithium battery charging module

Also bought a lithium battery charger, [1PCS 5V TENSTAR ROBOT 1A Micro USB 18650 Lithium Battery Charging Board Charger Module+Protection Dual Functions TP4056](https://www.aliexpress.com/item/1PCS-5V-1A-Micro-USB-18650-Lithium-Battery-Charging-Board-Charger-Module-Protection-Dual-Functions/32467578996.html). It uses a [TP4056 1A Standalone Linear Li-lon Battery Charger with Thermal Regulation](https://dlnmh9ip6v2uc.cloudfront.net/datasheets/Prototyping/TP4056.pdf). Accepts micro USB charging input or +/- terminals, then outputs to B+/B- battery terminals and OUT+/- output:

![TP4065 Lithium Battery Charging Board](https://user-images.githubusercontent.com/26856618/34074018-856f8496-e25b-11e7-8e25-5a4d8673239f.png)

Without a battery (need to acquire a 18650 lithium battery to charge), tested this charger board only plugged into USB power:
Measured 5.1 V on the input:

![Charger module USB input voltage](https://user-images.githubusercontent.com/26856618/34074033-d0a438da-e25b-11e7-9470-440a63f481ab.png)

and about 4.112 V on the outputs:

![Charger module OUT output voltage](https://user-images.githubusercontent.com/26856618/34074036-e21d737e-e25b-11e7-910a-43936ad4030d.png)

Is it possible to [undervolt](https://www.raspberrypi.org/forums/viewtopic.php?t=9549) the Pi, running it on less than 5 volts? 

Attached a capacitor to the battery terminals, with large capacitance (1 F) the charge and standby LEDs would continuously alternate. Used a smaller 47 µF capacitance (rated for 50 V) and it seemed to be more stable:

![Capacitor on charging board](https://user-images.githubusercontent.com/26856618/34074051-2d50677a-e25c-11e7-9a29-ad5f9160f691.png)

Not sure if this is a good idea, but the battery charger is supposed to supply constant current and shutdown when current drops to a minimum value. Without a battery, the charge LED quickly yet dimly flashes, so I didn't want to leave it open. In any case, for now I'm using the battery charger board as a dumb micro USB adapter, since I don't have a battery and the output is less than the 5 V the Pi requires.

Put the charger board in the laptop, plugged into the Y splitter charger port. The other side of the Y splitter plugs into the Morphie juice pack, so external battery power both charges the juice pack and powers the Pi (via the lithium ion charging board, which I'm not using for charging at the moment, only for the micro USB port):

![Y splitter to battery and board](https://user-images.githubusercontent.com/26856618/34074062-52d27218-e25c-11e7-9dd9-60d46df4d419.png)

Plug in external USB power to the micro USB port, the Pi powers up:

![Powered up externally powered Pi](https://user-images.githubusercontent.com/26856618/34074086-a4da55d0-e25c-11e7-8ded-689a7f5ecc41.png)

## Front panel external power/charging LED

There's a small problem. The charger board has a blue "charging" LED, but it is on the module which I put *inside* the laptop, so it isn't visible on the outside. To fix this, soldered a cable across the LEDs and broke it out to the front panel:

![Front panel blue LED](https://user-images.githubusercontent.com/26856618/34074106-2d217c3e-e25d-11e7-92e4-d0d4fffefe33.png)

This LED joins the other front panel LEDs, now three total:

* Blue: external/charging power
* Green: system power
* Red: activity

To switch to battery power, a cable has to be plugged into the battery and out to the charger port. Eventually I'd like to bring it out directly to the breadboard, but I'm waiting on those [micro USB breakouts](https://www.aliexpress.com/item/5Pcs-CJMCU-Micro-USB-Board-Power-Adapter-5V-Breakout-Switch-Interface-Module-For-Arduino/32838024725.html) to do this. Importantly, the charging cable to the battery has to be unplugged or the Morphie juice pack will try to charge itself, intermittently blinking the power indicator, which can't be good, and it won't supply power. Making these configuration changes, the system boots up:

![Laptop off power](https://user-images.githubusercontent.com/26856618/34074207-e12c8f2e-e25e-11e7-9bb3-5d5d9cfc8a89.png)

## Problems

This "laptop" is getting better, but still has a lot of problems compared to a real commercial production-quality laptop:

* The charging module is only used as a micro USB port, it doesn't charge any battery yet (need to get a 18650), so the "charging" LED is misleading, since it isn't tied to the actual battery (the Morphie juice pack)
* Running from battery power requires swapping cables (waiting on a breakout board to fix this)
* No seamless transition between external/battery power (could be solved using the charger module with load sharing, or a microcontroller to detect the voltage transitions and automatically switch?)

Nonetheless, it's a start.
# The smart bulb / smart switch dilemma: smartening up a dumb wall switch

by snm, November 9th, 2017

How to make a "dumb" wall switch into a smart switch. We all have switches on our walls, such as these:

![Wall switch](https://user-images.githubusercontent.com/26856618/33756395-304aaade-dbaa-11e7-9430-339c9c3bf21d.jpg)

which controls power to an AC wall outlet. 

As a new user of home automation, I plugged a lamp with a smart light bulb into this switched outlet. This creates a problem: when the dumb light switch is off, you can't turn on the smart bulb. *Obviously*, because the switch cuts the power. But it is annoying because I would have to whip out my smartphone whenever I want to turn on/off the lights! Despite the ability to change bulb color etc. remotely e.g. via smart speakers or smartphones, installing this smart bulb was a step backwards.

This isn't an unknown problem, and there are possible solutions on the market. [Smart light switches](https://www.amazon.com/s/field-keywords=smart+light+switch)? But they are $25-45 and are installed into the wall, replacing the dumb switch. Very invasive, not to mention costly. My landlord probably won't appreciate ripping out her switches. I wanted to use my existing wall switch, efficiently. Hence this blog post. I'll be using an [ESP8266 WeMos D1 mini purchased from Aliexpress](https://www.aliexpress.com/wholesale?SearchText=wemos+d1+mini) for a couple bucks (varies about $2-5 + shipping), with two spare USB chargers and a few small inexpensive miscellaneous electronic components. Here's the behavior before, which sucks:

| State | Desired Result | Action Needed |
| ----- | -------------- | ------------- |
| Lamp on | Turn off     | Flip switch or use app |
| Lamp off | Turn on     | If the switch was on: flip it *twice* (off then on) or use the app<br>If the switch was off: flip the switch, you **can't** use the app

Behavior after implementing the "smartened up" device for the dumb switch, it all just works:

| State | Desired Result | Action Needed |
| ----- | -------------- | ------------- |
| Lamp on | Turn off     | Flip switch or use app |
| Lamp off | Turn on     | Flip switch or use app |

This I believe achieves the best of both worlds. If you only use a smart bulb with a dumb switch, then the problems above arise. If you only use a smart switch, then you lose the ability to control the color of the bulb, and have to change out or alter your wall switches. By using a smart bulb and dumb light switch plus this device we'll build soon, both these problems are solved.

## Design overview

In my case, I have a dual-outlet face plate, the top is switched, the second is always-on:

![Outlets](https://user-images.githubusercontent.com/26856618/33756443-6c7be0fe-dbaa-11e7-81c1-bf7831b129fa.png)

We'll wire the switched socket to GPIO on a microcontroller powered by the always-on socket, in order to sense when the dumb switch is on or off, allowing it to smoothly integrate with the rest of the home automation network.

## USB +5V power to GPIO digital inputs

Connect a USB charger to the wall socket, break out the +5V power from the USB cable, then connect to an ESP8266 GPIO input. The ESP8266 is a 3.3V device, but the input pins are [5 volt tolerant](https://hackaday.com/2016/07/28/ask-hackaday-is-the-esp8266-5v-tolerant/) so no level shifter is needed. I took a WeMos D1 Mini ESP8266, cannabilized an old USB cable, split out the +5 V power supply line (red wire) to the D4 GPIO input, and ground (black wire and shield) to ground, via a header plugged into the WeMos:

![USB power as GPIO input, take 1](https://user-images.githubusercontent.com/26856618/33756465-86f35566-dbaa-11e7-9416-bd37c988a7bd.png)

Wrote an Arduino sketch to loop and `digitalRead(D4)`, logging any changes, as in the [Arduino StateChangeDetection](https://www.arduino.cc/en/Tutorial/StateChangeDetection) example. First I tested with the ESP8266 powered by my computer plugged into USB, and the D4/GND cable plugged into a USB wall charger *not* plugged into the wall. When the charger is off, yet +5V from its USB cable is wired to D4, we consistently read a low, and when disconnected, a high. The built-in LED also illuminates respectively. So far so good.

However I ran into trouble powering the ESP8266 itself from *another* USB charger plugged into the wall. The built-in LED would light up dimly:

![Dim LED](https://user-images.githubusercontent.com/26856618/33756657-5637fb24-dbab-11e7-8ae2-5cb79e026313.png)

and if I plug it into an AC outlet, the ESP8266 crashes. Fortunately it doesn't kill it, and I can recover by cycling power, but something is clearly wrong. Measuring the voltage from the USB charger output shows 5.1 volts with no load, maybe this is too much for the 5-volt tolerant ESP8266 GPIO inputs? Maybe the USB charger expects a load, or the GPIO inputs need pull-down resistors to not float open? If anyone has more insight into why I can't simply connect +5V USB power to GPIO inputs I'd be interested but until then, time to try a different approach.

## Voltage divider to analog input

The ESP8266 has one analog ADC input pin, A0. Read it in Arduino using `analogRead(A0)`. But when I read it in a tight loop, the Wi-Fi became unresponsive, see: [ESP8266 Analog read interferes with wifi?](https://arduino.stackexchange.com/questions/19787/esp8266-analog-read-interferes-with-wifi). Read it only every 50 ms:

```c++
static int analog_value = 0;

void loop() {
  server.handleClient();

  if (millis() % 50 == 0) {
    analog_value = analogRead(A0);
  }
}
```

but don't connect it just yet. The analog pin only accepts 1 V max, so the 5 V has to be divided using a [voltage divider](https://en.wikipedia.org/wiki/Voltage_divider). With 10 kΩ and 47 kΩ, 5 volts will read as 5 * 10 / (10 + 47) = 0.877 volts, providing some margin of error if it exceeds this voltage, up to 5.7 V for 1.0 V. Current is I=V/R, so 5V/(10kΩ) = 0.5 mA, and 5V/(47kΩ) = 0.11 mA, with power dissipation of P=IV, (0.5 mA)(5 V) = 0.53 mW. The common 1/4 W or 1/8 W resistors or even smaller will be more than enough. Wire it up:

![Voltage divider with 10k and 47k resistors](https://user-images.githubusercontent.com/26856618/33792448-8357d4d6-dc54-11e7-90d0-47e5186f32cb.png)

Reads 0.888 V with the USB charger plugged into the wall socket with the light switch on:

![5V USB divided to <1V](https://user-images.githubusercontent.com/26856618/33792451-9eabfa1e-dc54-11e7-8a3f-33cef60ad2ea.png)

Good, it is safe to connect to the ESP8266 analog input:

![Connected to A0](https://user-images.githubusercontent.com/26856618/33792458-c141b708-dc54-11e7-942d-c6bf6806d1b4.png)

Moving from a breadboard to protoboard, solder it up:

![ESP8266 + vdiv to A0 on protoboard](https://user-images.githubusercontent.com/26856618/33792461-e3fb8ddc-dc54-11e7-8a38-5d4c4dc0023f.png)

![Reverse side](https://user-images.githubusercontent.com/26856618/33792462-f49416d2-dc54-11e7-96aa-34f2eb75d885.png)

Back to the software. This voltage is 264 or 263 units from analogRead. When flipped off, the voltage quickly drops to 3 or 2 units (the units are 1024th of a volt). If the USB cable is unplugged, then it instantly drops, but if the charger is unplugged, then it drops slower, due to residual charge. A bigger load may make it drop faster, but we can detect the change in software. If it drops below a threshold, consider it off:

```c
  int analog_value = analogRead(A0);
  input = analog_value > 260;
```

Detecting when the light switch turns on is visually instanteous, but even using a closer threshold, detecting when it turns *off* is noticeably slower. Would like it to be instant. [RE: WHAT IS SAMPLE RATE OF ANALOGREAD()? #16837](http://www.esp8266.com/viewtopic.php?f=32&t=2915) says it is 200 Hz. With the 1/(50 ms) delay we were reading only 20 Hz. Tenfold increase, sampling at the maximum rate:

```c
  if (millis() % 5 != 0) {
    return;
  }
```

With this change, the web server still works. And yet, off detection remains perceptably slow. Not good enough.

Not a complete loss, I kept this circuit, but the voltage is shown (converted to for display: `/1024*(47.0+10)/10`) only for informational purposes, on the ESP8266 web server interface. However for triggering will be done digitally instead:

## Optocoupler to digital input

The slow sample rate of the ESP8266's analog-to-digital converter is unbearable, so how about going back to digital? We may be able to get the resistive voltage divider working for logic level input, or use a fancier MOSFET-based level shifter, but for better isolation (avoiding tying together both grounds), one can use an *optocoupler*. I had one lying around I wanted to use anyways, the [Fairchild FOD817 4-pin DIP phototranistor optocoupler](http://www.onsemi.com/PowerSolutions/product.do?id=FOD817):

![Optocoupler FOD817 on breadboard](https://user-images.githubusercontent.com/26856618/33793765-bc978b74-dc72-11e7-8a53-3524a1ff5a07.png)

![Optocoupler FOD817 schematic](https://user-images.githubusercontent.com/26856618/33792835-42ffd2ca-dc5f-11e7-9d26-1b9f9fbee65a.png)

Per the data sheet, the absolute maximum continuous forward current is 50 mA, so like with any LED, we need a current-limiting resistor. R=V/I, (5 volts) / (50 mA) = 100 Ω. To allow for tolerances and overvoltages, a higher resistance of 220 Ω gives 22.7 mA, not far from typical LED current. On the other side, 10 kΩ resistor from the collector pulls up to 3.3V. An instructive diagram from [WestFlorida components](https://www.westfloridacomponents.com/blog/digital-isolators-vs-optocouplers/) (pardon the blurriness and low resolution):

![Optocoupler resistor usage](https://user-images.githubusercontent.com/26856618/33792937-64d5d342-dc62-11e7-990c-fb6713802823.jpg))

Construct the circuit on a breadboard:

![Breadboarded optocoupler in circuit](https://user-images.githubusercontent.com/26856618/33793780-f651b286-dc72-11e7-87d7-bdb8c1abc423.png)

Now when the light switch is turned on, the optocoupler emitter activates and the detector turns on, pulling the GPIO input (using D2 on the ESP8266) low. When the light switch is off, it is pulled high (3.3V). Since it works, solder it up permanently:

![Soldered optocoupler](https://user-images.githubusercontent.com/26856618/33793874-27387fcc-dc75-11e7-9bc1-924ee409ec3a.png)

![Reverse side of soldered optocoupler complete board](https://user-images.githubusercontent.com/26856618/33793883-4bb4def4-dc75-11e7-9e27-8aaa9c1839ba.png)

**Software (Arduino sketch for the ESP8266): [https://github.com/satoshinm/wallswitch](https://github.com/satoshinm/wallswitch)**

## Home automation

Now that the hardware and software is complete, time to integrate it with the rest of your home automation system. YMMV, but I decided to use [Homebridge](https://github.com/nfarina/homebridge), due to its plethora of [plugins](https://www.npmjs.com/search?q=homebridge-plugin).

[homebridge-udp-multiswitch](https://www.npmjs.com/package/homebridge-udp-multiswitch) looks useful, but it does the opposite of what we want: sending UDP packets, not receiving them. However with some changes, we can make it into a UDP server, here:

**Homebridge plugin UDP server: [https://github.com/satoshinm/homebridge-udpserver-multiswitch](https://github.com/satoshinm/homebridge-udpserver-multiswitch)**

Configure in ~/.homebridge/config.json:

```json
        {
                "accessory": "UdpServerMultiswitch",
                "name": "Wallswitch",
                "port": 8261,
                "on_payload": "ff",
                "off_payload": "00",
                "toggle_payload": "80"
        }
```

The switch reacts when driven by the physical wall switch via the ESP8266 device, but can also be switched in the app. Next up: automation, tying this switch to a lamp. I used a lamp with a smart bulb, and configured it to turn on with the wall switch turns on, turn off when it turns off. The smartified wall switch controls the smart lamp almost the same as before:

* Flip the light switch on, the lamp turns on
* Flip the light switch off, the lamp turns off

All that work for the same results? No actually this is way better. We have eliminated the fundamental flaw of losing control when the wall switch turns off the smart lamp. Now this works:

1. Flip the light switch off
2. Turn the lamp on using your smartphone (or other home automation tool)

The lamp turns on!

### Toggling

Since this a smart switch, we can do better. 

You ever used a lamp controlled by two wall switches? Flip one of the switches, it toggles the lamp, flip the other, it also toggles the lamp. How does this work? If the two switches were wired in series, then both switches would have to be on (AND), if in parallel, then either switch (OR) but not both. [Physics 230](http://users.wfu.edu/matthews/courses/p230/switches/SwitchesTut.html) has the answer: these wall switches are "3-way switches":

![3-way switches](https://user-images.githubusercontent.com/26856618/33799833-6e2aca34-dce8-11e7-916c-d9b52eca5f3e.gif)

technically a [SPDT instead of SPST](https://en.wikipedia.org/wiki/Switch#Contact_terminology),  implementing a logical [XOR](https://en.wikipedia.org/wiki/Exclusive_or). To expand beyond two, there are also 4-way switches:

![4-way switches](https://user-images.githubusercontent.com/26856618/33799831-50745da2-dce8-11e7-9d37-adf5f105ec34.gif)

These types of switches are very useful because they allow controlling the lamp from two places, and whenever you flip a switch something happens. They don't have the problem where if you turn off a switch when the lamp is already off, or vice versa, nothing happens. Sound familiar?

The current smart switch implementation has this problem, but it is easily fixed in [homebridge-udpserver-multiswitch](https://github.com/satoshinm/homebridge-udpserver-multiswitch) by adding a *toggling* UDP payload, effectively handled like this:

```js
var previousValue = services[i].getCharacteristic(Characteristic.On).value

service.getCharacteristic(Characteristic.On).setValue(!previousValue);          
```

Now whether you turn the physical light switch "on" or "off", it will always toggle the lamp. There no longer strictly an or or off physical state, because the switch can also be controlled programmatically. Much better.

## Summary

* Arduino sketch for the ESP8266: [https://github.com/satoshinm/wallswitch](https://github.com/satoshinm/wallswitch)
* [Homebridge](https://github.com/nfarina/homebridge) plugin UDP server: [https://github.com/satoshinm/homebridge-udpserver-multiswitch](https://github.com/satoshinm/homebridge-udpserver-multiswitch)**

The lamp can now be equally controlled programmatically by the app and by the physical light switch, as we expect, no compromise:

| State | Desired Result | Actions Supported |
| ----- | -------------- | ------------- |
| Lamp on | Turn off     | Flip switch or use app |
| Lamp off | Turn on     | Flip switch or use app |

---

**[Comments](https://www.reddit.com/r/esp8266/comments/7iqx0r/xpost_rhomeautomation_the_smart_bulb_smart_switch/?st=jb039502&sh=3d32eec5)[?](https://www.reddit.com/r/homeautomation/comments/7iqv12/the_smart_bulb_smart_switch_dilemma_smartening_up/?st=jb02utq6&sh=f68a2577)**

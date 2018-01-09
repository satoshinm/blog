# USB breakout boards for supplying power to your projects

by snm, January 8th, 2018

* auto-gen TOC:
{:toc}

The USB connector on one of my STM32 blue pills (primarily the one I was using for testing *[Pill Duck: Scriptable USB HID device using an STM32 blue pill, from mouse jigglers to rubber duckies](https://satoshinm.github.io/blog/171227_stm32hid_pill_duck_scriptable_usb_hid_device_using_an_stm32_blue_pill_from_mouse_jigglers_to_rubber_duckies.html)*) was getting loose, making a poor connection; this is a [known issue](http://wiki.stm32duino.com/index.php?title=Blue_Pill#Known_issues):

> * The micro-USB connector is not soldered to the board very well and is easily broken. Then first weld the connector better and if you want can cover the connector in epoxy glue or hot glue. There are multiple versions of this board with different connectors. Refer to the pictures for examples.

unfortunately, the damage was already done. Fortunately, there is another way to power these boards. I constantly seem to be needing to power devices using +5V from USB, so I purchased these Micro USB breakout boards: [5Pcs CJMCU Micro USB Board Power Adapter 5V Breakout Switch Interface Module For Arduino](https://www.aliexpress.com/item/5Pcs-CJMCU-Micro-USB-Board-Power-Adapter-5V-Breakout-Switch-Interface-Module-For-Arduino/32838024725.html), $1.50 for 5. They come with headers you can solder on:

![Micro USB breakouts out of the box](https://user-images.githubusercontent.com/26856618/34706133-3d99f234-f4ba-11e7-9b1e-2ba9dfad100c.png)

## USB-powered breadboard

Solder the header onto the USB breakout board, plug a cable in, plug it into the breadboard, then wire the GND and VCC lines to the + and - rails of the breadboard. From there, to 5V and G of the blue pill:

![USB-powered breadboard powering STM32 blue pill](https://user-images.githubusercontent.com/26856618/34706179-a65f1010-f4ba-11e7-94f0-86badc774e80.png)

The STM32 is powered successfully. But the ESP32 board, shown on the right of the breadboard, which I was using for *[ESP32-based wireless ticker display for Minecraft server activity: pushing up against the fourth wall](https://satoshinm.github.io/blog/180107_esp32ticker_esp32_based_wireless_ticker_display_for_minecraft_server_activity_pushing_up_against_the_fourth_wall.html)*, is a [LOLIN32 Lite](https://wiki.wemos.cc/products:lolin32:lolin32_lite), doesn't break out the 5V power pin on its headers, so it isn't powered here. Power can be supplied from its micro USB port, or the battery charge port. Can it be powered from 3.3V? Haven't tried

### Adding a voltmeter

To see the voltage, these little thingies are useful: [DC mini 0.36 " Digital Red LED Display 0-100V Voltmeter 3 Wires Voltage Meter volt tester for car battery test 40% off](https://www.aliexpress.com/item/DC-Digital-Display-0-100V-Voltmeter-mini-3-Wires-Red-0-36-LED-Voltage-Meter-volt/32801800193.html), 86¢. Ordered a few. 

Very easy to use, all you do is connect ground (G), power (V; 5V does the trick, 3.3V doesn't), and a third wire (IN) for the voltage you want to measure. IN and V can be tied together if the measured voltage is sufficient to supply the voltmeter. I'd like to use it on the breadboard to view the supply voltage, or potentially probe other voltages. A slight problem: the wire included soldered onto the voltmeter circuit board is stranded, not easily plugged into breadboards, and it is too long. Desoldered the stranded wire, can reused for other projects, and soldered on some solid-core. The G and V pins are oriented opposite to the breadboard's + and - rails, but with some careful twisting using pliers the leads can be reshaped to connect correctly, and the IN lead (here the yellow wire) wired to VCC to see the USB power supply voltage, 4.93 V:

![Voltmeter on USB breadboard](https://user-images.githubusercontent.com/26856618/34706922-42744548-f4bf-11e7-9bf3-b512bf032ee9.png)

Plug in a blue pill, and it drops to 4.83 V:

![Voltmeter dropping with blue pill](https://user-images.githubusercontent.com/26856618/34706939-4e562584-f4bf-11e7-966d-82d3469b8bca.png)

## Adding a micro USB port to the Little Ditzel

Another improvement I'd wanted to make using these micro USB breakout boards is to add a proper port to the device implemented in *[The smart bulb / smart switch dilemma: smartening up a dumb wall switch](https://satoshinm.github.io/blog/171209_wallswitch_the_smart_bulb_smart_switch_dilemma_smartening_up_a_dumb_wall_switch.html)*. At the time I didn't have these breakouts, so I desparately split open a USB cable and soldered the wires onto the board, but as the board moved around during usage, the wires became damaged, leading to this ugly mess on the right:

![Ugly soldered USB cable wires](https://user-images.githubusercontent.com/26856618/34707619-5a2217d4-f4c3-11e7-92e1-d65a1d4571ae.png)

A perfect opportunity to improve it, adding a real port to plug a complete USB cable into. First I soldered the headers onto the USB breakout board, but I removed the header spacers:

![Soldering breakout](https://user-images.githubusercontent.com/26856618/34706349-e6cb9960-f4bb-11e7-8f37-35eb3c7ba961.png)

This allows the breakout board to become flush with the main board, for mechanical stability. The VCC and GND wires pins are soldered from the bottom, onto wires going to the same place as the previous cable. Here's how it looks from the top:

![Micro USB on wallswitch](https://user-images.githubusercontent.com/26856618/34706388-36e790de-f4bc-11e7-9010-ac9fffe9a781.png)

The port on the bottom right replaces the mess on the top right (but I left both on for now). Let's see if it works in production:

![Wallswitch detecting power from newly soldered Micro USB port](https://user-images.githubusercontent.com/26856618/34706405-5e50c230-f4bc-11e7-89ee-f408ef9daee0.png)

Success! 

Now for the semi-bad news. I can't actually use this newly-soldered port in practice, yet, because it would take up my last micro USB cable. These things are in short supply in my inventory. I did order some more, three of these:

* [Micro USB Cable for arduino with DUE R3 D1 mini NodeMcu V3 TP4056 18650 Board](https://www.aliexpress.com/item/Micro-USB-Cable-for-arduino-with-DUE-R3-D1-mini-NodeMcu-V3-TP4056-18650-Board/32731302411.html) for $0.80

...58 days ago. Hasn't arrived yet. One of the downsides with Aliexpress, you never know if it'll arrive quickly or take a very long time. Patience is key.

## USB-B ports, or "full-sized female USB"

USB has a bunch of different types of connectors, you can read all about them on Wikipedia: [USB#Host and device interface receptacles](https://en.wikipedia.org/wiki/USB#Host_and_device_interface_receptacles). Here's their photo of the common connector types:

![Wikipedia USB connector photo](https://user-images.githubusercontent.com/26856618/34707013-db64ba44-f4bf-11e7-9fbe-8623f85af84d.png)

Not shown is the newer [USB-C](https://en.wikipedia.org/wiki/USB-C), which is gaining ground, but until then micro-B seems to be leading the pack as a device connector. On the host side, USB type-A, but again it is ceding ground to USB-C. Mini USB is largely obsoleted by micro USB, I have more mini USB cables than I'd like to admit (and in fact tore one up for the USB-A port on the wallswitch board, see above):

![Mini USB cables](https://user-images.githubusercontent.com/26856618/34707146-af527256-f4c0-11e7-8153-722c4c9d4d1e.png)

But there's another connector type, many may have forgotten, worth a second look: USB type-B. You may have seen it on printers. It is a bulkier square shape, and I happen to have four such USB type-B to USB type-A cables:

![USB-B cables](https://user-images.githubusercontent.com/26856618/34707218-1d301ad0-f4c1-11e7-81a8-89b0000bf41c.png)

This connector is older and larger, but may be more robust than the fragile micro USB. Haven't thought of the full-sized USB-B in years, but was brought back to my attention by this Kickstarter project: [ZeptoBit: Optically Isolated USB to Serial Adapter](https://www.kickstarter.com/projects/902741881/optically-isolated-usb-to-serial-adapter?ref=discovery), which offers both a Micro USB and USB-B connector. Featured in [EEVblog #1049 - Mailbag](https://www.youtube.com/watch?v=eBzixNYF5K4&t=10s), the creator says he prefers USB-B so it's not as easy to knock off your workbench. Fair point, perhaps I put these surplus USB-B cables to good use.

So I ordered some of these: [10Pcs/lot Usb Connector B Type 180-Degree Vertical USB Female Connector for Printer Socket Square Head Connector Free Shipping](https://www.aliexpress.com/item/50Pcs-lot-Usb-Connector-B-Type-180-Degree-Vertical-USB-Female-Connector-for-Printer-Socket-Square/32708296836.html), $1.58/10. Note they are 180° connectors, not right-angle 90°. Aliexpress has the 90° connectors but they are much more expensive. With the 180° connectors, the cable will plug straight into the connector, or perpendicular to the PCB. Awaiting arrival, then I'll probably switch over some of my projects to using USB-B connectors and cables for power instead.

## Current limitations

I haven't gone into the prerequisites necessary for increasing the maximum current allowed on these ports. Hasn't been a problem for my projects, but some useful reference material: [StackExchange: How to get more than 100mA from a USB port](https://electronics.stackexchange.com/questions/5498/how-to-get-more-than-100ma-from-a-usb-port), and also [USB Power Delivery](https://en.wikipedia.org/wiki/USB#PD).

## Summary

Gone are the days of a myriad of incompatible wall power adapters, for many projects powering with USB is sufficient, and breakout boards are an easy way to do this without a custom-designed PCB. The choice comes down to the type of USB connector: USB-C is the newest, but micro USB is the most popular, and full-sized USB type B may be a good choice for making more robust connections.

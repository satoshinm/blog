# Triple USB-to-serial adapter using STM32 blue pill

by snm, December 23rd, 2017

With the USB port for device mode, one could present a virtual serial port over USB, connected to a real serial port. I already have one of these devices (in fact, a USB-to-serial adapter (or a USB-to-SWD adapter) was required to program the blue pill from a PC), but it's a simple yet useful application worth touching on.

FTDI makes a [FT232R - USB UART IC], specifically for interfacing USB to serial. The chip is about [$4.50/1 on Digi-Key](https://www.digikey.com/products/en/integrated-circuits-ics/interface-controllers/753?k=FT232R), so Chinese manufacturers have taken to using alternatives: the [SiLabs CP2102](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers) or [Olimex CH430](https://www.olimex.com/Products/Breadboarding/BB-CH340T/resources/CH340DS1.PDF). I've seen these chips on ESP8266 boards including those I used in *[The smart bulb / smart switch dilemma: smartening up a dumb wall switch](https://satoshinm.github.io/blog/171209_wallswitch_the_smart_bulb_smart_switch_dilemma_smartening_up_a_dumb_wall_switch.htm)*. Requires installing their own drivers, but the hardware is cheaper. Case in point: [6Pin USB 2.0 to TTL UART Module Serial Converter CP2102 Replace Ft232 Adapter Module](https://www.aliexpress.com/item/6Pin-USB-2-0-to-TTL-UART-Module-Serial-Converter-CP2102-STC-Replace-Ft232/32364013343.html) for $1.20 on Aliexpress.

However, I use a genuine FT232, on this board: [SparkFun USB to Serial Breakout - FT232RL](https://www.sparkfun.com/products/12731), a steep $14.95. Didn't know of the cheaper alternatives at the time. But this converter works well, so I don't regret it.

Supposedly there are USB-to-serial adapters available based on the STM32. It's a practical application of the blue pill, but may not be the most competitive to alternatives.

The Black Magic Probe firmware is a great alternative, since it provides a composite USB device with a virtual serial port connecting to a UART (and another for GDB debugging SWD/JTAG). What are the pins used? Check [platform.h](https://github.com/blacksphere/blackmagic/blob/master/src/platforms/stlink/platform.h):

```c
#define USBUSART_PORT GPIOA
#define USBUSART_TX_PIN GPIO2
```

which is PA2 for TX. According to [this article](https://medium.com/@paramaggarwal/converting-an-stm32f103-board-to-a-black-magic-probe-c013cf2cc38c), RX is PA3 even though I couldn't find it in code. To connect a Black Magic Probe's virtual serial port to another blue pill target to flash it:

| Black Magic Probe (UART) | Blue Pill target |
| ------------------------ | ---------------- |
| A2 (TX) | A10 |
| A3 (RX) | A9 | 


then try flashing the target through this port:

```sh
stm32loader.py -p /dev/cu.usbmodemE3C896E3 -e -w -v stm32/pill_blink/cubemx/build/pill_blink.bin
```

Works well. Now I can save a USB port and get rid of my standalone USB-to-serial adapter in favor of the Black Magic Probe's built-in functionality.

## USB virtual serial, and what it has to do with modems

Some background on how USB implements what we see as a serial port on the PC.

```c
static const struct usb_device_descriptor dev = {
    .bLength = USB_DT_DEVICE_SIZE,
    .bDescriptorType = USB_DT_DEVICE,
    .bcdUSB = 0x0200,
    .bDeviceClass = 0xEF,       /* Miscellaneous Device */
    .bDeviceSubClass = 2,       /* Common Class */
    .bDeviceProtocol = 1,       /* Interface Association */
```

What is device class/protocol? FTDI has a useful document: [Simplified Description of USB Device Enumeration Technical Note TN_113](http://www.ftdichip.com/Support/Documents/TechnicalNotes/TN_113_Simplified%20Description%20of%20USB%20Device%20Enumeration.pdf). Quoting their description:

> 4.3 bDeviceClass, bDeviceSubClass and bProtocol

> The bDeviceClass of device defines the device type e.g. a USB Mouse is a Human Interface Device (HID) class device. This is given a hex value of 0x03.

> More complex devices such as Communication Device Class (CDC) may also use a sub class to break down the device type into a smaller group.

> bProtocol is used to qualify the sub class.

curious, the HID mouse example from libopencm3 uses a class of 0, not 3. Aha, Appendix C has the answer, class 0x00 means "USE Class information in the Interface Descriptors". The official definitions are found at USB.org: [USB Class Codes](http://www.usb.org/developers/defined_class):

> There are two places on a device where class code information can be placed.One place is in the Device Descriptor, and the other is in Interface Descriptors.

From the table, 03h for HID (Human Interface Device) is for use in the interface descriptor, so is the FTDI documentation incorrect? Some classes can be used in both the device and interface descriptor. Anyways, the pill_serial example (and BMP) uses EFh, "miscellaneous" class, with 02h subclass (common class), and 01h protocol: 

> Interface Association Descriptor. The usage of this class code triple is defined in the Interface Association Descriptor ECN that is provided on www.usb.org . This class code may only be used in Device Descriptors.

Why not class 02h (Communications and CDC Control) or 0Ah (CDC-Data)? The comments say:

```c
/* This file implements a the USB Communications Device Class - Abstract
 * Control Model (CDC-ACM) as defined in CDC PSTN subclass 1.2.
```

Part of the zip file linked on [USB-IF Device Class Documents](http://www.usb.org/developers/docs/devclass_docs/)


> [Class definitions for Communication Devices 1.2](http://www.usb.org/developers/docs/devclass_docs/CDC1.2_WMC1.1_012011.zip) (.zip file format, size 3.42 MB) the components of CDC 1.1 have been reorganized as five separate documents and associated errata:

* ATM120.pdf -- CDC Subclass for Asynchronous Transfer Mode Devices
* CDC120-20101103-track.pdf - CDC Subclass for Communications Devices
* CDC120-Errata1.pdf ECM120.pdf - CDC Subclass for Ethernet Control Model Devices
* ISDN120.pdf -- CDC Subclass for ISDN Devices
* PSTN120.pdf -- CDC Subclass for PSTN devices
* WMC110-20101103-track.pdf - CDC Subclass for Wireless Mobile Communication Devices 1.1
* WMC110-Errata1.pdf and CDC v1.2 WMC v1.1 Errata 1 Adopters Agreement

That's a lot of documents. [ATM](https://en.wikipedia.org/wiki/Asynchronous_Transfer_Mode) is "a telecommunications concept defined by ANSI and ITU (formerly CCITT) standards for carriage of a complete range of user traffic, including voice, data, and video signals", ISDN is the [Integrated Services Digital Network](https://en.wikipedia.org/wiki/Integrated_Services_Digital_Network), PSTN is the [Public Switched Telephone Network](https://en.wikipedia.org/wiki/Public_switched_telephone_network). How did the telephone system get brought into this? It is a useful abstraction, as the specification includes:

* Direct Line Control Model, for modem devices directly controlled by the USB host
* Abstract Control Model, for modem devices controlled through a serial command set
* Telephone Control Model, for voice telephony devices.

The abstract control model, used for modems controlled through.. *serial*. Seems like a roundabout way to get serial, but that's how it was done. And this explains why my computer recognizes these devices as "USB modems", with /dev/tty.usbmodem device nodes.

But is there another mechanism for USB serial? The FTDI 232RL board I have shows up as /dev/tty.usbserial. Trying to find the difference, found [choosing between /dev/tty.usbserial vs /dev/cu.usbserial](https://stackoverflow.com/questions/37688257/choosing-between-dev-tty-usbserial-vs-dev-cu-usbserial#37688347) which explains the difference between /dev/tty (teleype) and /dev/cu (callout) devices, with its relation to signalling DCD. Back to the /usbserial vs usbmodem distinction, turns out Linux has the same distinction: /dev/ttyUSB vs /dev/ttyACM, Samuel Tardieu sheds some light in [What is the difference between /dev/ttyUSB and /dev/ttyACM?](https://rfc1149.net/blog/2013/03/05/what-is-the-difference-between-devttyusbx-and-devttyacmx/):

> When developping on a USB-enabled embedded microcontroller that needs to exchange data with a computer over USB, it is tempting to use a standardized way of communication which is well supported by virtually every operating system. This is why most people choose to implement CDC/PSTN with ACM (did you notice that the Linux kernel driver for /dev/ttyACM0 is named cdc_acm?) because it is the simplest way to exchange raw data.

Direct Line Control would be exchanging data with the modem through an audio class, Abstract Control lets the modem perform hardware analog functions, as instructed to over serial, but this conveniently happens to be useful for non-modem purposes.

In contrast, ttyUSB devices are proprietary UART to USB bridges. FTDI and Prolific have their own. With other vendors you may need to install custom drivers, e.g. SiLabs [CP2102 drivers](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers) which I mentioned in *[STM32 Blue Pill ARM development board first look: from Arduino to bare metal programming](https://satoshinm.github.io/blog/171212_stm32_blue_pill_arm_development_board_first_look_bare_metal_programming.html)* as being present on some ESP8266 boards and inexpensive USB-to-serial converters. Installing proprietary drivers is a pain, and trying to implement to the spec of a 3rd party proprietary driver runs the risk of incidents like [FTDI admits to bricking innocent users' chips in silent update](http://www.zdnet.com/article/ftdi-admits-to-bricking-innocent-users-chips-in-silent-update/). [Hackaday: Watch that Windows Update: FTDI drivers are killing fake chips](https://hackaday.com/2014/10/22/watch-that-windows-update-ftdi-drivers-are-killing-fake-chips/). Given this, using the USB CDC ACM instead seems reasonable.

## Three serial ports at the same time

USB-to-serial is cool and all, but how about we take it a step further? The STM32 has three UARTs, so how about a USB to three serial ports? We could call it USB-to-serialx3 or USB to triple serial. Map each of the serials to a virtual serial port over USB using a composite device. Then this blue pill serial adapter would be three times more useful than the garden variety bargain basement singular USB-to-serial, in theory. 

Refer to the [libopencm3-examples](https://github.com/libopencm3/libopencm3-examples). The [stm32-h103/usb_cdcacm](https://github.com/libopencm3/libopencm3-examples/tree/master/examples/stm32/f1/stm32-h103/usb_cdcacm) example looks instructive, as it "implements a USB CDC-ACM device (aka Virtual Serial Port) to demonstrate the use of the USB device stack", Atmel's [Migrating from RS-232 to USB Bridge Specification application note (2003)](http://www.atmel.com/images/doc4322.pdf) explains how to use the Communications Device Class, Abstract Control Model, to implement a "Virtual COM port". That's what we want. For important nuances and gotchas in using these examples, see my earlier experience with developing [pill_blink](https://github.com/satoshinm/pill_blink).

Build and flash the cdcacm.bin binary, then connect the target blue pill to your PC over USB. It enumerates correctly:

```sh
 $ system_profiler SPUSBDataType
 
         CDC-ACM Demo:

          Product ID: 0x5740
          Vendor ID: 0x0483  (STMicroelectronics)
          Version: 2.00
          Serial Number: DEMO
          Speed: Up to 12 Mb/sec
          Manufacturer: Black Sphere Technologies
          Location ID: 0x14200000 / 7
          Current Available (mA): 500
          Current Required (mA): 100
          Extra Operating Current (mA): 0
```

and shows up in /dev as cu.usbmodemDEM1. Connect to this serial port using GNU Screen:

```sh
screen -L /dev/cu.usbmodemDEM1 115200
```

type and you see the characters you typed echoed back. To exit screen, Ctrl-A, K, Y. Nice example. It could use more LEDs, though. As usual, the example uses the wrong LED for our board: PC12 but the blue pill uses PC13 for the built-in LED. Change it:

```diff
--- a/examples/stm32/f1/stm32-h103/usb_cdcacm/cdcacm.c
+++ b/examples/stm32/f1/stm32-h103/usb_cdcacm/cdcacm.c
@@ -242,16 +242,16 @@ int main(void)
 
        rcc_periph_clock_enable(RCC_GPIOC);
 
-       gpio_set(GPIOC, GPIO11);
+       gpio_set(GPIOC, GPIO13);
        gpio_set_mode(GPIOC, GPIO_MODE_OUTPUT_2_MHZ,
-                     GPIO_CNF_OUTPUT_PUSHPULL, GPIO11);
+                     GPIO_CNF_OUTPUT_PUSHPULL, GPIO13);
 
        usbd_dev = usbd_init(&st_usbfs_v1_usb_driver, &dev, &config, usb_strings, 3, usbd_control_buffer, sizeof(usbd_control_buffer));
        usbd_register_set_config_callback(usbd_dev, cdcacm_set_config);
 
        for (i = 0; i < 0x800000; i++)
                __asm__("nop");
-       gpio_clear(GPIOC, GPIO11);
+       gpio_clear(GPIOC, GPIO13);
 
        while (1)
                usbd_poll(usbd_dev);
```

Now when the USB cable is plugged in, the PC13 start starts off, then flashes quickly and turns on once connected to the host. 

Echoing back what we typed over the virtual serial port isn't too useful, of course. What would be more useful is bridging the USB virtual serial port to a real UART (or USART). Refer to the [stm32-h103/usart](https://github.com/libopencm3/libopencm3-examples/blob/master/examples/stm32/f1/stm32-h103/usart/usart.c) example. Make the same PC13 LED change:

```diff
--- a/examples/stm32/f1/stm32-h103/usart/usart.c
+++ b/examples/stm32/f1/stm32-h103/usart/usart.c
@@ -86,9 +86,9 @@ static void usart_setup(void)
 
 static void gpio_setup(void)
 {
-       /* Set GPIO12 (in GPIO port C) to 'output push-pull'. */
+       /* Set GPIO13 (in GPIO port C) to 'output push-pull'. */
        gpio_set_mode(GPIOC, GPIO_MODE_OUTPUT_2_MHZ,
-                     GPIO_CNF_OUTPUT_PUSHPULL, GPIO12);
+                     GPIO_CNF_OUTPUT_PUSHPULL, GPIO13);
 }
 
 int main(void)
@@ -101,7 +101,7 @@ int main(void)
 
        /* Blink the LED (PC12) on the board with every transmitted byte. */
        while (1) {
-               gpio_toggle(GPIOC, GPIO12);     /* LED on/off */
+               gpio_toggle(GPIOC, GPIO13);     /* LED on/off */
                usart_send_blocking(USART1, c + '0');   /* USART1: Send byte. */
                usart_send_blocking(USART2, c + '0');   /* USART2: Send byte. */
                usart_send_blocking(USART3, c + '0');   /* USART3: Send byte. */
```

This example initializes each of the three USARTs, and sends incrementing ASCII digits, blinking the LED for each byte. Where are the UARTS, what pins? [libopencm3/stm32/f1/gpio.h](https://github.com/libopencm3/libopencm3/blob/master/include/libopencm3/stm32/f1/gpio.h#L335) defines thusly:

| Function | Pin and Port |
| -------- | ------------ |
| USART3 TX | PB10 | 
| USART3 RX | PB11 |
| USART2 TX | PA2 |
| USART2 RX | PA3 |
| USART1 TX | PA9 |
| USART1 RX | PA10 |

PA9/PA10 looks familiar... this is the serial port (USART1) used to reflash the firmware in BOOT0=1 mode. I already have it connected through the BMP, so try to connect through it using screen, specifying the baud rate from the example:

```sh
screen /dev/cu.usbmodemE3C896E3 38400
```

as expected, it continuously prints "12345678901234567890123456789012345678901234567890123456789012345678901234567890" then a newline and repeats indefinitely. So this port is working. Typing does nothing since RX is not configured, only USART1 TX PA9, connected to Black Magic Probe's PA3. Try the other ports: connect Black Magic Probe's PA3 to the target blue pill's A2, and then disconnect and connect to PB10. In all three cases, the characters are received. So far so good.

The BMP's [usbacm.c](https://github.com/blacksphere/blackmagic/blob/master/src/platforms/common/cdcacm.c) is also a useful example. Doesn't while(1) usbd_poll(), instead has a USB_ISR (interrupt service routine). The [usart_irq](https://github.com/libopencm3/libopencm3-examples/blob/master/examples/stm32/f1/stm32-h103/usart_irq/usart_irq.c) example also uses an ISR, usart1_isr, but instead of usbd_poll() it checks why they were called. Testing the usart_irq example (modifying to blink PC13 as usual):

```sh
screen -L  /dev/cu.usbmodemE3C896E3 230400
```

type and the LED blinks and the device echos back. Next up: [usart_irq_printf](https://github.com/libopencm3/libopencm3-examples/tree/master/examples/stm32/f1/stm32-h103/usart_irq_printf). Change PC13, same screen command, this example continuously prints hello world every 10 ms in the 1 ms sys_tick():

```
Hello World! 2865 28.650560 28.650000
Hello World! 2866 28.660561 28.660000
Hello World! 2867 28.670561 28.670000
Hello World! 2868 28.680561 28.680000
Hello World! 2869 28.690561 28.690000
Hello World! 2870 28.700562 28.700000
Hello World! 2871 28.710562 28.710000
Hello World! 2872 28.720562 28.720000
Hello World! 2873 28.730562 28.730000
Hello World! 2874 28.740562 28.740000
Hello World! 2875 28.750563 28.750000
Hello World! 2876 28.760563 28.760000
Hello World! 2877 28.770563 28.770000
Hello World! 2878 28.780563 28.780000
Hello World! 2879 28.790564 28.790000
```

Last usart example, [usart_printf](https://github.com/libopencm3/libopencm3-examples/tree/master/examples/stm32/f1/stm32-h103/usart_printf), is almost exactly the same except not using IRQs. Many other examples also use the USART.

Now the next step could be to combine the cdcacm and usart examples, building a USB-to-serial bridge from scratch. However, I found it easier to use an existing implementation in the Black Sphere Probe and modify it appropriately. To simplify, removed DFU update mode, removed JTAG/SWD debugging, and only left its USB-to-serial code. Test with a standalone USB-to-serial adapter, wire TXD to PA3, and RXD to PA2:

![Testing the USB-to-serial adapter blue pill (pill_serial) with a Sparkfun USB-to-serial adapter](https://user-images.githubusercontent.com/26856618/34323739-88113b52-e80c-11e7-8b3a-99bb21cd914b.png)

connect to each virtual USB serial port on the PC, type keystrokes and watch the characters flow.

With some work, stripped the firmware of all but the USB-to-serial converter, and enhanced it to support up all three USARTs. The result is what I call `pill_serial`, available here: 

* **[https://github.com/satoshinm/pill_serial](https://github.com/satoshinm/pill_serial)**


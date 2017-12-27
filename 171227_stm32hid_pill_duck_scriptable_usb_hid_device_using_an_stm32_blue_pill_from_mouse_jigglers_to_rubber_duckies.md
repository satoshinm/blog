# Pill Duck: Scriptable USB HID device using an STM32 blue pill, from mouse jigglers to rubber duckies

by snm, December 27th, 2017

* auto-gen TOC:
{:toc}

One of the possible useful applications of the impressively inexpensive STM32F103 "blue pill" ARM Cortex-M3 board is to use it as a USB Human Interface Device (HID), where we can program and replay mouse movements or keyboard keystrokes.

Other solutions are possible for automating keyboard input. It can be done in software, and also a low-tech hardware hack was featured in *The Simpsons*, [S07E07](https://www.youtube.com/watch?v=R_rF4kcqLkI):

![Simpsons S07E07 drinking bird](https://user-images.githubusercontent.com/26856618/34393915-e17ac7ac-eb0a-11e7-8aac-d9b177fbca13.png)

This is [Homer's typing bird](https://www.youtube.com/watch?v=R_rF4kcqLkI), let's take a closer look:

![Closeup of typing bird](https://user-images.githubusercontent.com/26856618/34393931-f3149dda-eb0a-11e7-9f32-a41ea23193a8.png)

[Carlos Fernandez](https://www.youtube.com/watch?v=iapECJKx4k0) made this for real way back in 2010, but pressing the "Space" key instead of "Y" as Homer did. YTEngineer [explains how the drinking/dipping bird works](https://www.youtube.com/watch?v=cwyDsHG-ymQ) using the laws of thermodynamics. However, it is an inflexible solution to automating typing, only supporting one key repeated indefinitely.

Since the advent of the [Universal Serial Bus](https://en.wikipedia.org/wiki/USB), it is now possible and easy to turn a suitable microcontroller into almost any USB peripheral you want! In this article, I'll use a STM32F103 board which can be had for a couple bucks, and make it show up to a PC as a composite device consisting of a USB keyboard, USB mouse, and USB modem serial port. I call it the *Pill Duck*, for reasons which will become apparent later.

This post is a detailed set of rough notes, if all you want is to try it out, skip to the source repository here:

* **[https://github.com/satoshinm/pill_duck](https://github.com/satoshinm/pill_duck)**

Additional blog posts for background information and previous related projects:

* *[2017/12/12 STM32 Blue Pill ARM development board first look: bare metal programming](https://satoshinm.github.io/blog/171212_stm32_blue_pill_arm_development_board_first_look_bare_metal_programming.html)*
* *[2017/12/23 JTAG/SWD debugging via Black Magic Probe on an STM32 blue pill and blinking a LED using STM32CubeMX, libopencm3, and bare metal C](https://satoshinm.github.io/blog/171223_jtagswdpillblink_jtagswd_debugging_via_black_magic_probe_on_an_stm32_blue_pill_and_blinking_a_led_using_stm32cubemx_libopencm3_and_bare_metal_c.html)*
* *[2017/12/23 Triple USB-to-serial adapter using STM32 blue pill](https://satoshinm.github.io/blog/171223_stm32serial_triple_usb-to-serial_adapter_using_stm32_blue_pill.html)*

Without further ado, let's begin. 

## Mouse Jiggler demo

Try the libopencm3 example [stm32/f1/other/usb_hid](https://github.com/libopencm3/libopencm3-examples/tree/master/examples/stm32/f1/other/usb_hid). Enumerates as follows:

```sh
usb_hid $ system_profiler SPUSBDataType

        HID Demo:

          Product ID: 0x5710
          Vendor ID: 0x0483  (STMicroelectronics)
          Version: 2.00
          Serial Number: DEMO
          Speed: Up to 12 Mb/sec
          Manufacturer: Black Sphere Technologies
          Location ID: 0x14200000 / 33
          Current Available (mA): 500
          Current Required (mA): 100
          Extra Operating Current (mA): 0
```

and it shows up as a USB mouse, moving the cursor back and forth 30 px. That is, it's a simple hardware [mouse jiggler](https://mousejiggler.codeplex.com). Mouse jigglers could be useful to [keep your computer awake](https://www.pcworld.com/article/203917/Keep_Your_Computer_Awake_with_Mouse_Jiggler.html), according to PC World. Maybe if you don't have to or want to configure the OS software to not sleep? Not too interesting to me seems kinda silly, but if you are interested in more of this see also [Dr Phil - Mouse Jiggler: Offense and Defense](https://www.youtube.com/watch?v=5Nk6iDryW0Y). It's a good simple HID demo, at least.

## Building a Rubber Ducky clone

[USB Rubber Ducky](https://github.com/hak5darren/USB-Rubber-Ducky) retails for [$44.99](https://hakshop.com/products/usb-rubber-ducky-deluxe), presents itself as a USB keyboard to a computer. You can program it in a scripting language, to send keystrokes for various purposes (tons of [payloads](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Payloads)). The official firmware is written using Atmel AVR Studio 5, for a "fast 60 MHz 32-bit processor". From looking at the project files, appears they are targeting a [Microchip AT32UC3B0256](http://www.microchip.com/wwwproducts/en/AT32UC3B0256) (AVR was [bought by Microchip](https://makezine.com/2016/01/25/why-im-excited-that-microchip-is-buying-atmel/) in 2016):

> The high-performance, low-power 32-bit AVR RISC-based microcontroller combines 256KB flash memory, 32KB SRAM, full-speed (12 Mbps) USB Mini-host + device, and I2S. The MCU achieves 83 Dhrystone MIPS (DMIPS) at 60MHz and consumes only 23mA at 3.3 volts.

The blue pill is also 32-bit RISC (albeit ARM not AVR), but faster at 72 MHz, it has USB device (but not host), sadly a smaller amount of flash memory (64 KB or 128 KB), and only 20 KB SRAM. Can't complain for the price. Not drastically worse or better, comparable. Other people have realized the Rubber Ducky can be realized more inexpensively. Adam Eaton in [Building a USB Rubber Ducky for $7](https://medium.com/@EatonChips/building-a-usb-rubber-ducky-for-7-c851aae30a1d) used an [Adafruit Trinket](https://www.adafruit.com/product/1501) ($6.95), which is built on a the Atmel [ATtiny85](http://www.microchip.com/wwwproducts/en/ATtiny85): a measly 8 KB flash, 512 bytes EEPROM, 512 bytes SRAM. Teeny! The blue pill seems to be a much better value. Adam's ATtiny85-based "rubber ducky" looks similar on the surface, but its doesn't appear compatible with the Rubber Ducky scripting language and payloads; you write code in Arduino then upload it. Compatibility with this existing ecosystem would be nice.

### USB CDC ACM (virtual serial port)

To help interactively use the HID device, it would be nice to also provide a virtual serial port over USB. The user could connect to the serial port and send commands which would be sent from the HID device to the host PC. For more about USB serial, which is implemented using the CDC ACM (Communications Device Class - Abstract Control Model) specification of USB, see *[Triple USB-to-serial adapter using STM32 blue pill](https://satoshinm.github.io/blog/171223_stm32serial_triple_usb-to-serial_adapter_using_stm32_blue_pill.html)*, where I introduce `pill_serial`. Add just one serial port to the HID example, so both the HID and serial show up over USB. How can we do this?

The mouse HID example uses USB device descriptor as follows:

```c
const struct usb_device_descriptor dev_descr = {
    .bLength = USB_DT_DEVICE_SIZE,
    .bDescriptorType = USB_DT_DEVICE,
    .bcdUSB = 0x0200,
    .bDeviceClass = 0,
    .bDeviceSubClass = 0,
    .bDeviceProtocol = 0,
```

versus the pill_serial example:

```c
static const struct usb_device_descriptor dev = {
    .bLength = USB_DT_DEVICE_SIZE,
    .bDescriptorType = USB_DT_DEVICE,
    .bcdUSB = 0x0200,
    .bDeviceClass = 0xEF,       /* Miscellaneous Device */
    .bDeviceSubClass = 2,       /* Common Class */
    .bDeviceProtocol = 1,       /* Interface Association */
```

Add another interface, based on pill_serial's USART3. Leave the HID mouse at interface 0, add UART comm at 1 and data at 2. Most is straightforward to merge, but the USB_DT_CONFIGURATION descriptor has bmAttributes = 0xC0 in the HID demo, versus bmAttributes = 0x80 in the serial demo. [Beyond Logic USB in a Nutshell](http://www.beyondlogic.org/usbnutshell/usb5.shtml) explains the bitmap attributes. D7 reserved is set to 1 in both values, but the HID demo sets D6 to 1 meaning self-powered, whereas the serial device did not. Left it at 0xC0. Add all the CDC ACM code for the UART, then the two other interfaces:

```c
const struct usb_interface ifaces[] = {{
    .num_altsetting = 1,
    .altsetting = &hid_iface,
}, {
    .num_altsetting = 1,
    .iface_assoc = &uart_assoc,
    .altsetting = uart_comm_iface,
}, {
    .num_altsetting = 1,
    .altsetting = uart_data_iface,
}};
```

rebuild and flash and plug into USB. After a few seconds the HID device appears and so does the virtual serial port, as `/dev/tty.usbmodemDEM2` (truncated part of the serial number iSerialNumber = 3, corresponding to usb_strings 3rd element "DEMO"):

```sh
crw-rw-rw-  1 root  wheel   34, 0x0000016c Dec 24 14:32 /dev/tty.usbmodemDEM2
```

but it doesn't do anything yet, need callbacks. In the config callback set by `usbd_register_set_config_callback()`, configure both the HID device and CDC ACM. The control callback set by `usbd_register_control_callback()` is used for setting the line coding, meaning the baud rate, parity, stop bits; in pill_serial this was used to set the physical USART port to what the host PC via USB requested. No need for that here, but implement the callback anyway, maybe can delete it later. The most important callbacks are of course for data input/output. For testing, toggling the built-in LED for each character received:

```c
int main(void)
...
        rcc_periph_clock_enable(RCC_GPIOC);
        gpio_set_mode(GPIOC, GPIO_MODE_OUTPUT_2_MHZ,
                GPIO_CNF_OUTPUT_PUSHPULL, GPIO13);
        gpio_set(GPIOC, GPIO13);

...

static void usbuart_usb_out_cb(usbd_device *dev, uint8_t ep)
{
        (void)ep;

        char buf[CDCACM_PACKET_SIZE];
        int len = usbd_ep_read_packet(dev, CDCACM_UART_ENDPOINT,
                                        buf, CDCACM_PACKET_SIZE);


        for(int i = 0; i < len; i++)
                gpio_toggle(GPIOC, GPIO13);
}
```

It works, now we can wire it up to control the HID device from the serial port, for development. Here's a start, send characters over the virtual serial port to change the mouse jiggler direction:

```c
static int dir = 1;
static bool jiggler = true;
void sys_tick_handler(void)
{
        static int x = 0;
        uint8_t buf[4] = {0, 0, 0, 0};

        buf[1] = dir;
        if (jiggler) {
                x += dir;
                if (x > 30)
                        dir = -dir;
                if (x < -30)
                        dir = -dir;
        }

        usbd_ep_write_packet(usbd_dev, 0x81, buf, 4);
}

static void usbuart_usb_out_cb(usbd_device *dev, uint8_t ep)
{
        (void)ep;

        char buf[CDCACM_PACKET_SIZE];
        int len = usbd_ep_read_packet(dev, CDCACM_UART_ENDPOINT,
                                        buf, CDCACM_PACKET_SIZE);


        for(int i = 0; i < len; i++) {
                gpio_toggle(GPIOC, GPIO13);
                if (buf[i] == 'a') {
                        dir = -1;
                        jiggler = false;
                } else if (buf[i] == 'd') {
                        dir = 1;
                        jiggler = false;
                } else if (buf[i] == 'w') {
                        dir = 2;
                        jiggler = false;
                } else if (buf[i] == 's') {
                        dir = 0;
                        jiggler = false;
                } else if (buf[i] == 'q') {
                        jiggler = !jiggler;
                }
        }
}
```

But more interesting than controlling a mouse, would be controlling a keyboard!

### Adding a USB HID keyboard

[Universal Serial Bus: HID Usage Tables](http://www.usb.org/developers/hidpage/Hut1_12v2.pdf), table 1, shows the defined "usage pages" for the USB Human Interface Device specification:

![USB HID Usage Pages](https://user-images.githubusercontent.com/26856618/34328991-9d1f6b02-e8a4-11e7-8f7d-d75f4311b576.png)

The pages are groups of related device types (not shown: 91 Arcade Page OOAF Definitions for arcade and coinop related Devices, 92-FEFF Reserved, FF00-FFFF Vendor-defined). The generic desktop page (0x01) is probably the most common, its usage IDs are further subdivided and defined in table 6:

![USB HID Usage IDs for Generic Desktop Page, part 1](https://user-images.githubusercontent.com/26856618/34329015-6df79bdc-e8a5-11e7-9968-974c302a8c6b.png)

![USB HID Usage IDs for Generic Desktop Page, part 2](https://user-images.githubusercontent.com/26856618/34329019-837d34f8-e8a5-11e7-889c-e1c9877de76e.png)

The libopencm3_examples usb_hid demo includes a descriptor with usages for mouse, pointer, button, X, Y, wheel, motion wakeup, and vendor usage 1:

```c
static const uint8_t hid_report_descriptor[] = {
    0x05, 0x01, /* USAGE_PAGE (Generic Desktop)         */
    0x09, 0x02, /* USAGE (Mouse)                        */
    0xa1, 0x01, /* COLLECTION (Application)             */
    0x09, 0x01, /*   USAGE (Pointer)                    */
    0xa1, 0x00, /*   COLLECTION (Physical)              */
    0x05, 0x09, /*     USAGE_PAGE (Button)              */
    0x19, 0x01, /*     USAGE_MINIMUM (Button 1)         */
    0x29, 0x03, /*     USAGE_MAXIMUM (Button 3)         */
    0x15, 0x00, /*     LOGICAL_MINIMUM (0)              */
    0x25, 0x01, /*     LOGICAL_MAXIMUM (1)              */
    0x95, 0x03, /*     REPORT_COUNT (3)                 */
    0x75, 0x01, /*     REPORT_SIZE (1)                  */
    0x81, 0x02, /*     INPUT (Data,Var,Abs)             */
    0x95, 0x01, /*     REPORT_COUNT (1)                 */
    0x75, 0x05, /*     REPORT_SIZE (5)                  */
    0x81, 0x01, /*     INPUT (Cnst,Ary,Abs)             */
    0x05, 0x01, /*     USAGE_PAGE (Generic Desktop)     */
    0x09, 0x30, /*     USAGE (X)                        */
    0x09, 0x31, /*     USAGE (Y)                        */
    0x09, 0x38, /*     USAGE (Wheel)                    */
    0x15, 0x81, /*     LOGICAL_MINIMUM (-127)           */
    0x25, 0x7f, /*     LOGICAL_MAXIMUM (127)            */
    0x75, 0x08, /*     REPORT_SIZE (8)                  */
    0x95, 0x03, /*     REPORT_COUNT (3)                 */
    0x81, 0x06, /*     INPUT (Data,Var,Rel)             */
    0xc0,       /*   END_COLLECTION                     */
    0x09, 0x3c, /*   USAGE (Motion Wakeup)              */
    0x05, 0xff, /*   USAGE_PAGE (Vendor Defined Page 1) */
    0x09, 0x01, /*   USAGE (Vendor Usage 1)             */
    0x15, 0x00, /*   LOGICAL_MINIMUM (0)                */
    0x25, 0x01, /*   LOGICAL_MAXIMUM (1)                */
    0x75, 0x01, /*   REPORT_SIZE (1)                    */
    0x95, 0x02, /*   REPORT_COUNT (2)                   */
    0xb1, 0x22, /*   FEATURE (Data,Var,Abs,NPrf)        */
    0x75, 0x06, /*   REPORT_SIZE (6)                    */
    0x95, 0x01, /*   REPORT_COUNT (1)                   */
    0xb1, 0x01, /*   FEATURE (Cnst,Ary,Abs)             */
    0xc0        /* END_COLLECTION                       */
};
```

To change it to a keyboard, we'll need to add usage page 0x01 (generic desktop page), usage ID 0x06 (keyboard). What is keypad (0x07)? The specs explain the difference: keyboards must have a minimum of 103 keys so you can use them during system boot, but keypads don't meet these requirements and are often "supplementary calculator-style keyboards". So a keyboard is what we want. Gamepads and joysticks could also be interesting to add. But a keyboard is definitely necessary.

How does the Rubber Ducky do it? There's a lot of boilerplate and library code, but [main.c](https://github.com/hak5darren/USB-Rubber-Ducky/blob/master/Firmware/Source/Ducky_HID/src/main.c#L29) includes the header file corresponding to [udi_hid_kbd.c](https://github.com/hak5darren/USB-Rubber-Ducky/blob/33a834b0e19f9d4f995432eb9dbcccb247c2e4df/Firmware/Source/Ducky_HID/src/asf/common/services/usb/class/hid/device/kbd/udi_hid_kbd.c), which defines this report descriptor:

```c
//! HID report descriptor for standard HID keyboard
UDC_DESC_STORAGE udi_hid_kbd_report_desc_t udi_hid_kbd_report_desc = {
	{
				0x05, 0x01,	/* Usage Page (Generic Desktop)      */
				0x09, 0x06,	/* Usage (Keyboard)                  */
				0xA1, 0x01,	/* Collection (Application)          */
				0x05, 0x07,	/* Usage Page (Keyboard)             */
				0x19, 224,	/* Usage Minimum (224)               */
				0x29, 231,	/* Usage Maximum (231)               */
				0x15, 0x00,	/* Logical Minimum (0)               */
				0x25, 0x01,	/* Logical Maximum (1)               */
				0x75, 0x01,	/* Report Size (1)                   */
				0x95, 0x08,	/* Report Count (8)                  */
				0x81, 0x02,	/* Input (Data, Variable, Absolute)  */
				0x81, 0x01,	/* Input (Constant)                  */
				0x19, 0x00,	/* Usage Minimum (0)                 */
				0x29, 101,	/* Usage Maximum (101)               */
				0x15, 0x00,	/* Logical Minimum (0)               */
				0x25, 101,	/* Logical Maximum (101)             */
				0x75, 0x08,	/* Report Size (8)                   */
				0x95, 0x06,	/* Report Count (6)                  */
				0x81, 0x00,	/* Input (Data, Array)               */
				0x05, 0x08,	/* Usage Page (LED)                  */
				0x19, 0x01,	/* Usage Minimum (1)                 */
				0x29, 0x05,	/* Usage Maximum (5)                 */
				0x15, 0x00,	/* Logical Minimum (0)               */
				0x25, 0x01,	/* Logical Maximum (1)               */
				0x75, 0x01,	/* Report Size (1)                   */
				0x95, 0x05,	/* Report Count (5)                  */
				0x91, 0x02,	/* Output (Data, Variable, Absolute) */
				0x95, 0x03,	/* Report Count (3)                  */
				0x91, 0x01,	/* Output (Constant)                 */
				0xC0	/* End Collection                    */
			}
};
```

Quite a mouthful. Frank Zhao demystifies the report descriptor in this tutorial: [Tutorial about USB HID Report Descriptors](http://eleccelerator.com/tutorial-about-usb-hid-report-descriptors/). The recommendedUSB HID Descriptor tool is for Windows only, but it runs in Wine using [WineBottler](http://winebottler.kronenberg.org), just right-click on Dt.exe and open in Wine:

![USB.org's Dt.exe in Wine](https://user-images.githubusercontent.com/26856618/34329890-7a4eca24-e8c6-11e7-93fa-82e0b3ff4ea9.png)

There are included examples of several kinds of HID descriptors. keybrd.hid differs from Rubber Ducky's udi_hid_kbd_report_desc slightly, after `Input (Data, Variable, Absolute)` instead of Input Constant (0x81, 0x81) keybrd.hid includes REPORT_COUNT 1 (95 01) followed by REPORT_SIZE 8 (75 08), then INPUT Cnst,Var,Abs (81 03). Dt.exe supports parsing descriptors too, so I saved the udi_hid_kbd descriptor as binary (amusingly, file(1) detects the type as: udi_hid_kbd.hid: PDP-11 old overlay), then opened it but it wasn't recognized as a .hid file. Lame, but the tool does support writing C header files (.h), if I do that to the included keybrd.hid, this is the C code we get:

```c
char ReportDescriptor[63] = {
    0x05, 0x01,                    // USAGE_PAGE (Generic Desktop)
    0x09, 0x06,                    // USAGE (Keyboard)
    0xa1, 0x01,                    // COLLECTION (Application)
    0x05, 0x07,                    //   USAGE_PAGE (Keyboard)
    0x19, 0xe0,                    //   USAGE_MINIMUM (Keyboard LeftControl)
    0x29, 0xe7,                    //   USAGE_MAXIMUM (Keyboard Right GUI)
    0x15, 0x00,                    //   LOGICAL_MINIMUM (0)
    0x25, 0x01,                    //   LOGICAL_MAXIMUM (1)
    0x75, 0x01,                    //   REPORT_SIZE (1)
    0x95, 0x08,                    //   REPORT_COUNT (8)
    0x81, 0x02,                    //   INPUT (Data,Var,Abs)
    0x95, 0x01,                    //   REPORT_COUNT (1)
    0x75, 0x08,                    //   REPORT_SIZE (8)
    0x81, 0x03,                    //   INPUT (Cnst,Var,Abs)
    0x95, 0x05,                    //   REPORT_COUNT (5)
    0x75, 0x01,                    //   REPORT_SIZE (1)
    0x05, 0x08,                    //   USAGE_PAGE (LEDs)
    0x19, 0x01,                    //   USAGE_MINIMUM (Num Lock)
    0x29, 0x05,                    //   USAGE_MAXIMUM (Kana)
    0x91, 0x02,                    //   OUTPUT (Data,Var,Abs)
    0x95, 0x01,                    //   REPORT_COUNT (1)
    0x75, 0x03,                    //   REPORT_SIZE (3)
    0x91, 0x03,                    //   OUTPUT (Cnst,Var,Abs)
    0x95, 0x06,                    //   REPORT_COUNT (6)
    0x75, 0x08,                    //   REPORT_SIZE (8)
    0x15, 0x00,                    //   LOGICAL_MINIMUM (0)
    0x25, 0x65,                    //   LOGICAL_MAXIMUM (101)
    0x05, 0x07,                    //   USAGE_PAGE (Keyboard)
    0x19, 0x00,                    //   USAGE_MINIMUM (Reserved (no event indicated))
    0x29, 0x65,                    //   USAGE_MAXIMUM (Keyboard Application)
    0x81, 0x00,                    //   INPUT (Data,Ary,Abs)
    0xc0                           // END_COLLECTION
};
```

Study the [Tutorial about USB HID Report Descriptors](http://eleccelerator.com/tutorial-about-usb-hid-report-descriptors/) for how to make a composite USB device descriptor. The example starts with a mouse, which we started with too (usb_hid example in libopencm3), very similar except the libopencm3 example adds a mouse wheel axis (in addition to X and Y), and also this mysterious feature (motion wakeup = wake up the computer when the mouse is moved?):

```c
        0x09, 0x3c, /*   USAGE (Motion Wakeup)              */
        0x05, 0xff, /*   USAGE_PAGE (Vendor Defined Page 1) */
        0x09, 0x01, /*   USAGE (Vendor Usage 1)             */
        0x15, 0x00, /*   LOGICAL_MINIMUM (0)                */
        0x25, 0x01, /*   LOGICAL_MAXIMUM (1)                */
        0x75, 0x01, /*   REPORT_SIZE (1)                    */
        0x95, 0x02, /*   REPORT_COUNT (2)                   */
        0xb1, 0x22, /*   FEATURE (Data,Var,Abs,NPrf)        */
        0x75, 0x06, /*   REPORT_SIZE (6)                    */
        0x95, 0x01, /*   REPORT_COUNT (1)                   */
        0xb1, 0x01, /*   FEATURE (Cnst,Ary,Abs)             */
```

and then I found this nifty webpage: [USB Descriptor and Request Parser](http://eleccelerator.com/usbdescreqparser/), a web-based (JavaScript) descriptor parser from Frank Zhao, so no need for the Windows-based Dt.exe program. The author also recommends the [Beagle USB 12 - Low/Full Speed USB Protocol Analyzer + Sticker](https://www.adafruit.com/product/708), looks interesting: a USB protocol analyzer, but at $400 I'll pass. 

Combining the two as the tutorial describes, using two collections with a different `REPORT_ID` (0x85 xx), 1 for the keyboard 2 for the mouse, gives this bigass descriptor:

```c
0x05, 0x01,        // Usage Page (Generic Desktop Ctrls)
0x09, 0x06,        // Usage (Keyboard)
0xA1, 0x01,        // Collection (Application)
0x85, 0x01,        //   Report ID (1)
0x05, 0x07,        //   Usage Page (Kbrd/Keypad)
0x19, 0xE0,        //   Usage Minimum (0xE0)
0x29, 0xE7,        //   Usage Maximum (0xE7)
0x15, 0x00,        //   Logical Minimum (0)
0x25, 0x01,        //   Logical Maximum (1)
0x75, 0x01,        //   Report Size (1)
0x95, 0x08,        //   Report Count (8)
0x81, 0x02,        //   Input (Data,Var,Abs,No Wrap,Linear,Preferred State,No Null Position)
0x95, 0x01,        //   Report Count (1)
0x75, 0x08,        //   Report Size (8)
0x81, 0x03,        //   Input (Const,Var,Abs,No Wrap,Linear,Preferred State,No Null Position)
0x95, 0x05,        //   Report Count (5)
0x75, 0x01,        //   Report Size (1)
0x05, 0x08,        //   Usage Page (LEDs)
0x19, 0x01,        //   Usage Minimum (Num Lock)
0x29, 0x05,        //   Usage Maximum (Kana)
0x91, 0x02,        //   Output (Data,Var,Abs,No Wrap,Linear,Preferred State,No Null Position,Non-volatile)
0x95, 0x01,        //   Report Count (1)
0x75, 0x03,        //   Report Size (3)
0x91, 0x03,        //   Output (Const,Var,Abs,No Wrap,Linear,Preferred State,No Null Position,Non-volatile)
0x95, 0x06,        //   Report Count (6)
0x75, 0x08,        //   Report Size (8)
0x15, 0x00,        //   Logical Minimum (0)
0x25, 0x65,        //   Logical Maximum (101)
0x05, 0x07,        //   Usage Page (Kbrd/Keypad)
0x19, 0x00,        //   Usage Minimum (0x00)
0x29, 0x65,        //   Usage Maximum (0x65)
0x81, 0x00,        //   Input (Data,Array,Abs,No Wrap,Linear,Preferred State,No Null Position)
0xC0,              // End Collection
0x05, 0x01,        // Usage Page (Generic Desktop Ctrls)
0x09, 0x02,        // Usage (Mouse)
0xA1, 0x01,        // Collection (Application)
0x85, 0x02,        //   Report ID (2)
0x09, 0x01,        //   Usage (Pointer)
0xA1, 0x00,        //   Collection (Physical)
0x05, 0x09,        //     Usage Page (Button)
0x19, 0x01,        //     Usage Minimum (0x01)
0x29, 0x03,        //     Usage Maximum (0x03)
0x15, 0x00,        //     Logical Minimum (0)
0x25, 0x01,        //     Logical Maximum (1)
0x95, 0x03,        //     Report Count (3)
0x75, 0x01,        //     Report Size (1)
0x81, 0x02,        //     Input (Data,Var,Abs,No Wrap,Linear,Preferred State,No Null Position)
0x95, 0x01,        //     Report Count (1)
0x75, 0x05,        //     Report Size (5)
0x81, 0x01,        //     Input (Const,Array,Abs,No Wrap,Linear,Preferred State,No Null Position)
0x05, 0x01,        //     Usage Page (Generic Desktop Ctrls)
0x09, 0x30,        //     Usage (X)
0x09, 0x31,        //     Usage (Y)
0x09, 0x38,        //     Usage (Wheel)
0x15, 0x81,        //     Logical Minimum (-127)
0x25, 0x7F,        //     Logical Maximum (127)
0x75, 0x08,        //     Report Size (8)
0x95, 0x03,        //     Report Count (3)
0x81, 0x06,        //     Input (Data,Var,Rel,No Wrap,Linear,Preferred State,No Null Position)
0xC0,              //   End Collection
0x09, 0x3C,        //   Usage (Motion Wakeup)
0x05, 0xFF,        //   Usage Page (Reserved 0xFF)
0x09, 0x01,        //   Usage (0x01)
0x15, 0x00,        //   Logical Minimum (0)
0x25, 0x01,        //   Logical Maximum (1)
0x75, 0x01,        //   Report Size (1)
0x95, 0x02,        //   Report Count (2)
0xB1, 0x22,        //   Feature (Data,Var,Abs,No Wrap,Linear,No Preferred State,No Null Position,Non-volatile)
0x75, 0x06,        //   Report Size (6)
0x95, 0x01,        //   Report Count (1)
0xB1, 0x01,        //   Feature (Const,Array,Abs,No Wrap,Linear,Preferred State,No Null Position,Non-volatile)
0xC0,              // End Collection

// 141 bytes
```

Update the function to send data in the new format, now with a 1-byte report ID prefix:

```c
void sys_tick_handler(void)
{       
        static int x = 0;
        uint8_t buf[5] = {0, 0, 0, 0, 0};
        
        buf[0] = 2; // mouse

        buf[2] = dir;
        if (jiggler) {
                x += dir;
                if (x > 30) 
                        dir = -dir;
                if (x < -30)
                        dir = -dir;
        }
        
        usbd_ep_write_packet(usbd_dev, 0x81, buf, 4);
}
```

Plug it in and... immediately I get an unrecognized keyboard prompt from the OS:

### Debugging USB keyboard issues

![Unrecognized keyboard](https://user-images.githubusercontent.com/26856618/34330409-1e3f1eb8-e8d1-11e7-8dd8-3a953d3d96e3.png)

simultaneously, the mouse jiggler is enabled. Nice, it basically worked. But I would surely like to avoid that unrecognized keyboard prompt. Is there anyway I can tell the computer what kind of keyboard it is?

As a test, obviously we should try using the [official USB Rubber Ducky's HID descriptor](https://github.com/hak5darren/USB-Rubber-Ducky/blob/33a834b0e19f9d4f995432eb9dbcccb247c2e4df/Firmware/Source/Ducky_HID/src/asf/common/services/usb/class/hid/device/kbd/udi_hid_kbd.c#L101). Added it to the composite device collection, added the report ID, processed through the [USB Descriptor and Request Parser tool](http://eleccelerator.com/usbdescreqparser/), here are the differences between using the usb.org keybrd.hid:

```diff
@@ -61,26 +61,24 @@ static const uint8_t hid_report_descriptor[] = {
        0x75, 0x01,        //   Report Size (1)
        0x95, 0x08,        //   Report Count (8)
        0x81, 0x02,        //   Input (Data,Var,Abs,No Wrap,Linear,Preferred State,No Null Position)
-       0x95, 0x01,        //   Report Count (1)
+       0x81, 0x01,        //   Input (Const,Array,Abs,No Wrap,Linear,Preferred State,No Null Position)
+       0x19, 0x00,        //   Usage Minimum (0x00)
+       0x29, 0x65,        //   Usage Maximum (0x65)
+       0x15, 0x00,        //   Logical Minimum (0)
+       0x25, 0x65,        //   Logical Maximum (101)
        0x75, 0x08,        //   Report Size (8)
-       0x81, 0x03,        //   Input (Const,Var,Abs,No Wrap,Linear,Preferred State,No Null Position)
-       0x95, 0x05,        //   Report Count (5)
-       0x75, 0x01,        //   Report Size (1)
+       0x95, 0x06,        //   Report Count (6)
+       0x81, 0x00,        //   Input (Data,Array,Abs,No Wrap,Linear,Preferred State,No Null Position)
        0x05, 0x08,        //   Usage Page (LEDs)
        0x19, 0x01,        //   Usage Minimum (Num Lock)
        0x29, 0x05,        //   Usage Maximum (Kana)
-       0x91, 0x02,        //   Output (Data,Var,Abs,No Wrap,Linear,Preferred State,No Null Position,Non-volatile)
-       0x95, 0x01,        //   Report Count (1)
-       0x75, 0x03,        //   Report Size (3)
-       0x91, 0x03,        //   Output (Const,Var,Abs,No Wrap,Linear,Preferred State,No Null Position,Non-volatile)
-       0x95, 0x06,        //   Report Count (6)
-       0x75, 0x08,        //   Report Size (8)
        0x15, 0x00,        //   Logical Minimum (0)
-       0x25, 0x65,        //   Logical Maximum (101)
-       0x05, 0x07,        //   Usage Page (Kbrd/Keypad)
-       0x19, 0x00,        //   Usage Minimum (0x00)
-       0x29, 0x65,        //   Usage Maximum (0x65)
-       0x81, 0x00,        //   Input (Data,Array,Abs,No Wrap,Linear,Preferred State,No Null Position)
+       0x25, 0x01,        //   Logical Maximum (1)
+       0x75, 0x01,        //   Report Size (1)
+       0x95, 0x05,        //   Report Count (5)
+       0x91, 0x02,        //   Output (Data,Var,Abs,No Wrap,Linear,Preferred State,No Null Position,Non-volatile)
+       0x95, 0x03,        //   Report Count (3)
+       0x91, 0x01,        //   Output (Const,Array,Abs,No Wrap,Linear,Preferred State,No Null Position,Non-volatile)
        0xC0,              // End Collection
        0x05, 0x01,        // Usage Page (Generic Desktop Ctrls)
        0x09, 0x02,        // Usage (Mouse)
```

Flash and plug in the USB. I still get unrecognized keyboard :(. Could the composite USB be causing incompatibilities? Zhao provides a disclaimer at the end of his tutorial "But since we changed the data structure, your device no-longer supports boot protocol, and hence you will not need to define a protocol." - so to eliminate this as a cause, try commenting-out the mouse descriptor and report ID. Same! Could there be something else to it? The USB vendor/product ID, maybe? Found these IDs in [Ducky_Multi_Payload conf_usb.h](https://github.com/hak5darren/USB-Rubber-Ducky/blob/33a834b0e19f9d4f995432eb9dbcccb247c2e4df/Firmware/Source/Ducky_Multi_Payload/src/config/conf_usb.h#L53):

```c
//! Device definition (mandatory)
//#define  USB_DEVICE_VENDOR_ID             USB_VID_ATMEL
//#define  USB_DEVICE_PRODUCT_ID            USB_PID_ATMEL_AVR_HIDKEYBOARD
#define USB_DEVICE_VENDOR_ID		      0x05ac
#define USB_DEVICE_PRODUCT_ID			  0x2227
#define  USB_DEVICE_MAJOR_VERSION         1
#define  USB_DEVICE_MINOR_VERSION         0
#define  USB_DEVICE_POWER                 100 // Consumption on Vbus line (mA)
#define  USB_DEVICE_ATTR                  \
	(USB_CONFIG_ATTR_REMOTE_WAKEUP|USB_CONFIG_ATTR_BUS_POWERED)
//	(USB_CONFIG_ATTR_SELF_POWERED)
//  (USB_CONFIG_ATTR_BUS_POWERED)
//	(USB_CONFIG_ATTR_REMOTE_WAKEUP|USB_CONFIG_ATTR_SELF_POWERED)
```

I was using these:

```c
        .idVendor = 0x0483,
        .idProduct = 0x5710,
```

Change the to 0x05ac/0x2227, no more unrecognized keyboard dialog! Alas, too early to celebrate. This is with only the keyboard descriptor. Uncomment the mouse descriptor and report ID, making it a composite keyboard + mouse, then try it. And we get.. no dialog! Sweet.

Now to test sending keystrokes from our fresh new keyboard HID device. For sending the reports, we can look at the descriptor above, but Atmel's [udi_hid_kbd.c](https://github.com/hak5darren/USB-Rubber-Ducky/blob/33a834b0e19f9d4f995432eb9dbcccb247c2e4df/Firmware/Source/Ducky_Multi_Payload/src/asf/common/services/usb/class/hid/device/kbd/udi_hid_kbd.c) used in the Rubber Ducky is instructive. 8 bytes: modifiers, keys down, then LEDs (but the LEDs don't seem to be used - presumably since they are more applicable to a real keyboard accepting user input and with physical LEDs). StackOverflow to the rescue, [List of hex keyboard scan codes and USB HID keyboard documentation](https://stackoverflow.com/questions/27075328/list-of-hex-keyboard-scan-codes-and-usb-hid-keyboard-documentation):

> `Byte 0: Keyboard modifier bits (SHIFT, ALT, CTRL etc)`

> `Byte 1: reserved`

> `Byte 2-7: Up to six keyboard usage indexes representing the keys that are `

> `          currently "pressed". `

> `          Order is not important, a key is either pressed (present in the `

> `          buffer) or not pressed.`

> Note that the USB spec doesn't define keyboard layouts. It simply lists the usage codes assigned to particular key functions. The letter "a" is usage code 0x04 for example. If you want an uppercase "A", then you would also need to set the Byte 0 modifier bits to select "Left Shift" (or "Right Shift").

but it'll be 9 bytes since we changed the format so the report ID is first, and for a keyboard, the report ID will be 1. So I made a simple change to test spamming the keyboard when enabled over serial:

```c
 static int dir = 1;
 static bool jiggler = true;
+static bool spam_keyboard = false;
 void sys_tick_handler(void)
 {
        static int x = 0;
@@ -403,7 +404,16 @@ void sys_tick_handler(void)
                        dir = -dir;
        }
 
-       usbd_ep_write_packet(usbd_dev, 0x81, buf, 4);
+       usbd_ep_write_packet(usbd_dev, 0x81, buf, 5);
+
+       if (spam_keyboard) {
+               uint8_t report[9] = {0};
+               report[0] = 1; // keyboard
+               report[1] = 0; // no modifiers down
+               report[2] = 0;
+               if (dir < 0) report[3] = 0x04; // 'A'
+               usbd_ep_write_packet(usbd_dev, 0x81, buf, sizeof(report));
+       }
 }
 
 static void usbuart_usb_out_cb(usbd_device *dev, uint8_t ep)
@@ -431,6 +441,8 @@ static void usbuart_usb_out_cb(usbd_device *dev, uint8_t ep)
                        jiggler = false;
                } else if (buf[i] == 'q') {
                        jiggler = !jiggler;
+               } else if (buf[i] == 'k') {
+                       spam_keyboard = !spam_keyboard;
                }
        }
 }
```

compile, flash, plug in... but then I couldn't connect to the virtual USB serial device. The device node kept disappearing and reappearing with different numbers: /dev/tty.usbmodem326, then /dev/tty.usbmodem327 a few seconds later. I tried to debug, but subsequently the Black Magic Probe which also connects over a USB serial modem disconnected:

```
(gdb) monitor swdp_scan
Target voltage: unknown
Available Targets:
No. Att Driver
 1      STM32F1 medium density
(gdb) att 1
Attaching to Remote target
warning: No executable has been specified and target does not support
determining executable automatically.  Try using the "file" command.

Thread 1 stopped.
0x08001528 in ?? ()
(gdb) c
Continuing.
Remote communication error.  Target disconnected.: Device not configured.
(gdb) 
```

How did I break it? Think I attached to the debugger twice in two different windows, and also I had a stale `/dev/tty.usbmodemDEM2` lying around. Reboot and reattach, our serial comsole appears. Still disappears and reattaches. This happens if I change the usbd_ep_write_packet() call for the mouse to write 5 bytes (sizeof(buf)) instead of 4. But without this change, the last byte of the packet was read out-of-bounds, since I had to add the 1-byte report ID at the front. Consequently, the mouse wheel was constantly held down, manifesting itself as zooming in whenever I pressed control on the keyboard. Ah, must be the wMaxPacketSize field in the descriptors:

```c
const struct usb_endpoint_descriptor hid_endpoint = {
        .bLength = USB_DT_ENDPOINT_SIZE,
        .bDescriptorType = USB_DT_ENDPOINT,
        .bEndpointAddress = 0x81,
        .bmAttributes = USB_ENDPOINT_ATTR_INTERRUPT,
        .wMaxPacketSize = 4,
        .bInterval = 0x20,
};
```

This should be 9, to accomodate both the mouse and keyboard. Fix it and all the glitchy behavior disappears.

But then another bizarre problem. I try to press the "A" key by setting the 4th byte of the packet:

```c
                uint8_t report[9] = {0};
                report[0] = 1; // keyboard
                report[1] = 0; // no modifiers down
                report[2] = 0;
                report[3] = 0x04; // 'A'
                usbd_ep_write_packet(usbd_dev, 0x81, report, sizeof(report));
```

but when I step through in the debugger, it is zero:

```
(gdb) b main.c:414
Breakpoint 1 at 0x8000358: file main.c, line 414.
(gdb) c
Continuing.
Note: automatically using hardware breakpoints for read-only addresses.

Breakpoint 1, sys_tick_handler () at main.c:414
414			usbd_ep_write_packet(usbd_dev, 0x81, report, sizeof(report));
(gdb) p report
$1 = "\001\000\000\000\000\000\000\000"
```

How can this be? Heisenbug ish behavior, more mysteriously if I continue in the debugger (having loaded symbols from the .elf), then it corrects itself and begins typing a's:

```
(gdb) c
Continuing.
a
Breakpoint 1, sys_tick_handler () at main.c:414
414			usbd_ep_write_packet(usbd_dev, 0x81, report, sizeof(report));
(gdb) aaaaaakaaaaaaaayaaaaaaaa^CQuitaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
(gdb) aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

Some weird problem during programming? Memory issue? This is from an interrupt handler (sys_tick_handler), maybe memory-constrained? But I moved the buffer declaration outside of the function, same behavior. Crazy, must be missing something fundamental.

Testing further, if I disable the mouse jiggler, which previously calls usbd_ep_write_packet() in the same function, then the keyboard spammer works as expected. Perhaps we should RTFM? The [usbd.h header comment](https://github.com/libopencm3/libopencm3/blob/599dd43190efd48d5a4086de46d96ec4652a5fbf/include/libopencm3/usb/usbd.h#L157) says only this:

```c
/** Write a packet
 * @param addr EP address (direction is ignored)
 * @param len # of bytes
 * @return 0 if failed, len if successful
 */
extern uint16_t usbd_ep_write_packet(usbd_device *usbd_dev, uint8_t addr,
				const void *buf, uint16_t len);
```

[usbd.c](https://github.com/libopencm3/libopencm3/blob/599dd43190efd48d5a4086de46d96ec4652a5fbf/lib/usb/usb.c#L129) only calls the `ep_write_packet` function pointer, defined by the driver. The available drivers are:

* [st_usbfs_v1.c](https://github.com/libopencm3/libopencm3/blob/599dd43190efd48d5a4086de46d96ec4652a5fbf/lib/stm32/st_usbfs_v1.c)
* [st_usbfs_v2.c](https://github.com/libopencm3/libopencm3/blob/8c74128ff6aaa2f785d8e994e82746a6e937e46c/lib/stm32/st_usbfs_v2.c)
* [usb_f207.c](https://github.com/libopencm3/libopencm3/blob/825b51a6a8e7cf6ebef9477879a43ca7d89a313b/lib/usb/usb_f207.c)
* [usb_efm32lg.c](https://github.com/libopencm3/libopencm3/blob/599dd43190efd48d5a4086de46d96ec4652a5fbf/lib/usb/usb_efm32lg.c)
* [usb_lm4f.c](https://github.com/libopencm3/libopencm3/blob/599dd43190efd48d5a4086de46d96ec4652a5fbf/lib/usb/usb_lm4f.c)

The latter drivers are for different microcontrollers. This leaves st_usbfs_v1 and st_usbfs_v2. "fs" here refers to USB Full Speed (12 Mbit/s). The libopencm3 hid_usb example I started with uses v1, as it calls `usbd_init()` with `&st_usbfs_v1_usb_driver`. [None of the examples use st_usbfs_v2_usb_driver](https://github.com/libopencm3/libopencm3-examples/search?utf8=✓&q=st_usbfs_v2_usb_driver&type=). What's the point of v2? `diff -ur st_usbfs_v1.c st_usbfs_v2.c` shows v1 was written by Gareth McMullin in 2010, and v2 by Kuldeep Singh Dhaka in v2, it has a couple additions: v2 calls `SET_REG(USB_BCDR_REG, USB_BCDR_DPPU)` when initializing, uses a bytewise copy for Cortex-M0 in st_usbfs_copy_to_pm, and also has a different copy in st_usbfs_copy_from_pm:

```c
 void st_usbfs_copy_to_pm(volatile void *vPM, const void *buf, uint16_t len)
 {
-	const uint16_t *lbuf = buf;
-	volatile uint32_t *PM = vPM;
-	for (len = (len + 1) >> 1; len; len--) {
-		*PM++ = *lbuf++;
+	/*
+	 * This is a bytewise copy, so it always works, even on CM0(+)
+	 * that don't support unaligned accesses.
+	 */
+	const uint8_t *lbuf = buf;
+	volatile uint16_t *PM = vPM;
+	uint32_t i;
+	for (i = 0; i < len; i += 2) {
+		*PM++ = (uint16_t)lbuf[i+1] << 8 | lbuf[i];
 	}
 }
```

The commit history shows v2 was added here: [usb: Add st_usbfs_v2 for f0/l0 devices](https://github.com/libopencm3/libopencm3/commit/f5eb96caf310b7e288aba5678d2d77f13a5a2ed3) with this commit message: "Based on previous work, add a new driver for the v2 usb peripheral found on
stm32f0 and l0 devices." F0 and L0 are other lower-cost STM32 processors, but the blue pill is F1 (STM32F103), so the v2 driver isn't needed, v1 should be fine. There is more explanation in [usb: extract ST USB FS peripheral core. BREAKING CHANGE](https://github.com/libopencm3/libopencm3/commit/e121243ce20b688f0cd11e5280ca968656e3d414)

> * instead of the confusingly specific "f103" name for the driver, use "st_usbfs_v1"  [BREAKING_CHANGE]

>  Instead of:

> `   usbd_init(&stm32f103_usb_driver, .....) ==>`

> `    usbd_init(&st_usbfs_v1_usb_driver, .....) ==>`

> The purpose of these changes is to reduce some confusion around naming, but primarily to prepare for the "v2" peripheral  available on stm32f0/l0 and some f3 devices.

So that's that. We use the v1 driver.

Looking into common/st_usbfs_core.c, `st_usbfs_ep_write_packet()` copies the buffer to "pm" using `st_usbfs_copy_to_pm()`. That is, to `USB_GET_EP_TX_BUFF(addr)`, defined in [st_usbfs_v1.h](https://github.com/libopencm3/libopencm3/blob/599dd43190efd48d5a4086de46d96ec4652a5fbf/include/libopencm3/stm32/common/st_usbfs_v1.h#L55) as:

```c
/* --- USB BTABLE manipulators --------------------------------------------- */

#define USB_GET_EP_TX_BUFF(EP) \
	(USB_PMA_BASE + (uint8_t *)(USB_GET_EP_TX_ADDR(EP) * 2))
```	

`USB_PMA_BASE` is defined in [memorymap.h] as `(PERIPH_BASE_APB1 + 0x6000)`, and PERIPH_BASE_APB1 as (PERIPH_BASE + 0x00000), finally PERIPH_BASE as (0x40000000U). From the [reference manual](http://www.st.com/content/ccc/resource/technical/document/reference_manual/59/b9/ba/7f/11/af/43/d5/CD00171190.pdf/files/CD00171190.pdf/jcr:content/translations/en.CD00171190.pdf), PMA = packet memory area. My suspicion is that the previous packet is being sent asynchronously simultaneously when I'm trying to send another packet. [USB in a Nutshell](http://www.beyondlogic.org/usbnutshell/usb3.shtml) explains:

> As the data is flowing out from the host, it will end up in the EP1 OUT buffer. Your firmware will then at its leisure read this data. If it wants to return data, the function cannot simply write to the bus as the bus is controlled by the host. 

Lots of open libopencm3 issues/pulls, could this be [#626 STM32 USB PMA memory overlap](https://github.com/libopencm3/libopencm3/issues/626) or something similar? Or how about this: [#791 Strange behavior of usbd_ep_write_packet() when len equals to EP max packet size](https://github.com/libopencm3/libopencm3/issues/791), where devanlai answers:

> You may need to send a zero-length packet after sending a packet of the maximum packet size before the host USB stack will consider the USB transfer complete and pass the data onto the application:
[http://stackoverflow.com/questions/41855995/when-should-a-usb-device-send-a-zlp-on-a-bulk-pipe](http://stackoverflow.com/questions/41855995/when-should-a-usb-device-send-a-zlp-on-a-bulk-pipe
)

To test these theories, first tried adding an empty packet after the second send, no difference. Then I added a delay with a loop of `asm("nop")`, and the keyboard packets were sent and processed correctly. But the mouse jiggler broke. Added a delay loop before both sends, now both keyboard and mouse work as expected. Hrmph.

Delays seem to be a horrible hack, so instead, I set a toggle a variable and alternate sending packets for the mouse and keyboard, this works:

```c
void sys_tick_handler(void)
{
        static bool toggle = false;

        if (jiggler && toggle) {
                for (int i = 0; i < 1000000; ++i) asm("nop");

                static int x = 0;
                uint8_t buf[5] = {0, 0, 0, 0, 0};

                buf[0] = 2; // mouse
                buf[2] = dir;

                x += dir;
                if (x > 30)
                        dir = -dir;
                if (x < -30)
                        dir = -dir;
                usbd_ep_write_packet(usbd_dev, 0x81, buf, sizeof(buf));
        }

        if (spam_keyboard && !toggle) {
                report[0] = 1; // keyboard
                report[1] = 0; // no modifiers down
                report[2] = 0;
                report[3] = 0x06; // 'c'
                usbd_ep_write_packet(usbd_dev, 0x81, report, sizeof(report));
        }

        toggle = !toggle;
}
```

### Replaying keystrokes

Spamming the keyboard isn't too useful, of course, we would rather like to playback arbitrary keystrokes, configured ahead-of-time. My idea for implementing this is to maintain a buffer of USB HID report packets, sending one on each system tick. To start off, we could define this structure, with a union for overlapping the mouse and keyboard structures:

```c
// Structure of HID report packets, must match hid_report_descriptor
struct composite_report {
        uint8_t report_id;
        union { 
                struct {
                        uint8_t buttons;
                        uint8_t x;
                        uint8_t y;
                        uint8_t wheel;
                } __attribute__((packed)) mouse;
                
                struct {
                        uint8_t modifiers;
                        uint8_t reserved;
                        uint8_t keys_down[6];
                        uint8_t leds;
                } __attribute__((packed)) keyboard;
        };
} __attribute__((packed));
```

then replay the packets from an array of this structure. I also overloaded the meaning of `report_id` here, using 0 for no-operation and any other value (besides 1 for keyboard, 2 for mouse) for returning to the beginning:


```c
static struct composite_report packets[1024] = {0};
static int report_index = 0;

void sys_tick_handler(void)
{
        struct composite_report report = packets[report_index];
        uint16_t len = 0;
        uint8_t id = report.report_id;

        if (id == REPORT_ID_NOP) {
                return;
        } else if (id == REPORT_ID_KEYBOARD) {
                len = 9;
        } else if (id == REPORT_ID_MOUSE) {
                len = 5;
        } else {
                report_index = 0;
                return;
        }

        usbd_ep_write_packet(usbd_dev, 0x81, &report, len);

        report_index += 1;
}
```

The mouse jiggler and keyboard spammer can be easily reimplemented using these data structures:

```c
void add_mouse_jiggler(int width)
{       
        int j = report_index;
        for (int i = 0; i < width; ++i) {
                packets[j].report_id = REPORT_ID_MOUSE;
                packets[j].mouse.buttons = 0;
                packets[j].mouse.x = 1;
                packets[j].mouse.y = 0;
                packets[j].mouse.wheel = 0;
                ++j;
        }
        
        for (int i = 0; i < width; ++i) {
                packets[j].report_id = REPORT_ID_MOUSE;
                packets[j].mouse.buttons = 0;
                packets[j].mouse.x = -1;
                packets[j].mouse.y = 0;
                packets[j].mouse.wheel = 0;
                ++j;
        }
        
        packets[j].report_id = REPORT_ID_END;
        
        report_index = j;
}

void add_keyboard_spammer(int scancode)
{       
        int j = report_index;
        
        packets[j].report_id = REPORT_ID_KEYBOARD;
        packets[j].keyboard.modifiers = 0;
        packets[j].keyboard.reserved = 0;
        packets[j].keyboard.keys_down[0] = scancode;
        packets[j].keyboard.keys_down[1] = 0;
        packets[j].keyboard.keys_down[2] = 0;
        packets[j].keyboard.keys_down[3] = 0;
        packets[j].keyboard.keys_down[4] = 0;
        packets[j].keyboard.keys_down[5] = 0;
        packets[j].keyboard.leds = 0;
        ++j;

        packets[j].report_id = REPORT_ID_END;

        report_index = j;
}
```

but the real power is in programmability, so we'll need to find a way to allow the user to change these payload packets at runtime. Fortunately, there is precedent in the USB Rubber Ducky: Ducky Script

## Ducky Script
[Ducky Script](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Duckyscript) is a simple language for controlling the Rubber Ducky. It is compiled to binary using [duckencoder](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Downloads), to an `inject.bin` file which is read by the microcontroller and executed. An example hello.duckyscript:

```
REM see https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Duckyscript
DELAY 1000
STRING hello world
DELAY 500
ENTER
```

Compile and dump:

```sh
USB-Rubber-Ducky $ java -jar duckencoder.jar -i hello.duckyscript 
Hak5 Duck Encoder 2.6.3

Loading File .....		[ OK ]
Loading Keyboard File .....	[ OK ]
Loading Language File .....	[ OK ]
Loading DuckyScript .....	[ OK ]
DuckyScript Complete.....	[ OK ]

USB-Rubber-Ducky $ hexdump inject.bin 
0000000 00 ff 00 ff 00 ff 00 eb 0b 00 08 00 0f 00 0f 00
0000010 12 00 2c 00 1a 00 12 00 15 00 0f 00 07 00 00 ff
0000020 00 f5 28 00                                    
0000024
USB-Rubber-Ducky $ 
```

The binary format can be found by examining the firmware source, after [case state_INJECTING](https://github.com/hak5darren/USB-Rubber-Ducky/blob/33a834b0e19f9d4f995432eb9dbcccb247c2e4df/Firmware/Source/Ducky_HID/src/main.c#L143):

```c
		case state_INJECTING:				
			
			if( file_eof() ) {
				file_close();	
				state = state_IDLE;
				break;
			}
			
			injectToken = ( file_getc() | ( file_getc() << 8 ) );			
						
			if( ( injectToken&0xff ) == 0x00 ) {				
				wait = injectToken>>8;
				state = state_WAIT;
			}
			else if( ( injectToken>>8 ) == 0x00 ) {
				state = state_KEY_DOWN;
			}				
			else {
				state = state_MOD_DOWN;					
			}					
			break;
```

16-bit words. First byte zero indicates a delay specified by the second byte. Second byte zero indicates a keydown of the first byte, other indicate a modifier down. This is a state machine, so after entering the key down state, the next state is key up, and so on. It is instructive to look at the [duckencoder source](https://github.com/hak5darren/USB-Rubber-Ducky/blob/master/Encoder/src/Encoder.java#L239):

```java
                                	} else if (instruction[0].equals("DELAY")) {
                                        int delay = Integer.parseInt(instruction[1].trim());
                                        while (delay > 0) {
                                                file.add((byte) 0x00);
                                                if (delay > 255) {
                                                        file.add((byte) 0xFF);
                                                        delay = delay - 255;
                                                } else {
                                                        file.add((byte) delay);
                                                        delay = 0;
                                                }
                                        }
                                        delayOverride = true;
                                	} else if (instruction[0].equals("STRING")) {
                                        for (int j = 0; j < instruction[1].length(); j++) {
                                                char c = instruction[1].charAt(j);
                                                addBytes(file,charToBytes(c));
                                        }
```

To support delays more than 255, the encoder splits up the larger delay into multiple smaller delays. To encode typing a string, addBytes() pads to an even length, charToBytes() calls codeToBytes(charToCode(c)):

```java
        private static byte[] charToBytes (char c){
                return codeToBytes(charToCode(c));
        }
        private static String charToCode (char c){
                String code;
                if(c<128){
                code = "ASCII_"+Integer.toHexString(c).toUpperCase();
            }else if (c<256){
                code = "ISO_8859_1_"+Integer.toHexString(c).toUpperCase();
            }else{
                code = "UNICODE_"+Integer.toHexString(c).toUpperCase();
            }
                return code;
        }       
```

codeToBytes() reads the keyboard layout and properties, from these resources: [keyboard.properties](https://github.com/hak5darren/USB-Rubber-Ducky/blob/master/Encoder/resources/keyboard.properties) and [us.properties](https://github.com/hak5darren/USB-Rubber-Ducky/blob/master/Encoder/resources/us.properties) (or another language). As an example, "// 99 c" followed by "ASCII_63 = KEY_C" in us.properties, maps to keyboard.properties's "KEY_C = 6". [ducky-decode.pl](https://github.com/hak5darren/USB-Rubber-Ducky/blob/master/Decode/ducky-decode.pl), for some reason written in Perl. The script is curiously a big list of regexes replacing hex, but it sheds some more light on the compiled Ducky Script binary structure of keystroke scancodes followed by modifiers:

```perl
	$hex=~ s/^6320/\</g;
	$hex=~ s/^7320/\>/g;
        $hex=~ s/^7300/\./g;
        $hex=~ s/^8300/\//g;
        $hex=~ s/^4000/a/g;
        $hex=~ s/^4020/A/g;
        $hex=~ s/^5000/b/g;
        $hex=~ s/^5020/B/g;
        $hex=~ s/^6000/c/g;
	$hex=~ s/^6020/C/g;
```

The scancodes are...backwards? KEY_C is 0x06, the first byte for 'c' here is 0x60. The modifiers 0x20 are for shift, to capitalize the C. And check this out, the decodings are hardcoded but nibble-reversed:

```perl
	$hex=~s/^0082/\nDELAY 40\n/g;
	$hex=~s/^0023/\nDELAY 50\n/g;
        $hex=~s/^00b4/\nDELAY 75\n/g;
	$hex=~s/^0069/\nDELAY 150\n/g;
```	

0x82 reversed is 0x28, which is precisely 40. 0x23 reversed to 0x32 is 50, 0x4b is 75, 0x96 is 150. Weird they did it this way instead of simply swapping the nibbles and taking the decimal value. The swapped nibbles is an artifact of Perl `unpack("h*")`:

```sh
  DB<5> ~ $ perl -de0

Loading DB routines from perl5db.pl version 1.39_10
Editor support available.

Enter h or 'h h' for help, or 'man perldebug' for more help.

main::(-e:1):	0
  DB<1> x unpack("h*", "\x06")
0  60
  DB<2> 
```

if they simply used `H*` instead of `h*`, then the nibbles would be correct. Fortunately, this is not a bug in the `inject.bin` file format itself, it has the expected nibble order so we don't need to swap. Testing encoding a trivial script "STRING cccc" we see the 0x06 codes repeated as expected:

```sh
USB-Rubber-Ducky $ cat c.duckyscript 
STRING cccc
USB-Rubber-Ducky $ hexdump c.bin 
0000000 06 00 06 00 06 00 06 00                        
0000008
```

There's a great tutorial by Hartley Brody on August 29, 2017 including more about using these tools: [USB Rubber Ducky Tutorial: The Missing Quickstart Guide to Running Your First Keystroke Payload Hack](https://blog.hartleybrody.com/rubber-ducky-guide/).

As for this project, we can process the compiled Ducky Script binary and generate USB HID report frames. The Rubber Ducky seems to send more USB packets than necessary, a separate packet for pressing and releasing the modifier, independent from the normal keys. But can't one packet be used for pressing both a key and modifier key? [Microchip forums USB Keyboard modifier keys?](http://www.microchip.com/forums/m133681.aspx) says there are keycodes also defined for the modifiers but they seem to be ignored:

> Also, p. 56, pp. 59-60 are helpful in understanding the contents of the keyboard report. The state of the modifier keys are encoded as a bit field in the first byte (i.e., offset 0) of the keyboard input report. There is a table on p. 56 that tells which bit corresponds to which modifier (bit 0 is L CTRL, bit 1 is L SHIFT, bit 2 is L ALT, bit 3 is L GUI, bit 4 is R CTRL, bit 5 is R SHIFT, bit 6 is R ALT, and bit 7 is R GUI). There are keycodes listed for each of these modifier keys as well; I don't quite get why these keycodes have been defined, as they seem to be ignored by the operating system.

Page 56 of [HID1_11.pdf](http://www.usb.org/developers/hidpage/HID1_11.pdf):

![Keyboard bitmap modifiers byte bits](https://user-images.githubusercontent.com/26856618/34366415-010f2a74-ea50-11e7-9ee6-1b0b9e9d82e6.png)

The above example shows the packet sequence that would occur if the user actually pressed and released Ctrl+Alt+Delete in sequence, but in this Sparkfun tutorial [Turn your ProMicro into a USB Keyboard/Mouse](https://www.sparkfun.com/tutorials/337), they show sending only two packets, for pressing Ctrl+Alt+Delete all simultaneously, then releasing it:

```c
    sendKey(KEY_DELETE, KEY_MODIFIER_LEFT_CTRL | KEY_MODIFIER_LEFT_ALT);  // send a CTRL+ALT+DEL to the computer via Keyboard HID

...

void sendKey(byte key, byte modifiers)
{
  KeyReport report = {0};  // Create an empty KeyReport
  
  /* First send a report with the keys and modifiers pressed */
  report.keys[0] = key;  // set the KeyReport to key
  report.modifiers = modifiers;  // set the KeyReport's modifiers
  report.reserved = 1;
  Keyboard.sendReport(&report);  // send the KeyReport
  
  /* Now we've got to send a report with nothing pressed */
  for (int i=0; i<6; i++)
    report.keys[i] = 0;  // clear out the keys
  report.modifiers = 0x00;  // clear out the modifires
  report.reserved = 0;
  Keyboard.sendReport(&report);  // send the empty key report
}
```

Curious how they set reserved to 1 in the first packet then 0 in the second, an error? This [mbed documentation](https://docs.mbed.com/docs/ble-hid/en/latest/api/md_doc_HID.html) confirms reserved is "Input: Static Value (0x1). This byte is reserved (constant).", so I set it to always 1. Write a simple converter from Ducky Script compiled binary to USB HID report packets, try sending "\x07\x00\x07\x00\x07\x00\x07\x00" and it works expected, sending several 'd's. But when I add a left-shift modifier, "\x07\x02\x07\x00\x07\x00\x07\x00", attempting to capitalize only the first character, the results are inconsistent:

> DdDddDDDDDDDDDdDdDddDDDDDDDdDdDddDDDDDDDDDdDdDDdDdDddDDD

It seems to be missing a lot of packets. How frequently is sys_tick_handler called? [Basic systick configuration on the STM32](http://www.micromouseonline.com/2016/02/02/systick-configuration-made-easy-on-the-stm32/) explains how to use `SysTicks_Config` from CMSIS but I'm using libopencm3. See the [systick defines](http://libopencm3.org/docs/latest/stm32f1/html/group__CM3__systick__defines.html); I had hid.c from the usb_hid example hid_set_config() set as follows:

```c
        systick_set_clocksource(STK_CSR_CLKSOURCE_AHB_DIV8);
        /* SysTick interrupt every N clock pulses: set reload to N-1 */
        systick_set_reload(99999);
        systick_interrupt_enable();
        systick_counter_enable();
```

STK_CSR_CLKSOURCE_AHB_DIV8 makes an appearance on the stm32dunio forums, post [http://www.stm32duino.com/viewtopic.php?t=1076](http://www.stm32duino.com/viewtopic.php?t=1076):

```c
    systick_set_clocksource(STK_CSR_CLKSOURCE_AHB_DIV8);    /// SysTick at 72Mhz/8
    systick_set_reload(8999);                               /// SysTick Reload for 1ms tick ( 72M/8/1000 =9000 )
    systick_interrupt_enable();
    systick_counter_enable();
```    

100000 / (72 mhz / 8 ) is 11.1' ms. For testing and investigating timing issues, we can increase the period, I picked 100 ms. 900000 / (72 mhz / 8 ) is 100 ms so set systick reload to 899999. Test with compiled Ducky Script binary "\x07\x02\x07\x00\x07\x00\x08\x00", and it works as expected, nothing missed:

> DddeDddeDddeDddeDddeDddeDddeDddeDddeDddeDddeDddeDddeDddeDddeDddeDddeDddeDddeDddeDddeDdde

For now this is sufficient, but it would be preferrable to send packets as fast as possible and fix the synchronization issue, possibly by using USB stalls to wait for the host to process the packet before immediately sending another. Real keyboards obviously can type faster than 100 ms. Here's the initial Ducky Script binary execution routine:

```c
void add_ducky_binary(uint8_t *buf, int len)
{       
        int j = report_index;
        
        // 16-bit words, must be even
        if ((len % 2) != 0) len -= 1;
        
        for (int i = 0; i < len; i += 2) {
                uint16_t word = buf[i] | (buf[i + 1] << 8);
                
                if ((word & 0xff) == 0) {
                        // TODO: wait
                }
                
                // Press key and modifier
                packets[j].report_id = REPORT_ID_KEYBOARD;
                packets[j].keyboard.modifiers = word >> 8;
                packets[j].keyboard.reserved = 1;
                packets[j].keyboard.keys_down[0] = word & 0xff;
                packets[j].keyboard.keys_down[1] = 0;
                packets[j].keyboard.keys_down[2] = 0;
                packets[j].keyboard.keys_down[3] = 0;
                packets[j].keyboard.keys_down[4] = 0;
                packets[j].keyboard.keys_down[5] = 0;
                packets[j].keyboard.leds = 0;
                ++j;
                
                // Release key
                packets[j].report_id = REPORT_ID_KEYBOARD;
                packets[j].keyboard.modifiers = 0;
                packets[j].keyboard.reserved = 1;
                packets[j].keyboard.keys_down[0] = 0;
                packets[j].keyboard.keys_down[1] = 0;
                packets[j].keyboard.keys_down[2] = 0;
                packets[j].keyboard.keys_down[3] = 0;
                packets[j].keyboard.keys_down[4] = 0;
                packets[j].keyboard.keys_down[5] = 0;
                packets[j].keyboard.leds = 0;
                ++j;
        }
        
        packets[j].report_id = REPORT_ID_END;
        
        report_index = j;
}
```

Let's try the hello world Ducky Script we saw earlier (except I added capitalization and punctuation, for a better test):

```
REM see https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Duckyscript
DELAY 1000
STRING Hello, world!
DELAY 500
ENTER
```

Compile with duckencoder.jar, then try it in our device firmware:

```c
        add_ducky_binary((uint8_t *)
                "\x00\xff\x00\xff\x00\xff\x00\xeb\x0b\x02\x08\x00\x0f\x00\x0f\x00"
                "\x12\x00\x36\x00\x2c\x00\x1a\x00\x12\x00\x15\x00\x0f\x00\x07\x00"
                "\x1e\x02\x00\xff\x00\xf5\x28\x00", 36);
```

flash and plug in the USB, and it works! Types "Hello, world!" as expect. No delays yet, but it's a start. As we can see, the advantage of supporting the Ducky Script binary format is we can use existing Ducky Script tools and payloads, there are ton of existing scripts available here: [https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Payloads](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Payloads).

But modifying our firmware source code and recompiling and reflashing isn't too practical, some way to let the user program the payload without recompiling and reflashing is needed. What can we do?

## Storage of payload on flash

The USB Rubber Ducky uses a [Secure Digital](https://en.wikipedia.org/wiki/Secure_Digital) card, here's a picture of it [from Hartley Brody](https://blog.hartleybrody.com/rubber-ducky-guide/), including the microSD card and microSD-to-USB adapter:

![Official USB Rubber Ducky with microSD card and adapter](https://user-images.githubusercontent.com/26856618/34366928-a4a96e4a-ea57-11e7-9da2-bb5f094130fc.jpg)

The [firmware](https://github.com/hak5darren/USB-Rubber-Ducky/blob/33a834b0e19f9d4f995432eb9dbcccb247c2e4df/Firmware/Source/Ducky_HID/src/main.c#L62) has code to initalize the SD/MMC interface. From the [hardware](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Hardware) page on their wiki, these are all the components:

* Atmel 32bit AVR Microcontroller AT32UC3B1256
* MicroSD card reader
* Micro push-button
* Multi-color LED indicator
* JTAG Interface (can be used for I/O)
* Standard “Type A” USB connector

The microSD card is extra hardware, equating to extra cost. But the STM32F103 blue pill has built-in flash! There is [some discussion](https://github.com/blacksphere/blackmagic/blob/dd055b675ec4dd3d7c66f066497415ea9f91bde4/src/platforms/stlink/Flashsize_F103) about whether the blue pill has 64 KB or 128 KB flash, long story short officially the F103C8 part has 64 KB and F103CB has 128 KB but in practice, most/many blue pills actually have 128 KB. All the boards I ordered I have 128 KB, at least, your milage may vary. In either case, 64 KB ought to be enough for anybody.

A portion of the flash will be used for the program code. How much? Currently up to this point, the binary comes to 6.4 KB, only 6580 bytes. Plenty of space for a moderately large payload.

libopencm3 has tons of functions for manipulating [flash](http://libopencm3.org/docs/latest/stm32f1/html/group__flash__defines.html). The libopencm3-example [stm32/f1/other/usb_dfu](https://github.com/libopencm3/libopencm3-examples/blob/master/examples/stm32/f1/other/usb_dfu/usbdfu.c#L140) demonstrates writing to flash by calling flash_unlock(), flash_erase_page(), flash_program_half_word(), then flash_lock(). Other examples `#include <libopencm3/stm32/flash.h>` but do not use the flash functions. [usbiap](https://github.com/libopencm3/libopencm3-examples/blob/master/examples/stm32/f1/stm32-h103/usb_iap/usbiap.c) implements the same device firmware update (DFU) USB interface, writing to flash. It is intended for a more powerful micro than we have, the '107 series (versus '103), but [stm32/f1/stm32-h107/flash_rw_example](https://github.com/libopencm3/libopencm3-examples/blob/master/examples/stm32/f1/stm32-h107/flash_rw_example/flash_rw_example.c) is more instructive, providing a UART interface to interact with an read data to write to flash. This is their flash programming function in its entirety:

```c
#define FLASH_OPERATION_ADDRESS ((uint32_t)0x0800f000)

...

static uint32_t flash_program_data(uint32_t start_address, uint8_t *input_data, uint16_t num_elements)
{
	uint16_t iter;
	uint32_t current_address = start_address;
	uint32_t page_address = start_address;
	uint32_t flash_status = 0;

	/*check if start_address is in proper range*/
	if((start_address - FLASH_BASE) >= (FLASH_PAGE_SIZE * (FLASH_PAGE_NUM_MAX+1)))
		return 1;

	/*calculate current page address*/
	if(start_address % FLASH_PAGE_SIZE)
		page_address -= (start_address % FLASH_PAGE_SIZE);

	flash_unlock();

	/*Erasing page*/
	flash_erase_page(page_address);
	flash_status = flash_get_status_flags();
	if(flash_status != FLASH_SR_EOP)
		return flash_status;

	/*programming flash memory*/
	for(iter=0; iter<num_elements; iter += 4)
	{
		/*programming word data*/
		flash_program_word(current_address+iter, *((uint32_t*)(input_data + iter)));
		flash_status = flash_get_status_flags();
		if(flash_status != FLASH_SR_EOP)
			return flash_status;

		/*verify if correct data is programmed*/
		if(*((uint32_t*)(current_address+iter)) != *((uint32_t*)(input_data + iter)))
			return FLASH_WRONG_DATA_WRITTEN;
	}

	return 0;
}
```

Same pattern: unlock, erase the page, then program each word. How did they know the starting address of 0x0800f000? Found some insight on StackOverflow at [Allocating memory in Flash for user data (STM32F4 HAL)](https://stackoverflow.com/questions/28503808/allocating-memory-in-flash-for-user-data-stm32f4-hal#28505272), editing the `.ld` linker file to add a new section for user data. Here is what we had before:

```
/* Define memory regions. */
MEMORY
{
    rom (rx) : ORIGIN = 0x08000000, LENGTH =  128K
    ram (rwx) : ORIGIN = 0x20000000, LENGTH = 20K
}

/* Include the common ld script from libopenstm32. */
INCLUDE libopencm3_stm32f1.ld
```

128KB is 0x20000, since the program is only currently 0x19b4 bytes I'll allocate most to user data, but allow some room for future expansion of the program. It can always be shifted later if necessary, however. For now, going with 8 KB (0x2000) program:

```
MEMORY
{
    rom (rx)   : ORIGIN = 0x08000000, LENGTH =  8K
    data (rwx) : ORIGIN = 0x08002000, LENGTH =  128K-8K

    ram (rwx) : ORIGIN = 0x20000000, LENGTH = 20K
}

SECTIONS
{
    .user_data :
    {
        . = ALIGN(4);
            *(.user_data)
        . = ALIGN(4);
    } > data
}
```

then I added a 56 KB variable in this new user_data section, in C:

```c
__attribute__((__section__(".user_data"))) const uint8_t user_data[(128 - 8) * 1024];
```

but when I write code accessing it, the firmware binary balloons from 6K to 128 KB! This user had a similar problem: [Huge Binary size while ld Linking](https://stackoverflow.com/questions/40502095/huge-binary-size-while-ld-linking): "When i built, the binary size is just 6 KB. But i can not add any initialized variable. When i add an initialized variable, the binary size jumps to ~246 MB. ". We can see the sections with `readelf`:

```sh
src $ arm-none-eabi-readelf --sections pill_duck.elf 
There are 22 section headers, starting at offset 0x58274:

Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .user_data        PROGBITS        08002000 022000 002800 00  WA  0   0  1
  [ 2] .text             PROGBITS        08000000 010000 00198c 00  AX  0   0  4
  [ 3] .preinit_array    PREINIT_ARRAY   0800198c 024800 000000 04  WA  0   0  1
  [ 4] .init_array       INIT_ARRAY      0800198c 024800 000000 04  WA  0   0  1
  [ 5] .fini_array       FINI_ARRAY      0800198c 024800 000000 04  WA  0   0  1
  [ 6] .data             PROGBITS        20000000 020000 000034 00  WA  0   0  4
  [ 7] .bss              NOBITS          20000034 020034 00298c 00  WA  0   0  4
  [ 8] .debug_info       PROGBITS        00000000 024800 00bb62 00      0   0  1
  [ 9] .debug_abbrev     PROGBITS        00000000 030362 002102 00      0   0  1
  [10] .debug_loc        PROGBITS        00000000 032464 00356d 00      0   0  1
  [11] .debug_aranges    PROGBITS        00000000 0359d1 0005f8 00      0   0  1
  [12] .debug_ranges     PROGBITS        00000000 035fc9 000760 00      0   0  1
  [13] .debug_macro      PROGBITS        00000000 036729 005d17 00      0   0  1
  [14] .debug_line       PROGBITS        00000000 03c440 00498b 00      0   0  1
  [15] .debug_str        PROGBITS        00000000 040dcb 013d4e 01  MS  0   0  1
  [16] .comment          PROGBITS        00000000 054b19 00007f 01  MS  0   0  1
  [17] .ARM.attributes   ARM_ATTRIBUTES  00000000 054b98 000031 00      0   0  1
  [18] .debug_frame      PROGBITS        00000000 054bcc 000e94 00      0   0  4
  [19] .symtab           SYMTAB          00000000 055a60 001940 10     20 233  4
  [20] .strtab           STRTAB          00000000 0573a0 000de8 00      0   0  1
  [21] .shstrtab         STRTAB          00000000 058188 0000ea 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  y (purecode), p (processor specific)
src $ 
```

old_timer responded with a very informative answer well worth reading to better understand linkers, but long story short, adding NOLOAD will cause the data to not be part of the binary. Do we want it to be? On one hand, if the user data was part of the binary firmware image, then a premade payload could be stashed there by default. The user could overwrite it, but it would be immediately available after flashing the image from scratch. Nonetheless, this wouldn't be the only way to accomplish having a default payload, the main program could detect no payload is flashed and load its own from its own memory section. This sounds better, and also would let us avoid wiping the user data when doing firmware updates. OK, where do I add the NOLOAD. [Emprog Thunderbench Linker Script Guide](http://www.emprog.com/support/documentation/thunderbench-Linker-Script-guide.pdf) shows the output section types are added in parenthesizes after the sectio name, either: NOLOAD, DSECT, COPY, INFO, or OVERLAY. Here's the new linker script section directive:

```
SECTIONS
{
    .user_data (NOLOAD) :
    {   
        . = ALIGN(4);
            *(.user_data)
        . = ALIGN(4);
    } > data
}
```

With this change the binary shrinks back to its normal size. Fire up the debugger and confirm the user_data array is located and sized as expected (128 KB - 8 KB = 120 KB):

```
(gdb) p/x &user_data
$1 = 0x8002000
(gdb) p sizeof(user_data)
$3 = 122880
(gdb) p sizeof(user_data)/1024
$4 = 120
```

Now to program it. What user interface shall we use? How about making the USB device show up (additionally) as a disk drive, that is, a USB mass storage class (MSC) device? There is an [stm32f4-discovery/usb_msc](https://github.com/libopencm3/libopencm3-examples/tree/9fbac7daaf4ec2326eb80b37dd6039036f11050d/examples/stm32/f4/stm32f4-discovery/usb_msc) example, but for the F4 processors, not F1. libopencm3 provides a [usb_msc_init](https://github.com/libopencm3/libopencm3/blob/599dd43190efd48d5a4086de46d96ec4652a5fbf/lib/usb/usb_msc.c#L786) function, with the caveat "Currently you can only have this profile active". Could be interesting to try this, but setting aside mass storage for now, there is an easier USB device class I already implemented earlier (twice: first in [pill_serial](https://satoshinm.github.io/blog/171223_stm32serial_triple_usb-to-serial_adapter_using_stm32_blue_pill.html), and also in this post). All we need is a simple protocol to allow sending the binary payload over serial.

A binary protocol could be developed, but then a client-side script would likely be needed. For simplicity, ought to at least have a text-based serial protocol, then anyone can run `screen /dev/tty.usbmodem` to access this device without installing any additional software. Start with implementing echo, so the user can type into the serial port like a terminal. But pressing enter only returns the cursor to column zero, without advancing to the next line. [Black Sphere](https://github.com/blacksphere/blackmagic/blob/dd055b675ec4dd3d7c66f066497415ea9f91bde4/src/platforms/stm32/usbuart.c#L192) has a check for '\n' (LF), prepending '\r' (CR). But this isn't the whole story. Pressing return or ^M goes to column zero, and ^J advances to the next line. This means hitting enter, at least on my system, types \r (CR), not \n (LF). Here's the terminal-like serial echo implementation:

```c
static void usbuart_usb_out_cb(usbd_device *dev, uint8_t ep)
{
        (void)ep;

        char buf[CDCACM_PACKET_SIZE];
        char buf2[CDCACM_PACKET_SIZE];
        int len = usbd_ep_read_packet(dev, CDCACM_UART_ENDPOINT,
                                        buf, CDCACM_PACKET_SIZE);


        int j = 0;
        for(int i = 0; i < len; i++) {
                gpio_toggle(GPIOC, GPIO13);
                
                if (buf[i] == '\r') buf2[j++] = '\n';
                buf2[j++] = buf[i];
        }

        usbd_ep_write_packet(dev, CDCACM_UART_ENDPOINT, buf2, j);
}
```

Expanding on this, implement a command processor, and at first a version command:

```c
static char *process_serial_command(char *buf, int len) {
        (void) len;

        if (buf[0] == 'v') return "version " FIRMWARE_VERSION "\r\n";

        return "";
}

static void usbuart_usb_out_cb(usbd_device *dev, uint8_t ep)
{       
        (void)ep;
        
        char buf[CDCACM_PACKET_SIZE];
        char reply_buf[CDCACM_PACKET_SIZE];

        static char typing_buf[256] = {0};
        static int typing_index = 0;
        
        int len = usbd_ep_read_packet(dev, CDCACM_UART_ENDPOINT,
                                        buf, CDCACM_PACKET_SIZE);
        
        
        int j = 0; 
        for(int i = 0; i < len; i++) {
                gpio_toggle(GPIOC, GPIO13);
        
                // Echo back what was typed
                // Enter sends a CR, but an LF is needed to advance to next line
                if (buf[i] == '\r') reply_buf[j++] = '\n';
                reply_buf[j++] = buf[i];

                typing_buf[typing_index++] = buf[i];

                if (buf[i] == '\r' || buf[i] == '\n') {
                        char *response = process_serial_command(typing_buf, typing_index);
                        typing_index = 0;
        
                        for (size_t k = 0; k < strlen(response); ++k) {
                                reply_buf[j++] = response[k];
                        }

                        // prompt
                        reply_buf[j++] = 'd';
                        reply_buf[j++] = 'u';
                        reply_buf[j++] = 'c';
                        reply_buf[j++] = 'k';
                        reply_buf[j++] = '>';
                        reply_buf[j++] = ' ';
                }
        }

        usbd_ep_write_packet(dev, CDCACM_UART_ENDPOINT, reply_buf, j);
}
```

Testing it:

```sh
screen -L /dev/tty.usbmodem*
� � hello
duck> v
version 802ebe3-dirty
duck> 
```

Now we have a foothold to write code to allow uploading payloads. But this serial interface is going to be text-based, yet the payload is binary, so it'll have to be encoded/decoded; opted for hex, co-opted [hex_utils.c](https://github.com/blacksphere/blackmagic/blob/master/src/hex_utils.c). Added commands to write and read flash data, try writing, reboot and verify can still read what we wrote, confirming the flash persistence:

```
duck> w4142434445
wrote flash
duck> r
41424344ffffffffffffffffffffffff
```

(Unwritten or erased flash data is all ff's.)

Read this flash into the payload buffer and process it. Let's try the hello world, added a new command 'd' to translate compiled DuckyScript to HID and write to flash:

```
duck> d00ff00ff00ff00eb0b0208000f000f00120036002c001a00120015000f000700
wrote flash
duck> r
0102010b000000000000000000000000
duck> p
resumed
duck> HellHellHellHellHellHell
invalid command, try ? for help
duck> p
paused
```

it's something. The data is being truncated somewhere along the line, at element 16, presumably in flashing pages. While researching, found this article on [Micromouse USA: How to use FLASH in STM32 to save explored maze instead of using EEPROM](http://micromouseusa.com/?p=606) ([what is](https://news.ycombinator.com/item?id=16003683) [Micromouse](https://en.wikipedia.org/wiki/Micromouse)?):

> The low and medium density version is 1KB per page, and the high density is 2KB per page. every time the you write data to FLSAH, you need to erase everything on the page that you are going to use, even if you only use just one byte on it. But for reading, it acts just as normal as other process in your program. you can only write as 16 bit data type otherwise it will cause error.

but the problem in my case was a more mundane off-by-one and scaling error. Nonetheless, with only 1KB pages, the page boundary is something one may have to worry about for larger payloads, keep it in mind.

Finally, added a check to start executing the payload if there is one:

```c
        if (user_data[0].report_id != REPORT_ID_END) {
                paused = false;
        }
```

Now the device can be plugged in and it automatically executes, typing whatever keystrokes we programmed into it.

At this point, the micro USB port on the blue pill started acting up, giving an intermittent connection, no doubt because I was constantly unplugging and replugging it. Consider hardwiring a switch on the USB line instead of causing undue stress on the plugs :/

Anyways, nonetheless this device is basically functional. There are a lot of small bugs and features to fix or implement next.

### Small fixes and enhancements

#### Reimplementing the mouse jiggler

At the beginning we had a "mouse jiggler", a USB HID device which moves the mouse back and forth along the X axis by 30 units until changing direction and repeating. This was lost in the conversion to read HID packets from flash, so I reimplemented it:

```c
int add_mouse_jiggler(int width)
{
        int j = 0;
        for (int i = 0; i < width; ++i) {
                packet_buffer[j].report_id = REPORT_ID_MOUSE;
                packet_buffer[j].mouse.buttons = 0;
                packet_buffer[j].mouse.x = 1;
                packet_buffer[j].mouse.y = 0;
                packet_buffer[j].mouse.wheel = 0;
                ++j;
        }

        for (int i = 0; i < width; ++i) {
                packet_buffer[j].report_id = REPORT_ID_MOUSE;
                packet_buffer[j].mouse.buttons = 0;
                packet_buffer[j].mouse.x = -1;
                packet_buffer[j].mouse.y = 0;
                packet_buffer[j].mouse.wheel = 0;
                ++j;
        }

        packet_buffer[j].report_id = REPORT_ID_END;
        ++j;

        return j;
}
```

and in the serial command processing, the user can type "j" to install this mouse jiggling demo:

```c
        } else if (buf[0] == 'j') {
                int records = add_mouse_jiggler(30);
                int binary_len = records * sizeof(struct composite_report);

                int result = flash_program_data((uint32_t)&user_data, (uint8_t *)&packet_buffer, binary_len);
                if (result == RESULT_OK) {
                        return "wrote flash";
                } else if (result == FLASH_WRONG_DATA_WRITTEN) {
                        return "wrong data written";
                } else {
                        return "error writing flash";
                }
```

but this is still limited, we would like to let the user arbitrarily move the mouse. Fortunately, they can write arbitrary HID commands, which may target the mouse device instead of the keyboard. DuckyScript doesn't support it, but raw HID packets do.

#### Error writing flash?!

At this point I started receiving flash errors:

```
wrote flash
duck> v
Pill Duck version da646c9-dirty
duck> r
0102010b000000000000000000000000
duck> j
error writing flash
duck> 
invalid command, try ? for help
duck> j
error writing flash
duck> d00ff00ff00ff00eb0b0208000f000f00120036002c001a00120015000f000700
error writing flash
duck> d00ff00ff00ff00eb0b0208000f000f00120036002c001a00120015000f000700
error writing flash
duck> j
error writing flash
duck> j
error writing flash
duck> 
```

Time to take a break...

#### JavaScript HID report decoder

For testing and development, it is useful to have a tool to decode/encode the HID report packets, and also for users to construct payloads. To this end, wrote a simple Node-based module `hid_report_decoder` with a Webpack interface, try it here: [https://satoshinm.github.io/pill_duck/js/web/index.html](https://satoshinm.github.io/pill_duck/js/web/index.html)

Screenshot:

![Decoder screenshot](https://user-images.githubusercontent.com/26856618/34394323-3f685170-eb0d-11e7-8a6f-12789088be54.png)

#### What's left

The text-based serial console mode is useful for testing, but has limitations which make a scripted binary mode more attractive.  Upload sizes are constrained, to support bigger uploads more than a flash page (1 KB) the payloads should be uploaded over serial in a sequence. Needs to be tested with real payloads: [https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Payloads](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Payloads)

The payload loops forever, but it should have an option to halt after executing once, and not loop. Delays in milliseconds should be implemented. Most importantly, fix slow sending (100 ms systick), investigate USB stalls to confirm packet before sending another.

Would be nice to add a replay button (using GPIO), and enhance the web-based JavaScript HID report tool to make it more useful for end users. And it would be kinda cool to have a gamepad/joystick HID as another device, making this the ultimate composite input device, but not essential.

As you can see there is a lot of work remaining to be done. For now, posted what I have so far on [https://github.com/satoshinm/pill_duck](https://github.com/satoshinm/pill_duck). Feedback and pull requests welcome, this is my first "real" blue pill project so I appreciate any suggestions and help.
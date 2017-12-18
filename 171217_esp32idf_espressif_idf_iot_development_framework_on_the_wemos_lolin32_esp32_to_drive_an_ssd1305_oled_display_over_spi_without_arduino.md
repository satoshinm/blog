# Espressif IDF IoT Development Framework on the WEMOS LOLIN32 ESP32 to drive an SSD1305 OLED display over SPI without Arduino

by snm, December 17th, 2017

* auto-gen TOC:
{:toc}

Espressif released the [ESP32](https://en.wikipedia.org/wiki/ESP32) last year, a successor to the immensely popular [ESP8266](https://en.wikipedia.org/wiki/ESP8266), a low-cost microcontroller featuring built-in Wi-Fi and now Bluetooth. To try it out I purchased a WEMOS LOLIN32 ESP32 development board from Aliexpress for $6.71 (price since dropped further, to <$5):

* [WEMOS LOLIN32 Lite V1.0.0 - wifi & bluetooth board based ESP-32 esp32 Rev1 MicroPython 4MB FLASH](https://www.aliexpress.com/item/WEMOS-LOLIN32-Lite-V1-0-0-wifi-bluetooth-board-based-ESP-32-esp32-Rev1-MicroPython-4MB/32831394824.html)

![WEMOS LOLIN32 ESP32](https://user-images.githubusercontent.com/26856618/34087607-f7ffbcbc-e358-11e7-83e2-a9998bd449e4.jpg)

There is an [Arduino-esp32](https://github.com/espressif/arduino-esp32) core for programming the ESP with the [Arduino](https://www.arduino.cc) environment, which is cool and all, but especially after having dived deeper into "bare metal" programming on a different microcontroller with 
*[STM32 Blue Pill ARM development board first look: from Arduino to bare metal programming](https://satoshinm.github.io/blog/171212_stm32_blue_pill_arm_development_board_first_look_bare_metal_programming.html)*, I figured I'd go deeper here as well and get started with the [Espressif IoT Development Framework. Official development framework for ESP32](https://github.com/espressif/esp-idf), a lower-level alternative to Arduino-esp32.


## Espressif ESP IDF setup

Follow their [getting started guide](https://esp-idf.readthedocs.io/en/latest/get-started/index.html). For [macOS setup](https://esp-idf.readthedocs.io/en/latest/get-started/macos-setup.html), download a prebuilt toolchain. The [from scratch](https://esp-idf.readthedocs.io/en/latest/get-started/macos-setup-scratch.html) guide instructs to create a case-sensitive disk image. What if we remove the case-sensitive checks in scripts/crosstool-NG.sh? Begins to build, but didn't stick around to see where it fails. Building from scratch is actually unnecessary, just download the prebuilt xtensa-esp32-elf toolchain then proceed to the [Get ESP-IDF](https://esp-idf.readthedocs.io/en/latest/get-started/index.html#get-started-get-esp-idf) instructions. Copy the examples/get_started/hello_world example.

Add IDF_PATH to ~/.bash_profile, but instead of adding esp32/xtensa-esp32-elf/bin to PATH, I prefer to symlink from /usr/local/bin:

```sh
cd esp32/xtensa-esp32-elf/bin
for x in `ls`; do ln -s `pwd`/$x /usr/local/bin; done
```

Now you can `make` the example to compile, then `make flash` to flash and `make monitor` to watch over the serial port. I like this better than Arduino.app, feels cleaner and more functional (but I am biased with CLI over GUI). Here is the boot message:

```
rst:0xc (SW_CPU_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)
configsip: 0, SPIWP:0xee
clk_drv:0x00,q_drv:0x00,d_drv:0x00,cs0_drv:0x00,hd_drv:0x00,wp_drv:0x00
mode:DIO, clock div:2
load:0x3fff0018,len:4
load:0x3fff001c,len:5568
ho 0 tail 12 room 4
load:0x40078000,len:0
load:0x40078000,len:13712
entry 0x40079020
I (30) boot: ESP-IDF v3.1-dev-69-g8688f0ec 2nd stage bootloader
I (30) boot: compile time 14:06:41
I (39) boot: Enabling RNG early entropy source...
I (39) boot: SPI Speed      : 40MHz
I (41) boot: SPI Mode       : DIO
I (45) boot: SPI Flash Size : 4MB
I (49) boot: Partition Table:
I (52) boot: ## Label            Usage          Type ST Offset   Length
I (60) boot:  0 nvs              WiFi data        01 02 00009000 00006000
I (67) boot:  1 phy_init         RF data          01 01 0000f000 00001000
I (74) boot:  2 factory          factory app      00 00 00010000 00100000
I (82) boot: End of partition table
I (86) esp_image: segment 0: paddr=0x00010020 vaddr=0x3f400020 size=0x04ef0 ( 20208) map
I (102) esp_image: segment 1: paddr=0x00014f18 vaddr=0x3ffb0000 size=0x0214c (  8524) load
I (107) esp_image: segment 2: paddr=0x0001706c vaddr=0x40080000 size=0x00400 (  1024) load
0x40080000: _iram_start at esp-idf/components/freertos/./xtensa_vectors.S:1685

I (113) esp_image: segment 3: paddr=0x00017474 vaddr=0x40080400 size=0x08274 ( 33396) load
I (136) esp_image: segment 4: paddr=0x0001f6f0 vaddr=0x400c0000 size=0x00000 (     0) load
I (136) esp_image: segment 5: paddr=0x0001f6f8 vaddr=0x00000000 size=0x00918 (  2328) 
I (143) esp_image: segment 6: paddr=0x00020018 vaddr=0x400d0018 size=0x1331c ( 78620) map
0x400d0018: _stext at ??:?

I (183) boot: Loaded app from partition at offset 0x10000
I (183) boot: Disabling RNG early entropy source...
I (184) cpu_start: Pro cpu up.
I (187) cpu_start: Starting app cpu, entry point is 0x40080e00
0x40080e00: call_start_cpu1 at esp-idf/components/esp32/./cpu_start.c:222

I (179) cpu_start: App cpu up.
I (198) heap_init: Initializing. RAM available for dynamic allocation:
I (205) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (211) heap_init: At 3FFB2958 len 0002D6A8 (181 KiB): DRAM
I (217) heap_init: At 3FFE0440 len 00003BC0 (14 KiB): D/IRAM
I (223) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
I (230) heap_init: At 40088674 len 0001798C (94 KiB): IRAM
I (236) cpu_start: Pro cpu start user code
I (254) cpu_start: Starting scheduler on PRO CPU.
I (0) cpu_start: Starting scheduler on APP CPU.
Hello world!
This is ESP32 chip with 2 CPU cores, WiFi/BT/BLE, silicon revision 1, 4MB external flash
Restarting in 10 seconds...
Restarting in 9 seconds...
Restarting in 8 seconds...
Restarting in 7 seconds...
Restarting in 6 seconds...
Restarting in 5 seconds...
Restarting in 4 seconds...
Restarting in 3 seconds...
Restarting in 2 seconds...
Restarting in 1 seconds...
Restarting in 0 seconds...
Restarting now.
```

The hello world starts a [FreeRTOS](https://www.freertos.org) task. You may have heard recently [FreeRTOS has joined Amazon](https://aws.amazon.com/blogs/opensource/announcing-freertos-kernel-v10/). It's a [real-time operating system](https://en.wikipedia.org/wiki/Real-time_operating_system):

> A real-time operating system (RTOS) is an operating system (OS) intended to serve real-time applications that process data as it comes in, typically without buffer delays.

## Bluetooth

How about a more interesting example? A new feature to the ESP32 (over ESP8266) is Bluetooth: [https://github.com/espressif/esp-idf/tree/master/examples/bluetooth](https://github.com/espressif/esp-idf/tree/master/examples/bluetooth). There is a [gatt_server](https://github.com/espressif/esp-idf/blob/master/examples/bluetooth/gatt_server/main/gatts_demo.c) demo. What is GATT? Generic Attribute Profile, Adafruit explains: [Introduction to Bluetooth Low Energy](https://learn.adafruit.com/introduction-to-bluetooth-low-energy/gatt). 

> The peripheral is known as the GATT Server, which holds the ATT lookup data and service and characteristic definitions, and the GATT Client (the phone/tablet), which sends requests to this server.

What Bluetooth peripherals can exist? Wikipedia has an incomplete but lengthy [list of Bluetooth profiles](https://en.wikipedia.org/wiki/List_of_Bluetooth_profiles). Keyboards and mice fall under the category of HID, Human Interface Device, and lightly wrap USB's HID. Allowing a Bluetooth keyboard to connect to the ESP32 may require a bit of work. Found [asterics/esp32_mouse_keyboard](https://github.com/asterics/esp32_mouse_keyboard) but it is for making the ESP32 act as a keyboard and mouse, not connecting to an existing keyboard or mice. Clone it anyways and run `make`, accept all defaults. Edit `sdkconfig` and change `CONFIG_ESPTOOLPY_PORT` to the appropriate port in `/dev/cu.*`, then `make flash`. Flashes without error, then watch with `make monitor`, see this:

```sh
esp32_mouse_keyboard $ make monitor
MONITOR
--- idf_monitor on /dev/cu.wchusbserial1420 115200 ---
--- Quit: Ctrl+] | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H ---
ets Jun  8 2016 00:22:57

rst:0x1 (POWERON_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)
configsip: 0, SPIWP:0xee
clk_drv:0x00,q_drv:0x00,d_drv:0x00,cs0_drv:0x00,hd_drv:0x00,wp_drv:0x00
mode:DIO, clock div:1
load:0x3fff0018,len:4
load:0x3fff001c,len:5568
ho 0 tail 12 room 4
load:0x40078000,len:0
load:0x40078000,len:13700
entry 0x4007901c
W (92) rtc_clk: Potentially bogus XTAL frequency: 35 MHz, guessing 40 MHz
W (93) rtc_clk: Possibly invalid CONFIG_ESP32_XTAL_FREQ setting (26MHz). Detected 40 MHz.
```

followed by lots of garbage. Edit sdkconfig change CONFIG_ESP32_XTAL_FREQ to 40. But `make` overwrites it back to 26. Hmm, try changing CONFIG_ESP32_XTAL_FREQ_40=y and CONFIG_ESP32_XTAL_FREQ_26=, this works. `make monitor` shows boot messages then if I type 'a' and press enter:

```
E (446) BT: hid svc handle = 2d
Not connected, ignoring 'a'
received enter, forward command to UART parser
'ot connected, ignoring '
```

Not connected? Ctrl+T, Ctrl-H shows idf_monitor help. This isn't from the monitor tool but the ESP32, main/ble_hidd_demo_main.c:

```c
        if (!sec_conn) {
            printf("Not connected, ignoring '%c'\n", character);
```

Need to connect the ESP32 Bluetooth device to a computer. I checked on my Mac in System Preferences and found a keyboard named FABI:

![Bluetooth FABI](https://user-images.githubusercontent.com/26856618/34087457-16c7b40c-e358-11e7-9398-06fd3a491a79.png)

proving it is working in some capacity. I clicked connect and it spun:

![Bluetooth FABI spinning](https://user-images.githubusercontent.com/26856618/34087477-2f1dc0c8-e358-11e7-9df5-8354bc89273b.png)

but after a few seconds it disconnected. Not sure what is wrong, nothing in the console while I connect. Boot messages:

```sh
$ make monitor
MONITOR
--- idf_monitor on /dev/cu.wchusbserial1420 115200 ---
--- Quit: Ctrl+] | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H ---
ets Jun  8 2016 00:22:57

rst:0x1 (POWERON_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)
configsip: 0, SPIWP:0xee
clk_drv:0x00,q_drv:0x00,d_drv:0x00,cs0_drv:0x00,hd_drv:0x00,wp_drv:0x00
mode:DIO, clock div:1
load:0x3fff0018,len:4
load:0x3fff001c,len:5568
ho 0 tail 12 room 4
load:0x40078000,len:0
load:0x40078000,len:13700
entry 0x4007901c
I (30) boot: ESP-IDF v3.1-dev-69-g8688f0ec 2nd stage bootloader
I (30) boot: compile time 15:40:03
I (39) boot: Enabling RNG early entropy source...
I (39) boot: SPI Speed      : 80MHz
I (40) boot: SPI Mode       : DIO
I (44) boot: SPI Flash Size : 4MB
I (48) boot: Partition Table:
I (52) boot: ## Label            Usage          Type ST Offset   Length
I (59) boot:  0 nvs              WiFi data        01 02 00009000 00006000
I (66) boot:  1 phy_init         RF data          01 01 0000f000 00001000
I (74) boot:  2 factory          factory app     tition table
I (86) esp_image: segment 0: paddr=0x00010020 vaddr=0x3f400020 size=0x3dffc (253948) map
I (168) esp_image: segment 1: paddr=0x0004e024 vaddr=0x3ffc0000 size=0x01fec (  8172) load
I (171) esp_image: segment 2: paddr=0x00050018 vaddr=0x400d0018 size=0x8e050 (581712) map
0x400d0018: _flash_cache_start at ??:?

I (341) esp_image: segment 3: paddr=0x000de070 vaddr=0x3ffc1fec size=0x00450 (  1104) load
I (342) esp_image: segment 4: paddr=0x000de4c8 vaddr=0x40080000 size=0x00400 (  1024) load
0x40080000: _WindowOverflow4 at esp-idf/components/freertos/./xtensa_vectors.S:1685

I (349) esp_image: segment 5: paddr=0x000de8d0 vaddr=0x40080400 size=0x12334 ( 74548) load
I (383) esp_image: segment 6: paddr=0x000f0c0c vaddr=0x400c0000 size=0x00000 (     0) load
I (394) boot: Loaded app from partition at offset 0x10000
I (394) boot: Disabling RNG early entropy source...
I (395) cpu_start: Pro cpu up.
I (399) cpu_start: Single core mode
I (403) heap_init: Initializing. RAM available for dynamic allocation:
I (410) heap_init: At 3FFAFF10 len 000000F0 (0 KiB): DRAM
I (416) heap_init: At 3FFCFB88 len 00010478 (65 KiB): DRAM
I (422) heap_init: At 3FFE0440 len 00003BC0 (14 KiB): D/IRAM
I (429) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
I (435) heap_init: At 40092734 len 0000D8CC (54 KiB): IRAM
I (441) cpu_start: Pro cpu start user code
I (123) cpu_start: Starting scheduler on PRO CPU.
E (126) MAIN: error reading NVS - bt name, setting to FABI
E (126) MAIN: error reading NVS - locale, setting to US_INTERNATIONAL
I (126) system_api: Base MAC address is not set, read default base MAC address from BLK0 of EFUSE
I (416) phy: phy_version: 366.0, ba9923d, Oct 31 2017, 18:06:17, 0, 0
E (446) BT: esp_hidd_prf_cb_hdl(), start added the hid service to the stack database. incl_handle = 40
E (456) BT: hid svc handle = 2d
E (33936) BT: Call back not found for application conn_id=3
```

Callback not found? Printed in red, seems serious. Maybe an error configuring Bluetooth, who knows. Something to look into later, going to move onto getting an OLED display module working first.

## Arduino Adafruit_SSD1306

You may recall I have two OLED display modules which I previously connected to a Raspberry Pi Zero using luma.oled, in *[Monochrome 2.7” and 2.42” 128x64 OLED displays on a Raspberry Pi Zero](https://satoshinm.github.io/blog/171110monochrome_2.7_and_2.42_128x64_oled_displays_on_a_raspberry_pi_zero.html)*. I ended up using the 2.7" display for the custom laptop in *[Building a small custom Raspberry Pi Zero laptop in a cardboard box](https://satoshinm.github.io/blog/171112_building_a_small_custom_raspberry_pi_zero_laptop_in_a_cardboard_box.html)*, which means the 2.42" had been sitting unused. Until now. Time to use it on the ESP32.

Clone [Adafruit_SSD1306](https://github.com/Adafruit/Adafruit_SSD1306) to Arduino/libraries. Apparently I have an SSD1305 not SSD1306, but they seem to interoperate fine. Open up the ssd1305test example and change the pin definitions to:

```c
// Used for software SPI
#define OLED_CLK 23
#define OLED_MOSI 18

// Used for software or hardware SPI
#define OLED_CS 5
#define OLED_DC 19

// Used for I2C or SPI
#define OLED_RESET 17
```

The LOLIN32 has the pins numbered directly on the board. No more D0, D1, D2, etc. pin names, they just use the raw numbers which we use in the code. VP, VN, EN, 34, 35, 32, 33, 25, 26, 27, 14, 12, G, and then 3V, 22, 19, 23, 18, 5, 17, 16, 4, 0, 2, 15, 13. Unordered but I'll take it, I selected the pins above so I could wire the module up all to adjacent pins. Connect the OLED module:
 
| OLED Module Pin | LOLIN ESP32 Pin | Remarks |
| ------------- | ----------- | ------- |
| 1 GND         | G | GND, ground |
| 2 +3V         | 3V | 3.3V, power |
| 4 DC          | 19 | data/command, the fourth wire of "4-wire SPI" (sic) | 
| 7 SCLK        | 23 | serial clock
| 8 DIN         | 18 | MOSI, master out slave in (MISO is not connected, no slave output) |
| 15 /CS        | 5 | Chip enable/select 0 |
| 16 /RES       | 17 | reset |

![Adafruit_SSD1306](https://user-images.githubusercontent.com/26856618/34087029-65f74018-e355-11e7-8ca5-c77fcea98d6e.png)

Build and flash the code. It works! But this is Arduino, I'd like to use the IDF.

## ESP IDF with SSD1306

TarableCode posted native (ESP IDF) code for driving the SSD1306 here: [In progress SSD1306 component for ESP-IDF](https://www.reddit.com/r/esp32/comments/7id5fh/in_progress_ssd1306_component_for_espidf/) and on GitHub: [https://github.com/TaraHoleInIt/ssd1306-esp32-idf-testing](https://github.com/TaraHoleInIt/ssd1306-esp32-idf-testing). Attempt to build with `make` but fails to link:

```sh
LD build/ssd1306-esp32-idf-testing.elf
esp32/ssd1306-esp32-idf-testing/build/main/libmain.a(main.o):(.literal.app_main+0x20): undefined reference to `Font_Liberation_Sans_15x16'
collect2: error: ld returned 1 exit status
make: *** [esp32/ssd1306-esp32-idf-testing/build/ssd1306-esp32-idf-testing.elf] Error 1
```

Looks like Font_Liberation_Sans_15x16 is supposed to be defined in components/tarablessd1306/fonts/font_Liberation_Sans_15x16.c. Reported the [issue #1](https://github.com/TaraHoleInIt/ssd1306-esp32-idf-testing/issues/1) to TaraHoleInit, meanwhile investigating it myself. The project builds using makefiles, the top-level `Makefile` only sets the project name and includes `$(IDF_PATH)/make/project.mk`, as is common for ESP IDF projects. Additionally there are three component makefiles:

```
./components/tarablessd1306/component.mk
./components/tarableina226/component.mk
./main/component.mk
```

The main one is empty except for a comment "(Uses default behaviour of compiling all source files in directory, adding 'include' to include path.)". tarablessd1306/component.mk sets `COMPONENT_SRCDIRS := . fonts`. Refer to the [ESP32 Programming Guide: Build System](https://esp-idf.readthedocs.io/en/v1.0/build_system.html):

> This document explains the Espressif IoT Development Framework build system and the concept of “components”

> COMPONENT_SRCDIRS: Directory paths, must be relative to the component directory, which will be searched for source files (`*.cpp, *.c, *.S`). Defaults to ‘.’, ie the component directory itself. Override this to specify a different list of directories which contain source files.

The problem turned out to be in components/tarablessd1306/include/config.h, after defining the liberation font we can build:

```diff
--- a/components/tarablessd1306/include/config.h
+++ b/components/tarablessd1306/include/config.h
@@ -2,7 +2,7 @@
 #define _CONFIG_H_
 
 //#define CONFIG_FONT_COMIC_NEUE_25x28
-//#define CONFIG_FONT_LIBERATION_SANS_15x16
+#define CONFIG_FONT_LIBERATION_SANS_15x16
 #define CONFIG_FONT_LIBERATION_SERIF_19x19
 #define CONFIG_FONT_UBUNTU_MONO_6x10
```
 
To flash, edit sdkconfig and change CONFIG_ESPTOOLPY_PORT to your serial port, then `make flash`. Builds and flashes but what pins is it expecting? From [main/main.c](https://github.com/TaraHoleInIt/ssd1306-esp32-idf-testing/blob/master/main/main.c#L68):
 
```c
const int RSTPin = 5;
const int DCPin = 16;
const int CSPin = 4;
const int SCLPin = 22;
const int SDAPin = 21;
```

To avoid having to rewire, changed to my own pin definitions:

```c
const int RSTPin = 17; 
const int DCPin = 19;
const int CSPin = 5;
const int SCLPin = 23;
const int SDAPin = 18;
```

To observe, `make monitor`, but I get the same junk on output and a crystal warning; edit CONFIG_ESP32_XTAL_FREQ_40 and change to y and CONFIG_ESP32_XTAL_FREQ_26 to blank, then rebuild and reflash. Why isn't 40 MHz the default in the ESP IDF if it seems more common (at least, it is the default in this inexpensive WEMOS LOLIN32 ESP32 board) than the slower 26 MHz? Anyways, after making this change I can see the sd1306-esp32-idf-testing output:

```
Ready...
V (410) intr_alloc: esp_intr_alloc_intrstatus (cpu 0): checking args
V (420) intr_alloc: esp_intr_alloc_intrstatus (cpu 0): Args okay. Resulting flags 0xE
D (420) intr_alloc: Connected src 50 to int 13 (cpu 0)
i2c master initialized.
i2c display initialized.
V (440) intr_alloc: esp_intr_alloc_intrstatus (cpu 0): checking args
V (440) intr_alloc: esp_intr_alloc_intrstatus (cpu 0): Args okay. Resulting flags 0x80E
D (450) intr_alloc: Connected src 30 to int 17 (cpu 0)
SPI Master Init OK.
Here: Device handle = 0x3FFB69E8
Done resetted
SSD1306 Init OK.
Span created!
```

except there is no output on the OLED. What is this about I2C? The [Monochrome 2.42" 128x64 OLED Graphic Display Module Kit](https://www.adafruit.com/product/2719) has a SSD1305 driver chip which supports 8-bit, I2C, or SPI. I had previously soldered the jumpers on this module to select SPI (it came from the factory configured to use 8-bit parallel). So this code supports both I2C and SPI, and writes to both - but on what pins? Looks like SDAPin and SCLPin are for I2C only:

```c
    if ( ESP32_InitI2CMaster( SDAPin, SCLPin ) ) {
        printf( "i2c master initialized.\n" );

        if ( SSD1306_Init_I2C( &Dev_I2C, 128, 64, 0x3C, 0, ESP32_WriteCommand_I2C, ESP32_WriteData_I2C, NULL ) == 1 ) {
            printf( "i2c display initialized.\n" );
            Screen0 = true;

            //SSD1306_SetFont( &Dev_I2C, &Font_Comic_Neue_25x28 );
            //FontDrawAnchoredString( &Dev_I2C, "Smile!", TextAnchor_Center, true );

            //SSD1306_Update( &Dev_I2C );
        }
    }

    if ( ESP32_InitSPIMaster( DCPin ) ) {
        printf( "SPI Master Init OK.\n" );

        if ( ESP32_AddDevice_SPI( &Dev_SPI, 128, 64, CSPin, RSTPin ) == 1 ) {
            printf( "SSD1306 Init OK.\n" );
            Screen1 = true;

            //SSD1306_SetFont( &Dev_SPI, &Font_Comic_Neue_25x28 );
            //FontDrawAnchoredString( &Dev_SPI, "Okay.", TextAnchor_Center, true );

            //SSD1306_Update( &Dev_SPI );
        }
    }
```

components/tarablessd1306/iface_esp32_spi.c sets these pins for MOSI and SCK:

```c
static const int MOSIPin = 18;
static const int SCKPin = 23;
```

change to my pins:

```c
static const int MOSIPin = 18;
static const int SCKPin = 23;
```

and back in main/main.c set SCLPin and SDAPin to -1 to try to not use them since we are not using I2C. `make && make flash` but there is an error:

```
i2c master initialized.
E (1430) i2c: esp32/esp-idf/components/driver/./i2c.c:489 (i2c_master_clear_bus):scl gpio number error
```

[Comment out](https://github.com/satoshinm/ssd1306-esp32-idf-testing/commit/cd07a3a5b3e7a75da1ad252edbdf02baf717a43a) all the I2C code, and it works!

![ssd1306-esp32-idf-testing](https://user-images.githubusercontent.com/26856618/34087055-8a778376-e355-11e7-9795-acadb917bc6f.png)

Scrolls the text "Okay" on the screen as we expect. But due to how I positioned the module, it is up-side down (I had to carefully take this picture and reverse it to get the text to show up straight). Fortunately this can be fixed in software by enabling horizontal and vertical flipping:

```c
            SSD1306_SetHFlip( &Dev_SPI, true );
            SSD1306_SetVFlip( &Dev_SPI, true );
```

I also changed the text to a friendly greeting. All my code changes: [https://github.com/satoshinm/ssd1306-esp32-idf-testing/tree/mine](https://github.com/satoshinm/ssd1306-esp32-idf-testing/tree/mine). Here is the result:

![ESP32 flipped hello](https://user-images.githubusercontent.com/26856618/34087919-f4693d1a-e35a-11e7-9c7d-c6a1929de0d7.png)

This concludes this blog post. After having gotten acquainted with the ESP IDF, now it is time to think of a project to do with it.
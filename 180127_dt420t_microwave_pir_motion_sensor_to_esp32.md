# DT420T microwave/PIR motion sensor to ESP32

by snm, January 27th, 2018

* auto-gen TOC:
{:toc}

In this article I'll setup the ESP32 with a DualTec IV DT420T motion detector by C&K Systems, which bills itself as "The Most Intelligent Detection Technology". Combining a [passive infrared sensor](https://en.wikipedia.org/wiki/Passive_infrared_sensor) (PIR) with a microwave [motion detector](https://en.wikipedia.org/wiki/Motion_detector), it is part of series with the low-end module covering an area of 20' x 20'.

## Unboxing and configuring the DT420T

![Box](https://user-images.githubusercontent.com/26856618/35476580-4526cfd4-0367-11e8-9d31-4cd8f5118340.png)

Screw terminals, power with 12V DC. C and NC contacts short together under the alarm condition, otherwise open. Crack it open:

![Inside](https://user-images.githubusercontent.com/26856618/35476631-f6d3e078-0367-11e8-942c-dbc13b8b6d99.png)

Wire in some cables, red and green for power, yellow and black for the alarm:

![Motion detector with cables](https://user-images.githubusercontent.com/26856618/35476615-c2d19478-0367-11e8-91e2-b3ecaac8e137.png)

Read the documentation:

![Literature page 1](https://user-images.githubusercontent.com/26856618/35476600-8a24ee4a-0367-11e8-84b2-0958bba0c1d3.png)

second page:

![Literature page 2](https://user-images.githubusercontent.com/26856618/35476604-ab9f4cd2-0367-11e8-8e7d-844815598412.png)

The green LED turns on when the PIR sensor activates, the yellow LED for microwave, and red when the alarm condition triggers. The microwave uses a 10.525 GHz center frequency, with adjustable range. This is part of the [X-band](https://en.wikipedia.org/wiki/X_band#Radar), from Wikipedia:

> Motion detectors often use 10.525 GHz. 10.4 GHz is proposed for traffic light crossing detectors. Comreg in Ireland has allocated 10.450 GHz for Traffic Sensors as SRD.

Walk in front of the sensor, confirm the lights light up. Now to wire to a system.

## Connecting to a Raspberry Pi

Wire up to a GPIO input with internal pull-up resistors, normally-closed, and common to ground. Detect edges with a simple Python script on the Raspberry Pi:

```python
#!/usr/bin/python
# Connects via GPIO to DualTec IV DT420T motion sensor (PIR/microwave)

import RPi.GPIO as GPIO
import signal

GPIO.setmode(GPIO.BOARD)

# CE1 = BCM 7 = 26
SENSOR = 26 # G7

GPIO.setwarnings(False)
GPIO.setup([SENSOR], GPIO.IN, pull_up_down=GPIO.PUD_UP)

def handle(pin):
	print pin,GPIO.input(pin)

GPIO.add_event_detect(SENSOR, GPIO.BOTH, handle)

signal.pause()
```

But would really like to connect to the ESP32. 


## Connecting to an ESP32

The device I used most recently in *[ESP32-based wireless ticker display for Minecraft server activity: pushing up against the fourth wall](https://satoshinm.github.io/blog/180107_esp32ticker_esp32_based_wireless_ticker_display_for_minecraft_server_activity_pushing_up_against_the_fourth_wall.html)* and first in *[Espressif IDF IoT Development Framework on the WEMOS LOLIN32 ESP32 to drive an SSD1305 OLED display over SPI without Arduino](https://satoshinm.github.io/blog/171217_esp32idf_espressif_idf_iot_development_framework_on_the_wemos_lolin32_esp32_to_drive_an_ssd1305_oled_display_over_spi_without_arduino.html)*. Had it wired up on a breadboard:

![ESP32 breadboard](https://user-images.githubusercontent.com/26856618/35476334-01710a74-0363-11e8-93c9-0a50a540024d.png)

to make space, moved it off the breadboard wiring directly using female-to-female jumpers:

| LOLIN ESP32 | Wire color | OLED module |
| ----- | ---------- | ----------- |
| 3V | red | 2 +3V
| 22 | (not connected) | (not connected)
| 19 | yellow | 4 DC
| 23 | white (was brown) | 7 SCLK
| 18 | green | 8 DIN
| 5 | blue | 15 /CS
| 17 | orange (was purple) | 16 /RES
| G | black | 1 GND

When I first wired it up, mistakingly reversed polarity of ground and power. The ESP32 board started smoking! But I quickly disconnected it, corrected the polarity, and it all miraculously survived:

![ESP32 + OLED off breadboard](https://user-images.githubusercontent.com/26856618/35476357-7d9c361e-0363-11e8-8161-d4807ab96698.png)

Now to wire up the motion sensor to a GPIO pin with an internal pull-up resistor. What ESP32 GPIO pin? The documentation [GPIO & RTC GPIO](https://esp-idf.readthedocs.io/en/latest/api-reference/peripherals/gpio.html) refers to the [peripherals/gpio](https://github.com/espressif/esp-idf/blob/daa8cfa800677a71a3447eba418a4336979a3cb8/examples/peripherals/gpio/main/gpio_example_main.c) example, which configures the pins as follows:

```c
/**
 * Brief:
 * This test code shows how to configure gpio and how to use gpio interrupt.
 *
 * GPIO status:
 * GPIO18: output
 * GPIO19: output
 * GPIO4:  input, pulled up, interrupt from rising edge and falling edge
 * GPIO5:  input, pulled up, interrupt from rising edge.
 *
 * Test:
 * Connect GPIO18 with GPIO4
 * Connect GPIO19 with GPIO5
 * Generate pulses on GPIO18/19, that triggers interrupt on GPIO4/5
 *
 */
 ```
 
I'm already using 18, 19, and 5 for the OLED module. But I can use 4 for input, and for good measure, use 0 as output, something to measure with the oscilloscope built in *[Building an Amazing $10 Oscilloscope with an STM32 blue pill, LCD touchscreen, and STM32-O-Scope software](https://satoshinm.github.io/blog/180105_stm32scope_building_an_amazing_10_oscilloscope_with_an_stm32_blue_pill_lcd_touchscreen_and_stm32-o-scope_software.html)*. Remove the `vTaskDelay(1000 / portTICK_RATE_MS);` so the GPIO output twiddles as fast as possible, run `make flash && make monitor`, we see it logging each oscillation count, and can observe this square wave using the oscilloscope:

![GPIO output on scope](https://user-images.githubusercontent.com/26856618/35477848-461cd88c-0382-11e8-92f4-e01f4afaaebe.png)

so the GPIO output works. Committed the working modified example here: [https://github.com/satoshinm/ssd1306-esp32-idf-testing/commit/f173978183b0b8986224fa3ee3195ea02142f673](https://github.com/satoshinm/ssd1306-esp32-idf-testing/commit/f173978183b0b8986224fa3ee3195ea02142f673), since that's the repository I had available on hand. Remove the output, leaving only GPIO input, and configure to trigger an interrupt on any edge (both rising and falling), slightly simplified from the example:

```c
    gpio_config_t io_conf;

    io_conf.intr_type = GPIO_PIN_INTR_ANYEDGE;
    io_conf.pin_bit_mask = ((1ULL<<GPIO_INPUT_IO_0));
    io_conf.mode = GPIO_MODE_INPUT;
    io_conf.pull_down_en = 0;
    io_conf.pull_up_en = 1;
    gpio_config(&io_conf);
 ```
 
 then use a FreeRTOS queue and task to handle the interrupt, this is straight from Espressif's example:
 
 ```c
     //create a queue to handle gpio event from isr
    gpio_evt_queue = xQueueCreate(10, sizeof(uint32_t));
    //start gpio task
    xTaskCreate(gpio_task_example, "gpio_task_example", 2048, NULL, 10, NULL);
    
    //install gpio isr service
    gpio_install_isr_service(ESP_INTR_FLAG_DEFAULT);
    //hook isr handler for specific gpio pin
    gpio_isr_handler_add(GPIO_INPUT_IO_0, gpio_isr_handler, (void*) GPIO_INPUT_IO_0);
```

Actually I added the GPIO output on pin 0 back, but the level always set to 0, because it wasn't convenient to run another wire from ground. To trigger the alarm we'll short pin 4 to 0. 

```c
    // GPIO output
    io_conf.intr_type = GPIO_PIN_INTR_DISABLE;
    io_conf.pin_bit_mask = ((1ULL<<GPIO_OUTPUT_IO_0));
    io_conf.mode = GPIO_MODE_OUTPUT;
    io_conf.pull_down_en = 0;
    io_conf.pull_up_en = 0;
    gpio_config(&io_conf);
    gpio_set_level(GPIO_OUTPUT_IO_0, 0);
```

Test with a piece of wire, it crashes the ESP32:

```
/esp-idf/components/freertos/./queue.c:1156 (xQueueGenericSendFromISR)- assert failed!
abort() was called at PC 0x40085650 on core 0
0x40085650: xQueueGenericSendFromISR at /Users/admin/rf/esp32/esp-idf/components/freertos/./queue.c:2037


Backtrace: 0x4008853c:0x3ffb0560 0x4008863b:0x3ffb0580 0x40085650:0x3ffb05a0 0x40083057:0x3ffb05c0 0x400843f2:0x3ffb05f0 0x40082979:0x3ffb0610 0x400d2317:0x00000000
0x4008853c: invoke_abort at /Users/admin/rf/esp32/esp-idf/components/esp32/./panic.c:572

0x4008863b: abort at /Users/admin/rf/esp32/esp-idf/components/esp32/./panic.c:572

0x40085650: xQueueGenericSendFromISR at /Users/admin/rf/esp32/esp-idf/components/freertos/./queue.c:2037

0x40083057: gpio_isr_handler at /Users/admin/rf/esp32/ssd1306-esp32-idf-testing/main/./main.c:473 (discriminator 4)

0x400843f2: gpio_intr_service at /Users/admin/rf/esp32/esp-idf/components/driver/./gpio.c:428

0x40082979: _xt_lowint1 at /Users/admin/rf/esp32/esp-idf/components/freertos/./xtensa_vectors.S:1105

0x400d2317: esp_vApplicationIdleHook at /Users/admin/rf/esp32/esp-idf/components/esp32/./freertos_hooks.c:85


Rebooting...
```

What is it asserting? Looking at the source esp-idf/components/freertos/./queue.c:

```c
BaseType_t xQueueGenericSendFromISR( QueueHandle_t xQueue, const void * const pvItemToQueue, BaseType_t * const pxHigherPriorityTaskWoken, const BaseType_t xCopyPosition )
{
BaseType_t xReturn;
UBaseType_t uxSavedInterruptStatus;
Queue_t * const pxQueue = ( Queue_t * ) xQueue;

    configASSERT( pxQueue );
```

xQueueSendFromISR() is called from gpio_isr_handler() with `gpio_evt_queue` as the queue, which should have been initialized with xQueueCreate(). Test the [gpio_example_main](https://github.com/espressif/esp-idf/blob/daa8cfa800677a71a3447eba418a4336979a3cb8/examples/peripherals/gpio/main/gpio_example_main.c), it works, must be missing something. Commenting out of all the OLED (SSD1306) code fixes the crash, a conflict of some sort. How about polling the queue in the ShiftTask instead? Changed the last parameter from portMAX_DELAY to 0 to not wait:

```c
        uint32_t io_num;
        if(xQueueReceive(gpio_evt_queue, &io_num, 0)) {
            printf("GPIO[%d] intr, val: %d\n", io_num, gpio_get_level(io_num));
        }
```

but it crashes again:

```
Guru Meditation Error: Core  0 panic'ed (StoreProhibited)
. Exception was unhandled.
Register dump:
PC      : 0x40086138  PS      : 0x00060133  A0      : 0x8008582f  A1      : 0x3ffc98c0  
0x40086138: uxPortCompareSet at esp32/esp-idf/components/freertos/./tasks.c:3529
 (inlined by) vPortCPUAcquireMutexIntsDisabled at esp32/esp-idf/components/freertos/./portmux_impl.h:86
 (inlined by) vTaskEnterCritical at esp32/esp-idf/components/freertos/./tasks.c:4211

A2      : 0x00a5a5ed  A3      : 0x00000000  A4      : 0x00000000  A5      : 0x00000000  
A6      : 0x00000000  A7      : 0x00000000  A8      : 0x0000cdcd  A9      : 0x0000cdcd  
A10     : 0xb33fffff  A11     : 0x0000abab  A12     : 0x00060120  A13     : 0x00000001  
A14     : 0x00060120  A15     : 0x00060a23  SAR     : 0x0000000a  EXCCAUSE: 0x0000001d  
EXCVADDR: 0x00a5a5ed  LBEG    : 0x4000c2e0  LEND    : 0x4000c2f6  LCOUNT  : 0xffffffff  

Backtrace: 0x40086138:0x3ffc98c0 0x4008582c:0x3ffc98e0 0x4010a6d5:0x3ffc9920
0x40086138: uxPortCompareSet at esp32/esp-idf/components/freertos/./tasks.c:3529
 (inlined by) vPortCPUAcquireMutexIntsDisabled at esp32/esp-idf/components/freertos/./portmux_impl.h:86
 (inlined by) vTaskEnterCritical at esp32/esp-idf/components/freertos/./tasks.c:4211

0x4008582c: xQueueGenericReceive at esp32/esp-idf/components/freertos/./queue.c:2037

0x4010a6d5: ShiftTask at esp32/ssd1306-esp32-idf-testing/main/./main.c:480 (discriminator 4)


Rebooting...
```

Back to a separate task, isolate the problem further. The problem is definitely related to an interaction with ShiftTask(), if I don't create it, no scrolling text but no crash either. Would be a good idea to investigate and fix it but for now disabled the scrolling, as this also now fixes changing the text dynamically. Show the GPIO input value:

```c
static void gpio_task_example(void* arg)
{
    uint32_t io_num;
    for(;;) {
        if(xQueueReceive(gpio_evt_queue, &io_num, portMAX_DELAY)) {
            printf("GPIO[%d] intr, val: %d\n", io_num, gpio_get_level(io_num));

            if (gpio_get_level(io_num)) {
                drawString("   on");
            } else {
                drawString("   off");
            }   
        }
    }
}
```

well, it worked for a while. Toggling too many times:

```
Span created!
GPIO[4] intr, val: 1
Span created!
GPIO[4] intr, val: 1
Span created!
GPIO[4] intr, val: 1
Span created!
GPIO[4] intr, val: 1
Span created!
GPIO[4] intr, val: 1
GPIO[4] intr, val: 1
GPIO[4] intr, val: 1
GPIO[4] intr, val: 1
GPIO[4] intr, val: 0
GPIO[4] intr, val: 0
```

Stops at 108 iterations. Maybe a memory leak? Is `Virt_DeviceInit()` needed each time a string is drawn? Moved to main but no output. Not really understanding this code too well, I admit. This will need some more tinkering and development to get working well. But the purpose was to use the DT420T motion sensor, so let's do that now (can fix software bugs later):

![DT420T connected to ESP32](https://user-images.githubusercontent.com/26856618/35478637-64c0dce2-0397-11e8-8dca-338fc9ac3d2c.png)

Finally, I changed the strings to revert back to the server status string when motion detection turned off, and a greeting message when it turned on, and as a last-ditched hack, automatically restart after triggering over 100 times to workaround the aforementioned (memory leak?) bug. 

## Conclusions and future directions

The DT420T module is straightforward to interface to a microcontroller, with its normally-closed and common terminals. The 12V power supply is somewhat inconvenient however, since most microcontrollers these days run on 3.3V, usually through a 5V USB plug. Either stepping down the 12V to 5V, or boosting 5V to 12V, would be an obvious improvement to avoid having two power adapters and wall plugs, which severely constrains where I can use this apparatus, without adding cumbersome power strips.

My software for the ESP32 is a start (available if you are interested at [https://github.com/satoshinm/ssd1306-esp32-idf-testing/tree/mine](https://github.com/satoshinm/ssd1306-esp32-idf-testing/tree/mine)), but relatively rough, and could be greatly improved such as by sending a packet when the motion triggers (for integration into a home automation system), or sounding an auditory alarm. The ESP32, after all, has both Wi-Fi and a DAC (digital to analog converter, for generating sound waves). There is no shortage of potential with the ESP32, but like all good things, this blog post has to come to an end now.
# ESP32-based wireless ticker display for Minecraft server activity: pushing up against the fourth wall

by snm, January 7th, 2018

The computer game [Minecraft](https://en.wikipedia.org/wiki/Minecraft) has been wildly successful, especially as a platform for modders, partly because modifying it allows you to create new mechanisms, phenomena, and behavior in a *virtual* world, expressing creativity unconstrained by physical limitations of engineering in the real world, as you are with [Arduino](https://en.wikipedia.org/wiki/Arduino) or other hardware platforms interfacing with the tangible. But what if we could bridge the two, the virtual and the physical?

One of the earliest pioneers in this field, as far as I could find, were experimenting with this idea back in 2014. For example syntagmata posted this to /r/minecraft on October 3rd, 2013: [App syncs Philips Hue light bulbs to Minecraft](https://www.reddit.com/r/Minecraft/comments/1nnijo/app_syncs_philips_hue_light_bulbs_to_minecraft/). The vine link video is now dead, but it was a demonstration of synchronizing the [Philips Hue](https://en.wikipedia.org/wiki/Philips_Hue) smart light bulbs in the real world to the virtual world of Minecraft. You could imagine the lighting of your surroundings around your computer changing for a more immersive experience in the game world.

Blodjer posted to /r/huelights (since replaced by /r/hue) on August 20th, 2014, another take on it this concept: [Hue lights sync to your screen](https://www.reddit.com/r/huelights/comments/2e3vq9/hue_lights_sync_to_your_screen/). Another logical progression is to allow an in-game action to control a real-world peripheral, such as a lever operating a desk lamp. Slumber_watcher posted to /r/minecraft on May 2nd, 2014: [Desk lamp controlled from Minecraft lamp controlled by phone. (I suck at titles) (More info in the comments)](https://www.reddit.com/r/Minecraft/comments/24jffn/desk_lamp_controlled_from_minecraft_lamp/?sort=top). 

>  Made this a while back. The minecraft server is a regular one without any mods and a webserver that can run cgi-scripts.
How it works: The lever not only controlls the lamp but also sends a signal to command blocks that writes to the server log if the lamp is powered or not. The logfile is read by a script that then sends a message (via the serial port) to an arduino with one of [these](http://www.dx.com/p/433mhz-wireless-transmitter-module-superregeneration-for-arduino-green-149254). The arduino then sends a signal to the desk lamp, connected to one of [these](http://www.rusta.com/fjarrstrombrytare-3i1-vit-p92201055.aspx).

> To make the phone controll the redstone lamp, it simply make the script running on the webserver send setblock-commands to change the lever.

and deepsheet to /r/minecraft on November 22nd, 2015: [Controlling a WiFi bulb with an in-game lever](https://www.reddit.com/r/Minecraft/comments/3ttu48/controlling_a_wifi_bulb_with_an_ingame_lever/), by giannoug. The idea also came up to control your Minecraft lights with a real light switch in your house. But the title of giannoug's blog post nails the concept well: [Breaking the fourth wall with Minecraft](https://hashbang.gr/breaking-the-4th-wall-with-minecraft/). The [fourth wall](https://en.wikipedia.org/wiki/Fourth_wall):

> The fourth wall is a performance convention in which an invisible, imagined wall separates actors from the audience. While the audience can see through this "wall", the convention assumes, the actors act as if they cannot. From the 16th century onwards, the rise of illusionism in staging practices, which culminated in the realism and naturalism of the theatre of the 19th century, led to the development of the fourth wall concept.

> ...

> "Breaking the fourth wall" is any instance in which this performance convention, having been adopted more generally in the drama, is violated. This can be done through either directly referencing the audience, the play as a play, or the characters' fictionality. ... [metatheatrical](https://en.wikipedia.org/wiki/Metatheatre). A similar effect of [metareference](https://en.wikipedia.org/wiki/Meta-reference) is achieved when the performance convention of avoiding direct contact with the camera, generally used by actors in a television drama or film, is temporarily suspended. The phrase "breaking the fourth wall" is used to describe such effects in those media. Breaking the fourth wall is also possible in other media, such as video games and books.

Making some modern changes to the Wikipedia diagram:

![Fourth wall, computer games](https://user-images.githubusercontent.com/26856618/34658860-7f05193e-f3e7-11e7-8eb7-747ee7cf2562.png)

So what can we do to achieve this? There are a lot of possible ideas (usually revolving around lighting and switches), but as a start, let's try something simple.

## Ticker

![ESP32 Ticker - has left the game](https://user-images.githubusercontent.com/26856618/34658467-2ed399de-f3e4-11e7-8ed5-ecfd8e7719f3.png)

Looking to find a use for the ESP32 I first setup in *[Espressif IDF IoT Development Framework on the WEMOS LOLIN32 ESP32 to drive an SSD1305 OLED display over SPI without Arduino](https://satoshinm.github.io/blog/171217_esp32idf_espressif_idf_iot_development_framework_on_the_wemos_lolin32_esp32_to_drive_an_ssd1305_oled_display_over_spi_without_arduino.html)*, settled on developing a wireless "ticker" display. It'll simply show the last line of the Minecraft server log, updated regularly.

In the photo above, we see someone has left the game. The image is blurry, seemingly due to the high (infinite) contrast of the OLED, but in real life it looks sharp and crisp. It scrolls across the screen to get your attention.

To build it, networking capability will need to be added to the ESP32 OLED example I'm starting with. The device will connect to the Wi-Fi network, fetch the last server log message, then display it. Refer to the [ESP-IDF Programming Guide: Wi-Fi](https://esp-idf.readthedocs.io/en/latest/api-reference/wifi/esp_wifi.html), and the [esp-idf-template](https://github.com/espressif/esp-idf-template/blob/master/main/main.c) example. Adding to the example, `make flash` then `make monitor` and watch it connect:

```sh
Connecting to network...
D (183) nvs: nvs_flash_init_custom partition=nvs start=9 count=6
D (253) nvs: nvs_open_from_partition misc 1
D (253) nvs: nvs_get_str_or_blob log
I (253) wifi: wifi firmware version: f204566
I (253) wifi: config NVS flash: enabled
I (263) wifi: config nano formating: disabled
I (263) system_api: Base MAC address is not set, read default base MAC address from BLK0 of EFUSE
I (273) system_api: Base MAC address is not set, read default base MAC address from BLK0 of EFUSE
D (283) nvs: nvs_open_from_partition nvs.net80211 1
D (283) nvs: nvs_get opmode 1
D (293) nvs: nvs_get_str_or_blob sta.ssid
D (293) nvs: nvs_get_str_or_blob sta.mac
D (303) nvs: nvs_get sta.authmode 1
D (303) nvs: nvs_get_str_or_blob sta.pswd
D (303) nvs: nvs_get_str_or_blob sta.pmk
D (313) nvs: nvs_get sta.chan 1
D (313) nvs: nvs_get auto.conn 1
D (313) nvs: nvs_get bssid.set 1
D (323) nvs: nvs_get_str_or_blob sta.bssid
D (323) nvs: nvs_get sta.phym 1
D (333) nvs: nvs_get sta.phybw 1
D (333) nvs: nvs_get_str_or_blob sta.apsw
D (333) nvs: nvs_get_str_or_blob sta.apinfo
D (343) nvs: nvs_get sta.scan_method 1
D (343) nvs: nvs_get sta.sort_method 1
D (343) nvs: nvs_get sta.minrssi 1
D (353) nvs: nvs_get sta.minauth 1
D (353) nvs: nvs_get_str_or_blob ap.ssid
D (363) nvs: nvs_get_str_or_blob ap.mac
D (363) nvs: nvs_get_str_or_blob ap.passwd
D (363) nvs: nvs_get_str_or_blob ap.pmk
D (373) nvs: nvs_get ap.chan 1
D (373) nvs: nvs_get ap.authmode 1
D (373) nvs: nvs_get ap.hidden 1
D (383) nvs: nvs_get ap.max.conn 1
D (383) nvs: nvs_get bcn.interval 2
D (383) nvs: nvs_get ap.phym 1
D (393) nvs: nvs_get ap.phybw 1
D (393) nvs: nvs_get ap.sndchan 1
D (393) nvs: nvs_set_blob sta.mac 6
D (413) nvs: nvs_set_blob ap.mac 6
I (413) wifi: Init dynamic tx buffer num: 32
I (413) wifi: Init data frame dynamic rx buffer num: 32
I (413) wifi: Init management frame dynamic rx buffer num: 32
I (423) wifi: wifi driver task: 3ffc1220, prio:23, stack:4096
I (423) wifi: Init static rx buffer num: 10
I (423) wifi: Init dynamic rx buffer num: 32
I (433) wifi: wifi power manager task: 0x3ffc5e60 prio: 21 stack: 2560
D (443) RTC_MODULE: Wi-Fi takes adc2 lock.
D (443) phy_init: loading PHY init data from application binary
D (443) nvs: nvs_open_from_partition phy 0
D (453) nvs: nvs_get cal_version 4
V (453) phy_init: phy_get_rf_cal_version: 366

D (463) nvs: nvs_get_str_or_blob cal_mac
D (463) nvs: nvs_get_str_or_blob cal_data
D (473) nvs: nvs_close 3
V (473) phy_init: register_chipv7_phy, init_data=0x3f414148, cal_data=0x3ffc6084, mode=0
I (513) phy: phy_version: 366.0, ba9923d, Oct 31 2017, 18:06:17, 0, 0
I (513) wifi: mode : sta (30:...)
D (513) event: SYSTEM_EVENT_STA_START
V (523) event: enter default callback
D (523) tcpip_adapter: check: local, if=0 fn=0x401192bc
0x401192bc: tcpip_adapter_start_api at esp32/esp-idf/components/tcpip_adapter/./tcpip_adapter_lwip.c:1065


Ready...
D (523) tcpip_adapter: call api in lwip: ret=0x0, give sem
D (533) tcpip_adapter: check: remote, if=0 fn=0x401192bc
0x401192bc: tcpip_adapter_start_api at esp32/esp-idf/components/tcpip_adapter/./tcpip_adapter_lwip.c:1065


V (543) event: exit default callback
V (543) intr_alloc: esp_intr_alloc_intrstatus (cpu 0): checking args
V (553) intr_alloc: esp_intr_alloc_intrstatus (cpu 0): Args okay. Resulting flags 0x80E
D (553) intr_alloc: Connected src 30 to int 13 (cpu 0)
SPI Master Init OK.
```

Seems there is an error... what went wrong? Looking at esp32/esp-idf/components/tcpip_adapter/./tcpip_adapter_lwip.c, line 1065 is the end of this function:

```c
esp_err_t tcpip_adapter_eth_input(void *buffer, uint16_t len, void *eb)
{
    ethernetif_input(esp_netif[TCPIP_ADAPTER_IF_ETH], buffer, len);
    return ESP_OK;
}
```

Changed to use the [http_request](https://github.com/espressif/esp-idf/tree/master/examples/protocols/http_request/main) example. On startup the server sends an HTTP request to a web server, then parses and displays the response. On the Minecraft server, run ncat, see [ncat tips and tricks](https://nmap.org/ncat/guide/ncat-tricks.html) - serves the last line of the server log:

```sh
ncat -lk -p 8000 --sh-exec "printf 'HTTP/1.0 200 OK\r\nContent-Type: text/plain\r\nConnection: close\r\n\r\n~'; tail -1 modpack/server/logs/latest.log"
```

To fetch continuously, a quick hack, restarting the system every 30 seconds:

```c
int64_t GetMillis( void ) {
    return esp_timer_get_time( ) / 1000;
}

...
    int64_t VeryStart = GetMillis( );
    
...

        uint64_t TotalElapsed = GetMillis() - VeryStart;
        if (TotalElapsed > 30000) {
            printf("Time elapsed, restarting!\n");
            esp_restart();
        }
```        

This effectively polls the server. What I have so far pushed to [https://github.com/satoshinm/ssd1306-esp32-idf-testing/commits/mine](https://github.com/satoshinm/ssd1306-esp32-idf-testing/commits/mine).

## ESP-IDF vs Arduino dilemma

Espressif makes its own "native" development framework, the [ESP-IDF](https://github.com/espressif/esp-idf), very powerful, but lower-level, so there is also a high-level competitor (which Espressif also maintains the board support package for ESP32): [Ardunio-ESP32](https://github.com/espressif/arduino-esp32). Choosing the path of least resistance, one would go with Arduino, however, as [Sparkfun explains](https://learn.sparkfun.com/tutorials/esp32-thing-hookup-guide), Arduino-ESP32 doesn't yet support the full functionality of the device:

> Not Yet Implemented

> The Arduino board definitions for the ESP32 are still a work in progress. There are a handful of peripherals and features that have yet to be implemented, including:

> * Bluetooth

> * Analog Input (analogRead([pin]))

> * Analog Ouptut (analogWrite([pin], [value]))

> * WiFi Server and WiFI UDP

> * Real-Time Clock

> * Touch-controller interface

> These peripherals are available (if, also, still in their infancy) in the IoT Development Framework for the ESP32. If your application requires Bluetooth, analog input, or any of the features above, consider giving the ESP-IDF a try!

Since Bluetooth is one of the major selling points of the ESP32 over its predecessor the ESP8266, restricting oneself by using Arduino-ESP32 seems a bit backwards. Even if I don't immediately need it, wouldn't want to be limited by the choice of development environment. So the ESP-IDF it is. But the downside is complexity, I haven't yet figured out how to synchronize the scrolling RTOS task with the string updating procedure. As a workaround, I just reboot the device at the refresh interval, causing it to refetch the latest server log message.

This is obviously non-ideal.  Getting the updates pushed would be more efficient, but for now this is what I have working. Lots of other improvements are possible: displaying the users connected, last connected, last chat message, location of users, state of the world/time, state of various blocks. Left as an exercise for the reader. And how about synchronizing your home automation with your Minecraft world? The possibilities are endless. As for me, I'm not too satisfied with this project, not sure what I'll do with it. Surely one can more strongly push and break through the fourth wall.
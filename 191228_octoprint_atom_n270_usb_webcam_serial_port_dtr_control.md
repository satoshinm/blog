# OctoPrint on an Atom N270 with a USB Webcam and Serial Port DTR Control

by snm, December 28th, 2019

* auto-gen TOC:
{:toc}

As mentioned in *[3D Printing with 3D MARS white PLA: pass-through wallplate, paper towel holder, right-angle spool holder, shower curtain rings, and a prototype case](https://satoshinm.github.io/blog/180208_3dprint2_3d_printing_with_3d_mars_white_pla_pass_through_wallplate_paper_towel_holder_right_angle_spool_holder_shower_curtain_rings_and_a_prototype_case.html)*,
I always wanted to install [OctoPrint](https://octoprint.org) as long as I've had this printer,
but until now, I was printing by manually walking over and inserting an SD card. Following
the upgrades covered in *[Glass bed and heater rewire mods to Monoprice Select Mini v2 3D printer](https://satoshinm.github.io/blog/191207_glass_bed_and_heater_rewire_mpsm.html)*, it is finally time to install OctoPrint. This post
details my setup.

## Initial setup

### Atom N270

Lots of people run OctoPrint on a Raspberry Pi 3+, but I happened to have
another embedded system lying around unused I wanted to put to good use.
It runs an [Intel Atom® Processor N270, 512K Cache, 1.60 GHz, 533 MHz FSB](https://ark.intel.com/content/www/us/en/ark/products/36331/intel-atom-processor-n270-512k-cache-1-60-ghz-533-mhz-fsb.html).

What is [Intel Atom](https://en.wikipedia.org/wiki/Intel_Atom)?

> Intel Atom is the brand name for a line of IA-32 and x86-64 instruction set ultra-low-voltage microprocessors by Intel Corporation.

The N270 I have is from the N2xx series, [Diamondville](https://en.wikipedia.org/wiki/Intel_Atom#Diamondville)
code name, with a single 45 nm core, hyperthreading, but no Intel 64, released June 2008.
[Wikipedia: List of Intel Atom microprocessors](https://en.wikipedia.org/wiki/List_of_Intel_Atom_microprocessors)
classifies it has a "netbook (sub-notebook)" processor.

An older processor, but not ancient. The Atom series is still being used,
and was mentioned in [Hackaday: 2019: As the hardware world turns](https://hackaday.com/2019/12/27/2019-as-the-hardware-world-turns/)
year-end review:

> But the Raspberry Pi isn’t the only SBC game in town. It’s not even the only one with a cute “Pi” name, for that matter. [We saw significant interest in the Atomic Pi](https://hackaday.com/2019/05/08/this-atomic-pi-eats-other-pis-for-lunch/), which delivered the power of a quad-core Intel Atom processor at a size and price not far from that of its berry-flavored peer. We’ve since featured [a number of impressive modifications](https://hackaday.com/2019/06/26/how-do-you-get-pci-e-on-the-atomic-pi-very-carefully/) to the powerful board, but the discovery that the Atomic Pi was actually surplus hardware purchased from the now defunct Mayfield Robotics raised some valid questions about the long-term viability of the product.

The Atomic Pi runs an Intel Atom x5-Z8350 (Cherry Trail) at 1.44 GHz, surprisingly
a lower clock speed than my 1.60 GHz N270 (not that clockspeed is all that matters).
Anyways, either are more powerful than the ARMv6 BCM2835 1GHz in the Pi Zero, and
should be sufficient to run OctoPrint. Here's a picture of my board (unknown motherboard, not Atomic) running the Atom N270:

![Atom N270 board](https://user-images.githubusercontent.com/26856618/71550497-f7284c00-29c9-11ea-8ee4-2ff380447564.png)

### OctoPrint

As the Atom N270 is only 32-bit, I installed [Arch Linux 32](https://www.archlinux32.org),
an actively-maintained 32-bit distribution based on [Arch Linux](https://www.archlinux.org).
Many other distributions have dropped 32-bit support, moving onto 64-bit only, so a distribution
had to be selected meeting the architectural requirements. I picked Arch due to its philosophy
as a "lightweight and flexible Linux® distribution that tries to Keep It Simple", which makes
a lot of sense for this application.

After following the very informative [Arch Linux installation guide](https://wiki.archlinux.org/index.php/Installation_guide),
I installed OctoPrint from [https://github.com/foosel/OctoPrint](https://github.com/foosel/OctoPrint#installation),
following the steps as documented. Well, sort of: I used the `rc/devel` branch instead of `master`, and avoided setting up
a virtual Python environment, instead installing everything for Python 3.8.0 (from the `python 3.8.0-1.0` pacman package).

OctoPrint listens on port 5000. The Pi Zero camera I'll setup in the next section is available on 192.168.7.2... a private address
only available to the host PC (this Atom N270) and the Pi Zero itself. To get the camera working on
the web interface, it will have to be accessible from the web. To accomplish this, I installed
[HAproxy](https://wiki.archlinux.org/index.php/HAproxy), and configured it as follows in `/etc/haproxy/haproxy.cfg`:

```
global
    maxconn     20000
    log         127.0.0.1 local0
    user        haproxy
    chroot      /usr/share/haproxy
    pidfile     /run/haproxy.pid
    daemon

frontend http-in
    bind *:80
    mode http
    option httplog
    timeout              client  30s

    acl is_pizero_camera path_beg /pizero_camera
    use_backend pizero_camera if is_pizero_camera
    default_backend octoprint

backend pizero_camera
    mode http
    timeout     connect 5s
    timeout     server  5s
    http-request set-path %[path,regsub(^/pizero_camera/,/)]
    server mjpgstreamer 192.168.7.2:80 maxconn 32

backend octoprint
    mode http
    timeout     connect 5s
    timeout     server  5s
    server mjpgstreamer 127.0.0.1:5000 maxconn 32
```

Now OctoPrint can be accessed on port 80, and the camera can be accessed on the
same address and port, but at the `/pizero_camera` path. In OctoPrint settings
Webcam and Timelapse, configured:

* Check "Enable webcam support"
  * Stream URL: http://172.16.0.87/pizero_camera/?action=stream
* Check "Enable timelapse support"
  * Snapshot URL: http://172.16.0.87/pizero_camera/?action=snapshot
  * Path to FFMPEG: /usr/bin/ffmpeg

Install [FFmpeg](https://ffmpeg.org) for video encoding (note, when installing
FFmpeg on a Pi, for some reason a lot of guides explain how to compile it yourself,
but you can just use [FFmpeg Static Builds](https://johnvansickle.com/ffmpeg/),
armhf version on the Pi 3... I used this technique later, for setting up
[homebridge-camera-ffmpeg](https://github.com/KhaosT/homebridge-camera-ffmpeg/)
to allow the camera to be streamed over Homebridge. But in this case, I just
installed FFmpeg using the Arch Linux repository, via `pacman`).

The great advantage of OctoPrint is the vast quantity of plugins available.

I installed [OctoPrint-MalyanConnectionFix](https://github.com/OctoPrint/OctoPrint-MalyanConnectionFix)
to fix the connection error with the MPSM.

### Raspberry Pi Zero: CSI camera, USB Ethernet gadget

To view the print, we'll need a camera. I decided to repurpose the Raspberry Pi Zero
I had previously used for these projects:

* *[Building a small custom Raspberry Pi Zero laptop in a cardboard box](https://satoshinm.github.io/blog/171112_building_a_small_custom_raspberry_pi_zero_laptop_in_a_cardboard_box.html)*
* *[Custom laptop upgrades: internal breadboard, power indicator, and larger case](https://satoshinm.github.io/blog/171202_ztop2_custom_laptop_upgrades_internal_breadboard_power_indicator_and_larger_case.html)*
* *[Custom laptop power management: external activity, power on button, graceful power off, hard power switch](https://satoshinm.github.io/blog/171206_ztop2a_custom_laptop_power_management_external_activity_power_on_button_graceful_power_off_hard_power_switch.html)*
* *[Custom laptop charging: adding a USB charging port and module](https://satoshinm.github.io/blog/171216_ztop2c_custom_laptop_charging_adding_a_usb_charging_port_and_module.html)*

The Pi Zero is connected to a compatible camera (via the miniature [Camera Serial Interface](https://en.wikipedia.org/wiki/Camera_Serial_Interface) port):

* [SainSmart Brand New Camera Module 5MP Mini Size for Raspberry Pi Zero](https://www.amazon.com/gp/product/B01LH10HYE)

However, the Pi Zero is not necessarily recommended to run OctoPrint.
At least, the [official site](https://octoprint.org/download/) says
the "Raspberry Pi Zero *W* is not recommended since severe performance issues were observed",
and the recommended hardware is a Raspberry Pi 3B.

Hence, I'll only be using the Pi Zero for the camera. To do this, it will be configured as
a USB Ethernet gadget, following these tutorials:

* [Adafruit: Turning your Raspberry Pi Zero Into a USB Gadget: Ethernet Gadget](https://learn.adafruit.com/turning-your-raspberry-pi-zero-into-a-usb-gadget/ethernet-gadget)
* [Circuit Basics: Raspberry Pi Zero USB/Ethernet Gadget Tutorial](http://www.circuitbasics.com/raspberry-pi-zero-ethernet-gadget/)

After editing `/boot/config.txt` to enable `dwc2,g_ether`, the device can be plugged in
to a PC (from the "USB" port, not PWR) and the PC will both supply power and treat it as
an Ethernet device. I statically configured the Pi's address to `192.168.7.2`, and my
host PC's address to `192.168.7.1`.

Next I setup [mjpg-streamer](https://github.com/jacksonliam/mjpg-streamer) to stream
from `input_raspicam` to `output_http`. To allow `apt` to access the Internet, edited
`/etc/apt/apt.conf` and added:

```
Acquire::http::Proxy "http://USERNAME:PASSWORD@192.168.7.1:9011";
Acquire::https::Proxy "https://USERNAME:PASSWORD@192.168.7.1:9011";
```

then ran [inaz2/proxy2](https://github.com/inaz2/proxy2) on the host machine, modified
to listen on all interfaces:

```diff
--- a/proxy2.py
+++ b/proxy2.py
@@ -373,6 +373,7 @@ def test(HandlerClass=ProxyRequestHandler, ServerClass=ThreadingHTTPServer, prot
     else:
         port = 8080
     server_address = ('::1', port)
+    server_address = ('', 9011)

     HandlerClass.protocol_version = protocol
     httpd = ServerClass(server_address, HandlerClass)
```

Once mjpg-streamer is working properly, 
[http://192.168.7.2/?action=stream](http://192.168.7.2/?action=stream) can be accessed
from the host PC to view a live stream from the Pi Zero camera.

Not pretty (to say the least), but it works:

![Pi Zero camera facing printer](https://user-images.githubusercontent.com/26856618/71550226-f213ce80-29c2-11ea-87d9-0bff1b404ab9.png)

To view the Pi's display, I connected the mini HDMI port to a VGA monitor using
[Mini HDMI to VGA Adapter, Benfei Gold-Plated Mini HDMI to VGA Adapter (Male to Female)](https://www.amazon.com/gp/product/B07D5WGCG1).
To access the console, I enabled the serial console by adding `enable_uart=1` to `/boot/config.txt`,
and wired TXD, RXD through a [SparkFun FTDI Basic Breakout - 3.3V](https://www.sparkfun.com/products/9873)
serial to mini-USB adapter.

Not too happy with the complexity of the setup, but I'll improve it in the next sections, stay tuned.

### Power control with ESP8266

The other feature I wanted, besides remote monitoring, is remote power control. Forcefully
turn the printer power off, if it is needed for emergency purposes or any other reason.

Similar to the design here:

* *[The smart bulb / smart switch dilemma: smartening up a dumb wall switch](https://satoshinm.github.io/blog/171209_wallswitch_the_smart_bulb_smart_switch_dilemma_smartening_up_a_dumb_wall_switch.html)*

I went with an Espressif ESP8266 connected to a GPIO to drive a relay,
controllable over WiFi. Now I can access the web interface and turn the printer's power
off or on.

### Taking stock and eyeing improvements

To make sense of this setup, I drew this diagram on [draw.io](https://www.draw.io):

![Draw.io diagram](https://user-images.githubusercontent.com/26856618/71550468-d14e7780-29c8-11ea-8e53-5d781662cf3f.png)

If it looks overly complex, that is because it is. If the diagram
looks messy, it is even worse in person, which I won't show. While it is
functional, and I learned a lot building this complex configuration, there is a
lot to be improved. Without further ado, let's get started.

## Improvements

### Power control with DTR signal on RS232 port

In the first iteration, I used a separate microcontroller to provide GPIO to
control the relay to switch the printer's AC power brick. Not too elegant
having yet another system, merely for one purpose. Why not merge it with the
Atom N270 system? Doesn't it have any GPIO signals I can tap for controlling a relay?

It turns out: yes, sort of. Taking a look at the motherboard backplane:

![Atom N270 motherboard backplane](https://user-images.githubusercontent.com/26856618/71550530-b3821200-29ca-11ea-85c0-5fad11aae526.png)

USB, PS/2, VGA, DVI, Ethernet, audio, then (getting warmer) parallel, and two RS232 serial ports!
Yes, this board actually has serial. Parallel could also be used. But it so happens RS232
has a few signals that can be used as GPIO, as this post explains:

* [Stack Overflow: Possible to use a 9 Pin Serial port as “GPIO” using ioctl()?](https://stackoverflow.com/questions/27789099/possible-to-use-a-9-pin-serial-port-as-gpio-using-ioctl)

Here's the connector pinout (from [Lammert Bies: RS232 serial cables pinout](https://www.lammertbies.nl/comm/cable/RS-232)):

![RS232 "DB9" pinout](https://user-images.githubusercontent.com/26856618/71550556-4458ed80-29cb-11ea-8de4-4f71b5b2e617.png)

I decided to use the [Data Terminal Ready (DTR)](https://en.wikipedia.org/wiki/Data_Terminal_Ready) pin.
[Linux Journal: The tty Layer, Part II](https://www.linuxjournal.com/article/6226) explains the
`ioctl()`'s available to control these signals, basically either `TIOCMSET` to change all the
values, or `TIOCMBIS` to set (to high) and `TIOCMBIC` to clear (to low).

[tty_ioctl(4)](https://linux.die.net/man/4/tty_ioctl) also documents this API quite well:

```
   Modem control
       TIOCMGET  int *argp
              Get the status of modem bits.

       TIOCMSET  const int *argp
              Set the status of modem bits.

       TIOCMBIC  const int *argp
              Clear the indicated modem bits.

       TIOCMBIS  const int *argp
              Set the indicated modem bits.

       The following bits are used by the above ioctls:

       TIOCM_LE        DSR (data set ready/line enable)
       TIOCM_DTR       DTR (data terminal ready)
       TIOCM_RTS       RTS (request to send)
       TIOCM_ST        Secondary TXD (transmit)
       TIOCM_SR        Secondary RXD (receive)
       TIOCM_CTS       CTS (clear to send)
       TIOCM_CAR       DCD (data carrier detect)
       TIOCM_CD         see TIOCM_CAR
       TIOCM_RNG       RNG (ring)
       TIOCM_RI         see TIOCM_RNG
       TIOCM_DSR       DSR (data set ready)
```

A simple C program can now be written to set DTR. However, when I measured
the voltage on the DTR pin with a multimeter, it would start out low
(RS232 signaling levels are ±12V):

![RS232 low signal](https://user-images.githubusercontent.com/26856618/71550610-baaa1f80-29cc-11ea-9db1-aed111c845a4.png)

and briefly spike high when executing TIOCMBIS:

![RS232 high signal](https://user-images.githubusercontent.com/26856618/71550600-86cefa00-29cc-11ea-9b34-5787678934cc.png)

but would fall back to low after the program exits (at least, on my system). 
To cope with this behavior, I wrote `set_dtr.c` to hold DTR while it is running;
it pauses indefinitely, then it can be killed to return DTR to low:

```c
// set_dtr.c: hold DTR (pin 4) of RS-232 port high
#include <stdio.h>
#include <stdlib.h>
#include <sys/ioctl.h>
#include <fcntl.h>
#include <unistd.h>

int main(int argc, char **argv) {
	char *port_path = "/dev/ttyS0";
	int fd = open(port_path, O_RDWR | O_NOCTTY);
	if (fd < 0) {
		perror("open");
		exit(EXIT_FAILURE);
	}

	int status = TIOCM_DTR;
	int request = TIOCMBIS; // set

	if (ioctl(fd, request, &status) < 0) {
		perror("ioctl");
		exit(EXIT_FAILURE);
	}

	printf("DTR set, press ^C to quit\n");
	pause();
	printf("exiting\n");

	return 0;
}
```

Compile with `clang set_dtr.c -o set_dtr`.

To more conveniently allow starting/stopping the printer, wrote a systemd service, saved in `/etc/systemd/system/power3dprinter.service`:

```
[Unit]
Description=Power on the MPSM 3D Printer through DTR signal of RS232 serial port

[Service]
ExecStart=/home/admin/rs232/set_dtr

[Install]
WantedBy=multi-user.target
```

Now it is possible to raise the DTR signal high using `sudo systemctl start power3dprinter.service`,
and return it to low with `sudo systemctl stop power3dprinter.service`. 

As for the circuit, an NPN BJT transistor driver will do. Whipped this up in [CircuitLab](https://www.circuitlab.com):

![Circuit](https://user-images.githubusercontent.com/26856618/71550713-89caea00-29ce-11ea-8901-6d3da8c17905.png)

I used a random transistor I had on hand, the 2SC2688 "designed for use in Color TV chroma
circuits", probably where I salvaged it
([datasheet](http://pdf.datasheetcatalog.com/datasheet/nec/2SC2688.pdf])
(maybe from *[Pioneer SD-P453S Rear-Projection (RPTV) teardown: inside an 80s vintage big screen TV](https://satoshinm.github.io/blog/180107_rptv_pioneer_sd_p453s_rear_projection_rptv_teardown_inside_an_80s_vintage_big_screen_TV.html)*?).
The h<sub>FE</sub>, DC current gain, measured about 50. R1 was calculated to saturate
the transistor, allowing the relay to turn on. D1 is a
[flyback diode](https://en.wikipedia.org/wiki/Flyback_diode) to protect from inductive voltage spikes
as the relay coil deenergizes.

C2688's V<sub>BO0</sub>, maximum emitter to base voltage, is 5.0V. This poses a problem,
since RS232 is ±12V. [Stack Overflow: Protecting NPN transistor from negative base-emitter voltage?](https://electronics.stackexchange.com/questions/100709/protecting-npn-transistor-from-negative-base-emitter-voltage)
suggests a solution: a clamping diode from ground to base. This is D2 in the circuit above.
I actually wired it up without this diode at first, and it worked, but little did I know
I may have been _zenering_ the transistor (as in [zener diode](https://en.wikipedia.org/wiki/Zener_diode)), interesting discussion:
[sci.electronics.design: Zenering a big transistor](https://groups.google.com/forum/m/#!topic/sci.electronics.design/INYYXRnCrWQ):

> I'm using a TIP31C (pnp in to-220 pac) as a temp sensor (diode connected) There's a couple of depletion fets in series as current limiters. (LND150) The b-e junction starts to zener at ~30 V (only two tested so far).   

see also: [Stack Overflow: At which negative Vce voltage will an NPN transistor be damaged?](https://electronics.stackexchange.com/questions/65404/at-which-negative-vce-voltage-will-an-npn-transistor-be-damaged).

To double as a standby power indicator, I used an LED for D2, so when DTR is low, it lights up.
When DTR goes high, it turns off. Will the high +12V voltage be a problem? I'm not sure, but it hasn't yet.

The final problem: how to power this thing? The relay requires 12V. Would be nice if could power the relay
off of the RS232's 12V signals, but serial ports won't provide enough current (~60 mA for the relay coil), not much more than necessary
to say power a serial mouse. Considered powering over USB, but USB (without [USB-PD](https://en.wikipedia.org/wiki/USB#PD)),
only provides 5V; could use the buck converter from
*[XL6019 boost/buck DC-DC converter unboxing](https://satoshinm.github.io/blog/180224_xl6019dcdc_xl6019_boost_buck_dc_dc_converter_unboxing.html)*, but would be too bulky. Instead, I salvaged a [Molex connector](https://en.wikipedia.org/wiki/Molex_connector)
from an old hard drive enclosure:

![Molex connector](https://user-images.githubusercontent.com/26856618/71550858-de239900-29d1-11ea-93ce-b697d48c6a7f.png)

The disk drive Molex connectors provide +12V on the yellow wire, perfect. I ran the 12V wire out through
the [Kensington Security Slot](https://en.wikipedia.org/wiki/Kensington_Security_Slot),
and covered it in heat shrink tubing 
([Aliexpress: 580PCS 8 Sizes 5 Multi Color Polyolefin 2:1 Heat Shrink Tubing Tube SleeveTube Assortment Sleeve Wrap Wire Kit tubes Kits](https://www.aliexpress.com/item/32832137085.html), shrank with *[Youyue 858D hot air gun first look](https://satoshinm.github.io/blog/180114_hotair_youyue_858d_hot_air_gun_first_look.html)*):

![Serial + 12V through K-Slot](https://user-images.githubusercontent.com/26856618/71550885-58ecb400-29d2-11ea-83d0-c557c3de6058.png)

The 12V wire joins the serial connector cable. Only GND and DTR are used currently, but
since I reused a [CAT 5 cable](https://en.wikipedia.org/wiki/Category_5_cable) with 6 wires, I hooked
up a few other signals for future expansion. Coloring code for my future reference:

| Wire color | Function | Use |
| ---------- | ------- | ---- |
| black | ground | ground |
| yellow | 12V | power the 12V relay |
| blue | DTR | "GPIO" to control the relay |
| red | TX | |
| green | RX | |
| white | RTS | |

Notably, [RTS (Request to Send)](https://en.wikipedia.org/wiki/RTS/CTS) is another RS232
signal easily usable as GPIO, through `TIOCM_RTS` in Linux tty_ioctl, and TX/RX are for
transmit/receive which could be used if serial communication is to be implemented later,
for some reason. Nonetheless for now, only three wires are used to control the AC-switching
relay from the PC: ground, 12V, and DTR.

### OctoPrint plugin

The printer can now be turned on and off through the `set_dtr` program or `systemd`,
but it would be nice to control it from the OctoPrint web interface.

I searched for [power](https://plugins.octoprint.org/search/?q=power) plugins, and found:

| Plugin | Python compatibility |
| ------ | -------------------- |
| [OctoPrint-PSUControl](https://plugins.octoprint.org/plugins/psucontrol/) | >=2.7,<3 |
| [OctoPrint-OrviboS20](https://plugins.octoprint.org/plugins/orvibos20/) | >=2.7,<3 |
| [OctoPrint-Tasmota](https://plugins.octoprint.org/plugins/tasmota/) | >=2.7,<4 |
| [OctoPrint-TPLinkSmartplug](https://plugins.octoprint.org/plugins/tplinksmartplug/) | >=2.7,<4 |
| [OctoPrint-TuyaSmartplug](https://plugins.octoprint.org/plugins/tuyasmartplug/) | >=2.7,<3 |
| [OctoPrint-WemoSwitch](https://plugins.octoprint.org/plugins/wemoswitch/) | >=2.7,<4 |

Most of these plugins are for specific IoT hardware. Is there anything to let me
control power through shell commands, `sudo systemctl start power3dprinter.service`
and `sudo systemctl stop power3dprinter.service`?

[OctoPrint-PSUControl](https://github.com/kantlivelong/OctoPrint-PSUControl/wiki/Settings) looks promising:

> **Switching Method**
> PSUControl can switch on your power supply using either:
> ...
> **System Command**
> Executes a system command on the OctoPrint server.
> **Note**: _Commands on *nix systems will be executed within the sh interpreter._

however, the Python compatibility string is ">=2.7,<3", meaning it is not currently
compatible with Python 3. And I installed Python 3.8.0. Why use anything older?
[PEP 373 -- Python 2.7 Release Schedule](https://www.python.org/dev/peps/pep-0373/)
explains when Python 2 will be EOL'd:

> The End Of Life date (EOL, sunset date) for Python 2.7 has been moved five years into the future, to 2020.

The future is now—the clock is ticking, literally next week Python 2 will be sunset, at the time of
this writing [3 days from now according to Pythonclock.org](https://pythonclock.org).
Not that it will suddenly stop working, but I wouldn't want to rely on unsupported software.

Fortunately, the [OctoPrint Plugin Tutorial](http://docs.octoprint.org/en/master/plugins/gettingstarted.html)
explains how to get up and running with a custom plugin in no time. I followed the steps
to create `OctoPrint-Helloworld`, and replaced the `octoprint_helloworld/__init__.py` with:

```python
# coding=utf-8
from __future__ import absolute_import

import octoprint.plugin
import subprocess
import json

class HelloWorldPlugin(octoprint.plugin.StartupPlugin,
                       octoprint.plugin.TemplatePlugin,
                       octoprint.plugin.AssetPlugin,
                       octoprint.plugin.SimpleApiPlugin):
    ##~~ AssetPlugin mixin
    def get_assets(self):
            return dict(
                    js=["js/helloworld.js"],
                    css=["css/helloworld.css"],
            )

    ##~~ SimpleApiPlugin mixin
    def get_api_commands(self):
        return dict(turnOn=[], turnOff=[])

    def on_api_command(self, command, data):
        try:
            if command == 'turnOn':
                rc = subprocess.check_output("/home/admin/on_3dprinter.sh")
                response = {"status": str(rc)}
            elif command == 'turnOff':
                rc = subprocess.check_output("/home/admin/off_3dprinter.sh")
                response = {"status": str(rc)}
            else:
                response = {"error": "bad command"}
        except Exception as e:
            response = {"error": str(e)}
        return json.dumps(response)

__plugin_name__ = "Hello World"
__plugin_implementation__ = HelloWorldPlugin()
```

The JavaScript, in `octoprint_helloworld/static/js/helloworld.js`, only has to implement methods to call turnOn and turnOff:

```js
$(function() {
    function HelloworldViewModel(parameters) {
        var self = this;

	    // https://github.com/jneilliii/OctoPrint-TPLinkSmartplug/blob/master/octoprint_helloworld/static/js/helloworld.js
		self.sendTurnOn = function(data) {
			$.ajax({
				url: API_BASEURL + "plugin/helloworld",
				type: "POST",
				dataType: "json",
				data: JSON.stringify({
					command: "turnOn",
					data: {},
				}),
				contentType: "application/json; charset=UTF-8"
			}).done(function(data){
					console.log('Turn on command completed.');
					console.log(data);
				});
		};

		self.sendTurnOff = function() {
			$.ajax({
			url: API_BASEURL + "plugin/helloworld",
			type: "POST",
			dataType: "json",
			data: JSON.stringify({
				command: "turnOff",
				data: {},
			}),
			contentType: "application/json; charset=UTF-8"
			}).done(function(data){
					console.log('Turn off command completed.');
					console.log(data);
				});
		};

	    window.sendTurnOn = self.sendTurnOn;
	    window.sendTurnOff = self.sendTurnOff;
    }

    OCTOPRINT_VIEWMODELS.push({
        construct: HelloworldViewModel,
        dependencies: [],
        elements: [],
    });
});
```

Finally, `octoprint_helloworld/templates/helloworld_navbar.jinja2` provides
the HTML template to insert into the OctoPrint navigation bar:

```html
<nobr>
<a href="#" onclick="sendTurnOn(); return false">🔌</a> |
<a href="#" onclick="sendTurnOff(); return false">❌</a>
</nobr>
```

Would have liked to use the [Unicode power symbols](https://unicodepowersymbol.com)
new in Unicode 9.0, but they are not yet supported in my operating system; if
they are in yours these symbols should not appear as question marks or empty boxes:

```
Power	&#x23FB;	⏻
Toggle Power	&#x23FC;	⏼
Power On	&#x23FD;	⏽
Power Off	&#x2B58;	⭘
Sleep Mode	&#x23FE;	⏾
```

Anyways, with this change the new buttons show up in the UI:

![OctoPrint power control buttons](https://user-images.githubusercontent.com/26856618/71551052-7cfdc480-29d5-11ea-9d08-f4dd6271f125.png)

They may not be properly aligned, and placed dangerously close to the settings button,
but they function properly. Now I can
click 🔌 to turn on the printer, and ❌ to turn it off, easy as that.

It would still be a good idea to use PSUControl once it updates, as this
quick-and-dirty plugin doesn't implement any additional features such as
auto-connecting after turning on, turning off after a print, warning before
turning off during a print, and so on. 

#### systemd user woes

A final note before we move on, here is the contents of `/home/admin/on_3dprinter.sh`:

```sh
sudo /usr/bin/systemctl start power3dprinter.service
```

and `/home/admin/off_3dprinter.sh`:

```sh
sudo /usr/bin/systemctl stop power3dprinter.service
```

Recall how the power3dprinter.service is installed under `/etc/systemd/system`. Would have
liked to install it as a user service under `/etc/systemd/user`, and start with
`systemctl --user`, so OctoPrint wouldn't have to escalate from the admin user to root.
However, when executed from OctoPrint the user command failed with
an error about `DBUS_SESSION_BUS_ADDRESS`, see similar problems reported at:

* [Stack Overflow: How to export DBUS_SESSION_BUS_ADDRESS](https://stackoverflow.com/questions/41242460/how-to-export-dbus-session-bus-address)
* [Unix.com: DBUS_SESSION_BUS_ADDRESS for script called from crontab.](https://www.unix.com/shell-programming-and-scripting/203631-dbus_session_bus_address-script-called-crontab.html)

First attempt to solve this was to set the value to what I observed over SSH:

```sh
export DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/1000/bus"
```

This worked... until I logged out of SSH, and interrupted my print job! Whoops.

An approved solution is to use [dbus-launch](https://linux.die.net/man/1/dbus-launch), like so:

```sh
export $(dbus-launch)
```

This sort of works, in that the service is started, but then is immediately stopped once the shell returns.
I'm not sure why, or the best way to fix it. In any case, running as a system service
instead works around the issue, and I can now remotely control the power of my printer without interruption.

### USB webcam

In the first iteration, I used a Raspberry Pi Zero with a camera. 
Printed a case: [One Piece Raspberry Pi Zero + Camera Case (with GPIO)](https://www.thingiverse.com/thing:1595429),
but the camera viewport was too high; cut out a hole lower. Even with the case,
using the Pi Zero only for a camera felt like a waste.

Therefore, I switched to a standard USB webcam, with a better quality 1080p resolution. Like most webcams it is
supported under by the USB Video Class (UVC) driver with `mjpg_streamer`, making it easy to adapt
into OctoPrint. All I would have to do to make it into a workable solution is build a camera mount.

### Camera mount

There are many mounts for MPSM on Thingiverse, I liked this one:

* [MPSM_C270_Mount](https://www.thingiverse.com/thing:3695964) by TheKrush

![TheKrush MPSM_C270_Mount](https://user-images.githubusercontent.com/26856618/71551324-1c728580-29dd-11ea-94bf-ba5070cf080b.jpg)

It attaches to the front of the bed, so it moves as the bed moves, giving a consistent
view of the print for timelapsing. This mount _almost_ works with my tripod camera. I printed:

| Model | Est filament (m) | Est filament (g) | Actual time | Estimated cost |
| ----- | ---------------- | ---------------- | ------------------- | ------------- |
| MPSM_C270_Mount | 5.06 | 16 | 3h 13min | 25¢ |
| Tripod_Arm | 1.39 | 4 | 49min | 6¢ |
| Tripod_Top | 1.69 | 5 | 40min | 8¢ |

The bed mount and arm fit, but there is a problem with the "Tripod_Top".
Even though it is called "tripod", it doesn't have a standard
[Tripod screw thread](https://en.wikipedia.org/wiki/Tripod_(photography)#Screw_thread) of
1/4-20 UNC. The creator says "I reused the old plastic bolt/screw that came with the C270 to attach the hinge",
but I only have a standard tripod mount screw hole.

Found this tripod screw model:

* [Tripod Screw](https://www.thingiverse.com/thing:3160367)

Printed with 0.22m of filament, ~0g, in 13 minutes for an estimated cost of 0.4¢:

![Tripod screw](https://user-images.githubusercontent.com/26856618/71551145-d535c600-29d7-11ea-994e-99a44bf9d680.png)

Carefully able to thread the tripod screw into the camera base:

![Tripod screwed](https://user-images.githubusercontent.com/26856618/71551153-0b734580-29d8-11ea-82c6-5744c4ee168f.png)

This should work. It screwed into up where I marked the thread with a sharpie,
and this is how the parts would be assembled:

![Tripod parts disassembled](https://user-images.githubusercontent.com/26856618/71551158-3d84a780-29d8-11ea-8890-7a2767181084.png)

All we'll have to do is combine the 1/4-20" UNC tripod screw model with the `Tripod_Top` model.
Imported both STLs into [Tinkercad](https://www.tinkercad.com), and positioned the screw over the top
hole:

![Merged models](https://user-images.githubusercontent.com/26856618/71551168-89cfe780-29d8-11ea-915c-5238f3d5b79e.png)

then exported to STL. The 1/4-20" UNC tripod screw sticks out of the top where the Tripod_Top hole was.

The first print attempt failed at the screw:

![Failure screw](https://user-images.githubusercontent.com/26856618/71551174-a704b600-29d8-11ea-8572-8296f9fcaab0.png)

even with supports. Reprinted with a large raft:

![Tripod top with UNC screw with raft](https://user-images.githubusercontent.com/26856618/71551176-cef41980-29d8-11ea-99a8-66cfd14e424b.png)

This time it succeded (1hr 32min, 3.02m, 8g, 16¢). Carefully screwed the camera onto tripod mount:

![Screwed](https://user-images.githubusercontent.com/26856618/71551180-e7fcca80-29d8-11ea-9ae0-fdba2efe8f32.png)

The final task was to assemble the mount. The designer used metric fasteners, which I didn't have, so I ordered:

* [DYWISHKEY 1220 Pieces M2 M3 M4 M5, 10.9 Grade Alloy Steel Hex Flat Head Cap Bolts Nuts Washers Assortment Kit with Hex Wrenches](https://www.amazon.com/gp/product/B07WZL3Z7H)

A few days later the box arrived:

![Metric fasteners](https://user-images.githubusercontent.com/26856618/71551198-4aee6180-29d9-11ea-99e0-0715ab491202.png)

I used:

* 4x M3-12 (didn't come with 10mm) + 4x M3 nuts to attach base, self-taps into the plastic, tighten with allen wrench
* 2x M3-20 + 2x M3 nuts to attach arms

first, tapping the screw through the plastic:

![Screwing](https://user-images.githubusercontent.com/26856618/71551206-72ddc500-29d9-11ea-8b6d-f4636a0fe34f.png)

then attaching to the bed:

![Bedding](https://user-images.githubusercontent.com/26856618/71551208-8be67600-29d9-11ea-98ac-92c1a7b3e0bf.png).

The hex nut fits snugly in the holes in the arm:

![Nut](https://user-images.githubusercontent.com/26856618/71551214-b0425280-29d9-11ea-950e-5488c90641d2.png)

Although I had to warm up one of the holes with water to shape it into place. The mount is attached:

![Attached mount](https://user-images.githubusercontent.com/26856618/71551228-fa2b3880-29d9-11ea-880d-9c9283eedb43.png)

With everything said and done, the OctoPrint GUI now shows the complete view of the bed through the webcam:

![OctoPrint with webcam mounted](https://user-images.githubusercontent.com/26856618/71551049-432cbe00-29d5-11ea-850c-21a0b62050f9.png)

---

**[Comments?](https://old.reddit.com/r/octoprint/comments/egzv02/detailed_writeup_of_my_octoprint_setup_for_a/?st=k4qdabf6&sh=bb91e381)**

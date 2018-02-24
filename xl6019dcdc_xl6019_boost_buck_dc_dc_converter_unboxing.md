# XL6019 boost/buck DC-DC converter unboxing

by snm, February 24th, 2018

Ordered [XL6019 (XL6009 upgrade) Automatic step-up step-down Dc-Dc Adjustable Converter Power Supply Module 20W 5-32V to 1.3-35V](https://www.aliexpress.com/item/Boost-Buck-DC-DC-Adjustable-Step-Up-Down-Converter-XL6009-Power-Supply-Module-20W-5-32V/32788804655.html) for $1.32/each. Arrived in 19 days. It is a [DC-to-DC converter](https://en.wikipedia.org/wiki/DC-to-DC_converter) which can both increase (boost) and decrease (buck) voltage, efficiently, as compared to a [linear regulator](https://en.wikipedia.org/wiki/Linear_regulator) which dissipates the excess voltage as heat. Bought three expecting these will prove useful for various projects:

![Three packages](https://user-images.githubusercontent.com/26856618/36633836-b914d2f2-1950-11e8-8c3e-b6eb6dbc0536.png)

Tear open the bag, the top of the circuit board is simple enough, with IN+/IN- voltage inputs, OUT+/OUT- outputs, and a potentiometer for adjusting the voltage:

![Front board](https://user-images.githubusercontent.com/26856618/36633852-eb525690-1950-11e8-9e0a-d18693307335.png)

The reverse side of the board has no components, but is silkscreened "TENSTAR ROBOT", the makers of this device. There are two screw mounting holes, curiously some perforation near the middle (for heat dissipation? prototype?) and some solder flux on the potentiometer through-hole leads:

![Back board](https://user-images.githubusercontent.com/26856618/36633860-091b800c-1951-11e8-8359-f8ccc47c8813.png)

How well does it work? Tested hooking it up to a fresh 9V battery on the input, and a multimeter on the output:

![9V battery on input](https://user-images.githubusercontent.com/26856618/36633888-7f7fc348-1951-11e8-8c62-b3d289453375.png)

The battery input read 8.99V when connected. Out of the box, the output from the boost/buck converter with this input was 22.00V. Turning the pot clockwise, cranking up full blast:

![Screwdriver on pot](https://user-images.githubusercontent.com/26856618/36633921-d6e5ec3e-1951-11e8-86ce-2d34e18179bb.png)

was able to reach a maximum of 39.61V. Any higher, the knob kept turning, but no voltage increase. Decreasing the voltage, bottomed out at 1.253V. The advertised output voltage range is 1.3-35V, so 1.253-39.61V isn't bad at all. And the multi-turn potentiometers allows fairly precise adjustment of the desired value. Without much difficulty, I was able to set it to about 5V and it barely bounced between 5.001-5.002V (no load).

So far so good. The XL6019 boost/buck DC-DC converter works as expected, at least in initial tests. The next step would be to use it for some useful purpose, perhaps as an improvement to the setup described in *[3D Printing with 3D MARS white PLA: pass-through wallplate, paper towel holder, right-angle spool holder, shower curtain rings, and a prototype case: Remote monitoring using Pi Zero camera](https://satoshinm.github.io/blog/180208_3dprint2_3d_printing_with_3d_mars_white_pla_pass_through_wallplate_paper_towel_holder_right_angle_spool_holder_shower_curtain_rings_and_a_prototype_case.html#remote-monitoring-using-pi-zero-camera)*, stepping down the 3D printer's 12 volts from the wall adapter to 5 volts to power the Raspberry Pi Zero without requiring a second AC wall adapter unit, if the first one can handle the additional power draw. However, most any scenario where you have one voltage but need another (especially if the voltage difference is large, within the limitations), these boost/buck converters would be useful.
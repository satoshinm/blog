# Powerful Function LTCMD720NIR CCD Camera teardown

by snm, January 28nd, 2018

* auto-gen TOC:
{:toc}

## Unboxing and manual

Came in a box labeled "Powerful Function - CCD Camera":

![Box](https://user-images.githubusercontent.com/26856618/35487468-c2dc6938-0430-11e8-97b6-cc2477efdec9.png)

The short manual says it is model LTCMD702NIR, featuring a 1/3" Sony Super Had CCD, with 480 TVL, 0 LUX, 3.6mm lens, and powered with 12VDC, emitting an [NTSC](https://en.wikipedia.org/wiki/NTSC) video signal:

![Manual 1](https://user-images.githubusercontent.com/26856618/35487495-fb3d1192-0430-11e8-8325-0b7a59a277da.png)

A closer look at the end of the manual since I couldn't fit it all in my camera:

![Manual 2](https://user-images.githubusercontent.com/26856618/35487527-4f3df644-0431-11e8-9b2f-9a9fbda5678b.png)

## Under the cover

Unscrewing the protective cover, we see the camera assembly:

![The camera inside](https://user-images.githubusercontent.com/26856618/35487538-799d85a8-0431-11e8-856f-e600a5a442ae.png)

It is affixed to the body with two screws. Remove them and the camera comes right off:

![Removed camera from enclosure](https://user-images.githubusercontent.com/26856618/35487554-97dd6128-0431-11e8-8774-201142c0918c.png)

The power and video cable can also be unplugged from the socket. Next step, take apart the mounting brackets:

![Brackets removed](https://user-images.githubusercontent.com/26856618/35487568-bb802caa-0431-11e8-800c-19487c8417e6.png)

And finally, the adjustment arms:

![Arms removed](https://user-images.githubusercontent.com/26856618/35487574-d2779e5c-0431-11e8-8eef-eed6bd4e3606.png)

Plenty of screws and washers for the taking. But the triple stacked PCB design is pretty interesting. Let's take a closer look.

## Unstacking the PCBs

The camera viewed head-on:

![Head-on camera](https://user-images.githubusercontent.com/26856618/35487597-16ae26a4-0432-11e8-8e9f-5ed884c7eb2a.png)

It is screwed to the top board, the power/video plugs into the bottom board, and there is a third board in the middle. Presumably the top is for interfacing to the camera, and the bottom is for the power supply. Easily taken apart by removing the corner screws and standoffs:

![Triple PCBs apart](https://user-images.githubusercontent.com/26856618/35487615-58fcbffc-0432-11e8-9d02-4b0242b584fe.png)

### Power board

This is the bottom-most board the external cable plugs into. A large capacitor, and small transformer, definitely related to power. The silkscreen label reads "DUAL BOARD 04/07/06". A few large SMD diodes, capacitors, and a fuse. DIP switches per the manual. The transistor in the middle is a [Fairchild IRFR224B](http://www.alldatasheet.com/datasheet-pdf/pdf/53017/FAIRCHILD/IRFR224B.html), a 250V N-channel MOSFET:

![Top power](https://user-images.githubusercontent.com/26856618/35487625-854e1c36-0432-11e8-8b2d-3c41a4960cdf.png)

The other side of the board has some diodes, passives, and connectors for stacking to the board below:

![Bottom power](https://user-images.githubusercontent.com/26856618/35487662-1d078f80-0433-11e8-9890-977cfd89544d.png)

### Signal processing board

The middle board (silkscreen labeled "ONIX") is more intricate, it contains a chip labeled Sony CXD2163BR 519L57V:

![Top DSP](https://user-images.githubusercontent.com/26856618/35487687-5a671792-0433-11e8-87fc-01c3505927d3.png)

and the other side, whose silkscreen "DSP B/D" gives away its function, [digital signal processing](https://en.wikipedia.org/wiki/Digital_signal_processing):

![Bottom DSP](https://user-images.githubusercontent.com/26856618/35487703-91306e18-0433-11e8-9df1-40c022939832.png)

Found the [Sony CXD2163BR data sheet](https://www.promelec.ru/pdf/CXD2163BR.pdf), which says it is a "Signal Processor LSI for Single-Chip CCD Color Camera", in a 100-pin LQFP, with a built-in microcontroller:

> The CXD2163BR is a signal processor LSI for Ye, Cy, Mg and G single-chip CCD color cameras. In addition to basic camera signal processing functions, it includes an AE/AWB detection circuit, a sync signal generation circuit and an external sync circuit, etc.
> This chip also has a built-in microcontroller to realize basic camera functions without a microcomputer.
> The CXD2163BR is a fully improved version and pin compatible with the CXD2163R by adding some functions and improving its performance.

A sophisticated block diagram:

![Block diagram](https://user-images.githubusercontent.com/26856618/35487748-f31af24c-0433-11e8-9285-dcbd8990db47.png)

and pinout:

![Pinout](https://user-images.githubusercontent.com/26856618/35487753-022ea4e0-0434-11e8-9c2c-87aeb6da6176.png)

very application-specific, probably not much use outside this application, although the built-in microcontroller may be interesting to get up and running on its own. Onto the next and final board.

### CCD board

This is the printed circuit board the camera screws into:

![Camera board bottom](https://user-images.githubusercontent.com/26856618/35487775-3a94f438-0434-11e8-8c85-b13e6c6b22e7.png)

Several chips, related to the CCD camera as we might expect:

* [Sony A2006Q 513 E03V](http://www.alldatasheet.com/datasheet-pdf/pdf/46731/SONY/CXA2006Q.html) - Digital CCD Camera Head Amplifier
* [Sony D2480R 513 B83N](http://www.datasheetlib.com/datasheet/1287785/cxd2480r_sony-corporation.html) - Timing Controller IC with Built-in CCD Driver
* [3414A 519V JRC](http://www.njr.com/semicon/products/NJM3414A.html) - Single-Supply Dual High Current Operational Amplifier

Again quite application-specific, but repurposing the opamp could be interesting. Opamps were also very common in a previous teardown, *[Pioneer SD-P453S Rear-Projection (RPTV) teardown: inside an 80s vintage big screen TV](https://satoshinm.github.io/blog/180107_rptv_pioneer_sd_p453s_rear_projection_rptv_teardown_inside_an_80s_vintage_big_screen_TV.html)*.

Flipping it over, after removing the camera lens, the camera itself:

![Camera board top](https://user-images.githubusercontent.com/26856618/35487826-0812ead2-0435-11e8-818f-dce5d366cd91.png)

The silkscreen says "CCD BOARD REV-02". The chip on the top is 88347 E10506 M38, I couldn't find much information for but it [may be](http://www.alldatasheet.com/view.jsp?Searchword=88347) a digital-to-analog converter.

## Conclusions

Stacked the boards back together:

![Stacking back up](https://user-images.githubusercontent.com/26856618/35487859-7c80fee0-0435-11e8-8202-3be5cc42bc32.png)

The stacking board design allowed the designers of this product to fit all of the needed circuitry into a few square inches beneath the camera mount, all inside a convenient enclosure. Not bad. There is however for better or worse not much too salvage from this device, since it is highly integrated. 
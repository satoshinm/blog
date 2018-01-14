# Youyue 858D hot air gun first look

by snm, January 14th, 2018

* auto-gen TOC:
{:toc}

Purchased a [YOUYUE 858D Hot Air Gun Rework Station SMD Solder Soldering Digital 110V 700W](https://www.amazon.com/gp/product/B00P8Z4RPG), for $45.99 minus a $5 Amazon "coupon". Planning to use along side a soldering station for various electronics projects where hot air beats the iron. 

## Unboxing and assembly

Arrived in a box within a box, labeled "SI858D Made in China":

![Box](https://user-images.githubusercontent.com/26856618/34911739-468f78c2-f885-11e7-96da-5d1f31ca93fc.png)

and an upside-down barcode X001CFII1L Youyue 858D Hot Air Gun Rework Soldering Digital 110V 700W Made in China. Some assembly was required: screwing on the handle, plugging in the cables, but that's it.

## Desolding SMD answering machine SOIC

First some practice, test it out desoldering a small surface-mount chip on the board from *[AT&T 1739 Digital Answering Machine teardown and resuscitation attempt](https://satoshinm.github.io/blog/180109_att_ansmachine_att_1739_digital_answering_machine_teardown_and_resuscitation_attempt.html)*. Turn on the air and heat, wave the gun around:

![Hot air on answering machine SMD SOIC](https://user-images.githubusercontent.com/26856618/34911766-b7a90ffa-f885-11e7-9030-d1dac254d7ff.png)

It lifts off, easier than expected, with anti-static tweezers:

![Lifting off the SOIC](https://user-images.githubusercontent.com/26856618/34911774-e29b77f2-f885-11e7-8731-ccc53e74ddec.png)

I couldn't read the label on this IC too well, but it says [Atmel](https://en.wikipedia.org/wiki/Atmel), makers of the [AVR](https://en.wikipedia.org/wiki/Atmel_AVR) family of microcontrollers, now owned by [Microchip](https://en.wikipedia.org/wiki/Microchip_Technology). AVR is/was popular as the microcontroller family for the [Arduino](https://en.wikipedia.org/wiki/Arduino) platform, but now ARM and ESP chips are gaining ground.

While I was blowing air on this micro, also blew off two surface-mount diodes, and then later intentionally desoldered two more:

![Spare diodes](https://user-images.githubusercontent.com/26856618/34911792-4b084df6-f886-11e7-892b-6d95a9ee354d.png)

Now for a real challenge.

## Desoldering through-hole microswitches on the RPTV

*[Pioneer SD-P453S Rear-Projection (RPTV) teardown: inside an 80s vintage big screen TV](https://satoshinm.github.io/blog/180107_rptv_pioneer_sd_p453s_rear_projection_rptv_teardown_inside_an_80s_vintage_big_screen_TV.html)* provided a wealth of parts, but I have a particular interest in the 14 pushbuttons on the frontpanel board since I'm running low:

![Frontpanel board](https://user-images.githubusercontent.com/26856618/34911804-93edcdfc-f886-11e7-9ff6-3c4e86a7b25a.png)

These could be desoldered with a soldering iron, but then I would use solder or flux, or soldering braid, and wear down the soldering tip. Turning to the hot air gun, I desoldered them all:

![Desoldered switches](https://user-images.githubusercontent.com/26856618/34911812-bde3dc14-f886-11e7-903c-04e7211da77e.png)

The only casualty was one of the buttons, which I pulled too hard with pliers and opened it up, but all the other 13 survived. Hot air desoldering feels a lot different than desoldering with a soldering iron, it is much drier, the solder can melt without beginning to flow as a liquid. The main difficulty was positioning the boards to allow an acceptable amount of force in the proper direction to pull out the components, but I managed. Practice makes perfect.

## Next steps

There are many clones of the "858D" hot air rework station. Dave Jones reviewed another variant in [EEVblog #167 - Atten 858D Hot Air Rework Review](https://www.youtube.com/watch?v=vva2t21sOAs). But I went with the Youyue due to [EEVblog forums: Youyue 858D+ some reverse engineering + custom firmware](http://www.eevblog.com/forum/reviews/youyue-858d-some-reverse-engineering-custom-firmware/) and supposedly perhaps better build quality.

Could be interesting to try replacing the firmware, but the stock firmware seems to work reasonably well for my purposes so far.

Besides soldering/desoldering, I also ordered some heat shrink tubing: [580PCS 8 Sizes 5 Multi Color Polyolefin 2:1 Heat Shrink Tubing Tube SleeveTube Assortment Sleeve Wrap Wire Kit tubes Kits](https://www.aliexpress.com/item/580PCS-8-Sizes-5-Multi-Color-Polyolefin-2-1-Heat-Shrink-Tubing-Tube-SleeveTube-Assortment-Sleeve/32832137085.html), awaiting arrival, which could be useful to use with this hot air gun for insulating [spliced](https://en.wikipedia.org/wiki/Western_Union_splice) wires or other components.

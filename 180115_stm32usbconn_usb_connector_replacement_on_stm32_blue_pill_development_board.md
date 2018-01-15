# USB connector replacement on STM32 blue pill development board

by snm, January 15th, 2018

As discussed in *[USB breakout boards for supplying power to your projects](https://satoshinm.github.io/blog/180108_usb_power_usb_breakout_boards_for_supplying_power_to_your_projects.html)*, it is a [known issue](http://wiki.stm32duino.com/index.php?title=Blue_Pill#Known_issues) the micro-USB connector on the STM32 blue pill board is not the best:

* *The micro-USB connector is not soldered to the board very well and is easily broken. Then first weld the connector better and if you want can cover the connector in epoxy glue or hot glue. There are multiple versions of this board with different connectors. Refer to the pictures for examples.*

This happened to mine. The USB connection of this port is very flakey:

![Bad USB](https://user-images.githubusercontent.com/26856618/34962288-f52ca99a-f9f7-11e7-8911-175ecfaa769f.png)

To remove it, used a hot air gun, specifically the one I just received and unboxed in *[Youyue 858D hot air gun first look](https://satoshinm.github.io/blog/180114_hotair_youyue_858d_hot_air_gun_first_look.html)*. It came off easily:

![Removed bad USB](https://user-images.githubusercontent.com/26856618/34962324-0dae2174-f9f8-11e7-92a1-149cc951e811.png)

Now to replace it. I already had ordered these micro USB breakout boards: [5Pcs CJMCU Micro USB Board Power Adapter 5V Breakout Switch Interface Module For Arduino](https://www.aliexpress.com/item/5Pcs-CJMCU-Micro-USB-Board-Power-Adapter-5V-Breakout-Switch-Interface-Module-For-Arduino/32838024725.html). Desoldered a connector on the board using the same hot air technique. The connector on this breakout is slightly different (middle) versus the blue pill's micro USB connector (right):

![USB breakout board and micro USB comparison](https://user-images.githubusercontent.com/26856618/34962432-72848052-f9f8-11e7-8b01-5805921370ea.png)

but I'm hoping it is compatible enough with the printed circuit board connector footprint. Carefully position the new connector on the circuit board traces:

![Adding new connector](https://user-images.githubusercontent.com/26856618/34962486-b56114a8-f9f8-11e7-8ed6-bafa25ff94d9.png)

There are some clear problems. The ground tabs on the side stick in between the two large pads. The ground tabs on the back stick out too far to the bottom. Bent them up and away, and also the side tabs upwards, to help allow the smaller pins to make a better connection to the board. First attempt:

![Fail 1](https://user-images.githubusercontent.com/26856618/34962599-2dd34faa-f9f9-11e7-8117-aa87781f84ea.png0

Plug it into a computer and...nothing. The power LED doesn't light up. Bad connection. Trial and error:

![Closeup](https://user-images.githubusercontent.com/26856618/34962618-51b65e62-f9f9-11e7-8891-ca8a3f310878.png)

It doesn't look pretty (largely because of the angling needed to get a connection), but finally was able to get a working port:

![Good port](https://user-images.githubusercontent.com/26856618/34962646-7732a4e8-f9f9-11e7-86f9-31553e2f4785.png)

While I was at it, soldered the bad port to the USB breakout board, just because:

![Bad port new home](https://user-images.githubusercontent.com/26856618/34962679-9f05a9f2-f9f9-11e7-920e-4568f9e8cebc.png)

So how well does this new port work? Tested by flashing the firmware in *[pill_6502: 8-bit 6502 CPU and 6850 ACIA emulation on the STM32 blue pill to run Microsoft BASIC from 1977](https://satoshinm.github.io/blog/180113_stm32_6502_pill_6502_8_bit_6502_cpu_and_6850_acia_emulation_on_the_stm32_blue_pill_to_run_microsoft_basic_from_1977.html)* then connecting over and powering from USB, and it worked perfectly. However, one still ought to be careful in unplugging from this fragile micro-USB port too frequently, I think I'm going to unplug from the USB-A side on the host computer instead when I can.

Also, if you want to do this replacement yourself, I wouldn't recommend buying this particular USB connector, since it isn't the exact same footprint. Buy the same connector as a replacement if you can find it. But as for me, this is what I had and it worked at least well enough for my purposes. 
# Sanyo LM3364K dynamic RAM (DRAM) pinout decoding

In *[Pioneer SD-P453S Rear-Projection (RPTV) teardown: inside an 80s vintage big screen TV](https://satoshinm.github.io/blog/180107_rptv_pioneer_sd_p453s_rear_projection_rptv_teardown_inside_an_80s_vintage_big_screen_TV.html)* we encountered an audio board ANPI309-B which had a memory chip, the Sanyo LM3364K-15 64KB DRAM. What is this chip and how can it be used?

[Dynamic RAM](https://en.wikipedia.org/wiki/Dynamic_random-access_memory) requires a refresh to maintain the bits. I can tell the LM3364K is a 64KB DRAM from eBay sellers, but was unable to locate any data sheet. So we'll need to ascertain the pinout ourselves.

The LM3364K is connected to a Mitsubishi M50199P, which *does* have a readily available [data sheet](https://www.datasheetarchive.com/pdf_dl/?f=3694162&t=1&g=M50199), albeit only one page. Page 13 of probably a larger book of chips. Here it is in its entirety, the Mitsubishi M50199P Sound Processor Digital Delay:

![Mitsubishi M50199P Sound Processor Digital Delay](https://user-images.githubusercontent.com/26856618/34804895-caba87de-f62f-11e7-9994-3f2d68878c37.png)

The two chips are next each other:

![Digital delay by DRAM](https://user-images.githubusercontent.com/26856618/34805132-40dc56f8-f631-11e7-9730-908013637696.png)

and a look at the traces on the reverse side shows they are connected:

![Digital delay traces to DRAM](https://user-images.githubusercontent.com/26856618/34805151-5f8c27a4-f631-11e7-8b4e-250f81879349.png)

This makes sense, because according to the sheet, "The M50199 converts an input analog signal to a digital signal and writes to a memory IC. After a delay it reads out the digital signal from the memory IC and then converts to an analog signal again." The pinout of the M50199 can therefore be used to help determine the pinout of the LM3364K.

| M50199 | LM3364K |
| ------ | ------- |
| 1 A6 (Address 6 Output) | 12 A6 (Address 6 Input)
| 2 MDI (Memory Data Input) | 11 MDO (Memory Data Output)
| 3 CAS (Column Address Strobe) | 10 CAS (Column Address Strobe)
| 4 RESET (Reset Input) | 
| 5 DO (Data Output) | 2 DI (Data Input)
| 6 W (Write Control Output) | 3 W (Write Control Input)
| 7 RAS (Row Address Strobe Output) | 4 RAS (Row Address Strobe Input)
| 8 A0 (Address 0 Output) | 5 A0
| 9 A2 (Address 2 Output) | 6 A2
| 10 A1 (Address 1 Output) | 7 A1
| 11 A7 (Address 7 Output) | 16 A7
| 12 A5 (Address 5 Output) | 15 A5
| 13 A4 (Address 4 Output) | 14 A4
| 14 A3 (Address 3 Output) | 13 A3
| 15 D-GND (Digital Ground) | 
| 16 CC1 (Current Control 1) |
| 17 CC2 (Current Control 2) |
| 18 SHORT (Short) |

Then the LM3364K DRAM pinout on its own:

| # | LM3364K |
| - | ------- |
| 1 | N/C
| 2 | DI (Data Input)
| 3 | W (Write Control Input)
| 4 | RAS (Row Address Strobe Input)
| 5 | A0
| 6 | A2
| 7 | A1
| 8 | 
| 9 | 
| 10 | CAS (Column Address Strobe)
| 11 | MDO (Memory Data Output)
| 12 | A6
| 13 | A3
| 14 | A4
| 15 | A5
| 16 | A7

Power may be pin 9 and ground pin 8, or pin 1 not connected; could identified with some effort. 5V nominal supply. Time to fire it up? But using DRAM is not trivial. A reply to [EEVblog forums: Trying to understand some old school logic...](http://www.eevblog.com/forum/projects/trying-to-understand-some-old-school-logic/) notes The dynamic ram memory back then could have been the [TMS4116](http://www.rogtronics.net/files/datasheets/mem_dram/TMS4116.pdf), and indeed its pinout looks very close to what I've ascertained above:

![TMS4116 pinout](https://user-images.githubusercontent.com/26856618/34805747-8a79e2a0-f634-11e7-965d-b1dc5e48c6a8.png)

except pin 9 on the LM3364K connects to pin 11 on the M50199 which is A7, the Address 7 Input, though the TMS4116's pin 9 is VCC (+5V). Pin 1 is VBB (-5V) on the TMS4116 but appears disconnected on the LM3364K. Pin 8 is VDD (+12V) on the TMS4116, it probably is also power-related on the LM3364K. Pin 16 connects through a bypass capacitor, it is VSS (ground) on the TMS4116, maybe the same (or power?) on the LM3364K.

To really understand dynamic RAM, the EEVblog forums pointed out [IBM Application Note: Understanding DRAM Operation](https://www.ece.cmu.edu/~ece548/localcpy/dramop.pdf). Specific sequences of CAS and RAS signals are needed. Not a trivial exercise. For now I'm shelving the project to get this DRAM working in a circuit, but at least the pinout is now known.
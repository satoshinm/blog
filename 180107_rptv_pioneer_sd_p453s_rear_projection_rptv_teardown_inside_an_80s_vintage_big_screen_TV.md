# Pioneer SD-P453S Rear-Projection (RPTV) teardown, an 80s vintage big screen TV 

by snm, January 7th, 2018

* auto-gen TOC:
{:toc}

[Rear-projection](https://en.wikipedia.org/wiki/Rear-projection_television) is an obsolete technology used in what was once called "big screen TV". This Pioneer unit was beginning to show its age:

![RPTV showing SMB3](https://user-images.githubusercontent.com/26856618/34652458-3456eb00-f393-11e7-847b-70ccee3014e9.png)

It is a Pioneer SD-P453S Projection Monitor Receiver, according to the label:

![Model label](https://user-images.githubusercontent.com/26856618/34652469-5d393d34-f393-11e7-8963-ddc6e79dedd4.png)

## Background

Found the manual: [Pioneer SD-P503FP Operating Instructions Manual](https://www.manualslib.com/manual/1013729/Pioneer-Sd-P503fp.html) includes SD-P453FP-Q, and starting on page 27, the SD-P453S. The "S" model adds surround sound. Not many technical details in the operating manual, but it includes a block diagram:

![SD-P453S-Q block diagram](https://user-images.githubusercontent.com/26856618/34653617-5c470996-f3a3-11e7-8d89-a36aa3bc2687.png)

Someone is selling the [SD-P453S-Q service manual](http://www.completeservicemanuals.com/electronics/pioneer/crt-projection-television/sd-p453s-q-service-manual.html) for $7.99. And [another](http://www.manualscenter.com/manuals/pioneer/sdp503pqd-service-manual.html) for $4.99, 134 pages Order No. ARP1869 (printed July 1989) for SDP503PQD and related models. I'll pass, but the preview has this useful diagram on the high voltage generation point, documenting the VR1 ACX1025 variable focus resistor, CRT assemblies, and deflection yokes:

![Fig 3-1 Charged section and high voltage generating point (pg 5 of service manual)](https://user-images.githubusercontent.com/26856618/34653665-1b50adec-f3a4-11e7-8de8-cf07673ddba9.png)

Found a $64.25 repair kit: [Pioneer SDP453S & SD-P453S Convergence Repair Kit](https://www.tvrepairkits.com/xcart/pioneer-sdp453s-sd-p453s-convergence-repair-kit.html), but I think this TV has outlived its usefulness. The technology is from the 1980s, or early 90s, time to say goodbye.

## Disassembly

Starting with the back panel:

![Back panel](https://user-images.githubusercontent.com/26856618/34652495-c259df8e-f393-11e7-8938-314c6fc66701.png)

"Pioneer Projection Monitor Receiver Model SD-P453S. AC120Volts 60HZ 255Watts. Pioneer Electronics (USA) INC 2265 East 220th Street Long Beach, California, 90910 USA. Assembled in U.S.A., with U.S.A. and imported  parts." Dolby Laboratories, and U.S. patents:

* [3,632,886 Quadrasonic sound system](https://www.google.com/patents/US3632886), filed December 29th, 1969
* [3,746,792 Multidirectional sound system](https://www.google.com/patents/US3746792), filed June 15th, 1970
* [3,959,590 Stereophonic sound system](https://www.google.com/patents/US3959590), filed July 9th, 1973

SD-P453S-Q, manufactured March 1990. Speaker impedance 8 Ω ~ 16 Ω/speaker. Among the usual warnings are the following: built-in surround processor on/off switch, external speakers left/right terminals, surround balance, audio output variable left/right jacks, control "out" port, two antenna jacks, video/audio input jacks: VDP (and S-Video), VIDEO-1, VIDEO-2, and the same output jacks "REC except video-1". Zooming out, the rear panel was quite small in relation to the TV, having a large metal grill and black plastic structure:

![Back](https://user-images.githubusercontent.com/26856618/34652586-50dc2522-f395-11e7-9841-ac1f308a1ac8.png)

Unscrew the grill. A dusty circuit board is visible, connected to an internal metal mesh:

![Power supply board inside](https://user-images.githubusercontent.com/26856618/34652619-b2b905ee-f395-11e7-9c12-5326512968e5.png)

This is part of the high-voltage power supply. Beware, [according](https://en.wikipedia.org/wiki/Rear-projection_television#Background) to Wikipedia, this HV powering the CRT may be as large as 25,000 volts:

> Utilising a 2-inch monochrome CRT driven at a very high accelerating voltage for the size (typically 25 kV), the tube produced an extremely bright picture which was projected via a schmidt lens and mirror assembly onto a semi translucent screen of typically 17 to 19 inches in size. 

Closer look, in all its dusty glory. We can now clearly see there are two boards here, power on the right, something else on the left:

![Two dusty boards](https://user-images.githubusercontent.com/26856618/34652668-412ef36a-f396-11e7-9851-299c95c6bf9b.png)

Remove the plastic panel on the top, it is an impressively shiny mirror, angled facing the screen. Behind it above the circuit boards are lenses from three miniature CRTs, facing backwards (hence "rear projection"):

![The rear projection](https://user-images.githubusercontent.com/26856618/34652682-7d17d5ea-f396-11e7-812a-8f4c269c98d6.png)

Heed the warnings:

![Safety warnings](https://user-images.githubusercontent.com/26856618/34652706-c0f6e206-f396-11e7-9cc7-4b30b00b4503.png)

"To determine presence of high voltage, discharge anode lead to chassis. High vacuum picture tube is dangerous to handle. Refer servicing to qualified service personnel. This picture tube in this receiver employs integral implosion protection. Replace with a tube of the same type number for continued safety. To reduce the risk of possible exposure to X-radiation, take X-radiation protective measures (see service manual) for personnel during servicing. This product includes critical mechanical and electrical parts which are important for safety", and here we find the exact magnitude of the high voltage: "This product is 31.4 kV at normal operaton", wow! Higher than the typical high voltage of RPTVs. Could be useful to someone more adventurous than I, but I'm more interested in the electronic circuitry.

Wiring to the front panel:

![Front panel wiring](https://user-images.githubusercontent.com/26856618/34652828-42ac044c-f398-11e7-90c9-e74d96dcd530.png)

beneath it is another label with a diagram showing how the projectors are positioned facing inward, and that they are for red, green, and blue (RGB), in that order. 

On the front are two APV1013-A 8Ω E950 speakers with 6.5" diameter:

![Front speakers](https://user-images.githubusercontent.com/26856618/34652987-6d341d92-f39a-11e7-994d-c45cf35ab0e8.png)

The system is quite modular, with removable cables between each of the boards. Removing the three small front panel circuit boards, and a shot of the three tubes:

![Small front circuit boards and three tubes](https://user-images.githubusercontent.com/26856618/34653046-202ef2fa-f39b-11e7-92a7-5f18cbb31b30.png)

Removing the tubes, a better view of the boards beneath it, and the nice wooden particle case:

![Boards in box](https://user-images.githubusercontent.com/26856618/34653060-7492a6b6-f39b-11e7-82e3-be6eec68474f.png)

Take out the boards, blow them off, and take a look. How many PCBs does it take to build a big screen TV in the 80s? Six or more, apparently:

## Boards

![Boards overview](https://user-images.githubusercontent.com/26856618/34653166-1397b0b6-f39d-11e7-9eb4-4fb11afdd14e.png)

From top to bottom, right to left, the main boards are labeled as follows, along with my estimate at their function:

* ANPI306-H: power supply
* ANPI309-B: surround sound, Dolby
* ANPI307-C: IF AGC, video level
* ANPI308-C: adjustments and the biggest heatsink I've seen in a while, "convergence board"
* ANPI305-E: main board (also one of the largest I've seen in a while), back panel connectors

The -letter suffix may be a revision code, ANPI30X a part code, we have all of: 5, 6, 7, 8, and 9. Looking at each in turn:

### ANPI305-E main board

This board is impressively large, measuring 15" x 11.75". Single-sided. You don't seem them this large much anymore. When I enter the dimensions in [PCBShopper](https://pcbshopper.com) to get a ballpark quote, it warns me: "You have requested a size of 15 x 11.75 inches. That seems unusually large. Do you want to continue?". In small quantities, getting a board of this size manufactured may set you back over a hundred dollars. Must have been worth it back then, I guess. Barely fits on the bench, these are the best shots I could get of it (and these photos were too big to upload here so I had to resize, bear with me):

![ANPI305-E component side](https://user-images.githubusercontent.com/26856618/34653332-5ddd4788-f39f-11e7-9e2d-80874840a073.png)

The printed side:

![ANPI305 traces](https://user-images.githubusercontent.com/26856618/34653360-a2a1d3e8-f39f-11e7-8505-573097f5157f.png)

Looking at some of the DIP chips:

* PA0040 9KC 110 - unknown, Pioneer ROM?
* AN5302K 9N7 - [Panasonic NTSC Luminance Signal, Chroma-Signal and Synchronous Signal Processing](http://www.alldatasheet.com/datasheet-pdf/pdf/13341/PANASONIC/AN5302K.html)
* Toshiba TC4066BP 8951HB JAPAN, part of the [4000 series](https://en.wikipedia.org/wiki/4000_series), [Quad Bilateral Switch](http://www.alldatasheet.com/datasheet-pdf/pdf/31641/TOSHIBA/TC4066BP.html), PDIP14
* NEC JAPAN D6145C 001 8928DK, found a replacement on eBay: [D6145C 18-PIN INTEGRATED CIRCUIT](https://www.ebay.com/itm/SYL-483520917024-D6145C-18-PIN-INTEGRATED-CIRCUIT-/263414887991), but couldn't find a datasheet
* 80011A 9463 - Aliexpress had a similar chip labeled [80011AA 1371](https://www.aliexpress.com/item/in-stock-80011A-SOP-10-80N70F4-STP80N70F4-78R12F-KIA78R12F-TO-252-75ALS176-SN75ALS176-DIP-8-5pcs/32819833504.html) but didn't describe what it is, if its a Mitsubishi M6M80011AP, then it could be a [1024-bit (64-word by 16-bit) electrically erasable and programmable ROM](http://pdf.datasheetcatalog.com/datasheets2/25/259544_1.pdf), "suitable for nonvolatile channel memory for electronic tuner"
* Pioneer 9H1 PDG040 5048H-197, probably a parallel ROM, an [eBay seller](https://www.ebay.com/p/Pioneer-IC-PDG040-in-Various-Models/1986203632) is asking $39.95 for it
* Toshiba TC4051BP 8952HB JAPAN (2x) - [Single 8-channel multiplexer/demultiplexer](http://pdf.datasheetcatalog.com/datasheet/toshiba/109.pdf)
* PA0030 9K1 116 - another Pioneer chip, this also makes an appearance in the [Pioneer PDP-501MX Plasma Television](https://www.manualslib.com/manual/130294/Pioneer-Pdp-501mx.html), whose technical manual is available (PA0030 is IC9961 there) and is used for sharpness control
* T TA7630P - [Toshiba Dual Volume/Balance/Tone Bass/Treble DC Control IC](http://pdf.datasheetcatalog.com/datasheet/toshiba/3469.pdf)

There are also 3-pin linear voltage regulators: a 78M09A (IC601, 9 volts), and 78M05 (IC206, 5 volts). The chip in the HZIP-12 package with the big heat sink is a [Toshiba TA8200AH Dual Audio Power Amplifier](http://pdf.datasheetcatalog.com/datasheet/toshiba/1143.pdf). This thing has some big speakers, no wonder it needs a big amplifier. Other interesting parts are the brown lumps labeled ATNI1013-A (two of them) and ATNI1014-A, and the large orange box labeled ADL-CT03AU ATN-101A 1907, possibly some radio components? Inductors?

From this an APNI307, scored a few linear voltage regulators, two 7809's and one 7805:

![78M09A 9001B, 78M05 8948A, and 78M09A 9006B](https://user-images.githubusercontent.com/26856618/34656322-cb31ff86-f3cc-11e7-8781-8f2347e1a778.png)

The date codes indicate these were manufactured in 1990 and 1989, respectively.

---

I'd like to try dumping the EEPROM, if that's what it is. 80011A is an 8-pin DIP. Is it the M6M80011AP detailed in [1024-bit (64-word by 16-bit) electrically erasable and programmable ROM](http://pdf.datasheetcatalog.com/datasheets2/25/259544_1.pdf)? From the data sheet:

![M6M80011AP data sheet page 1](https://user-images.githubusercontent.com/26856618/34656397-64fdbcbc-f3ce-11e7-8e77-ae60a96f2ca6.png)

Wired up to a Bus Pirate 3.6 as follows:

| 80011A  | Bus Pirate | Remarks |
| ------- | ---------- | ------- |
| 1 (/CS) | CS | Chip select |
| 2 (/SCK) | CLK | Clock |
| 3 (DI) | MOSI | Slave input |
| 4 (DO) | MISO | Slave output |
| 5 (Vss 0V) | GND | Ground |
| 6 (RESET) | (wire to /CS) | "This pin and /CS can be interconnected" (pg. 3-4) |
| 7 (RDY /BUSY) | AUX | Busy output |
| 8 (Vdd 5V) | 5V | Power |

The data sheet doesn't use the term SPI, but it says "M6M80011AP is a clocked serial port compatible EEPROM, and data is input from the rising edge of the clock signal and output by synchronizing to the falling edge of the clock signal." On the Bus Pirate, type `m` and `5` to select SPI, go with a conservive `1` 30 kHz, `1` clock polarity idle low (default), `1` output clock edge active to idle (default), `1` input sample middle phase, `2` chip select /CS active low (default), `1` output type open drain.

The read command is 0b10101000, followed by the 8-bit address. 16-bit words. Turn on the power with `W` thentTry to read the first word:

```
SPI>[0b10101000 0 rr]
/CS ENABLED
WRITE: 0xA8 
WRITE: 0x00 
READ: 0x00 
READ: 0x00 
/CS DISABLED
SPI>
```

There are 64 words, surely some must be non-zero. What's the problem? For starters, this is a 5 volt part but the Bus Pirate outputs 3.3 V. To solve this, interfacing with non-3.3 chips, the Bus Pirate can output open drain, see: [Hackaday: Mixed voltage interfacing with the Bus Pirate](https://hackaday.com/2009/07/01/mixed-voltage-interfacing-with-the-bus-pirate/). Connect the voltage pull-up (VPU) to 5V, then enable pull-ups with `P`, now it reads all 0xFFFF's:

```
SPI>[0b10101000 0 rr]
/CS ENABLED
WRITE: 0xA8 
WRITE: 0x00 
READ: 0xFF 
READ: 0xFF 
/CS DISABLED
```

No such luck. This chip may be something else than EEPROM, I may have damaged it, or hooked it up wrong. This was my setup:

![80011A to Bus Pirate](https://user-images.githubusercontent.com/26856618/34656787-a474fa36-f3d3-11e7-8d91-6a14ac23e8c6.png)

### ANPI306-H power board

Moving onto the next board, the grimiest of them all. Made some effort to clean it up, but a lot of dust stuck to this board and was covered in oil dripping from the lenses, and its the board with the chunkiest components making it difficult to clean. Almost considered discarding it, but the parts will be easier to clean after removal from the board. Brace yourself, here is this dirty PCB:

![ANPI306-H component side](https://user-images.githubusercontent.com/26856618/34653930-c8c1f37e-f3a8-11e7-985e-37ad878d34b0.png)

The other side is cleaner:

![ANPI306-H trace side](https://user-images.githubusercontent.com/26856618/34653943-19b05776-f3a9-11e7-87b5-79640193c222.png)

This board is serious business, check out those fat traces for high voltage on the hot side. Power resistors made out of [cement](http://www.learningaboutelectronics.com/Articles/What-are-cement-resistors). A thick aluminum heat sink cornering the transformer. Off the board, but nearby, are two 0.47Ωk (not kΩ, I measured - this is milliohms not kilohms) RGB20 NOBLEU9D cement resistors in series. Lots of fuses. The big capacitor in the corner is a 200V 1000µF [Nippon Chemi-Com](https://en.wikipedia.org/wiki/Nippon_Chemi-Con). Predating the [capacitor plague](https://en.wikipedia.org/wiki/Capacitor_plague) of 1999-2007, this capacitor and the others like it (a 200V 560µF nearby) are still good.

Close by is an Omron relay, G2R-1117P-FD-V-US-PN 12VDC, contact: TV-8 10A 250VAC or 10A 30VDC. The tiny transformer is labeled ATT1099 A 35S008 JA 12N SMD, the medium size transformer ATK1044 -ADOTJA12T. Bridge rectifier labeled RB60 SK with +, -, and two `~` terminals. ATF1031 9D J, a current transformer? The flyback transformer has no labels immediately visible, and on the other side are four more even smaller transformers: ATL1058 A SMD (blue), ATL1057 B JA11NSMD (yellow), 260291 8947M (black), and 2250291 8947M (black). There are few integrated circuits, a few transistors here and there. Notable ICs:

* 4558DX JRC (2x), IC552_5 and IC551, DIP-8 - [JRC558 dual operational amplifier](https://www.rcscomponents.kiev.ua/datasheets/jrc45584i743ncft874nfdt34ufguygf43.pdf)

Not my favorite board.

### ANPI307-C RF board

Back to something more manageable. The ANPI307-C is clearly related to RF, there were two metal shields over components, I removed:

![ANPI307-C component side](https://user-images.githubusercontent.com/26856618/34654234-db883176-f3ad-11e7-8a55-9d3a2bfe45f7.png)

Other side:

![ANPI307-C trace side](https://user-images.githubusercontent.com/26856618/34654253-08056430-f3ae-11e7-8f81-dbe464074505.png)

* Toshiba TD6359P 9 J 611 - [Frequency Synthesizer for TV/CATV](http://elcodis.com/parts/5774872/TD6359P.html#datasheet)
* 944003 M51365SP - [Mitsubishi PLL-split VIF/SIF](https://www.silicon-ark.co.uk/datasheets/M51365P-datasheet-mitsubishi.pdf), "consisting of an IF signal processor for color TV sets... circuits include VIF amplifiers, video detector, VCO, APC detector, AFT, video equalizer, plus IF/RF AGC and SIF detector functions", lots of TV-related RF stuff here
* 4.00 NDK99 crystal
* Toshiba F52LM 9H3 JPN - in a metal can, "Active Filter for Telecommunications - TV-IF Saw Filter, 45.7 MHz carrier, adjustment free", maybe useful for radio hobbyists? [Surface Acoustic Wave filters](https://en.wikipedia.org/wiki/Electronic_filter#SAW_filters), an analog implementation of a [FIR filter](https://en.wikipedia.org/wiki/Finite_impulse_response).
* Mitsubishi 5223 9210 [Dual Single Power Supply Operational Amplifier](http://www.alldatasheet.com/datasheet-pdf/pdf/906/MITSUBISHI/M5223L.html), DIP mini flat
* Sony CXA1124AS JAPAN 9G37K - [US Audio Multiplexing Decoder IC](http://www.alldatasheet.com/datasheet-pdf/pdf/46583/SONY/CXA1124.html), "designed for the MTS decoding systems. This device contains a stereo signal demodulator, a SAP (Second Audio Program) signal demodulator and dbx-TV noise reduction decoder. The Main 100% reference input level of the CXA1124 (CXA1534) is 245mVrms (100mVrms)."

The crystal's leads are behind a metal shield, desoldered it then desoldered the crystal. For some reason the crystal had longer leads and they were bent in a zig-zag (were the long leads picking up interference so they put a shield over it instead of cutting them?). Tested the crystal in a frequency tester, expecting 4 MHz. But it only read zero, unclear why (damaged?).

Not much generically useful here, but it does have some [trimmer pots](https://en.wikipedia.org/wiki/Trimmer_(electronics)). Moving on.

### ANPI308-C convergence board

Speaking of trimmer pots, this board has 32 of them, ought to be enough for anybody:

![ANPI308-C component side](https://user-images.githubusercontent.com/26856618/34654414-2174d50c-f3b0-11e7-83f8-79a64538dc97.png)

The DIP chip is a Pioneer PA0036 9LC 111, [sold on eBay for $8.95](https://www.ebay.com/itm/PIONEER-IC-PA0036-USED-IN-VARIOUS-MODELS-/172352020860?rmvSB=true), brand new, "used in various models", but it doesn't say what it is. Given the adjustment labels:

| GV | RH | RV | BH | BV |    |
| -- | -- | -- | -- | -- | -- |
| TP-702 | VR709 | VR718 | VR75 | VR734 | **`SKEW ///`** |
| VR709 | VR710 | VR719 | VR726 | VR735 | **`BOW )))`** |
| VR706 | VR711 | VR720 | VR727 | VR736 | **`KEY \\|/`** |
| VR707 | VR712 | VR721 | VR728 | VR737 | **`SUB KEY`** |
| VR708 | VR713 | VR722 | VR729 | VR738 | **`PIN (\|)`** |
|       | VR714 |       | VR730 |       | **`SUB PIN`** |
|       | VR715 | VR723 | VR731 | VR739 | **`LIN \|->\|->`** |
|       | VR716 |       | VR732 |       | **`SUB LIN \|<-\|<-\|->\|->\|`** |
|       | VR717 | VR721 | VR733 | VR740 | **`SIZE \|<-\|->\|`** |

I'd say it is related to video. The three SIPs, for red, green, and blue, are Mitsubishi M5220L 9702, more opamps: [Dual Low-noise Operational Amplifiers (dual power supply type)](http://www.alldatasheet.com/datasheet-pdf/pdf/158246/MITSUBISHI/M5220L.html). Of course, the elephant in the room is that big heat sink over there. What's under it?

![ANPI308-C trace side](https://user-images.githubusercontent.com/26856618/34654429-41edea12-f3b0-11e7-9551-60c6f2e4215f.png)

Labeled "STK4277 SL OA 18", and much of the board connects to it. Pioneer has it on their website, not available but says retail price $75.25: [PART DETAILS - STK4277-SL](http://parts.pioneerelectronics.com/part.asp?productNum=STK4277-SL). Found a post from someone trying to replace this part in 2004, fone.freaky asked on electronicspoint forums, [Convergence still off after replaced STK4277 in Pioneer 6081 tv](https://www.electronicspoint.com/threads/convergence-still-off-after-replaced-stk4277-in-pioneer-6081-tv.55584/), John Del replied:

> The STK is just the amplifier, you need to scope the error correction waveform inputs that come from the ramp generator section.

The STK4277-SL is a replacement for the $80 [STK4729-SL](http://parts.pioneerelectronics.com/part.asp?productNum=STK4279%2DSL). Found a [cached page on Aliexpress](https://webcache.googleusercontent.com/search?q=cache:0vXl5iZ1LM4J:https://www.aliexpress.com/store/product/STK4279-2-Channel-AF-Power-Amp/835156_1603363384.html+&cd=3&hl=en&ct=clnk&gl=us) ([mirror](http://archive.is/nbW2E)), selling it for $8.90 as "STK4279:2 Channel AF Power Amp". [Newegg has 50 for $217.33](https://www.newegg.com/Product/Product.aspx?Item=2A3-0029-0TM98). [eBay has it for $5.95](https://www.ebay.com/itm/PIONEER-STK4279-STK4277-CONVERGENCE-IC-REPLACEMENT-PART-VARIOUS-TV-MODELS-/322497898055?_ul=CL). [bdent](http://www.bdent.com/stk4279-sanyo-convergence-sip25-integrated-circuit.html) calls it a "5-channel Convergence Correction Circuit IC for video projectors", a SIP-25 convergence IC. [Amazon for $22](https://www.amazon.com/Sanyo-STK4277-SANYO-INTEGRATED-CIRCUIT/dp/B002M6F5N2). 

Couldn't find the exact data sheet, but we know it is a 5-channel convergence correction power amplifier. Found the [STK4145MK2 data sheet](http://www.alldatasheet.com/datasheet-pdf/pdf/108397/ETC/STK4145MK2.html), maybe not too dissimilar. Can this chip be usefully used in another circuit?

### ANPI309-B audio board

Moving into the final stretch, this board is for audio, indicated by the "Dolby Level Adjust" text:

![ANPI309-B component side](https://user-images.githubusercontent.com/26856618/34654682-1c8be09a-f3b4-11e7-8b34-3af687da1cbe.png)

![ANPI309-B trace side](https://user-images.githubusercontent.com/26856618/34654689-32b334d6-f3b4-11e7-91f6-065680ee2613.png)

Were [Single Inline Packages](https://en.wikipedia.org/wiki/Dual_in-line_package#Single_in-line) (SIP) more common back then? This board has several, as well as some DIPs. Or actually, they are ZIP, [Zig-zag in-line packages](https://en.wikipedia.org/wiki/Zig-zag_in-line_package). List:

* BU4066BL (2x) (IC510, IC508) - [Quad Analog Switch](http://rohmfs.rohm.com/en/products/databook/datasheet/ic/logic_switch/standard_logic/bu4066xx-e.pdf), but in ZIP-14
* Mitsubishi M5222L - [Dual VCA for Low Voltage Electronic Volume Control](http://datasheet.octopart.com/M5222L-Mitsubishi-datasheet-108557.pdf), SIP-8
* Sanyo LA2730 - [Dolby B-Type Noise Reduction System](https://www.utsource.net/itm/p/1272647.html)
* Mitsubishi M5218L - [Dual low-noise operational amplifiers (dual power supply type)](http://www.alldatasheet.com/datasheet-pdf/pdf/908/MITSUBISHI/M5218.html)
* Sanyo LM3364K, $6.95/10 on eBay reveals this is RAM: [SANYO LM3364K-15 Dynamic RAM Gated RAS-Only Refresh 16-Pin Dip Qty-10](https://www.ebay.com/itm/SANYO-LM3364K-15-Dynamic-RAM-Gated-RAS-Only-Refresh-16-Pin-Dip-Qty-10-/371281693982)
* Mitsubishi 944002 M50199P [Mitsubishi Sound Processors M50199P Digital Delay](https://www.datasheetarchive.com/pdf_dl/?f=3694162&t=1&g=M50199)
* Mitsubishi M5233 9701 (IC502_5), [discussion](https://www.electronicspoint.com/threads/does-anyone-know-m52332-op-amp-maybe-from-mitsubishi-very-old-part-i-need-pinout.14539/). This is a [Dual Comparator](http://www.alldatasheet.com/datasheet-pdf/pdf/920/MITSUBISHI/M5233L.html), "a differential circuit which is equivalent to a conventional single power supply operational amplifier", voltage range from 2 to 36V.

Could be an interesting project to try using the DRAM somehow. dmitry.gr's [Linux on an 8-bit micro?](http://dmitry.gr/index.php?r=05.Projects&proj=07.%20Linux%20on%208bit) used a 30-pin SIMM memory module, refreshed every 62 ms, driven from an ATmega644a, for a maximum bandwidth of 30 KB/s. Could the  Sanyo LM3364K dynamic RAM be driven in the same way? But I had trouble finding a data sheet, and it is not clear that's even what this chip is.

### Front panel boards

Not much to these, buttons and jacks. There's a lot of good switches here, I desoldered one but it took too long, taking them all off would best wait for a hot air gun. But the cadmium sulfide CdS [photoresistor](https://en.wikipedia.org/wiki/Photoresistor) is worth salvaging. I measured about 15 kΩ resistance in ordinary room lighting, and about 5 kΩ under a bright lamp.

![Front panel component sides](https://user-images.githubusercontent.com/26856618/34654879-847b7f56-f3b6-11e7-86ea-b7beff6c5bab.png)

![Front panel trace sides](https://user-images.githubusercontent.com/26856618/34654888-9c3fd7f4-f3b6-11e7-8fac-05426fd8b973.png)

The RF adapter is actually not part of this TV, but was a separate unit labeled "Dreamcast Model No. HKT-8820 NTSC-M CH3-CH4", with ANT.IN antenna input, TV output, A/V IN, and a select channel 3 or channel 4 switch:

![Dreamcast and answering machine](https://user-images.githubusercontent.com/26856618/34654916-0d8ec0e6-f3b7-11e7-8eb4-87733fee2c26.png)

[Dreamcast](https://en.wikipedia.org/wiki/Dreamcast) was a 1999 home video game console. This adapter was used to allow the TV to pickup the video game console on analog TV [channel 3 or 4](https://en.wikipedia.org/wiki/Channel_3/4_output):

> A channel 3/4 output was a common output selection for consumer audiovisual devices sold in North America that were intended to be connected to a TV using a radio frequency (RF) signal. This channel option was provided because it was rare to have broadcast channels 3 and 4 used in the same market, or even just channel 3 itself. The choice allowed the user to select the unused channel in their area so that the connected device would be able to provide video and audio on an RF feed to the television without excessive interference from a broadcast signal.

Bend away the metal shields, and parts can be desoldered from the boards. Nothing terribly interesting, but the color-coded inductors look kinda cool, they came in red, green, and blue:

![RF to channel 3/4 desoldered](https://user-images.githubusercontent.com/26856618/34655981-5ae3de16-f3c7-11e7-947e-3d6814942ad7.png)

[Memories of Analog TV: The 90s and 00s](https://www.sandiegoreader.com/weblogs/san-diego-radio-views/2009/apr/22/memories-of-analog-tv-the-90s-and-00s/#) provides some insight into the old analog era. Nowadays, it is largely superseded by [digital televison](https://en.wikipedia.org/wiki/Digital_television), analog services turned off as part of the [digital television transition](https://en.wikipedia.org/wiki/Digital_television_transition). Receiving modern television on this big screen  would have required a [digital television adapter](https://en.wikipedia.org/wiki/Digital_television_adapter). Better to buy a new TV.


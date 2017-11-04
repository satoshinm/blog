# Routefinder RF102S dual serial router + 4-port 10/100 Ethernet switch teardown

by snm, November 3rd, 2017

![Front panel](https://user-images.githubusercontent.com/26856618/32399322-31b3cb8e-c0b2-11e7-8471-568b9de39300.png)

The MultiTech Systems "Routefinder RF102" is a dual serial port router with a built-in 4-port 10/100 Ethernet switch. Slow by today's standards (with [Gigabit Ethernet](https://en.wikipedia.org/wiki/Gigabit_Ethernet) or faster), this device could be useful in 2005 when it was manufactured. The two serial ports are for connecting to an [ISDN](https://en.wikipedia.org/wiki/Integrated_Services_Digital_Network) or [56k modem](https://en.wikipedia.org/wiki/Modem#Using_digital_lines_and_PCM_.28V.90.2F92.29) uplink, both now obsolete technologies. So let's crack it open:

![Circuit board](https://user-images.githubusercontent.com/26856618/32399418-f8e002ea-c0b2-11e7-8564-5bd47eccb9d9.png)

The board is silkscreened "RF102S Ver 10 ISDN/56k Router+Switch", an apt description.

Notable chips:

* [SQ-H44R Quad Port Magnetics Module](https://www.digchip.com/datasheets/parts/datasheet/922/SQ-H44R-pdf.php), connects to the 4 Ethernet ports
* [Kendin KS8995 LAN Switching Circuit](http://www.alldatasheet.com/view.jsp?Searchword=KS8995), providing the guts of the Ethernet switch
* [74HC132D Quad 2-input NAND Schmitt trigger](https://assets.nexperia.com/documents/data-sheet/74HC_HCT132.pdf)
* Various LEDs

The serial circuitry on the right is more complex:

* 2 x IMP16C550 [16550 UARTs](https://en.wikipedia.org/wiki/16550_UART), "(universal asynchronous receiver/transmitter) is an integrated circuit designed for implementing the interface for serial communications"
* 2 x [HIN208 +5V powered RS-232 transmitters/receivers](https://www.intersil.com/content/dam/Intersil/documents/hin2/hin202-06-07-08-11-13.pdf), the 208 model contains 4 x RS-232 drivers and 4 x RS-232 receivers
* 2 x [ESMT M12L1616A-7T 512K x 16-bit x 2-bank Synchronous RAM](http://www.alldatasheet.com/view.jsp?Searchword=M12L1616A-7T)
* [Eon Silicon EN29F040A 4 Megabit (512K x 8-bit) Flash Memory](http://www.alldatasheet.com/datasheet-pdf/pdf/103018/EON/EN29F040A.html)
* [Altera EPM3032ALC44-10 Programmable Logic Device, MAX 3000A family](https://www.altera.com/content/dam/altera-www/global/en_US/pdfs/literature/ds/m3000a.pdf)
* [HT93LC46 1K 3-Wire CMOS Serial EEPROM](http://www.alldatasheet.com/datasheet-pdf/pdf/64514/HOLTEK/HT93LC46.html?)
* [Samsung S3C4510B01-QER0 16/32-bit RISC (ARM7TDMI) microcontroller](http://www.icchip.info/datasheet/S3C4510B01-QER0.pdf) - with integrated 8 KB SRAM and Ethernet controller
* T025.000 MHz (X1), 10.000 MHz (X2), X3 ???, 7.3735 MHz (X4) crystals

On the reverse side, there is not much but surface-mount passives:

![Reverse board](https://user-images.githubusercontent.com/26856618/32399723-a78fce2c-c0b5-11e7-89e6-dfa52563a655.png)

## Dumping the 1 Kbit EEPROM

Taking a closer look at the HT93LC46 1k 3-wire serial EEPROM, which appears to be connected to the PLD.

![HT93LC46](https://user-images.githubusercontent.com/26856618/32399759-1009f892-c0b6-11e7-8aca-9447372de49f.png)

Desolder the chip and insert into a breadboard:

![HT93LC46 in breadboard](https://user-images.githubusercontent.com/26856618/32399773-3ed8d68e-c0b6-11e7-93a5-4446d048b6af.png)

Powered the device up without the EEPROM and verified the 4-port 10/100 Mbps switch continued to operate normally:

![Switch operating without EEPROM](https://user-images.githubusercontent.com/26856618/32399229-991d41de-c0b1-11e7-8b00-467d065882fb.png)

This is what we would expect since the EEPROM and PLD appear to be used for the serial ports (ISDN/56k), not the Ethernet ports.

Now to dump it. Found this helpful thread on the Dangerous Prototypes forum: [93LC46 eeprom dump](http://dangerousprototypes.com/forum/viewtopic.php?f=4&t=5273)

Referring to the [HT93LC46 Datasheet](http://www.alldatasheet.com/datasheet-pdf/pdf/64514/HOLTEK/HT93LC46.html?), operating voltage 2.0V to 5.5V read and 2.4V to 5.5V write, so used 5 V. Clock frequency at 5V±10% minimum 0 maximum 2000 kHz, so any of the Bus Pirate's supported frequencies should work. Probing the ORG pin (#6) in circuit, reads 0.7 V = low, so the memory is organized as 128 groups of 8 bits: 128 bytes (as opposed to 64 groups of 16 bits).

Configuring the mode mode on a Bus Pirate 3.6:

   m
   7 (3WIRE)
   1 (~5KHz)
   1 (CS)
   2 (normal)

Turn on the power supply with "W". Wire up the AUX pin to ORG, then pull it low with "a" (128 groups of 8-bits = 128 bytes).

Send the "READ" instruction, opcode 0b10. Start bit 1. 

```
3WIRE>[0b110 0 r:128]
CS ENABLED
WRITE: 0x06 
WRITE: 0x00 
READ: 0x11 0x82 0x47 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFE 0x11 0x82 0x45 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFE 0x00 0x00 0xB1 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFE 0x00 0x00 0xB1 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF 0xFE 
CS DISABLED
3WIRE>
```

Reformatting 16 bytes per line:

```
11 82 47 FF FF FF FF FF  FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF  FF FF FF FF FF FF FF FF
FF FE 11 82 45 FF FF FF  FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF  FF FF FF FF FF FF FF FE
00 00 B1 FF FF FF FF FF  FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF  FF FF FF FF FF FF FF FF
FF FE 00 00 B1 FF FF FF  FF FF FF FF FF FF FF FF
FF FF FF FF FF FF FF FF  FF FF FF FF FF FF FF FE
```

TODO: what is this? An ASCII hexdump -C with offset:

```
00000000  11 82 47 ff ff ff ff ff  ff ff ff ff ff ff ff ff  |..G.............|
00000010  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
00000020  ff fe 11 82 45 ff ff ff  ff ff ff ff ff ff ff ff  |....E...........|
00000030  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff fe  |................|
00000040  00 00 b1 ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
00000050  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
00000060  ff fe 00 00 b1 ff ff ff  ff ff ff ff ff ff ff ff  |................|
00000070  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff fe  |................|
00000080
```


## Programmable Logic Device

This router contains an Altera EPM3032ALC44, part of the MAX 3000A family. The family's [datasheet](https://www.altera.com/content/dam/altera-www/global/en_US/pdfs/literature/ds/m3000a.pdf) reveals the EPM3032A has 600 usable gates, with 32 macrocells, 2 logic array blocks, and 32 maximum user I/O pins. This makes it the smallest of the series:

| Feature | EPM3032A | EPM3064A | EPM3128A | EPM3256A | EPM3512A |
| ------- | -------- | -------- | -------- | -------- | -------- |
| Usable gates | 600 | 1,250 | 2,500 | 5,000 | 10,000 |
| Macrocells | 32 | 64 | 128 | 256 | 512 |
| Logic array blocks | 2 | 4 | 8 | 16 | 32 |
| Maximum user I/O pins | 34 | 66 | 98 | 161 | 208 |

but it looks to be still used, especially in networking equipment. Another user salvaged the same EPM3032A (from "NetApp DS14 Filers") and [designed a breakout board](https://bradthx.blogspot.co.uk/2011/11/altera-epm3032a-cpld-breakout-board.html), then programmed it successfully with Quartus II, writing a simple 4-bit counter in [VHDL](https://en.wikipedia.org/wiki/VHDL). This looks promising, the author concludes:

> The EPM3032A is not a large CPLD, with only 32 macrocells it's by no means a device for large scale logic implementations. The 4 bit counter ended up using 4 macrocells or 13% of the usable space in the CPLD, but it is perfect when you need a small custom logic device where many individual chips would be required.

While searching more information about this device, found a [EPM3032 CPLD Breakout Board](https://oshpark.com/shared_projects/75P2GAzi) on OSHPark. This PCB measures 1.07x2.37" so it would cost about $10 to manufacture with their prototype service. It was made for an online course, [PyroEDU Course 5: FPGA and CPLD](http://www.pyroelectro.com/edu/fpga/), and the completed populated [CPLD Breakout Board (EPM3032ATC44-10)](https://gadgetory.com/index.php?route=product/product&filter_name=EPM3032&product_id=124) can be ordered for $14.95, however, it is for the TQFP package not the PLCC which is the package used in this device. The 44-pin [TQFP](https://en.wikipedia.org/wiki/Quad_Flat_Package) (thin quad flat package) and 44-pin [PLCC](https://en.wikipedia.org/wiki/Chip_carrier#Plastic_leaded_chip_carrier) (plastic leaded chip carrier) pinouts are completely different:

![EPM3032A pinouts PLCC and TQFP](https://user-images.githubusercontent.com/26856618/32399189-56a4ec9e-c0b1-11e7-9119-e2f721d90f49.png)

so unfortunately PyroEDU's breakout board will not be useful for using this chip (although it could still be useful for educational purposes, the complete [Introduction to FPGA and CPLD Kit](https://gadgetory.com/index.php?route=product/product&path=66&product_id=126) is only $37.95 from The Gadgetory, for the 10-lesson course). 

There are other comparable introductory kits, such as [Adafruit's Mojo FPGA Development Board](https://www.adafruit.com/product/1553) for $79.95. Mojo uses a Xilinx Spartan 6 XC6SLX9 FPGA instead of an Altera EPM3032 CPLD, and their [tutorial](https://embeddedmicro.com/tutorials/mojo/) uses [Verilog](https://en.wikipedia.org/wiki/Verilog) instead of [VHDL](https://en.wikipedia.org/wiki/VHDL). [CPLDs](https://en.wikipedia.org/wiki/Complex_programmable_logic_device) come with built-in non-volatile memory to store the configuration, whereas [FPGAs](https://en.wikipedia.org/wiki/Field-programmable_gate_array) usually don't. 

TODO: is it possible to dump the programmed logic on this CPLD's non-volatile memory?

TODO: learn how to desolder/solder PLCCs (hot air station?), and make a breakout board for the EPM3032ALC44 (not the EPM3032ATC44)

but further research into FPGAs and CPLDs will have to wait for later. Although they are very powerful, there are even powerful video game consoles built on FPGAs such as the [Analogue Nt mini](https://www.analogue.co/pages/nt-mini/).

## Flash memory

The 512 KByte (4 Megabit) [EN29F040A](http://www.alldatasheet.com/datasheet-pdf/pdf/103018/EON/EN29F040A.html) flash memory is connected to the [ARM7TDMI microcontroller](http://www.icchip.info/datasheet/S3C4510B01-QER0.pdf), likely as operating system code. According to Wikipedia, [ARM7TDMI](https://en.wikipedia.org/wiki/ARM7#ARM7TDMI) stands for "ARM7 + 16 bit Thumb + JTAG Debug + fast Multiplier + enhanced ICE", and was one of the most popular ARM cores in 2009. Reverse-engineering it is out of the scope of this post, but we can take a closer look at the flash:

![EN29F040A flash](https://user-images.githubusercontent.com/26856618/32400120-7ecf128c-c0b9-11e7-9a22-e01e5e9cfb4c.png)

Pinout for the PLCC package from page 4 of the data sheet (their blurry text, not mine):

![EN29F040A PLCC pinout](https://user-images.githubusercontent.com/26856618/32400130-9b6fed4e-c0b9-11e7-95b0-04f744ce37e4.png)

The designer of this board seemed to really like PLCC packages. Probably good for production, but not for us to salvage and repurpose these components. Anyways, A0-A18 are the address lines, and DQ0-DQ8 are the data output bits. Vcc is 5.0V, /CE chip enable, /OE output enable, /WE write enable. With the right tools, dumping this chip should be relatively trivial, but the PLCC package and parallel interface makes it beyond my reach. Oh well.

## Crystals

There are several crystals on this board, of different frequencies and shapes. Desoldered the X4 crystal and confirmed the 4-port switch still functioned (as we would expect since it is for the serial function, not Ethernet). Measuring the X4 crystal with a frequency counter shows 7.3735 MHz, not far from the "T07.372" label:

![7.3735 MHz](https://user-images.githubusercontent.com/26856618/32400488-09140372-c0be-11e7-8949-09339f086e90.png)

The other crystals I would guess are more important to the function of the switch so I didn't bother desoldering them, but at least I got a working 7.3735 MHz to add to my parts bin.

## Connectors

Since I'm not planning on using ISDN/56k (who uses that anymore?), I desoldered the serial ports:

![Serial ports desoldered](https://user-images.githubusercontent.com/26856618/32401110-230ee08c-c0c6-11e7-8318-e0a0c582f662.png)

The power connector could also be worthy of salvaging, to use along with the 5 V power brick (TTL logic levels), but I kept it so the device could be used as a switch without further modifications. After removing the serial ports, the Ethernet switch continues to operate normally as you would expect:

![Switch without serial](https://user-images.githubusercontent.com/26856618/32401129-6daf1a6c-c0c6-11e7-8834-21c15a6b8649.png)


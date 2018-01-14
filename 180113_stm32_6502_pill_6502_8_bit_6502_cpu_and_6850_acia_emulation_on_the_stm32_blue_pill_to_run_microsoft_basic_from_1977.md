# pill_6502: 8-bit 6502 CPU and 6850 ACIA emulation on the STM32 blue pill to run Microsoft BASIC from 1977

by snm, January 13th, 2018

![pill_6502 running Microsoft BASIC](https://user-images.githubusercontent.com/26856618/34910886-d1869f5e-f872-11e7-8dd9-e034348619dd.png)

This post describes emulating an 8-bit 6502 processor and communications interface to allow running 1977 Microsoft BASIC on the STM32F103 blue pill ARM microcontroller board. Read on for excruciating detail about what I tried, but if you want to just see the source code and firmware to run it yourself, skip to the pill_6502 GitHub repository at: **[https://github.com/satoshinm/pill_6502](https://github.com/satoshinm/pill_6502)**

---

* auto-gen TOC:
{:toc}

Inspired by [EEVblog Guest Video: Raising Awesome - Young Dave Jones](https://www.youtube.com/watch?v=vr8ROiYs8AQ), a demonstration of building a Simple6502 computer, a minimal [MOS Technologies 6502](https://en.wikipedia.org/wiki/MOS_Technology_6502) computer using Grant Searle's design: [Grant's 8-chip (or 7-chip) 6502 computer](http://searle.hostei.com/grant/6502/Simple6502.html), I became interested in building a 6502 computer system. Grant also has minimal [Zilog Z80](https://en.wikipedia.org/wiki/Zilog_Z80) and [Motorola 6809](https://en.wikipedia.org/wiki/Motorola_6809) designs, also from the 8-bit era. But the 6502 was famously used in the [Nintendo Entertainment System](https://en.wikipedia.org/wiki/Nintendo_Entertainment_System) and pioneering personal computers of the time. Building a 6502-based computer in the current year is an exercise in [retrocomputing](https://en.wikipedia.org/wiki/Retrocomputing).

So what are the specs of the Simple6502?

> * 16K ROM
> * 32K RAM
> * 6502 Processor with a 1.8432MHz crystal (0.9216MHz or 1.8432MHz clock)
> * 115200 Baud serial interface, RS232 specification voltage levels
> * Hardware serial handshaking ensures characters not lost when transferring programs from the PC to this board
> * Power consumption - 160mA
> * Microsoft BASIC
> * Minimal possible component count - 8 (or 7 if running the CPU at 1.8MHz) ICs and a small number of discrete components

You can run the Microsoft BASIC implementation from 1977, that's pretty cool. But to make it you need at least a 6502 CPU ([~$1.65](https://www.aliexpress.com/item/5pcs-lot-Mos-6502-in-stock-can-pay/32616171390.html)), a 62256 32KB RAM ([~92¢](https://www.aliexpress.com/item/HM62256-in-stock-can-pay/32625469880.html)), 27128 16KB ROM (???), and interfacing and glue logic. Could be a fun project, but the price quickly adds up. How would the Simple6502 compare to a modern, yet dirt cheap (~$1.69), microcontroller such as the STM32 [Blue Pill](http://wiki.stm32duino.com/index.php?title=Blue_Pill)?

|     | Simple6502 | STM32F103C8 Blue Pill |
| --- | ---------- | --------------------- |
| ROM | 16 KB | 64 KB / 128 KB flash |
| RAM | 32 KB | 20 KB |
| CPU architecture | 6502 | ARM Cortex M3 |
| CPU speed | 1.8 MHz | 72 MHz |
| Interface | RS-232 serial | USB |
| Cost | >$2 | <$2 |

The blue pill only loses on RAM. This is unfortunate, but not fatal. The [NES only has 2 KB internal RAM](http://wiki.nesdev.com/w/index.php/CPU_memory_map), as a point of comparison. External RAM could possibly be added, I happen to have a 64KB DRAM chip, see *[Sanyo LM3364K dynamic RAM (DRAM) pinout decoding](https://satoshinm.github.io/blog/180112_dram_lm3364k_sanyo_lm3364k_dynamic_ram_dram_pinout_decoding.html)*, but connecting it to the microcontroller is non-trivial so I'm sticking with the built-in RAM for now.

## Evaluating existing 6502 emulators

Since the blue pill is at least as powerful as a 6502 system in all regards but the RAM, could it reasonably emulate a 6502? Make a small NES emulator? The 72 MHz ARM should be more than be able to keep up with the 1.8 MHz 6502 and PPU. If this is possible, then the blue pill may be able to usefully serve as a low-cost widely available component of a retrocomputer, and it could still integrate with other peripherals, perhaps even maintaining some semblance of compatible pinout.

### NES emulators

What emulator to port? [Nesdev wiki: Emulators](http://wiki.nesdev.com/w/index.php/Emulators) shows the most popuilar NES emulators, which all include 6502 processor emulation. [higan](https://byuu.org/emulation/higan/) is a multi-system emulator, source code at GitLab, "unofficial": [https://gitlab.com/higan/higan](https://gitlab.com/higan/higan). The author has major plans: [2018 goal - NES emulation improvements](https://board.byuu.org/viewtopic.php?f=4&t=1901&sid=f5a477f0f6827cbba637b3664f5b2a51). Or how about [Nestopia UE](http://0ldsk00l.ca/nestopia/)? Source at GitHub: [https://github.com/rdanbrook/nestopia])(https://github.com/rdanbrook/nestopia), and it is cross-platform. Executed `mkdir build; cd build; cmake ..; make` but it required libarchive; need prerequisites to install.

Is there a more lightweight embedded 6502 emulator? Something less complete, but smaller. [KNES](https://github.com/komrad36/KNES) was speed-written in a few days with minimal dependencies, looks promising, but only supported Windows and Linux. Nestopia, or some variant of it, is probably the best bet for a complete NES emulator.

### 6502 emulators for microcontrollers: from Cortex-M0+ to M4

But looking at something simpler, what about just a 6502 emulator? Simple6502 isn't as nearly complex as a NES. No graphic output (PPU), no audio output (APU), the only I/O is a serial interface at 0xa000-0xbfff. Turn away from the NES scene for a moment and look towards 6502.org: [Development Tools: Emulators](http://www.6502.org/tools/emu/). Listed first is web-based 6502 emulators (JavaScript), then emulators written in other languages like C#, but further down the page there are emulators written in C like the cycle-accurate [fake6502.c](http://rubbermallet.org/fake6502.c) by Mike Chambers (supplied as a single 971-line C file, you provide functions to read and write memory, and the main function to drive the CPU). But in the third section, there is even a list of 6502 emulators on microcontrollers! Seems I wasn't the first to think of this.

6502 on a Microchip PIC, or an Atmel ATmega, but the blue pill is an ARM, and there are several ARM-based 6502 emulators. Going through each in turn.

[TGL-6502](https://github.com/thegaragelab/tgl6502) by Shane Gough. Forum thread: [fuzix, caching, and emulated 6502 on an 8-pin ARM](http://forum.6502.org/viewtopic.php?t=3146) and [blog](http://blog.thegaragelab.com/tag/tgl6502/). Does not use a STMicroelectronics STM32, but an NXP [LPC810](https://www.nxp.com/products/processors-and-microcontrollers/arm-based-processors-and-mcus/lpc-cortex-m-mcus/lpc800-series-cortex-m0-plus-mcus/low-cost-microcontrollers-mcus-based-on-arm-cortex-m0-plus-cores:LPC81X_LPC83X) with even more constrained resources than the blue pill: 30 MHz (vs 72 MHz), 4 KB flash (vs 64/128 KB), and a mere 1 KB of RAM (vs 20 KB in the blue pill). The 6502 CPU is implemented in [tgl6502/firmware/cpu/mos6502/cpu6502.c](https://github.com/thegaragelab/tgl6502/blob/e19af313a3a10979f80ec5c7c3f843e01c89e3b4/firmware/cpu/mos6502/cpu6502.c) with this comment on its origins:

```
* 24-Nov-2014 ShaneG
*
* This implementation was taken from a Stack Exchange post found here -
*  http://codegolf.stackexchange.com/questions/12844/emulate-a-mos-6502-cpu
*
* Original code is (c)2011-2013 Mike Chambers and was released as the
* Fake6502 CPU emulator core v1.1. Modifications have been made to allow it
* run on an AVR processor.
```

Now this is what I'm looking for! [Code Golf: Emulate a MOS 6502 CPU](https://codegolf.stackexchange.com/questions/12844/emulate-a-mos-6502-cpu), Mike Chambers posted his fake6502.c. Looks quite promising. But to complete the survey, what else is out there?

[PiTubeClient](https://github.com/hoglet67/PiTubeClient) by Dave Banks, an "emulator on Raspberry Pi baremetal (ARM)", stardot forums thread: [Re: Raspberry Pi Zero as a Second Processor anyone?](http://stardot.org.uk/forums/viewtopic.php?f=3&t=10421&p=127347#p127347). Relatedly, [PiTubeDirect: A Raspberry Pi as a BBC Micro Second Processor](https://news.ycombinator.com/item?id=16117053), and this informative post by myelin giving some great context:

> For more background on the whole Second Processor system, see the "Acorn's Second Processors and the Tube - what, and why" thread at Stardot: [http://www.stardot.org.uk/forums/viewtopic.php?f=3&t=14211](http://www.stardot.org.uk/forums/viewtopic.php?f=3&t=14211)

> On that note, Stardot is a small but thriving community around the BBC Micro, and Acorn Computers' other machines -- the Acorn Atom, Acorn Electron, BBC Master, BBC Master Compact, and the Archimedes/RISC PC machines. The author of PiTubeDirect is an active poster there, and the tone is generally very friendly and on-topic. I've been reading/posting for the last year or so and it's turned Acorn-focussed retro hardware into my favourite hobby. I'd highly recommend taking a look if you used/loved one of these machines back in the 80s/90s.

Indeed, this is the ARM of [Arm Holdings](https://en.wikipedia.org/wiki/Arm_Holdings) as in "Acorn RISC Machine", who still license (newer versions of) ARM processor cores today, including the [ARM Cortex-M3](https://en.wikipedia.org/wiki/ARM_Cortex-M#Cortex-M3) in the infamous STM32F103 blue pill. Small world!

The other two 6502 emulators for ARMs are for the STM F4: [stm6502](https://github.com/BigEd/stm6502) by Chris Baird (2012), and [a6502](https://github.com/BigEd/a6502) by Ed Spittles (2016) which was inspired by stm6502. Forum threads: [stm6502 -- a 6502 simulator for the STM32F4Discovery board](http://forum.6502.org/viewtopic.php?t=2177) and [a6502 - an emulator in ARM assembly (for STM32F4Discovery)](http://forum.6502.org/viewtopic.php?t=2209). I like [Chris Baird's introduction to stm6502](https://github.com/BigEd/stm6502/blob/86cbdd6b9d8d83a3b6ace50ef4aa1063628b6877/README#L1-L8) in his readme, I'll reproduce here:

```
stm6502 is a (currently NMOS) 6502 CPU simulator for the
STMicroelectronics STM32F4-Discovery evalutation board, a neat little
product with a 168MHz Cortex-M4 processor, 128 kB of RAM, and 1 MB of
Flash rom... for under $20. (See your favourite electronics retailer
and http://www.st.com/internet/evalboard/product/252419.jsp for more
details.) And unlike that Tivo-ized PIC32 rubbish, you can hack on it
with a wholly free development system on a wholly GNU system, without
Corporate wankers having final control on what you do or how you do it.
```

...under $20, I can do you one better Chris, the blue pills are under $2, an order of magnitude cheaper than the [STM32F4DISCOVERY](http://www.st.com/en/evaluation-tools/stm32f4discovery.html) kits. But he's made progress towards what I was also aiming for, looking through the code, it implements a USB CDC-ACM virtual serial port (see my previous post about what this is and why it is useful: *[Triple USB-to-serial adapter using STM32 blue pill](https://satoshinm.github.io/blog/171223_stm32serial_triple_usb-to-serial_adapter_using_stm32_blue_pill.html)*, and [pill_serial](https://github.com/satoshinm/pill_serial)). This obviates the need for any interfacing chip, either an a separate USB-to-serial adapter or the [Asynchronous Communications Interface Adapter (ACIA)](http://www.cpcwiki.eu/index.php/6850_ACIA_chip) as in Simple6502. stm6502 even uses [libopencm3](http://libopencm3.org), which I previously became familiar with in *[JTAG/SWD debugging via Black Magic Probe on an STM32 blue pill and blinking a LED using STM32CubeMX, libopencm3, and bare metal C](https://satoshinm.github.io/blog/171223_jtagswdpillblink_jtagswd_debugging_via_black_magic_probe_on_an_stm32_blue_pill_and_blinking_a_led_using_stm32cubemx_libopencm3_and_bare_metal_c.html)*, and have had good experience with.

The options are converging. On the low end, ShaneG's TGL-6502 based on the LPC810 (30 MHz Cortex-M0_, 4KB flash, 1KB RAM) and Mike Chamber's fake6502, on the higher end stm6502/a6502 targeting the STM32F4DISCOVERY (168 MHz Cortex-M4, 1 MB flash, 128 KB RAM), and in the middle, our cherished blue pill with the 72 MHz Cortex-M3, this niche right in the middle:

| Emulator | Target CPU | CPU Speed | Flash | RAM |
| -------- | ------ | --------- | ---------- | -------- |
| [tgl6502](https://github.com/thegaragelab/tgl6502) | ARM Cortex-M0+ | 30 MHz | 4 KB | 1 KB |
| ??? | ARM Cortex-M3 | 72 MHz | 64-128 KB | 20 KB |
| [stm6502](https://github.com/BigEd/stm6502), [a6502](https://github.com/BigEd/a6502) | ARM Cortex-M4 | 168 MHz | 1 MB | 128 KB |

Without installing any dependencies, I tried to naively build a6502:

```
a6502 $ make
arm-none-eabi-gcc -mcpu=cortex-m4 -std=c99 -Os -g -Wall -Wextra -fno-common -mcpu=cortex-m4 -mthumb -msoft-float -MD -DSTM32F4 -DIMAGEFILE=extras/cmon.bin -DDEBUG -c a6502.c
a6502.c:24:37: fatal error: libopencm3/stm32/f4/rcc.h: No such file or directory
 #include <libopencm3/stm32/f4/rcc.h>
                                     ^
compilation terminated.
make: *** [a6502.o] Error 1
```

needs libopencm3, they didn't add a specific commit as a git submodule as recommended :(. How about stm6502:

```
stm6502 $ make
arm-none-eabi-gcc -O3 -g -std=gnu99 -fno-common -Wall -mcpu=cortex-m4 -mtune=cortex-m4 -mthumb -march=armv7-m \
	 -DSTM32F4 -I. -I/usr/local/arm-none-eabi/include \
	 -o addressing_modes.o -c addressing_modes.c
addressing_modes.c:1:0: warning: switch -mcpu=cortex-m4 conflicts with -march=armv7-m switch
 // addressing_modes.c -- involving the various virtual cpu instruction
 
addressing_modes.c: In function 'addrmode_indx_deref':
addressing_modes.c:171:7: warning: assignment makes pointer from integer without a cast [-Wint-conversion]
   ret = cpu_peek_16(cpu, addr);
       ^
addressing_modes.c:173:10: warning: return makes integer from pointer without a cast [-Wint-conversion]
   return ret;
          ^~~
arm-none-eabi-gcc -O3 -g -std=gnu99 -fno-common -Wall -mcpu=cortex-m4 -mtune=cortex-m4 -mthumb -march=armv7-m \
	 -DSTM32F4 -I. -I/usr/local/arm-none-eabi/include \
	 -o cpu.o -c cpu.c
cpu.c:1:0: warning: switch -mcpu=cortex-m4 conflicts with -march=armv7-m switch
 // cpu.c -- memory access routines for the stm6502 virtual cpu.
 
cpu.c:28:26: warning: initialization makes pointer from integer without a cast [-Wint-conversion]
 unsigned char *rawmemory=0x20000000;
                          ^~~~~~~~~~
arm-none-eabi-gcc -O3 -g -std=gnu99 -fno-common -Wall -mcpu=cortex-m4 -mtune=cortex-m4 -mthumb -march=armv7-m \
	 -DSTM32F4 -I. -I/usr/local/arm-none-eabi/include \
	 -o init_6502.o -c init_6502.c
init_6502.c:1:0: warning: switch -mcpu=cortex-m4 conflicts with -march=armv7-m switch
 // init_6502.c -- initalize the memory-mapped hooks for the virtual cpu
 
init_6502.c: In function 'readport_checkstatus':
init_6502.c:80:36: fatal error: libopencm3/stm32/usart.h: No such file or directory
 #include <libopencm3/stm32/usart.h>
                                    ^
compilation terminated.
make: *** [init_6502.o] Error 1
```

Would need more investigation to determine how to build these projects. But before I do, a note on the emulated CPU speed:

> On [the STM32F4DISCOVERY] dev board, the CPU runs at 168MHz and the emulated speed of the 6502 is 18MHz.

BigEd asked: [emulator performance on embedded cpu](http://forum.6502.org/viewtopic.php?f=8&t=1603)

> WDC's 65c02 and 65816 are rated up to 14MHz. An FPGA T65 seems to promise about 40-50MHz. 

How fast is the emulated 6502 of the 30 MHz [tgl6502](https://github.com/thegaragelab/tgl6502)? The readme doesn't say, but given the blue pill runs at 72 MHz, and 168 MHz / 18 MHz = 9.3 for a6502, 72 MHz / 9.3 = 7.7 MHz, several times the [Simple6502](http://searle.hostei.com/grant/6502/Simple6502.html) and [NES](http://wiki.nesdev.com/w/index.php/CPU) speed of 1.8 MHz. Would not expect performance to be a problem for emulating 6502 on the STM32F103 blue pill.

Has anyone emulated 6502 on the STM32F103 blue pill yet? Searching the 6502.org forums, found only two posts: [1](http://forum.6502.org/viewtopic.php?f=1&t=1935&p=42681&hilit=stm32f103#p42681), [2](http://forum.6502.org/viewtopic.php?f=8&t=1603&p=42644&hilit=stm32f103#p42644), by stecdose in December of 2015. But there are different variants of the STM32F103, stecdose's has 256KB flash and 48KB RAM, more than the blue pill's 64/128KB and 20KB.

> I'm working on an emulator on an STM32F103 with 256kb Flash and 48kb RAM.

> It is written in C. The CPU ist just below 900 lines. It uses ~9,5kB of Flash and ~50bytes of RAM, exculding the emulated memory.

> I've assigned a few k to this emulator and assembled the skier and the random demo from 6502asm.org. Both are running at about ~1,03MHz without graphics.

and later, more details:

> At the moment I'm working on an emulator on a STM32F103, 72MHz, 48k RAM, 256k flash. My first approach of adopting some existing code gave me ~1.03MHz without graphic output.  I have a 3,2" 320x240px TFT connected and wrote some code to have a 32x32px graphics output. It is zoomed by 7 to see something on that small display. At the moment this is takes more than emulating the 6502. It slows the whole thing down to ~440kHz. The plan is to connect two SIDs to it as well and two Joysticks. Maybe the display has to move away for a SAA7182 scart rgb interface. Up to now there's not much working but of showing a test-screen that i have to bitbang into that chip, no idea how to interface this properly and fast with a STM32.

but [stecdose](http://forum.6502.org/memberlist.php?mode=viewprofile&u=2239) (Nils) last visited Sun Jan 03, 2016 11:47 am, hasn't posted since 2015, doesn't appear to have shared his code, so, let's forge ahead.

## Coding it up

Porting a6502 or tgl6502 could be a worthwhile endeavor, but I decided to make my own just for fun. I'll use [fake6502](http://rubbermallet.org/fake6502.c) as the processor core.

First install the [GNU ARM Embedded Toolchain](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm) (with [Homebrew](https://brew.sh), run `brew cask install gcc-arm-embedded`), then try compiling it but not linking:

```sh
fake6502 $ arm-none-eabi-gcc fake6502.c -c
fake6502 $ 
```

It compiles! Portable C, I like it. But try to link and we get these errors:

```sh
fake6502 $ arm-none-eabi-gcc fake6502.c
/usr/local/Caskroom/gcc-arm-embedded/6-2017-q2-update/gcc-arm-none-eabi-6-2017-q2-update/bin/../lib/gcc/arm-none-eabi/6.3.1/../../../../arm-none-eabi/lib/libc.a(lib_a-exit.o): In function `exit':
exit.c:(.text.exit+0x2c): undefined reference to `_exit'
/usr/local/Caskroom/gcc-arm-embedded/6-2017-q2-update/gcc-arm-none-eabi-6-2017-q2-update/bin/../lib/gcc/arm-none-eabi/6.3.1/../../../../arm-none-eabi/lib/crt0.o: In function `_start':
(.text+0xec): undefined reference to `main'
/var/folders/81/3xnrrjrn3hzd1sks0yvxwp1w0000gn/T//cca4VCDj.o: In function `push16':
fake6502.c:(.text+0x4c): undefined reference to `write6502'
fake6502.c:(.text+0x94): undefined reference to `write6502'
/var/folders/81/3xnrrjrn3hzd1sks0yvxwp1w0000gn/T//cca4VCDj.o: In function `push8':
fake6502.c:(.text+0x110): undefined reference to `write6502'
/var/folders/81/3xnrrjrn3hzd1sks0yvxwp1w0000gn/T//cca4VCDj.o: In function `pull16':
fake6502.c:(.text+0x16c): undefined reference to `read6502'
fake6502.c:(.text+0x1b4): undefined reference to `read6502'
/var/folders/81/3xnrrjrn3hzd1sks0yvxwp1w0000gn/T//cca4VCDj.o: In function `pull8':
fake6502.c:(.text+0x248): undefined reference to `read6502'
/var/folders/81/3xnrrjrn3hzd1sks0yvxwp1w0000gn/T//cca4VCDj.o: In function `reset6502':
fake6502.c:(.text+0x270): undefined reference to `read6502'
fake6502.c:(.text+0x284): undefined reference to `read6502'
/var/folders/81/3xnrrjrn3hzd1sks0yvxwp1w0000gn/T//cca4VCDj.o:fake6502.c:(.text+0x3c8): more undefined references to `read6502' follow
/var/folders/81/3xnrrjrn3hzd1sks0yvxwp1w0000gn/T//cca4VCDj.o: In function `putvalue':
fake6502.c:(.text+0xcd8): undefined reference to `write6502'
/var/folders/81/3xnrrjrn3hzd1sks0yvxwp1w0000gn/T//cca4VCDj.o: In function `brk':
fake6502.c:(.text+0x1788): undefined reference to `read6502'
fake6502.c:(.text+0x179c): undefined reference to `read6502'
/var/folders/81/3xnrrjrn3hzd1sks0yvxwp1w0000gn/T//cca4VCDj.o: In function `nmi6502':
fake6502.c:(.text+0x3a44): undefined reference to `read6502'
fake6502.c:(.text+0x3a58): undefined reference to `read6502'
/var/folders/81/3xnrrjrn3hzd1sks0yvxwp1w0000gn/T//cca4VCDj.o: In function `irq6502':
fake6502.c:(.text+0x3aec): undefined reference to `read6502'
/var/folders/81/3xnrrjrn3hzd1sks0yvxwp1w0000gn/T//cca4VCDj.o:fake6502.c:(.text+0x3b00): more undefined references to `read6502' follow
collect2: error: ld returned 1 exit status
fake6502 $ 
```

Actually reading the code, we are meant to define two functions to provide memory access (and a main() to run the CPU):

```c
/*****************************************************
 * Usage:                                            *
 *                                                   *
 * Fake6502 requires you to provide two external     *
 * functions:                                        *
 *                                                   *
 * uint8_t read6502(uint16_t address)                *
 * void write6502(uint16_t address, uint8_t value)   *
```

A simple main.c to start with, this compiles and runs on my host machine (just resets the processor and exits):

```c
#include <stdio.h>
#include <stdint.h>

void reset6502(void);

uint8_t memory[0x10000];

uint8_t read6502(uint16_t address)
{
    return memory[address];
}

void write6502(uint16_t address, uint8_t value)
{
    memory[address] = value;
}

int main()
{
    reset6502();
}
```

Compile and run:

```sh
fake6502 $ gcc main.c fake6502.c
fake6502 $ ./a.out 
fake6502 $ 
```

but compiling it for ARM embedded requires more configuration, of course:

```sh
fake6502 $ arm-none-eabi-gcc main.c fake6502.c
/usr/local/Caskroom/gcc-arm-embedded/6-2017-q2-update/gcc-arm-none-eabi-6-2017-q2-update/bin/../lib/gcc/arm-none-eabi/6.3.1/../../../../arm-none-eabi/lib/libc.a(lib_a-exit.o): In function `exit':
exit.c:(.text.exit+0x2c): undefined reference to `_exit'
collect2: error: ld returned 1 exit status
fake6502 $
```

At a crossroads, we could implement low-level hardware support by hand, but I went with the [libopencm3](http://libopencm3.org) library. To help interact with and test the device, it is useful to have a serial interface first.

### Adding USB CDC-ACM, the virtual serial port

As a base, started with code originally from the [Black Sphere Black Magic Probe](https://github.com/blacksphere/blackmagic), adopted into *[Pill Duck: Scriptable USB HID device using an STM32 blue pill, from mouse jigglers to rubber duckies](https://satoshinm.github.io/blog/171227_stm32hid_pill_duck_scriptable_usb_hid_device_using_an_stm32_blue_pill_from_mouse_jigglers_to_rubber_duckies.html)*. There is a lot of boilerplate code here, the details are not too relevant to this project, but it is important.

Flashed it using the JTAG/SWD and serial setup described in *[JTAG/SWD debugging via Black Magic Probe on an STM32 blue pill and blinking a LED using STM32CubeMX, libopencm3, and bare metal C](https://satoshinm.github.io/blog/171223_jtagswdpillblink_jtagswd_debugging_via_black_magic_probe_on_an_stm32_blue_pill_and_blinking_a_led_using_stm32cubemx_libopencm3_and_bare_metal_c.html)*. The serial port shows up as /dev/cu.usbmodem6502 on my system, and is reachable by screen:

```sh
src $ screen -L /dev/cu.usbmodem6502 
� � � � v
Pill 6502 version 83fed56-dirty> 
```

### Memory that fits internally

The full size of the CPU address space, 0x0000-0xffff, is of size 0x10000 = 64KB, too big for the blue pill's 20KB RAM. Even half, covering up 0x0000-0x7fff, would be too big, 32KB. This is how much [Simple6502](http://searle.hostei.com/grant/6502/Simple6502.html) has, but remember the [NES](http://wiki.nesdev.com/w/index.php/CPU_memory_map) only has 2KB internal RAM (0x0000-0x07ff). Since I'm not adding any external memory, settled on somewhere in the middle, 16 KB of RAM from 0x0000-0x3fff. This leaves 4 KB for the emulator itself. Declare the emulated RAM:

```c
uint8_t ram[0x4000];
```

Note that if the variable is declared larger than the available space in the blue pill, the linker will let you know, this is the linker failure with `uint8_t ram[0x10000];`:

```
  GIT     version.h
  CC      cdcacm.c
  CC      main.c
  LD      pill_6502.elf
/usr/local/Caskroom/gcc-arm-embedded/6-2017-q2-update/gcc-arm-none-eabi-6-2017-q2-update/bin/../lib/gcc/arm-none-eabi/6.3.1/../../../../arm-none-eabi/bin/ld: pill_6502.elf section `.bss' will not fit in region `ram'
/usr/local/Caskroom/gcc-arm-embedded/6-2017-q2-update/gcc-arm-none-eabi-6-2017-q2-update/bin/../lib/gcc/arm-none-eabi/6.3.1/../../../../arm-none-eabi/bin/ld: region `ram' overflowed by 47528 bytes
collect2: error: ld returned 1 exit status
make: *** [pill_6502.elf] Error 1
```

If other code added causes a similar error, we'll know the emulator RAM needs to be decreased, or the code that was added needs to be optimized to reduce memory usage.

### Execution

In main(), first call `reset6502()`. To execute each instruction, `step6502()` will be called in the systick handler, called by an interrupt periodically. This may be better setup differently but for now the periodic stepping will allow seeing execution more clearly. The LED will also be toggled:

```c
void sys_tick_handler(void)
{
        step6502();
        gpio_toggle(GPIOC, GPIO13);
}
```

To view how many instructions were executed, and how many clock cycles elapsed, added this serial command:

```c
        } else if (buf[0] == 't') {
                static char buf[64];
                snprintf(buf, sizeof(buf), "%ld ticks\r\n%ld instructions", clockticks6502, instructions);
                return buf;
```

by connecting to the serial port we can type 't' to see the instruction execution advances:

```
> v
Pill 6502 version 71c0b67-dirty
> t
100709 ticks
14387 instructions
> t
103810 ticks
14830 instructions
> t
106757 ticks
15251 instructions
```

Of course, it isn't executing anything useful yet. To do so, a ROM needs to be loaded.

### Microsoft ROM BASIC and memory map

[Grant's 6502 computer](http://searle.hostei.com/grant/6502/Simple6502.html) includes a link to osi_bas.zip, containing osi_bas.bin, a 16384 byte ROM file to be loaded at 0xc000-0xffff.

How to embed this binary file into the firmware? [balau82 Linking a binary blob with GCC](https://balau82.wordpress.com/2012/02/19/linking-a-binary-blob-with-gcc/) describes a technique using `objcopy -I binary`. But what output format (`-O` flag)? elf32-i386 is balau82's example, but arm-none-eabi-objcopy supports these targets: elf32-littlearm elf32-bigarm elf32-little elf32-big plugin srec symbolsrec verilog tekhex binary ihex. Do I have a little or big ARM? `file` on the compiled .elf binary only says:

```sh
pill_6502.elf: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, with debug_info, not stripped
```

[Quora: Is ARM big endian or little endian?](https://www.quora.com/Is-ARM-big-endian-or-little-endian) has an answer: "It’s little-endian on 99.9% of implementations, including all new ones". elf32-littlearm it is. `arm-none-eabi-objdump -f *.o` confirms:

```
fake6502.o:     file format elf32-littlearm
architecture: arm, flags 0x00000011:
HAS_RELOC, HAS_SYMS
start address 0x00000000
```

So here's the command to turn the raw binary ROM file into an ELF object file:

```sh
osi_bas $ arm-none-eabi-objcopy -I binary -O elf32-littlearm -B arm osi_bas.bin osi_bas.o
osi_bas $ ls -l osi_bas.bin osi_bas.o
-rw-r--r--@ 1 admin  staff  16384 Feb  4  2014 osi_bas.bin
-rw-r--r--  1 admin  staff  16828 Jan 12 23:33 osi_bas.o
osi_bas $ file osi_bas.bin osi_bas.o
osi_bas.bin: data
osi_bas.o:   ELF 32-bit LSB relocatable, ARM, version 1 (ARM), not stripped
```

To find out what symbols are exported, use `nm`:

```sh
osi_bas $ arm-none-eabi-nm osi_bas.o
00004000 D _binary_osi_bas_bin_end
00004000 A _binary_osi_bas_bin_size
00000000 D _binary_osi_bas_bin_start
```

Now this object file can be linked with our program, and the 6502 memory read function modified to access it at the proper emulated offsets (0xc000-0xffff).

But wait. osi_bas.zip also has the ROM in [Intel HEX](https://en.wikipedia.org/wiki/Intel_HEX) format, as `ROM.HEX`. This is an ASCII format, which plays nicer with version control, and objcopy happens to support it as `ihex`. I would expect this to work the same as `-I binary` with osi_bas.bin:

```sh
arm-none-eabi-objcopy -I ihex -O elf32-littlearm -B arm ROM.HEX osi_bas.o
```

but the object file it produces has no symbols, for some reason. Could ROM.HEX and osi_bas.bin differ? No, in fact objcopy can convert between .HEX and .bin, and the resulting file is identical to the .bin inside osi_bas.zip, and furthermore, objcopy'ing the converted .bin (from .HEX) produces an .o with symbols. What could be different? `file` reveals the object with missing symbols is `stripped`. Looks like the binary format is special, from the [objcopy documentation](https://sourceware.org/binutils/docs/binutils/objcopy.html):

> objcopy can be used to generate a raw binary file by using an output target of ‘binary’ (e.g., use -O binary). When objcopy generates a raw binary file, it will essentially produce a memory dump of the contents of the input object file. All symbols and relocation information will be discarded. The memory dump will start at the load address of the lowest section copied into the output file.

but shouldn't `-B bfdarch` create these symbols?

> -B bfdarch
> --binary-architecture=bfdarch
> Useful when transforming a architecture-less input file into an object file. In this case the output architecture can be set to bfdarch. This option will be ignored if the input file has a known bfdarch. You can access this binary data inside a program by referencing the special symbols that are created by the conversion process. These symbols are called \_binary_objfile_start, \_binary_objfile_end and \_binary_objfile_size. e.g. you can transform a picture file into an object file and then access it in your code using these symbols.

This flag was [added in 2001](https://www.cygwin.com/ml/binutils/2001-02/msg00381.html) by Stefan Geuken and only applies to the `binary` input target:

```c
   if (strcmp(input_target, "binary") == 0)
   {
     if (binary_architecture != NULL)
     {
       const bfd_arch_info_type* temp_arch_info = bfd_scan_arch(binary_architecture);
```

fine, I'll convert the .HEX to .bin then to .o. Add makefile rules, so `make` knows how to make `.o` from `.HEX`:

```make
ROM = ../osi_bas/ROM.HEX

OBJ += $(ROM:.HEX=.o)

%.o:   %.HEX
       @echo "  OBJCOPY $@.bin"
       $(Q)$(OBJCOPY) -I ihex -O binary $^ $@.bin
       @echo "  OBJCOPY $@"
       $(Q)$(OBJCOPY) -I binary -O elf32-littlearm -B arm $@.bin $@
```

Now to access the data programmatically. Take the address of these symbols:

```c
extern const char _binary____osi_bas_ROM_o_bin_end;
extern const char _binary____osi_bas_ROM_o_bin_size;
extern const char _binary____osi_bas_ROM_o_bin_start;
```

but I ran into a problem: [.bss](https://en.wikipedia.org/wiki/.bss) overflowed again: section `.bss' will not fit in region `ram', and region `ram' overflowed by 14984 bytes. This means the ROM is being loaded into RAM. `nm` shows "D" section, indicating data. Even if I declare as `const`, same problem. Hrmph.

An interesting opposite problem: [Fool ARM Linker to place a function code in some data memory](https://stackoverflow.com/questions/32054884/fool-arm-linker-to-place-a-function-code-in-some-data-memory). Add `__attribute__((section("rom")))`? Nope. [balau82](https://balau82.wordpress.com/2012/02/19/linking-a-binary-blob-with-gcc/) gives these hints:

> It is also possible to rename the symbols that are created by objcopy using the “--redefine-sym” option, and also put the data in a section with a different name and different flags, using “--rename-section“.

Aha, and back to the [objcopy documentation](https://sourceware.org/binutils/docs/binutils/objcopy.html), it describes exactly this use case:

> --rename-section oldname=newname[,flags]
> Rename a section from oldname to newname, optionally changing the section’s flags to flags in the process. This has the advantage over usng a linker script to perform the rename in that the output stays as an object file and does not become a linked executable.

> This option is particularly helpful when the input format is binary, since this will always create a section called .data. If for example, you wanted instead to create a section called .rodata containing binary data you could use the following command line to achieve it:

```
  objcopy -I binary -O <output_format> -B <architecture> \
   --rename-section .data=.rodata,alloc,load,readonly,data,contents \
   <input_binary_file> <output_object_file>
```
   
Update the makefile rule:

```make
%.o:    %.HEX
    @echo "  OBJCOPY $@.bin"
    $(Q)$(OBJCOPY) -I ihex -O binary $^ $@.bin
    @echo "  OBJCOPY $@"
    $(Q)$(OBJCOPY) -I binary -O elf32-littlearm -B arm --rename-section .data=.rodata,alloc,load,readonly,data,contents $@.bin $@
```

now it fits and read6502() can be updated:

```c
uint8_t read6502(uint16_t address) {
        // RAM
        if (address < sizeof(ram)) {
                return ram[address];
        }

        // ROM
        if (address >= 0xc000) {
                const uint8_t *rom = &_binary____osi_bas_ROM_o_bin_start;
                
                return rom[address - 0xc000];
        }

        return 0xff;
}

void write6502(uint16_t address, uint8_t value) {
        if (address < sizeof(ram)) {
                ram[address] = value;
        }
}
```

Make these changes, then boot up. The CPU seems to be running, loading the reset vector pointer from 0xfffc and 0xfffd, which in osi_bas is 0xff00, the location of the serial routines. From osi_bas.s:

```asm
segment "IOHANDLER"
.org $FF00
Reset:
    LDX     #STACK_TOP
    TXS

    LDA     #$95        ; Set ACIA baud rate, word size and Rx interrupt (to control RTS)
    STA ACIAControl

; Display startup message
    LDY #0
ShowStartMsg:
    LDA StartupMessage,Y
    BEQ WaitForKeypress
    JSR MONCOUT
    INY
    BNE ShowStartMsg

; Wait for a cold/warm start selection
WaitForKeypress:
    JSR MONRDKEY
    BCC WaitForKeypress

    AND #$DF            ; Make upper case
    CMP #'W'            ; compare with [W]arm start
    BEQ WarmStart

    CMP #'C'            ; compare with [C]old start
    BNE Reset

    JMP COLD_START  ; BASIC cold start

WarmStart:
    JMP RESTART     ; BASIC warm start
```

## 6850 ACIA emulation

Grant Searle's modifications to osi_bas.s interface with the ACIA (Asynchronous Communications Interface Adapter) chip, memory-mapped starting at 0xa000:

```asm
; STARTUP AND SERIAL I/O ROUTINES ===========================================================
; BY G. SEARLE 2013 =========================================================================
ACIA := $A000
ACIAControl := ACIA+0
ACIAStatus := ACIA+0
ACIAData := ACIA+1
```

What is this chip? Found this documentation on [cpcwiki: 6850 ACIA chip](http://www.cpcwiki.eu/index.php/6850_ACIA_chip). The 6850 is an important piece of 6502 computing history, and is one of the questions on the [6502.org forums](http://forum.6502.org/ucp.php?mode=register) registration. The [Motorola Semiconductors MC6850 data sheet](http://www.cpcwiki.eu/imgs/3/3f/MC6850.pdf) has all the details. Block diagram:

![MC6850 block diagram](https://user-images.githubusercontent.com/26856618/34904452-2fa84aa6-f7fb-11e7-8362-923077acff08.png)

summarizing the registers:

| Simple6502 address | MC6850 register |
| 0xa000 | ACIAControl |
| 0xa001 | ACIAStatus |
| 0xa002 | ACIAData |

ACIAControl is only set on startup to `#$95`, binary 10010101, which decodes as (left to right): 

* 01: counter divide select bits: divide clock by 16
* 101: word select bits: 8 bits + 1 stop bit
* 00: transmitter control bits: /RTS = low, transmitting interrupt disabled
* 1: receive interrupt enable bit: receive data register full, overrun, or a low-to-high transition on Data Carrier Detect (/DCD)

osi_bas.s reads the ACIStatus register in MONCOUT and MONRDKEY, checking bits 1 (transmit data register empty, TDRE) and 0 (receive data register full, RDRF), respectively. This implements flow control, allowing the 6502 to wait until the 6850 transmitted the byte in the ACAIData register before writing the next byte, and likewise for receiving data.

First implement output. When 0xa000 is read, return 2 (0b10), the TDRE bit is set indicating the program can write data to 0xa001, and when it does write there, send the byte over USB CDC-ACM. We see output from the ROM BASIC:

```
> r
reset
> Cold [C] or warm [W] start?

```

To make it easier to use, added a USB reset callback to reset and resume the 6502 processor, so it starts when the USB cable is attached to the host computer. Now to add user input.

### Input from the virtual serial port

When the user types something, the USB callback `usbuart_usb_out_cb()` is called to process the CDC-ACM packet data. Buffer this data first-in first-out, returning the bytes when read from the ACIAData register (0xa001) and incrementing an offset. The RDRF flag in the ACIAStatus register (0xa000) should be set when there is unread data pending. If it works, we should be able to type 'C' at the prompt and get prompted for the memory size:

```
Cold [C] or warm [W] start?
C
MEMORY SIZE? 
16384
```

then it performs a memory test. After waiting some time, received errors, which may be due to my other incidental typing:

```
?SN ERROR
OK
t [

?SN ERROR
OK
p
```

An obvious problem occurs. To help bootstrap the system I implemented a few simple one-character commands over serial, but these keystrokes interfere with the user input sent to the 6502 (via the 6850). Couple possible solutions: add an escaping mechanism, SSH for example uses [EscapeChar ~](https://man.openbsd.org/ssh_config#EscapeChar), preceded by a newline, followed by another character. But ASCII already has [control characters](https://en.wikipedia.org/wiki/ASCII#ASCII_control_characters), and they can be sent on your PC by pressing the Control key in conjunction with an ASCII character with the same lower bits. Could even go further and use [ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code). Changed the commands to control characters:

```c
char *process_serial_command(char b) {
        if (b == '\x16') { // ^V
                return "Pill 6502 version " FIRMWARE_VERSION;
        } else if (b == '\x10') { // ^P
                paused = !paused;
                return paused ? "paused" : "resumed";
        } else if (b == '\x12') { // ^R
                reset6502();
                paused = false;
                return "reset";
        } else if (b == '\x14') { // T 
                static char buf[64];
                snprintf(buf, sizeof(buf), "%ld ticks\r\n%ld instructions", clockticks6502, instructions);
                return buf; 
        } else if (b == '\x07') { // G
                return "^V=version ^R=reset ^P=pause ^T=ticks ^G=help";
        }
                        
        return NULL;    
}
```

If a command isn't handled here, it is sent to the 6850 ACIA input buffer. This isn't a perfect solution since it prevents these characters from being transmitted, so a kind of "monitor mode" or escaping solution could be useful, but this is good enough for now. Finally able to get past the startup configuration:

```
C� � old [C] or warm [W] start?

4963 ticks
1448 instructions
Cold [C] or warm [W] start?

42392 ticks
12158 instructions
Cold [C] or warm [W] start?
c
MEMORY SIZE? 10102424

TERMINAL WIDTH? 808
0

 511 BYTES FREE

OSI 6502 BASIC VERSION 1.0 REV 3.2
COPYRIGHT 1977 BY MICROSOFT CO.

OK
```

Another problem is now immediately apparent: local echo. To see what I was typing, I made the USB CDC-ACM port echo back what it received. But osi_bas has its own echo, leading to duplicate characters. This is straightforward to fix, merely removing the extra code, but it was useful to see the serial port is responding (as opposed to the 6502 program), so I added it as an optional feature, toggled with ^E. Off by default. Now to check if the STM32 blue pill microcontroller is responding over the serial port, use ^G or another command, or ^E then type something. Normally we would want local echo to be off so the ROM BASIC responds instead, however. Fixing the echo:

```
C� � old [C] or warm [W] start?

MEMORY SIZE? 16384
TERMINAL WIDTH? 80

 15871 BYTES FREE

OSI 6502 BASIC VERSION 1.0 REV 3.2
COPYRIGHT 1977 BY MICROSOFT CO.

OK
```

There are still "�" replacement characters after the first byte, not clear what these bytes are, and they are not visible when connecting to the serial port with pyserial. Maybe some control characters used by the terminal. Anyways, time to write BASIC.

## Writing BASIC code

Start with the classic [Hello, world!](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program), as is tradition:

```basic
10 PRINT "Hello, world!"
RUN

Hello, world!

OK

```

It works! Next how about [Grant Searle's](http://searle.hostei.com/grant/6502/Simple6502.html) example program, an ASCII snake:

```basic
OK
10 FOR A=0 TO 6.2 STEP 0.2
20 PRINT TAB(40+SIN(A)*20);"*"
30 NEXT A
RUN
```

It runs, slowly:

```
                                        *
                                           *
                                               *
                                                   *
                                                      *
                                                        *
                                                          *
                                                           *
                                                           *
                                                           *
                                                          *
                                                        *
                                                     *
                                                  *
                                              *
                                          *
                                      *
                                  *
                               *
                           *
                        *
                      *
                    *
                    *
                    *
                    *
                      *
                        *
                           *
                              *
                                  *
                                      *

OK
```

To check how much memory is available:

```basic
OK
PRINT FRE(0)
 15834 

OK
```

indicating most of the emulated 16KB RAM is available.

## Optimizations and enhancements

This is only a start, there are a lot of possible improvements that could be made to pill_6502. Most of all, tightening up the performance. Executing 6502 instructions on a systick timer every 1 millisecond gives about a 1 kilohertz clock rate, slightly faster for instructions taking more than one cycle, this certainly is not the best way to do it. The systick interrupt has excessive overhead as well. fake6502 supports `exec6502(uint32_t tickcount)` to execute multiple instructions in one go, and furthermore, it would be nice to have cycle-accurate emulation like on good NES emulators: check `clockticks6502` and delay as needed compensating to emulate accurately.

Emulating more hardware is another possible area of expansion, but even cooler, in my opinion, would be to expose the emulated 6502 CPU's [address and data bus](https://en.wikipedia.org/wiki/MOS_Technology_6502#Technical_description) as well as the interrupt pins IRQ and NMI, and SO overflow status bit pin. Interface with real hardware, as part of a larger retrocomputing system. Speed it up, then use it as a replacement for a real 6502 CPU + 62128 16KB RAM + ROM + 6850 ACIA?

For comparison, Sean Miller's [build of the Simple6502](https://www.youtube.com/watch?v=vr8ROiYs8AQ&t=3m36s) running ROM BASIC: 

![Raising Awesome Simple6502](https://user-images.githubusercontent.com/26856618/34911057-bb272608-f876-11e7-8249-f94c7d1185a1.png)

versus the fully integrated blue pill loaded with pill_6502 connected to the computer over USB for virtual serial:

![Blue pill hardware](https://user-images.githubusercontent.com/26856618/34911084-21b6cc52-f877-11e7-9706-94790db3f704.png)

And comparing the pinouts of the [STM32F103 blue pill](http://wiki.stm32duino.com/index.php?title=Blue_Pill) and [MOS Technology 6502](https://en.wikipedia.org/wiki/MOS_Technology_6502#Technical_description):

![6502 vs blue pill pinout](https://user-images.githubusercontent.com/26856618/34910807-7ae880fa-f871-11e7-996c-24e31a616b10.png)

If you want to make your own, or help contribute, check out the pill_6502 source code and binaries at: [https://github.com/satoshinm/pill_6502](https://github.com/satoshinm/pill_6502)

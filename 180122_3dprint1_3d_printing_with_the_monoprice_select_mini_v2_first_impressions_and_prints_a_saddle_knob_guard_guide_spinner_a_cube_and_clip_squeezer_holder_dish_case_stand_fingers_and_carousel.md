# 3D printing with the Monoprice Select Mini v2: first impressions and prints: a saddle, knob,  guard, guide, spinner; a cube and clip; squeezer, holder, dish; case, stand, fingers, and carousel

by snm, January 22nd, 2018

* auto-gen TOC:
{:toc}

## Monoprice Select Mini v2

![Monoprice Select Mini 3D Printer v2 from user's manual](https://user-images.githubusercontent.com/26856618/35086404-668eb58e-fbe1-11e7-8769-e8565a1dd890.png)

* [Monoprice 115365 Select Mini 3D Printer with Heated Build Plate, Includes Micro SD Card and Sample PLA Filament](https://www.amazon.com/gp/product/B01FL49VZE), $220

Why the Monoprice Select Mini v2? Many opinions on [/r/3Dprinting](https://www.reddit.com/r/3Dprinting/) and other forums recommend it as a great first printer. Watched the following reviews, a good balance of the pros and cons of this printer:

* [Now, Everyone Can Afford 3D Printing (Monoprice Select Mini Review), by Mike and Lauren](https://www.youtube.com/watch?v=ispolAHB4jA)
* [Monoprice Select Mini V2 Unboxing & Review: PLA, SemiFlex, Cheetah, Tech-G, by Lindy Design Lab](https://www.youtube.com/watch?v=oFKU9N0nAPo)
* [Best Beginner 3d Printer 2017 | Monoprice Select Mini, by ModBot](https://www.youtube.com/watch?v=Aoss9SKhOAM)

Not perfect, but it has a lot going for it, including coming preassembled out-of-the-box. You can build one yourself or from a kit for cheaper but this adds more variables on what can go wrong. Or you can pay more for a larger bed but 120x120x120 mm^3 is enough for plenty of useful objects. Also has an active community:

* [/r/MPSelectMiniOwners](https://www.reddit.com/r/MPSelectMiniOwners/)
* [MP Select Mini / ProFab Mini / Malyan M200 Wiki](https://www.mpselectmini.com/)

What is the electronics hardware behind it? [Hackaday: Reverse-engineering the Monoprice printer](https://hackaday.com/2017/06/20/reverse-engineering-the-monoprice-printer/) links to Robin Reiter's [Electronics Reverse Engineering Walkthrough - Hacking the Monoprice Select Mini 3D Printer](https://www.youtube.com/watch?v=T-o-ibGUEoA), which reveals it is powered by two familiar microprocessors: the STM32F103C8T6 (well-aquainted with, see my *[STM32 Blue Pill ARM development board first look: from Arduino to bare metal programming](https://satoshinm.github.io/blog/171212_stm32_blue_pill_arm_development_board_first_look_bare_metal_programming.html)* post, among others) for driving the motors/heaters, and the ESP8266 (a predecessor to the ESP32, which I also covered in *[ESP32-based wireless ticker display for Minecraft server activity: pushing up against the fourth wall](https://satoshinm.github.io/blog/180107_esp32ticker_esp32_based_wireless_ticker_display_for_minecraft_server_activity_pushing_up_against_the_fourth_wall.html)*), for the Wi-Fi and display interface. Robin Reiter was able to backup the firmware with esp_tool, then flash his own to control the LCD. Lots of interesting possibilities for hacking here. But for now I'm using the stock firmware, as it is most complete.

## Setup

Followed the instructions in the [User's Manual](https://downloads.monoprice.com/files/manuals/15365_Manual_170509.pdf) to configure the printer, fairly straightforward: slide in the spool rack, level the bed with a piece of paper, etc. But I did have some difficulty getting the filament extruding successfully. You pull back the spring then push it through until it hits the hot end. Then heat up the hot end to the recommended temperature, about 200 degrees. Tell it to move the filament, extrude it outwards. The motor tried to push, but nothing came out of the end. I tried pushing too, to no avail. Removed the filament, put it back, tried higher temperatures, cutting a sharper edge, nothing.

Finally, I moved the printer head so the PTFE tube to it was nearly level, instead of at an awkward angle. Then forcefully pushed the filament through, an audible *SNAP* occurred, and it began extruding! The filament melting at last:

![First filament melted](https://user-images.githubusercontent.com/26856618/35134171-82759684-fc89-11e7-8e66-816bb184069c.png)

Not sure if there was a seal or something over the hot end but this is all water under the bridge now. Ready to do the first print.

### Test print: a cat

Monoprice includes a micro SD card with a sample model, cat.gcode. Start printing it, then observe the first layer. To my untrained eye, it appears acceptable, the printer bed not to close nor too far. Left it continue printing. The layer of the cat slowly builds up, awesome. You can see it come together:

![Half a cat](https://user-images.githubusercontent.com/26856618/35134195-a7afc348-fc89-11e7-8eac-1a8d60594ef5.png)

Takes quite a long time, at 1.0x speed, but this isn't the fastest printer. For the first print I watched it intently, but for future prints once the printer is known to be working well, it should be able to be left on its own. As for the sound, you can hear it making noise, sounds like, well, a printer. Patience is required. We're talking hours. Clocked in at 2 hours 52 minutes according to the display:

![Print time](https://user-images.githubusercontent.com/26856618/35134203-b7f66a54-fc89-11e7-849b-e924befbd14a.png)

After I heard the squeaking die down, checked on it and the freshly printed cat was complete, waving up at me:

![Freshly printed cat](https://user-images.githubusercontent.com/26856618/35134219-da67f936-fc89-11e7-8a50-5716bcd7eb3e.png)

Required some amount of force to remove the print from the printing bed. The included squeegee was barely useful, instead, I tried to grab it from the top and carefully apply force, but the printer head kept getting in the way, clearly I have a lot to learn, and/or better tools to acquire. But eventually I was able to get it off:

![Removed cat](https://user-images.githubusercontent.com/26856618/35134299-40b5409a-fc8a-11e7-80c2-e912f5f65ba4.png)

The object is printed on a "raft", which is meant to be removed. Broke it mostly off:

![Cat and raft](https://user-images.githubusercontent.com/26856618/35134311-5c13b9d4-fc8a-11e7-8fe1-b81d5c9ed349.png)

How much filament was used? Almost all of the included sample. Supposedly v1 came with not enough filament for anything, but I can confirm v2 has plenty to print the sample cat. Measured about 63 inches remaining (46 inches plus about 15 inches I cut off while trying to extrude the filament). Would be cool if the printer measured the amount of filament extruded per print so you could calculate the cost, but I guess you're on your own here. Anyways, time to move onto a real print, with a real spool of filament.

### Filament

You need to also buy filament, I picked this one:

* [AIO Robotics AIOWHITE PLA 3D Printer Filament, 0.5 kg Spool, Dimensional Accuracy +/- 0.02 mm, 1.75 mm, White](https://www.amazon.com/gp/product/B01HYYPMAM), $11.99

What filament? The Monoprice Select Mini comes with a small amount of sample filament, but not enough for most anything. It takes a 1.75 mm diameter filament, and officially supports either PLA ([polylactic acid](https://en.wikipedia.org/wiki/Polylactic_acid), a biodegradable thermoplastic), or ABS ([acrylonitrile butadiene styrene](https://en.wikipedia.org/wiki/Acrylonitrile_butadiene_styrene), commonly injection molded). So should I start with PLA or ABS? Maker's Muse says [Should you 3D Print in ABS? Probably not.](https://www.youtube.com/watch?v=C-HhEnn0fuw) so I went with PLA. There are more exotic materials but PLA is supposedly the easiest to work with and most forgiving.

There are plenty of filament manufacturers, too many to choose from, so I semi-randomly settled on [AIO Robotics](http://www.zeus.aiorobotics.com). A nice feature of their spools is they are half a kilogram, instead of the usual kilogram, and sealed in a reusable bag, so they can be stored longer and you can buy more of different colors instead of getting stuck with more than you need. Seems like a reasonably good selection, but if there's one thing you can go wild in trying all the possibilities with in 3D printing, it is the filament. After purchasing it arrived quickly a day later after the printer, in a box with this label:

![AIO Robotics PLA filament label](https://user-images.githubusercontent.com/26856618/35134608-3e53de36-fc8c-11e7-87ae-830d8ba651e7.png)

PLA Filament Diamter ("diameter"): 1.75 mm ± 0.02 mm, ROHS approved, printing temperature: 195ºC-230ºC, color: white. Rip open the vacuumed-sealed packaging, cut the end off the filament, thread through the extruder, extrude. Filament comes out. Very white, much whiter than the Monoprice sample filament which is "clear/natural", uncolored. Finished in 2 hours 52 minutes, exactly the same as with the sample filament, no surprise since it was the same G-code. Looks much nicer, the natural/clear sample filament on the left, and AIO Robotics white on the right (with the raft):

![Cat printed with AIO Robitics white PLA](https://user-images.githubusercontent.com/26856618/35144709-2ee00c3a-fcba-11e7-83f1-2124675af0c6.png)

### Slicing to G-code

Thingiverse has tons of objects to download, but how to get the models to the printer? Could [sneakernet](https://en.wikipedia.org/wiki/Sneakernet) it through the micro SD card, but then we need a micro SD card reader, and sufficient dexterity to unplug the SD card and replug it, meh. Or USB, but my printer is nowhere near my compupter. Wireless is the solution. The Monoprice Select Mini supports Wi-Fi, followed the instructions in the manual to associate to my network, then accessed its web interface through an ordinary web browser. An simple and uncluttered web interface:

![Web interface](https://user-images.githubusercontent.com/26856618/35134921-8cecf85a-fc8e-11e7-88da-00de852040d7.png)

The printer wants [G-code](https://en.wikipedia.org/wiki/G-code), yet the downloaded files from Thingiverse are [.stl, stereolithography](https://en.wikipedia.org/wiki/STL_(file_format)). You need to "slice" the model into layers for the 3D printer. There are others, but [Ultimaker Cura](https://ultimaker.com/en/products/ultimaker-cura-software) is one such [slicer](https://www.mpselectmini.com/slicers/start). Downloaded and installed, but my printer wasn't listed in Add Printer, the closest was Malyan M180, but the MP Select Mini is a rebadged Malyan M200. There is a profile on the MP Select Wiki, here: [https://www.mpselectmini.com/slicers/cura](https://www.mpselectmini.com/slicers/cura), except it is for Cura 3.0.4 not Cura 3.1. Fine, download the old 3.0.4 version they linked, install the profile, then Monoprice Select Mini can be selected in the Other list. Accept all the defaults. Now the .stl file (vhtoolholder.stl) can be dragged to Cura, and the model shows up.

The model designer recommends "Print this at a small layer height of around 0.1mm", but Cura's layer height selection has jumbled overlapping text I can't read:

![Cura](https://user-images.githubusercontent.com/26856618/35135504-8998443a-fc92-11e7-86ef-0559a1d9dcb0.png)

so instead of using the slider, choose Custom and enter 0.1 mm in the Layer Height text box. Cura calculates the time, length and mass of the material to be used. Click Save to File, this creates the .gcode file, then drag a drop this 2.5 MB file to the printer's web interface. The printer display shows "Downloading..." and the web interface shows a progress bar, surprisingly slow to upload. It appears to write a new file to the SD card, cache.gc. Selected it under the Print menu on the device. The printer began printing...

...then disaster struck. The nozzle hit the bed and tore across it:

![Bed damage](https://user-images.githubusercontent.com/26856618/35136546-11762f24-fc99-11e7-8ace-67aedebcdedd.png)

I guess you need to recalibrate the bed level after each print... it was perfect before the cat. Now it is way off, carefully relevel using a sheet of paper, then print. But it failed again, in a different manner: the strings of filament would be wiped away as the bed moved, not sticking. No raft. Cura has a "Build Plate Adhension" option, check it, and the Advanced Settings lets you choose from a skirt, brim, raft, or none. 3D printing isn't yet to the point of "point and click", may require fiddling, it appears.

The sample cat.gcode on the SD card lays down a nice even raft, but when I use Cura to create G-code, it lays down too thin and the plastic becomes stringy and doesn't adhere. There is undoubtedly some settings that ought to be tweaked to fix this. But coupled with the overlapping unreadable Layer Height text in Cura, this pushed me to try another slicer: [Slic3r](http://slic3r.org). Downloaded and launched Slic3r version 1.2.9 stable, but it hung on the configuration assistant screen. Force-quit and relaunch, no hang this time. MP Select Wiki has a configuration bundle here: [https://mpselectmini.com/slicers/slic3r](https://mpselectmini.com/slicers/slic3r), but no instructions to install. Guessing, I replaced the contents of `~/Library/Application\ Support/Slic3r/slic3r.ini` with `slic3r_config_bundle_mp_select_mini.ini` and relaunched. Slic3r notified me:

> Bed Shape
> Hello! In this version a new Bed Shape option was added. If the bed coordinates in the plater preview screen look wrong, go to Print Settings and click the "Set" button next to "Bed Shape".

the rectangular bed looks fine, though. Slic3r > Filament Settings > Filament > Diameter defaulted to 3 mm, that's wrong, this printer and filament is 1.75 mm. Changed that and set the extruder temperature to 200 celsius and bed to 50. Generated the G-code, uploaded, printed, but still, too stringy. And the sample cat.gcode is perfect.

Taking a step back, what if we try to recreate the cat.gcode from an .stl? To confirm our settings are correct, and reconcile any differences. Searching for cat on Thingiverse finds this design, looks just like it:

* [maneki-neko -money cat-](https://www.thingiverse.com/thing:923108) by bs3, published Jul 13, 2015

Another supported slicer is [Repetier-Host](https://www.repetier.com). Repetier actually includes a bundled version of Slic3r. Loaded the money_cat.stl, sliced, but there was no raft unlike cat.gcode on the included SD card. The first layer is very important.

Groguards Creations published [My Cura setup for the Monoprice Maker Select V2](https://www.youtube.com/watch?v=IIHelSpeL7s). Change the print speed to 40 mm/s (from 50 mm/s). Shell wall thickness is a multiple of the nozzle size 0.40 mm, use three walls, 0.4 mm * 3 = 1.2 mm. Make these changes.

For the money cat, Cura estimates a 3 hour 2 minute print time, 6.31 m / ~19 g filament. Try with the same Build Adhension Type as the sample, a raft:

![Cura money cat](https://user-images.githubusercontent.com/26856618/35144795-7cbc643a-fcba-11e7-94d2-7f7da518977a.png)

Generate the G-code (this one is big, 8.4 MB) and upload. Took too long (why is Wi-Fi so slow? SD card is nearly instanteous) and I'm sick of cats, would like to try a real object soon. But my slicing settings seem to be good, the object is larger but now the extruded filament it lays down is thicker, although I didn't let the print complete, it stuck like a champ. Keep these settings! Now for *real* prints, no more cats.

Try a different object, slice it. But it failed, repeatedly, the location of the failure gives some insight to the cause of the failure:

![Failed print](https://user-images.githubusercontent.com/26856618/35178480-750b9d1c-fd3e-11e7-8963-154326b51809.png)

relevel the bed especially on the rear left-hand corner. Leveling is a drag, but this budget printer lacks [auto-leveling](http://www.instructables.com/id/Enable-Auto-Leveling-for-your-3D-Printer-Marlin-Fi/). After releveling manually, the print adheres in this corner:

![Fixed print leveling](https://user-images.githubusercontent.com/26856618/35178542-6b3cc9ea-fd3f-11e7-943b-f06f583389ab.png)

This print was successful, but I had more problems with printing in that area, another print first layer:

![First layer uneven print](https://user-images.githubusercontent.com/26856618/35179899-d19c3234-fd58-11e7-8a56-9dd22a145029.png)

This looked really bad, but the object actually turned out fine, luckily enough. Nonetheless, definitely something to fix.

Nearly ready to print a real object. But first, a final note about this filament. At the $11.99/0.5 kg price I got for the half spool, along with the estimated grams from Cura, we can estimate the cost of the filament for a print. I'll be estimating cost and providing these stats for each print to give an idea of how cost effective 3D printing is, using this filament at $0.02398/g. Of course, it is only an estimate, and I'm not measuring the actual length or mass extruded. Your mileage may vary.

This is where I started printing, skip to [#Printing printer parts](Printing printer parts) if you want to see the prints, or keep reading in this section for other printer enhancements.

## Printing printer parts

After trying the first print, there are some clear areas of improvement that can be made to this printer. What better first things to print than improvements to the printer itself? 

### Tool rack

<table>
  <tr>
    <th>Model</th>
    <td><a href="https://www.thingiverse.com/thing:1817172">Tool Saddle for MP Select Mini</a> by ArmArakel, published Oct 10, 2016</td>
  </tr>
  <tr>
    <th>Estimated print time</th>
    <td>2:11 (0:38 + 1:33)</td>
  </tr>
  <tr>
    <th>Actual print time</th>
    <td>2:12 (0:42 + 1:30), 1 minute slower (1%)</td>
  <tr>
    <th>Estimated filament length</th>
    <td>3.90m (1.14m + 2.76m)</td>
  </tr>
  <tr>
    <th>Estimated filament mass</th>
    <td>11g (3g + 8g)</td>
  </tr>
  <tr>
    <th>Filament used</th>
    <td>AIO Robotics AIOWHITE PLA, $11.99/500 g</td>
  </tr>
  <tr>
    <th>Estimated cost</th>
    <td>$0.26</td>
  </tr>
</table>

Simple, yet functional. A handy object to hold various tools from the printer. There is a large selection of these objects on the MP Select Mini wiki: [MPSM - Tool Holders](https://www.mpselectmini.com/thingiverse/tool-holders).

How about this simpler "tool saddle", which screws onto the top of the printer.

It holds the SD card box, hex key, and a standard SD card or adapter, components used with this printer, so you can store them here instead of getting lost elsewhere. Also considered the [MP Select Mini Starter's Tool Holder](https://www.thingiverse.com/thing:2621297) by stijnvanderheyden, published Nov 3, 2017, but it is meant to attach to the printer with double-sided tape, which I don't have. With a brim, for the tool saddle hanger.stl, Cura calculates only a 38 minute print time, 1.14 meters / ~3 grams. Estimating from the cost of the $11.99/0.5 kg spool, this comes to $0.07194 of filament. Not bad, 7¢ for this small part. 

Slice it, only 0.8 MB G-Code file, upload to the printer. Print. With this one, unlike the cat, the bed leveling on the corners proved much more crucial. The (disposible) brim kind of messed up, but the part came out alright:

![Holder hanger printed](https://user-images.githubusercontent.com/26856618/35179003-302719da-fd46-11e7-8477-30e7a921c9d8.png)

Next is the holder, use the second file sdcardholderholderPrototype2.STL since it includes an extra slot for a full-sized SD card (or SD-to-microSD adapter, sold separately). 1:33, 2.54m, ~7g. Cura imports it facing up:

![Holder default orientation](https://user-images.githubusercontent.com/26856618/35178738-0200b434-fd42-11e7-887d-8573aa769984.png)

but I imagine it will be easier to print lying down flat. Rotate 90 degrees, then it takes 1:23, 2.76m, ~8g. 

![Holder model rotated](https://user-images.githubusercontent.com/26856618/35178767-60e71a38-fd42-11e7-85b7-8433b4bf0313.png)

I possibly could have printed both models the same time, rearranging on the print area bed. But I printed them separately, separation of concerns. The holder finished in 1:30. Both parts total, estimated cost 26¢. The holder finished printing with an ugly brim:

![Holder printed](https://user-images.githubusercontent.com/26856618/35179908-fa82242e-fd58-11e7-8d56-437777d37e65.png)

The backs of these parts are not smooth:

![Rough backs](https://user-images.githubusercontent.com/26856618/35179911-11e942dc-fd59-11e7-8a71-5a3565198529.png)

a good reason to switch to a [glass bed](https://mpselectmini.com/glass_bed), alas, don't have one of those yet. The top side looks better:

![Holder parts top](https://user-images.githubusercontent.com/26856618/35179917-3589eafc-fd59-11e7-8f39-54a146dc0636.png)

Snap the two pieces together, fits snugly. Push the hex wrench into the sheath on the side, it also fits snugly. Then I pushed in my Kingston microSD-to-SD adapter, but it fit *too* tightly, had to use a screwdriver to pry it out; not going to push it in this far anymore. The included micro SD card case fit in alright. The screw holes were too small, and had some junk inside them, so I drilled them out with a 7/64 bit and they screwed in easily. Here is where the holder is meant to screw in, these two screws on the back of the printer:

![Back of printer](https://user-images.githubusercontent.com/26856618/35179923-599ba002-fd59-11e7-9c80-d0b5169cea84.png)

Screwed in with the hex key, microSD card, and SD-to-microSD adapter:

![Back of printer with printed tool saddle holder](https://user-images.githubusercontent.com/26856618/35179929-7c31dd20-fd59-11e7-86c1-499a4283e35a.png)

I like it, now the tools are tucked away yet readily available on the printer when I need them, already proving useful. Here is another shot:

![Tool saddle](https://user-images.githubusercontent.com/26856618/35179935-9fb02cac-fd59-11e7-82fa-f0826afd68f6.png)

### Knob

<table>
  <tr>
    <th>Model</th>
    <td><a href="https://www.thingiverse.com/thing:2245341">MPSM Spinner Knob</a> by VA3TNE, published Apr 12, 2017</td>
  </tr>
  <tr>
    <th>Estimated print time</th>
    <td>1:05</td>
  </tr>
  <tr>
    <th>Actual print time</th>
    <td>1:15, 10 minutes slower (15%)</td>
  <tr>
    <th>Estimated filament length</th>
    <td>2.01m</td>
  </tr>
  <tr>
    <th>Estimated filament mass</th>
    <td>5g</td>
  </tr>
  <tr>
    <th>Filament used</th>
    <td>AIO Robotics AIOWHITE PLA, $11.99/500 g</td>
  </tr>
  <tr>
    <th>Estimated cost</th>
    <td>$0.12</td>
  </tr>
</table>


The dial knob on the rotary encoder is slippy, worthy of replacement. Thingiverse collection [MPSM - Dial / Knob](https://www.mpselectmini.com/thingiverse/dial), many to choose from, picked one that looked cool, the designer describes as having "a Low Profile, and a nice Knurled circumference, AND a deep dimple". Loaded it into Cura, this time I moved it to the right of the bed to see if it prints better there in my printer (since the left side is damaged or unlevel or warped), and added a brim:

![Knob model moved](https://user-images.githubusercontent.com/26856618/35180092-0ceef8b8-fd5d-11e7-8cc3-2f9af39d995e.png)

First layer, a pretty good circle:

![Circle](https://user-images.githubusercontent.com/26856618/35180221-7e1388ee-fd60-11e7-8e7e-db3c464246dc.png)

This smaller part felt like it printed faster. Progress:

![Knob printing](https://user-images.githubusercontent.com/26856618/35180223-91c800ae-fd60-11e7-84c9-ea72eb9c7a69.png)

Done:

![Knob printed](https://user-images.githubusercontent.com/26856618/35180545-8ec1562e-fd67-11e7-8f24-0d95bf329f3f.png)

The designer recommended 30% infill, but I neglected to change it from 0%. Nonetheless, it seems to be sturdy. Here is the stock factory knob we'll be replacing:

![Factory knob](https://user-images.githubusercontent.com/26856618/35180727-5d804da6-fd6a-11e7-85ef-77078a6cac87.png)

Pop it off (pull forcefully, I used pliers to get a better grip, but some people suggest dental floss):

![Factory knob removed](https://user-images.githubusercontent.com/26856618/35180733-746c2440-fd6a-11e7-91c1-b7aae7745c3a.png)

Push on the new knob:

![Printed knob installed](https://user-images.githubusercontent.com/26856618/35180738-95497a78-fd6a-11e7-887d-af26eaaf6d81.png)

Since the front of the knob was the bottom of the print, and I was using the stock build surface, the front of the knob has a rough surface. Not the best looking knob in the world. Maybe could be printed upside-down, or on a better surface, or post-print finished. But despite appearances, this knob works quite well, I already like it better than the old knob. It is easier to grip with the serrated edges and the bowling-ball holes. You can also still see the LEDs behind it, and press the button to select. One downside is this replaces the old knob, there are designs to augment your existing knob, but for now I replaced it and tossed the old knob aside.

This gives me an idea, could also print knobs for potentiometers or variable autotransformers in electronics projects.

### Fan guard

<table>
  <tr>
    <th>Model</th>
    <td><a href="https://www.thingiverse.com/thing:1736041">30mm Minimal Fan Guard: MP Select Mini</a> by numanair, published Aug 25, 2016</td>
  </tr>
  <tr>
    <th>Estimated print time</th>
    <td>0:45</td>
  </tr>
  <tr>
    <th>Actual print time</th>
    <td>0:53, 8 minutes slower (18%)</td>
  <tr>
    <th>Estimated filament length</th>
    <td>0.77m</td>
  </tr>
  <tr>
    <th>Estimated filament mass</th>
    <td>2g</td>
  </tr>
  <tr>
    <th>Filament used</th>
    <td>AIO Robotics AIOWHITE PLA, $11.99/500 g</td>
  </tr>
  <tr>
    <th>Estimated cost</th>
    <td>$0.05</td>
  </tr>
</table>

A small safety modification: [fan grills](https://www.mpselectmini.com/thingiverse/fan-grills), covering the exposed fan near the hotend. There are two basic types, snap-on covers or those that require unscrewing the fan screws, I opted for the latter, a minimal grill the designer describes as "roughly based on the printer's design language while still allowing maximum airflow". Included in the download are two models, the original is 30mm_minimal_fan_guard.stl and then after feedback about the grill hitting the fan, the author added fan_guard_slots_06_standoffs.stl.stl. 

Import into Cura, rotate so it lies flat. The standoffs were facing up, but to get a cleaner print for the user-facing side, I wanted them to face downwards on the bed. Rotate then add a raft and supports. This will take more material, but I think (hope) it'll give a better result:

![Cura fan grills](https://user-images.githubusercontent.com/26856618/35181299-29851842-fd74-11e7-95df-5135612a5527.png)

Started out with printing a very nice raft, no problems there:

![Raft](https://user-images.githubusercontent.com/26856618/35181906-ce8da808-fd7f-11e7-859d-03d41ce66a64.png)

and before I knew it, the fan grill finished printing:

![Fan grill printed](https://user-images.githubusercontent.com/26856618/35181913-e68c1534-fd7f-11e7-8530-ea5b338a5726.png)

Gently separate the grill from the raft:

![Fan grill separated from raft](https://user-images.githubusercontent.com/26856618/35181920-fa2104ba-fd7f-11e7-9da7-7119ee49a365.png)

There is a lot of stringiness going on in the grill, cut it away with a knife. But while installing the grill into the fan, ran into a problem: the screws were too short. Cut away some of the layers of the standoffs, carefully. After doing so, was able to install the grill. Before:

![Fan before](https://user-images.githubusercontent.com/26856618/35181944-3ade2352-fd80-11e7-9b7b-fcd902ac9310.png)

After, the fan grill installed:

![Fan grill installed](https://user-images.githubusercontent.com/26856618/35181954-6d7561cc-fd80-11e7-8f43-a688938b9f78.png)

Powered on, preheated and homed, the fan turned on and did *not* rub against the grill, so there is sufficient clearance, cool.

### Extruder level arm

<table>
  <tr>
    <th>Model</th>
    <td><a href="https://www.thingiverse.com/thing:1812623">MP Select Mini Bolt-able Extruder Lever Arm</a> by ripcurl777, published Oct 7, 2016</td>
  </tr>
  <tr>
    <th>Estimated print time</th>
    <td>0:53</td>
  </tr>
  <tr>
    <th>Actual print time</th>
    <td>0:58, 5 minutes slower (9%)</td>
  <tr>
    <th>Estimated filament length</th>
    <td>1.84m</td>
  </tr>
  <tr>
    <th>Estimated filament mass</th>
    <td>5g</td>
  </tr>
  <tr>
    <th>Filament used</th>
    <td>AIO Robotics AIOWHITE PLA, $11.99/500 g</td>
  </tr>
  <tr>
    <th>Estimated cost</th>
    <td>$0.12</td>
  </tr>
</table>

The somewhat alarmist [This is your Daily reminder to PRINT REPLACEMENT PARTS WHILE THE PRINTER HAS NOTHING WRONG WITH IT!!!](https://www.reddit.com/r/MPSelectMiniOwners/comments/6a2wdz/this_is_your_daily_reminder_to_print_replacement/) thread has some good tips on what to print early on. Wasn't sure which of these parts I'll actually need but the having a spare extruder arm seems like a good idea. Sliced with infill density 100%, layer height 0.2mm, brim, print completed:

![Extruder arm printed](https://user-images.githubusercontent.com/26856618/35188572-899201e6-fdec-11e7-88b2-f45948a3066e.png)

Since the stock extruder arm still works I didn't replace it with this printed part, but here it is alongside for comparison:

![Extruder arm by its predecessor](https://user-images.githubusercontent.com/26856618/35188576-9f314ffc-fdec-11e7-882a-aaf717803450.png)

Stashed it away as a replacement. Who knows, maybe I'll desparately need it some day and be thankful I printed this now.

### Filament guide

<table>
  <tr>
    <th>Model</th>
    <td><a href="https://www.thingiverse.com/thing:2520971">Monoprice Select Mini Filament Guide with extra support</a> by ekopapers, published Sep 6, 2017</td>
  </tr>
  <tr>
    <th>Estimated print time</th>
    <td>0:32</td>
  </tr>
  <tr>
    <th>Actual print time</th>
    <td>0:40, 8 minutes slower (25%)</td>
  <tr>
    <th>Estimated filament length</th>
    <td>1.16m</td>
  </tr>
  <tr>
    <th>Estimated filament mass</th>
    <td>3g</td>
  </tr>
  <tr>
    <th>Filament used</th>
    <td>AIO Robotics AIOWHITE PLA, $11.99/500 g</td>
  </tr>
  <tr>
    <th>Estimated cost</th>
    <td>$0.07</td>
  </tr>
</table>

Found from [/r/MPSelectMiniOwners: First 3 days with my first 3D printer!](https://www.reddit.com/r/MPSelectMiniOwners/comments/7raeyl/first_3_days_with_my_first_3d_printer/). Sliced with layer height 0.2mm, infill 30%, no build plate adhension. Printed it on the righthand side of my bed because that's where I seem to get the best results:

![Filament guide printed](https://user-images.githubusercontent.com/26856618/35200116-3319d790-febd-11e7-9f98-5f2a055a6010.png)

Even without any build plate adhension options (no raft, no brim, no skirt, etc.), it printed relatively well, although there are some mysterious brown spots not sure what to make of those:

![Filament guide closeup each side](https://user-images.githubusercontent.com/26856618/35200126-69eb9d4e-febd-11e7-85c9-b601f3a32928.png)

Good enough for a functional part. This is where it will be inserted, attached through the screw in the corner:

![Without filament guide](https://user-images.githubusercontent.com/26856618/35200146-a4011fae-febd-11e7-86c6-5904393600f8.png)

Retract the filament using the home screen move menu, then press the latch and pull it out. Unscrew the corner screw and thread through the filament guide hole, then screw in, and thread the filament through the guide. The part inside the guide moves freely, but it was printed all in one piece (amazing). The installed guide:

![Filament guide installed](https://user-images.githubusercontent.com/26856618/35200167-fff62746-febd-11e7-8fa5-48c9127a43a1.png)

Now you can see the filament isn't fed from the spool at such an aggressive angle. Not that I have had any issues with in this department yet, but better to proactively make the upgrade to reduce the likelihood of problems, especially since it is so cheap. Performed a print with this guide installed, worked great.

---

I also tried to print a spool holder, but failed and I ditched this effort for now. Notes below:

The stock spool holder spools the filament at an awkward angle, instead of directly feeding into the extruder.  Fugatech 3D Printing recommended several improvements, including adding a new spool rack for better filament feeding: [Modding the Monoprice Select Mini (Part 1)](https://www.youtube.com/watch?v=TFO-qwQ8dLQ&t=8m10s). Two items on Thingiverse, the holder and an extension:

* [Monoprice Select Mini spool holder](https://www.thingiverse.com/thing:2006457) by Toys, published Dec 31, 2016
* [MPSM Spool Holder Extensions](https://www.thingiverse.com/thing:2192876)by Geonovast, published Mar 21, 2017

Download the first one. Cura change layer height to 0.1 mm and infill density to 30%.

A problem I had was how to fit the model onto the building area? The .stl model comes oriented unusually. But with some rotations and moves, I placed the most flat part down. This is a serious print: Lside_spool.stl alone is estimated to take 5 hours 42 minutes to print.  The designer said "rafts: no", I printed with no build adhension, 0.1 mm layer height, but failed catastrophically:

![Print failure](https://user-images.githubusercontent.com/26856618/35180907-4d19b9e4-fd6e-11e7-84ed-cc4f285811de.png)

I had a lot of trouble getting the pieces to fit into the bed. Rotate diagonally, but then raft can't fit.

There is another design: [Monoprice Select Mini Good Spec Spool Holder (larger Capacity)](https://www.thingiverse.com/thing:2723805) by 77-17, published Dec 21, 2017, found in the things for the [Monoprice Select Mini Owners](https://www.thingiverse.com/groups/monoprice-select-mini-owners/things) group, but the designer recommends printing with "PETG, or Tough-PLA", which I don't have. Guess I'll be passing on this upgrade for now, we'll see how well the filament guide by itself works.

### Spinner topper for extruder

<table>
  <tr>
    <th>Model</th>
    <td><a href="https://www.thingiverse.com/thing:2299324">Gyro Spinner For MPSM (keyless)</a> by TrunkFullaAmps, published May 7, 2017</td>
  </tr>
  <tr>
    <th>Estimated print time</th>
    <td>2:08</td>
  </tr>
  <tr>
    <th>Actual print time</th>
    <td>2:23, 15 minutes slower (12%)</td>
  <tr>
    <th>Estimated filament length</th>
    <td>4.56m</td>
  </tr>
  <tr>
    <th>Estimated filament mass</th>
    <td>13g</td>
  </tr>
  <tr>
    <th>Filament used</th>
    <td>AIO Robotics AIOWHITE PLA, $11.99/500 g</td>
  </tr>
  <tr>
    <th>Estimated cost</th>
    <td>$0.31</td>
  </tr>
</table>

Although may seem frivolous, actually provides a functional purpose. The extruder movement can be difficult to see, so by adding this topper it provides a clearer visual indicator to help diagnose problems like I first had when setting up, of the extruder not pushing through the filament. Slice with layer height 0.175m, infill 20%, no supports. 

The first layer came out poorly:

![Gyro spinner first layer](https://user-images.githubusercontent.com/26856618/35200665-36d6eae4-fec7-11e7-9686-bdb6925d9f18.png)

but apparently you can't judge a print by its first layer, because somehow it recovered:

![Gyro spinner printing](https://user-images.githubusercontent.com/26856618/35200679-6205413e-fec7-11e7-8c6d-04202c941582.png)

and finished beautifully:

![Gyro spinner printed](https://user-images.githubusercontent.com/26856618/35200685-72e1d2ba-fec7-11e7-9601-8c9f4b49b0e8.png)

The top piece although printed in one go is composed of concentric rings which rotate freely, and the bottom piece attaches to the extruder motor shaft. Specifically, it is going to go right here:

![Extruder shaft](https://user-images.githubusercontent.com/26856618/35200724-a2ecb5f6-fec7-11e7-9743-4d453571e9ea.png)

Took this opportunity to clean off the packing peanuts left there from shipment. Slide on the connector, then slide the tab from the gyro into it, it holds securely and spins when the extruder turns:

![Gyro spinner installed](https://user-images.githubusercontent.com/26856618/35200740-eb56d254-fec7-11e7-8d96-de90186126a4.png)

The large spinner is visible from afar, in another room, so I can quickly glance at it to check on it and confirm everything is alright. Good deal.

## Kitchen

Now that the printer is modded and optimized, we can turn to practical prints for functional purposes. See [/r/functionalprint](https://www.reddit.com/r/functionalprint/) for other ideas, and browse through [Thingiverse popular](https://www.thingiverse.com/explore/popular). Among the most popular, number one at the time of this writing, is related to kitchen stuff.

### Measuring cube

<table>
  <tr>
    <th>Model</th>
    <td><a href="https://www.thingiverse.com/thing:2676324">Measuring Cube</a> by iomaa, published Dec 1, 2017</td>
  </tr>
  <tr>
    <th>Estimated print time</th>
    <td>10:12</td>
  </tr>
  <tr>
    <th>Actual print time</th>
    <td>10:50, 38 minutes slower (6%)</td>
  <tr>
    <th>Estimated filament length</th>
    <td>25.59m</td>
  </tr>
  <tr>
    <th>Estimated filament mass</th>
    <td>76g</td>
  </tr>
  <tr>
    <th>Filament used</th>
    <td>AIO Robotics AIOWHITE PLA, $11.99/500 g</td>
  </tr>
  <tr>
    <th>Estimated cost</th>
    <td>$1.82</td>
  </tr>
</table>

An alternative to the clumsiness of measuring cups. First heard about this model from [3D Printing a Measuring Cube Kitchen Gadget + Thoughts on FDA Approved Filament and 3D Printing](https://www.youtube.com/watch?v=rRR7TU2lqPo) by 3D Printing Nerd. 

Load measuring_cube_version_2.STL into Cura. This is a big model, even reducing infill to 0%, no supports, increasing layer height to 0.2mm, estimated 15 hours 33 minutes, 39.5m / ~119g. That's a large percentage of my available filament. Instead, load up the miniature version: miniCube.STL. It doesn't have 1 cup, but has 1/2 cup and smaller. Only 10 hours 12 minutes, 25.59m / 76g, works out to $1.82 estimated cost of filament. I'll take it. Slightly apprehensive about this print because I'm going to have to leave it overnight...but I did, and so far so good (a few hours to go):

![Measuring cube printing](https://user-images.githubusercontent.com/26856618/35186424-d365f9cc-fdc8-11e7-9092-800145554b4b.png)

At less than $2 of filament, this is less than the $9 + shipping to $12 + shipping (based on their survey) iomaa is planning to ask for. But how useful is volumetric measurement really? There is some discussion about [legal vs customary cups](https://cooking.stackexchange.com/questions/28272/do-cooks-in-the-us-measure-volume-using-traditional-or-legal-units), a 1.44% difference, as well as variations caused by packing density, all which can be avoided by weighing the ingredients instead. But the measuring cube looks cool and is easier to use, if less precise. The print completed almost 11 hours later:

![Measuring cube printed](https://user-images.githubusercontent.com/26856618/35188280-5fa96b12-fde7-11e7-8b12-4eb58f8cfa57.png)

Looks quite nice. The only imperfection is the bottom of the cube (not shown), probably due to my warping bed. To test it, how about making some rice? Measured slightly less than half cup (so you can see the label for this photo):

![Measuring cube rice](https://user-images.githubusercontent.com/26856618/35188288-77430bf2-fde7-11e7-9bee-240faf4c024d.png)

but I used a glass measuring cup to measure the water, due to concerns about water safety of PLA. Cook it up in a rice cooker, eat lunch. Yum:

![Rice measured with measuring cube](https://user-images.githubusercontent.com/26856618/35188561-3bde5bc0-fdec-11e7-9ded-dabd3df9d002.png)

### Bag clip

<table>
  <tr>
    <th>Model</th>
    <td><a href="https://www.thingiverse.com/thing:1212828">Chip Bag Clip V7</a> by srwilson58, published Dec 19, 2015
</td>
  </tr>
  <tr>
    <th>Estimated print time</th>
    <td>2:08</td>
  </tr>
  <tr>
    <th>Actual print time</th>
    <td>2:21, 13 minutes slower (10%)</td>
  <tr>
    <th>Estimated filament length</th>
    <td>4.41m</td>
  </tr>
  <tr>
    <th>Estimated filament mass</th>
    <td>13g</td>
  </tr>
  <tr>
    <th>Filament used</th>
    <td>AIO Robotics AIOWHITE PLA, $11.99/500 g</td>
  </tr>
  <tr>
    <th>Estimated cost</th>
    <td>$0.31</td>
  </tr>
</table>


Says it is for bags of chips, but probably works with bags of other contents like salad. Slice with layer height 0.2mm, infill 100% (seems like a lot, but it asked for it, for the added strength I imagine), no build plate adhension options.

Is it worth 31¢ for a bag clip? Well, it is a different design but [OXO Good Grips Bag Clip -2 Pack](https://www.amazon.com/OXO-Good-Grips-Clip-Pack/dp/B0002YTG5Y) is $5.99/2 red (or $18.95/2 for white, geez), or $2.995 apiece, $0.31 is 90% cheaper.

Completed print:

![Bag clip printed](https://user-images.githubusercontent.com/26856618/35208484-9640d9be-fefd-11e7-9451-7d21dfe2a0f0.png)

The white PLA flexes when squeezing the clip, no issue. Successfully used to close a bag of salad:

![Bag clip clipping bag](https://user-images.githubusercontent.com/26856618/35208577-02b3c53e-fefe-11e7-95ca-96ca958c91b3.png)


## Bathroom

[Useful 3D prints: #1 Bathroom](https://www.youtube.com/watch?v=le1jvn8kMoo) by Nikodem Bartnik has some cool applications. Maybe he'll do more useful 3D prints, this is the first of what appears to be a series, published October 24th, 2017 but there isn't a sequel yet, I look forward to it. Anyways, some useful bathroom things.

A neat set of towel hooks, soap dish, toilet paper holder, and toothbrush holder, all themed with dinosaurs: [T-REX Remix Bathroom Set](https://www.thingiverse.com/thing:325294) by Solstie, published May 12, 2014. But I'll try simpler, unthemed models first.

### Toothpaste squeezer

<table>
  <tr>
    <th>Model</th>
    <td><a href="https://www.thingiverse.com/thing:1147252">Toothpaste Tube Squeezer</a> by ottenjr, published Nov 21, 2015</td>
  </tr>
  <tr>
    <th>Estimated print time</th>
    <td>2:56</td>
  </tr>
  <tr>
    <th>Actual print time</th>
    <td>3:17, 21 minutes slower (12%)</td>
  <tr>
    <th>Estimated filament length</th>
    <td>6.50m</td>
  </tr>
  <tr>
    <th>Estimated filament mass</th>
    <td>19g</td>
  </tr>
  <tr>
    <th>Filament used</th>
    <td>AIO Robotics AIOWHITE PLA, $11.99/500 g</td>
  </tr>
  <tr>
    <th>Estimated cost</th>
    <td>$0.46</td>
  </tr>
</table>

There are two basic designs, the simpler one including [Bartnik's](https://cdn.instructables.com/ORIG/F8Q/E49B/J8YRBTIC/F8QE49BJ8YRBTIC.stl) is just a small slot you push over the toothpaste tube to push out the paste, similar to this design: [Simple Toothpaste Squeezer](https://www.thingiverse.com/thing:867811) by ChatterComa, published Jun 6, 2015. But it leaves the empty part of the tube dangling out the end. A more sophisticated design rolls up the empty tube, out of the way.

I sliced it with a brim since I've had good results with build plate adhension options, but the designer didn't say it was needed, better safe than sorry. The print completed no problem:

![Toothpaste squeezer](https://user-images.githubusercontent.com/26856618/35190007-310a1aa2-fe0c-11e7-97b2-c533ecb22eba.png)

Comes in three parts. Scrape away the debris, separate from the brims, lay out the components:

![Toothpaste squeezer parts](https://user-images.githubusercontent.com/26856618/35190013-53230cf2-fe0c-11e7-912b-df056f47ed3b.png)

To assemble, insert the pin into the tube then affix the cap on the end, it came together easily. Slide a toothpaste tube through both slots, in the holder and pin:

![Toothpaste squeezer squeezing toothpaste](https://user-images.githubusercontent.com/26856618/35190019-7c71ced6-fe0c-11e7-92fb-1e716aa4fa7f.png)

You can feel the paste being squeezed out, works well. But this squeezer has another feature, it can roll up the tube:

![Toothpaste squeezer rolled](https://user-images.githubusercontent.com/26856618/35190022-9ea35f38-fe0c-11e7-9f95-3afca7d0aae5.png)

Nifty!

### Toothbrush holder

<table>
  <tr>
    <th>Model</th>
    <td><a href="https://www.thingiverse.com/thing:1340542">toothbrush Holder</a> by theTeV, published Feb 13, 2016</td>
  </tr>
  <tr>
    <th>Estimated print time</th>
    <td>3:37</td>
  </tr>
  <tr>
    <th>Actual print time</th>
    <td>3:58, 21 minutes slower (10%)</td>
  <tr>
    <th>Estimated filament length</th>
    <td>8.60m</td>
  </tr>
  <tr>
    <th>Estimated filament mass</th>
    <td>25g</td>
  </tr>
  <tr>
    <th>Filament used</th>
    <td>AIO Robotics AIOWHITE PLA, $11.99/500 g</td>
  </tr>
  <tr>
    <th>Estimated cost</th>
    <td>$0.60</td>
  </tr>
</table>

The original: <a href="https://www.thingiverse.com/thing:265761/">Quad Toothbrush Holder</a> by by JMSchwartz11, published Mar 7, 2014. There are several [remixes](https://www.thingiverse.com/thing:265761/#remixes) of this design, but the "more printable" variants are more square; I like the rounded corners of the original, if I can print it. Can I? Only one way to find out... load into Cura. Too big :(. Try theTeV's 3-hole version, rotate on its side. It fits. Layer height 0.2mm, infill 20%, no build plate adhension. Completed print:

![Toothbrush holder printed](https://user-images.githubusercontent.com/26856618/35205527-a5ae8800-feea-11e7-8f8a-8e8cdaa52c9a.png)

A wider base so it doesn't fall over as easily would be nice, but it works:

![Toothbrush holder holding toothbrush](https://user-images.githubusercontent.com/26856618/35205538-c039948a-feea-11e7-9a08-3f476f4a0e94.png)

### Soap dish

<table>
  <tr>
    <th>Model</th>
    <td><a href="https://www.thingiverse.com/thing:808786">Corner Soap Dish</a> by hammerbot, published May 5, 2015</td>
  </tr>
  <tr>
    <th>Estimated print time</th>
    <td>6:10 (5:41 + 0:29)</td>
  </tr>
  <tr>
    <th>Actual print time</th>
    <td>6:34 (5:59 + 0:35), 24 minutes slower (6%)</td>
  <tr>
    <th>Estimated filament length</th>
    <td>16.11m (14.35m + 1.76m)</td>
  </tr>
  <tr>
    <th>Estimated filament mass</th>
    <td>47g (42g + 5g)</td>
  </tr>
  <tr>
    <th>Filament used</th>
    <td>AIO Robotics AIOWHITE PLA, $11.99/500 g</td>
  </tr>
  <tr>
    <th>Estimated cost</th>
    <td>$1.13</td>
  </tr>
</table>

[Soap holder](https://www.thingiverse.com/thing:404028) by piuLAB, published Jul 23, 2014, includes two pieces, detachable for cleaning. But 141.6 mm x 123.1 mm is beyond 120 mm x 120 mm of the Maker Select Mini v2. Too small. [Bathroom Things](https://www.thingiverse.com/thing:2270557) by pegasocube, published Apr 24, 2017, includes another soap tray, more open, with a raised grill. Also wanted to make the [Modular Shower Caddy](https://www.thingiverse.com/thing:1610224) by Borchei, published Jun 6, 2016, but it is just too big for the Monoprice Select Mini v2. One side is 184 mm, above the 120 mm bed. 

Found a model that fits, the corner soap dish. This uses an interesting technique, the grate is created by setting shell top/bottom thickness to 0, then adding an infill, about 20% looks good. The grate should be very quick to print, only about a half hour. The full dish is more time-consuming, a good candidate for overnight printing. No infill but thick 1.2m walls with 0.2mm layer height. Started the full dish late at night, layed down a gorgeous first few layers:

![Soap dish printing](https://user-images.githubusercontent.com/26856618/35254182-0a4b67d4-ff9e-11e7-977f-b4c8ecc14535.png)

finished a minute less than six hours later:

![Soap dish printed](https://user-images.githubusercontent.com/26856618/35254210-2558ebdc-ff9e-11e7-8069-966d5bc46422.png)

The grate soon after:

![Soap dish grate printed](https://user-images.githubusercontent.com/26856618/35254236-3f496daa-ff9e-11e7-8eaa-d8d36cf5638d.png)

Fits the soap and into my shower corner decently well, although not as perfect fit of a custom design:

![Soap dish holding soap](https://user-images.githubusercontent.com/26856618/35254274-6724115e-ff9e-11e7-90d6-8a34c41dc4d0.png)

## Electronics

Should 3D printers be used to print electronics project enclosures? The comments in [Hacker News discussion: The Guide to Not Buying a 3D Printer](https://news.ycombinator.com/item?id=8235016) have naysayers:

> moron4hire 1241 days ago
> I'd also like to add: make sure you actually need a 3D print. Most of the stuff I see people printing could be made faster and more reliably with more traditional manufacturing methods. For example: a 3D printed case for just about anything is a terrible idea. It's very hard to get square, true prints of specific dimensions. I've seen many a warped Raspberry Pi case.

but also success stories:

> bri3d 1240 days ago
> What if I'm optimizing for cost, not speed and reliability? My printer works just fine while I'm away, so I don't really care about a print taking an hour or two, and plastic is bendy and forgiving enough that a bit of accuracy sacrificed is fine.
> I'd argue that additive manufacturing via hobbyist FDM printing is a lot cheaper for one off designs than subtractive manufacturing of any form, because the consumable cost is limited to utilized filament and a tiny amount of printer maintenance rather than huge amounts of waste combined with expensive, finicky tool bits and the same kinds of maintenance.
> I do agree that for items like a Raspberry Pi case which could be mass-produced via molding to much better tolerance at a low cost, 3D printing might be silly, but for truly one-off, low accuracy consumer parts I don't think 3D printing can be beat.
> I personally have also not had much problem getting square, true, dimensionally accurate prints with a crappy printer (Solidoodle 2). The most deflection I've ever seen is about .8mm across the 150x150mm build area. That isn't perfect and obviously won't work for precision equipment, but combined with ABS's flexibility and forgiving nature it's certainly good enough to make one-off enclosures, fasteners, buttons and knobs, adapters, and other "glue" parts, which is most of my use for 3D printing.
> I will say that 3D printers are hard to set up, and that's probably the source of a lot of the warped cases you've seen. CNC mills and lathes are just as hard to set up, though - you've just generally paid the vendor tons of money to do it for you.

How about in practice, in reality? As John Salvatier acutely pointed out, [Reality has a surprising amount of detail](http://johnsalvatier.org/blog/2017/reality-has-a-surprising-amount-of-detail) (see also [discussion](https://news.ycombinator.com/item?id=16184255)). Maybe you can get away with purchasing commonly-sized enclosures and fitting your electronics into it. But I purchased this case: [New ABS DIY Plastic Electronic Project Box Enclosure Instrument 100x60x25mm VE834 P](https://www.aliexpress.com/item/New-ABS-DIY-Plastic-Electronic-Project-Box-Enclosure-Instrument-100x60x25mm-VE834-P/32503805997.html), $1 from Aliexpress, and have yet to find a suitable circuit board for it. Would need to cut the boards to make fit or design specifically for it. How about instead designing the case to fit the board? At $1 we have about 42 grams of filament (or more) to work with, that's a lot to be cost-competitive.

### STM32 blue pill case for Black Magic Probe

<table>
  <tr>
    <th>Model</th>
    <td><a href="https://www.thingiverse.com/thing:2513993">STM32 blue pill case (blackmagic programmer)</a> by thmjpr, published Sep 2, 2017</td>
  </tr>
  <tr>
    <th>Estimated print time</th>
    <td>1:54</td>
  </tr>
  <tr>
    <th>Actual print time</th>
    <td>2:10, 16 minutes slower (14%)</td>
  <tr>
    <th>Estimated filament length</th>
    <td>3.92m</td>
  </tr>
  <tr>
    <th>Estimated filament mass</th>
    <td>11g</td>
  </tr>
  <tr>
    <th>Filament used</th>
    <td>AIO Robotics AIOWHITE PLA, $11.99/500 g</td>
  </tr>
  <tr>
    <th>Estimated cost</th>
    <td>$0.26</td>
  </tr>
</table>

There is another <a href="https://www.thingiverse.com/thing:2461399">Bluepill case</a> by BogdanKecman, published Jul 31, 2017, but it is only the bottom. Cover the various STM32-based projects. For the first one, since it is most complete, would like to make a case for:

* *[JTAG/SWD debugging via Black Magic Probe on an STM32 blue pill and blinking a LED using STM32CubeMX, libopencm3, and bare metal C](https://satoshinm.github.io/blog/171223_jtagswdpillblink_jtagswd_debugging_via_black_magic_probe_on_an_stm32_blue_pill_and_blinking_a_led_using_stm32cubemx_libopencm3_and_bare_metal_c.html)*

Drag both of the files bmagic_case_bottom.STL and bmagic_case_top.STL into Cura, position on the printing area. The designer used a 100% infill but I'm trying 0%, layer height 0.2mm. Finished printing:

![Printed blue pill case](https://user-images.githubusercontent.com/26856618/35191345-ce4e0d98-fe2d-11e7-939a-3cd8cf5b83af.png)

There are some imperfections I had to carve away, but a bare STM32 blue pill board fits in:

![Blue pill case apart with blue pill in](https://user-images.githubusercontent.com/26856618/35191358-f4d4484c-fe2d-11e7-8b3b-a692ae890b4f.png)

luckily I had just received two new blue pill development boards, and didn't solder the headers on this one yet. But before we go any further, best to flash the blue pill with the Black Magic Probe firmware (following the classic instructions [Converting an STM32F103 board to a Black Magic Probe](https://medium.com/@paramaggarwal/converting-an-stm32f103-board-to-a-black-magic-probe-c013cf2cc38c) from Param Aggarwal) before sealing up the case:

![Trying to program the blue pill](https://user-images.githubusercontent.com/26856618/35191372-273b1efa-fe2e-11e7-8a98-3d8ffbb8df95.png)

attempting to flash with another Black Magic Probe, and I also attempted with a USB-to-serial adapter, but both kept failing with:

```
__main__.CmdException: NACK 0x43
```

[stm32duino forums: Can't initiate chip erase](http://www.stm32duino.com/viewtopic.php?t=747) indicates this means the chip is locked, and needs to be unlocked and erased, such as by using the [STM32 Flash loader demonstrator (UM0462)](http://www.st.com/en/development-tools/flasher-stm32.html). Well, flash_loader_demo_v2.8.0.exe is Windows-only, and [stm32loader.py](https://github.com/jsnyder/stm32loader) doesn't support seem to unlocking. Running stm32flasher.py in verbose mode (-V) shows the commands it supports:

```
*** Get command
    Bootloader version: 0x22
    Available commands: 0x0, 0x1, 0x2, 0x11, 0x21, 0x31, 0x43, 0x63, 0x73, 0x82, 0x92
Bootloader version 22
*** GetID command
Chip id: 0x410 (STM32 Medium-density)
Traceback (most recent call last):
  File "/usr/local/bin/stm32loader.py", line 451, in <module>
    cmd.cmdEraseMemory()
  File "/usr/local/bin/stm32loader.py", line 212, in cmdEraseMemory
    if self.cmdGeneric(0x43):
  File "/usr/local/bin/stm32loader.py", line 115, in cmdGeneric
    return self._wait_for_ask(hex(cmd))
  File "/usr/local/bin/stm32loader.py", line 88, in _wait_for_ask
    raise CmdException("NACK "+info)
__main__.CmdException: NACK 0x43
```

From looking at the source, these commands are implemented by these functions in stm32loader:

| Command | Function |
| ------- | -------- |
| 0x00 | cmdGet |
| 0x01 | cmdGetVersion | 
| 0x02 | cmdGetID | 
| 0x11 | cmdReadMemory |
| 0x21 | cmdGo | 
| 0x31 | cmdWriteMemory |
| 0x43 | cmdEraseMemory |
| 0x63 | cmdWriteProtect |
| 0x73 | cmdWriteUnprotect |
| 0x82 | cmdReadoutProtect |
| 0x92 | cmdReadoutUnprotect |

Unprotect...that sounds a lot like unlock, is it the same thing? Nothing calls these functions, but there is commented-out code which could:

```python
        mdebug(0, "Chip id: 0x%x (%s)" % (id, chip_ids.get(id, "Unknown")))
#    cmd.cmdGetVersion()
#    cmd.cmdGetID()
#    cmd.cmdReadoutUnprotect()
#    cmd.cmdWriteUnprotect()
#    cmd.cmdWriteProtect([0, 1])
```

Uncomment cmdReadoutUnprotect and then cmdWriteUnprotect. Execute and both seem to fail with a timeout, but then after rebooting and rerunning erase, I can write to the chip! Shows up as Black Magic Probe in firmware update mode, then run dfu-util as usual, and the firmware is all set: two new device nodes in /dev, cu.usbmodemDFE17CC3 and cu.usbmodemDFE17CC1. Back to making the case.

There are two ports on this case, 3-pin and 2x3-pin. What pinouts to use? Couldn't find any, and the official [JTAG SWD 10pin IDC Cable](https://1bitsquared.com/products/jtag-idc-cable) is 10-pin, nor the [1bitsquared 1b2-adapter-collection/bmp_4pin_50mil_swd](https://github.com/1bitsquared/1b2-adapter-collection/tree/master/bmp_4pin_50mil_swd) connector, so I just picked my own pins. For the 3-pin port, use it as the serial port, bridged to the virtual USB serial port (/dev/cu.usbmodemDFE17CC3 on my system):

* 1 (black): GND
* 2 (yellow): RXD (A3)
* 3 (blue): TXD (A2)

Push wires through the plastic tabs (may require some cutting, I used a small flat-head screwdriver to push the wire) then solder onto the other end of a 100 mil header. Used the yellow 0.1" headers included with the blue pill, broke off a few pieces, since they are not needed for the board itself. Getting the hang of it:

![UART port soldered](https://user-images.githubusercontent.com/26856618/35191965-0f1589b4-fe3d-11e7-9b5f-016bab10d9b1.png)

For the double header, which is for JTAG/SWD, most important are the SWD pins so placed those first, but also the additional JTAG pins if needed. Top header wires, away from the USB port side, all you need for Serial Wire Debug:

* 1 (black): GND
* 2 (green): SWDIO/TMS (B14)
* 3 (white): SWCLK/TCK (A5)

![SWD port](https://user-images.githubusercontent.com/26856618/35191976-470193b8-fe3d-11e7-80e9-3a97ccd53632.png)

On the other side, additional pins for JTAG, serial wire output, and power:

* 1 (red). 3.3V
* 2 (green): TDI (A7)
* 3 (red): SWO/TDO (A6)

![JTAG port](https://user-images.githubusercontent.com/26856618/35191980-5a35e3ee-fe3d-11e7-9e62-a4824128a8b3.png)

Before closing, tested with another serial adapter connected to this device's serial port. Worked perfectly, the data was transmitted/received, and the red power LED shines through the PLA plastic:

![BMP powered on case half open](https://user-images.githubusercontent.com/26856618/35192008-b1f20a0e-fe3d-11e7-9e67-6d368f1df8a6.png)

Fold it up. This is a snap-in case, so the two pieces should fit together. However, the debug port right-angle header stuck out too far. I tried to desolder it but hit an adjacent capacitor, so instead, cut off the ends so it fits. Nearly seamless:

![Closed case](https://user-images.githubusercontent.com/26856618/35192001-987e84f8-fe3d-11e7-8887-b9f53c1dd3a2.png)

Plug it in and power it up:

![Powered while in case](https://user-images.githubusercontent.com/26856618/35192024-ff10a390-fe3d-11e7-92e9-6420acf5824f.png)

Cool. Looks nice, yet compact. One problem: no labels. For a later version, would be nice to include labels raised or recessed on the case. But until then, I affixed general-purpose sticky labels and wrote on them with a pen, labeling this device "Black Magic Probe - UART, SWD/JTAG" and the pin outs. 

How well does it work in practice? Here I am flashing *[pill_6502: 8-bit 6502 CPU and 6850 ACIA emulation on the STM32 blue pill to run Microsoft BASIC from 1977](https://satoshinm.github.io/blog/180113_stm32_6502_pill_6502_8_bit_6502_cpu_and_6850_acia_emulation_on_the_stm32_blue_pill_to_run_microsoft_basic_from_1977.html)* onto a new blue pill board (had to un-write-protect this flash too) using this blue pill-based Black Magic Probe with 3D printed case:

![BMP flashing pill_6502](https://user-images.githubusercontent.com/26856618/35192275-4709e4b8-fe43-11e7-92a1-e47235c7afe4.png)

This elegant construction replaces the mess of wires on a breadboarded blue pill you may have seen in previous blog posts, this printed case is definitely a win. I can also see using variations of it for other blue pill related projects.

### Solder spool stand

<table>
  <tr>
    <th>Model</th>
    <td><a href="https://www.thingiverse.com/thing:563949">Solder Spool Stand Improved</a> by AndrewBCN, published Nov 25, 2014 </td>
  </tr>
  <tr>
    <th>Estimated print time</th>
    <td>3:03</td>
  </tr>
  <tr>
    <th>Actual print time</th>
    <td>3:26, 23 minutes slower (13%)</td>
  <tr>
    <th>Estimated filament length</th>
    <td>6.37m</td>
  </tr>
  <tr>
    <th>Estimated filament mass</th>
    <td>18g</td>
  </tr>
  <tr>
    <th>Filament used</th>
    <td>AIO Robotics AIOWHITE PLA, $11.99/500 g</td>
  </tr>
  <tr>
    <th>Estimated cost</th>
    <td>$0.43</td>
  </tr>
</table>

Not too much to say about this one, it is an object to hold your spool of solder. Completed print:

![Solder spool stand printed](https://user-images.githubusercontent.com/26856618/35192127-6f2e1958-fe40-11e7-913b-d4c3ecb33f32.png)

Two pieces: the spindle and stand itself. Thread the spindle through the solder spool (removing any labels that block the passage), then lay it on the stand. No snapping in, it rests inside. To help guide the solder, take a thick wire, bend it into a small coil, then stick into the hole into the base. Heat it up with a the soldering iron then press to insert it into the plastic. Then you can thread the solder itself through, and pull it whenever you need solder:

![Solder spool stand with solder](https://user-images.githubusercontent.com/26856618/35192151-e5d9fd6a-fe40-11e7-831f-a90392f8e1d3.png)

The spool doesn't roll too smoothly, I think I'd prefer a round spindle instead of rectangular, but it'll do. Also would be nice to have a stand for solder wick, maybe even a combined stand, but since that is used less, a solder spool stand is at least useful for now.

### Soldering fingers

<table>
  <tr>
    <th>Model</th>
    <td><a href="https://www.thingiverse.com/thing:1725308">Soldering Fingers</a> by mistertech, published Aug 19, 2016 </td>
  </tr>
  <tr>
    <th>Estimated print time</th>
    <td>2:11</td>
  </tr>
  <tr>
    <th>Actual print time</th>
    <td>2:26, 15 minutes slower (11%)</td>
  <tr>
    <th>Estimated filament length</th>
    <td>7.59m</td>
  </tr>
  <tr>
    <th>Estimated filament mass</th>
    <td>22g</td>
  </tr>
  <tr>
    <th>Filament used</th>
    <td>AIO Robotics AIOWHITE PLA, $11.99/500 g</td>
  </tr>
  <tr>
    <th>Estimated cost</th>
    <td>$0.53</td>
  </tr>
</table>

For holding wires together when soldering or heatshrinking. 0.3mm layer height, 20% infill. Printed:

![Printed soldering fingers](https://user-images.githubusercontent.com/26856618/35198488-864de94c-fea4-11e7-9366-498b5c735a9c.png)

The base was very solid, you could hear the extruder extruding what sounded excessive, but I'm happy with the result, it is fairly sturdy. The most consistent solid base I have printed yet:

![Soldering fingers base](https://user-images.githubusercontent.com/26856618/35198559-7698fcfc-fea5-11e7-9150-7210e4ca09d4.png)

Does it work? Tested soldering two small white wires:

![Solder fingers soldering](https://user-images.githubusercontent.com/26856618/35198569-9a5191ae-fea5-11e7-97c0-69546d0d7f20.png)

The fingers grip the wires well. Used it conjunction with the solder spool made previously, a killer combination.

### Tool carousel for small tools

<table>
  <tr>
    <th>Model</th>
    <td><a href="https://www.thingiverse.com/thing:702912">Tool Carousel for small tools</a> by mbeyerle116, published Feb 28, 2015</td>
  </tr>
  <tr>
    <th>Estimated print time</th>
    <td>9:17</td>
  </tr>
  <tr>
    <th>Actual print time</th>
    <td>10:04, 47 minutes slower (8%)</td>
  <tr>
    <th>Estimated filament length</th>
    <td>23.38m</td>
  </tr>
  <tr>
    <th>Estimated filament mass</th>
    <td>69g</td>
  </tr>
  <tr>
    <th>Filament used</th>
    <td>AIO Robotics AIOWHITE PLA, $11.99/500 g</td>
  </tr>
  <tr>
    <th>Estimated cost</th>
    <td>$1.65</td>
  </tr>
</table>

Layer height 0.2mm, 0% infill. Rotate to fit both on the bed. Raft, and supports because I rotated the base vertically to fit:

![Tool carousel in Cura](https://user-images.githubusercontent.com/26856618/35198425-b8b25bd0-fea3-11e7-9f66-b13260b68665.png)

The raft printed beautifully uniform, no problems with adhension:

![Nice raft](https://user-images.githubusercontent.com/26856618/35198653-93961532-fea6-11e7-9dbe-b90fbe6fbf03.png)

A long, overnight print, but it is shaping up:

![Tool carousel printing](https://user-images.githubusercontent.com/26856618/35198659-c2a7d98c-fea6-11e7-9549-3c9f7566d5a6.png)

Finally completed. Took some care to remove from the bed. The raft looks as nice coming out as it did going in:

![Bottom of raft](https://user-images.githubusercontent.com/26856618/35198911-cbc2eb20-feaa-11e7-808e-31810f041f05.png)

but the downside is a fair amount of sacrifical pieces that have to be broken away, including supports (which again may have not been strictly necessary but I wanted to be sure with the smaller print area and how I rotated the parts):

![Tool carousel breaking apart](https://user-images.githubusercontent.com/26856618/35198916-eaadf28c-feaa-11e7-8772-bd232150eb1e.png)

There was moderate stringiness around the tool holes, but nothing a knife can't fix. The top wasn't as smooth as I would have liked (supposedly, glass beds would be good for improving this), which I was first annoyed with, but then realized when occupied the tools mostly cover these imperfections up (although sanding it down wouldn't be a bad idea later). Fully stocked with tools:

![Tool carousel with tools](https://user-images.githubusercontent.com/26856618/35198943-5ed47b36-feab-11e7-978f-59fa0e0f1cb8.png)

Screwdrivers, tweezers, pliers, side cutters, marker, brush, wire strippers. Meant for smaller tools, but slightly larger tools like those bigger screwdrivers seem to fit fine, downwards. The very small eyeglass screwdrivers don't hold too well, some kind of indentation in the base to catch the end would be nice for a future revision. The larger middle hole is good for bigger tools, albeit only one. But all in all, this tool carousel is a great space saver and organizer for my workspace, spacious and stable. Another shot, top down, with all the tools I could fit:

![Tool carousel with tools from top](https://user-images.githubusercontent.com/26856618/35198982-d8023dd6-feab-11e7-929e-0f01da800b54.png)

## Summary & conclusion

<table>
  <tr>
    <th>Total models printed (15)</th>
    <td>
      <ul>
        <li>     <a href="https://www.thingiverse.com/thing:1817172">Tool Saddle for MP Select Mini</a> by ArmArakel, published Oct 10, 2016
<li>     <a href="https://www.thingiverse.com/thing:2245341">MPSM Spinner Knob</a> by VA3TNE, published Apr 12, 2017
<li>     <a href="https://www.thingiverse.com/thing:1736041">30mm Minimal Fan Guard: MP Select Mini</a> by numanair, published Aug 25, 2016
<li>     <a href="https://www.thingiverse.com/thing:1812623">MP Select Mini Bolt-able Extruder Lever Arm</a> by ripcurl777, published Oct 7, 2016
<li>     <a href="https://www.thingiverse.com/thing:2520971">Monoprice Select Mini Filament Guide with extra support</a> by ekopapers, published Sep 6, 2017
<li>     <a href="https://www.thingiverse.com/thing:2299324">Gyro Spinner For MPSM (keyless)</a> by TrunkFullaAmps, published May 7, 2017
<li>     <a href="https://www.thingiverse.com/thing:2676324">Measuring Cube</a> by iomaa, published Dec 1, 2017
<li>     <a href="https://www.thingiverse.com/thing:1212828">Chip Bag Clip V7</a> by srwilson58, published Dec 19, 2015
<li>     <a href="https://www.thingiverse.com/thing:1147252">Toothpaste Tube Squeezer</a> by ottenjr, published Nov 21, 2015
<li>     <a href="https://www.thingiverse.com/thing:1340542">toothbrush Holder</a> by theTeV, published Feb 13, 2016
<li>     <a href="https://www.thingiverse.com/thing:808786">Corner Soap Dish</a> by hammerbot, published May 5, 2015
<li>     <a href="https://www.thingiverse.com/thing:2513993">STM32 blue pill case (blackmagic programmer)</a> by thmjpr, published Sep 2, 2017
<li>     <a href="https://www.thingiverse.com/thing:563949">Solder Spool Stand Improved</a> by AndrewBCN, published Nov 25, 2014 
<li>     <a href="https://www.thingiverse.com/thing:1725308">Soldering Fingers</a> by mistertech, published Aug 19, 2016 
<li>     <a href="https://www.thingiverse.com/thing:702912">Tool Carousel for small tools</a> by mbeyerle116, published Feb 28, 2015
      </ul>
    </td>
  </tr>
  <tr>
    <th>Total actual print time</th>
    <td>53:27, about 74% utilization over 72:00</td>
  <tr>
    <th>Total estimated filament mass</th>
    <td>339g, about 68% of the 500g roll</td>
  </tr>
</table>

Got the printer late in the week, set it up, then started printing almost non-stop from Friday throughout the weekend. Estimating 3 days = 72 hours, this means the printer was in use almost 75% of the time, since my total print lengths summed to 53 hours, 27 minutes. Strictly more than the 48 hours composing the weekend but I think we all can agree this was a productive few days. More than half of the 500g filament spool was used, summing to 339g not including failed or test prints.

All in all I [made](https://www.thingiverse.com/satoshinm/makes) 15 useful objects. Not bad for my first experience with 3D printing, I'd say. Overall I'm very satisified with the Monoprice Select Mini v2 and agree with the chorus of opinions on various forums it is an excellent beginner starter 3D printer. About the only limitation I occasionally ran into was its limited print bed size (although there are [MPSelectMini wiki: Mods & Improvements](https://www.mpselectmini.com/mods) that can be made to expand the bed size, and also these more advanced mods which I didn't try: [Let's Print 3D: Essential Mods to Upgrade Your 3D Printer (Maker Select v2)](https://letsprint3d.net/2017/07/14/essential-mods-upgrade-3d-printer/), so I couldn't print everything I wanted, but this problem rarely came up. Most useful objects you can print fit within the limited 120 mm dimensions.

The AIOWHITE filament was also superb, although pricey. I may try other midrange brands once/if I run out of this spool. By my estimates I have only 161g remaining, but I haven't actually weighed the spool before and after to get an accurate measurement (note to self: next time, weigh immediately). The estimates from the Cura slicing program were consistently lower than the actual print times, from 1 minute to 47 minutes depending on the total length, or from 1% to 25% (both very short prints), an arithmetic average underestimate of 11.3%. Keep this in mind when scheduling print jobs in the future. Real-time print length estimation and scheduling would be nice, alas, the Monoprice Select Mini v2 only shows you the actual print time when it completes, and only has an unlabeled progress bar during printing.

Long story short, the MP Select Mini v2 provided a perfect introduction to 3D printing. There are lower-end and higher-end printers but I'll probably stay with the MPSM for a while, until I outgrow it. Even with this limited machine, thanks to the vast universe of models designed and published by modelers from around the world on [Thingiverse](https://www.thingiverse.com) and elsewhere, tons of everyday useful items (and specialized niche items) can be readily available at your fingertips, manufactured conveniently at your home. And I haven't even scratched the surface of the true power of 3D printing: designing our own custom models.

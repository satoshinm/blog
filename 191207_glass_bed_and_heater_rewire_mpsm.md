# Glass bed and heater rewire mods to Monoprice Select Mini v2 3D printer

by snm, December 7th, 2019

* auto-gen TOC:
{:toc}

I wrote about the Monoprice Select Mini v2 3D printer after extensively using it early last year:

* *[3D Printing with 3D MARS white PLA: pass-through wallplate, paper towel holder, right-angle spool holder, shower curtain rings, and a prototype case](https://satoshinm.github.io/blog/180208_3dprint2_3d_printing_with_3d_mars_white_pla_pass_through_wallplate_paper_towel_holder_right_angle_spool_holder_shower_curtain_rings_and_a_prototype_case.html)*
* *[3D printing with the Monoprice Select Mini v2: first impressions and prints: a saddle, knob, guard, guide, spinner; a cube and clip; squeezer, holder, dish; case, stand, fingers, and carousel](https://satoshinm.github.io/blog/180122_3dprint1_3d_printing_with_the_monoprice_select_mini_v2_first_impressions_and_prints_a_saddle_knob_guard_guide_spinner_a_cube_and_clip_squeezer_holder_dish_case_stand_fingers_and_carousel.html)*

but since then, prints have started failing, causing the printer to fall into disuse.

What went wrong? The common cause of the print failures can be traced to the bed.
The MPSM comes with a [BuildTak](https://www.buildtak.com) surface, but it doesn't last forever.
Even after carefully re-leveling the bed, there was only a small area of the surface I could
reliably successfully print on, a frustrating and limiting experience.

To revive this printer, it is some to perform some much-needed upgrades.

## Glass bed

The first upgrade I'll perform is replacing the aging BuildTak surface with a fresh new and
glass bed. This is quite a common upgrade for the MPSM, because glass is much flatter than
BuildTak, making it easier to consistently level and successfully print on.

Ordered this borosilicate glass bed from Amazon:

* [GO-3D PRINT 130mm x 160mm Borosilicate Glass Plate Bed Flat Polished Edge w/Corners Cut for Monoprice MP Select Mini 3D Print](https://www.amazon.com/gp/product/B075XJ5ZP1) - $18.95

It is cut specifically to fit the MPSM bed, conveniently. Using a suitably sized picture frame or mirror or having a piece of
glass cut to the required dimensions is another option, but I went with this precut piece.

For bed adhesion, ordered these gluesticks:

* [Elmer's Glue Stick (E579), Disappearing Purple, 3 Sticks](https://www.amazon.com/gp/product/B00MZ5Q5QG) - $4.54

Lastly I bought this these thermal pads, though I didn't end up using them:

* [uxcell 400mm x 205mm x 0.5mm Silicone Thermal Pad for CPU GPU Heatsink](https://www.amazon.com/gp/product/B007PPEW52) - $10.07

Now that ordering the parts is out of the way, time to prepare the bed. You'll need a Z spacer to
adjust the Z limit upwards so the extruder nozzle doesn't hit the glass. I printed this one,
the last print on the dreaded BuildTak:

* [MP Select Mini Z Spacer (customizable, no disassembly!)](https://www.thingiverse.com/thing:1808029)

This Z spacer took 3.02 meters of filament, 8 grams, and completed in 1 hour 10 minutes, for an estimated cost of 15¢.

Now to install the glass bed. First remove the BuildTak with dental floss:

![Removing the BuildTak](https://user-images.githubusercontent.com/26856618/71549013-6abc6000-29ae-11ea-9673-e126cd58855f.png)

Some other guides recommended removing the adhesive with acetone and attaching the glass
to the bed with binder clips. I didn't have either, but fortunately, the Buildtak came all
off in one piece, leaving the adhesive mostly remaining on the heater block:

![BuildTak removed](https://user-images.githubusercontent.com/26856618/71549028-86276b00-29ae-11ea-9fc2-e3cc6e27692b.png)

So I stuck the glass bed directly on top of it, with the previous adhesive, and it stuck just fine! 

![Glass bed installed](https://user-images.githubusercontent.com/26856618/71549034-a5be9380-29ae-11ea-9c28-dd148349342c.png)

No tedious removal or buying extra clips needed. Easier than expected.

Your mileage may vary, but with the bed stuck on top of the original adhesive, and
after applying a thin layer of purple gluestick on top of the glass, I was able to print without any problems
(and without any build plate adhesion techniques (brims, skirts, or rafts)):

![Glass print](https://user-images.githubusercontent.com/26856618/71549039-ba029080-29ae-11ea-844f-aacf26661a9a.png)

## Bed rewire
A necessary upgrade for reliablity according to
[/r/MPSelectMiniOwners](https://www.reddit.com/r/MPSelectMiniOwners/), this upgrade
involves rerouting the bed heater and thermistor wires so they don't rub against
the housing as the bed slides along the axes. My favorite guide describing this procedure:

* [YouTube: Kevin Loughin: 3D printing - Correcting a design flaw in the Monoprice select mini, bed wire re-routing](https://www.youtube.com/watch?v=Q7CcCGBJLgc)

Printed the models linked in the YouTube video, [http://bitchen.com/kb9rlw/bed_heater_cable_mod.zip](http://bitchen.com/kb9rlw/bed_heater_cable_mod.zip):

| Model | Est filament (m) | Est filament (g) | Actual time (hh:mm) | Estimated cost |
| ----- | ---------------- | ---------------- | ------------------- | ------------- |
| cable_brace_2.stl | 0.93m | 2 | 31min | 3¢ |
| MPSM_Passive_Cooling_Panel_NO_DRILL_REMIX_-_Part_2.stl | 4.65 | 13 | 2h 32min | 21¢ |
| MPSM_Passive_Cooling_Panel_Split_-_Part_1.stl | 4.58 | 13 | 2h 56min | 21¢ |

Here's the stock sheet metal panel compared the to printed two-part panel:

![Printed side panel replacement](https://user-images.githubusercontent.com/26856618/71549143-6f821380-29b0-11ea-97b0-d1f295d536cc.png)

Opening up the case to reveal the control board:

![Control board](https://user-images.githubusercontent.com/26856618/71549161-b243eb80-29b0-11ea-967d-fe58dd56b59b.png)

Unplug the heater and thermistor cables:

![Unplug cables](https://user-images.githubusercontent.com/26856618/71549166-c4be2500-29b0-11ea-96c9-e6fa1f99e94b.png)

Unrouting the cables from the heated bed, sure eonough, they were nicked:

![Nicked cables](https://user-images.githubusercontent.com/26856618/71549174-dc95a900-29b0-11ea-8319-7df2c52cff90.png)

Should have performed this upgrade earlier, but better late than never. Could have been worse. 🔥

For the cable cover, there were cable chains you could print, or cable sleeves to buy, but
I used an old enamaled wire I had lying around, which was wound around a pencil. This fit all
the cables perfectly, and snugly attached through the cable brace:

![New cable starting](https://user-images.githubusercontent.com/26856618/71549188-24b4cb80-29b1-11ea-8060-28f4465ec829.png)

After completing the rewiring, here is the final result:

![New cable completed](https://user-images.githubusercontent.com/26856618/71549197-431ac700-29b1-11ea-8cd7-9564f09d9ded.png)

Much better! The cable now has room to safely flex without damage.

With these two upgrades, the printer is now ready for some serious business.

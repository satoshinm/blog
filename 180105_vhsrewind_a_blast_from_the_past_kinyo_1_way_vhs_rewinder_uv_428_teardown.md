# A blast from the past: Kinyo 1-way VHS Rewinder UV-428 teardown

by snm, January 5th, 2017

* auto-gen TOC:
{:toc}

When salvaging old electronics, you'll never know what you'll find. This peculiar device was used to rewind [VHS tapes](https://en.wikipedia.org/wiki/VHS):

![Rewinder dusty case](https://user-images.githubusercontent.com/26856618/34635039-08e8aa94-f23f-11e7-909b-42afe57078ac.png)

The Kinyo 1-way VHS Rewinder. In ancient times, films (and sometimes data) were stored on magnetic tapes. After watching the movie, the tape would be at the end, necessitating rewinding back to the beginning before rewatching (e.g., while watching another tape), especially on tapes from video rental stores this was good practice, hence the recommendation ["be kind, rewind"](http://www.imdb.com/title/tt0799934/):

![Be Kind Rewind](https://user-images.githubusercontent.com/26856618/34635086-843bf822-f23f-11e7-8525-fc37583c73ce.jpg)

You could rewind in a [VCR](https://en.wikipedia.org/wiki/Videocassette_recorder), too, but this standalone VHS rewinder (not to be confused with the [Microsoft SideWinder](https://en.wikipedia.org/wiki/Microsoft_SideWinder)) let you rewind tapes without a VCR. Seems of very limited use, it is not clear how much rewinding this particular device saw. I found the same model on Amazon, [Kinyo 1Way VHS #UV-428 Rewinder](https://www.amazon.com/Kinyo-1Way-VHS-UV-428-Rewinder/dp/B0073EYGZQ/), apparently it still sells for $29.99 and the box says "The World's Best Selling VHS Rewinder":

![VHS Rewinder box](https://user-images.githubusercontent.com/26856618/34635926-71c6a5e6-f24b-11e7-9f06-c92ce3342fcf.jpg)

The most recent review at the time of this writing was on August 24th, 2017 (the reviewer still uses a VCR and explains "my VCR player does not rewind anymore"), followed by another from 2015 ("Believe it or not, there are those of us who still have and use VCR's. When I discovered that I could still obtain a VHS rewinder, I knew I had to order one."). There are surprisingly many other models available, including the UV-614 RED shaped as a race car, but it was discontinued:

![UV-614 RED](https://user-images.githubusercontent.com/26856618/34635967-0417ddac-f24c-11e7-921f-779e833842e5.jpg)

Some models like the [VR-1601](https://www.amazon.com/Kinyo-Cassette-Rewinder-VR-1601-Forward/dp/B004V1NEP2) have fast forward, for the low price of $129.98, but not this one. But mine has outlived its usefulness, time to disassemble.

## Taking it apart

The back label reads: Kinyo Video Cassette Rewinder, Video Equipment, Date Code: 0496 (April 1996?), AC117V 5W 60Hz, Natural ability, Made in China. UL Listed, QA passed. What's inside? Not much:

![Rewinder inside](https://user-images.githubusercontent.com/26856618/34635144-4a1c9eac-f240-11e7-97ae-26210b4ed10b.png)

Amusingly, the current-limiting resistor (1 kΩ) is soldered directly onto the LED, flapping in the breeze, no circuit board. This LED lights up when the circuit is closed by the switch activated by closing the case and pushing a tape down, which also activates the motor which drives a belt to rewind the tape reels. A closer look at the motor:

![Rewinder motor](https://user-images.githubusercontent.com/26856618/34635191-d4bf5946-f240-11e7-8a06-4711b9dabb79.png)

Kinyo MEN-7E2-A, 27Sep96CE. This is a DC motor, measured 12.86 V across it when running in the circuit. The driver power supply circuit is trivial, consisting of a line transformer (labeled EI 35-001 91406103 HK, E84506 HEC-3396C) stepping down the mains voltage, two diodes for rectification (not four, so it is not a [full bridge rectifier](https://www.youtube.com/watch?v=sI5Ftm1-jik)? At least not the kind demonstrated by Mehdi Sadaghdar, The Grand Rectifier), and a 16 V 470 µF electrolytic capacitor:

![Rewinder circuit](https://user-images.githubusercontent.com/26856618/34635223-5c073c70-f241-11e7-87bc-54bc4223bc12.png)

## Schematic

The schematic can be easily recreated from the components, where we can see the motor is in parallel with the LED/resistor, and pressing the pushbutton powers both:

![Rewinder schematic](https://user-images.githubusercontent.com/26856618/34635745-a5de9c9c-f248-11e7-94bc-bab9b5bb0194.png)

### Rectifier

What is this 2-diode center-tap transformer configuration? I had not seen it before, but some research finds it is actually a full-wave rectifier (albeit not a full-wave *bridge* rectifier) built from a center-tapped transformer, from [ECE2204](https://filebox.ece.vt.edu/~LiaB/ECE2204/Experiments/Full_wave_rectifier/Full_wave_rectifier_transformer.pdf):

> There are several different circuit designs for full-wave rectifiers. Two common topologies (circuit designs) are a full-wave rectifier with a center-tapped transformer and a full-wave bridge rectifier. The full-wave rectifier with center-tapped transformer used two diodes while the full-wave bridge rectifier uses four diodes. In addition to the reduction in number of electronic components in the circuit, the transformer is used to step up or step down the voltage on the secondary coil. 

![Full-wave rectifier with center-tapped transformer](https://user-images.githubusercontent.com/26856618/34635801-7fa065be-f249-11e7-9373-0d1bd2eb6c85.png)

The center tap provides the return path, per [Why Center Tapped Transformers Don't Need a Bridge (4 Diode) Rectifier](http://www.tdpri.com/threads/why-center-tapped-transformers-dont-need-a-bridge-4-diode-rectifier.518644/) which explains why this works, and this 2-diode design is also used with tube rectifiers. Without a center tap transformer, you need two additional diodes (4-diodes total). Mehdi Sadaghdar in his [full bridge rectifier](https://www.youtube.com/watch?v=sI5Ftm1-jik) didn't use any transformer, instead opting to rectify the 120VAC as-is, so he needed 4 diodes, but was able to get about 170VDC on the output at about 1000W. This puny transformer and rectifier in the VHS rewinder is not nearly as powerful, but it does the trick for driving the small DC motor.

Why did the designers choose to use 2-diode + CT instead of 4-diode FWBR? Cost, no doubt. [Electro-Tech-Online Center tap transformer and 2 diode question?](https://www.electro-tech-online.com/threads/center-tap-transformer-and-2-diode-question.95899/) mvs sarma explains:

> you might appreciate , the cost saving of 2 diodes, supposing the load is 10 to 40 amps. 

> We see saving of energy being wasted across the additional 2 diodes in case of bridge.

> In full wave,the Transformer windings take partial load only during half cycle for each winding. In bridge, the winding takes load during both half cycles, thus high power transformer tends to work too hot.

So not only did they obviate the need for two extra diodes, but they can get away with using a smaller, cheaper transformer - [specifically](http://www.radio-electronics.com/info/circuits/diode-rectifier/two-diode-full-wave-rectifiers.php), a transformer √2 smaller. Win/win, for cost savings. The disadvantage is any unbalance in the two windings can cause power supply [ripple](https://en.wikipedia.org/wiki/Ripple_%28electrical%29), however since it is driving only a motor, no sensitive microelectronics of any kind, this doesn't matter.

## Salvaging

What's next for this old VHS rewinder? The parts will be extracted and repurposed. First I carefully desoldered the red LED (~1.876V voltage drop measured with multimeter diode mode) and 1 kΩ 1/4W carbon resistor, along with the red and black stranded wires attached to them:

![LED, resistor, red and black wire](https://user-images.githubusercontent.com/26856618/34636081-b34878ac-f24e-11e7-8e13-ceea8639e5c4.png)

then stashed these pieces away in my parts bin for use in other projects. Note that since the LED was wired in parallel with the motor, the circuit still works, just without the LED indicator. Also desoldered and stashed the momentary pushbutton and blue wire, then reconnected the circuit.

But the real find here is the line transformer. Measured across the two windings 21 VAC, and from center about 10.5 VAC. Recall the rectified DC voltage is about 12-13 VDC. The AC wall plug is wired into the transformer with insulated wire nuts, so by keeping this circuit intact we can have access to 21 VAC, 10.5 VAC, and 12 VDC, all from a wall outlet, for powering other projects.

## Conclusions

VHS is often used as an example of [worse is better](https://en.wikipedia.org/wiki/Worse_is_better), [compared to Betamax](https://en.wikipedia.org/wiki/VHS#Competition_with_Betamax):

> Sony's Betamax competed with VHS throughout the late 1970s and into the 1980s (see Videotape format war). Betamax's major advantages were its smaller cassette size, higher video quality, and earlier availability but

as seen in the [videotape format war](https://en.wikipedia.org/wiki/Videotape_format_war):

> Betamax is, in theory, a superior recording format over VHS due to resolution (250 lines vs. 240 lines), slightly superior sound, and a more stable image; Betamax recorders were also of higher quality construction. But these differences were negligible to consumers, and thus did not justify either the extra cost of a Betamax VCR (which was often significantly more expensive than a VHS equivalent) or Betamax's shorter recording time.

Nonetheless, VHS succeeded, but it too succumbed and has largely been relegated to the dustbins of history, just like this anachronistic tape rewinder. Engineered for extreme low cost, at least I got a line transformer out of it.
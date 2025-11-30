# Project: Active Sound System Build

## Problem Statement

I went to this concert at this guy's house, kind of a house party and performance thing. And a lot of the other people remarked that the sound was really good and they could hear things that they had never heard before and they could really hear changes in the music that they didn't expect. And that was because these speakers were high performance and tuned well and so forth. And I thought, I should have that for my music. So I talked to the person who had brought the speakers, he told me he built them 10 years ago and just carried them around to places.
And so I was thinking that I should build my own speaker set and that's because I'm interested in making my own music, modular synthesizer music but generated on CPU, because I don't actually want to deal with a lot of the hard electronic stuff. And I figure if I can't hear it well, then I need to come up with speakers that I can listen to and hear things properly.

And then I started reading about sound reproduction and I learned that home theaters are a big thing and that there are these Dolby Atmos systems that have height speakers and surround speakers and so forth. And in particular there was this [video](https://youtu.be/olaFPhZmZu0) where he did it pretty cheap. And I thought, well, I should build a home theater system that can do Dolby Atmos, because that seems like the best way to get high-quality sound reproduction. And then I learned about active crossover and how that can improve the sound quality by allowing for more precise tuning of the speakers. And so I decided that I should build an active sound system with active crossover and multiple channels of amplification.

### Near-term Goal

The focal point right now is buying a measurement microphone. So there's a Dayton Audio iMM-6C measurement microphone for about \$45 and it's available on both Amazon and Parts Express. But Parts Express gives you a 10%-12% off discount off your order, they run specials a lot of the time. But to get free shipping at Parts Express, you have to buy at least \$100 worth of stuff. And so I was thinking I should buy some speaker parts as well along with the microphone to get to the \$100. And so the issue is that I have to have a decent idea of the system design to do that, before I buy the parts, because I don't want to buy parts that are not going to be useful for the design that I end up going with.

Once I have the measurement microphone, I can start tuning my existing speakers:

* Car's sound system
* 30W Aomais Go Jr Bluetooth speaker
* Speakers on computer monitor
* Any other speakers I have lying around

I can also use the measurement microphone to measure the acoustic properties of my room, to help design the new speakers that I want to build. And then I also want to experiment with more complicated machine learning-based tuning. So overall, this is a step towards building a high-quality sound system that I can use for listening to music and also for producing music.

### Core Strategy

The core strategy for making the system high-quality is to do active, software-controlled crossover, and drive each driver with its own dedicated amplifier channel. This allows for immense control and precise tuning of the system. Active crossover can adjust for many common issues with speakers:
- flatten on-axis frequency response, correcting for driver and cabinet resonances (up to thermal/mechanical limits of the driver)
- correct relative time alignment and phase/group delay (per frequency band), improving imaging and focus and avoiding smearing
- perfectly control crossover slopes and match efficiency/sensitivity between drivers and speakers - no guesswork with passive components, just pure measurement and optimization
- limit excursions and power in problematic frequency bands
- correct for room reflections (to some extent, at the listening position)
- correct for nonlinear distortion (to some extent - with machine learning and other advanced techniques)
- apply listener-specific HRTF filtering for immersive rendering and to steer localization cues

This approach is more complex than a traditional passive crossover system, amounting to building a custom receiver, but the benefits seem worth it. The system can be tuned to perfection. Looking at the cost of parts, it even seems that with cheap DACs and class-D amplifiers, the cost difference is not that large - an active digital crossover is not really much more expensive than a passive analog one. It's important to not get ripped off by low-quality 1% THD amplifiers, but it seems like it's possible to get good quality for a reasonable price without spending a fortune for a high-end audiophile system.

However, even the physical design of the speakers cannot be ignored, even with active crossover. The drivers and the cabinet's volume, shape, and materials all affect the sound, and a poor design can limit the system's performance. DSP cannot fix physical damage or mechanical noise, and it cannot overcome severe distortion, reflections, or resonances. Essentially, if, when the speaker plays a test tone at a certain frequency and volume, there is an audible noise on top, such as an overtone, chuffing, etc., then really the DSP's only option is to avoid that frequency band for that driver, and compensate with other drivers - it is like a piano with a broken key. There are some tricks with relying on masking effects, but these are ultimately band-aids over a fundamental problem. So the speaker design still matters, and I will try to choose good drivers and cabinet designs that minimize these issues.

Budget is all over the place. I want to keep it reasonable, but I also want to get good quality. DMS did it for $1800, a guy in the comments says you can just buy a receiver and do it for $1,668. I'm aiming for something way less than that, but I don't want to skimp on quality too much. I think the most important parts are the speakers, and I've read that DIY speakers can be much better than commercial ones for the same price. But I'm also lazy, so I might buy some pre-made speakers and modify them. The amplifiers can be cheap class-D ones, as long as they have low distortion and enough power. The DACs can also be cheap, as long as they have low noise and good dynamic range. The DSP can be done on a computer, so no cost there. Overall, I think it's possible to get a high-quality system for a reasonable price, but it will require some research and careful selection of parts. And then, I'm not even sure I want to build a full 7.2.4 system right away, I'm going to start with a core 3.1 system + bass shaker, and then add more speakers later as needed. With so many options, and no hard requirements, it's hard to pin down a budget. A reference point is that a Klipsch 5.1 system is $599, but it's passive, so with the amplifier and DAC, it would be more like $800. I want to say that getting it done for under $500 is a reasonable goal, but I don't know if that's realistic.

## Speaker System Layout and Count

### Format overview

A review of modern audio formats as found on Blu-ray discs and streaming services:

* 2.0 channel LPCM 24/48 bit depth: This is a high-resolution, uncompressed stereo format. It offers the absolute highest quality for a two-channel mix, as no data is lost during compression. This is often the preferred format for music or for users who want the purest stereo sound.
* 2.0 channel LPCM (Binaural Headphone Mix) 24/48 bit depth: This is a specialized, uncompressed stereo mix designed specifically to be listened to on headphones. The "binaural" processing creates a three-dimensional sound experience by using audio cues to trick your brain into perceiving sounds as if they are coming from different locations around you.
* DSD64 2.0: A high-resolution stereo track using Direct Stream Digital (DSD) technology, which uses a 1-bit sampling method at an extremely high rate to capture audio.
* 5.1 channel Dolby Digital Plus with Dolby Atmos: A popular format for streaming services and Blu-ray discs. It uses a lossy, compressed "bed" (Dolby Digital Plus) for the 5.1 channels and then adds a layer of object-based height information (Atmos). This provides a more immersive, three-dimensional sound experience than a standard 5.1 mix.
* 5.1 channel DTS-WAV (*.dts.wav) 16/48 bit depth: This is a DTS stream packaged within a WAV file. It's often used for a surround sound mix that can be played on computers or some media players. The DTS codec is a competing lossy format to Dolby Digital, but some argue it uses less compression and therefore sounds better.
* 5.1 channel DTS-HD Master Audio 24/48 bit depth: This is a lossless, high-definition surround sound format. It is a direct competitor to Dolby TrueHD and is often found on Blu-ray discs. It's considered one of the best formats for surround sound, as the audio is bit-for-bit identical to the studio master.
* DST64 5.1: A 5.1 surround mix that uses a lossless compression method (Direct Stream Transfer) to compress the DSD data.
* DTS ES 6.1: A lossy surround sound format that adds a rear center channel to the standard 5.1 layout.
* 7.1 channel Dolby Atmos 24/48 bit depth: This is a higher-quality version of an Atmos track. It uses a 7.1 channel layout, which includes two extra side-surround channels, providing a more detailed and enveloping soundstage before the Atmos height information is added. The higher bit depth (24-bit) indicates higher fidelity than a 16-bit track.

Of these, the 7.1 channel Dolby Atmos track is the most advanced and immersive format, as it combines a high channel count with object-based audio for height information. This allows for a truly three-dimensional sound experience when played on a compatible system. Supporting this format is a key goal for a modern home theater setup.

### Surround Sound Configuration

The "AudioDome" system is a custom system with 91 KEF E301 speakers arranged in a geodesic-like sphere, and four subwoofers. They concluded in a [paper](https://pubs.aip.org/asa/jasa/article/157/4/2802/3344072/Focality-of-sound-source-placement-by-higher-ninth) that sound sources reproduced with ninth-order ambisonic panning are sufficiently focal such that two adjacent sources can be resolved at or below the level of normal human spatial acuity, and that their system was actually flawed, in that it did not reproduce high-frequency sounds correctly. The interesting graph is Figure 3/Table I, which shows that humans can discriminate 1 degree of sound up front, increasing to only 20 degrees at the sides. These are probably the best numbers available for human sound localization acuity, although there are [similar results](https://www.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.2024.1408073/full) and probably more papers.

As far as practical speaker configurations go, [this video](https://www.youtube.com/watch?v=5-wyS_E8b2A) is a good overview of Dolby Atmos speaker layouts. The most common home theater setup is 5.1, but the "gold standard" is 7.2.4, which has seven ear-level speakers (front left/center/right, left/right forward/back surround), two subwoofers, and four height speakers (front/back left/right). A 7.2.4 is set up like a normal 7.2 system but with 4 additional speakers above you (top left and right, back left and right) He says that's what the movies are mixed for, and so it's his recommendation. The angles don't matter so much if you tune the system, just that the speakers are roughly in the right layout.

In practice the number of channels really depends on the room - for a small room, 5 channels probably is all that will fit, but for a larger room, 9 channels might be better than 7, and extra heights too. The heights are more important than the surrounds and really should be prioritized if you want Atmos sound. He says a 5.1.4 system sounds better than a 7.1.2 system. The extra subwoofer is also useful for smoothing out the bass response, but for movies it's not as critical as the other speakers.

Overall, this question will depend on the room size and layout, budget, and personal preference. So for complete future-proofing, we'll design it as an incremental progression towards a 7.2.4 system - it's easy enough to build out part of it and add more speakers later, or just stop if it's too complicated or expensive.

### Incremental build plan

**Stage 1: 1.0 (Mono)**

* **Speaker:** 1 × floorstanding speaker
* **Use:** All audio through a single speaker.
* **Impact:** A surprisingly solid mono experience with your DSP. Great for ASMR and casual listening, but no soundstage yet.

**Stage 2: 2.0 (Stereo)**

* **Speaker:** Add the matching second floorstander.
* **Use:** Classic stereo soundstage.
* **Impact:** The **biggest leap for music and ASMR.** Vocals anchor between speakers, soundstage opens up. A must for EDM mixes, flute/singing bowl textures, and YouTube.

**Stage 3: 2.1 (The EDM Foundation)**

* **Speaker:** Add Subwoofer #1.
* **Use:** Subs take over deep bass, mains focus on mids/highs.
* **Impact:** The first taste of *real bass*. EDM drops hit, meditation music gains body, even YouTube audio sounds fuller. This is where your car’s stock system finally gets outclassed.

**Stage 4: 3.1 (Dialogue Clarity)**

* **Speaker:** Add center channel.
* **Use:** Anchors speech for YouTube, TV, and film. DSP tuning keeps it natural for music too.
* **Impact:** Huge for your **2.5h/day YouTube** habit. Voices now feel locked to the screen, ASMR whispers can be centered as a “presence in the room.”

**Stage 5: 3.2 (Club-Grade Bass)**

* **Speaker:** Add Subwoofer #2.
* **Use:** Dual subs smooth room modes, even bass everywhere.
* **Impact:** EDM now feels *effortless*. No dead zones or boom spots. A six-hour mix will feel consistent no matter where you sit.

**Stage 6: 3.2 + Tactile (Feel the Drop)**

* **Speaker:** Add bass shaker (on seating).
* **Use:** Wired into LFE channel, low-passed (\~20–60 Hz).
* **Impact:** Turns your chair into a “mini club stage.” Drops vibrate through your body without annoying neighbors. Optional for ASMR/meditation — you can disable it at night.

**Stage 7: 5.2 (Surround Begins)**

* **Speakers:** Add 2 × bookshelf/wall-mount surrounds.
* **Use:** Proper surround sound for Atmos music and movies.
* **Impact:** Suddenly, effects wrap around you. Not critical for EDM, but makes movie night and Atmos tracks much more immersive.

**Stage 8: 5.2.1 (Height Layer Opens)**

* **Speaker:** Add 1 × ceiling/up-firing speaker.
* **Use:** First taste of height.
* **Impact:** Feels unbalanced, but your DSP can spread it. Useful for ambient ASMR soundscapes too.

**Stage 9: 5.2.2 (Front Heights)**

* **Speaker:** Add second ceiling/up-firing speaker.
* **Use:** Left/right stereo overhead.
* **Impact:** Major upgrade. Now you can pan Atmos sounds above. Meditation/ASMR tracks with ambient effects get a more “airy” feel.

**Stage 10: 5.2.3 (Rear Height Center)**

* **Speaker:** Add third height speaker behind listening position.
* **Use:** Expands overhead bubble rearward.
* **Impact:** Still asymmetric, but DSP helps. Adds scale for Atmos music and film.

**Stage 11: 5.2.4 (Full Height Dome)**

* **Speaker:** Add fourth height speaker.
* **Use:** Now a full rectangular overhead array.
* **Impact:** Seamless 3D audio. Atmos music and immersive film finally sound “proper.”

**Stage 12: 6.2.4 (Rear Surround Begins)**

* **Speaker:** Add 1 × rear surround (center behind).
* **Use:** Completes rear directionality halfway.
* **Impact:** Like the single surround stage earlier, a bit unbalanced but tolerable.

**Stage 13: 7.2.4 (Finale)**

* **Speaker:** Add final rear surround.
* **Use:** Full 7.2.4 Atmos system achieved.
* **Impact:** Complete envelopment. The EDM rig has long been “done,” but now you have world-class Atmos immersion for everything else.

### Speaker Matching Considerations

It's important to understand that DSP and measurement can't do perceptual matching at every listening position in the room. You can get a good match at a sweet spot or two, but other locations will hear mismatched power responses. This is especially true if the speakers have very different off-axis directivity patterns - no amount of DSP can change how sound radiates into the room. 

* Match front left/right exactly: Phantom imaging, stage, and most listening focus depend on L/R symmetry.
* Match center to L/R in the 500 Hz–4 kHz range (same mid-range driver if possible)
* Surrounds: similar directivity (narrow/wide dispersion), but can be different models
* Heights: hodepodge, use whatever fits ceiling
* Subs: mix and match

### Channel count

A basic breakdown of the speakers is:

* LCR speakers (front left, center, right): 3-way, 3 x (1 woofer + 1 mid-range driver + 1 tweeter)
* Surrounds (left/right front surround, left/right back surround): 2-way, 4 x (1 woofer + 1 tweeter)
* Heights (left/right front height, left/right back height): 1-way, 4 x (1 full-range driver)
* Subwoofers: 1-way, 2 x (1 woofer specialized for deep bass)
* Exciter / tactile transducer: 1-way, 1 x (1 exciter)
* Super-tweeter (optional): 1-way, 1 x (1 super-tweeter)

The total channel count is 9 + 8 + 4 + 2 + 2 = 25 channels. The actual number may be higher or lower depending on the final design choices.

A standard A/V receiver will not have this many pre-outs or amplification channels. I will build a custom setup using multi-channel USB and amplifier boards.

## Electronics chain

### Bit depth and sample rate

The human auditory range is roughly 20 Hz–20 kHz with a potential dynamic range of about 120 dB in ideal conditions. Real-world listening, however, is limited by background noise, speaker performance, and comfort levels — nobody listens at the threshold of pain just to squeeze out a few more decibels. From this perspective, CD-quality audio (44.1 kHz sample rate, 16-bit depth) has long been accepted as sufficient for high-fidelity playback.

Digital signal processing (DSP) corrections, such as equalization and phase adjustment, are typically implemented with FIR filters. These don’t require a higher sample rate or bit depth than the source audio, so long as the internal precision of the DSP is sufficient. Professional production often uses 24-bit/96 kHz or more, but that is mostly to provide headroom for multiple processing stages and to reduce aliasing artifacts. For playback, dithering and proper gain staging already ensure that 16-bit/44.1 or 48 kHz remains transparent.

Machine learning-based DSP, which can attempt to correct nonlinear distortions in amplifiers and speakers, raises a more subtle concern. Nonlinear models can generate ultrasonic content that, in theory, could alias back into the audible range. But any well-designed playback chain includes low-pass filtering, and any amplifier or speaker that actually emitted ultrasonic noise at audible levels would be fundamentally flawed. In practice, then, even advanced DSP correction does not demand higher output rates. A 48 kHz signal suffices, and adding an ideal 48 kHz low-pass filter before the DAC would leave the audible result unchanged.

Bit depth often causes more confusion. A 16-bit DAC provides 65,536 discrete levels; at typical consumer line levels of 1–2 volts, that means each step is on the order of tens of microvolts. This is already smaller than most real-world noise sources in an audio system. While a 24-bit DAC has a theoretical dynamic range of 144 dB, practical designs rarely achieve this; even high-quality DACs are closer to 110–120 dB once noise and distortion are considered. Meanwhile, speakers, amplifiers, microphones, and rooms all have much higher noise floors and nonlinearities. In short: 24-bit precision is invaluable in production, but its benefits vanish in playback, where analog limitations dominate.

This reasoning matches what I found when I looked at the available hardware. Inexpensive stereo DAC audiophile dongles do exist, like the ES9018K2M or CX31993, but scaling them to 25 channels would require a dozen devices and an impractical number of USB endpoints. DIY boards and breakout chips also provide higher precision, but they require custom circuitry and are not plug-and-play. By contrast, the CM6206 USB audio chip offers six to eight output channels at 16-bit/48 kHz for just a couple of dollars. It fits within the USB Audio Class 1.0 standard, which avoids driver headaches, and it scales efficiently to the number of channels I need.

So both theory and practice converge: higher sample rates and bit depths don’t improve playback in a system dominated by speaker and room limitations, and the most practical, cost-effective solution — the CM6206 — already provides exactly the precision required for high-quality DSP-tuned audio. Yes, audiophile-grade DACs could sound better in theory, but the price/performance tradeoff is not favorable for a multi-channel system.

### Tools and Materials Needed

* Measurement Microphone: Dayton Audio iMM-6C $45, for acoustic measurements and tuning.
* Signal chain: computer → USB DACs → 3.5mm to something → Amp → speaker wire → speakers
* DACs: Multiple CM6206-based 6-channel USB audio interfaces to provide the required number of output channels. ~$5 each.
   * Looking at Amazon and AliExpress for options, there are various inexpensive 6-channel USB sound cards based on the CM6206 chip. This chip apparently has out-of-the-box support on Linux. However it is a "buyer beware" situation - reviews are mixed, and some boards are poorly designed or do not meet specifications.
   * There is a "blue box", "6 Channel External Sound Card USB External 5.1/7.1 Surround Sound Optical S/PDIF Sound Card Adapter for PC Laptop", using mini-USB. Cheapest [\$13.59 on Amazon](https://www.amazon.com/Channel-External-Surround-Optical-Adapter/dp/B0DLP52G7Q/), [\$4.45 on AliExpress](https://www.aliexpress.us/item/3256804491843466.html). Also comes in a [black box](https://www.amazon.com/Supports-Sampling-IEC60958-Interface-Standards/dp/B0DMTKX2M4/). [One 2021 reviewer](https://www.amazon.com/gp/customer-reviews/R2RZ1QH8EPY9NT/) did a teardown and analysis of his card, and found that it was poorly designed and did not follow the chip's specifications, with reduced input/output signal amplitude due to incorrect supply voltage, backwards electrolytic caps, and low-frequency roll-off due to incorrect input coupling caps (0.1uF vs. 10uF). Not clear if this is a one-off or a common problem. Other reviews are mixed, most say it works fine. But it only has 6 outputs, so worth avoiding in favor of the next one. Also there is [a complaint](https://www.amazon.com/gp/customer-reviews/R314YFRNE1W1AE/) that the SPDIF output is just a placeholder screw, not actually connected to anything. 
   * There is also a silver/black box, this offers 5.1/7.1. They have the jacks for 8 analog output channels (4 outs / 1 in, 5 total), but it's not clear if they actually work - some reviewers had trouble setting it to 7.1 mode. [\$14.92 on Amazon](https://www.amazon.com/Multiple-Channel-External-Converters-Playback/dp/B0F27BGBQ6/), [similar model](https://www.amazon.com/Adapter-Channel-Surround-Optical-Output/dp/B0DZTVNZ4J/), on AliExpress for [\$4.51](https://www.aliexpress.us/item/3256805779104490.html), [\$3.56](https://www.aliexpress.us/item/3256808331813720.html).
   * There is also a rounded black box, [\$24.00 on Amazon](https://www.amazon.com/YEY-Loop-Back-Support-standalone-Software/dp/B0BQ6W9N9X/), [\$18.10](https://www.aliexpress.us/item/3256805460083104.html) on AliExpress (maybe lower). This one definitely supports 7.1 output. It has a mini-USB connector and volume buttons as well. This is definitely the newest iteration of the product, I would say.
   * Then there is Cubilux 7.1 channel USB sound card, [\$29.09 on Amazon](https://www.amazon.com/Cubilux-External-Surround-Optical-Headphones/dp/B09V3H3W8L/). This one has a full-size USB connector, and it definitely supports 7.1 output. It doesn't have an SPDIF connector. It still only offers 48 kHz / 16-bit. But, the reviews are pretty much uniformly positive.
   * Overall, I do not think it is worth the premium for the rounded box, the silver adapter seems to be the best value. With 25 channels and 6 channels per adapter, I will need 5 adapters, or if they actually do support 8 channels, then 4 adapters. So the total cost will be 5*\$3.56 = \$17.80 or 4*\$3.56 = \$14.24 on AliExpress. Given the questions about build quality, having an extra adapter or two as backup is probably a good idea. Thanks to the savings of ordering from AliExpress, not a huge expense. And given that my motherboard has 10 USB ports, I can plug them all in directly without a hub, although I may experiment with a powered hub.
* DSP Unit: We will do the active crossover and tuning all on PC, so just a powerful computer connected to multi-channel USB audio interfaces. miniDSP units are not needed. And it's Linux, so no Loopback or similar software needed - it should be pretty much vanilla ALSA and PipeWire. I'm thinking I will do the DSP as a custom PipeWire plugin.
* 3.5mm to amplifier: So each adapter has 3.5mm stereo outputs. With 25 channels, I will be using 13 stereo outputs, so I will need 13 stereo 3.5mm to RCA cables, or maybe just breakout cables with 3.5mm stereo on one end and wires on the other end, which I can then connect directly to the amplifier inputs. For these I want them to be as short as possible, to avoid noise pickup. On Amazon there is a [10-pack](https://www.amazon.com/Fancasee-Replacement-Connector-Headphone-Earphone/dp/B082VW49VV/) of 3.5mm to bare wire pigtails for \$12.89. They are relatively long, 12 inches, but I can cut off the excess. For RCA cables, there is a [splitter](https://www.amazon.com/Splitter-Female-Converter-Adapter-Conversion/dp/B0F83R5NSF/) for \$3.98, and then a [2-pack](https://www.amazon.com/eBoot-3-5mm-Audio-Stereo-Adapter/dp/B06XC5KWJN/) of cables for $5.99. On AliExpress it is probably all a lot cheaper, with a wider selection, but I want to figure out the amplifier connection first. But let's go with the pigtails for now, since they are the cheapest option. So I will need 3 packs of those, for a total of \$38.67.
* Amplifiers: At least 25 channels of amplification. See below for details.
* Speakers: See below for details.
* Speaker stands or mounts: I will just use cardboard boxes or similar for now, these are not critical.
* Speaker wire: Lower gauge is better, 16 gauge is considered sufficient for runs up to 50 feet. My room is small, maximum length is probably 20 feet. There are 25 channels though. It seems most speaker wire is duplex, with two conductors per cable, so at least I don't need 50 wires, only 25. But estimate as a maximum, 500 feet of wire, which you can get for $50 on Amazon.
* Connectors: I will probably just solder or screw in the speaker wire directly to the speakers and amplifiers, but if I want connectors, banana plugs are a common choice. A 20-pack of banana plugs is [about \$10](https://www.amazon.com/HIGHROCK-Banana-Speaker-Cable-Connectors/dp/B00APVQZ8U/) on Amazon (10 pairs), and I will need at least 25 pairs, so 3 packs for \$30.
* Soldering Iron and Solder: For modifying pre-assembled speakers and making connections. These are super cheap, like [\$10](https://www.amazon.com/Soldering-Digital-Welding-Portable-Electric/dp/B08R3515SF/) for a basic soldering iron with solder included.
* Multi-meter: For checking connections and troubleshooting. (Already have one)

skip the dayton 12 channel amp and use 6 mini power amps. 30-40$ for a chifi 50wpc amp x6. Dayton MA1240a (amplifier) its 40 Watts per channel RMS at 8 ohms…  Or  "half" powered speakers as Presonus Eris E3.5 ([https://www.amazon.com/PreSonus-Eris-E3-5-Professional-Multimedia/dp/B075QVMBT9](https://www.amazon.com/PreSonus-Eris-E3-5-Professional-Multimedia/dp/B075QVMBT9?tag=hififorums-20)). Each pair is $99, totalling around $500 for 12 speakers. The same DACs and software can be used. Only an active subwoofer is needed, witch can be bought cheap. Only problem would be weight for the cealing mounted speakers. But here the passive side of the pairs can be used.

[https://www.amazon.com/Bluetooth-Digital-Amplifier-Channel-Audio/dp/B0875WY9TX/ref=sr\_1\_1\_sspa?crid=OVW2YMPWP9N8&keywords=4+channel+amplifier+home+audio&qid=1670922589&sprefix=4+channel+ampli%2Caps%2C145&sr=8-1-spons&ufe=app\_do%3Aamzn1.fos.006c50ae-5d4c-4777-9bc0-4513d670b6bc&psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFPTFBXVTYxMDdPMlImZW5jcnlwdGVkSWQ9QTA1NDI3MjE4NklIWEZYWlQxN00mZW5jcnlwdGVkQWRJZD1BMDAyNzQ0ODIzN0NWRkxOUkY0U0Umd2lkZ2V0TmFtZT1zcF9hdGYmYWN0aW9uPWNsaWNrUmVkaXJlY3QmZG9Ob3RMb2dDbGljaz10cnVl](https://www.amazon.com/Bluetooth-Digital-Amplifier-Channel-Audio/dp/B0875WY9TX/ref=sr_1_1_sspa?tag=hififorums-20)

Parts express has lots of boards using class D amplifier chips like tda7498e or tda8954th. I happen to have a large lifepo4 battery at 24v in my house, and was thinking I could skip the costly AC to DC power supplies and just hook it straight up to the 24v battery (I assume similar to car audio, but I don’t have direct experience).

### Software Challenges for spatial audio on Linux

Finding a software stack that can handle these modern audio formats, especially object-based formats like Dolby Atmos, on a Linux system is challenging. The [Cavern project](https://github.com/VoidXH/Cavern) is the closest, they have an audio rendering engine capable of handling Dolby Atmos. Cavern supports using a microphone to perform an acoustic self-calibration of the listening environment, as will be needed when using the measurement microphone to tune the new speaker system. The Cavern project is open-source so it can be freely used and modified to suit specific needs.

However Cavern only handles audio decoding/playback with Atmos. It is primarily an ahead-of-time converter, meaning that it converts audio files into a multi-channel format. Linux support is not a primary focus of the Cavern project, although there are CLI-only Linux builds. There is a newer sub-project called **CavernPipe** that acts as a real-time converter, but it is not integrated with PipeWire. The developer of Cavern does not run Linux and has expressed their opinion that Linux support isn't worth it.

The OpenAL Soft project has some support for 3D audio, so it might be possible to adapt it based on Cavern, but it is abandoned.

Overall, the software ecosystem for handling Dolby Atmos on Linux is still immature and fragmented. It seems it will require some custom development and integration work to get everything working together smoothly, probably involving forking Cavern and rewriting it from C# to C++. I don't expect to play a lot of Atmos content, but I do want to be able to support it for future-proofing and to verify that I have obtained the best possible sound experience. I think the main goal is to be able to play back stereo content with high fidelity in a solid default configuration, and then if I can also decode and play back 7.1.4 Dolby Atmos content with Cavern or a fork of it, that would be a bonus.

For super-high frequencies, I'll have to test my hearing - I might not even be able to hear above 15 kHz. But if I can, I might want to add a super-tweeter, like a ribbon tweeter, electrostatics, AMT, or even a plasma tweeter, although those are all expensive. The existing tweeters on the speakers I have might be good enough though - it's something to measure.

### Speaker Types



For the LCR speakers, custom-built cabinets are an option, but they're expensive. Coaxial or MTM designs offer better imaging.

## Amplifier Options

Possibilities:
     - Dayton Audio MA1240a 12-channel amplifier (x2) + additional 1-2 channel amp for subwoofers. $629.98 https://www.amazon.com/Dayton-Audio-MA1240a-Multi-Zone-Amplifier/dp/B003DKVZHQ

* **Sensitivity mismatch**: compensate with gain, but check amplifier headroom. Low-sensitivity speakers will require much more power for the same SPL.
* **Impedance dips & amplifier load**: ensure amp can handle the load, or place speakers on different amps where appropriate.
* **Power handling & limiting**: DSP should include excursion limits and per-band limiting for speakers that can’t take high energy at low frequencies.

## Speaker Options

* Speaker Drivers: Woofers + mid-range drivers + tweeters (x3), exciter/tactile transducer, and possibly super-tweeter
* Speaker Cabinets: 3x custom-built for LCR speakers, woodworking tools and materials (MDF, screws, glue, etc.)
* 4x pre-assembled surrounds, modifiable to remove passive crossover
* 4x pre-assembled full-range height speakers
* 2x pre-assembled active subwoofers

12X (6 pairs) Dayton B452 airs - $45 for a pair - they where chosen for a combination of price/performance/size. they have nice drivers in a crappy box with none existent crossover(its like 1 cap or something). with help, they sound better then their price. and that crappy box weighs nothing, making it easier to hang. with say magnets from the ceiling

I wanna build some speakers. 12 to be exact. The goal is to get the parts for under $300, but inflation ruins everything. Definitely has to be under $600. I'm looking at the Dayton Audio B452-AIR as a reference. Specs:
Model Number: B452-AIR
Description: 2-Way Bookshelf Speaker Pair
Woofer: 4-1/2" polypropylene cone with 4-layer coil
Tweeter: Mini AMT type with pleated Kapton diaphragm
Finish: Black ebony pica vinyl on particle board
Power Handling: 30W RMS/60W max
Impedance: 6 ohms
Response: 85 Hz to 20,000 Hz
Sensitivity: 84 dB 1W/1m
Crossover: First order high-pass
Terminals: Quick-connect spring-loaded terminals
Other: Keyhole mounting bracket
Dimensions: 9.5" H x 5.5" W x 5.7" D
Depth includes grill - Grill is removable, acoustically-transparent cloth

I'm just looking at parts express for now - I will check amazon and aliexpress later, but this is just seeing if it's feasible.

So first off that woofer - not so many 4.5 inch woofers anymore. I found this one:
Goldwood GW-204/8S 4" Shielded Poly Woofer 8 Ohm $17.98
also comes in a 4 ohm version for the same price. I honestly don't know which is better.

but, way more common is 4" woofers. so I found this:
GRS 4" Woofer Surface Mount Poly Cone 8 Ohm 4SMP-8 $14.49
again the 4 ohm version is the same price.

If I'm willing to go bigger there's also this one:
GRS 6PR-8 6-1/2" Poly Cone Rubber Surround Woofer $14.98

even an 8" for $16.98, but that might be too big to fit on my desk.

For the tweeter, the B452-AIR uses a mini AMT type with pleated Kapton diaphragm. It's also reasonable to look at the non-air version, which uses a 5/8" polycarbonate dome tweeter with ferrofluid cooling.

But I think I found a reasonable one:
Dayton Audio PTMini-6 Planar Tweeter 6 Ohm $10.98

the description says it's a ribbon of heat-resistant Kapton material and an etched aluminum conductor, which sounds similar to the AIR.

As far as the particle board, I have no idea about how thick the Dayton speakers are, but you can get a 3/4-in x 4-ft x 8-ft Pine Sanded Particle Board for $38 from lowe's. You can do the math to see how many speakers you can get out of that. I don't feel like adding a vinyl finish, but I maybe I can spraypaint it or something.

So, crunch the math and see, what's the total cost for 12 speakers?

Klipsch VB-15s 2 for $40
These bookshelf speakers deliver rich, powerful sound through 5-1/4" fiber-composite lightweight cone woofers and 1" aluminum dome tweeters. Handles up to 300 watts peak and 75 watts RMS (continuous) power @ 8 ohms. Tractrix horn technology. Bass-reflex MDF enclosure. 92.5dB sensitivity (although Klipsch is known for inflated sensitivity numbers). 8 ohms impedance. 

Pioneer (model unknown, 80s) 2 for $60
15-inch woofers

Pioneer CS-C9000 x 2 + sony center speaker (unknown model) for $150
Type: 3 way, 3 driver loudspeaker system
Frequency Response: 32Hz to 20kHz
Max Music Power: 150W
Crossover Frequency: 3000, 6000Hz
Impedance: 6.3Ω
Sensitivity: 90dB
Bass: 1 x 300mm cone
Midrange: 1 x 120mm cone
Tweeter: 1 x 66mm cone
Enclosure: bass reflex
Dimensions: 380 x 800 x 330mm

Pioneer CS-R580 3-way speaker system 2 for $100
6 ohms 150 watts
forums say "hard pass"

Sanyo SS V6-150 Floor standing speakers 2 for $50
max power 150 watt
impedance 8 OHM
3 drivers with a big woofer (8")
5" and 3" Mid drivers are just a simple paper cone type, they're nothing to write home about.
The enclosure is in fact particle board with a Laminated vinyl finish.
Not much insulating material inside. Simple low & high pass filter network.
part of an 'all-in-one' rack system, produced from 1987-1988. 


Sensitivity dB at 1 watt/1 meter
Impedance ohms (Ω)
RMS Power (watts)
Peak Power (watts)

Theater Logic T2400
3 way floor standing speaker with 8" side-firing subwoofer
impedance: 8 ohms
minimum power: 20 watts RMS
peak power: 200 watts "music power"
sensitivity: 92 dB at 1 watt per 1 meter

Tweeter: 1" titanium ferrofluid
Midrange: 5.25" x 2
Woofer: 8" modulus cone
Frequency response: 30-25k Hz
Crossover Frequency: 800, 5kHz
Tuned ported cabinet
Digital Ready
Dimensions: 9.25" w x 14.37" d x 40.95" t

left/right matched set

    The Drivers: The individual drivers (woofer, midrange, and tweeter) are likely the cheapest components available from bulk manufacturers in Asia. A reasonable estimate for their bulk cost is a few dollars each. So, a total of 6 drivers across two speakers might be worth about $15 to $20 in raw parts.

    The Cabinet: The cabinet is made of thin particle board with a cheap vinyl wrap. The cost of materials and assembly is very low. You might be able to get $5-$10 worth of wood out of it.

    The Wiring and Terminals: The wiring is likely thin gauge, and the terminals are cheap spring-clips. Maybe a dollar's worth of parts.

    The "Crossover": It's likely just a single capacitor and a cheap coil, adding less than a dollar to the cost.
KEF 5.2

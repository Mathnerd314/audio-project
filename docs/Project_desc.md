# Project: Active Sound System Build

## Problem Statement

I went to this concert at this guy's house, kind of a house party and performance thing. And a lot of the other people remarked that the sound was really good and they could hear things that they had never heard before and they could really hear changes in the music that they didn't expect. And that was because these speakers were high performance and tuned well and so forth. And I thought, I should have that for my music. So I talked to the person who had brought the speakers, he told me he built them 10 years ago and just carried them around to places.
And so I was thinking that I should build my own speaker set and that's because I'm interested in making my own music, modular synthesizer music but generated on CPU, because I don't actually want to deal with a lot of the hard electronic stuff. And I figure if I can't hear it well, then I need to come up with speakers that I can listen to and hear things properly.

## Near-term Goal

The focal point right now is buying a measurement microphone. So there's a Dayton Audio iMM-6C measurement microphone for about \$45 and it's available on both Amazon and Parts Express. But Parts Express gives you a 10% off discount off your order, they run specials a lot of the time. But to get free shipping at Parts Express, you have to buy at least \$100 worth of stuff. And so I was thinking I should buy some speaker parts as well along with the microphone to get to the \$100. And so the issue is that I have to have a decent idea of the system design to do that, before I buy the parts, because I don't want to buy parts that are not going to be useful for the design that I end up going with.

Once I have the measurement microphone, I can start tuning my existing speakers:

* Car's sound system
* 30W Aomais Go Jr Bluetooth speaker
* Speakers on computer monitor
* Any other speakers I have lying around

I can also use the measurement microphone to measure the acoustic properties of my room, to help design the new speakers that I want to build. And then I also want to experiment with more complicated machine learning-based tuning. So overall, this is a step towards building a high-quality sound system that I can use for listening to music and also for producing music.

I recognize that it's a foolish framing to predicate buying a microphone on designing a custom sound system from scratch, simply for the $5 discount on Parts Express, where otherwise I could just buy the microphone from Amazon and be done with it. It's more like, I have a sneaking suspicion that I will end up buying more parts from Parts Express anyway, for example because I think my car's speakers have gone bad and I want to replace them, not just tune them. So I might as well get the microphone from Parts Express and save a few bucks that way.

## Software stack for spatial audio on Linux

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

### Software Challenges

Finding a software stack that can handle these modern audio formats, especially object-based formats like Dolby Atmos, on a Linux system is challenging. The [**Cavern** project](https://github.com/VoidXH/Cavern) is the closest, they have an audio rendering engine capable of handling Dolby Atmos. Cavern supports using a microphone to perform an acoustic self-calibration of the listening environment, as will be needed when using the measurement microphone to tune the new speaker system. The Cavern project is open-source so it can be freely used and modified to suit specific needs.

However Cavern only handles audio decoding/playback with Atmos. It is primarily an ahead-of-time converter, meaning that it converts audio files into a multi-channel format. Linux support is not a primary focus of the Cavern project, although there are CLI-only Linux builds. There is a newer sub-project called **CavernPipe** that acts as a real-time converter, but it is not integrated with PipeWire. The developer of Cavern does not run Linux and has expressed their opinion that Linux support isn't worth it.

The OpenAL Soft project has some support for 3D audio, so it might be possible to adapt it based on the  Cavern, but it is abandoned.

Overall, the software ecosystem for handling Dolby Atmos on Linux is still immature and fragmented. It seems it will require some custom development and integration work to get everything working together smoothly, probably involving rewriting Cavern from C# to C++ or Rust.

## Surround Sound Configuration

The "AudioDome" system is a custom system with 91 KEF E301 speakers arranged in a geodesic-like sphere, and four subwoofers. They concluded in a [paper](https://pubs.aip.org/asa/jasa/article/157/4/2802/3344072/Focality-of-sound-source-placement-by-higher-ninth) that sound sources reproduced with ninth-order ambisonic panning are sufficiently focal such that two adjacent sources can be resolved at or below the level of normal human spatial acuity, and that their system was actually flawed, in that it did not reproduce high-frequency sounds correctly. The interesting graph is Figure 3/Table I, which shows that humans can discriminate 1 degree of sound up front, increasing to only 20 degrees at the sides. These are probably the best numbers available for human sound localization acuity, although there are [similar results](https://www.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.2024.1408073/full) and probably more papers.

As far as practical speaker configurations go, [this video](https://www.youtube.com/watch?v=5-wyS_E8b2A) is a good overview of Dolby Atmos speaker layouts. The most common home theater setup is 5.1, but the "gold standard" is 7.2.4, which has seven ear-level speakers (front left/center/right, left/right forward/back surround), two subwoofers, and four height speakers (front/back left/right). He says that's what the movies are mixed for, and so it's his recommendation. For a larger room, 9.2.4 or 9.2.6 is better, but that's mainly because of the larger room size. The angles don't matter so much, just that the speakers are roughly in the right places.

The extra surround speakers (left/right back surround) help with the "envelopment" effect, making it feel like you're in the middle of the action, but the extra height speakers are more important for forward/back, so prefer 5.1.4 to 7.1.2. The extra subwoofer helps smooth out the bass response, so it's more a personal preference.

Adding extra speakers beyond 7.2.4 is diminishing returns, and the extra cost and complexity isn't worth it, particularly with a small room. The rear surround speakers are also hard to get right, because they can be blocked by furniture or walls, so cutting it doesn to a 5.2.4 system is worth considering. A 7.2.4 system is designed for several listeners, so if it's just for one person, a 5.2.4 system is probably fine, but the extra surround speakers do give flexibility and robustness. Overall, this question will depend on the room size and layout, budget, and personal preference. For now I'm leaning towards a 7.2.4 system, and we'll see how it goes - if it's too complicated or expensive, we can always scale back, and it's easy enough to build out part of it and add more speakers later.

However this doesn't the full picture as there is also the range of frequencies that each driver can handle. A surround system handles a good range of frequencies, but even high-fidelity speakers struggle with very low frequencies (sub-bass) and very high frequencies (super-tweeters). For super-low frequencies, I'm thinking I want an exciter or tactile transducer to give that "feel" of the bass, because sub-bass is more about feeling than hearing and certainly a lot of the music I listen to has a lot of sub-bass. For super-high frequencies, I'll have to test my hearing - I might not even be able to hear above 15 kHz. But if I can, I might want to add a super-tweeter, like a ribbon tweeter, electrostatics, AMT, or even a plasma tweeter, although those are all expensive. The existing tweeters on the speakers I will buy/build might be good enough though - it's something to measure.

### Core Strategy

The core strategy for making the system high-quality is to do active, software-controlled crossover, and drive each driver with its own dedicated amplifier channel. This allows for immense control and precise tuning of the system, avoiding many potential distortion and phase issues. For example, achieving a perfectly flat frequency response is possible, and time-domain issues that cause sound to be smeared can be corrected. This approach is more complex and expensive than a traditional passive crossover system, requiring building a custom receiver, but the benefits are worth it for a high-end system. There's no need to compromise on sound quality, and the system can be tuned to perfection. However, the physical design of the speakers cannot be ignored, even with active crossover. The driver and the cabinet's volume, shape, and materials all affect the sound, and a poor design can limit the system's performance.

To go back to the 7.2.4 system, here's the breakdown of the speakers:

* Front Left, Center, and Right (3 Speakers): These are the most critical speakers. The center channel, in particular, should be robust for dialogue clarity, meaning a mid-range driver. The target is 3-way crossover speakers, so with 1 woofer, 1 mid-range driver, and 1 tweeter. These will use custom-built cabinets and drivers.
* Surround and Rear Surrounds (4 Speakers): These speakers handle ambient and directional effects. They are a simpler design, 1 woofer and 1 tweeter, and all four should be matched for sound field consistency. The goal here is to find a good quality pre-assembled speaker, for example with a coaxial design, and then modify them by removing the passive crossover network (big circuit board in the back - will have to check how easy it is to remove).
* Overhead/Height Speakers (4 Speakers): These provide the vertical dimension for Atmos effects. They can be a simple design, a full-range driver, so the goal here is to find 4 good quality pre-assembled speakers and use them as-is, ceiling-mounted. Again they should be matched for consistency.
* Subwoofers (2 Units): Each subwoofer is a dedicated unit, containing 1 woofer specialized for deep bass ("subwoofer driver"). These can be pre-built enclosures with the driver and amplifier included, and again the goal is to find two good quality pre-assembled subwoofers, preferably matched.

And then we have our exciter / tactile transducer, and possibly a super-tweeter, depending on my hearing test results. So the channel count is:

* 3 x LCR speakers (woofer + mid + tweeter) = 9 channels
* 4 x surrounds (woofer + tweeter) = 8 channels
* 4 x heights (full-range) = 4 channels
* 2 x subwoofers (active, so they have their own amps) = 2 channels
* Extra channels for exciter and super-tweeter = 2 channels
* Total: 9 + 8 + 4 + 2 + 2 = 25 channels.
 
A standard A/V receiver will not have this many pre-outs or amplification channels. I would need several, probably a 16-channel amplifier and then a second 10-channel amplifier. But I'm planning a custom electronics setup with direct PC control. I'll need to ensure these channels can all be controlled independently, with signals sent from the PC to each channel. This is a complex setup, but it allows for precise control and tuning of the system.

### Maximum Volume and Loudness

The goal is to be able to "feel" the music at concert levels. Concerts can be very loud, such as 120 dB SPL or more, but this is not a safe listening level for extended periods. At the concerts I attend, the speakers are indeed far too loud for comfort, and I wear earplugs and never closely approach the speakers. The earplugs have a rating of NRR 25dB, and the distance is about 250 feet, which also reduces the sound level by `20 * np.log10(250 / 25) = 20 dB` or so. So the effective loudness with earplugs at that distance is `120 - 25 - 20 = 75 dB`, which is a comfortable listening level. We see similarly that OSHA states to activate precautions above 75 dB(A). So listening at 75 dB is comfortable and safe, and we can set that as our target volume.

But this isn't the whole story, because we also have to consider the volume by frequency. Earplugs block high frequencies better than low frequencies, and distance also attenuates high frequencies more than low frequencies. This suggests that at the concert, the low frequencies (bass) are still quite loud in sound pressure terms, even with earplugs and distance. But, looking at the equal loudness contours, we see that in fact the ear is less sensitive to low frequencies, so even if the bass is loud in sound pressure terms, it is't perceived as loud. And similarly with the A-weighting, low frequencies don't count towards the loudness. So effectively, this combination of loud concert, earplugs, and distance is just to get a good bass response, and the actual perceived loudness is still around 75 dB. Based on this, I can set my subjective maximum volume to be around 80 phons, which is roughly equivalent to 75 dB at 1 kHz, and then adjust the maximum volume at other frequencies based on the equal loudness contours.

There's also dynamic range to consider. Music has a wide dynamic range, with quiet passages and loud passages. Even if 75 dB is a comfortable average, loud passages can exceed this. So I should allow for some headroom. Dolby standards are for 120 dB SPL, which seems excessive for home listening. I still haven't fully worked this out.

### Power and wattage

The goal is to build a high-quality home theater system for concert-level volume on a 15-amp circuit in a 13' x 10' x 8' room with a popcorn ceiling. Currently my PC is on the same circuit, but I plan to run a heavy-duty, 12-gauge extension cord from an outlet in the adjacent room (which is on a separate circuit) to isolate the power draw of my PC and other devices from my audio system, to maximize the available power for the audio equipment and ensure a clean circuit. A 15-amp circuit in a 120-volt system provides a maximum of 1,800 watts (15 amps x 120 volts), but the National Electrical Code recommends only loading a circuit to 80% capacity for continuous use, leaving you with about **1,440 watts of available power.** 

### Speaker and Amplifier Selection

Your goal of "feeling" the music at concert levels means you need a system capable of very high Sound Pressure Levels (SPL). The SPL at a typical concert can exceed 120 dB. , you can achieve very high SPLs with less power by choosing **highly sensitive speakers**.

* **Speaker Sensitivity:** A high-sensitivity speaker requires less power to produce the same volume. Look for speakers with a sensitivity rating of **92 dB or higher** (measured at 1 watt at 1 meter). This is at the higher end of consumer speakers, but it's crucial for your power-constrained setup.
* **Amplifier Wattage:** To achieve a 120 dB SPL at a 12-foot listening distance, you would need roughly 500 watts per channel for a speaker with 92 dB sensitivity.
* **Popcorn Ceiling:** Popcorn ceilings are often called "acoustic ceilings" because their textured, porous surface helps **diffuse and absorb sound**, which is actually a benefit. This will reduce echo and reflections, making the sound clearer.

### Power Breakdown

Given your 1,440-watt power budget, here's a potential breakdown for your 7.2.4 system:

* **LCR (Front Left, Center, Right) Speakers:** These are the most critical channels. You'll need the most power here. Using high-sensitivity speakers, you could allocate **200 watts** per channel. (600 watts total)
* **Surround and Rear Speakers (4 channels):** These don't need as much power since they primarily handle ambient sounds. You could use speakers that are a bit smaller or have lower power handling, perhaps **50 watts** per channel. (200 watts total)
* **Height/Ceiling Speakers (4 channels):** These also don't need a lot of power. They could be rated at **50 watts** per channel. (200 watts total)
* **Subwoofers (2 channels):** To "feel" the music, subwoofers are key. Since they have their own amplifiers, their power is separate from your receiver's channels. However, a high-power subwoofer can consume a lot of electricity. You will need to choose subwoofers with a lower continuous power draw. Aim for two subwoofers with **150 watts** of continuous power each. (300 watts total)

Total wattage for this configuration:
> LCR (600W) + Surrounds (200W) + Heights (200W) + Subwoofers (300W) = **1,300 watts**

This leaves you with about 140 watts for your A/V receiver, gaming console, and other devices, which is a comfortable margin.

### Budget Considerations

This [deal](https://slickdeals.net/f/18618031-klipsch-speakers-r-610f-pair-r-41m-pair-r-52c-r-12sw-599-or-w-denon-avr-s670h-899-free-s-h) was $599 for 5 speakers and a subwoofer. It didn't include a receiver or anything, but the price did stick in my mind as a reasonable budget for a mid-range speaker system. So I am aiming for maybe like $600 - $700 for the whole thing. This is apparently a pretty tight budget for a 7.2.4 system, but I mainly am focusing on getting good LCR speakers and subwoofers, and then my plan was to go to thrift stores and find used speakers for the surrounds and heights, which are less critical. So I think it's doable, but I will have to be careful with my choices. And I kind of expect to go over budget, the real budget is like $1500 or so, but I want to try to keep it as low as possible.

### Tools and Materials Needed

* Measurement Microphone: Dayton Audio iMM-6C or similar, for acoustic measurements and tuning.
* Multi-Channel Amplifiers: At least 25 channels of amplification, possibly using two or three separate amplifiers (e.g., a 16-channel and a 10-channel).
* DSP Unit: A mini PC with a real-time Linux kernel, running custom software for active crossover and tuning. (if not using PC directly)
* Speaker Drivers: Woofers + mid-range drivers + tweeters (x3), exciter/tactile transducer, and possibly super-tweeter
* Speaker Cabinets: 3x custom-built for LCR speakers, woodworking tools and materials (MDF, screws, glue, etc.)
* 4x pre-assembled surrounds, modifiable to remove passive crossover
* 4x pre-assembled full-range height speakers
* 2x pre-assembled active subwoofers
* Speaker Wire and Connectors: Sufficient length and quality for all speaker connections.
* Soldering Iron and Solder: For modifying pre-assembled speakers and making connections.
* Multi-meter: For checking connections and troubleshooting. (Already have one)

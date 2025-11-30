Look into DIY building speakers. The library here has a ShopBot Desktop CNC (plastic or wood, up to 24" x 18" x 2.5"), available for use at no cost, and there is also Pikes Peak Makerspace which offers access to a PRO4896 4' x 8' CNC Router Machine for $55. The goal would be to produce speakers comparable to the Micca MB42X, currently priced at $100 / pair.

review solutions for driving DIY speakers with a PC. focus on specific chips, probably will need 3:
- usb chip, like CM6206 (but that's old), or maybe something which outputs i2s or some similar digital audio format
- dac chip, like the Cirrus Logic CS42526, or Texas Instruments PCM4202
- amplifier chip, like the TPA3116D2 or semi-knockoff CS8673E

I see really two options:
- order a custom PCB
- assemble from breakout boards, there are some cheap ones on aliexpress
Investigate both options.
Regarding the enclosure, it will probably just be cardboard, don't pay too much attention to that.

I want to drive DIY speakers with a PC. The design is to have active crossover with my computer, with each driver controlled individually. For a full 7.1.4 Atmos system, I am estimating about 25 audio channels, but I can start with a smaller number for prototyping. As I see it, USB is the best I/O from my PC, I don't have enough audio jacks. But on the other hand, even a basic USB 2 connection has enough bandwidth for hundreds of audio streams. I am thinking I need specific chips for USB, DAC, and amplifier functions.

There are two main approaches I see to building this system:
1. Assemble from breakout boards or modules available cheaply on marketplaces like AliExpress.
2. Order a custom PCB (or several) that integrate these chips, preferably pre-assembled, probably also from China.
Investigating both options is worthwhile. The pre-made boards are very cheap, but may not have the best performance or features. Custom PCBs can be tailored to my needs, but will require more upfront design work and potentially higher costs due to the low volume.

Sample chips I am considering, explore the market and make a comprehensive recommendation:
- The CM6206 USB audio chip is older but widely seen in cheap USB sound cards. Supports 16-bit / 48kHz audio and basic features.
- Newer USB audio interface chips may output I2S or other digital audio formats/connectors, but this requires further research to identify suitable options.
- **Cirrus Logic CS42526**: A high-performance DAC known in professional audio for clear, neutral sound quality.
- PCM1798, CS4398: less expensive Digital-to-Analog Converter options
- **TPA3116D2**: A popular Class D amplifier chip offering clean power output, well-reviewed in DIY boards. Typically powers 20-30W per channel at 8 ohms, efficient with good audio quality. Often comes on cheap boards for DIY use. Requires correct inductors and filter caps for optimal response.
- **CS8673E**: A less common but favored alternative for high-quality sound, apparently with very clean audio output, good for mid-power applications. Very cheap boards on AliExpress.

Also I need power supplies, but those are easy to find. Regarding power requirements, they are all over the map, some channels will drive tweeters, some midrange, some woofers. I am thinking of 20-30W per channel as a reasonable average, but I can adjust based on actual speaker designs later. I might do one box with everything, or one box for the USB, and separate boxes for the DAC and amplifiers. It sort of depends on whether it is easier/cheaper to run lots of speaker wires or lots of digital signals, without excessive interference.
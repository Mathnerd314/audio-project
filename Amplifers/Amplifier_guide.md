# Amplifier buying guide

## SINAD and THD+N measurements

When evaluating audio amplifiers, the main metric reported is the **THD+N** (Total Harmonic Distortion plus Noise) or its related metric, **SINAD** (Signal to Noise and Distortion). These measurements are directly convertible using the formula: `sinad_db = -20*log10(THDN)`, where `THDN` is expressed as a decimal (e.g., 0.01 for 1%). [ref](https://www.analog.com/media/en/training-seminars/tutorials/MT-003.pdf) For example, a SINAD of 60 dB corresponds to a THD+N of 0.1%, a SINAD of 80 dB corresponds to a THD+N of 0.01%, and a SINAD of 100 dB corresponds to a THD+N of 0.001%. These numbers are both measured a specific test signal, typically a 1 kHz sine wave, and adjusted to a specific output power level, such as 5 watts into 4 ohms, to provide a standardized basis for comparison.

Definitions:
\[
\begin{align*}
\text{SINAD} & = -10 * \log_{10}\left( \frac{P_\text{noise} + P_\text{distortion}}{P_\text{noise} + P_\text{distortion} + P_\text{signal}}\right) \\
\text{THD+N} & = \frac{\text{rms}(\text{noise + distortion})}{\text{rms}(\text{reference})}
\end{align*}
\]
P is the average power of the signal, noise and distortion components. For THD+N the reference can be either the fundamental frequency (THD_F) or the total signal including harmonics (THD_R, corresponding to SINAD).

The actual measurement involves capturing the output signal, applying a notch filter to remove the fundamental frequency, and then calculating the RMS of the remaining signal (noise + distortion) relative to the original signal. The measurement is typically done using a high-quality audio analyzer and a test jig that includes a known load (e.g., 4 ohms), and must have sufficient bandwidth and dynamic range to accurately capture low levels of distortion and noise. If you want noise and distortion separately, then there are other measurements like SNR (Signal to Noise Ratio) and THD (Total Harmonic Distortion) that can be reported, but these are less common in amplifier reviews.

SINAD combines electrical measures of noise and distortion into a single measurement primarily for convenience. However, perceptual noise and distortion have different characteristics, and neither corresponds well to SINAD, or even to the electrical measures. The main thing SINAD measures is how much engineering effort the designers have put into reducing SINAD. But it can be concluded from a THDN measurement that an amplifier will have electrical noise and distortion less than the measured value, and this can in turn lead to some conclusions about the audibility of noise and distortion.

## Distortion

Distortion is the alteration of the original signal waveform, arising from non-linearities in the correspondence of input to output. Total harmonic distortion (THD) is a measurement of the harmonic distortion present in a signal and is defined as the ratio of the sum of the powers of all harmonic components to the power of the fundamental frequency, $-20 \log_{10}(\frac{D}{S})$. THD has been criticized as a metric, because the human ear is less sensitive to higher-order harmonics. For example, a 20th-order harmonic (20 kHz) of a 1 kHz tone will be inaudible, but will likely still be included in THD. The ear is also less sensitive to distortion with complex musical signals, compared to a pure tone. This means that a high THD measured with a sine wave may not translate directly to perceived poor audio quality in music. Several studies have proposed a "perceptual distortion" metric that applies a windowing function to weight the distortion components based on their audibility, but this is not commonly used in amplifier reviews.

Speakers are far less efficient and far more prone to non-linear behavior than modern amplifiers.  They're converting electrical energy into physical motion, and that process is inherently messy. That being said, distortion in amplifiers can still be significant, with poor design or as the amplifier approaches its clipping point. Consider this example:

* The Revel F328Be floor standing speaker, retailing for $16,000/pair, has measured harmonic signals of 30 - 50 dB SPL at a test signal level at 96 dB SPL. This corresponds to a THD of 0.5% to 0.05%. This is an extremely low distortion level for a speaker, reflecting the high quality of the speaker design and components.
* The Facmogu S-288 is one of the cheapest amplifiers available, costing around $20. It advertises a harmonic distortion of 0.4%, corresponding to a SINAD of about 48 dB. This is a relatively high distortion level for an amplifier.

There are different ways to estimate the combined distortion levels from different components in the audio chain. A common method is to use a root-sum-square (RSS) approach, which assumes that the distortion contributions are uncorrelated. However, in practice, audio distortion is significantly correlated - the same signal is being distorted by multiple components. In this case, a simple sum of the distortion levels may be more appropriate. Considering the above example, this would give 0.45% to 0.9% distortion with the simple sum, or 0.40% to 0.64% with the RSS method.

Considering a more typical amplifier, it might have a THD+N of 0.01% (equivalent to 80 dB SINAD). When combined with the speaker distortion of 0.5% to 0.05%, the total distortion becomes 0.06% to 0.51% using the simple sum, or 0.05% to 0.50% using the RSS method. In this case, the amplifier's distortion contribution is negligible compared to the speaker's distortion. A simple rule of thumb is that distortion below 1/10 of the speaker's distortion is inaudible, so an amplifier with a THD+N of 0.1% (60 dB SINAD) would be sufficient for most speakers. However, many reviewers and audiophiles prefer much lower distortion levels (e.g., 0.01% or 80 dB SINAD) for high-fidelity listening.

## Noise

Perceptually, the key metric for noise is the noise level at the listening position. This can be compared to the listener's hearing threshold, and to masking at the ambient noise level, which are both frequency dependent. The audible threshold can vary significantly between individuals and listening environments. A typical quiet room might have an ambient noise level of around 30-40 dB SPL, while a very quiet room could be as low as 20 dB SPL. The human hearing threshold varies with frequency, being most sensitive around 2-5 kHz, where it can be as low as 0 dB SPL for young, healthy ears. At lower frequencies (e.g., 20 Hz to 200 Hz), the threshold can be much higher, often around 20-30 dB SPL or more.

Electrically, the noise floor can be most simply modeled as a constant power level, although factors like thermal noise, mechanical noise, and electronic hiss all contribute to the overall noise and hence frequency, temperature, signal level, and other factors can affect the actual noise floor. In practice, the noise floor of an amplifier is often specified in terms of the **Signal-to-Noise Ratio (SNR)**, which is a log transform of the power of the noise divided by a specific output power level. If it is not specified, the most likely reference point is the amplifier's maximum rated power (continuous RMS, e.g. 50W). This is also a conservative estimate, as postulating a lower reference like 1W would be postulating a much better amplifier.

It's reasonable to estimate noise for typical listening scenarios by calculating the noise level at the speaker using its sensitivity and the power level of the noise, then applying the inverse square law to estimate the volume at the listening position. This gives the formula:

\[
L_\text{noise} = S + 10 \log_{10} \left( \frac{P_\text{SNR}}{P_s} \right) - SNR - 20 \log_{10} \left( \frac{d_\text{listening}}{d_s} \right)
\]

where:

* **$L_\text{noise}$**: The final noise level in dB SPL.
* **$S$**: Speaker sensitivity in dB SPL at a reference power ($P_s$) and distance ($d_s$).
* **$P_s$**: The reference power for sensitivity, typically 1 watt.
* **$d_s$**: The reference distance for sensitivity, typically 1 meter.
* **$P_\text{SNR}$**: The signal power in watts at which the SNR was measured, typically the amplifier's maximum rated power.
* **$SNR$**: The given signal-to-noise ratio in dB.
* **$d_\text{listening}$**: The listening distance in meters.

## Table of amps

| Amplifier | SINAD (db) | Price | Channels | Price/channel |
| --- | --- | --- | --- | --- |
| Topping B100 | 120 | $299.00 | 1 | $299.00 |
| Topping LA9O | 120 | $799.00 | 2 | $399.50 |
| Topping LA9O Discrete | 120 | $799.00 | 2 | $399.50 |
| Topping B200 | 120 | $599.00 | 1 | $599.00 |
| Topping PA5 II | 106 | $219.00 | 2 | $109.50 |
| TP RA3 | 105 | $229.00 | 2 | $114.50 |
| 3e-audio A5 | 105 | $249.99 | 2 | $125.00 |
| 3e audio 260-2-29A | 102 | $149.00 | 2 | $74.50 |
| 3E Audio 480-1-29A | 102 | $129.00 | 1 | $129.00 |
| Fosi Audio V3 Mono | 100 | $109.99 | 1 | $109.99 |
| AIYIMA A70 | 95 | $199.00 | 2 | $99.50 |
| JLESYLPH-D200 | 94 | $130.00 | 2 | $65.00 |
| AIYIMA A70 Mono | 94 | $129.99 | 1 | $129.99 |
| SONOS Amp Digital In | 92 | $149.00 | 2 | $74.50 |
| Yamaha WXA-50 RCA | 90 | $35.00 | 2 | $17.50 |
| Extron XPA1002 Plus | 85 | $50.00 | 2 | $25.00 |

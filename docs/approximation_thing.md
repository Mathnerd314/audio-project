---
title: Ideal Filtering
---

# Approximation of continuous-time filtering via discrete convolution after ideal reconstruction

Nice — this is a great line of thought. Short answer up-front: **yes, it’s mathematically tractable and (under the usual bandlimited assumption) equivalent to a discrete convolution with the continuous filter sampled at the sampling instants.** Whether it’s efficient in practice depends on how fast that sampled impulse decays (which you can exploit), and on whether you prefer block FFT methods or a truncated FIR / IIR fit. Below I walk through the math, the equivalence, time-decay behavior, and practical implementation/optimization strategies (with concrete tradeoffs).

---

# 1) The mathematics / exact equivalence

Let $T=1/F_s$ and let $x[n]=x_c(nT)$ be samples of a continuous-time, bandlimited signal $x_c(t)$ (bandlimit $|f|<F_s/2$). Reconstruct with ideal sinc interpolation

$$
x_c(t)=\sum_{m} x[m]\,\operatorname{sinc}\!\big(\tfrac{t-mT}{T}\big).
$$

Apply a continuous LTI filter with impulse response $h_c(t)$ (frequency response $H_c(f)$), and then sample:

$$
y[n]=\big(y_c(t)\big)_{t=nT}=\left.\big(h_c * x_c\big)(t)\right|_{t=nT}.
$$

Because convolution is linear and sampling commutes with convolution, you can show

$$
y[n]=\sum_m x[m]\; g[n-m],\qquad g[k]=T\,h_c(kT).
$$

(That factor $T$ depends on your definitions of Fourier transform conventions; the key point is $g[k]$ is the continuous impulse sampled at integer multiples of $T$.)

So the entire continuous → continuous → sample pipeline is **exactly** equivalent to a discrete convolution with the (usually infinite-length) sequence $g[k]$. In other words: applying an arbitrary continuous-time LTI filter to an ideal-reconstructed bandlimited signal is the same as convolving the sample sequence with the sampled continuous impulse response.

---

# 2) Frequency-domain viewpoint (why no aliasing if bandlimited)

If $H_c(f)$ is supported (or you only care about it) for $|f|<F_s/2$, then there is no aliasing when sampling $y_c(t)$. In frequency terms the discrete-time frequency response (DTFT) of $g[k]$ equals the continuous frequency response $H_c(f)$ restricted to the baseband and mapped to digital frequency:

$$
G(e^{j\omega}) \big|_{\omega=2\pi fT} \;=\; H_c(f),\qquad |f|<F_s/2,
$$

so multiplication by $H_c(f)$ in continuous-time corresponds exactly to multiplication by $G(e^{j\omega})$ in discrete-time (no aliasing) and you recover the identical sampled output.

Thus designing $h_c(t)$ for the continuous magnitude you specified and then sampling it gives a discrete-time filter whose effect at the sample points is exactly the continuous one.

---

# 3) Impulse decay / how long is $g[k]$?

Important practical question: is $g[k]=T\,h_c(kT)$ very long (intractable), or does it decay fast so it can be truncated?

Time-domain decay of the sampled impulse depends on the smoothness of $H_c(f)$:

* If $H_c(f)$ is infinitely smooth inside the band, $h_c(t)$ decays super-fast (faster than any polynomial), so sampled $g[k]$ becomes negligible quickly.
* If $H_c(f)$ has discontinuities (jumps) in value, derivative, or higher derivatives at some frequency, the decay is algebraic. Concretely:

  * A jump (discontinuity) in $H_c(f)$ → $h_c(t)$ decays as $1/t$.
  * A jump in the first derivative of $H_c(f)$ → $h_c(t)$ decays as $1/t^2$.
  * In general, if the first non-zero discontinuous derivative is the $m$-th derivative, decay is $\sim 1/t^{m+1}$.

Your magnitude law is continuous at $f_c=4$ kHz but its slope changes (the slope of the magnitude vs frequency has a corner at 4 kHz). That implies a discontinuity in a derivative (so not infinitely smooth). For the power-law → flat join you get roughly a $1/t^2$ decay for the impulse—i.e. coefficients fall roughly like $C/k^2$. That’s **much** better than $1/k$ but not exponentially small.

**Implication:** the impulse is infinite but decays reasonably fast (≈ $1/k^2$). That usually means you can truncate to a few hundred or a few thousand taps depending on your precision/quantization goals — not infinite order and definitely not as pathological as an ideal brickwall.

---

# 4) Quantization and truncation criterion

Because you’re working with 16-bit audio, there is a natural numerical/irrelevance threshold: once tap amplitudes are below the level that contributes less than half a quantization unit (or any chosen perceptual/effective threshold), you can truncate.

Heuristic procedure:

1. compute $g[k]=T h_c(kT)$ numerically (via inverse FFT of $H_c(f)$ sampled on a fine grid).
2. decide on an absolute contribution threshold $\epsilon$ tied to your dynamic range / audible criterion (e.g. $\epsilon = 2^{-16}$ relative to full scale or a stricter energy criterion).
3. truncate where $|g[k]|<\epsilon$ and/or cumulative tail energy is small (e.g. tail energy < −120 dB). Because of $1/k^2$ decay, required truncation length $N$ grows like $\sqrt{C/\epsilon}$ (C depends on the spectral discontinuity strength).

So you *can* make the “implicit window because of quantization” rigorous and choose a finite filter length that meets a numeric error bound.

---

# 5) Efficiency — how to actually compute/apply it

Multiple practical routes; pick based on latency/cost constraints:

A. **Precompute $g[k]$ and convolve (FIR)**

* Compute $g[k]=T h_c(kT)$ via IFFT of the desired continuous spectrum sampled at frequencies up to $F_s/2$ (enforce conjugate symmetry).
* Truncate to useful length $L$. Convolve in time (direct if $L$ small) or overlap-save if $L$ large.
* Pros: exact up to truncation; simple; can be made minimum-phase (via spectral factorization) to reduce causal latency if you’ll use it as a single time-invariant kernel.
* Cons: if required $L$ is large, cheap FFT-block filtering may be better.

B. **FFT block processing (frequency-domain multiplication)**

* For block size $N$ ≥ the filter length, per block FFT → multiply each bin by $H_c(f)$ sampled at FFT bins → IFFT and overlap-add.
* This is exactly the “few FFTs away” approach you suggested and is **mathematically identical** to the precomputed long FIR when the FFT size and overlap are consistent.
* Pros: very efficient for long $g$, exact on FFT bins, flexible.
* Cons: block latency = FFT size/2 (unless small hop sizes / small FFT).

C. **Combine sinc and low-cut analytically**

* The interpolation (sinc LP at Nyquist) and your continuous low-cut can be multiplied in frequency to get a single continuous response $H_{\text{combined}}(f)=H_{\text{recon}}(f)\cdot H_c(f)$. Because ideal reconstructor is an ideal LP with cutoff at $F_s/2$, and your $H_c(f)$ is only needed inside $|f|<F_s/2$, the reconstructor typically contributes only the rectangular baseband support; multiply is trivial. Sampling that combined filter yields the same $g[k]$ as before. So yes—you *can* algebraically combine reconstruction + continuous filter before sampling. That’s exactly the step that shows reconstruction and continuous-time filtering commute and is a source of simplification.

D. **Model reduction / approximation**

* If you want a small causal filter, you can fit a low-order IIR to the precomputed FIR via Prony/Levinson/Padé or rational approximation in the frequency domain (complex curve fit). This gives a compact IIR that approximates the sampled continuous response across the band. It will *not* be exact, but you can bound the approximation error and often get very short filters (2–10 poles) with very good matches.
* You can also spectral-factor the FIR to minimum-phase to reduce latency without increasing length.

---

# 6) Is this equivalent to the DFT/bin approach?

* **Yes and no.** The FFT/DMT block approach and the “reconstruct → filter → sample” approach are the same in the sense that both implement the same ideal continuous response sampled on the FFT grid if you (in the FFT approach) set the bin gains equal to $H_c(f)$ on the bins.
* The difference is **representation**: the FFT approach represents the operator as chunkwise frequency multiplications (exact at FFT bins), while the continuous-convolution viewpoint represents it as a time-domain convolution with $g[k]$. With sufficiently large FFT size, the FFT approach approximates the continuous response everywhere in the band. The two are interchangeable: IFFT of the bin gains yields the FIR $g[k]$. So yes, they are two sides of the same coin.

---

# 7) Practical recommendations for your 48 kHz / 16-bit case

* If you want mathematically exact (to machine precision) action on blocks: use the FFT block method with large enough FFT (e.g. 32768 or 65536) and overlap-add. This is simple and exact on the FFT grid. Latency = FFTsize/2/Fs (choose tradeoff).
* If you want a **single filter** you can ship as coefficients (no block processing), compute $g[k]$ by IFFT of $H_c$ sampled densely up to Nyquist, truncate where coefficients fall below your quantization/error threshold, and use overlap-save (or direct conv if short). Expect required length to be modest because of the $1/k^2$ tail — often a few hundreds of taps gives very low error for audio-grade precision; worst-case maybe a few thousands if you demand extreme accuracy near the corner.
* If latency is critical, do spectral factorization to minimum phase (reduces causal latency significantly) or fit a low-order IIR to the sampled response.
* If CPU efficiency is critical and you can accept block latency, FFT-based filtering is usually best.

---

# 8) Want me to do it for you now?

I can compute the exact sampled impulse $g[k]$ for your specific continuous magnitude (the $+12\text{ dB/octave}$ law with the join at 4 kHz) at $F_s=48$ kHz and:

* produce the full-length FIR (by IFFT on a chosen grid), show its decay and estimate truncation length for a chosen error threshold;
* produce a truncated FIR with error guarantees at DFT bins and give coefficient files (text/CSV);
* or fit a low-order IIR and show the frequency error.

If you want that, tell me which you prefer (FFT block size to use, or target FIR length or target maximum error in dB), and I’ll produce the coefficients and plots. If you don’t want to choose, I can make a reasonable default (e.g. Nfft=65536, report g[k] decay, suggest truncation to where tail energy < −120 dB).

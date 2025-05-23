

## System Requirements 

We're building a USB-powered, single-channel, low-frequency signal analyser and signal generator. 

What does that mean ? 

- USB powered: typically less than 500mA current consumption, at nominally +5V bus voltage. 
  This gives us max 2.5W power. Since we're using USB, we'll also use the data lines to stream data and control our device. 

- Low frenquency: audio band (20 Hz to 20kHz)

- Signal analyser: need some form of analogue to digital converter (ADC, one channel)

- Signal generator: need some form of digital to analogue converter (DAC, one channel)

-----------------------------------------------------------------------------------------------------------------------

an OTG (on the go) USB can act as both a host and as a device. host can supply power and also do a bunch of other coordination. 


SYSTEM REQUIREMENTS: USB

• For this design, interested in ‘low-frequency’ (or low-bandwidth) signals. 20 Hz to 20 kHz approximately.
  • Therefore… lower requirements for data storage and transfer speeds!

• Max frequency of interest is 20 kHz, so due to Nyquist we need to sample at double the maximum frequency of interest.
  • Sampling rate at least 40 kHz.

• To reduce quantisation error and get accurate readings, we’ll need an ADC with a minimum bit depth of 12.
  • Single channel data-rate: 1 Channel × 12 bits × 40,000 samples per second (fs = 40 kHz) = 0.46 Mbit/s

** What is quantisation error ? **
Quantization error is the difference between the true analog input voltage and the nearest discrete digital level that the ADC can represent. 
Because an ADC has only a finite number of output codes (2ᴺ for an N-bit converter), every sample must be “rounded” to one of those levels—this rounding introduces an error, 
often modeled as a small random noise uniformly distributed in ±½ LSB (least significant bit).

• USB 2.0 is standard for many microcontrollers.
  • FS: up to 12 Mbits/s and HS: up to 480 Mbits/s → FS is more than sufficient!

• System is bus-powered device only (not acting as host), guess: far less than 500 mA current consumption.
  • Choose USB Type C as popularity is growing and more user-friendly.

**What is bus-powered device ? **
A “bus-powered device only” means it draws all its power (up to 500 mA@5 V) from the USB port and never has its own supply or acts as a USB host.
In this context, the “bus” is the shared set of wires over which both power and data are carried—in USB’s case the four pins (VBUS, GND, D+, D–). 

-----------------------------------------------------------------------------------------------------------------------


SYSTEM REQUIREMENTS: PROCESSOR

- Low bandwidth and data storage requirements (previous slide: <0.5 Mbit/s in).
  - FPGA unnecessary. Powerful MCU largely unnecessary (unless we're doing a lot of DSP).

- Ease-of-programming and software development is a plus.
  Microcontroller beats FPGA by a large margin. Writing in C/C++ rather than VHDL/Verilog.
- Can use SWD (Serial Wire Debug) or JTAG (Joint Test Action Group) debug protocols.

Peripheral requirements:
- USB for data streaming.
- Most likely: SPI (one for ADC, one for DAC).

**SPI vs I2C Speeds:**
SPI: 1-50 MHz typical, can reach 100+ MHz, full-duplex (simultaneously bidirectional)
I2C: 100 kHz standard, up to 5 MHz in ultra-fast mode, half-duplex (unidirectional at a given time)
SPI is substantially faster, making it better suited for timing-critical applications like the ADC/DAC interfaces in your system.

Processing power? Packaging? Brand
- Not much processing power required (ARM Cortex M0 should suffice, low-ish clock speed).
- **AVOID** BGA and QFN packages to reduce cost (troubleshooting is also hard).
-----------------------------------------------------------------------------------------------------------------------


SYSTEM REQUIREMENTS: POWER SUPPLIES

• Split digital and analogue supplies.
  • Digital: switching supply for efficiency, digital is far more tolerant of noise. Our digital circuitry draws 100s of mAs maximum.
  • Analogue: linear (LDO) regulators for improved noise performance. Analogue circuitry typically draws less current (10s of mAs maximum) → regulator efficiency is not much of an issue.


• Input power from USB.
  • (Very) noisy power rail → Requires adequate filtering!
  • Nominally +5 V. Can go as low as +4.5 V → Watch out for regulator drop-out voltages!
  • 500 mA max. current (if negotiated with host), 150 mA otherwise.


**Why do we use digital and analogue supplies ?**

- Switching regulators give you high efficiency (so less heat and longer battery life) but they switch at high frequency, injecting ripple and EMI onto the rail.

- Digital ICs (MCUs, FPGAs, logic chips) generally tolerate a few tens or hundreds of millivolts of ripple just fine.

- Analogue blocks (ADCs, precision amplifiers, voltage references, sensors) are much more sensitive: even small noise on their supply can translate into measurement error or jitter.

So the common pattern is:

- Use a switcher to feed your “bulk” 3.3 V rail for all the digital bits.

- Tap off that rail with an LDO to create a super-clean 3.3 V (or 1.8 V, 2.5 V, etc.) supply dedicated to analogue circuitry.

-----------------------------------------------------------------------------------------------------------------------

SYSTEM REQUIREMENTS: CONVERTERS

• Basics:
 • More bits = better signal-to-noise (SNR) ratio! ☺

**Why?**
Smaller slices → smaller quantization error → higher theoretical signal-to-noise ratio (SNR) and dynamic range. 
In an ideal ADC, every extra bit buys you about 6 dB of SNR (that’s the 6.02 × N rule).

 • Higher sampling rate = higher Nyquist limit and (typically) less aliasing! ☺
 • → However, in both cases: more data to store/process/stream! ☹


• Additional features could include:
 • Oversampling (to reduce aliasing but ultimately provide a lower output sample-rate).
 • Anti-aliasing filters incorporated into converter ICs.

**What is aliasing ?**
Aliasing is what happens when a signal is sampled too slowly:
- Anything in the input that’s above the Nyquist frequency (½ × sample-rate) “folds” down and shows up as a false, lower-frequency component in the digitized data.
- Those fake tones can’t be separated from the real ones afterward, so you try to prevent them with an anti-aliasing filter or by sampling faster.

• Interface:
 • SPI for lower data-rate converters → Perfect for us since we are using an MCU and only require low data-rates.
 • For more ‘involved’ applications, parallel interfaces, LVDS, etc…

-----------------------------------------------------------------------------------------------------------------------

SYSTEM REQUIREMENTS: ANALOGUE SECTIONS

For ADC:
- High-impedance (> 1 MΩ?) input buffer with ESD protection, RF filtering.

- Anti-aliasing filter with cut-off at (max.) half the ADC sampling rate (Nyquist limit).

- Potentially: single-ended-to-differential conversion, or differential-to-single-ended conversion.


**Why might we need RF filtering ?**
- Any high-frequency energy (radio stations, Wi-Fi, LTE, nearby switching regulators, clock edges) can couple into long traces or cables that feed a high-impedance ADC input.

- Once inside, non-linear junctions (ESD diodes, sample-and-hold switches, op-amps) can rectify or mix that RF down into the audio/baseband range, showing up as offsets or extra noise.

- RF that sits above Nyquist can also alias into the pass-band if the anti-alias filter lacks enough stop-band rejection at tens-of-MHz.

A small RC or ferrite-bead + capacitor at the connector presents a low-impedance path for RF to ground while leaving your signal band undisturbed, preventing all of those issues


**Why do we have high input impedence for ADC but low output impedence for DAC ?**
for an ADC: 
- Acts like a voltmeter: must sense the source without drawing current, or the source voltage would drop and the reading be wrong.
- Many sensors have large source-resistance; a mega-ohm-plus input keeps I ≈ 0, so V_measured ≈ V_true​.

for a DAC: 
- Acts like a voltage source: must drive downstream loads, cables, or filters without its own voltage changing.
- A few tens-of-ohms output lets it supply current; the load sees the programmed voltage independent of load variations or external noise.


For DAC:s
- Low-impedance (~ 50 Ω?) output buffer with ESD protection to drive loads.

- Anti-aliasing filter with cut-off at (max.) half the DAC sampling rate (Nyquist limit).

- Potentially: single-ended-to-differential conversion, or differential-to-single-ended conversion.

note: last point means, be ready to translate between single-ended and differential domains so your ADC/DAC see the format they’re designed for, maximising noise immunity and dynamic range.

-----------------------------------------------------------------------------------------------------------------------


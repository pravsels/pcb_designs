

## System Requirements 

We're building a USB-powered, single-channel, low-frequency signal analyser and signal generator. 

What does that mean ? 

- USB powered: typically less than 500mA current consumption, at nominally +5V bus voltage. 
  This gives us max 2.5W power. Since we're using USB, we'll also use the data lines to stream data and control our device. 

- Low frenquency: audio band (20 Hz to 20kHz)

- Signal analyser: need some form of analogue to digital converter (ADC, one channel)

- Signal generator: need some form of digital to analogue converter (DAC, one channel)

----------------------------------------------------------------------------------------------------------------------------------

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
A “bus-powered” device takes its 5 V supply directly from that USB bus rather than from any external source.

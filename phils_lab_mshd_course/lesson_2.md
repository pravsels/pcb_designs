

## System Requirements 

We're building a USB-powered, single-channel, low-frequency signal analyser and signal generator. 

What does that mean ? 

- USB powered: typically less than 500mA current consumption, at nominally +5V bus voltage. This gives us max 2.5W power. Since we're using USB, we'll also use the data lines to stream data and control our device. 

- Low frenquency: audio band (20 Hz to 20kHz)

- Signal analyser: need some form of analogue to digital converter (ADC, one channel)

- Signal generator: need some form of digital to analogue converter (DAC, one channel)


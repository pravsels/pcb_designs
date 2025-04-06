
# ESP32 IOT Device 

The board is a general IoT device based on an ESP32 WROOM module.

The following features must be integrated:

- Si7021 - I2C temperature and humidity sensor
- QWIIC connector for other I2C devices
- CP2104 - USB 2 serial UART and on board programmer for the ESP32
- RGB LED and a button
- Interface for LED strips, 4 level converted outputs and fused powersupply available on screw terminals
- Power relay
- 3.2mm mouting holes in each corner

The board will measure 80x80mm and it will be a 4 layer board design


## USB-C to UART subcircuit

- USB-C input 
- USBLC6-2P6 for ESD (electro-static discharge) protection 
- CPI2104 for USB to UART conversion 


### USB-C device pins 

- VBUS & GND → Power and ground.
- D+ & D- → USB 2.0 data transfer.
- CC1 & CC2 → Orientation detection & role negotiation.
- SBU1 & SBU2 → Used for alternate modes like DisplayPort.


### USBLC and CP Background 

What is USBLC?

USBLC refers to USB Line Protection Components, specifically USB ESD protection diodes. 
These components are used to protect USB data lines (D+ and D-) from electrostatic discharge (ESD), voltage spikes, and electromagnetic interference (EMI).

How USBLC Works: 

    - The diodes inside the USBLC component act as voltage clamps, preventing voltage spikes from damaging sensitive electronics.
    - They redirect excessive voltage surges (e.g., static electricity from plugging in a USB cable) away from the USB data lines and safely discharge them to ground.
    - This prevents damage to USB transceivers like the CP2104.
    - The USBLC6-2P6 in the circuit is a 6-line ESD protection device, designed to handle fast transient voltage surges.


Why Are USB Data Lines Protected ?

USB ports are externally exposed, making them vulnerable to electrostatic discharge (ESD), voltage spikes, and RF noise. 
Without protection, several issues may arise:

    - ESD discharge (up to ±15kV) can instantly destroy a USB transceiver.
    - Voltage spikes from hot-plugging can cause glitches, corrupted data, or permanent damage.
    - Electromagnetic interference (EMI) can introduce noise and distort USB communication.
    - Using USBLC ensures stable and reliable data transfer while protecting the circuit.


What is CP2104 ?

CP2104 belongs to the CP210x series, a USB-to-UART bridge IC family developed by Silicon Labs.

How CP2104 Works: 

    - Converts USB data (D+ and D-) into UART signals (TX, RX).
    - Enables a USB host (e.g., a PC) to communicate with microcontrollers or embedded systems via serial UART.


Why do we need pull-down resistors for the CC pins ? 

In USB-C, the CC pin is used for detection. The USB host (computer or charger) applies voltage to CC, 
and the device (your circuit) is supposed to pull it down slightly with a 5.1kΩ resistor.

| **Case**                            | **Voltage on CC Pin** | **What the USB Host Thinks**            |
|-------------------------------------|-----------------------|-----------------------------------------|
| ✅ **Correct: 5.1kΩ pull-down**     | ~0.6V                 | "USB 2.0 device attached"               |
| ❌ **No resistor (floating CC)**    | Unknown / unstable    | Device may not be detected              |
| ❌ **Direct connection to GND**     | 0V                    | Host may think there's a short circuit  |


Without a Resistor:

    - The CC pin could pick up noise, making the voltage random and unstable.
    - The USB host won't detect the device properly because it expects a specific voltage drop caused by the 5.1kΩ resistor.
    - The USB host might not enable power or data transfer because it doesn't detect a valid connection.


### Inductor and Fuse Background 

1. Inductor (FB1 - Ferrite Bead)

🔹 Component: BLM21PG221 (Ferrite Bead)

🔹 Purpose: Filters high-frequency noise (EMI suppression)

Why It’s Needed

    - USB power (VBUS) carries noise from chargers, PCs, and RF sources (Wi-Fi, Bluetooth).
    - A ferrite bead acts as a low-pass filter, allowing DC power while blocking high-frequency noise.
    - Prevents EMI from affecting USB signals and internal components.

How It Works

    - Placed in series with +5VP, so all power passes through it first.
    - DC power flows normally, but high-frequency noise is blocked.
    - Improves signal integrity, reducing interference in the CP2104 and other circuits.


2. Fuse (F2 - Polyfuse 1A)

🔹 Component: Polyfuse 1A (Resettable Fuse)

🔹 Purpose: Protects against overcurrent and short circuits

Why It’s Needed

    - USB power has current limits—excess draw can damage components.
    - A short circuit could cause dangerous current spikes.
    - The fuse limits current to 1A and disconnects power if exceeded.

How It Works

    - Placed in series with VBUS (+5V).
    - Normal operation: Passes power if current is < 1A.
    - Overcurrent event (e.g., short circuit): Cuts power if >1A, preventing damage.
    - Auto-resetting: Unlike regular fuses, it recovers when cooled down.


### Capacitors 

Why we need capacitors 

- Stabilize voltage: prevent sudden drops or spikes that can reset or disturb ICs.
- Filter noise: clean up high-frequency and low-frequency noise from USB power (VBUS).
- Ensure reliable USB communication: avoid data errors caused by unstable voltage.


| **Capacitor** | **Value**  | **Purpose**                                                               | **Why It's Needed (Reasoning)**                                                                                                                               |
|---------------|------------|---------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **C4**        | 22 µF      | **Bulk decoupling** for low-frequency noise and large voltage swings.     | Smooths out **slow** changes in USB VBUS, handles large current draws (e.g., device startup).                                                                 |
| **C3**        | 4.7 µF     | **Mid-frequency filtering** for medium-speed noise.                       | Absorbs **medium-speed** disturbances that bulk cap (C4) is too slow for, and small cap (C2) can't handle.                                                    |
| **C2**        | 100 nF     | **High-frequency decoupling** for fast noise and transients.              | Filters **high-frequency noise** caused by CP2104's **fast internal switching** (USB traffic, UART activity), provides quick bursts of current when needed.   |



### CP2104 Important Pins and Their Functions

| **Pin**  | **Function**                  | **What You Connect**                                       | **Required** |
|----------|--------------------------------|------------------------------------------------------------|-------------|
| **RST**  | Resets the chip (Active-Low)   | **Pull-up resistor (4.7kΩ - 10kΩ)**                         | ✅ Yes, otherwise random resets may happen. |
| **VDD**  | Main Power Supply              | **100nF + 1µF capacitor**                                   | ✅ Yes, provides power to CP2104. |
| **VIO**  | Sets logic level for I/O pins  | **Tie to 3.3V (or system voltage), add 100nF capacitor**   | ✅ Yes, must match the system logic voltage. |
| **VPP**  | Factory programming voltage    | **Usually leave unconnected**                              | ❌ No, unless you're programming the chip. |


### Power Distribution in the CP2104 Circuit

| **Power Label** | **Voltage**        | **Source**                               | **Used For**       |
|-----------------|--------------------|------------------------------------------|--------------------|
| **+5VP**        | 5V (Raw USB Power) | USB Connector (J5)                       | Direct USB Power (before filtering) |
| **+5V**         | 5V (Filtered)      | After Ferrite Bead (FB1) & Polyfuse (F2) | Powering components, feeding 3.3V regulator. |
| **+3.3VA**      | 3.3V (Regulated)   | From Regulator (CP2104 or external)      | Powering CP2104 and any other 3.3V devices.  |


Why is it Called Analog (+3.3VA)?

In mixed-signal circuits (where digital and analog components coexist), we often separate their power supplies to reduce noise. 
The "A" in +3.3VA stands for Analog, meaning this power rail is meant for sensitive components that need a cleaner voltage supply.


### Difference Between Analog and Digital Power

| **Type**          | **Characteristics**                                      | **Where It's Used?**                          |
|-------------------|---------------------------------------------------------|-----------------------------------------------|
| **Digital Power** | Supplies **logic circuits** that operate in **discrete steps** (1s and 0s). | Microcontrollers, CPUs, UART, GPIO.           |
| **Analog Power**  | Supplies **sensitive circuits** that handle **continuous signals**. | Sensors, ADCs, USB transceivers, RF circuits. |


How Does This Apply to Our Circuit?

    - The CP2104 handles USB signals, which involve continuous waveforms (analog behavior).
    - If digital noise contaminates the USB transceiver power, it can cause bad signal quality or data errors.
    - So, "Analog" power (+3.3VA) is labeled separately to indicate that this voltage should be as clean as possible.


What Happens If You Use the Same 3.3V for Everything?

    - Digital circuits (GPIO, UART) create noise due to fast switching (MHz speeds).
    - USB transceivers (analog part of CP2104) are sensitive to noise.

    - If you use the same power line, USB signal quality may degrade, causing:
        - USB disconnection issues.
        - Increased bit errors in data transfer.
        - EMI (Electromagnetic Interference) problems.


### UART Communication Pins

| **Pin** | **Name**  | **Direction** | **Function**                                               |
|--------|-----------|-------------|------------------------------------------------------------|
| 21     | **TXD**   | Output      | UART Transmit Data (Data sent from CP2104).                |
| 20     | **RXD**   | Input       | UART Receive Data (Data received by CP2104).               |
| 19     | **RTS**   | Output      | Request to Send (Flow control, optional).                  |
| 18     | **CTS**   | Input       | Clear to Send (Flow control, optional).                    |
| 22     | **DSR**   | Input       | Data Set Ready (Handshake signal, often unused).           |
| 23     | **DTR**   | Output      | Data Terminal Ready (Handshake signal, often unused).      |
| 24     | **DCD**   | Input       | Data Carrier Detect (Detects an active serial connection). |
| 1      | **RI**    | Input       | Ring Indicator (For modems, rarely used in standard UART). |


### GPIO and Special Function Pins

| **Pin**  | **Name**          | **Function**                                                         |
|----------|-------------------|----------------------------------------------------------------------|
| 17       | **SUSPEND**       | Active HIGH when USB is suspended. Can be used to power down external circuits. |
| 15       | **SUSPEND**       | Active LOW during USB suspend (opposite logic of pin 17).           |
| 14       | **TXT / GPIO.0**   | Can be used as **GPIO 0** or **Transmitter Enable** for RS485 mode. |
| 13       | **RXT / GPIO.1**   | Can be used as **GPIO 1** or **Receiver Enable** for RS485 mode.    |
| 12       | **RS485 / GPIO.2** | Can be used as **GPIO 2** or **RS485 Direction Control**.           |
| 11       | **GPIO.3**         | General-purpose I/O pin (user-defined function).                    |


### ⚡ General Guideline: Which Components Need Analog Power?
| **Component Type**        | **Power Type** | **Why?** |
|--------------------------|--------------|----------|
| **DACs (Digital-to-Analog Converters)** | **Analog (AVDD)** | Outputs a **continuous signal** that must be clean and low-noise. |
| **ADCs (Analog-to-Digital Converters)** | **Analog (AVDD)** | Converts weak analog signals into digital data, so noise can distort accuracy. |
| **Op-Amps & Analog Signal Amplifiers** | **Analog (AVDD)** | Small voltage fluctuations can cause errors, so a **low-noise** power supply is needed. |
| **Precision Voltage References** | **Analog (AVDD)** | Any noise in the power affects accuracy of **DACs, ADCs, and sensors**. |
| **RF Circuits (WiFi, Bluetooth, etc.)** | **Analog (AVDD)** | Radio signals are highly sensitive to noise from switching power supplies. |
| **Sensors (e.g., temperature, pressure, light sensors)** | **Analog (AVDD)** | Sensor output voltages are often weak, so noise from digital circuits can corrupt them. |


### 🔌 Which Components Can Use Digital Power?
| **Component Type**      | **Power Type** | **Why?** |
|------------------------|--------------|----------|
| **Microcontrollers (MCUs, CPUs, GPUs)** | **Digital (VDD, DVDD)** | Only handle **1s and 0s**, so noise isn’t a big issue. |
| **Digital Logic ICs (AND, OR, Flip-Flops, etc.)** | **Digital (VDD)** | Works with binary signals that tolerate noise. |
| **Flash Memory, RAM, ROM** | **Digital (VDD)** | Purely digital storage, immune to small voltage noise. |
| **UART, SPI, I2C Communication Interfaces** | **Digital (VDD)** | These protocols use high/low logic levels and are designed to handle small noise margins. |
| **LEDs & Display Drivers** | **Digital (VDD)** | LEDs and screens are not affected by minor power fluctuations. |


## RTS/DTR sub-circuit 

- RTS (Request To Send) and DTR (Data Terminal Ready) signals generated by a USB-to-serial interface 
    (such as CP2104) control two digital transistors (UMH3N), labeled as Q1A and Q1B.

- When activated, these transistors pull the microcontroller's IO0 and EN pins LOW, automating the 
    reset sequence and enabling entry into programming mode.

**Signal flow:**
PC → CP2104 (USB-to-UART) → RTS/DTR → Q1A/Q1B transistors → IO0/EN (Microcontroller)


**The proper sequence for programming mode is:**

    - Pull IO0 LOW (by setting RTS LOW).

    - Briefly pull EN LOW (by setting DTR LOW), then release (back HIGH).

After this sequence, if the microcontroller detects IO0 LOW at reset, it enters the programming mode 
(also known as "bootloader mode").


**What is Programming Mode?**

Programming mode (bootloader mode) is a special boot state in microcontrollers that allows you to load new firmware via serial communication (UART). While in this mode, the chip waits for firmware uploads from external tools like the Arduino IDE, ESP flashing tool, or esptool.py.

    - Normal Operation Mode: Microcontroller runs user firmware.

    - Programming (Bootloader) Mode: Microcontroller waits to receive new firmware.


## Relay control sub-circuit 

This sub-circuit is designed to switch a relay on and off using a transistor controlled by a low-voltage input (e.g., from a microcontroller).

### Circuit Components

#### Relay (K1 - RT314A05)

Coil Pins: A1 (+5V), A2 (controlled by transistor)

#### Contacts:

- Pin 11: Common (COM)

- Pin 12: Normally Closed (NC)

- Pin 14: Normally Open (NO)


#### Transistor (Q2 - BC817)

Acts as a low-side switch.

#### Pins:

- 1 (Base): Receives control signal (RELAY input).

- 2 (Emitter): Ground

- 3 (Collector): Connected to relay coil

#### Resistors

R15 (1kΩ): Limits base current into transistor.

R14 (10kΩ): Pull-down resistor, ensures transistor stays off by default.

#### Diode (D1 - LL4148)

Flyback diode, protects transistor from voltage spikes when relay coil turns off.

#### LED Indicator (D2 - Yellow) and Resistor (R16 - 1kΩ)

Indicates when the relay coil is energized.

#### Screw Terminal (J2)

External connections for relay contacts.


### 🔌 Relay Operation
| **Component Type**    | **Power Type** | **Why?**                                           |
|-----------------------|----------------|----------------------------------------------------|
| **Input (RELAY HIGH)**| **Relay ON**   | Relay energized, connects Pin 11 → Pin 14 (NO).    |
| **Input (RELAY LOW)** | **Relay OFF**  | Relay de-energized, connects Pin 11 → Pin 12 (NC). |


**How does high and low inputs activate different circuits?**

HIGH input (RELAY pin):

    - Current flows into transistor base (Q2) → turns transistor ON.

    - Current from +5V passes through relay coil (K1) → energizes relay.

    - Relay contacts move from default position (Pin 11 → Pin 12) to energized position (Pin 11 → Pin 14), activating the connected NO circuit.

    - LED (D2) lights up, indicating the relay is active.


LOW input:

    - No current at transistor base → transistor OFF.

    - Relay coil is not energized → relay contacts return to default (Pin 11 → Pin 12), activating NC circuit.

    - LED turns off.


## 74AHCT245 bi-directional tranceiver sub-circuit 

It receives digital signals from LEDSTR1-4, buffers them using the 74AHCT245 and sends them through 22Ω resistors 
to an 8-pin screw terminal (J3), likely for connecting LED strips. 

subcircuit components: 

🔹 U5: 74AHCT245 - Octal Bus Transceiver

    - Buffers 8 digital signals, here only using 4 (A0 - A3 to B0 - B3) 
    - Direction is fixed A → B (pin 1 DIR is tied to GND).
    - Output Enable (CE pin 19) is pulled LOW (enabled). 
    - Operates on +5V (pin 20) with GND on pin 10.

🔹 Inputs: LEDSTR1-4 (Pins 2-5)

    - These go into A0-A3 and are forwarded to B0-B3 by the transceiver.

🔹 Outputs: B0-B3 (Pins 18-15)

    - Buffered versions of inputs. 
    - Each output goes through a 22Ω resistor (R17-R20) to J3 (Pins 2-5).

🔹 J3: 8-Pin Screw Terminal

    - Outputs to the external world.
    - Pins 2-5 carry signals for LED strips. 
    - Pins 1 and 6-8 are GND or unused (GND is explicitly provided on pin 7). 

🔹 Power Supply:

    - Supplied via screw terminal J4 (pins 1: +5V, 2: GND). 
    - Passes through a fuse (F1) for protection.
    - C1 (1000μF) provides bulk decoupling for inrush when LEDs power up.
    - C14 (100nF) is a local decoupling cap for the IC, handling high-frequency noise.


**What's the output enable pin ?** 

It's a control pin on devices like the 74AHCT245 that determines whether the outputs are active or disabled.

In the tranceiver, Pin 19 (CE) is the Output Enable. It's tied to GND, so it's enabled all the time (i.e., outputs are always active).


**Why do we need buffered versions of input signals ?** 

- Microcontrollers can only supply a few milliamps. Buffer chips can supply more current — especially useful when driving long wires or LEDs.

- The buffer cleans up edges (transitions from LOW to HIGH) and ensures strong digital levels. Prevents voltage drop, ringing, or signal degradation on long traces.

- Some buffers support shifting logic levels (e.g., 3.3V logic to 5V signals). Allows 3.3V MCU to control 5V logic devices. 


**What's the meaning of cleaning up edges ?**

When a digital signal switches from LOW (0V) to HIGH (e.g., 3.3V or 5V), it should happen quickly and sharply.

But:

    - Microcontrollers or long traces might create slow or noisy transitions (not a clean square wave).

    - This can cause unreliable behavior in fast digital circuits.

✅ What the buffer does:

    - It takes that sloppy or weak input signal and re-drives it with a clean, sharp edge — fast rise/fall times, full logic levels.

    - Ensures the output is a strong, reliable digital HIGH or LOW.



## Overview of the whole schematic 


### Power & Programming Interface 

The USB port delivers 5V and provides serial communication via a USB-to-UART bridge, 
which includes ESD protection on the data lines. Auto-boot signals (RTS/DTR) allow the 
ESP32 to enter bootloader mode for hands-free programming. 


### 5V to 3.3V LDO 

A linear regulator drops the USB 5V supply to 3.3V for the ESP32 and peripherals, with 
decoupling capacitors to ensure stable operation. An LED on the 3.3V rail indicates when 
power is active. 

Note: 'decoupling' implies that the capacitors isolate a device's local power demands from the larger supply rails. By acting as a local energy reservoir, a decoupling capacitor reduces how 
much sudden current draws or noise in one part of the circuit can affect others. 


### ESP32 Module 

A WiFi/Bluetooth enabled ESP32 module, featuring recommended pull-ups, pull-downs and 
capacitors on its strapping pins, forms the core of the design. The auto-boot circuitry 
ties into EN and IO0 to simplify firmware flashing. 

### I2C (Qwiic/STEMMA QT)

A dedicated connector offers a 3.3V I2C interface for easily adding sensors and modules. 
An on-board temperature/humidity sensor resides on the same bus, making use of the same 
pull-up resistors. 

### LED Driver 

A 74AHCT245 IC level-shifts the 3.3V signals from the ESP32 to 5V logic, buffering 4 data 
lines that exit via JST-style connectors for driving external LED strips or modules. 

### Relay 

A transistor based driver energizes the relay coil under ESP32 control, with a flyback diode 
preventing voltage spikes during switch offs. The relay's contacts are exposed for switching external loads at higher voltages and currents. 

### User I/O 

Push buttons for RESET, BOOT or general purpose use, provide direct user interaction. Status LEDs offer visible feedback for various functions. 

### Mounting 

Mounting holes at the corners allow secure attachment to an enclosure or chassis. Keep-out 
areas around these holes ensure that standoffs or screws won't interfere with nearby components. 


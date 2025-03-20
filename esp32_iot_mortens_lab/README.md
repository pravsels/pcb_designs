
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


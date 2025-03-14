
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

| **Capacitor** | **Value**  | **Purpose**                                                               | **Why It's Needed (Reasoning)**                                                                                                                                             |
|---------------|------------|---------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **C4**        | 22 µF      | **Bulk decoupling** for low-frequency noise and large voltage swings.     | Smooths out **slow** changes in USB VBUS, handles large current draws (e.g., device startup).                                                                               |
| **C3**        | 4.7 µF     | **Mid-frequency filtering** for medium-speed noise.                       | Absorbs **medium-speed** disturbances that bulk cap (C4) is too slow for, and small cap (C2) can't handle.                                                                  |
| **C2**        | 100 nF     | **High-frequency decoupling** for fast noise and transients.              | Filters **high-frequency noise** caused by CP2104's **fast internal switching** (USB traffic, UART activity), provides quick bursts of current when needed.                 |
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|


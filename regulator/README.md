# TPS79333-EP Power Regulation Circuit
5V to 3.3V regulator with noise filtering and status indicators.

## Components

### Core
- **U1 (TPS79333-EP)**
  - Regulates 5V input to stable 3.3V output
  - Handles load current changes
  - EN pin enables regulator when powered
  - BP pin for internal reference stability

- **J1, J2 (Conn_01x02)**
  - Simple 2-pin power connectors
  - Handle power in (5V) and out (3.3V)
  - Ground return path for current

### Power Stability
- **C1 (0.1μF)**
  - Smooths input voltage fluctuations
  - Protects regulator from input noise
  - Fast response to high-frequency noise

- **C2 (2.2μF)**
  - Main output stabilization
  - Supplies current during load changes
  - Maintains 3.3V while regulator adjusts
  - Required for regulator stability

- **C3 (0.01μF)**
  - Catches high-frequency output noise
  - Partners with C2 for better stability
  - Fast response to small fluctuations

### Indicators
- **R1 (649Ω) & D1**
  - Shows 5V power is present
  - R1 limits LED current (~5mA)

- **R2 (280Ω) & D2**
  - Shows 3.3V output is working
  - R2 limits LED current (~10mA)

## Usage Notes
- Designed for unstable/noisy power sources
- Can handle varying load currents
- LEDs help with troubleshooting


## Key Concepts

### Power Stability

- Sudden current draws from devices can cause voltage fluctuations (V = IR)
- Capacitors act like tiny, fast batteries to maintain stable voltage
- Different capacitor sizes handle different types of fluctuations:

- Larger caps (C2) handle slower, bigger current changes
- Smaller caps (C1, C3) catch quick spikes



### Load Current

"Load" is whatever device you're powering

Devices draw varying amounts of current:
- LEDs: ~20mA (steady)
- Microcontrollers: ~100mA (can spike)
- Motors: several amps (very variable)

C2 maintains 3.3V while regulator adjusts to load changes


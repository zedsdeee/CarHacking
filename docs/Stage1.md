# Stage 1: Getting started with Car Hacking 

**Goal**: Understand what automotive networks are, what ECUs do, how the CAN bus works, and how to safely interact with them.

Book: https://archive.org/details/the-car-hackers-handbook

Since I dont own a car, I will setup a virtual CAN interface in linux machine with my Arduino, raspberry pi, and CAN shield/MCP2515 CAN module.

I guess the prerequisite knowledge I should have before diving into car hacking is something like programming skills, linux basic, network basic, embedded systems, and most importantly automotive concepts.

JATG/UART? Maybe, unless doing firmware reverse engineering of actual ECUs but we'll see.

# Funcations 

## 1. ECUs

A modern vehicle is not just a cool running thing, but also it is a network of computers on wheels.

Each computer is called an **ECU (Electronic Control Unit)**.

<img src="/images/ecu.jpg" width="500">

ECUs control: Engine, brakes, transmission, Doors, windows, lights, Infotainment, navigation, airbags, ABS(Anti-lock Braking System)

Instead of each ECU being directly wired to every other ECU, CAN Bus allows them to talk over shared networks.

## 2. CAN

CAN bus stands for **Controller Area Network bus**, and it’s basically a communication system for vehicles and other embedded systems. The most common, high reliability, up to 1 Mbps.

### Why Bus topology?

CAN bus is a **Bus topology** which means All devices are connected to the same two-wire bus (CAN_H and CAN_L lines). 

<img src="/images/MCP2515-Parts.jpg" width="500">

The CAN communication is **message-based protocol**: Each ECU sends messages with an **identifier (ID)**, not directly to another ECU. Other ECUs decide if the message is relevant to them. Something looks like this: **ID + DATA**.

e.g.)
```
ID: 0x123  Payload: 11 22 33 44 55 66 77 88
```

Plus, CAN bus is **Half-Duplex**, meaning devices can either send or receive at any given time, but not both simultaneously. CAN uses a single pair of wires (CAN_H and CAN_L) for communication. 

Because it’s a shared bus, only one ECU can “talk” at a time, and all others listen. WHY? Because vhicles don’t need simultaneous two-way communication on the same line.

### Why Differential signaling?

So basically, CAN bus has two wires: **CAN_H (High)** and **CAN_L (Low)**. In digital terms, usually, 0 and 1 are represented by a high or low voltage.

Dominant bit (0): 

CAN_H ~ 3.5V 
CAN_L ~ 1.5V
**Voltage difference (CAN_H - CAN_L) ≈ 2V**

Recessive bit (1):
  
CAN_H ~ 2.5V
CAN_L ~ 2.5V
**Voltage difference ≈ 0V**

<img src="/images/differential-signal.png" width="500">

The differential voltage is what the nodes actually detect.

But what if it is a normal signal? lets say 0V for 0 and 5V for 1. Suppose noise occurs (like from a nearby motor or magnetic induction), and the 0 V line spikes to 4 V. The receiver sees 4 V and interprets it as 1. This creates a bit error, because the message has been corrupted.

So, CAN doesn’t rely on the absolute voltage of a single wire. It uses the voltage difference between CAN_H and CAN_L (CAN_H - CAN_L). And suppose noise occurs (like from a nearby motor or magnetic induction), and the 0 V line spikes to 4 V here again. Because both wires are next to each other, both levels will be affected with the same amount. so it still gives a 2V.

###  CAN bus data frame

CAN bus is asynchronous communication, which means devices don’t share a global clock signal.

If one device is sending data at a speed of 1Mbit/s and the other is 5Mbit/s, they cannot communicate. 

<img src="/images/Standard-format-of-CAN-data-frame.jpg" width="500">

**1. Two CAN Frame Types**

- **Standard Frame (CAN 2.0A)** → 11-bit identifier (CAN ID)
- **Extended Frame (CAN 2.0B)** → 29-bit identifier (used in some modern vehicles)

**2. Standard CAN Data Frame Structure**

```
| Start of Frame | Identifier | Control | Data | CRC | ACK | End of Frame |
```

| Field               | Size (bits)             | Purpose                                                         |
| ------------------- | ----------------------- | --------------------------------------------------------------- |
| **Start of Frame**  | 1                       | Signals the start of a message                                  |
| **Identifier (ID)** | 11 (or 29 for extended) | Tells what the message is about (priority too)                  |
| **RTR**             | 1                       | Remote Transmission Request (0 = data frame, 1 = request frame) |
| **IDE**             | 1                       | Identifier Extension (0 = standard, 1 = extended)               |
| **DLC**             | 4                       | Data Length Code (0–8 bytes for CAN 2.0)                        |
| **Data**            | 0–64 bits (0–8 bytes)   | Actual payload                                                  |
| **CRC**             | 15 + 1 bit              | Error checking                                                  |
| **ACK**             | 2                       | Acknowledge from receiving nodes                                |
| **EOF**             | 7                       | Marks the end of the frame                                      |


## 3. OBD-II port

OBD-II stands for On-Board Diagnostics version 2. It’s a standardized interface that can access a car’s diagnostic information, emissions data, and in many cases, raw CAN messages.

<img src="/images/obd2port.jpg" width="500">

It is 16-pin trapezoid-shaped connector. Each pin has a specific function like power, ground, CAN_H, CAN_L, and sometimes other protocols (K-Line, J1850, etc.).

<img src="/images/obdii-port.jpg" width="500">

### Why OBD-II?

- Read/clear diagnostic trouble codes (DTCs).
- Monitor live engine data (RPM, temperature, fuel trim, etc.).
- Sniff CAN messages for reverse engineering.
- Perform certain control actions (if you have the right access).

# Threat Models

| External  | Vehicle |     Internal     |
|-----------|---------|------------------|
| cellular  |         | Infotainment     |
| wifi      |         | USB              |
| Bluetooth |         | OBD-II Connector |
| TPMS      |         | CAN bus Splicing |
| KES       |         |                  |

# Attack Surfaces

### Physical Access

- OBD-II port (diagnostic port) → Direct CAN bus access
- USB ports in infotainment systems → Media file parsing vulnerabilities
- CD/DVD players → Exploiting firmware bugs in decoders
- In-car Ethernet → Service/debug ports for technicians
- Direct ECU access → Via UART, JTAG, or chip-off for firmware analysis

Example: Plugging a laptop into the OBD-II port and sniffing/replaying CAN messages.

### Short-Range Wireless

- Bluetooth → Infotainment or telematics pairing flaws
- Wi-Fi hotspots → Weak passwords or outdated firmware
- Key fobs (RFID) → Relay attacks, rolling code cracking
- Tire Pressure Monitoring System (TPMS) → Spoofing tire data
- NFC (keyless start) → Signal amplification attacks

Example: Relaying key fob signals to unlock/start a car from outside its normal range.

### Long-Range Wireless

- Cellular telematics units → Vulnerabilities in remote start, GPS tracking, OTA updates
- Vehicle-to-Vehicle (V2V) → Exploiting future mesh communication
- Satellite radio / GPS spoofing → Altering navigation data, SDR?

Example: Exploiting an insecure API in the manufacturer’s mobile app to send remote commands.

### Supply Chain / Firmware Updates

- Attacking the update process itself.
- OTA (Over-The-Air) updates → Weak cryptographic signing
- Maintenance USB updates → Injecting malicious firmware
- Third-party dongles → Vulnerable insurance trackers or fleet devices

Example: Compromising a manufacturer’s update server to push malicious ECU firmware.




# Getting started with Car Hacking 

Book: https://archive.org/details/the-car-hackers-handbook

Since I dont own a car, I will setup a virtual CAN interface lab on my linux machine with my arduino, raspberry pi, and CAN shield/MCP2515 CAN module.

I guess the prerequisite knowledge I should have before diving into car hacking is something like programming skills, linux, network bacis, embedded systems, and most importantly automotive concepts.

JATG/UART? Maybe, unless doing firmware reverse engineering of actual ECUs but we'll see.

# Funcations 

### ECUs

A cmodern vehicle is not just a cool running thing, but also it is a network of computers on wheels.

Each computer is called an **ECU (Electronic Control Unit)**.

<img src="/images/ecu.jpg" width="500">

ECUs control: Engine, brakes, transmission, Doors, windows, lights, Infotainment, navigation, airbags, ABS(Anti-lock Braking System)

Instead of each ECU being directly wired to every other ECU, CAN Bus allows them to talk over shared networks.

### CAN

CAN bus stands for **Controller Area Network bus**, and it’s basically a communication system for vehicles and other embedded systems.

CAN bus is a **Bus topology** which means All devices are connected to the same two-wire bus (CAN_H and CAN_L lines). 

The CAN communication is **message-based protocol**: Each ECU sends messages with an identifier(ID), not directly to another ECU. Other ECUs decide if the message is relevant to them. Something looks like this: **ID + DATA**.

PLus, CAN bus is **Half-Duplex**, meaning devices can either send or receive at any given time, but not both simultaneously. CAN uses a single pair of wires (CAN_H and CAN_L) for communication. Because it’s a shared bus, only one ECU can “talk” at a time, and all others listen. WHY? Because vhicles don’t need simultaneous two-way communication on the same line

**Asynchronous**

**Differential signal levels**

**CRC**





### OBD-II port

<img src="/images/obd2port.jpg" width="500">


# Threat Models

# Attack Surfaces

# Practices
Using `vcan` I can practice sniffing and injecting CAN messages without touching a real car.

```
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
cansend vcan0 123#1122334455667788
candump vcan0
```

This simulates a CAN network, and I can write scripts to generate traffic, replay messages.




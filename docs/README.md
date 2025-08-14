# Getting started with Car Hacking 

Book: https://archive.org/details/the-car-hackers-handbook

Since I dont own a car, I will setup a virtual CAN interface lab on my linux machine with my arduino, raspberry pi, and CAN sheild/MCP2515 CAN module.

I guess the prerequisite knowledge I should have before diving into car hacking is something like programming skills, linux, network bacis, embedded systems, and most importantly automotive concepts.

JATG/UART? Maybe, unless doing firmware reverse engineering of actual ECUs but we'll see.

# Funcations 

### ECUs

A cmodern vehicle is not just a cool running thing, but also it is a network of computers on wheels.

Each computer is called an **ECU (Electronic Control Unit)**.



ECUs control: Engine, brakes, transmission, Doors, windows, lights, Infotainment, navigation, airbags, ABS(Anti-lock Braking System)

Instead of each ECU being directly wired to every sensor, they talk over shared networks.



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


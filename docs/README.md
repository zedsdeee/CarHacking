# Getting started with car hacking 

Since I dont own a car, I will setup a virtual CAN interface lab on my linux machine.

Using `vcan` I can practice sniffing and injecting CAN messages without touching a real car.

```
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
cansend vcan0 123#1122334455667788
candump vcan0
```

This simulates a CAN network, and I can write scripts to generate traffic, replay messages.

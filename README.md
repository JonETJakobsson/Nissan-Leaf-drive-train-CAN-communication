# Nissan-Leaf-drive-train-CAN-communication
This repository aims to provide CAN communication with the Nissan Leaf 2014 invertor and on board charger (OBC).

## CAN communication
Here I provide implementation for communicating via an Arduino Uno and a micropython board (TinyPICO esp32) using the Mcp2515 CAN module via SPI. I use my own ESP32 Mcp2515 driver that can be found [Here](https://github.com/JonETJakobsson/micropython-mcp2515)

Many recordings exists from the leaf, and they can be found [Here](https://github.com/dalathegreat/leaf_can_bus_messages). Any finding of the meaning of each bit is annotated here. The **AZE0** is the Gen 2 model if I'm not mistaken.

## Inverter
All nessasary hacking to talk with the Gen2 inverter has been performed by [Celeron55](http://productions.8dromeda.net/c55-leaf-inverter-protocol.html) and an arduino Due implementation has been released by [Damien Maguire](https://github.com/damienmaguire/Nissan-Leaf-Inverter-Controller)

### Connections

### 10ms messages
11A, 1D4

### 100ms messeges
50B


## On board charger and DC to DC converter
There seems to be two ways to control the OBC. Using pure CAN communication (not implemented publically), or hacking the boards ([Peter's hack](https://mynissanleaf.com/viewtopic.php?f=44&t=30915&start=80#p596439)) , supplying you own signal to change the current generated (Constat current charger ?)

Initial attempts using CAN has been done by [TrueSoln](https://mynissanleaf.com/viewtopic.php?f=44&t=30915&start=40). Briefly:

* Message 1D4:
  * interval: 10ms
  * Data: 'F7 07 00 04 AA 46 E0 BB'
  * AA: Cycle ['87 c7 07 47']
  * BB: CRC
  * Note: This message is also used to send torque requests to the inverter. Same cycler is implemented when talking to the inverter but the static data bytes are different.

* Message 1F2:
  * interval: 10ms
  * Data: '30 64 20 00 00 82 AA BB'
  * AA: cycler ['00 01 02 03']
  * BB: CRC? or cycler ['0B 0C 0D 0E']?
  * Note: CAN database says that BB is a checksum adding all nibbles of the messages and adds 2 and keeping the four LSB. 

* Message 50B
  * interval: 100ms
  * Data: '00 00 00 C0 00 00' (6 bytes) 
  * Note: inverter likes this message with '00 00 **06** C0 00 00 00' (7 bytes)

It is interesting that the 50B messages has different length. maybe TrueSoln works on the gen 1 charger? 50B might set the "Mode" of the car, only allowing  one mode, such as driving, standby or charging. I personally would love to be able to charge at the same time as I drive, as putting a generator on the tail whould allow the car to be driven to car shows etc. with out adding a huge battery pack, any ways...

It is important to be able to control the current of the charger in order to charge ant battery pack. This data byte is not identified yet.

## How to implement in the car
Preferable, one controller will provide the syncronization of starting the inverter and precharge. This contoller would also deliver the torque requests.

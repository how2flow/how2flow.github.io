---
permalink: /documents/peripherals/i2c/
title: I2C
excerpt: "what is protocol. how i2c work. there are i2c examples"
toc: true
---

## Info

<span style="{{ site.code }}">I²C</span> is a series of buses developed by Philips.<br>
It is used to connect low-speed peripherals to motherboards, embedded systems, etc ...<br>

## Protocols

The i2c bus consists of two signal lines: <span style="{{ site.code }}">SDA</span> and <span style="{{ site.code }}">SCL</span> .<br>
<span style="{{ site.code }}">SDA</span> : Serial Data<br>
<span style="{{ site.code }}">SCL</span> : Serial Clock<br>

I2C communication is a synchronous serial communication.<br>
It is a <span style="{{ site.code }}">half-duplex</span> system.<br>
you can have multiple masters and multiple slaves.<br>

Anyway, to understand the i2c protocol, let's check the i2c data frame first.<br>

### I2C frame

The data frame is as follows
```
| Start | Address | R/W | ACK | Data | Data(burst mode) | ACK | Stop |
```

| Frame | Info |
| :---: | :--- |
| Start | SDA line switches from High to Low, when SCL line is High. |
| Address | Usually, Devices can have 127 addresses ranging from 0 to 7F. |
| R/W | A single bit specifying whether the master is sending(Low) or the slave is requesting(High) |
| ACK | If an address frame or data frame was successfully received,<br>an ACK bit is returned to the sender from the receiving device. |
| Stop | SDA line switches from Low to High, when SCL line is Low. |

#### Start/Stop

The start/stop is determined by the status of the two signal line lines.<br>

Start
```
        -----
             \
              \
               \
SDA             -----

        -------------


SCL
```

Stop
```
                -----
               /
              /
             /
SDA     -----

        -------------


SCL 
```

#### Addressing

Connect multiple devices with two lines,<br>
System must be able to distinguish between devices.<br>

In Linux system, there is packages for i2c.<br>
A typical example is <span style="{{ site.code }}">i2c-tools</span> .<br>

Usage:
```
$ sudo apt install i2c-tools
$ sudo i2cdetect -y 0
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- UU -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --                        
```
It shows i2c device addresses connected to the system.<br>
There is <span style="{{ site.code }}">UU</span> , It is 0x51, It means that the Linux driver is using the address.<br>

Most devices are assigned a unique address of i2c for the device from the manufacturer.<br>
Of course, there is also an IC chip that supports the ability to change the address.<br>

A device with the same address cannot be connected to one bus.<br>
This is <span style="{{ site.code }}">address conflict</span> . To avoid <span style="{{ site.code }}">address conflict</span> ,<br>
Devices that can change addresses can change addresses or Use i2c multiplexer<br>

#### Read/Write

Since communication is required in two lines,<br>
it is necessary to distinguish whether it is a message transmission from the master or a message request from the slave.<br>

The mechanism performs 1 shift operation on the address bit to the left,<br>
and then divides the last bit value by 0/1.<br>
shift operation is [here](http://0.0.0.0:4000/documents/wiringpi/bit-operation-programming/#shift-operation)<br>

#### ACK

Each slaves compares the address sent from master to its own address.<br>
if the address matches, the slave returns an ACK bit(Low). else, the slave returns an ACK bit(High).<br>

#### DATA

After the master detects the ACK bit from slave, the first data frame is ready to be sent.<br>

### Steps of i2c

This is the step of i2c communication
```
1. master --(Start)--> slave
2. master --(Address + R/W)--> slave
3. slave compares address with its
4. slave  --(ACK)--> master # if addresses match.
5. master --(DATA)--> slave or master <--(DATA)-- slave
6. master --(ACK) --> slave or master <--(ACK)-- slave
7. master --(STOP)--> slave
8. master set SCL High (before SDA is high)
```

## Feature of i2c

The SDA and SCL lines require pull-up resistance.<br>
Because the i2c device itself is either an open collector or an open drain.<br>
Therefore, the open and ground states must be clear, so pull-up resistance is required.<br>

And there are some advantages and disadvantages.<br><br>

This is advantages of i2c
```
1. only uses 2 wires
2. support multiple masters and slaves
   problem: when two or more masters try to send or receive data at the same time.
            To solve it, each master needs to detect if the SDA line is low or high.
            If SDA is Low, another master has control of the bus, masters should wait to control bus.
3. ACK bit gives confirmation that each frame is transferred successfully
```
<br>

This is advantages of i2c
```
1. slower than SPI.
2. data is limited to 8 bits.
```

## Reference

[Circuit Basics](https://www.circuitbasics.com/basics-of-the-i2c-communication-protocol/)

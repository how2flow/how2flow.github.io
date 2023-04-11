---
permalink: /documents/peripherals/can/
title: CAN-bus
excerpt: "How to flow CAN. can, can protocols, can bus, can network"
toc: true
---

## Info

<span style="{{ site.en }}">It was first developed by Bosch in 1983, and since its official introduction in 1986,<br>
it has been used in most cars currently produced and has been applied in various other fields.</span>

## Protocols

CAN2.0A / CAN2.0B / CANFD

CAN2.0A: The normal format.<br>
CAN2.0B: The extended format of CAN2.0A.<br>
CANFD: Flexible bitrate & max 64 byte data.<br>

This page is based on CAN 2.0A.<br>
Signal interpretation is easy if you only know the frame and the stuff bit.

### CAN bus

The key to CAN communication is to send and receive messages with the volt difference of CAN bus H/L.<br>
If there is a voltage difference, it is <span style="{{ site.code }}">dominent</span> , and if there is no voltage difference, it is <span style="{{ site.code }}">receiving</span> .<br>
The signal value that the MCU receives through the transceiver is<br>

<p align="center">
  <img src="/documents/images/peripherals/can/canbus.png" alt="atomic1" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [1] CANBUS states</span>
</p>
<br>

| state | value |
| :---: | :---: |
| dominent | 0 |
| recessive | 1 |

### Frame

It is can2.0A frame (44 + 8N bit).

```
| SOF(1) | ID(11) | RTR(1) | IDE(1) | RB(1) | DLC(4) | DATA(8N) | CRC(15) | DEL(1) | ACK(2) | EOF(7) |
```
<span style="color:red; font-size:60%">RB: reserved</span>

#### message competition

In CAN interface, multiple nodes transmit messages at the same time.<br>
But, CAN has one bus, It is <span style="{{ site.code }}">half-duplex</span> system.<br>
So, CAN messages compete with each other.<br>

The highest priority message is delivered by comparing the **ID** value between the <span style="{{ site.code }}">SOF</span> and <span sytle="{{ site.code }}">RTR</span> of the CAN frame.<br>
The remaining nodes are ready to be sent again.<br>

The <span style="{{ site.code }}">dominant</span> is **LOW**. The <span style="{{ site.code }}">recessive</span> is **HIGH**.<br>
Suppose 3 nodes are connected by a CAN interface.<br>
The message being sent is determined by the priority of the **ID**.<br>

<p align="center">
  <img src="/documents/images/peripherals/can/can-competition.png" alt="can-competition" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [2] CAN messages competition</span>
</p>
<br>

All bits from 10 to 6 of the message id progress to the same dominant and recessive bit.<br>
Only NODE 2 of bit 5 is a recessive bit, and is eliminated from message competition.<br>
at bit 2, NODE 1 is eliminated from message competition for the same reason.<br>

Comparing the CANBUS state value with the ID value of NODE3, it is the same.<br>

### Stuff bit

If the same bit comes out five times in a row, the next bit must use one opposite bit.<br>

```
00000'1'01 ... 11111'0'01 ...
```

## Errors

CAN bus errors can occur for several reasons<br>
faulty cables, noise, incorrect termination, malfunctioning CAN nodes, etc ...<br>

Error handling identifies and rejects erroneous messages, enabling a sender to re-transmit the message.<br>
Further, the process helps identify and disconnect CAN nodes that consistently transmit erroneous messages.<br>

step-by-step

<p align="center">
  <img src="/documents/images/peripherals/can/can-errors-handle.png" alt="can-errors-handle" width="640" height="180"><br>
  <span style="{{ site.img }}">picture [3] CAN errors handling sequence</span>
</p>
<br>

```
 1. CAN node 1 transmits a message onto the CAN bus - and reads every bit it sends
 2. In doing so, it discovers that one bit that was sent dominant was read recessive
 3. This is a 'Bit Error' and node 1 raises an Active Error Flag to inform other nodes
 4. In practice, this means that node 1 sends a sequence of 6 dominant bits onto the bus
 5. In turn, the 6 dominant bits are seen as a 'Bit Stuffing Error' by other nodes
 6. In response, nodes 2 and 3 simultaneously raise an Active Error Flag
 7. This sequence of raised error flags comprise part of a 'CAN error frame'
 8. CAN node 1, the transmitter, increases its 'Transmit Error Counter' (TEC) by 8
 9. CAN nodes 2 and 3 increase their 'Receive Error Counter' (REC) by 1
10. CAN node 1 automatically re-transmits the message - and now succeeds
11. As a result, node 1 reduces its TEC by 1 and nodes 2 and 3 reduce their REC by 1
```

### CAN error types

| Error types | Node | Info |
| :---: | :---: | :--- |
| Bit Error | Transmitter | Occurs when opposite bit is detected during self-examination of the message. |
| Bit Stuffing Error | Receiver | Node detects a sequence of 6 bits of the same logical level between SOF and CRC. |
| Form Error | Receiver | Node detects a bit of an invalid logical level in the SOF/EOF fields or ACK/CRC delimiters. |
| ACK Error | Transmitter | Node transmits a CAN message, but the ACK slot is not made dominant by receiver(s). |
| CRC Error | Receiver | Node calculates a CAN messages CRC that differs from the transmitted CRC field value. |

### CAN node states

The canbus status depends on the error counter value.<br>
<span style="{{ site.code }}">TEC</span> : **T**ransmit **E**rror **C**ounter<br>
<span style="{{ site.code }}">REC</span> : **R**eceive **E**rror **C**ounter<br>

Canbus is default to <span style="{{ site.code }}">error-active</span> .
If the value of <span style="{{ site.code }}">TEC</span> <span style="{{ site.code }}">REC</span> is greater than 127,<br>
Canbus state changed the <span style="{{ site.code }}">error-passive state</span> .

If the <span style="{{ site.code }}">TEC</span> value exceeds 255, the node is disconnected from the bus.<br>
It is <span style="{{ site.code }}">BUS-OFF</span> .

<p align="center">
  <img src="/documents/images/peripherals/can/can-node-states.png" alt="can-node-sates" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [4] CAN node states</span>
</p>
<br>

Error counter changed by:

| EC | value | Info |
| :---: | :---: | :--- |
| TEC | +8 | Transmitter raises primary error flag. |
| TEC | -1 | Transmitter successfully sends message. |
| REC | +8 | Receiver raises primary error flag. |
| REC | +1 | Receiver raises secondary error flag. |
| REC | -1 | Receiver successfully receives message. |

<br>
Bus states:
```
A CAN node enters the Error Passive state if the REC or TEC exceed 127
A CAN node enters the Bus Off state if the TEC exceeds 255
```

## Example
An example of can oscilloscope Waveform (between CAN-rx and GND).

<p align="center">
  <img src="/documents/images/peripherals/can/can-example.png" alt="can-example" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [5] CAN example</span>
</p>
<br>

- Raw data
```
010100000100000100011000100010010001000110011101111101110000110
```

<br>

- Analyze the raw data according to the frame
```
                   RTR                                                        CRC.D
                    |                                                           |
                    |                                                           |
                    |                                                           |
  0 101 0000 0'1'0000 0'1'00011 0001 0001 0010 0010 0011 0011 1011 1110 1110 0001 10
  |   |    |   |   |  | | |   | <---------------------------> <---------------->  ||  
  |   |    |   |   |  | | |   |  Data:     11 22 33                   CRC         ||
  |   |    |   |   |  | | |   |                                                   ||
  s  '5'  '0'  s  '0' I s r   D                                                   AA
  o            t      D t 0   L                                                   CC
  f            u      E u     C                                                   KK
               f        f                                                          .
               f        f                                                          D
```

This message is <span style="{{ site.code }}">ID: 500</span>, <span style="{{ site.code }}">DATA: 11 22 33</span>

## Reference

[TEXAS INSTRUMENTS Introduction to theController Area Network (CAN)](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf?ts=1684085274811&ref_url=https%253A%252F%252Fwww.google.com%252F)<br>
[CSS ELECTRONICS](https://www.csselectronics.com/pages/can-bus-errors-intro-tutorial)

---
permalink: /documents/gpio/can/
title: Contoller Area Network
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

### Frame

It is can2.0A frame (44 + 8N bit).

```
| SOF(1) | ID(11) | RTR(1) | IDE(1) | RB(1) | DLC(4) | DATA(8N) | CRC(15) | DEL(1) | ACK(2) | EOF(7) |
```
<span style="color:red; font-size:60%">RB: reserved</span>

### Stuff bit

If the same bit comes out five times in a row, the next bit must use one opposite bit.<br>

```
00000'1'01 ... 11111'0'01 ...
```

## Example

An example of can oscilloscope Waveform (between CAN-rx and GND).
<div style="text-align:center;">
  <img src="/assets/images/can.jpg" alt="can-example" width="640" height="480">
</div>

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

This message is <span style="color:blue; font-size:120%">ID: 500 DATA: 11 22 33</span>

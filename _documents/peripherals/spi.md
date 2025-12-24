---
permalink: /documents/peripherals/spi/
title: I2C
excerpt: "what is SPI. how spi work. there are spi examples"
toc: true
---

## Info

<span style="{{ site.code }}">SPI</span> is a Synchronization serial data connection standard.<br>

SPI is a synchronous serial communication.<br>
It is a <span style="{{ site.code }}">full-duplex</span> system.<br>

The SPI bus consists of four elements.<br>

<p align="center">
  <img src="/assets/images/documents/peripherals/spi/spi-circuit.png" alt="spi-circuit" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [1] SPI circuit</span>
</p>
<br>

<span style="{{ site.code }}">SCLK</span>  : **S**erial **Cl**oc**k**. (output from master)<br>
<span style="{{ site.code }}">MOSI</span>  : **M**aster **O**utput **S**lave **I**nput.<br>
<span style="{{ site.code }}">MISO</span>  : **M**aster **I**nput **S**lave **O**utput.<br>
<span style="{{ site.code }}">CS/SS</span> : **C**hip **S**election/**S**lave **S**election (output from master)<br>

## Protocols

The maste select a slave to transmit a signal through the <span style="{{ site.code }}">SS</span> .<br>
master transmits a signal synchronized to the <span style="{{ site.code }}">SCLK</span> through the <span style="{{ site.code }}">MOSI</span> .<br>
The slave receives the signal transmitted through his (slave) <span style="{{ site.code }}">MOSI</span> according to the <span style="{{ site.code }}">SCLK</span> .<br>
while it is activated to receive the signal through the <span style="{{ site.code }}">SS</span> .<br><br>

To initiate communication,<br>
**the bus master configures the clock using frequencies supported by the slave device**.<br>
If the slave device has a unique crystal, the spi bus frequency must match the unique crystal frequency.<br><br>

The master sends a bit on the <span style="{{ site.code }}">MOSI</span> line and the slave reads it,<br>
while the slave sends a bit on the <span style="{{ site.code }}">MISO</span> line and the master reads it.<br>
This order is maintained even if only one-way data transfers are intended.<br>

<span style="{{ site.code }}">MOSI</span> and <span style="{{ site.code }}">MISO</span> look like one shift register.<br>
The reason for this is to form a circulation buffer between the chips.<br>
<p align="center">
  <img src="/assets/images/documents/peripherals/spi/spi-mosi-miso.png" alt="spi-mosi-miso" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [2] SPI MOSI and MISO</span>
</p>
<br>

This is Typical SPI circuit with multiple slaves.<br>
<p align="center">
  <img src="/assets/images/documents/peripherals/spi/spi-multiple-slaves.png" alt="spi-multiple-slaves" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [3] Typical SPI circuit: multiple slaves</span>
</p>
<br>

The disadvantage of the SPI interface is that as the number of slave devices increases,<br>
The <span style="{{ site.code }}">SS</span> pin of the master increases.<br>

This is Daisy Chained SPI circuit with multiple slaves<br>
The 'Daisy-Chained' circuit that improves this shortcoming.<br>
<p align="center">
  <img src="/assets/images/documents/peripherals/spi/spi-daisy-chained.png" alt="spi-daisy-chained" width="640" height="480"><br>
  <span style="{{ site.img }}">picture [4] Daisy Chained SPI circuit: multiple slaves</span>
</p>
<br>

The "Daisy Chained" circuit means that even if the slave device is increased,<br>
Only one <span style="{{ site.code }}">SSu</span> pin of the master can be used.<br>

## Feature of spi

There are some advantages and disadvantages.<br><br>

This is advantages of spi
```
1. There is no start/stop signal. The data can be streamed continuously without interruption.
2. There is no conflict slave's address
3. data bigger and faster than i2c
4. full-duplex system
```
<br>

This is disadvantages of spi
```
1. Uses four wires
2. No ACK
3. No form of error checking
4. Only single master
```


## Reference

[Wiki](https://en.wikipedia.org/wiki/Serial_Peripheral_Interface)<br>
[Analog.com]( https://www.analog.com/en/analog-dialogue/articles/introduction-to-spi-interface.html)

---
permalink: /documents/gic-summary/
title: GIC
excerpt: "Generic Interrupt Controller Summary"
comments: false
toc: true
---

## GIC summary

GIC is 'Generic Interrupt Contoller'.<br>

<div style="text-align: center;">
  <img src="/assets/images/documents/gic/gic_summary.png" alt="gic_summary" width="640" height="480"><br>
  <span style="{{ site.img }}">Picture [1] GIC summary</span>
</div>
<br>

<details>
  <summary>Activate interrupt Forcely with Trace32</summary>
  <p>
    Write Target interrupt status register in <span style="{{ site.code }}">ISPENDR(base + 0x200)</span><br>
    The <span style="{{ site.code }}">ICPENDR(base + 0x280)</span> is mirrored to the ISPENDR.<br>
    <br>

    If you write <span style="{{ site.code }}">0x101</span> on <span style="{{ site.code }}">ISPENDR</span> to force interrupts 32 and 34,<br>
    <U>you <B>should not write</B> <span style="{{ site.code }}">0x100</span> on <span style="{{ site.code }}">ISPENDR</span></U> to clear interrupt 32.<br>
    <U><span style="{{ site.code }}">0x100</span> <B>should be used</B> for <span style="{{ site.code }}">ICPENDR</span></U> .<br>
    <br>

    If your development environment is an environment<br>
    where you can toggle the interrupt polarity,(e.g. PIC .. programmable interrupt controller)<br>
    <span style="{{ site.code }}">You can do interrupt set/clear just by toggling the polarity</span> .<br>
  </p>
</details>

## Trace32

Set GIC peri
```
B:: System.CONFIG.GICD Base a:{gicd base address}
B:: PER.Reprograming
```
```
B:: System.CONFIG.GICR Base a:{gicr base address}
B:: PER.Reprograming
```
<br>

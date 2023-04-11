---
permalink: /documents/wiringpi/logical-operation/
title: Logic GATE
excerpt: "documents of logical operation"
comments: false
toc: true
---

## What is logical operation

In <span style="{{ site.code }}">mathematical logic</span> or <span style="{{ site.code }}">computer science</span> , two states, True and False, are called Boolean expression.<br>
After the advent of the Boolean algebra, logic begins to have a strong tendency in symbolic logic.<br>

This document is intended to describe wiringpi.<br>
Logic gate and shift operations are described on a <span style="{{ site.code }}">C</span> basis.<br><br>

It doesn't cover the details of logical mathematics.<br>

### Logic gates

A logic gate is a device that acts as a building block for digital circuits.<br>
There are <span style="{{ site.code }}">AND</span> , <span style="{{ site.code }}">OR</span> , <span style="{{ site.code }}">XOR</span> , <span style="{{ site.code }}">NOT</span> , <span style="{{ site.code }}">NAND</span> , <span style="{{ site.code }}">NOR</span> and <span style="{{ site.code }}">XNOR</span>

#### AND

<span style="{{ site.code }}">AND</span> is that In the boolean algebra, if all are 1, it is an operation that represents 1,<br>
and if any is 0, it represents 0.
```
A & B
```

| A | B | result |
| :---: | :---: | :---: |
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

#### OR

<span style="{{ site.code }}">OR</span> is that In the boolean algebra, if all are 0, it is an operation that represents 0,<br>
and if any is 1, it represents 1.
```
A | B
```

| A | B | result |
| :---: | :---: | :---: |
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 1 |

#### XOR

<span style="{{ site.code }}">XOR</span> is that In the boolean algebra, if all are equal, it is an operation that represents 0,<br>
and if any is different, represents 1.
```
A ^ B
```

| A | B | result |
| :---: | :---: | :---: |
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

#### NOT

<span style="{{ site.code }}">NOT</span> is that In the boolean algebra, Toggle all values to the opposite value.
```
~A
```

| A | result |
| :---: | :---: |
| 0 | 1 |
| 1 | 0 |

#### NAND

<span style="{{ site.code }}">NAND</span> is that In the boolean algebra, if all are 1, it is an operation that represents 0,<br>
and if any is 0, it represents 1.
```
~(A & B)
```

| A | B | result |
| :---: | :---: | :---: |
| 0 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

#### NOR

<span style="{{ site.code }}">NOR</span> is that In the boolean algebra, if all are 0, it is an operation that represents 1,<br>
and if any is 1, it represents 0.
```
~(A | B)
```

| A | B | result |
| :---: | :---: | :---: |
| 0 | 0 | 1 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 0 |

#### XNOR

<span style="{{ site.code }}">XNOR</span> is that In the boolean algebra, if all are equal, it is an operation that represents 1,<br>
and if any is different, represents 0.
```
~(A ^ B)
```

| A | B | result |
| :---: | :---: | :---: |
| 0 | 0 | 1 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |


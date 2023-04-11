---
permalink: /documents/wiringpi/bit-operation-programming/
title: bitwise operation
excerpt: "Bit operation programming basics and examples"
comments: false
toc: true
---

## Intro

This page will cover bit operations.<br>
Of course, it is a wiringpi sub-document, but the content will never be limited to wiringpi.<br>

## Bitwise Operation

### Bits operation

Comparison of values is key to bit-logic operations.<br>
You checked about digital logic-gate in the [previous document](/documents/wiringpi/logical-operation/).<br>
Then, you need to know the properties of <span style="{{ site.code }}">and</span>, <span style="{{ site.code }}">or</span>, and <span style="{{ site.code }}">not</span> for bitwise programming.<br>

#### AND

<span sytle="{{ site.code }}">And</span> has the characteristic that if one of them has zero, result is zero.<br>
To make it easier to understand, I'll give you an example in two bits.<br>

case 1
```
? & 1 = 0
```
what is "<span style="{{ site.code }}">?</span>"<br><br>

case 2
```
? & 1 = 1
```
what is "<span style="{{ site.code }}">?</span>"<br><br>

We know both answers.<br>
Isn't case 1 is 0 and case 2 is 1?<br>
Some of you may have noticed here, but there is one characteristic.<br>
The value of "<span style="{{ site.code }}">?</span>" is the same as the result value.<br>
**If you do the AND operation with 1, you protect it to original**.<br><br>

If then,<br>

case 3
```
? & 0 = 0
```
what is "<span style="{{ site.code }}">?</span>"<br>
Can you answer for sure between 0 and 1?<br>
**If you do the and operation with zero, you force it to zero**.<br>


Finally, **If you want to change the position you want to zero, you can use AND operation**.
```
target's bit2 and bit7 need to set  0. 

target = 10110101;
target = target & 01111011;
```

Will you use <span style="{{ site.code }}">01111011</span> to write bit2 and bit7 as 0 when you actually code?<br>
<span style="{{ site.code }}">01111011</span> is not intuitive.<br>
You can change it more intuitively using <span style="{{ site.code }}">NOT</span> here.
```
~(10000100)
```
It is more intuitive to mark only bit2 and bit7 as 1.<br>
This is the power of <span style="{{ site.code }}">NOT</span>.<br><br>


Modify the code like this.
```
target = 10110101;
target = target & ~(10000100);
```
<span style="{{ site.code }}">target = target & ~(10000100);</span> can be changed <span style="{{ site.code }}">target &= ~(10000100);</span><br><br>

**To write zero where I want to be,**
```
# if you want to change target's bit2 and bit7 as 0,
target &= ~(10000100);
```

#### OR

<span sytle="{{ site.code }}">OR</span> has the characteristic that if one of them has 1, result is 1.<br>
To make it easier to understand, I'll give you an example in two bits.<br>

case 1
```
? | 0 = 0
```
what is "<span style="{{ site.code }}">?</span>"<br><br>

case 2
```
? | 0 = 1
```
what is "<span style="{{ site.code }}">?</span>"<br><br>

We know both answers.<br>
Isn't case 1 is 0 and case 2 is 1?<br>
Some of you may have noticed here, but there is one characteristic.<br>
The value of "<span style="{{ site.code }}">?</span>" is the same as the result value.<br>
**If you do the OR operation with zero, you protect it to original**.<br><br>

If then,<br>

case 3
```
? | 1 = 1
```
what is "<span style="{{ site.code }}">?</span>"<br>
Can you answer for sure between 0 and 1?<br>
**If you do the OR operation with 1, you force it to 1**.<br>

Finally, **If you want to change the position you want to 1, you can use OR operation**.
```
target's bit2 and bit7 need to set 1. 

target = 10110101;
target = target | 10000100;
```
<span style="{{ site.code }}">target = target | 10000100;</span> can be changed <span style="{{ site.code }}">target |= 10000100;</span><br><br>

**To write zero where I want to be,**
```
# if you want to change target's bit2 and bit7 as 1,
target |= 10000100;
```

### Shift operation

Shift operations change binary values by moving them.<br>
There are <span style="{{ site.code }}"><<</span> and <span style="{{ site.code }}">>></span> .<br>

The direction of movement is the same as the shape<br>

<span style="{{ site.code }}"><<</span> moves binaries by filling the right-end with 0.<br>
If the memory size is exceeded, the highest digit is discarded first.<br>
For each space move, the value is <span style="{{ site.code }}">value x 2</span> .
```
input: 0b00001010 << 2      deci: 10
result: 0b00101000          deci: 40  move 2 space, value: (value x2 x2)
```

<span style="{{ site.code }}">>></span> moves binaries by filling the left-end with 0.<br>
If the memory size is exceeded, the highest digit is discarded first.<br>
For each space move, the value is <span style="{{ site.code }}">value / 2</span> .
```
input: 0b00010000 >> 3      deci: 16
result: 0b00000010          deci:  2  move 3 space, value: (value /2 /2 /2)
```

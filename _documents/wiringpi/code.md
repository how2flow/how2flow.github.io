---
permalink: /documents/wiringpi/code/
title: "The code review of wiringPi"
excerpt: "wiringPi Programming with bitwise operations"
comments: false
toc: true
---

## Intro

[Logic gate]() and [Bit operation]() were reviewed for wiringpi code analysis.<br>
This document starts code analysis based on <span style="{{ site.code }}">Raspberry Pi4</span> .

Once the code analysis is complete, proceed to the application case.<br>
The target is the <span style="{{ site.code }}">odroid-wiringpi</span> that I ported directly to the odroid.<br>
<span style="{{ site.code }}">odroid-wiringpi</span> can be uploaded to posting, not a document.<br>

## Code Review

It is assumed that the person looking at this document can understand the datasheet<br>
and has a good understanding of C/C++ and bit operations.<br>

### Requirements

Prepare what you need when analyzing the code.<br>
It's a optional to be prepared, but I still recommend you to be prepared.<br>

#### target

<p align="center">
  <img src="/documents/images/wiringpi/raspberrypi.png" alt="raspberrypi" width="320" height="240">
  <img src="/documents/images/wiringpi/rpi-header.png" alt="rpi-header" width="320" height="240"><br>
  <span style="{{ site.img }}">Raspberry Pi 4 & gpio header</span>
</p>
<br>

#### datasheet

GPIO control is a method of directly accessing memory addresses and entering values into registers.<br>
After accessing memory addresses through memory mapping, control gpio with bit operations.<br>

[bcm2711 datasheet](/documents/pdf/wiringpi/bcm2711.pdf)<br>

#### source code

[Git source](https://github.com/WiringPi/WiringPi.git)
```
$ git clone https://github.com/WiringPi/WiringPi.git
```

### Review

#### memory mapping

Memory mapping must precede access to the GPIO register.<br>
Check the model first and attach the correct memory address for the model.<br><br>

<span style="{{ site.code }}">wiringPi/wiringPi.c</span>
```
void piBoardId (int *model, int *rev, int *mem, int *maker, int *warranty)
{
  FILE *cpuFd ;
  char line [120] ;
  char *c ; 
  unsigned int revision ;
  int bRev, bType, bProc, bMfg, bMem, bWarranty ;

//  Will deal with the properly later on - for now, lets just get it going...
//  unsigned int modelNum ;

  (void)piGpioLayout () ;   // Call this first to make sure all's OK. Don't care about the result.

  if ((cpuFd = fopen ("/proc/cpuinfo", "r")) == NULL)
    piGpioLayoutOops ("Unable to open /proc/cpuinfo") ;

...

}
```
```
int wiringPiSetup (void)
{
  int   fd ;
  int   model, rev, mem, maker, overVolted ;

  if (wiringPiSetuped)
    return 0 ;

  ...

  piBoardId (&model, &rev, &mem, &maker, &overVolted) ;

  ...

  switch (model)
  {
    case PI_MODEL_A:    case PI_MODEL_B:
    case PI_MODEL_AP:   case PI_MODEL_BP:
    case PI_ALPHA:  case PI_MODEL_CM:
    case PI_MODEL_ZERO: case PI_MODEL_ZERO_W:
      piGpioBase = GPIO_PERI_BASE_OLD ;
      piGpioPupOffset = GPPUD ;
      break ;

    case PI_MODEL_4B:
    case PI_MODEL_400:
    case PI_MODEL_CM4:
      piGpioBase = GPIO_PERI_BASE_2711 ;
      piGpioPupOffset = GPPUPPDN0 ;
      break ;

    default:
      piGpioBase = GPIO_PERI_BASE_2835 ;
      piGpioPupOffset = GPPUD ;
      break ;
  }

  ...

  if ((fd = open ("/dev/mem", O_RDWR | O_SYNC | O_CLOEXEC)) < 0)
  {
    if ((fd = open ("/dev/gpiomem", O_RDWR | O_SYNC | O_CLOEXEC) ) >= 0)    // We're using gpiomem
    {
      piGpioBase   = 0 ;
      usingGpioMem = TRUE ;
    }
    else
      return wiringPiFailure (WPI_ALMOST, "wiringPiSetup: Unable to open /dev/mem or /dev/gpiomem: %s.\n"
    "  Aborting your program because if it can not access the GPIO\n"
    "  hardware then it most certianly won't work\n"
    "  Try running with sudo?\n", strerror (errno)) ;
  }

// Set the offsets into the memory interface.

  GPIO_PADS       = piGpioBase + 0x00100000 ;
  GPIO_CLOCK_BASE = piGpioBase + 0x00101000 ;
  GPIO_BASE   = piGpioBase + 0x00200000 ;
  GPIO_TIMER      = piGpioBase + 0x0000B000 ;
  GPIO_PWM    = piGpioBase + 0x0020C000 ;

...

  //  GPIO:

    gpio = (uint32_t *)mmap(0, BLOCK_SIZE, PROT_READ|PROT_WRITE, MAP_SHARED, fd, GPIO_BASE) ;
    if (gpio == MAP_FAILED)
      return wiringPiFailure (WPI_ALMOST, "wiringPiSetup: mmap (GPIO) failed: %s\n", strerror (errno)) ;

  //  PWM

    pwm = (uint32_t *)mmap(0, BLOCK_SIZE, PROT_READ|PROT_WRITE, MAP_SHARED, fd, GPIO_PWM) ;
    if (pwm == MAP_FAILED)
      return wiringPiFailure (WPI_ALMOST, "wiringPiSetup: mmap (PWM) failed: %s\n", strerror (errno)) ;

  //  Clock control (needed for PWM)

    clk = (uint32_t *)mmap(0, BLOCK_SIZE, PROT_READ|PROT_WRITE, MAP_SHARED, fd, GPIO_CLOCK_BASE) ;
    if (clk == MAP_FAILED)
      return wiringPiFailure (WPI_ALMOST, "wiringPiSetup: mmap (CLOCK) failed: %s\n", strerror (errno)) ;

  //  The drive pads

    pads = (uint32_t *)mmap(0, BLOCK_SIZE, PROT_READ|PROT_WRITE, MAP_SHARED, fd, GPIO_PADS) ;
    if (pads == MAP_FAILED)
      return wiringPiFailure (WPI_ALMOST, "wiringPiSetup: mmap (PADS) failed: %s\n", strerror (errno)) ;

  //  The system timer

    timer = (uint32_t *)mmap(0, BLOCK_SIZE, PROT_READ|PROT_WRITE, MAP_SHARED, fd, GPIO_TIMER) ;
    if (timer == MAP_FAILED)
      return wiringPiFailure (WPI_ALMOST, "wiringPiSetup: mmap (TIMER) failed: %s\n", strerror (errno)) ;

  // Set the timer to free-running, 1MHz.
  //  0xF9 is 249, the timer divide is base clock / (divide+1)
  //  so base clock is 250MHz / 250 = 1MHz.

    *(timer + TIMER_CONTROL) = 0x0000280 ;
    *(timer + TIMER_PRE_DIV) = 0x00000F9 ;
    timerIrqRaw = timer + TIMER_IRQ_RAW ;

  // Export the base addresses for any external software that might need them

    _wiringPiGpio  = gpio ;
    _wiringPiPwm   = pwm ;
    _wiringPiClk   = clk ;
    _wiringPiPads  = pads ;
    _wiringPiTimer = timer ;

...
```
The part of the function associated with memory mapping.<br>
You can see that <span style="{{ site.code }}">wiringPiSetup</span> is trying to map memory using the <span style="{{ site.code }}">/dev/mem</span> and <span style="{{ site.code }}">mmap</span> functions.<br>

First, check the gpio only.<br>
look at the specific binary values of gpio,
```
#define BLOCK_SIZE (4*1024)
...

GPIO_BASE = piGpioBase + 0x00200000;

...

#define GPIO_PERI_BASE_2711 0xFE000000

...
```

Now let's find out what this value means in the data sheet.<br>
At page 66,<br>
<p align="center">
  <img src="/documents/images/wiringpi/gpio_base.png" alt="gpio_base" width="640" height="240"><br>
  <span style="{{ site.img }}">[picture 2] gpio_base</span>
</p>
<br><br>

But look at the code, it's <span style="{{ site.code }}">0xFE200000</span>, not <span style="{{ site.code }}">0x7e200000</span>. Let's find the reason.<br>
At page 5,<br>
<p align="center">
  <img src="/documents/images/wiringpi/address_maps.png" alt="address_maps" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 3] address_maps</span>
</p>
<br>

<span style="{{ site.code }}">0x7e200000</span> is based on a Legacy Master view of Address Map.<br>
It belongs to the 'Main peripherals' area. If you follow the address map with the same offset<br>
<span style="{{ site.code }}">0xFE200000</span> is based on "Low Peripheral" mode.<br><br>

I don't know why, but from now on, we will follow the address system based on 'Low Peripheral' mode.<br>

mmap code
```
  //  GPIO:

    gpio = (uint32_t *)mmap(0, BLOCK_SIZE, PROT_READ|PROT_WRITE, MAP_SHARED, fd, GPIO_BASE) ;
    if (gpio == MAP_FAILED)
      return wiringPiFailure (WPI_ALMOST, "wiringPiSetup: mmap (GPIO) failed: %s\n", strerror (errno)) ;
...

    _wiringPiGpio  = gpio ;

...
```

mmap form
```
#include <sys/mman.h>

void *mmap(void *start, size_t length, int prot, int flags, int fd, off_t offset);
```

I don't cover on mmap because we assume that you have a good understanding in advance<br>
But I mention the important part.<br>

<span style="{{ site.code }}">size_t length</span> indicates the mapping size unit.<br>
The length is <span style="{{ site.code }}">BLOCK_SIZE</span> , The address to be mapped must be a multiple of BLOCK_SIZE.<br>
Our case has GPIO_BASE of 0xFE200000, so there is no problem.<br>

Otherwise, however, you must force the base-address to be a multiple of the nearest <span style="{{ site.code }}">BLOCK_SIZE</span> .<br>
and add or subtract the offset value to add to the bit operation accordingly.<br>
These cases can be found in <span style="{{ site.code }}">odroid-wiringpi: ODROID-C4<span> .<br>

Anyway, It used <span style="{{ site.code }}">mmap</span> to map the gpio variable to the <span style="{{ site.code }}">0xFE200000</span> .<br>

#### read/write pin registers

After <span style="{{ site.code }}">wiringPiSetup</span>, define pin using <span style="{{ site.code }}">pinMode</span> .<br><br>

pinMode
```
void pinMode (int pin, int mode)
{
  int    fSel, shift, alt ;
  struct wiringPiNodeStruct *node = wiringPiNodes ;
  int origPin = pin ;

  setupCheck ("pinMode") ;

  if ((pin & PI_GPIO_MASK) == 0)        // On-board pin
  {
    ...

    fSel    = gpioToGPFSEL [pin] ;
    shift   = gpioToShift  [pin] ;

    /**/ if (mode == INPUT)
      *(gpio + fSel) = (*(gpio + fSel) & ~(7 << shift)) ; // Sets bits to zero = input
    else if (mode == OUTPUT)
      *(gpio + fSel) = (*(gpio + fSel) & ~(7 << shift)) | (1 << shift) ;

    ...

    return ;

```

The values are written in the mapped <span style="{{ site.code }}">gpio</span>,<br>
<span style="{{ site.code }}">fsel</span>, which is the register offset,<br>
and <span style="{{ site.code }}">shift</span>, which is the bit shift value.<br><br>

fsel, shift
```
static uint8_t gpioToGPFSEL [] =
{
  0,0,0,0,0,0,0,0,0,0,
  1,1,1,1,1,1,1,1,1,1,
  2,2,2,2,2,2,2,2,2,2,
  3,3,3,3,3,3,3,3,3,3,
  4,4,4,4,4,4,4,4,4,4,
  5,5,5,5,5,5,5,5,5,5,
} ;


// gpioToShift
//  Define the shift up for the 3 bits per pin in each GPFSEL port

static uint8_t gpioToShift [] =
{
  0,3,6,9,12,15,18,21,24,27,
  0,3,6,9,12,15,18,21,24,27,
  0,3,6,9,12,15,18,21,24,27,
  0,3,6,9,12,15,18,21,24,27,
  0,3,6,9,12,15,18,21,24,27,
  0,3,6,9,12,15,18,21,24,27,
} ;
```

Find what these values mean.<br>
At page 66,<br>
<p align="center">
  <img src="/documents/images/wiringpi/gpfsel.png" alt="gpfsel" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 4] GPFSEL Maps</span>
</p>
<br>

At page 67-70, (gpfsel0 register)<br>
<p align="center">
  <img src="/documents/images/wiringpi/gpfsel0.png" alt="gpfsel0" width="640" height="480"><br>
  <span style="{{ site.img }}">[picture 5] GPFSEL[n]</span>
</p>
<br>

There are up to GPIO57 with the same rules.<br><br>

And then look at the code again.
```
void pinMode (int pin, int mode)
{
  ...

  if ((pin & PI_GPIO_MASK) == 0)        // On-board pin
  {
    ...

    fSel    = gpioToGPFSEL [pin] ;
    shift   = gpioToShift  [pin] ;

    /**/ if (mode == INPUT)
      *(gpio + fSel) = (*(gpio + fSel) & ~(7 << shift)) ; // Sets bits to zero = input
    else if (mode == OUTPUT)
      *(gpio + fSel) = (*(gpio + fSel) & ~(7 << shift)) | (1 << shift) ;

    ...

    return ;

```

Isn't something strange?<br>
<span style="{{ site.code }}">fSel</span> offset are 0x0, 0x4 0x8 ...<br><br>
But, code is<br>

| GPIO | fSel |
| :---: | :---: |
|  0 ~  9 | 0 |
| 10 ~ 19 | 1 |
| 20 ~ 29 | 2 |
| 30 ~ 39 | 3 |
| 40 ~ 49 | 4 |
| 50 ~ 57 | 5 |

Because of the <span style="{{ site.code }}">pointer operation rule</span> in <span style="{{ site.code }}">C</span> .<br>
The value added or subtracted from the pointer variable is calculated<br>
**by multiplying the value by the size of the pointer type, not by the value itself**.<br>

e.g.
```
int *foo = (int *)malloc(sizeof(int));

printf("*(foo) is 0x%x\n", *(foo));
printf("*(foo + 1) is 0x%x\n", *(foo + 1));
```
```
*(foo) is 0x00100000
*(foo + 1) is 0x00100004
```
It might be confusing if you play C after a long time.<br><br>

pinMode
```
    /**/ if (mode == INPUT)
      *(gpio + fSel) = (*(gpio + fSel) & ~(7 << shift)) ; // Sets bits to zero = input
    else if (mode == OUTPUT)
      *(gpio + fSel) = (*(gpio + fSel) & ~(7 << shift)) | (1 << shift) ;
```

if pinMode:GPIO3, shift is 9.<br>
<span style="{{ site.code }}">(7 << shich)</span> is <span style="{{ site.code }}">1110 0000 0000</span> .<br>
This means that you will force the 9th to 11th bit digits to zero.<br>
This part is covered in the [bit operation part](/documents/wiringpi/bit-operation-programming/), so let's move on.<br>

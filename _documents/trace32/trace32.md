---
permalink: /documents/trace32/
title: Trace32
excerpt: "Generic Interrupt Controller"
comments: false
toc: true
---

## Start Trace32

Introduce a rough order and checklist to start.<br>
For the most basic debugger document, see [trace32-base](/documents/trace32-base).<br>

### Quick Start Guide

Approximate order using the Trace32 debugger.
```
1. Install 'Trace32 PowerView' with T32 License.
2. Fasten the JTAG pin 1 to the TRACE32 connector so that the position of pin 1 is well matched,
   and the boot order follows tracer → board. The exit order follows board → tracer.
3. Run 'Trace32 PowerView'
4. Set Core Config and Break
   If, you have your cmm script for target, Run cmm script with B:: pedit {your_cmm_script}
   and push '> Go'
```
<br>

### Core Config: checklist

Check out the following factors.<br>
If any of the following lists are applicable, a debugger is not available.
```
1. The target device has no power, or the debug cable is not connected:
   In this case, the error message "target power fail" will appear.

2. The correct core type is not selected:
   Use the command SYStem.CPU <type> to check if the correct core is selected.

3. JTAG interface issue:
   Refer to the target’s schematic or manual to check the physical and electrical interface.
   You may need to set jumpers on the target to ensure the correct signals are connected to the JTAG connector.

4. Debug features need to be enabled:
   For example, if the nTRST signal is directly connected to ground on the target side, it may not work.

5. The target is in an unrecoverable state:
   Power cycle the target and try again.

6. The target cannot communicate with the debugger while in reset:
   In this case, try using the SYStem.Mode Attach command followed by "Break" or use SYStem.Option EnReset OFF.

7. The JTAG clock speed is too fast:
   Especially when using emulated cores or FPGA-based targets, try SYStem.JtagClock 50kHz.
   Once it works, optimize the speed.

8. The core has no power.

9. The core has no clock.

10. The core remains in reset.

11. The target requires setup time after reset release:
    By default, the SYStem.Up command tries to catch the CPU at the reset vector,
    leaving no time for initialization. Use 'SYStem.Option ResBreak' or 'SYStem.Option WaitReset'
    to grant the target setup time.

12. Core-base or CLI-base is not set:
    Use the SYStem.CONFIG COREDEBUG Base and SYStem.CONFIG CLI Base commands to set them.
    The Core-base is needed for communication, and the CLI-base is needed for start/stop control.

13. The watchdog timer is not disabled.

14. The target requires special debugger settings:
    Check if there is a suitable script file (*.cmm) for your target.

15. There are multiple TAP controllers in the JTAG chain:
    For example, the TAP of the DAP could be in a chain with other TAPs from other CPUs.
    In this case, check your pre and post bit configuration by using commands such as
    SYStem.CONFIG IRPRE or SYStem.CONFIG DAPIRPRE.
```
<br>

# Flashing code to the STM32F4 Microcontroller (uC)
 The following is a guide created to help my software team on [UBC's Biomedical Engineering Student Team](http://best.ece.ubc.ca) to target and develop for the STM32F4 Discovery board.

---------
## Overview
What does each component do?

 - OpenOCD
  - Communicates between the uC and the computer
 - Telnet
  - Communicates between your shell and OpenOCD
 - arm-none-eabi-gdb
  - Communicates with the target device (STM32F4 chip) through OpenOCD


---------
## Connecting to the uC
With the STM32f4 board plugged in, You should now be able to run the following. I recommend running this command from the directory your project is in as later commands will use this as the working directory.
```
$ openocd -f board/stm32f4discovery.cfg
```
and expect to see:

```
Open On-Chip Debugger 0.8.0 (2014-09-27-18:32)
Licensed under GNU GPL v2
For bug reports, read
  http://openocd.sourceforge.net/doc/doxygen/bugs.html
srst_only separate srst_nogate srst_open_drain connect_deassert_srst
Info : This adapter doesn't support configurable speed
Info : STLINK v2 JTAG v14 API v2 SWIM v0 VID 0x0483 PID 0x3748
Info : using stlink api v2
Info : Target voltage: 2.907282
Info : stm32f4x.cpu: hardware has 6 breakpoints, 4 watchpoints
```
---------
## Flashing the code
And in a second window, connect to OpenOCD and flash the code. In this case (vedder's) [demo](http://vedder.se/2012/07/usb-serial-on-stm32f4/) code.
```
$ telnet localhost 4444
> flash write_image erase build/stm32f4_usb_cdc.bin 0x8000000
> reset
```

If you receive the following error:
```
target not halted
```
Run the following commands through `arm-none-eabi-gdb` to halt the controller:
```
$ arm-none-eabi-gdb
> set remote hardware-breakpoint-limit 6
> set remote hardware-watchpoint-limit 4
> target extended-remote :3333
> monitor reset halt
```

### Viewing the results
Once the device is flashed and reset, it shows up as a /dev/ttyACM0 device if you have a Linux host. Connect to it with the following from the host, or if you want, enable a filter so the VM can do it.
```
$ sudo screen /dev/ttyACM0
```

Expect to see data from the microcontroller being output to the screen. As an exercise, compare the output to what the project source code says it should output to determine correctness.

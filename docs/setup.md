# Guide to setup a STM32F4 Development Environment on Fedora 20 (VirtualBox)
 The following is a guide created to help my software team on [UBC's Biomedical Engineering Student Team](http://best.ece.ubc.ca) to target and develop for the STM32F4 Discovery board.

----------
## Configuring VirtualBox (VM Only)

 1. Enable USB 2.0 (EHCI in VirtualBox, Requires Vbox extension)
 2. Add a USB device (filter) for STlink programmer(if some characters show up as garbage, just delete them)

Note - On a linux host, you may have to add yourself to the group "vboxusers" before you can see any USB devices in virtualbox. ([Original Article](http://askubuntu.com/questions/25596/how-to-set-up-usb-for-virtualbox))

 1. `$ sudo usermod -aG vboxusers YOUR_LINUX_HOST_USERNAME`
 2. Logout or reboot
 3. Check `$ lsusb` in your VM to ensure you have something like `Bus 002 Device 003: ID 0483:3748 STMicroelectronics ST-LINK/V2`.


----------
## Installing the Development Tools

```
$ sudo yum update
$ sudo yum install automake make gcc kernel-headers kernel-devel libusb-devel glibc.i686 ncurses-static.i686 screen nano wget telnet
$ sudo yum groupinstall "Development Tools"
```

kernel-headers kernel-devel | VBOX Guest Additions
libusb-devel | openOCD
glibc.i686 | arm-none-eabi
screen | connect to serial port

----------
## Install Virtual Box Guest Additions (VM Only)

reboot if you haven't
```
$ cd /run/media/BEST/VBOXADDITIONS*
$ sudo ./VBoxLinuxAdditions.run
```
----------
### Installing the GCC ARM Cross Compiling Toolchain

Download the newest gcc-arm from here https://launchpad.net/gcc-arm-embedded/+download
I've used the newest toolchain,  as of 2014-09-27, in the example below.

```
$ cd ~/Downloads
$ wget https://launchpad.net/gcc-arm-embedded/4.8/4.8-2014-q2-update/+download/gcc-arm-none-eabi-4_8-2014q2-20140609-linux.tar.bz2
$ tar xjf gcc-arm-none-eabi-4_8-2014q2-20140609-linux.tar.bz2
$ sudo mv gcc-arm-none-eabi-4_8-2014q2 /opt
$ cd /opt
$ chmod go-w -R gcc-arm-none-eabi-4_8-2014q2/
$ sudo chown -R root:root gcc-arm-none-eabi-4_8-2014q2/
```
#### Add gcc-arm-none-eabi to our PATH
This allows us to invoke the tools by typing `gcc-arm-none-eabi-gcc` in our shell.
```
$ cd ~/
$ nano .bash_profile
```

and append this:
```
:/opt/gcc-arm-none-eabi-4_8-2014q2/bin
```
to the end of the line starting with:
```
PATH=$PATH:
```
It should look something like the following when done...
```
PATH=$PATH:$HOME/.local/bin:$HOME/bin:/opt/gcc-arm-none-eabi-4_8-2014q2/bin
```
then  type `Ctrl+o`, `Enter`, and `Ctrl+x` to save and quit.

Finally, run the following command. This is only required in new shell windows until you logout/reboot. Normally, `.bash_profile` is loaded around login.
```
$ source .bash_profile
```

Test to see that you can run the commands by typing:
```
$ arm-none-eabi-gcc -v
```


----------
### Installing OpenOCD
Download the latest OpenOCD from here http://sourceforge.net/projects/openocd/files/openocd/

```
$ cd ~/Downloads
$ wget http://sourceforge.net/projects/openocd/files/openocd/0.8.0/openocd-0.8.0.tar.bz2
$ tar xjf openocd-0.8.0.tar.bz2
$ cd openocd-0.8.0
$ ./configure
```

This is what you want, "yes" for ST-Link, after running configure... (May need to install `libusb-devel` or `libusb` otherwise and try again).
```
OpenOCD configuration summary
--------------------------------------------------
MPSSE mode of FTDI based devices        yes (auto)
ST-Link JTAG Programmer                 yes (auto)
TI ICDI JTAG Programmer                 yes (auto)
Keil ULINK JTAG Programmer              yes (auto)
Altera USB-Blaster II Compatible        yes (auto)
Segger J-Link JTAG Programmer           yes (auto)
OSBDM (JTAG only) Programmer            yes (auto)
eStick/opendous JTAG Programmer         yes (auto)
Andes JTAG Programmer                   yes (auto)
Versaloon-Link JTAG Programmer          yes (auto)
USBProg JTAG Programmer                 yes (auto)
Raisonance RLink JTAG Programmer        yes (auto)
Olimex ARM-JTAG-EW Programmer           yes (auto)
CMSIS-DAP Compliant Debugger            no
```

Then continue to run the following:
```
$ make
$ sudo make install
$ make installcheck
```

#### Enabling permissions for USB
To run openocd without root, we add a rule to udev and reload it.
```
$ echo 'ATTRS{idProduct}=="3748", ATTRS{idVendor}=="0483", MODE="666"' | sudo tee /etc/udev/rules.d/91-openocd.rules
$ sudo udevadm trigger
```

With the STM32f4 board plugged in, You should now be able to run the following:
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


----------
## Putting it all together
This was cloned/adapted from the following [fantastic STM32F4 setup guide](https://github.com/Ell-i/Wiki/wiki/STM32F4-setup). I'm in no way affiliated, and wish to give them all the credit they deserve for a nice write up.

### Compiling the code

We are going to use someone's (vedder's) [demo](http://vedder.se/2012/07/usb-serial-on-stm32f4/) to test that our software is installed correctly. The first line creates a symbolic link to our toolchain, this is because many people in the past have used a different toolchain called "summon-arm-toolchain" or "sat" and we don't want to modify every makefile they use.
```
$ ln -s /opt/gcc-arm-none-eabi-4_8-2014q2 ~/sat
$ cd ~/Downloads
$ wget http://vedder.se/wp-content/uploads/2012/07/STM32F4_USB_CDC.zip
$ unzip STM32F4_USB_CDC.zip
$ cd STM32F4_USB_CDC
$ make
```

If the build succeeded you should have these files in "build":
```
$ ls build
stm32f4_usb_cdc.bin  stm32f4_usb_cdc.elf  stm32f4_usb_cdc.hex
```

### Flashing the code to the STM32F4
Connect a USB A to USB Mini B cable from target board port CN1 to the PC and a USB A to USB Micro B cable from target board port CN5 port to the PC. Next open two terminal windows, in the first we will run the OpenOCD "server". One thing to keep in mind, is that when you run telnet commands in the next step, they will be relative to where you ran openocd from.
```
$ openocd -f board/stm32f4discovery.cfg
```

And in the second window, connect to openOCD and flash the code.
```
$ telnet localhost 4444
> flash write_image erase build/stm32f4_usb_cdc.bin 0x8000000
> reset
```

### Viewing the results
Once the device is flashed and reset, it shows up as a /dev/ttyACM0 device if you have a Linux host. Connect to it with the following from the host, or if you want, enable a filter so the VM can do it.
```
$ sudo screen /dev/ttyACM0
```

### Using GDB
GDB can be used with the target through OpenOCD, although it's a bit buggy; attaching and tracing a running binary will fail. This can be worked around automatically by inserting following content to ~/.gdbinit file:
```
set remote hardware-breakpoint-limit 6
set remote hardware-watchpoint-limit 4
target extended-remote :3333
monitor reset halt
load
monitor reset halt
```
Test the debugger by inserting a breakpoint in the beginning of the neverending loop function:
```
$ arm-none-eabi-gdb build/stm32f4_usb_cdc.elf
(gdb) break main.c:49
(gdb) continue
(gdb) backtrace
(gdb) list
(gdb) print GPIO_SetBits
(gdb) continue
```


### Links & References

 1. [A fantastic STM32F4 setup guide](https://github.com/Ell-i/Wiki/wiki/STM32F4-setup)
 2. [OpenOCD User Manual](http://openocd.sourceforge.net/doc/html/index.html)
 3. [Debugging With GDB](https://sourceware.org/gdb/current/onlinedocs/gdb/)
 4. [MP3 player example](http://vedder.se/2012/12/stm32f4-discovery-usb-host-and-mp3-player/)
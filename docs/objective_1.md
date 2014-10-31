# Objective 1
 The following is a guide created to help my software team on [UBC's Biomedical Engineering Student Team](http://best.ece.ubc.ca) to target and develop for the STM32F4 Discovery board.

---------
## Overview
This module is intended to get familiar with IO, USB, and Make.

 - IO
   - Use the push button to toggle which LEDs light up
 - USB
   - Write the current status over usb to the serial console
 - Make
   - Configure to compile and use new files "b_io.h" and "b_io.c" where our nice io code should end up.


## Hints
 - Use "stm32f4_template" and "examples/stm32f4_usb_cdc" in BEST
 - Compile very often, mistakes will show up sooner
 - "diff" is used to compare files and directories. Compare stm32f4_template and examples/stm32f4_usb_cdc" to see what was added for usb functionality.
 - Commit to git often, even locally, it allows you to restore to previous working versions, and leave good comments.
   - $`git add "file"`
   - or to add all files $`git add -A`
   - $`git commit -m "a meaningful message"

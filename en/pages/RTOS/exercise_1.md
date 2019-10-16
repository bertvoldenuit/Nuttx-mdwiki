# Exercice 1: Blinking leds

The purpose of this exercice is to get hands on Nuttx. The first exercise, shows how to make (modify) the first application. Nuttx offers a lot of Application examples already set. Here, the `leds` example is perfect for what is needed to do:
- The Application is set in the 'Menu Config'
- The application file `leds_main.c` is easy to read and to modify.

At first, the aim is to make one LED to blink. That's all!

## Preliminaries

Windows WSL user should set the `nuttxspace` in the Windows `home` directory of their choice. For exemple, the following alias could be set:

```
alias win_home='cd /mnt/c/<win_home>'
```

Then to go to `nuttxspace`:

```
cd win_home/nuttxspace
```

For concistency, the Linux notation will be used. The `nuttxspace` directory is located in the `home` directory. The following shortcut is used: `~`

```
cd ~/nuttxspace
```

The exercice can be adapted for an other board using the following table:

| Replace          | by                                                                            |
|------------------|-------------------------------------------------------------------------------|
| `<board_name>` | your board's name   (e.g. `<board_name>`/src --> **nucleo-f4x1re**/src)     |
| `<arch_name>`  | MCU familly or arch   (e.g. `<arch_name>`_ssd1306.c -> **stm32**_ssd1306.c) |


## Nuttx configuration

The STM32F4Discovery should be set with the micro-usb port enabled, so let's use the pre-configuration option `usbnsh`

```
cd ~/nuttxspace/nuttx
make distclean
cd tools/
./configure.sh stm32f4discovery:usbnsh
cd ..
```

For other boards, the above can be adapted using your board_name instead of stm32f4discovery and using the normal pre-configuration `nsh`:

```
./configure.sh <board_name>:nsh
```

It's time to configure the `leds` Application in the `Menu Config`. Type:

```
make menuconfig
```

  |       Tree root           | Sublevel 1                  | Sublevel 2                   |  Sublevel 3             |   Sublevel 4            |Enable |
  | ------------------------- |-----------------------------|----------------------------- | ----------------------- |  ---------------------- |:-----:|
  | Build Setup               | Build Host Platform         | Linux                        |                         |                         | yes   |
  | Board Selection           | Board LED Status support    |                              |                         |                         | `no`  |  
  | Board Selection           | Enable boardctl() interface |                              |                         |                         | yes   |  
  | Device Drivers            | LED Support                 | LED driver                   |                         |                         | yes   |
  | Device Drivers            | Generic Lower Half LED Driver|                             |                         |                         | yes   | 
  | Application Configuration |  Example                    | LED driver example           |                         |                         | yes   |

Exit and Save

## Leds application modification

Let's open and modify the `leds_main.c`. But first, it is recommended to save a backup:

```
cd ~/nuttxspace/app
cp leds_main.c leds_main_bak.c
```

Edit `leds_main.c` with your favorite editor. The basic structure of the file is:

- Include files
- Led daemon function which provide algorithm to turn on and off leds
- Main function which is called, when you type the name of the application `leds` in the Nutt Shell, and which calls the Led daemon

For this first exercice, let's make it dead simple. The Led daemon is erased and the source code in between the main function brackets is also erase. The source code should look like this:

```c
/****************************************************************************
 * examples/leds/leds_main.c
 *
 *   Copyright (C) 2016 Gregory Nutt. All rights reserved.
 *   Author: Gregory Nutt <gnutt@nuttx.org>
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 *
 * 1. Redistributions of source code must retain the above copyright
 *    notice, this list of conditions and the following disclaimer.
 * 2. Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in
 *    the documentation and/or other materials provided with the
 *    distribution.
 * 3. Neither the name NuttX nor the names of its contributors may be
 *    used to endorse or promote products derived from this software
 *    without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
 * "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
 * LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
 * FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
 * COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
 * INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING,
 * BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS
 * OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED
 * AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
 * LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN
 * ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
 * POSSIBILITY OF SUCH DAMAGE.
 *
 ****************************************************************************/

/****************************************************************************
 * Included Files
 ****************************************************************************/

#include <nuttx/config.h>

#include <sys/ioctl.h>
#include <stdbool.h>
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>
#include <sched.h>
#include <errno.h>

#include <nuttx/leds/userled.h>

/****************************************************************************
 * leds_main
 ****************************************************************************/

int main(int argc, FAR char *argv[])
{

  return EXIT_SUCCESS;
}

```

This is how to create a variable that drive the LED: 

```c
struct userled_s led1; // create the variable 'led'
```

In ```~/nuttxspace/nuttx/board/arm/stm32f4discovery/src/stm32_userleds.c```, the LED number is mapped to the GPIO as the following array :

{GPIO_LED1, GPIO_LED2, GPIO_LED3, GPIO_LED4}    

```c
led1.ul_led = 0;       /* Identifies the LED. 0 -> GPIO_LED1; 1 -> GPIO_LED2, etc... */
led1.ul_on = true;		 /* The LED state.  true: ON; false: OFF */
```

To open the led driver:

```c
int fd;
fd = open(CONFIG_EXAMPLES_LEDS_DEVPATH, O_WRONLY);
```

To actually set the LED state:

```c
ret = ioctl(fd, ULEDIOC_SETLED, &led1);
```

a simple delay function in microseconds:
```c
usleep();
```

so the initialization of the main function should look like this:

```c
int main(int argc, FAR char *argv[])
{
  struct userled_s led1;
  int ret;
  int fd;

  led1.ul_led = 0;
  led1.ul_on = 1;
   
  // Open the LED driver 

  printf("led_driver: Opening %s\n", CONFIG_EXAMPLES_LEDS_DEVPATH);
  fd = open(CONFIG_EXAMPLES_LEDS_DEVPATH, O_WRONLY);
  if (fd < 0)
    {
      printf("led_driver: ERROR: Failed to open\n");
      return EXIT_FAILURE;
    }
 
/* Put your user code here */

  return EXIT_SUCCESS;
}
```

## Running the `leds` application

Once you have implemented your code, save and exit the file. If you feel like it, you can make all the leds blink like an LED chaser.
Then compile sources:

```
cd ~/nuttxspace/nuttx
make
```

```
cd ~/nuttxspace/nuttx
cd nuttx
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg -c init -c "reset halt" -c "flash write_image erase nuttx.bin 0x08000000"
```

** Important : Reset the board before connecting the micro USB cable**

launch `minicom` or `TeraTerm`, set the com port as explained in [installing nuttx Linux](../getting-started/installing_nuttx.md) or [installing nuttx Windows WSL](../getting-started/installing_nuttx_WSL.md)

** Important: for stm32f4discovery boad it is better not to reset the board at this point **

Hit `Enter` 3 times you should get :

```
help usage:  help [-v] [<cmd>]


  [         cd        echo      hexdump   mh        rmdir     time      xd        

  ?         cp        exec      kill      mv        set       true      

  basename  cmp       exit      ls        mw        sh        uname     

  break     dirname   false     mb        pwd       sleep     unset     

  cat       dd        help      mkdir     rm        test      usleep    


Builtin Apps:

leds

nsh> 

```

Type `leds`


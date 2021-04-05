# Exercice 1: Periodic single task

The purpose of this exercice is to create a periodic single task on Nuttx. It describes how to create a task and implement the task. The aim is to make an LED blink using Nuttx RTOS capabilities.

This exercise is based on ```Real-time Operating Systems Book 2 - The Practice, by Jim Cooling``` chapter 2 - exercise 1

## Preliminaries and Nuttx configuration
-----------------------------------------

In order to configure Nuttx for this exercise, visit [Preliminary Exercice](pages/RTOS/preliminary_exercise.md)

## Leds application modification
---------------------------------

Let's open and modify the `leds_main.c`. But first, it is recommended to save a backup:

```
cd ~/nuttxspace/app
cp leds_main.c leds_main_bak.c
```

Edit `leds_main.c` with your favorite editor. The basic structure of the file is:

- Include files
- Led daemon function which provide algorithm to turn on and off leds
- Main function which is called, when you type the name of the application `leds` in the Nutt Shell, and which calls the Led daemon

For this first exercice, `main` function is kept as is. In the `led_daemon` function, remove the code between the brackets of the `for` loop, remove code from `Get the set of LEDs supported` up until `for` loop and also and also the 3 first variable declaration. The source code should look like this:

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
#include<time.h>

#include <nuttx/leds/userled.h>

/****************************************************************************
 * Private Data
 ****************************************************************************/

static bool g_led_daemon_1_started;
static bool g_led_daemon_2_started;

/****************************************************************************
 * Private Functions
 ****************************************************************************/

/****************************************************************************
 * Name: led_daemon
 ****************************************************************************/


static int led_daemon(int argc, char *argv[])
{
  <Put your LED variable(s)>
  int ret;
  int fd;

  /* Indicate that we are running */

  g_led_daemon_started = true;
  printf("led_daemon: Running\n");

  /* Open the LED driver */

  printf("led_daemon: Opening %s\n", CONFIG_EXAMPLES_LEDS_DEVPATH);
  fd = open(CONFIG_EXAMPLES_LEDS_DEVPATH, O_WRONLY);
  if (fd < 0)
    {
      int errcode = errno;
      printf("led_daemon: ERROR: Failed to open %s: %d\n",
             CONFIG_EXAMPLES_LEDS_DEVPATH, errcode);
      goto errout;
    }


  for (; ; )
    {
      <Put your code to blink LED here>
    }

errout_with_fd:
  (void)close(fd);

errout:
  g_led_daemon_started = false;

  printf("led_daemon: Terminating\n");
  return EXIT_FAILURE;
}


/****************************************************************************
 * Public Functions
 ****************************************************************************/

/****************************************************************************
 * leds_main
 ****************************************************************************/

int main(int argc, FAR char *argv[])
{
  int ret;

  printf("leds_main: Starting the led_daemon\n");
  if (g_led_daemon_started)
    {
      printf("leds_main: led_daemon already running\n");
      return EXIT_SUCCESS;
    }

  ret = task_create("led_daemon", CONFIG_EXAMPLES_LEDS_PRIORITY,
                    CONFIG_EXAMPLES_LEDS_STACKSIZE, led_daemon,
                    NULL);
  if (ret < 0)
    {
      int errcode = errno;
      printf("leds_main: ERROR: Failed to start led_daemon: %d\n",
             errcode);
      return EXIT_FAILURE;
    }

  printf("leds_main: led_daemon started\n");
  return EXIT_SUCCESS;
}

```

### The `main` function
---------------------------

This is how to create a task that drive the LED, see [Nuttx Documentation](https://nuttx.apache.org/docs/latest/reference/user/01_task_control.html?highlight=task_create#c.task_create): 

```c
int task_create(char *name, int priority, int stack_size, main_t entry, char * const argv[]);
```
 Input Parameters:

- name. Name of the new task:                     "Led deamon"
- priority. Priority of the new task:             "100" This is the default value in Nuttx. The higher value the more the priority.
- stack_size. size (in bytes) of the stack needed:"CONFIG_EXAMPLES_LEDS_STACKSIZE" Keep default value. Here it is 2048 Bytes.
- entry. Entry point of a new task:               "led_deamon"
- argv. A pointer to an array of input parameters. The array should be terminated with a NULL argv[] value. If no parameters are required, argv may be NULL. 

Returned Value:

- Returns the non-zero task ID of the new task or ERROR if memory is insufficient or the task cannot be created (errno is not set).

Stack is a task dedicated amount of RAM. It is used to save the context of the task when it is interrupted. It is reloaded when the task is resumed.    

### The `led_deamon` function
-------------------------------

#### Variable declaration

This is how to create a variable that drive the LED: 

```c
struct userled_s led1; // create the variable 'led1'
```   

#### To open the led driver:

```c
int fd;
fd = open(CONFIG_EXAMPLES_LEDS_DEVPATH, O_WRONLY);
```

#### Set LED state
```c
led1.ul_led = 0;       /* Identifies the LED. 0 -> GPIO_LED1; 1 -> GPIO_LED2, etc... */
led1.ul_on = true;     /* The LED state.  true: ON; false: OFF */
```

In ```~/nuttxspace/nuttx/board/arm/stm32f4discovery/src/stm32_userleds.c```, the LED number is mapped to the GPIO as the following array :

{GPIO_LED1, GPIO_LED2, GPIO_LED3, GPIO_LED4} 

To actually set the LED state:

```c
ret = ioctl(fd, ULEDIOC_SETLED, &led1);
```

#### Delay

A simple delay function in microseconds:
```c
usleep();
```

Here is an interresting article in the Nuttx Documentation: [Short Time Delays](https://cwiki.apache.org/confluence/display/NUTTX/Short+Time+Delays)

usleep()'s behavior is to wait to assure that at least usec microseconds has elapsed. It will suspend the calling thread (led_deamon task) for at least usec microseconds with some microseconds of precision (see [Short Time Delays](https://cwiki.apache.org/confluence/display/NUTTX/Short+Time+Delays)). 

#### `for` loop

Inside the `for` loop, make the led(s) blink(s) using the functionnalities described above. For example, make the LED blinking with a `ON` period of 2000ms and `OFF` period of 500ms.

## Running the `leds` application
----------------------------------

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



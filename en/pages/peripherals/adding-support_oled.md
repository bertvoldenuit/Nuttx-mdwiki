# Adding OLED support

## Tutorial

As a basis, I used the following tutorial which is based on stm32f103-minimum board:
 
https://www.youtube.com/watch?v=TZ4i71ArtRo


## Porting to an other board

We will see how to port the OLED support to an other board. I will use the nucleo-f411re (having a stm32 architecture MCU) board but the following can be done by replacing : 

| Replace          | by                                                                            |
|------------------|-------------------------------------------------------------------------------|
| **<board_name>** | your board's name   (e.g. **<board_name>**/src --> **nucleo-f4x1re**/src)     |
| **<arch_name>**  | MCU familly or arch   (e.g. **<arch_name>**_ssd1306.c -> **stm32**_ssd1306.c) |


**Make sure your board initialization is [standardized](https://github.com/bertvoldenuit/NuttX-Wiki/wiki/peripherals_initialize)**

First let's check if <arch_name>_ssd1306.c exists, from your home directory type:

```
cd nuttxspace/nuttx/configs/<board_name>/src
ls <arch_name>_ssd1306.c
```

If it does not exist then copy it into configs/<board_name>/src :

```
cd nuttxspace/nuttx/configs/<board_name>/src
cp ../../stm32f103-minimum/src/stm32_ssd1306.c <arch_name>_ssd1306.c
```

then edit <arch_name>_ssd1306.c, and change :

`#include "stm32f103_minimum.h"`

by

`#include "<board_name>.h"`


let's check i2c or spi pin configurations in board.h. 

```
cd nuttxspace/nuttx/configs/<board_name>/include
```

The nucleo-f411re has several output options:
* Push-pull or Open-drain
* Pullup or pulldown can be on/off

In my case I use I2C. board.h sets I2C pins in open-drain so pullup resistors need to be added:
```
#define GPIO_I2C2_SCL_GPIO \
   (GPIO_OUTPUT|GPIO_OPENDRAIN|GPIO_SPEED_50MHz|GPIO_OUTPUT_SET|GPIO_PORTB|GPIO_PIN10)
#define GPIO_I2C2_SDA_GPIO \
   (GPIO_OUTPUT|GPIO_OPENDRAIN|GPIO_SPEED_50MHz|GPIO_OUTPUT_SET|GPIO_PORTB|GPIO_PIN11)
```

Now we need to configure the os.

From your home directory you can type:

```
cd nuttxspace/nuttx/
make menuconfig
```

  |       Tree root           | Sublevel 1                  | Sublevel 2                   |  Sublevel 3             |   Sublevel 3            |Enable |
  | ------------------------- |-----------------------------|----------------------------- | ----------------------- |  ---------------------- |:-----:|
  | System Type               | STM32 Peripheral Support    | I2C1                         |                         |                         | yes   |
  | Board Selection           | Enable boardctl() interface |                              |                         |                         | yes   |  
  | Device Drivers            | I2C Driver Support          |                              |                         |                         | yes   |
  | Device Drivers            | Video Device Support        |                              |                         |                         | yes   |
  | Device Drivers            | Video Device Support        | Framebuffer character driver |                         |                         | yes   |
  | Device Drivers            | LCD Driver Support          | Graphic LCD Driver Support   |                         |                         | yes   |
  | Device Drivers            | LCD Driver Support          | Graphic LCD Driver Support   |LCD framebuffer front end| LCD framebuffer front end | yes   |
  | Device Drivers            | LCD Driver Support          | Graphic LCD Driver Support   | LCD driver selection    |  UG-2864HSWEG01 OLED... | yes   |
  | Device Drivers            | LCD Driver Support          | Graphic LCD Driver Support   | LCD driver selection    |  SSD1306 Interface...   | I2C   |  
  | Application Configuration |  NSH Library                | NSH Library                  |                         |                         | yes   |
  | Application Configuration |  NSH Library                | Have arch-specific init      |                         |                         | yes   |
  | Application Configuration |  Example                    | Framebuffer driver example   |                         |                         | yes   |

Build the sources:

```
make
```
![image](https://user-images.githubusercontent.com/5957713/54497293-3f71f400-48f9-11e9-93f6-ed82098c881c.png)
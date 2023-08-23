# Installing NuttX


This tutorial is based on [cmn-lab tutorial](http://cmn-lab.com/first_contact_with_nuttx)
You can also refer to [NuttX Channel](https://www.youtube.com/watch?v=heSkSd-_70g&list=PLd73yQk5Fd8JEsVD-lhwYRQKVu6glfDa8&index=1)

In this tutorial, the Nucleo-F411re is used.

![image](https://user-images.githubusercontent.com/5957713/56863985-87d30600-69bd-11e9-98d2-30ab0db4a137.png)

## Installing the environment tools and NuttX RTOS

Open a terminal

1- Update and Upgrade your Linux distribution
```
sudo apt-get update && sudo apt-get upgrade
```

2- Install compiler 
```
sudo apt-get install automake bison build-essential flex gcc-arm-none-eabi gperf git libncurses5-dev libtool libusb-dev libusb-1.0.0-dev
```

3- Create `Nuttxspace` directory and go to it
```
mkdir ~/nuttxspace
cd ~/nuttxspace
```

4- Clone OpenOCD (Open On Chip Debugger) repository (see [git documentation](https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control))
This will allow to connect to the programmer/debugger stlink-v2 on the Nucleo board
```
git clone http://repo.or.cz/r/openocd.git
```

5- Configure and install OpenOCD
```
cd ~/nuttxspace/openocd
./bootstrap
./configure --enable-internal-jimtcl --enable-maintainer-mode --disable-werror --disable-shared --enable-stlink --enable-jlink --enable-rlink --enable-vslink --enable-ti-icdi --enable-remote-bitbang
make
sudo make install
```

6- Clone NuttX repository
```
cd ~/nuttxspace/
git clone https://github.com/apache/nuttx.git nuttx
git clone https://github.com/apache/nuttx-apps apps
git clone https://bitbucket.org/nuttx/tools
```

7- Configure Nuttx
```
cd tools/kconfig-frontends/
./configure
make
sudo make install
sudo ldconfig
cd ~/nuttxspace/nuttx/
make distclean
cd tools/
./configure.sh nucleo-f4x1re/f411-nsh
cd ..
make
```
As a result the `nuttx.bin` file is created. 

## Flashing target device

Plug the Nucleo board to the computer with the usb cable.

If you are using a Linux distribution on a Virtual Machine on Windows, the device will be recognize first by Windows.
- Go to the VM taskbar
- Select Devices
- Then USB
- Select your Device. For example `STMicroeletronics STM32.....`
- The device is now recognized by Linux

Let's look at how to flash the target board. With a Nucleo board there are two ways of doing it.

+ Nucleo board is seen by linux as a flash drive. Simply drag and drop `nuttx.bin` at the root of the device. You're done!
+ Use openocd to use the programer/debugger to flash the target board. The following is done once for all. It will give 'openocd' access and permission to the device (stlink-v2). It will be very useful later to use stlink-v2 as a debugger.

```
cd ~/nuttxspace/
cd openocd/contrib/
sudo cp 60-openocd.rules /etc/udev/rules.d/
sudo udevadm trigger
```

Let's test the connection:
```
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg
```
press ctrl-c to exit

If you get error messages, this is quite normal.

Press ctrl-c to exit

Just push and hold Reset button while executing the command again:
```
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg
```
Now you should get:
```
Open On-Chip Debugger 0.10.0+dev-00807-gbbdb820c (2019-04-27-15:34)
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
adapter speed: 2000 kHz
adapter_nsrst_delay: 100
none separate
Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : clock speed 2000 kHz
Info : STLINK V2J33M25 (API v2) VID:PID 0483:374B
Info : Target voltage: 3.260646
Info : stm32f4x.cpu: hardware has 6 breakpoints, 4 watchpoints
Info : Listening on port 3333 for gdb connections
```
Let go the reset button

Stlink-v2 is synchronized

Press ctrl-c to exit


In order to flash the MCU (Microcontroller Unit) memory, use the following command:
```
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg -c init -c "reset halt" -c "flash write_image erase nuttx.bin 0x08000000"
```

## NuttShell
In order to execute commands on the NuttX Operating System, we must connect to the NuttShell. Let's setup a serial terminal.

```
sudo apt-get install minicom
sudo minicom
`ctrl-A` then `Z` to get to the menu
Press `O` and select `Serial port setup`
Press `A` and change `/dev/tty8` by `/dev/ttyACM0` or whichever your configuration is, 'Enter' twice
Select `Screen and keyboard`
Press `P` and `Q` to set linefeed and local echo and press `Enter`
Exit
Press `ctrl A E` to toggle local echo (otherwise each key strike is repeated twice)
Press `ctrl-A H` to Hangup and reset
Press reset button on the Nucleo board
```

Now you should have:
```
NuttShell (NSH)                                                                 
                                                                                
nsh> 
```

Type `help` to get the command list:
```
help usage:  help [-v] [<cmd>]


  [         cd        echo      hexdump   mh        rmdir     time      xd        

  ?         cp        exec      kill      mv        set       true      

  basename  cmp       exit      ls        mw        sh        uname     

  break     dirname   false     mb        pwd       sleep     unset     

  cat       dd        help      mkdir     rm        test      usleep    


Builtin Apps:

nsh> 

```

Press `ctrl-A X` to exit minicom

## Hello world Application

In NuttX, a program is considered as an Application which can be launched by the nuttshell.
NuttX comes with many application examples. MenuConfig is the place to configure the NuttX OS and select Applications.
It is a graphical text based interface.
Let's try the `Hello World` Application example:

```
cd ~/nuttxspace/nuttx/
make menuconfig
```

Select `Application Configuration` then `Examples` and `Hello World example`

|       Tree root           | Sublevel 1                  | Sublevel 2                   |Enable |
| ------------------------- |-----------------------------|----------------------------- |:-----:|
| Application Configuration |  Examples                   | Hello World example          | yes   |

![image](https://user-images.githubusercontent.com/5957713/56863585-80f5c480-69b8-11e9-9ff8-139ab3a06da6.png)

Exit and save

Compile NuttX and flash the Nucleo board:

```
make
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg -c init -c "reset halt" -c "flash write_image erase nuttx.bin 0x08000000"
sudo minicom
Press reset button on the Nucleo board
```

Type `help` to get the command list:
```
help usage:  help [-v] [<cmd>]


  [         cd        echo      hexdump   mh        rmdir     time      xd        

  ?         cp        exec      kill      mv        set       true      

  basename  cmp       exit      ls        mw        sh        uname     

  break     dirname   false     mb        pwd       sleep     unset     

  cat       dd        help      mkdir     rm        test      usleep    


Builtin Apps:

hello

nsh> 

```

Note that `hello` Application is present

Type `hello` and press `Enter`

```
nsh> hello

Hello, World!!

nsh> 
```

Well done!

I hope everything went well.
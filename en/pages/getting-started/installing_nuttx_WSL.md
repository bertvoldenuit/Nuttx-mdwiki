# Installing NuttX


This tutorial is based on [cmn-lab tutorial](http://cmn-lab.com/first_contact_with_nuttx)
You can also refer to [NuttX Channel](https://www.youtube.com/watch?v=heSkSd-_70g&list=PLd73yQk5Fd8JEsVD-lhwYRQKVu6glfDa8&index=1)

In this tutorial, the Nucleo-F411re is used.

![image](https://user-images.githubusercontent.com/5957713/56863985-87d30600-69bd-11e9-98d2-30ab0db4a137.png)

## Setting a working directory

It is strongly not recommended to modify any file in your ubuntu file system by using a Windows application like 
Windows file explorer or a text editor executed on Windows. Then, how to interract with Nuttx files?? 
Simple! All files will be on your home directory in Windows. From the Ubuntu Bash, Windows files are reachable from the `/mnt` folder.
In order to acces your working directory easily from your Ubuntu Bash, let's create some alias. 

Open Ubuntu terminal and type:

```
alias win_home='cd /mnt/c/<win_home>'
alias nuttxspace='cd /mnt/c/<win_home>/nuttxspace'
```

Of course you can modify the above `c` drive letter by which ever you want. It is the same for `win_home`, it can be replace by the location
you want it to be, like for example `/mnt/c/Users/<YourUserName>/Documents`. Keep in mind you are under Linux so each folder name should be separated by a `/` not a `\`


## Installing the environment tools and NuttX RTOS

Open Ubuntu terminal (if it is not already open)

1- Update and Upgrade your Linux distribution
```
sudo apt-get update && sudo apt-get upgrade
```

2- Install compiler 
```
sudo apt-get install automake bison build-essential flex gcc-arm-none-eabi gperf git libncurses5-dev libtool libusb-dev libusb-1.0.0-dev pkg-config
```

Say `Yes` to any question asked by the system

3- Create `Nuttxspace` directory and go to it
```
win_home
mkdir nuttxspace
```

Now you can access nuttxspace from any where by using the alias previously set:

```
nuttxspace
```

4- Clone NuttX repository
```
nuttxspace
git clone https://bitbucket.org/nuttx/nuttx
git clone https://bitbucket.org/nuttx/apps
git clone https://bitbucket.org/nuttx/tools
```

5- Configure Nuttx
```
cd tools/kconfig-frontends/
./configure
make
sudo make install
sudo ldconfig
nuttxspace
```


6- Compiling Nuttx

First you need to clean
```
nuttxspace
cd nuttx
make distclean
```

Then to configure your target and compile
```
cd tools/
./configure.sh <board_name>:nsh
cd ..
make
```

For example, this is how to clean, set target as Nucleo-F411re and compile Nuttx for it.
```
nuttxspace
cd nuttx
make distclean
cd tools/
./configure.sh nucleo-f4x1re/f411-nsh
cd ..
make
```
As a result the `nuttx.bin` file is created. 


## Flashing target device

First, you need to install `OpenOCD`. You can dowload it by visiting this [website](https://gnutoolchains.com/arm-eabi/openocd/). You can
download a compiled version for windows. Save the zip file to your nuttxspace. You will need to unzip it with [7zip](https://www.7-zip.org/download.html). Rename the folder to `openocd`.

Then, set the `path` to this folder. type `environment variable` in the start menu search bar:

- set the `path='c:\<nuttxspace_directory>\openocd\bin'`.

![image](https://bertvoldenuit.github.io/Nuttx-mdwiki/en/pages/uploads/images/env_var.png)


Let's look at how to flash the target board. With a Nucleo board there are two ways of doing it.

+ Nucleo board is seen by linux as a flash drive. Simply drag and drop `nuttx.bin` at the root of the device. You're done!
+ Use openocd to use the programer/debugger to flash the target board. It will be very useful later to use stlink-v2 as a debugger.

Plug the Nucleo board to the computer with the usb cable.

Let's test the connection:

Open a Windows Termnial: In Start menu search bar type `cmd` and open `Command Prompt`.

```
cd <nuttxspace>
cd nuttx
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


In order to flash the MCU (Microcontroller Unit) memory, use the following command in the `nuttx` folder:
```
cd <nuttxspace>
cd nuttx
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg -c init -c "reset halt" -c "flash write_image erase nuttx.bin 0x08000000"
```

## NuttShell
In order to execute commands on the NuttX Operating System, we must connect to the NuttShell. Let's setup a serial terminal.
You can dowload `TeraTerm` [here](https://ttssh2.osdn.jp/). Install and launch it.

In order to get the `COM#` number, you can check it in the device manager. Once, Tera Term is set with the right com port,
reset the board.

![image](https://bertvoldenuit.github.io/Nuttx-mdwiki/en/pages/uploads/images/device_com_number.png)



Reset the board. Now you should have:
```
NuttShell (NSH)                                                                 
                                                                                
nsh> 
```

Type `help` or `?` to get the command list:
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

## Hello world Application

In NuttX, a program is considered as an Application which can be launched by the nuttshell.
NuttX comes with many application examples. MenuConfig is the place to configure the NuttX OS and select Applications.
It is a graphical text based interface.
Let's try the `Hello World` Application example:

In Ubuntu Bash, type:
```
nuttxspace
cd nuttx
make menuconfig
```

Select `Application Configuration` then `Examples` and `Hello World example`

|       Tree root           | Sublevel 1                  | Sublevel 2                   |Enable |
| ------------------------- |-----------------------------|----------------------------- |:-----:|
| Application Configuration |  Examples                   | Hello World example          | yes   |


`Note: menuconfig has a slightly different look than on a real Linux installation and has some graphical bugs but it works fine`

![image](https://bertvoldenuit.github.io/Nuttx-mdwiki/en/pages/uploads/images/menu_config_WSL.png)

Exit and save

Compile NuttX and flash the Nucleo board:

```
make
```

switch to Windows `Command Prompt` 
```
cd <nuttxspace>
cd nuttx
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg -c init -c "reset halt" -c "flash write_image erase nuttx.bin 0x08000000"
```

launch `Tera Term`
Press reset button on the Nucleo board

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
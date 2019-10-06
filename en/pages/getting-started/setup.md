# Setting up an environment to work with NuttX


## 1. Text editor

### 1.1 Sublime Text

When it comes to choose a text editor, everyone comes with theirs pros and cons. This is not the purpose of this topic. I chose to use Sublimetext because it is a powerful, lightweight, pretty and highly configurable editor. Moreover, it is working perfectly on windows and linux.

- Open software manager and look for `Sublime`
- Click on `install`
- Open `Sublime Text`
- Click on `File->Open Folder` Select `nuttxspace`
- Click on `View->Side Bar->Show Side Bar`, the nuttxspace tree will appear

![image](https://user-images.githubusercontent.com/5957713/56865309-ed7abe80-69cc-11e9-96ca-126bfabd77b4.png)

You're all set with Sublime Text! You can check out for the documentation [here](https://www.sublimetext.com/docs/3/).


## 2. Serial Terminal

A serial terminal allows to communicates with the MCU (Microcontroller Unit) via the serial port. You can easily send commands and receive text information from the MCU (convinient for primitive debugging).

When you first plug your serial device, here a Nucleo board, the terminal can not open the serial port.

you need to re-sync :

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

### 2.1 Minicom

Minicom is a text based serial terminal. Here is how to set it up:

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

### 2.2 PuTTY

PuTTY is both a text based and a graphical serial terminal. Here is how to set it up:

```
sudo apt-get install putty
putty
```


![image](https://user-images.githubusercontent.com/5957713/56864861-d1c0e980-69c7-11e9-91f5-cdc7d1f4062b.png)


## 3. Debugging

### 3.1 Code::Blocks

Code::Blocks can also be used as a text editor but more insterstingly, it is a nice graphical interface to use the debugger.

Here is the tutorial on [Nuttx Channel](https://www.youtube.com/watch?v=KSkv9sOLh4U&list=PLd73yQk5Fd8JEsVD-lhwYRQKVu6glfDa8&index=4)



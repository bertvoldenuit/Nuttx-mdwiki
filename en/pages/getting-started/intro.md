# Getting started with Nuttx

Here is a link to the official Nuttx [`README.txt`](https://bitbucket.org/nuttx/nuttx/src/master/README.txt) file.
It is interesting to have a look, it is very complete about possibilities on how to install Nuttx.


As said in the `README.txt`, Nuttx requires a POSIX development environment such as you would find under
Linux or macOS. 

In this Wiki, two ways of installing Nuttx are chosen to be explained. Installing Nuttx on both system is very similar,
but there are some slight differences which detailed.

## 1. Working on Windows

Actually, Linux Ubuntu is going to be used, but on Windows. Under Windows 10, a recent option allows to run Linux Bash. 
It is not the full graphical environment, just a terminal windows. It is even easier to install than creating a Virtual Machine.
It is probably the best solution for people who want to stay on Windows and work in a Windows environment.
It is safe for you PC, compiling Nuttx is performant. What else!?

In this section, It is explained :

- How to install Ubuntu 18.04 Bash on Windows, like a simple application. 
- How to install Nuttx and OpenOCD which is useful to load binaries (your compilied program) on the MCU and to debug it.
- Setting up an environment to work with Nuttx on Windows.


## 2. Working on Linux

The easiest way of installing Linux on a PC is by installing it on a Virtual Machine. The drawback is that it is much slower than if it is installed natively. However, there is no need of a fast Operating System to work with Nuttx. The most demanding softwares are text editors and source code compiler. The advantage there is no risk of screwing up your computer trying to fit two OS with dual boot, etc... Plus, installation and uninstallation are pretty easy as it’s like removing a program and doesn’t affect anything with boot loaders.

In this section, It is explained :

- How to install a Virtual Machine software, create a Linux VM and install Linux on this VM. 
- How to install Nuttx and OpenOCD which is useful to load binaries (your compilied program) on the MCU and to debug it.
- Setting up an environment to work with Nuttx on Linux.


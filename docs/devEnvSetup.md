# Updated mHST Development Setup
Intent is to keep using open-source (ideally), free (at least), well-supported, edu-friendly, professional-grade and portable software tools and standards for mHST project. This documentation should be for the entire development environment setup.

## Visual Studio Code 
 - For editing code & non-SysML/graphical files that is common to Linux and Windows environments.
 - [x] Install MS [VS Code on Windows](https://code.visualstudio.com/) 
  - Tyring this with its plugins for Python, C/C++, GitHub, and WSL
  - [x] Install [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) (if not already installed); I used all the defaults save credential manager
  - [x] Install [python 3](https://www.python.org/) ; add to PATH, and I also disabled path length check. This should also install [pip](https://pip.pypa.io/en/stable/), though usually I get a warning about a pip update, so I run that as well. I had to restart the machine to refresh env variables.
   - This didn't setup PATH for for me on one install. Per the [Python installion](https://docs.python.org/3/using/windows.html#excursus-setting-environment-variables), open System properties, Advanced system settings and click the Environment Variables button
  - [x] Install Microsoft Visual Studio Code extension for python; includes pylance and jupyter notebook support
   - [x] (not required) I find [numpy](https://numpy.org/install/) a very helpful python package, so I install numpy at this point either through the Windows Terminal or the terminal in VS Code 
   `C:\Users\userid> pip install numpy`
  - [x] Install Microsoft C/C++ extension for Visual Studio Code 
  - [x] Install Microsoft Remote - WSL extension for Visual Studio Code 
  - [x] Install GitHub Pull Requests and Issues extension for Visual Studio Code 
   - Note that with a full clean install of git and VS code, will need to conigure git within VS Code in order to push:
   ```
   git config --global user.email "you@example.com"
   git config --global user.name "Your Name"
   ```

## WSL2 on Windows 11
  - Using Windows Subsystem for Linux (WSL) on a Windows 11 machine. There's not really a reason for this vs a Virtual Box & Ubuntu setup, I just tried this this way. I suppose the VS Code thing works better with this.
 - [x] Install WSL2 on Windows 11 per the [Microsoft WSL Docs](https://docs.microsoft.com/en-us/windows/wsl/) ; need to reboot here. 
  - One time I rebooted and WSL came up with a distribtion and asked me to setup my userid, so I did that. The time before WSL did not come up with a distribtion, in which case: 
  - [x] Install a distribution; I tried installing via Windows Terminal (running as Administrator) with `wsl --install` per above. After reboot though I get:
  ```
  C:\Users\userid> wsl
  Windows Subsystem for Linux has no installed distributions.
  Distributions can be installed by visiting the Microsoft Store:
  https://aka.ms/wslstore
  ```
  So I installed a distribution from the [Microsoft Store](https://aka.ms/wslstore). Once installed, should see this in Windows terminal:
   ```
   C:\Users\userid> wsl -l -v
   NAME      STATE           VERSION
   * Ubuntu    Running         2
   ```
   - [x] Setup user in Ubuntu WSL. 
   - [x] Update WSL (need to exit WSL first, needs to be an administrator level terminal): 
  ```
  C:\Users\userid> wsl --update
  Checking for updates...
  Downloading updates...
  Installing updates...
  This change will take effect on the next full restart of WSL. To force a restart, please run 'wsl --shutdown'.
  Kernel version: 5.10.60.1
  C:\Users\userid> wsl --shutdown
  ```
   - [x] Restart WSL (from Windows terminal just `wsl` assuming just one distribution) and then update and upgrade distribution from within WSL:
   ```
   sudo apt update
   ...
   sudo apt upgrade
   ...
   ```
   - [x] Install Firefox (or other browser) within WSL Ubuntu. This is to get the F' gds tool to work. I also configure my browser preferences here. 
   `sudo apt install firefox`
  - [x] Test VS Code WSL plugin; in VS Code, open a terminal and in the right down arrow, select the WSL Ubuntu. Then Ctrl-o to open a file, click the Linux drive, then navigate to \\wsl.localhost\Ubuntu\home\userid and open the .bashrc file. Scroll down to the aliases in the file and add one, then save the file. Exit any WLS instances open, then reopen and try the alias. 

### USBIPD for WSL
 - This is to enable USB connections with WSL.
 - [x] Install and configure USBIPD for WSL per the [Microsoft WSL How-To](https://docs.microsoft.com/en-us/windows/wsl/connect-usb)
  - As of Dec 20 2021 WSL2 does not natively support USB devices, so use the WSL support in USBIPD
 - [ ] TODO Recently after power cycles of the host the USBIPD server is not restarting ("USBIP Device Host" in Windows Services app) though it looks like it's configured to do so.

### KCONFIG USB to Serial
- This was trying to debug why USB through USBIPD wasn't working in WSL. Not sure if this is/will always be needed, but kept here just in case.
- [x] Edit KCONFIG and rebuild WSL kernel for USB to Serial support
- Trying to debug my Segger J-Link connection, I tried gettting an Arduino Uno connected to [Arduino IDE 2.0](https://www.arduino.cc/en/software#experimental-software) on WSL to see if I could get that to work (it didn't). Even with USBIPD setup correctly, the Arduino IDE would not detect the Uno plugged in
- I tried enabling USB to Serial in KCONFIG, and maybe that was part of the solution. I don't remember that alone fixing the issue.
- I also ended up going through the Arduino IDE 1.8.X install script and make sure to add $USER to dialout and tty and I think that solved it. 
- Along the way I also did this (Not needed?) to get Arduino to work:
```
sudo usermod -a -G tty userid
sudo usermod -a -G dialout userid
sudo chmod 766
```
 - Maybe like [this?](https://devzone.nordicsemi.com/f/nordic-q-a/36986/windows-subsystem-for-linux-wsl---error-there-is-no-debugger-connected-to-the-pc) about the "Microsoft's blog post" part?

### SEGGER J-Link Seteup
 - This is because the microcontroller development board I stared working with ([Adafruit Feather nRF52840 Express](https://docs.zephyrproject.org/2.6.0/boards/arm/adafruit_feather_nrf52840/doc/index.html)) for Zephyr uses a Serial Wire Debug (SWD) interface for programming (not USB), so setup Segger J-Link Mini. I may move to a nominal J-Link, but per above trying to demonstate an edu-friendly setup.
- [x] Install JLink tools per Eclipse CDT [directions](https://eclipse-embed-cdt.github.io/debug/jlink/install/)
- I installed in /opt/ not in ~/opt/ ; note too that the cp command is .rules, not .rule
- Not sure if this is an issue [too](https://github.com/dorssel/usbipd-win/issues/96)
  - seems like it, add this: 
  ```
  sudo service udev restart
  sudo udevadm control --reload
  ```
- [x] Add /opt/SEGGER/JLink_Linux_V760b_x86_64 to PATH
- I struggled getting the J-Link Mini to work to start; when JLinkExe would detect the J-Link, it would jump to a firmware flash screen, which kept timinig out and failing. Then JLinkExe wouldn't see the J-Link over USB. Eventually this just worked; it looks like the firmware update worked, but I'm not sure why. Now JLinkExe starts up fine and finds the J-Link:
```
$ JLinkExe
SEGGER J-Link Commander V7.60b (Compiled Dec 22 2021 14:45:57)
DLL version V7.60b, compiled Dec 22 2021 12:54:38

Connecting to J-Link via USB...O.K.
Firmware: J-Link EDU Mini V1 compiled Dec  7 2021 08:38:51
Hardware version: V1.00
```

## NASA [F'](https://nasa.github.io/fprime/) and [F''](https://fprime-community.github.io/fpp/fpp-users-guide.html)
 - This is the flight software (FSW) application layer component architecture framework tool suite (F') I want to use, with its own editor (F'')
 - [x] Install F' dependencies; I did this by hand, but in the F' Installation [Troubleshooting](https://nasa.github.io/fprime/INSTALL.html#Troubleshooting) section there is a single commandline version of this
  - F' requires a JDK; my WSL installation did not have one, checked with `java -version`
  - [x] Install JDK; I used OpenJDK 17 `sudo apt install openjdk-17-jre-headless`
  - Now: 
  ``` 
  java -version
  openjdk version "17.0.1" 2021-10-19
  OpenJDK Runtime Environment (build 17.0.1+12-Ubuntu-120.04)
  OpenJDK 64-Bit Server VM (build 17.0.1+12-Ubuntu-120.04, mixed mode, sharing)
  ```
  - F' requires Cmake 3.5 or newer; downloaded 3.22.1 .sh file from [cmake.org](https://cmake.org/download/)
  - Installed in /opt/cmake by running the .sh file
  - added to Path `export PATH=$PATH:/opt/cmake/bin`
  - checked install:
  ```
  /opt/cmake/bin$ cmake --version
  cmake version 3.22.1

  CMake suite maintained and supported by Kitware (kitware.com/cmake).
  ```
   - Note cmake calls out export control; should confirm use is permitted per [Policy Information](https://www.kitware.com/policy/#exportcomplianceinformation) 
   - F' requires CLang or GCC; GCC was already on my WSL instance, not sure if I installed it or WSL comes with GCC (or via Zephyr install above):
  ```
  :~$ which gcc
  /usr/bin/gcc
  :~$ gcc --version
  gcc (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0
  Copyright (C) 2019 Free Software Foundation, Inc.
  This is free software; see the source for copying conditions.  There is NO
  warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
  ```
   - F' requires Python 3.6 or newer; python was already on my WSL instance:
  ```
  ~$ python3 --version
  Python 3.8.10
  :~$ which python3
  /usr/bin/python3
  ```
  - [x] Install F' per [Installation Guide](https://nasa.github.io/fprime/INSTALL.html) cloning the F´ core repository 
  - [x] Install F' python-support package
  - [x] Check installation: 
  ```
  cd Ref
  :~/fprime/Ref$ fprime-util generate
  ...
  -- Build files have been written to: /home/.../fprime/Ref/build-fprime-automatic-native
  
  :~/fprime/Ref$ fprime-util build --jobs "$(nproc || printf '%s\n' 1)"
  ...
  [100%] Built target Ref
  ```
  - [x] Test installation:
  `:~/fprime/Ref$ fprime-gds -g html -r ~/fprime/Ref/build-artifacts/`
  - Seems to work fine
 - [ ] TODO Re-run on-host demos on WSL with F''
 - [ ] TODO Run [cross-compile demo](https://github.com/nasa/fprime/blob/master/RPI/README.md) on Raspberry Pi (RPi) 4 single board computer (SBC). Note the existing demo is for the Raspberry Pi 2 model B
 - [ ] TODO Try porting F' to Zephyr and the Adafruit nRF52840 Feather. Probably use the [fprime-sphinx](https://github.com/fprime-community/fprime-sphinx) write up as a guide

## Eclipse IDE
 - The [Eclipse Integrated Development Environment](https://www.eclipse.org/ide/) (IDE) is primarily for UML and SysML modeling. There is a PlantUML plugin for VS Code, but I wasn't sure if that was a single-source of truth modeling tool with diagramming built in or just a diagramming tool. I've used Eclipse for other work before, including the Papyrus UML/SysML graphical editors, with a single model. I may also want to use the CDT tools, but I will try the VS Code ones first.
 - [x] Install [Eclipse 2021-21](https://www.eclipse.org/eclipseide/) 
  - Used the installer approach (per Eclipse recommendation) ; started with the Eclipse IDE for C/C++ Developers package (will add other tools to that) 
  - Installed on Windows in C:\Users\userid\eclipse
  - Under "Help" tab ran "check for updates" before proceeding

### Papyrus UML & SysML
 - These are the UML and SysML modeling & diagramming tools. I have these installed in the Eclipse instance on Windows , not the WSL
 - [x] Install [Papyrus 6.0.0](https://www.eclipse.org/papyrus/download.html#accordion) ; installed via Eclipse IDE, "Help" tab -> "Install New Software..." and then "Work with" the 2021-12 Eclipse site (or whatever version you've installed); search for "modeling" -> installed the entire modeling kit (which includes Papyrus UML 6.0.0). 
  - Restart Eclipse
  - Under "Help" tab ran "check for updates" before proceeding
 - [x] Install [SysML1.6 plugin](https://marketplace.eclipse.org/content/papyrus-sysml-16) (2.2?). Used the drag & drop into Eclipse IDE window installer.
 - Restart Eclipse

### GitHub Token (not required)
 - I use GitHub for most of this work, so setup Eclipse to use my github
 - [x] Create Personal Access token per [EGit directions](https://wiki.eclipse.org/EGit/GitHub/User_Guide)

## Zephyr
 - This is because the professional end goal will likely want some opertating system (OS) on the embedded system, and potentially some version of real-time (hard or soft). Rather than working through how to create a Linux distribution with the real-time patch, start with the Linux Foundation work on their real-time OS (RTOS), Zephyr. 
 - [x] Install [Zephyr](https://docs.zephyrproject.org/2.6.0/getting_started/index.html) (2.6)
 - The docs "don’t recommend using WSL when getting started" but I didn't really understand why so I tried WSL anyway
- [ ] [Blinky](https://docs.zephyrproject.org/2.6.0/getting_started/index.html#build-the-blinky-sample)
- I have done this, but want to try again
- [ ] TODO [PWM Blinky](https://docs.zephyrproject.org/2.6.0/samples/basic/blinky_pwm/README.html)
- Tried this and got the device tree error, so want to work that
- [ ] TODO Something with console out?





# Updated mHST Development Setup
Intent is to keep using open-source, supported, edu-friendly, professional-grade and portable software tools and standards for mHST project. 

## Visual Studio Code 
 - [x] Install MS [VS Code on Windows](https://code.visualstudio.com/) 
  - Tyring this with its plugins for GitHub and WSL2

## WSL2 on Windows 11
 - [x] Install WSL2 on Windows 11 per the [Microsoft WSL Docs](https://docs.microsoft.com/en-us/windows/wsl/)
  - There's not really a reason for this vs a Virtual Box & Ubuntu setup, I just tried this this way.

### USBIPD for WSL
 - [x] Install and configure USBIPD for WSL per the [Microsoft WSL How-To](https://docs.microsoft.com/en-us/windows/wsl/connect-usb)
  - As of Dec 20 2021 WSL2 does not natively support USB devices, so use the WSL support in USBIPD
 - [ ] TODO Recently after power cycles of the host the USBIPD server is not restarting ("USBIP Device Host" in Windows Services app) though it looks like it's configured to do so.

### KCONFIG USB to Serial
- [x] Edit KCONFIG and rebuild WSL kernel for USB to Serial support
- Trying to debug my Segger J-Link connection, I tried gettting an Arduino Uno connected to [Arduino IDE 2.0](https://www.arduino.cc/en/software#experimental-software) on WSL to see if I could get that to work (it didn't). Even with USBIPD setup correctly, the Arduino IDE would not detect the Uno plugged in
- I tried enabling USB to Serial in KCONFIG, and maybe that was part of the solution. I don't remember that alone fixing the issue.
- I also ended up going through the Arduino IDE 1.8.X install script and make sure to add $USER to dialout and tty and I think that solved it. 
- Along the way I also did this (Not needed?) to get Arduino to work:
```
sudo usermod -a -G tty djwait
sudo usermod -a -G dialout djwait
sudo chmod 766
```
 - Maybe like [this?](https://devzone.nordicsemi.com/f/nordic-q-a/36986/windows-subsystem-for-linux-wsl---error-there-is-no-debugger-connected-to-the-pc) about the "Microsoft's blog post" part?

### SEGGER J-Link Seteup
- [x] Install JLink tools per Eclipse CDT [directions](https://eclipse-embed-cdt.github.io/debug/jlink/install/)
- Turns out the board I wanted to use ([Adafruit Feather nRF52840 Express](https://docs.zephyrproject.org/2.6.0/boards/arm/adafruit_feather_nrf52840/doc/index.html)) needs to be flashed over SWD (?) not USB, so setup Segger J-Link Mini
- I installed in /opt/ not in ~/opt/ ; note too that the cp command is .rules, not .rule
- Not sure if this is an issue [too](https://github.com/dorssel/usbipd-win/issues/96)
  - seems like it, add this: 
  ```
  sudo service udev restart
  sudo udevadm control --reload
  ```
- [x] Add /opt/SEGGER/JLink_Linux_V760b_x86_64 to PATH
- I struggled getting the J-Link Mini to work to start; when JLinkExe would detect the J-Link, it would jump to a firmware flash screen, which kept timinig out. Then JLink Exe wouldn't see the J-Link over USB. Eventually this just worked; I'm not sure why. Now JLinkExe starts up fine and finds the J-Link:
```
$ JLinkExe
SEGGER J-Link Commander V7.60b (Compiled Dec 22 2021 14:45:57)
DLL version V7.60b, compiled Dec 22 2021 12:54:38

Connecting to J-Link via USB...O.K.
Firmware: J-Link EDU Mini V1 compiled Dec  7 2021 08:38:51
Hardware version: V1.00
```

## Zephyr
 - [x] Install [Zephyr](https://docs.zephyrproject.org/2.6.0/getting_started/index.html) (2.6)
 - The docs "don’t recommend using WSL when getting started" but I didn't really understand why so I tried WSL anyway
- [ ] [Blinky](https://docs.zephyrproject.org/2.6.0/getting_started/index.html#build-the-blinky-sample)
- I have done this, but want to try again
- [ ] TODO [PWM Blinky](https://docs.zephyrproject.org/2.6.0/samples/basic/blinky_pwm/README.html)
- Tried this and got the device tree error, so want to work that
- [ ] TODO Something with console out?

## Eclipse IDE
 - [ ] TODO Install [Eclipse 2021-21](https://www.eclipse.org/eclipseide/) 
 - I want to use Eclipse for its CDT tools, but I may end up using VS Code for that. 

### Papyrus UML & SysML
 - [ ] TODO Install [Papyrus 6.0.0](https://www.eclipse.org/papyrus/download.html#accordion)
 - [ ] TODO Install [SysML1.6 plugin](https://marketplace.eclipse.org/content/papyrus-sysml-16) (2.2?)

## NASA F''
 - [ ] TODO Install [F''](https://github.com/nasa/fprime/releases/tag/v3.0.0) 
 - [ ] TODO Re-run on host demos on WSL with F''
 - [ ] TODO figure out port to 

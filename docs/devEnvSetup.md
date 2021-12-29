# Updated Setup

## WSL2 on Windows 11

Per the [Microsoft WSL Docs](https://docs.microsoft.com/en-us/windows/wsl/)

### USB with usbipd

Per the [Microsoft WSL How-To](https://docs.microsoft.com/en-us/windows/wsl/connect-usb)

### KCONFIG USB to Serial

(Not needed?) to get Arduino 2.0 to work needed to:
`sudo usermod -a -G tty djwait
sudo usermod -a -G dialout djwait
sudo chmod 766`
Maybe like [this?](https://devzone.nordicsemi.com/f/nordic-q-a/36986/windows-subsystem-for-linux-wsl---error-there-is-no-debugger-connected-to-the-pc)

## Eclipse

### Papyrus UML & SysML

## Zephyr

### SEGGER
*Install JLink tools per Eclipse CDT [directions](https://eclipse-embed-cdt.github.io/debug/jlink/install/)
**I installed in /opt/ not in ~/opt/ ; note too that the cp command is .rules, not .rule
**not sure is this is an issue [too](https://github.com/dorssel/usbipd-win/issues/96)
*Add JLinkExe to $PATH

## NASA F''



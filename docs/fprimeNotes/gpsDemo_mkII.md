used the gpsDemo write up, but needed to update per latest 64bit rpi os:
```
$ uname -a
Linux raspberrypi 6.6.62+rpt-rpi-v8 #1 SMP PREEMPT Debian 1:6.6.62-1+rpt1 (2024-11-25) aarch64 GNU/Linux
$ lsb_release -a
No LSB modules are available.
Distributor ID: Debian
Description:    Debian GNU/Linux 12 (bookworm)
Release:        12
Codename:       bookworm
```
Using UART5 per gpsDemo, added to config.txt:
```
$ more /boot/firmware/config.txt 
# For more options and information see
# http://rptl.io/configtxt
# Some settings may impact device functionality. See link above for details

# Uncomment some or all of these to enable the optional hardware interfaces
...
[all]
enable_uart=1
dtoverlay=uart5
```
Used pinctrl to check:
```
$ pinctrl 
 0: ip    pu | hi // ID_SDA/GPIO0 = input
 1: ip    pu | hi // ID_SCL/GPIO1 = input
 2: a0    pu | hi // GPIO2 = SDA1
 3: a0    pu | hi // GPIO3 = SCL1
 4: ip    pu | hi // GPIO4 = input
 5: ip    pu | hi // GPIO5 = input
 6: ip    pu | hi // GPIO6 = input
 7: op -- pu | hi // GPIO7 = output
 8: op -- pu | hi // GPIO8 = output
 9: a0    pd | lo // GPIO9 = SPI0_MISO
10: a0    pd | lo // GPIO10 = SPI0_MOSI
11: a0    pd | lo // GPIO11 = SPI0_SCLK
12: a4    pn | hi // GPIO12 = TXD5
13: a4    pu | hi // GPIO13 = RXD5
...
```

```

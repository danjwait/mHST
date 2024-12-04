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

Also checked in /dev:

```
/dev $ ls
autofs           gpiochip0  loop1         net    ram6       tty11  tty28  tty44  tty60      vcs       vcsu1        video18
block            gpiochip1  loop2         null   ram7       tty12  tty29  tty45  tty61      vcs1      vcsu2        video19
btrfs-control    gpiochip4  loop3         port   ram8       tty13  tty3   tty46  tty62      vcs2      vcsu3        video20
bus              gpiomem    loop4         ppp    ram9       tty14  tty30  tty47  tty63      vcs3      vcsu4        video21
cachefiles       hidraw0    loop5         ptmx   random     tty15  tty31  tty48  tty7       vcs4      vcsu5        video22
cec0             hidraw1    loop6         pts    rfkill     tty16  tty32  tty49  tty8       vcs5      vcsu6        video23
cec1             hidraw2    loop7         ram0   serial0    tty17  tty33  tty5   tty9       vcs6      vcsu7        video31
char             hidraw3    loop-control  ram1   shm        tty18  tty34  tty50  ttyAMA5
...
```
So then after confirming GPS LED is flashing, checked the UART with `cat /dev/ttyAMA5` and see GPS NEMA strings.


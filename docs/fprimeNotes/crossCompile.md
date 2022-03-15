
# Cross Compile Testing; Porting Math Demo to RPi
Started with the "Cross Compiling for the Raspberry PI" section of [F Fprime GPS demo instructions](https://nasa.github.io/fprime/Tutorials/GpsTutorial/Tutorial.html) but intent is to just cross compile the Math Component to the RPi first.

Tested the Math Component (per above) on WSL2 linux "native" first, then edited to cross compile.

## Setup RPi4
Usual sudo apt-get update & upgrade
Followed RPi directions [here](https://www.raspberrypi.com/documentation/computers/remote-access.html) to setup SSH from host to RPi
Confirmed SSH ability from host to RPi4
Checked RPi version:
```
pi@raspberrypi:~ $ uname -m
armv7l
```
so using 32 bit OS. Version info:
```
pi@raspberrypi:~ $ lsb_release -a
No LSB modules are available.
Distributor ID: Raspbian
Description:    Raspbian GNU/Linux 10 (buster)
Release:        10
Codename:       buster
pi@raspberrypi:~ $ hostnamectl
   Static hostname: raspberrypi
         Icon name: computer
        Machine ID: b2ecfd255d374c23bf12149870d6520a
           Boot ID: 28be2f14f1974e218ebf32ab7a25bf06
  Operating System: Raspbian GNU/Linux 10 (buster)
            Kernel: Linux 5.10.63-v7l+
      Architecture: arm
pi@raspberrypi:~ $ cat /etc/rpi-issue 
Raspberry Pi reference 2021-05-07
Generated using pi-gen, https://github.com/RPi-Distro/pi-gen, dcfd74d7d1fa293065ac6d565711e9ff891fe2b8, stage4
```

## Toolchains
Using toolchains per [raspberrypi / tools](https://github.com/raspberrypi/tools):
```
sudo apt-get install gcc-arm-linux-gnueabihf
sudo apt-get install g++-arm-linux-gnueabihf
sudo apt-get install gcc-aarch64-linux-gnu
sudo apt-get install g++-aarch64-linux-gnu
```
Installed the former pair in case using a 32-bit OS (as in this case); installed the latter pair since RPi4 is 64 bit SoC and may change OS at somepoint. Per above, presently running the 32 bit OS (armv7l). 

My installations ended up at:
- /bin/aarch64-linux-gnu-gcc
- /bin/arm-linux-gnueabihf-gcc

## Cross-compile testing:
Tried the present default command: `fprime-util generate raspberrypi` in Ref/ and got error:
```
~/02_Projects/fprime/Ref$ fprime-util generate raspberrypi
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/Ref/settings.ini
[INFO] Generating build directory at: /home/djwait/02_Projects/fprime/Ref/build-fprime-automatic-raspberrypi
[INFO] Using toolchain file /home/djwait/02_Projects/fprime/cmake/toolchain/raspberrypi.cmake for platform raspberrypi
CMake Error at /home/djwait/02_Projects/fprime/cmake/toolchain/raspberrypi.cmake:26 (message):
  RPI toolchain not found at
  /opt/rpi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf.  Install it, set
  RPI_TOOLCHAIN_DIR environment variable, or choose another toolchain
Call Stack (most recent call first):
  /usr/share/cmake-3.16/Modules/CMakeDetermineSystem.cmake:93 (include)
  CMakeLists.txt:23 (project)


CMake Error: CMake was unable to find a build program corresponding to "Unix Makefiles".  CMAKE_MAKE_PROGRAM is not set.  You probably need to select a different build tool.
-- Configuring incomplete, errors occurred!
CMake Error: CMAKE_C_COMPILER not set, after EnableLanguage
CMake Error: CMAKE_CXX_COMPILER not set, after EnableLanguage
[ERROR] CMake erred with return code 1. Partial build cache remains. Run purge to clean-up.
```
Edited cmake/toolchain/raspberrypi.cmake for the RPI_TOOLCHAIN, CMAKE_C_COMPILER, and CMAKE_CXX_COMPILER setting for the toolchain installed above:
```
####
# Raspberry PI Toolchain
#
# A toolchain file for the raspberrypi. This toolchain can be used to build against the raspberry pi embedded Linux
# target. In order to use this toolchain, the raspberry pi build tools should be cloned onto a Linux build host. These
# tools are available at: [https://github.com/raspberrypi/tools](https://github.com/raspberrypi/tools)
#
# Typically these tools are cloned to `/opt/rpi/`.  If they are cloned elsewhere, the user must set the environment
# variable `RPI_TOOLCHAIN_DIR` to the full path to the `arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf` directory before
# running CMake or `fprime-util generate`.
#
# e.g. should the user install the tools in ``/home/user1` then the environment variable might be set using
# `export RPI_TOOLCHAIN_DIR=/home/user/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/`
####
# Set system name
set(CMAKE_SYSTEM_NAME "Linux")
set(CMAKE_SYSTEM_PROCESSOR "arm")

# Location of pi toolchain
set(RPI_TOOLCHAIN "$ENV{RPI_TOOLCHAIN_DIR}")
if ("${RPI_TOOLCHAIN}" STREQUAL "")
    #set(RPI_TOOLCHAIN "/opt/rpi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf")
    set(RPI_TOOLCHAIN "/bin")
endif()
# Check toolchain directory exists
IF(NOT EXISTS "${RPI_TOOLCHAIN}")
    message(FATAL_ERROR "RPI toolchain not found at ${RPI_TOOLCHAIN}. Install it, set RPI_TOOLCHAIN_DIR environment variable, or choose another toolchain")
endif()
message(STATUS "Using RPI toolchain at: ${RPI_TOOLCHAIN}")
# specify the cross compiler
# for 32 bit OS
set(CMAKE_C_COMPILER "${RPI_TOOLCHAIN}/arm-linux-gnueabihf-gcc")
set(CMAKE_CXX_COMPILER "${RPI_TOOLCHAIN}/arm-linux-gnueabihf-g++")
# for 64 bit OS
#set(CMAKE_C_COMPILER "${RPI_TOOLCHAIN}/aarch64-linux-gnu-gcc")
#set(CMAKE_CXX_COMPILER "${RPI_TOOLCHAIN}/aarch64-linux-gnu-g++")

# where is the target environment
set(CMAKE_FIND_ROOT_PATH  "${RPI_TOOLCHAIN}")

# search for programs in the build host directories
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
# for libraries and headers in the target directories
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
```
Then ran `~/02_Projects/fprime/Ref$ fprime-util generate raspberrypi` and see:
```
...
-- Adding Library: Ref_SignalGen
-- Adding Library: Ref_MathTypes
-- Adding Library: Ref_MathPorts
-- Adding Library: Ref_MathSender
-- Adding Library: Ref_MathReceiver
-- Adding Library: Ref_Top
-- Adding Deployment: Ref
-- Configuring done
-- Generating done
-- Build files have been written to: /home/djwait/02_Projects/fprime/Ref/build-fprime-automatic-raspberrypi
```
Then build:
```
/02_Projects/fprime/Ref$ fprime-util build raspberrypi
...
-- Installing: /home/djwait/02_Projects/fprime/Ref/build-artifacts/raspberrypi/lib/static/libSvc_StaticMemory.a
-- Installing: /home/djwait/02_Projects/fprime/Ref/build-artifacts/raspberrypi/lib/static/libSvc_SystemResources.a
-- Installing: /home/djwait/02_Projects/fprime/Ref/build-artifacts/raspberrypi/lib/static/libSvc_TlmChan.a
-- Installing: /home/djwait/02_Projects/fprime/Ref/build-artifacts/raspberrypi/lib/static/libRef_Top.a
-- Installing: /home/djwait/02_Projects/fprime/Ref/build-artifacts/raspberrypi/dict/RefTopologyAppDictionary.xml
[100%] Built target Ref
```
Find bin file:
```
/02_Projects/fprime/Ref/build-artifacts/raspberrypi/bin$ ls -lrt
~/02_Projects/fprime/Ref/build-artifacts/raspberrypi/bin$ ls -lrt
total 2476
drwxr-xr-x 3 djwait djwait    4096 Feb 19 18:03 logs
-rwxr-xr-x 1 djwait djwait 1107976 Feb 21 09:23 Ref
```
scp the Ref bin over to RPi:
```
~/02_Projects/fprime/Ref/build-artifacts/raspberrypi/bin$ scp -r Ref pi@<pi IP>:/home/pi
```
see Ref bin file on RPi:
```
pi@raspberrypi:~ $ pwd
/home/pi
pi@raspberrypi:~ $ ls -lrt
total 1120
drwxr-xr-x 2 pi pi    4096 May  7  2021 Bookshelf
drwxr-xr-x 2 pi pi    4096 May  7  2021 Desktop
drwxr-xr-x 2 pi pi    4096 May  7  2021 Videos
drwxr-xr-x 2 pi pi    4096 May  7  2021 Templates
drwxr-xr-x 2 pi pi    4096 May  7  2021 Public
drwxr-xr-x 2 pi pi    4096 May  7  2021 Pictures
drwxr-xr-x 2 pi pi    4096 May  7  2021 Music
drwxr-xr-x 2 pi pi    4096 May  7  2021 Downloads
drwxr-xr-x 2 pi pi    4096 May  7  2021 Documents
-rwxr-xr-x 1 pi pi 1107976 Feb 21 09:25 Ref
```
## Running the applicaiton on the target and interacting with it through the Fprime GDS on host
Start Ref bin on RPi (have to use sudo for the setup I am using) with the <host IP> set to my first guess at the host IP:
```
pi@raspberrypi:~ $ sudo ./Ref -a <host IP> -p 50000
Hit Ctrl-C to quit
EVENT: (1280) (2:1645473299,690936) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2100 registered to port 0 slot 0
EVENT: (1280) (2:1645473299,691085) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2101 registered to port 0 slot 1
EVENT: (1280) (2:1645473299,691206) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2102 registered to port 0 slot 2
EVENT: (1280) (2:1645473299,691313) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2200 registered to port 1 slot 3
EVENT: (1280) (2:1645473299,691416) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2201 registered to port 1 slot 4
EVENT: (1280) (2:1645473299,691520) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2202 registered to port 1 slot 5
EVENT: (1280) (2:1645473299,691623) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2300 registered to port 2 slot 6
EVENT: (1280) (2:1645473299,691725) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2301 registered to port 2 slot 7
EVENT: (1280) (2:1645473299,691827) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2302 registered to port 2 slot 8
EVENT: (1280) (2:1645473299,691932) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2400 registered to port 3 slot 9
EVENT: (1280) (2:1645473299,692034) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2401 registered to port 3 slot 10
EVENT: (1280) (2:1645473299,692136) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2402 registered to port 3 slot 11
EVENT: (1280) (2:1645473299,692238) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2500 registered to port 4 slot 12
EVENT: (1280) (2:1645473299,692343) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2501 registered to port 4 slot 13
EVENT: (1280) (2:1645473299,692485) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2502 registered to port 4 slot 14
EVENT: (1280) (2:1645473299,692594) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x500 registered to port 5 slot 15
EVENT: (1280) (2:1645473299,692697) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x501 registered to port 5 slot 16
EVENT: (1280) (2:1645473299,692798) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x502 registered to port 5 slot 17
EVENT: (1280) (2:1645473299,692900) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x503 registered to port 5 slot 18
EVENT: (1280) (2:1645473299,693003) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x600 registered to port 6 slot 19
EVENT: (1280) (2:1645473299,693105) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x601 registered to port 6 slot 20
EVENT: (1280) (2:1645473299,693207) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x602 registered to port 6 slot 21
EVENT: (1280) (2:1645473299,693310) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x603 registered to port 6 slot 22
EVENT: (1280) (2:1645473299,693411) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x604 registered to port 6 slot 23
EVENT: (1280) (2:1645473299,693513) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x605 registered to port 6 slot 24
EVENT: (1280) (2:1645473299,693615) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x606 registered to port 6 slot 25
EVENT: (1280) (2:1645473299,693717) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x607 registered to port 6 slot 26
EVENT: (1280) (2:1645473299,693819) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0xb00 registered to port 7 slot 27
EVENT: (1280) (2:1645473299,693921) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0xb02 registered to port 7 slot 28
EVENT: (1280) (2:1645473299,694024) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0xb03 registered to port 7 slot 29
EVENT: (1280) (2:1645473299,694126) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x700 registered to port 8 slot 30
EVENT: (1280) (2:1645473299,694228) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x701 registered to port 8 slot 31
EVENT: (1280) (2:1645473299,694330) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x702 registered to port 8 slot 32
EVENT: (1280) (2:1645473299,694433) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x800 registered to port 9 slot 33
EVENT: (1280) (2:1645473299,694536) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x801 registered to port 9 slot 34
EVENT: (1280) (2:1645473299,694637) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x802 registered to port 9 slot 35
EVENT: (1280) (2:1645473299,694738) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x803 registered to port 9 slot 36
EVENT: (1280) (2:1645473299,694840) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x804 registered to port 9 slot 37
EVENT: (1280) (2:1645473299,694945) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x805 registered to port 9 slot 38
EVENT: (1280) (2:1645473299,695049) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2000 registered to port 10 slot 39
EVENT: (1280) (2:1645473299,695152) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2001 registered to port 10 slot 40
EVENT: (1280) (2:1645473299,695255) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2002 registered to port 10 slot 41
EVENT: (1280) (2:1645473299,695359) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2700 registered to port 11 slot 42
EVENT: (1280) (2:1645473299,695462) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x270a registered to port 11 slot 43
EVENT: (1280) (2:1645473299,695563) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x270b registered to port 11 slot 44
EVENT: (1280) (2:1645473299,695666) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0xe00 registered to port 12 slot 45
EVENT: (1280) (2:1645473299,695770) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0xa00 registered to port 13 slot 46
EVENT: (1280) (2:1645473299,695873) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0xd00 registered to port 14 slot 47
EVENT: (1280) (2:1645473299,695977) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x4700 registered to port 15 slot 48
EVENT: (1280) (2:1645473299,696080) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x4701 registered to port 15 slot 49
EVENT: (1280) (2:1645473299,696184) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x4702 registered to port 15 slot 50
EVENT: (1280) (2:1645473299,696288) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x4703 registered to port 15 slot 51
EVENT: (1280) (2:1645473299,696391) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2600 registered to port 16 slot 52
EVENT: (1280) (2:1645473299,696495) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2601 registered to port 16 slot 53
EVENT: (1280) (2:1645473299,696597) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2602 registered to port 16 slot 54
EVENT: (1280) (2:1645473299,696700) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2603 registered to port 16 slot 55
EVENT: (1280) (2:1645473299,696802) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x260a registered to port 16 slot 56
EVENT: (1280) (2:1645473299,696905) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x260b registered to port 16 slot 57
EVENT: (1280) (2:1645473299,697010) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x260c registered to port 16 slot 58
EVENT: (1280) (2:1645473299,697113) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x260d registered to port 16 slot 59
EVENT: (1280) (2:1645473299,697217) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x4b00 registered to port 17 slot 60
EVENT: (1280) (2:1645473299,697320) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x4b01 registered to port 17 slot 61
EVENT: (3334) (2:1645473299,697451) WARNING_HI: (prmDb) PrmFileReadError : Parameter file read failed in stage OPEN with record 0 and error 1
EVENT: (3328) (2:1645473299,697583) WARNING_LO: (prmDb) PrmIdNotFound : Parameter ID 0x2700 not found
EVENT: (3328) (2:1645473299,697699) WARNING_LO: (prmDb) PrmIdNotFound : Parameter ID 0x4700 not found
EVENT: (3328) (2:1645473299,697810) WARNING_LO: (prmDb) PrmIdNotFound : Parameter ID 0x4701 not found
EVENT: (3328) (2:1645473299,697922) WARNING_LO: (prmDb) PrmIdNotFound : Parameter ID 0x2600 not found
EVENT: (3328) (2:1645473299,698034) WARNING_LO: (prmDb) PrmIdNotFound : Parameter ID 0x2601 not found
[WARNING] High task priority of 140 being clamped to 99
[WARNING] High task priority of 101 being clamped to 99
[WARNING] High task priority of 100 being clamped to 99
[WARNING] High task priority of 100 being clamped to 99
[WARNING] High task priority of 100 being clamped to 99
[ERROR] Failed to send framed data: 1
[ERROR] Failed to send framed data: 1
[ERROR] Failed to send framed data: 1
[ERROR] Failed to send framed data: 1
[ERROR] Failed to send framed data: 1
[ERROR] Failed to send framed data: 1
[WARNING] High task priority of 100 being clamped to 99
[WARNING] High task priority of 100 being clamped to 99
[WARNING] High task priority of 100 being clamped to 99
[WARNING] High task priority of 100 being clamped to 99
[WARNING] High task priority of 120 being clamped to 99
[WARNING] High task priority of 119 being clamped to 99
EVENT: (512) (2:1645473299,701746) DIAGNOSTIC: (rateGroup1Comp) RateGroupStarted : Rate group started.
[WARNING] High task priority of 118 being clamped to 99
EVENT: (768) (2:1645473299,702117) DIAGNOSTIC: (rateGroup2Comp) RateGroupStarted : Rate group started.
EVENT: (1024) (2:1645473299,702338) DIAGNOSTIC: (rateGroup3Comp) RateGroupStarted : Rate group started.
[ERROR] Failed to send framed data: 1
...
```
I wondered if the [ERROR} was because I had not started fprime-gds on host yet, so started that next:
```
~/02_Projects/fprime/Ref$ fprime-gds -g html -n --dictionary build-artifacts/raspberrypi/dict/RefTopologyAppDictionary.xml
[INFO] Ensuring TCP Server is stable for at least 5 seconds
[INFO] Running Application: /usr/bin/python3
[INFO] Log File: /home/djwait/02_Projects/fprime/Ref/logs/2022_02_21-12_14_47/ThreadedTCP.log
[INFO] Ensuring comm[ip] Application is stable for at least 1 seconds
[INFO] Running Application: /usr/bin/python3
[INFO] Ensuring HTML GUI is stable for at least 2 seconds
[INFO] Running Application: /usr/bin/python3
 * Serving Flask app "fprime_gds.flask.app"
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
[INFO] F prime is now running. CTRL-C to shutdown all components.
```
GDS comes up but with red X for no data flow. Stopped both the app on the RPi and the GDD on the hose, and checking the GDS log see no connection:
```
~/02_Projects/fprime/Ref/logs$ more 2022_02_21-12_14_47/ThreadedTCP.log
TCP Socket Server listening on host addr 0.0.0.0, port 50050
Registered client FSW_
Registration complete waiting for message.
Registered client GUI_
Registration complete waiting for message.
Ctrl-C received, server shutting down.
read data from socket is empty!
Header information is empty, client FSW_ exiting.
Closed FSW_ connection.
read data from socket is empty!
Header information is empty, client GUI_ exiting.
Closed GUI_ connection.
```
Not sure about the ports; checking [this write up](https://docs.microsoft.com/en-us/windows/wsl/networking) for WSL2, tried setting up ports to WSL:

### Allow the port on the host
Need to open port on the host; in admin terminal on Windows OS machine, enable the port:
```
netsh advfirewall firewall add rule name= "open port 50000" dir=in action=allow protocol=TCP localport=50000
ok.
```
### Connect the host OS to the WSL2 instance
Need to connect the ports between WSL2 IP to the host Windows IP, per the WSL2 networking notes linked above:
```
netsh interface portproxy add v4tov4 listenport=50000 listenaddress=0.0.0.0 connectport=50000 connectaddress=<WSL2 IP Address>
```
Where the `<WSL2 IP Address>` is found on the WSL2 OS via the command `~$ ip addr | grep eth0` per the WSL2 networking notes linked above.
### Start the GDS
I started the GDS w/o specifying the IP address:
``` /02_Projects/fprime/Ref$ fprime-gds -g html -n --dictionary build-artifacts/raspberrypi/dict/RefTopologyAppDictionary.xml```

### Start the application on the target
Start the app on the target with the Windows host IP address (found with `ipconfig` in a Windows terminal) and the port (not the WSL2 IP address)
```
pi@raspberrypi:~ $ sudo ./Ref -a <host IP Address> -p 50000
Hit Ctrl-C to quit
EVENT: (1280) (2:1645592921,897418) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2100 registered to port 0 slot 0
EVENT: (1280) (2:1645592921,897491) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2101 registered to port 0 slot 1
EVENT: (1280) (2:1645592921,897535) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2102 registered to port 0 slot 2
EVENT: (1280) (2:1645592921,897577) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2200 registered to port 1 slot 3
EVENT: (1280) (2:1645592921,897618) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2201 registered to port 1 slot 4
EVENT: (1280) (2:1645592921,897658) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2202 registered to port 1 slot 5
EVENT: (1280) (2:1645592921,897697) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2300 registered to port 2 slot 6
EVENT: (1280) (2:1645592921,897735) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2301 registered to port 2 slot 7
EVENT: (1280) (2:1645592921,897774) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2302 registered to port 2 slot 8
EVENT: (1280) (2:1645592921,897813) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2400 registered to port 3 slot 9
EVENT: (1280) (2:1645592921,897852) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2401 registered to port 3 slot 10
EVENT: (1280) (2:1645592921,897891) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2402 registered to port 3 slot 11
EVENT: (1280) (2:1645592921,897930) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2500 registered to port 4 slot 12
EVENT: (1280) (2:1645592921,897970) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2501 registered to port 4 slot 13
EVENT: (1280) (2:1645592921,898009) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2502 registered to port 4 slot 14
EVENT: (1280) (2:1645592921,898049) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x500 registered to port 5 slot 15
EVENT: (1280) (2:1645592921,898088) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x501 registered to port 5 slot 16
EVENT: (1280) (2:1645592921,898127) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x502 registered to port 5 slot 17
EVENT: (1280) (2:1645592921,898166) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x503 registered to port 5 slot 18
EVENT: (1280) (2:1645592921,898204) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x600 registered to port 6 slot 19
EVENT: (1280) (2:1645592921,898243) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x601 registered to port 6 slot 20
EVENT: (1280) (2:1645592921,898282) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x602 registered to port 6 slot 21
EVENT: (1280) (2:1645592921,898321) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x603 registered to port 6 slot 22
EVENT: (1280) (2:1645592921,898359) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x604 registered to port 6 slot 23
EVENT: (1280) (2:1645592921,898398) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x605 registered to port 6 slot 24
EVENT: (1280) (2:1645592921,898437) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x606 registered to port 6 slot 25
EVENT: (1280) (2:1645592921,898476) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x607 registered to port 6 slot 26
EVENT: (1280) (2:1645592921,898516) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0xb00 registered to port 7 slot 27
EVENT: (1280) (2:1645592921,898555) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0xb02 registered to port 7 slot 28
EVENT: (1280) (2:1645592921,898594) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0xb03 registered to port 7 slot 29
EVENT: (1280) (2:1645592921,898633) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x700 registered to port 8 slot 30
EVENT: (1280) (2:1645592921,898672) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x701 registered to port 8 slot 31
EVENT: (1280) (2:1645592921,898711) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x702 registered to port 8 slot 32
EVENT: (1280) (2:1645592921,898751) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x800 registered to port 9 slot 33
EVENT: (1280) (2:1645592921,898790) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x801 registered to port 9 slot 34
EVENT: (1280) (2:1645592921,898829) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x802 registered to port 9 slot 35
EVENT: (1280) (2:1645592921,898868) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x803 registered to port 9 slot 36
EVENT: (1280) (2:1645592921,898905) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x804 registered to port 9 slot 37
EVENT: (1280) (2:1645592921,898948) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x805 registered to port 9 slot 38
EVENT: (1280) (2:1645592921,898988) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2000 registered to port 10 slot 39
EVENT: (1280) (2:1645592921,899029) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2001 registered to port 10 slot 40
EVENT: (1280) (2:1645592921,899069) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2002 registered to port 10 slot 41
EVENT: (1280) (2:1645592921,899109) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2700 registered to port 11 slot 42
EVENT: (1280) (2:1645592921,899149) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x270a registered to port 11 slot 43
EVENT: (1280) (2:1645592921,899189) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x270b registered to port 11 slot 44
EVENT: (1280) (2:1645592921,899230) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0xe00 registered to port 12 slot 45
EVENT: (1280) (2:1645592921,899270) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0xa00 registered to port 13 slot 46
EVENT: (1280) (2:1645592921,899311) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0xd00 registered to port 14 slot 47
EVENT: (1280) (2:1645592921,899351) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x4700 registered to port 15 slot 48
EVENT: (1280) (2:1645592921,899391) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x4701 registered to port 15 slot 49
EVENT: (1280) (2:1645592921,899431) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x4702 registered to port 15 slot 50
EVENT: (1280) (2:1645592921,899471) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x4703 registered to port 15 slot 51
EVENT: (1280) (2:1645592921,899511) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2600 registered to port 16 slot 52
EVENT: (1280) (2:1645592921,899551) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2601 registered to port 16 slot 53
EVENT: (1280) (2:1645592921,899591) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2602 registered to port 16 slot 54
EVENT: (1280) (2:1645592921,899631) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2603 registered to port 16 slot 55
EVENT: (1280) (2:1645592921,899671) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x260a registered to port 16 slot 56
EVENT: (1280) (2:1645592921,899711) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x260b registered to port 16 slot 57
EVENT: (1280) (2:1645592921,899750) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x260c registered to port 16 slot 58
EVENT: (1280) (2:1645592921,899790) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x260d registered to port 16 slot 59
EVENT: (1280) (2:1645592921,899831) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x4b00 registered to port 17 slot 60
EVENT: (1280) (2:1645592921,899872) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x4b01 registered to port 17 slot 61
EVENT: (3334) (2:1645592921,899934) WARNING_HI: (prmDb) PrmFileReadError : Parameter file read failed in stage OPEN with record 0 and error 1
EVENT: (3328) (2:1645592921,899992) WARNING_LO: (prmDb) PrmIdNotFound : Parameter ID 0x2700 not found
EVENT: (3328) (2:1645592921,900039) WARNING_LO: (prmDb) PrmIdNotFound : Parameter ID 0x4700 not found
EVENT: (3328) (2:1645592921,900083) WARNING_LO: (prmDb) PrmIdNotFound : Parameter ID 0x4701 not found
EVENT: (3328) (2:1645592921,900127) WARNING_LO: (prmDb) PrmIdNotFound : Parameter ID 0x2600 not found
EVENT: (3328) (2:1645592921,900169) WARNING_LO: (prmDb) PrmIdNotFound : Parameter ID 0x2601 not found
[WARNING] High task priority of 140 being clamped to 99
[WARNING] High task priority of 101 being clamped to 99
[WARNING] High task priority of 100 being clamped to 99
[WARNING] High task priority of 100 being clamped to 99
[WARNING] High task priority of 100 being clamped to 99
[WARNING] High task priority of 100 being clamped to 99
Connected to 192.168.86.153:50000 as a tcp client
[ERROR] Failed to send framed data: 1
[WARNING] High task priority of 100 being clamped to 99
[WARNING] High task priority of 100 being clamped to 99
[WARNING] High task priority of 100 being clamped to 99
[WARNING] High task priority of 120 being clamped to 99
[WARNING] High task priority of 119 being clamped to 99
EVENT: (512) (2:1645592921,902737) DIAGNOSTIC: (rateGroup1Comp) RateGroupStarted : Rate group started.
[WARNING] High task priority of 118 being clamped to 99
EVENT: (768) (2:1645592921,902926) DIAGNOSTIC: (rateGroup2Comp) RateGroupStarted : Rate group started.
EVENT: (1024) (2:1645592921,903113) DIAGNOSTIC: (rateGroup3Comp) RateGroupStarted : Rate group started.
EVENT: (1281) (2:1645592972,264771) COMMAND: (cmdDisp) OpCodeDispatched : Opcode 0xe00 dispatched to port 12
EVENT: (3584) (2:1645592972,264834) ACTIVITY_LO: (mathSender) COMMAND_RECV : Math command received: 42.419998 SUB 0.420000
EVENT: (1282) (2:1645592972,264992) COMMAND: (cmdDisp) OpCodeCompleted : Opcode 0xe00 completed
EVENT: (9985) (2:1645592972,923961) ACTIVITY_HI: (mathReceiver) OPERATION_PERFORMED : SUB operation performed
EVENT: (3585) (2:1645592972,924164) ACTIVITY_HI: (mathSender) RESULT : Math result is 42.000000
```
That all seems to work

### Wireless testing
Enable wireless on RPi and connect RPi to wifi network. Should be same IP address on the RPi as previous wired instructions. Note I needed to connect the RPi to a monitor, keyboard, and mouse to get all the wifi setup complete.

With RPi connected to wireless (and configured for SSH in previous steps), SSH into the RPi from the host (I used a terminal in VS Code IDE).

Collect IP port info: within an admin terminal on host Windows machine first run `ipconfig` to get the Windows host IP address, and within the WSL terminal run `ip addr | grep eth0` to get the WSL2 IP address. 
  
Setup ports; for host machine, need to open port per above and connect that port to the WSL2 IP address. Within the Windows admin terminal run the two netsh commands above, first to open the port on the host and then connect that port to the WSL2 instance IP address.
  
Within a WSL terminal, start the GDS 
  
From the RPi SSH terminal, start the applicaiton and connect to the Windows host IP address `pi@raspberrypi:~ $ sudo ./Ref -a <host IP Address> -p 50000`. Data should flow from RPi to GDS. Within the SSH terminal, should see the FSW responses (Events) to commands from the GDS, as well as the results in the GDS.

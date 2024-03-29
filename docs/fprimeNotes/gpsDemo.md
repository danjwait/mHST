# GPS demo

Runing with documentation [here](https://github.com/nasa/fprime/blob/devel/docs/Tutorials/GpsTutorial/Tutorial.md). 

Created the dir GpsApp/Gps

## Creating /GpsApp/Gps.fpp

Rather than GpsComponentAi.xml per tutorial, went back to [Math Component](https://github.com/danjwait/fprime/blob/devel/docs/Tutorials/MathComponent/Tutorial.md) demo and tried to follow that process; starting at [The MathSender Component](https://github.com/nasa/fprime/blob/devel/docs/Tutorials/MathComponent/Tutorial.md#The-MathSender-Component).

Wrote the fprime/GpsApp/Gps/Gps.fpp file based on the .xml files in [Designing the GPS Component](https://github.com/danjwait/fprime/blob/devel/docs/Tutorials/GpsTutorial/Tutorial.md#designing-the-gps-component) of GPS tutorial:
```
module GpsApp {

    @ Component for reading GPS strings from GPS hardware
    active component Gps {

        #-----
        # general ports
        #-----

        #-----
        # special ports
        #-----

        @ command receive port
        command recv port cmdIn

        @ command registration port
        command reg port cmdRegOut

        @ command reponse port
        command resp port cmdResponseOut

        @ event port
        event port eventOut

        @ text event port
        text event port textEventOut

        @ telemetry port
        telemetry port tlmOut

        @ receive serial data port
        async input port serialRecv: Drv.SerialRead

        @ serial buffer port
        output port serialBufferOut: Fw.BufferSend

        #-----
        # parameters
        #-----

        #-----
        # events
        #-----

        @ notification on GPS lock acquired
        event GPS_LOCK_ACQUIRED \
        severity activity high \
        id 0 \
        format "GPS lock acquired"

        @ warning on GPS lock lost
        event GPS_LOCK_LOST \
        severity warning high \
        id 1 \
        format "GPS lock lost"

        #-----
        # commands
        #-----

        @ command to force an EVR reporting lock status
        async command REPORT_STATUS \
        opcode 0

        #-----
        # telemetry
        #-----

        @ current latitude
        telemetry GPS_LATITUDE: F32 id 0

        @ current longitude
        telemetry GPS_LONGITUDE: F32 id 1

        @ current altitude
        telemetry GPS_ALTITUDE: F32 id 2

        @ current number of satellites
        telemetry GPS_SV_COUNT: F32 id 3
    }
}
```

Ran `/fprime/GpsApp$ fprime-util generate` which looked like it worked; get an error with build:
```
/fprime/GpsApp$ fprime-util build
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/settings.ini
make: *** No rule to make target 'GpsApp'.  Stop.
[CMAKE] CMake failed to detect target, attempting CMake cache refresh and retry
make: *** No rule to make target 'GpsApp'.  Stop.
[ERROR] CMake erred with return code 2
```
So paused here to setup /GpsApp project

## Setting up /GpsApp Project
DWJ Note: I got this "working" to start (that is, it wouldn't throw an error) then went back to /GpsApp/Gps to work on the module, but when I tried to build /GpsApp I found there were additional issues. This is a condensed set of the changes I made to the /GpsApp project files, but these steps aren't in the order I ran them.

Copied the /Ref/Top directory over to /GpsApp/Top per [Clone the Ref Application](https://github.com/nasa/fprime/blob/devel/docs/Tutorials/GpsTutorial/Tutorial.md#clone-the-ref-application) in GPS tutorial. Did not find the files in that directory that were supposed to be removed per GPS Tutotorial

Copied the CMakeLists.txt file over from /Ref with `fprime/GpsApp$ cp ../Ref/CMakeLists.txt .` That CMakeLists.txt included other components in Ref & Math demo; so modified /GpsApp/CMakeLists.txt to point back to those components in /Ref/;
```
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/PingReceiver/")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/RecvBuffApp/")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/SendBuffApp/")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/SignalGen/")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/MathTypes")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/MathPorts")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/MathSender")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/MathReceiver")
```

Added Gps component to /GpsApp/CMakeLists.txt: `add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/Gps")`

Noticed that the copied Ref CMakeLists.txt had a line `project(Ref VERSION 1.0.0 LANGUAGES C CXX)` so changed Ref to GpsApp. 

Added Gps in /GpsApp/instances.fpp at the end of the active group:
```
  instance gps: GpsApp.Gps base id 0x0F00 \
    queue size Default.queueSize \
    stack size Default.stackSize \
    priority 100 \
  {

  }
 ```
I left the ` { } ` because I wasn't sure if gps needed additional configuration data like some of other componets.

In same file added  the linux serial driver at the end of  `# Passive component instances` as:
```
  instance gpsSerial: Drv.LinuxSerialDriver base id 0x4C00 \
    at "../../Drv/LinuxSerialDriver/LinuxSerialDriver.hpp" \
  {


  }
```
Again, I left the ` { } ` because I wasn't sure if gps needed additional configuration data like some of other componets.

In file /GpsApp/topology.fpp added:
```
    instance gpsSerial
    instance gps
```
in same file also modified:
```
module GpsApp {
....
 topology GpsApp {
 ...
```

and in `/GpsApp/Top/instances.fpp` change the first line to `module GpsApp`

Ended up going into `GpsApp/Top` (copied over from /Ref/Top previously) and deleting everything there unitl all that was left was:
```
djwait@TRON:~/02_Projects/fprime/GpsApp/Top$ ls -lrt
total 84
-rw-r--r-- 1 djwait djwait   427 Mar  8 20:38 CMakeLists.txt
-rw-r--r-- 1 djwait djwait    19 Mar  8 20:38 .gitignore
-rw-r--r-- 1 djwait djwait 45705 Mar 21 19:50 GpsTopologyAppAi.xml
-rw-r--r-- 1 djwait djwait  9952 Mar 21 21:31 instances.fpp
-rw-r--r-- 1 djwait djwait  4948 Mar 21 21:35 topology.fpp
drwxr-xr-x 5 djwait djwait  4096 Mar 21 21:35 ..
drwxr-xr-x 2 djwait djwait  4096 Mar 21 21:41 .
```
Looked at /GpsApp/Top/CMakeLists.txt and see a reference to a `RefTopologyDefs.cpp` file; copied that back into /GpsApp/Top and replace `Ref` with `GpsApp` within the file, changed file name to match, and updated /GpsApp/Top/CMakeLists.txt file to match

Likewise copied RefTopologyAc.hpp and RefTopologyAc.cpp over from /Ref and replaced `Ref` with `GpsApp` within files and changed file names to match. Ended up doing that over and over until /GpsApp/Top had this set of files, copied from /Ref and edited & renamed to replace `Ref` with `GpsApp`:
```
~/02_Projects/fprime/GpsApp/Top$ ls -lrt
total 140
-rw-r--r-- 1 djwait djwait 45705 Mar 21 19:50 GpsTopologyAppAi.xml
-rw-r--r-- 1 djwait djwait  9952 Mar 21 21:31 instances.fpp
-rw-r--r-- 1 djwait djwait  4948 Mar 21 21:35 topology.fpp
-rw-r--r-- 1 djwait djwait  5732 Mar 21 21:52 GpsAppTopologyAc.hpp
-rw-r--r-- 1 djwait djwait 39979 Mar 21 21:56 GpsAppTopologyAc.cpp
-rw-r--r-- 1 djwait djwait  2106 Mar 21 21:56 Main.cpp
-rw-r--r-- 1 djwait djwait   430 Mar 21 21:58 CMakeLists.txt
-rw-r--r-- 1 djwait djwait   196 Mar 21 22:07 GpsAppTopologyDefs.cpp
-rw-r--r-- 1 djwait djwait   694 Mar 21 22:10 FppConstantsAc.hpp
-rw-r--r-- 1 djwait djwait   488 Mar 21 22:10 FppConstantsAc.cpp
-rw-r--r-- 1 djwait djwait  1647 Mar 21 22:11 GpsAppTopologyDefs.hpp
```

Re-ran `fprime-util purge` in /GpsApp and now with `fprime-util generate` see:
```
...
-- Adding Library: Utils_Types
-- Adding Library: Ref_PingReceiver
-- Adding Library: Ref_RecvBuffApp
-- Adding Library: Ref_SendBuffApp
-- Adding Library: Ref_SignalGen
-- Adding Library: Ref_MathTypes
-- Adding Library: Ref_MathPorts
-- Adding Library: Ref_MathSender
-- Adding Library: Ref_MathReceiver
-- Adding Library: GpsApp_Gps
-- Adding Library: GpsApp_Top
-- Adding Deployment: GpsApp
-- Configuring done
-- Generating done
-- Build files have been written to: /home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-native
```

## Working on /GpsApp/Gps component
Change into GpsApp/Gds directory and run:
```
~/02_Projects/fprime/GpsApp/Gps$ fprime-util impl
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/GpsApp/settings.ini
Scanning dependencies of target Fw_Cfg
[  4%] Building CXX object F-Prime/Fw/Cfg/CMakeFiles/Fw_Cfg.dir/ConfigCheck.cpp.o
[  4%] Linking CXX static library ../../../lib/Linux/libFw_Cfg.a
[  4%] Built target Fw_Cfg
Scanning dependencies of target codegen
[ 12%] Built target codegen
...
uilt target Fw_CompQueued
Scanning dependencies of target GpsApp_Gps_impl
[ 96%] Generating GpsComponentAi.xml
fpp-to-xml
/home/djwait/02_Projects/fprime/GpsApp/Gps/Gps.fpp: 4.5
    active component Gps {
    ^
error: component with event specifiers must have time get port
make[3]: *** [GpsApp/Gps/CMakeFiles/GpsApp_Gps_impl.dir/build.make:75: GpsApp/Gps/GpsComponentAi.xml] Error 1
make[2]: *** [CMakeFiles/Makefile2:7848: GpsApp/Gps/CMakeFiles/GpsApp_Gps_impl.dir/all] Error 2
make[1]: *** [CMakeFiles/Makefile2:7855: GpsApp/Gps/CMakeFiles/GpsApp_Gps_impl.dir/rule] Error 2
make: *** [Makefile:2205: GpsApp_Gps_impl] Error 2
[ERROR] CMake erred with return code 2
```
So added time get port per Math Component tutorial for Gps.fpp:
```
    @ Time get port
    time get port timeGetOut
```
Re-run:
```
~/02_Projects/fprime/GpsApp/Gps$ fprime-util impl
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/GpsApp/settings.ini
-- Searching for F prime modules in: /home/djwait/02_Projects/fprime
-- Autocoder constants file: /home/djwait/02_Projects/fprime/config/AcConstants.ini
-- Configuration header directory: /home/djwait/02_Projects/fprime/config
-- [fpp-tools] Searching for fpp-tools
...
[ 96%] Generating GpsComponentAi.xml
[100%] Generating ../../../Gps/GpsComponentImpl.hpp-template, ../../../Gps/GpsComponentImpl.cpp-template
Parsing Component Gps
Parsing Interface SerialRead
Parsing Interface BufferSend
Parsing Interface Cmd
Parsing Interface CmdReg
Parsing Interface CmdResponse
Parsing Interface Log
Parsing Interface LogText
Parsing Interface Time
Parsing Interface Tlm
Enabled generation of implementation template files...
[100%] Built target GpsApp_Gps_impl
```
See the Impl files:
```
~/02_Projects/fprime/GpsApp/Gps$ ls -lrt
total 16
-rw-r--r-- 1 djwait djwait  149 Mar  8 20:31 CMakeLists.txt
-rw-r--r-- 1 djwait djwait    0 Mar 14 20:34 Gps.cpp
-rw-r--r-- 1 djwait djwait 1734 Mar 14 20:55 Gps.fpp
-rw-r--r-- 1 djwait djwait 2266 Mar 14 20:56 GpsComponentImpl.hpp-template
-rw-r--r-- 1 djwait djwait 1799 Mar 14 20:56 GpsComponentImpl.cpp-template
```
Noted that the Gps.cpp file I'd generated by `touch Gps.cpp` is still empty; removed it and ran `mv` to change the names of the Imp. - template files 

Started editing Gps.cpp per notes at [GpsApp/Gps/GpsComponentImpl.cpp (Sample)](https://github.com/nasa/fprime/blob/devel/docs/Tutorials/GpsTutorial/Tutorial.md#gpsappgpsgpscomponentimplcpp-sample)

When I got to [GpsApp/Gps/CMakeListst.txt (final)](GpsApp/Gps/CMakeListst.txt (final)) I tried using the CMakeLists.txt there but got an error w/ the ` fprime-util build` command, so edit the GpsApp/Gps/CMakeLists.txt file to include the .fpp in place of the .hpp file:
```
# Register the standard build
set(SOURCE_FILES
	"${CMAKE_CURRENT_LIST_DIR}/Gps.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/Gps.fpp"
)
register_fprime_module()
```

Edit the Gps.hpp file:
```
#ifndef Gps_HPP
#define Gps_HPP

#include "GpsApp/Gps/GpsComponentAc.hpp"
```

I then ran `/02_Projects/fprime/GpsApp/Gps$ fprime-util purge` and `fprime-util generate` and `fprime-util build` and see different errors:

Error in Gps.cpp at ` void Gps :: preamble()` with a type mismatch on the (U64) and the buffer; commented that out to see what the next error would be:
```
//this->m_recvBuffers[buffer].setData((U64)this->m_uartBuffers[buffer]);
```

At `Drv::SerialReadStatus &serial_status` I had missed the &, so fixed that. 

In ` void Gps ::serialRecv_handler` another error with the [renamed symbols](https://nasa.github.io/fprime/UsersGuide/dev/v3-renamed-symbols.html) ; `Drv::SER_OK` becomes `Drv::SerialReadStatus::SER_OK` 
    
Then another type error at `//Fw::Logger::logMsg("[WARNING] Received buffer with bad packet: %d\n", serial_status);` so commented that line out.

In step 4 generate telemetry getting other errors; this code (note I tried two different ways of using `tlmWrite_` on purpose to look for different errors):
```
// Step 4: generate telemetry
tlmWrite_Gps_Latitude(lat);
this->tlmWrite_Gps_Longitude(lon);
```
generates this errors with build:
```
...
[100%] Built target Drv_SerialDriverPorts
Scanning dependencies of target GpsApp_Gps
[100%] Building CXX object GpsApp/Gps/CMakeFiles/GpsApp_Gps.dir/Gps.cpp.o
/home/djwait/02_Projects/fprime/GpsApp/Gps/Gps.cpp: In member function ‘virtual void GpsApp::Gps::serialRecv_handler(NATIVE_INT_TYPE, Fw::Buffer&, Drv::SerialReadStatus&)’:
/home/djwait/02_Projects/fprime/GpsApp/Gps/Gps.cpp:169:5: error: ‘tlmWrite_Gps_Latitude’ was not declared in this scope
  169 |     tlmWrite_Gps_Latitude(lat);
      |     ^~~~~~~~~~~~~~~~~~~~~
/home/djwait/02_Projects/fprime/GpsApp/Gps/Gps.cpp:170:11: error: ‘class GpsApp::Gps’ has no member named ‘tlmWrite_Gps_Longitude’
  170 |     this->tlmWrite_Gps_Longitude(lon);
...
```
I needed to have `this->` preceeding the timWrite, and I'd 1) put the tememetry channels as all caps in the .fpp file (e.g. `GPS_LATITUDE`) and 2) had not defined the Gps_LockStatus in the .fpp file; I did that as:
```
...
        @ current lock status
        telemetry GPS_LOCK_STATUS: U32 id 4
...
```
I had made the same errors (all caps in Gps.fpp, camel case in .cpp); fixed as:
```
    // Only generate lock status event on change
    if (packet.lock == 0 && m_locked) {
      m_locked = false;
      log_WARNING_HI_GPS_LOCK_LOST();
    } else if (packet.lock == 1 && !m_locked) {
      m_locked = true;
      log_ACTIVITY_HI_GPS_LOCK_ACQUIRED();
```
Note I made the events camel case in two places in the Gps.cpp file, so needed to fix to all caps in both places.

With all those changes, building the GPS component appears to work:
```
djwait@Aero-FacLPT-01:~/02_Projects/fprime/GpsApp/Gps$ fprime-util build
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/GpsApp/settings.ini
[  4%] Built target Fw_Cfg
[ 12%] Built target codegen
[ 25%] Built target Fw_Types
[ 29%] Built target Utils_Hash
[ 29%] Built target Fw_Logger
[ 33%] Built target Fw_Obj
[ 37%] Built target Fw_Port
[ 41%] Built target Fw_Time
[ 45%] Built target Fw_Com
[ 50%] Built target Fw_Tlm
[ 58%] Built target Fw_Log
[ 66%] Built target Fw_Cmd
[ 70%] Built target Fw_Prm
[ 75%] Built target Fw_Buffer
[ 75%] Built target Fw_Comp
[ 91%] Built target Os
[ 95%] Built target Fw_CompQueued
[100%] Built target Drv_SerialDriverPorts
Scanning dependencies of target GpsApp_Gps
[100%] Building CXX object GpsApp/Gps/CMakeFiles/GpsApp_Gps.dir/Gps.cpp.o
[100%] Building CXX object GpsApp/Gps/CMakeFiles/GpsApp_Gps.dir/GpsComponentAc.cpp.o
[100%] Linking CXX static library ../../lib/Linux/libGpsApp_Gps.a
[100%] Built target GpsApp_Gps
djwait@Aero-FacLPT-01:~/02_Projects/fprime/GpsApp/Gps$ 
```

When building /GpsApp/ ran into type conversion error:
```
...
[100%] Built target GpsApp_Top
Scanning dependencies of target GpsApp_dict
[100%] Built target GpsApp_dict
Scanning dependencies of target GpsApp
[100%] Building CXX object CMakeFiles/GpsApp.dir/Top/Main.cpp.o
In file included from /home/djwait/02_Projects/fprime/Drv/LinuxSerialDriver/LinuxSerialDriver.hpp:9,
                 from /home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-native/GpsApp/Top/GpsAppTopologyAc.hpp:16,
                 from /home/djwait/02_Projects/fprime/GpsApp/Top/Main.cpp:6:
/home/djwait/02_Projects/fprime/Drv/LinuxSerialDriver/LinuxSerialDriverComponentImpl.hpp:74:65: error: conversion to ‘NATIVE_INT_TYPE’ {aka ‘int’} from ‘NATIVE_UINT_TYPE’ {aka ‘unsigned int’} may change the sign of the result [-Werror=sign-conversion]
   74 |       void startReadThread(NATIVE_INT_TYPE priority = Os::Task::TASK_DEFAULT, NATIVE_INT_TYPE stackSize = Os::Task::TASK_DEFAULT, NATIVE_INT_TYPE cpuAffinity = Os::Task::TASK_DEFAULT);
      |                                                       ~~~~~~~~~~^~~~~~~~~~~~
/home/djwait/02_Projects/fprime/Drv/LinuxSerialDriver/LinuxSerialDriverComponentImpl.hpp:74:117: error: conversion to ‘NATIVE_INT_TYPE’ {aka ‘int’} from ‘NATIVE_UINT_TYPE’ {aka ‘unsigned int’} may change the sign of the result [-Werror=sign-conversion]
   74 | NATIVE_INT_TYPE priority = Os::Task::TASK_DEFAULT, NATIVE_INT_TYPE stackSize = Os::Task::TASK_DEFAULT, NATIVE_INT_TYPE cpuAffinity = Os::Task::TASK_DEFAULT);
      |                                                                                ~~~~~~~~~~^~~~~~~~~~~~

In file included from /home/djwait/02_Projects/fprime/Drv/LinuxSerialDriver/LinuxSerialDriver.hpp:9,
                 from /home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-native/GpsApp/Top/GpsAppTopologyAc.hpp:16,
                 from /home/djwait/02_Projects/fprime/GpsApp/Top/Main.cpp:6:
/home/djwait/02_Projects/fprime/Drv/LinuxSerialDriver/LinuxSerialDriverComponentImpl.hpp:74:171: error: conversion to ‘NATIVE_INT_TYPE’ {aka ‘int’} from ‘NATIVE_UINT_TYPE’ {aka ‘unsigned int’} may change the sign of the result [-Werror=sign-conversion]
   74 |  NATIVE_INT_TYPE stackSize = Os::Task::TASK_DEFAULT, NATIVE_INT_TYPE cpuAffinity = Os::Task::TASK_DEFAULT);
      |                                                                                    ~~~~~~~~~~^~~~~~~~~~~~

cc1plus: all warnings being treated as errors
make[3]: *** [CMakeFiles/GpsApp.dir/build.make:63: CMakeFiles/GpsApp.dir/Top/Main.cpp.o] Error 1
make[2]: *** [CMakeFiles/Makefile2:2285: CMakeFiles/GpsApp.dir/all] Error 2
make[1]: *** [CMakeFiles/Makefile2:2292: CMakeFiles/GpsApp.dir/rule] Error 2
make: *** [Makefile:177: GpsApp] Error 2
[ERROR] CMake erred with return code 2
```
Wrote F' [question on this](https://github.com/nasa/fprime/discussions/1343) and worked around by going into `rv/LinuxSerialDriver/LinuxSerialDriver.hpp` and editing that line to:
```
void startReadThread(NATIVE_UINT_TYPE priority = Os::Task::TASK_DEFAULT, NATIVE_UINT_TYPE stackSize = Os::Task::TASK_DEFAULT, NATIVE_UINT_TYPE cpuAffinity = Os::Task::TASK_DEFAULT);
```
Then the GpsApp built:
```
...
-- Installing: /home/djwait/02_Projects/fprime/GpsApp/build-artifacts/Linux/lib/static/libSvc_TlmChan.a
-- Installing: /home/djwait/02_Projects/fprime/GpsApp/build-artifacts/Linux/lib/static/libGpsApp_Top.a
-- Installing: /home/djwait/02_Projects/fprime/GpsApp/build-artifacts/Linux/dict/GpsAppTopologyAppDictionary.xml
[100%] Built target GpsApp
```

## Finishing Topology
Checked for onconnected ports:
```
~/02_Projects/fprime/GpsApp/Top$ fprime-util fpp-check -u unconnected.txt
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/GpsApp/settings.ini
[INFO] Updating fpp locations file and build cache. This may take some time.
djwait@TRON:~/02_Projects/fprime/GpsApp/Top$ cat unconnected.txt 
Topology GpsApp.GpsApp:
  GpsApp.chanTlm.TlmGet
  GpsApp.cmdSeq.seqCancelIn
  GpsApp.cmdSeq.seqDone
  GpsApp.cmdSeq.seqRunIn
  GpsApp.comm.poll
  GpsApp.comm.ready
  GpsApp.fileDownlink.FileComplete
  GpsApp.fileDownlink.SendFile
  GpsApp.gps.serialBufferOut
  GpsApp.gps.serialRecv
  GpsApp.gpsSerial.readBufferSend
  GpsApp.gpsSerial.serialRecv
  GpsApp.gpsSerial.serialSend
  GpsApp.health.WdogStroke
  GpsApp.uplink.framedPoll
  GpsApp.uplink.schedIn
```
In /GpsApp/Top/topology.fpp:
```
      rateGroup1Comp.RateGroupMemberOut[6] -> uplink.schedIn
      ...
    connections Gps {
      gpsSerial.serialRecv -> gps.serialRecv
      gps.serialBufferOut -> gpsSerial.readBufferSend
    }
```
Did not connect the other ports, went back and rebuilt /GpsApp (native). Also generated for raspberrypi and built for raspberrypi

Both builds look successful:
```
/02_Projects/fprime/GpsApp/build-artifacts$ ls -lrt
total 12
drwxr-xr-x 5 djwait djwait 4096 Mar 22 20:25 Linux
drwxr-xr-x 5 djwait djwait 4096 Mar 22 22:03 raspberrypi
drwxr-xr-x 4 djwait djwait 4096 Mar 22 22:05 logs
```
Able to start GDS on host WSL2 machine with `~/02_Projects/fprime/GpsApp/build-artifacts$ fprime-gds -g html -r .` and see GPS command for EVR and telemetry channels

## Testing on RPi 4
Connect RPi 4 to [Adafruit Ultimate GPS Featherwing](Adafruit Ultimate GPS featherwing) and [SparkFun Pi Wedge](https://learn.sparkfun.com/tutorials/preassembled-40-pin-pi-wedge-hookup-guide/all), with separate 3.3V supply.

See FIX LED change to ~15 second interval on GPS Featherwing, which implies that the GPS Featherwing has GPS lock.

Setup UART serial on RPi 4 per directions [here](https://maker.pro/raspberry-pi/tutorial/how-to-use-a-gps-receiver-with-raspberry-pi-4). 

Connect to RPi from host per [Cross-compile testing](https://github.com/danjwait/mHST/blob/main/docs/fprimeNotes/crossCompile.md#cross-compile-testing) and scp RPi GpsApp over. 
Start fprime-gds on host. 
Start GpsApp on RPi

See data flow both directions from host to RPi, with Math commands from Math demo, and gps.REPORT_STATUS. EVR from each REPORT_STATUS reports "GPS lock lost" and no GPS channelized TLM shows up (including lock status). 

with `cat /dev/serial0` on RPi4 see GPS messages, so know that data flow from featherwing to RPi 4 is working. 

Looks like the GpsApp/Top/topology may have an issue; looks like I'm not starting the gps as part of the rate group; had this:
```
      # Rate group 1
      rateGroupDriverComp.CycleOut[Ports_RateGroups.rateGroup1] -> rateGroup1Comp.CycleIn
      rateGroup1Comp.RateGroupMemberOut[0] -> SG1.schedIn
      rateGroup1Comp.RateGroupMemberOut[1] -> SG2.schedIn
      rateGroup1Comp.RateGroupMemberOut[2] -> chanTlm.Run
      rateGroup1Comp.RateGroupMemberOut[3] -> fileDownlink.Run
      rateGroup1Comp.RateGroupMemberOut[4] -> systemResources.run
      rateGroup1Comp.RateGroupMemberOut[5] -> mathReceiver.schedIn
      rateGroup1Comp.RateGroupMemberOut[6] -> uplink.schedIn
```
which never started the gps component. It also looks like I didn't have a .schedIn port in Gps.fpp, so added that per pattern in mathReciever:
```
        #-----
        # general ports
        #-----

        @ the rate group scheduler input
        sync input port schedIn: Svc.Sched
```
and changed `rateGroup1Comp.RateGroupMemberOut[6] -> uplink.schedIn` to `rateGroup1Comp.RateGroupMemberOut[6] -> gps.schedIn` (I'm not sure why I had `uplink.schedIn` there).

Now get a new error:
```
...
[ 98%] Building CXX object GpsApp/Top/CMakeFiles/GpsApp_Top.dir/GpsAppTopologyDefs.cpp.o
[ 98%] Building CXX object GpsApp/Top/CMakeFiles/GpsApp_Top.dir/FppConstantsAc.cpp.o
[ 98%] Building CXX object GpsApp/Top/CMakeFiles/GpsApp_Top.dir/GpsAppTopologyAc.cpp.o
/home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-native/GpsApp/Top/GpsAppTopologyAc.cpp:179:9: error: cannot declare variable ‘GpsApp::{anonymous}::gps’ to be of abstract type ‘GpsApp::Gps’
  179 |     Gps gps(FW_OPTIONAL_NAME("gps"));
      |         ^~~
In file included from /home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-native/GpsApp/Top/GpsAppTopologyAc.hpp:18,
                 from /home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-native/GpsApp/Top/GpsAppTopologyAc.cpp:12:
/home/djwait/02_Projects/fprime/GpsApp/Gps/Gps.hpp:28:9: note:   because the following virtual functions are pure within ‘GpsApp::Gps’:
   28 |   class Gps :
      |         ^~~
 ...
 ```
 
 After trying to sole the above, still stuck. Still seeing in `/home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-native/GpsApp/Top/GpsAppTopologyAc.cpp` :
 ```
     // fileUplinkBufferManager
    Svc::BufferManager fileUplinkBufferManager(FW_OPTIONAL_NAME("fileUplinkBufferManager"));

    // gps
    Gps gps(FW_OPTIONAL_NAME("gps"));

    // gpsSerial
    Drv::LinuxSerialDriver gpsSerial(FW_OPTIONAL_NAME("gpsSerial"));

    // health
    Svc::Health health(FW_OPTIONAL_NAME("health"));
```
Can't figure out why Gps shows up that way, as opposed to what I'd expect like `GpsApp::Gps gps(gps(FW_OPTIONAL_NAME("gps"));`

## Present status
Reverted back to when I had `uplink.schedIn` in the rate group; left gps as an active component, commented out the `sync input port schedIn: Svc.Sched` in Gps.fpp. Got back to where I could build GpsApp for native and raspberrypi.

I noticed that the old version of the demo has this line to start the GpsApp on the RPi:
```
./GpsApp -a <ground system IP> -p 50000 -d <serial port, /dev/ttyACM0 for USB>
```
and the old [GpsApp Main.cpp](https://github.com/LeStarch/fprime/blob/gps-application/GpsApp/Top/Main.cpp) file has this code: 
```
int main(int argc, char* argv[]) {
    I32 option = 0;
    U32 port_number = 50000;
    char* hostname = NULL;
    char* device = (char*)"/dev/ttyUSB0";

    while ((option = getopt(argc, argv, "hp:a:d:")) != -1){
        switch(option) {
            case 'h':
                print_usage();
                return 0;
                break;
            case 'p':
                port_number = atoi(optarg);
                break;
            case 'a':
                hostname = optarg;
                break;
            case 'd':
                device = optarg;
                break;
            case '?':
                return 1;
            default:
                print_usage();
                return 1;
        }
    }
```
but my command to start the app did not have that; on the RPi I tied `sudo ./GpsApp -a <IP address> -p 50000 -d /dev/serial0` and get an invalid option error and a assert from Svc/BufferManager ; my Main.cpp does not have a -d option, since I copied it from /Ref: 
```
int main(int argc, char* argv[]) {
    U32 port_number = 0; // Invalid port number forced
    I32 option;
    char *hostname;
    option = 0;
    hostname = nullptr;

    while ((option = getopt(argc, argv, "hp:a:")) != -1){
        switch(option) {
            case 'h':
                print_usage(argv[0]);
                return 0;
                break;
            case 'p':
                port_number = static_cast<U32>(atoi(optarg));
                break;
            case 'a':
                hostname = optarg;
                break;
            case '?':
                return 1;
            default:
                print_usage(argv[0]);
                return 1;
        }
    }

    (void) printf("Hit Ctrl-C to quit\n");
```
so looks like I need to re-write Main.cpp    

### Updates to Main.cpp
Added `device` to /GpsApp/Top/Main.cpp:
```
int main(int argc, char* argv[]) {
    U32 port_number = 0; // Invalid port number forced
    I32 option;
    char *hostname;
    char *device;
    option = 0;
    hostname = nullptr;
    device = nullptr;

    while ((option = getopt(argc, argv, "hp:a:d:")) != -1){
        switch(option) {
            case 'h':
                print_usage(argv[0]);
                return 0;
                break;
            case 'p':
                port_number = static_cast<U32>(atoi(optarg));
                break;
            case 'a':
                hostname = optarg;
                break;
            case 'd':
                device = optarg;
                break;
            case '?':
                return 1;
            default:
                print_usage(argv[0]);
                return 1;
        }
    }
```
Also needed to update GpsAppTopologyDefs.hpp to include the device, followed pattern for hostName:
```
  // State for topology construction
  struct TopologyState {
    TopologyState() :
      hostName(""),
      portNumber(0),
      device("")
    {

    }
    TopologyState(
        const char *hostName,
        U32 portNumber,
        const char *device
    ) :
      hostName(hostName),
      portNumber(portNumber),
      device(device)
    {

    }
    const char* hostName;
    U32 portNumber;
    const char *device;
  };
```
Purged, generated, and built both native and raspberrypi. scp new GpsApp to RPi, start the GPS receiver, start the GpsApp on the RPI; don't see a seg fault this time when start GpsApp:
```
sudo ./GpsApp -a 192.168.86.153 -p 50000 -d /dev/serial0
```
but while `cat /dev/serial0` shows GPS data, and the LED on the receiver shows lock, the GPS command `gps.REPORT_STATUS` still returns lock lost, and the GPS telemetry is still not updating. So I think I've still not connected the serial driver correctly.

I think I need something like this from GpsApp/Top/instances.fpp:
```
    phase Fpp.ToCpp.Phases.startTasks """
    // Initialize socket server if and only if there is a valid specification
    if (state.hostName != nullptr && state.portNumber != 0) {
        Os::TaskString name("ReceiveTask");
        // Uplink is configured for receive so a socket task is started
        comm.configure(state.hostName, state.portNumber);
        comm.startSocketTask(
            name,
            true,
            ConfigConstants::comm::PRIORITY,
            ConfigConstants::comm::STACK_SIZE
        );
    }
```
but for the serial.

Yes, it looks like this is per [Init Specifiers](https://fprime-community.github.io/fpp/fpp-users-guide.html#Defining-Component-Instances_Init-Specifiers) in the FPP guide and [14.2. Implementing Deployments](https://fprime-community.github.io/fpp/fpp-users-guide.html#Writing-C-Plus-Plus-Implementations_Implementing-Deployments)

Tried this:
```
  instance gpsSerial: Drv.LinuxSerialDriver base id 0x4C00 \
    at "../../Drv/LinuxSerialDriver/LinuxSerialDriver.hpp" \
  {
    phase Fpp.ToCpp.Phases.startTasks """
    gpsSerial.open(
      state.device,
      Drv::LinuxSerialDriverComponentImpl::BAUD_RATE::BAUD_9600,
      Drv::LinuxSerialDriverComponentImpl::FLOW_CONTROL::NO_FLOW,
      Drv::LinuxSerialDriverComponentImpl::PARITY::PARITY_NONE,
      false
    );
    """
  }
```
With fprime-util build get an error:
```
...
sApp/Top/GpsAppTopologyAc.cpp:12:
/home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-native/GpsApp/Gps/GpsComponentAc.hpp:265:18: note:    ‘virtual void GpsApp::GpsComponentBase::PingIn_handler(NATIVE_INT_TYPE, U32)’
  265 |     virtual void PingIn_handler(
      |                  ^~~~~~~~~~~~~~
make[3]: *** [GpsApp/Top/CMakeFiles/GpsApp_Top.dir/build.make:251: GpsApp/Top/CMakeFiles/GpsApp_Top.dir/GpsAppTopologyAc.cpp.o] Error 1
...
```
so commented out the health pings I'd put in Gps.fpp. Think I should have put those in before and then added them in the implimentation.

Purged, generated, and built both native and raspberrypi. scp new GpsApp to RPi, start the GPS receiver, start the GpsApp on the RPI; this time I see that at least the device was opened:
```
...
[WARNING] High taks priority of 100 being clamped to 99
EVENT: (19460) (2:1648789092,437810) ACTIVITY_HI: (gpsSerial) DR_PortOpened : UART Device /dev/serial0 configured
...
```
but still no GPS data (Math commands still work, gps.REPORT_STATUS still works)

Tried adding startReadThread to /GpsApp/Top/instances.fpp:
```
...
  instance gpsSerial: Drv.LinuxSerialDriver base id 0x4C00 \
    at "../../Drv/LinuxSerialDriver/LinuxSerialDriver.hpp" \
  {
    phase Fpp.ToCpp.Phases.startTasks """
    gpsSerial.open(
      state.device,
      Drv::LinuxSerialDriverComponentImpl::BAUD_RATE::BAUD_9600,
      Drv::LinuxSerialDriverComponentImpl::FLOW_CONTROL::NO_FLOW,
      Drv::LinuxSerialDriverComponentImpl::PARITY::PARITY_NONE,
      false
    );
    gpsSerial.startReadThread(
      Os::Task::TASK_DEFAULT,
      Os::Task::TASK_DEFAULT,
      Os::Task::TASK_DEFAULT
    );
    """
  }
...
```
Purged, generated, and build both native and raspberrypi fine; loaded to RPi and ran, same behaivor as without the startReadThread. 

Added debug Fw::Logger statements to see where the code wasn't working. Keep seeing `[WARNING] Unfull message buffer: 1` message.

Tried rebuilding files in GpsApp/Top per [fprime/RPI/Top/](https://github.com/nasa/fprime/tree/devel/RPI/Top) :
 - changed the gpsSerial startup per `instance uartDrv: Drv.LinuxSerialDriver base id 2000 ` in [/RPI/Top/instances.fpp](https://github.com/nasa/fprime/blob/devel/RPI/Top/instances.fpp)
 - changed GpsApp/Top/GpsAppTopologyDefs.hpp and GpsApp/Top/GpsAppTopologyDefs.cpp to include namespace Init with a bool status = true

Ran that on RPI and didn't see a change. 
Block commented out the "if not enough data is available..." lines in Gps.cpp and tried again; see `[STATUS] GPS parsing in work: 0` come out, but then there's a set of those messages w/ numbers  from ~40 to ~ 116, then a seg fault

Changed all task priorities to less than 100 to avoid the "clamping" message; still see the same errors

April 11 2022:
 - Line commented out the serial parsing errors, and the GpsApp doesn't seg fault any more
 - Commands to the GpsApp work (EVR reports no lock)
 - `cat /dev/serial0` still shows chopped up looking GPS messages when the app is running, but messages are fine when app is stopped

Tired hard-coding values in for GPS telemetry, but still don't see data come back. Also note that gpsSerial.DR_BytesSent and gpsSerial.DR_BytesRecv aren't ever populated. I'd guess from looking at the code in the Gps component that the code is never getting to "step 4: generate telemetry" but seems like the serial driver should be reporting its status. 

Also noticed that no longer seeing the message `EVENT: (19460) (2:1648789092,437810) ACTIVITY_HI: (gpsSerial) DR_PortOpened : UART Device /dev/serial0 configured`

Tried adding `at "../../Drv/LinuxSerialDriver/LinuxSerialDriver.hpp" \` back in instances.fpp but don't see the port opened message back. Undid the addition.

April 12 2022:
As a guess, tried increasing the size of the buffer check:
```
    else if (buffsize < 42 ) {
```
Didn't change the behavior, so changed that back. Noted that still see the choppy response on the `cat /dev/serial0` on the RPi, so undid the change.

April 13 & 14 2022:
Noticed that the gpsSerial telemetry points weren't updating either, so thought it may be something broken in the GpsApp components. The LinuxSerialDriver has the telemetry points in the .fppi file, but they aren't "brought out" to the port in the .cpp file per [discussion](https://github.com/nasa/fprime/discussions/1404). 

Added to Gps.cpp a set of `this->tlmWrite_GPS_LATITUDE(lat);` debug TLM points; first at the start of serialRecv_handler(), which indicated that the handler was running. It looks like the line `U32 buffsize = static_cast<U32>(serBuffer.getSize());` is returning 1 almost all the time (as near as I could tell based on the telemetry updates). It looks like the sscanf call never returns a status other than 0, and the buffsize is always 1. So something wrong w/ the buffsize?

April 24 2022:
Added debug messages when UART connected; it looks like UART is working on the GpsApp, but somehow serialRecv_handler is still only seeing one buffer, ever.

April 25 2022:
tried adding `linuxTimer` per RPIdemo for rate groups, in case the serial driver was running at an odd rate. On RPI get a new segfault:
```
EVENT: (512) (2:1650945076,810468) DIAGNOSTIC: (rateGroup1Comp) RateGroupStarted : Rate group started.
EVENT: (768) (2:1650945076,810634) DIAGNOSTIC: (rateGroup2Comp) RateGroupStarted : Rate group started.
Assert file "/home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-raspberrypi/F-Prime/Svc/Cycle/CyclePortAc.cp
EVENT: (1024) (2:1650945076,810811) DIAGNOSTIC: (rateGroup3Comp) RateGroupStarted : Rate group started.
FATAL 16896 handled.
[ERROR] Failed to send framed data: 1
Connected to 192.168.86.153:50000 as a tcp client
Exiting with segfault and core dump file.
Segmentation fault
```

April 26 2022:
seems like segfault was beucase I was trying to use linuxTimer in place of the blockDrv timer; I thought I fixed it (and it looked like everything was running at 10Hz), but then it came back. 
posted question to [forum](https://github.com/nasa/fprime/discussions/1420) on the buffer 

April 28 2022:
 - Seemed to have resolved the segfault. I think I misconfigured something. But I also think that I've attached a lot of things to a 10 Hz rate group.
 - Updated per [fixing issues with small stack sizes #1427](https://github.com/nasa/fprime/pull/1427)
 - Still can't get Gps component to parse.
 - Running fpp-check getting weird error:
```
~/02_Projects/fprime/GpsApp/Top$ fpp-check instances.fpp topology.fpp -u unconnected.txt
fpp-check
/home/djwait/02_Projects/fprime/GpsApp/Top/instances.fpp: 19.22
  instance blockDrv: Drv.BlockDriver base id 0x0100 \
                     ^
error: undefined symbol Drv
```
but the driver is in GpsAppTopologyDefs.hpp and GpsAppTopologyDefs.cpp
`Drv::BlockDriver blockDrv(FW_OPTIONAL_NAME("blockDrv"));`

 - I think I'm either mis-handling the buffer, or mis-pointing to it. 

May 2 2022:
Tried to switch the UART. Worried that somehow I was using the "miniUART" in the [RPi4 documentation](https://www.raspberrypi.com/documentation/computers/configuration.html#configuring-uarts) ; per suggestion on [forum](https://forums.raspberrypi.com/viewtopic.php?t=244827) I went with UART5 (pins 12 and 13 per [datasheet](https://datasheets.raspberrypi.com/rpi4/raspberry-pi-4-datasheet.pdf)) by adding dtoverlay=uart5 to /boot/config.txt
 - with `cat /dev/ttyAMA1` see GPS data on the RPi, so know that works:
```
pi@raspberrypi:~ $ cat /dev/ttyAMA1
$GPGGA,050555.000,3517.1488,N,12039.4416,W,2,07,1.26,81.3,M,-30.7,M,0000,0000*67
$GPGSA,A,3,08,32,23,27,10,22,21,,,,,,1.55,1.26,0.91*01
$GPRMC,050555.000,A,3517.1488,N,12039.4416,W,0.29,283.89,030522,,,D*77
$GPVTG,283.89,T,,M,0.29,N,0.54,K,D*3A
$GPGGA,050556.000,3517.1488,N,12039.4417,W,2,07,1.26,81.3,M,-30.7,M,0000,0000*65
$GPGSA,A,3,08,32,23,27,10,22,21,,,,,,1.56,1.26,0.91*02
$GPGSV,3,1,12,32,85,205,25,22,62,210,26,10,55,042,20,27,38,231,14*79
$GPGSV,3,2,12,08,35,279,20,21,31,315,20,23,28,071,12,24,13,046,*77
$GPGSV,3,3,12,31,11,170,,18,10,132,,01,01,315,,40,,,*45
$GPRMC,050556.000,A,3517.1488,N,12039.4417,W,0.34,258.75,030522,,,D*7C
$GPVTG,258.75,T,,M,0.34,N,0.62,K,D*36
$GPGGA,050557.000,3517.1488,N,12039.4418,W,2,07,1.26,81.3,M,-30.7,M,0000,0000*6B
$GPGSA,A,3,08,32,23,27,10,22,21,,,,,,1.56,1.26,0.91*02
$GPRMC,050557.000,A,3517.1488,N,12039.4418,W,0.21,304.92,030522,,,D*77
$GPVTG,304.92,T,,M,0.21,N,0.39,K,D*3D
```
 - changed my boot command to `sudo ./GpsApp -a <IP address> -p 50000 -d /dev/ttyAMA1
 - noticed that the buffer size reported in telemetry was now larer than 1 on most samples; as high as 16 
 - set min buffersize for partsing to 4; 
 - disabled my fake/debug telemetry that I was routing in place of the GPS data so that the fake data didn't over-write the GPS data
 - eventually see lattitude at 0 and longitude at -1; not right, but at least indicative of the parser running
 - ran `cat /dev/ttyAMA1` on the RPi while running GpsApp and don't see "choppy" telemetry any more, at least not nearly as bad
 - so maybe it's just the parser that's off now?

May 3 2022:
 - slowed the rate group for Gps component to 1 Hz in Main.cpp; consistently see 16 packets in the buffer. 
 - also seeing the sscanf return 0 at 1 Hz; which implies that each pass through sscanf is failing. Looked at the [$GPGGA NEMA](http://aprs.gids.nl/nmea/#gga) string, and I was not parsing this correectly. Looks like there needs to be a case on the first line on the sscanf for the type of string.   
```
$GPGGA,043638.000,3517.1447,N,12039.4364,W,2,07,1.19,65.8,M,-30.7,M,0000,0000*61
```

May 4 2022:
 - think the error is with parsing now that the UART has been switched
 - wrote a stand-alone sscanf parser by copying the sscanf code from the Gps component in serialRecv_handler and passed the stand-alone parser the expected NEMA string from `cat /dev/ttyAMA1` on the RPi; the parser worked.
 - cleaned up some of the DEBUG, tried to just get the data out from within the parsing in serialRecv_handler within Gps component
 - added a "raw" serial dump; copied copde from RPi UART handler, just printed out to terminal near top pf serialRecv_handler:
```
    // DEBUG per RPi demo
    // convert incoming data to string. If it is not printable, set character to '*'
    char uMsg[serBuffer.getSize()+1];
    char* bPtr = reinterpret_cast<char*>(serBuffer.getData());
    for (NATIVE_UINT_TYPE byte = 0; byte < serBuffer.getSize(); byte++) {
        uMsg[byte] = isalpha(bPtr[byte])?bPtr[byte]:'*';
    }
    uMsg[sizeof(uMsg)-1] = 0;
    Fw::Logger::logMsg("Raw serial:\n");
    Fw::Logger::logMsg(uMsg);
```
 - Seems to work; see results on RPi ssh console:
```
Raw serial:
*GPGGA**********Raw serial:
************N***Raw serial:
*********W******Raw serial:
**********M*****Raw serial:
**M*************Raw serial:
***GPGSA*A******Raw serial:
****************Raw serial:
****************Raw serial:
*********GPRMC**Raw serial:
**********A*****Raw serial:
******N*********Raw serial:
***W************Raw serial:
**********D*****Raw serial:
*GPVTG********T*Raw serial:
*M******N******KRaw serial:
```
 - 16 characters per dump; it looks like the 16 count isn't the number of buffers used, it's the number of bytes in the buffer. Also looks like the whole string is there, just chopped up:
```
GPGGA**********************N************W****************M*******M****************
GPGGA,050556.000,3517.1488,N,12039.4417,W,2,07,1.26,81.3,M,-30.7,M,0000,0000*65
```
- looks like I need to glue the buffers back together

May 5 2022:
 - made an ugly mess of:
  - converted each buffer into a string
  - concatenated each new string into a big string
  - when big string was long enough, looked for GPGGA start
  - parsed after that
  - dumped the big new string
- seems to work:
```
sudo ./GpsApp -a 192.168.86.153 -p 50000 -d /dev/ttyAMA1
Hit Ctrl-C to quit
[INFO] Opened GPS UART driver: /dev/ttyAMA1
EVENT: (1280) (2:1651812780,192618) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2100 registered to port 0 slot 0
EVENT: (1280) (2:1651812780,192753) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2101 registered to port 0 slot 1
EVENT: (1280) (2:1651812780,192865) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2102 registered to port 0 slot 2
EVENT: (1280) (2:1651812780,192918) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2200 registered to port 1 slot 3
EVENT: (1280) (2:1651812780,192963) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2201 registered to port 1 slot 4
EVENT: (1280) (2:1651812780,193006) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2202 registered to port 1 slot 5
EVENT: (1280) (2:1651812780,193107) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2300 registered to port 2 slot 6
EVENT: (1280) (2:1651812780,193212) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2301 registered to port 2 slot 7
EVENT: (1280) (2:1651812780,193319) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2302 registered to port 2 slot 8
EVENT: (1280) (2:1651812780,193425) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2400 registered to port 3 slot 9
EVENT: (1280) (2:1651812780,193529) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x2401 registered to port 3 slot 10
...
EVENT: (1280) (2:1651812780,196269) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x4b00 registered to port 18 slot 61
EVENT: (1280) (2:1651812780,196316) DIAGNOSTIC: (cmdDisp) OpCodeRegistered : Opcode 0x4b01 registered to port 18 slot 62
EVENT: (3335) (2:1651812780,196432) ACTIVITY_HI: (prmDb) PrmFileLoadComplete : Parameter file load completed. Read 1 records.
EVENT: (3328) (2:1651812780,196522) WARNING_LO: (prmDb) PrmIdNotFound : Parameter ID 0x2700 not found
EVENT: (3328) (2:1651812780,196581) WARNING_LO: (prmDb) PrmIdNotFound : Parameter ID 0x4700 not found
EVENT: (3328) (2:1651812780,196642) WARNING_LO: (prmDb) PrmIdNotFound : Parameter ID 0x4701 not found
EVENT: (3328) (2:1651812780,196695) WARNING_LO: (prmDb) PrmIdNotFound : Parameter ID 0x2600 not found
EVENT: (3328) (2:1651812780,196747) WARNING_LO: (prmDb) PrmIdNotFound : Parameter ID 0x2601 not found
[ERROR] Failed to send framed data: 1
[ERROR] Failed to send framed data: 1
[ERROR] Failed to send framed data: 1
[ERROR] Failed to send framed data: 1
[ERROR] Failed to send framed data: 1
[ERROR] Failed to send framed data: 1
[INFO] Preamble size: 20
EVENT: (512) (2:1651812780,199507) DIAGNOSTIC: (rateGroup1Comp) RateGroupStarted : Rate group started.
EVENT: (768) (2:1651812780,199681) DIAGNOSTIC: (rateGroup2Comp) RateGroupStarted : Rate group started.
EVENT: (1024) (2:1651812780,199857) DIAGNOSTIC: (rateGroup3Comp) RateGroupStarted : Rate group started.
Connected to 192.168.86.153:50000 as a tcp client
EVENT: (3840) (2:1651812782,427612) ACTIVITY_HI: (gps) GPS_LOCK_ACQUIRED : GPS lock acquired
```
- see data in channels

May 9 2022:
 - Cleanup data; tried to cleanup comments (that seems to have worked)
 - Added parsers for the other NEMA sentences in my serialIn handler; I don't have the telemetry setup in the .fpp file yet, so don't know if that works yet
 - Played with the number of characters to wait on to start the parsing; 512 seems to work to be sure to get one GPGGA string
 - Haven't figured out a way to truncate the decimal places on output
 - Need to convert time to something useful

May 10 2022:
 - Added some of the new sentence data in GpsPacket to TLM; that seems to have worked. Needed to update the Gps.fpp and then the Gps.cpp
 - Edited the name of some of the TLM; prefer the instances to have all caps so TLM and CMD are INSTANCE.TLM or INSTANCE.CMD. Also edited the instance name of the Gps (component) to GPS (instance) from gps. So rather than gps.GPS_LOCK, TLM is now GPS.LOCK (etc.). Seems to have worked
 - Looked at the [commands to the Gps device](https://learn.adafruit.com/adafruit-ultimate-gps-featherwing/resources); looks like it's just a string on the UART

May 11 2022:
 - added command to Gps component; just one to set baud rate. updated Gps.fpp to point at cmdTypes.fdd in the same folder, with the enum for the command
 - added (by hand) the command handler in the Gps.hpp and Gps.cpp, though the only thing the .cpp does is acknoledge the command
 - Working to add the command output

May 12 2022:
 - Sorta added a command to Gps component; figured I could see a buad rate change, and once I got the code working found out that was true; sending the command to change from 9600 baud would cause a segfault some moments later. I could also see with `cat /dev/ttyAMA1` that the baud rate had changed. The problem appears to be that I didn't also change the baud rate on the LinuxSerialDriver component at the same time. It also isn't clear that  Ican change the baud rate on the driver after I've stared it. And as a bonus, the baud rate is sticky on the device, so I have to pull the battery out to reset it
 - Here is [the reference](https://cdn-shop.adafruit.com/datasheets/PMTK_A11.pdf) on the commands
 - Here is a [handy reference](https://nmeachecksum.eqth.net/) on checksum calcs

May 13 2022:
 - Commented out the baud rate command in .fpp, .hpp., and .cpp. Don't see it in the command list of the GDS any more
 - Added a COLD_START command; I thought this would only work on reset, but once GPS lock was acquired sending the command caused GPS lock to dorp. So command works, but not as expected. Likewise, the  report lock status command doesn't work as expected, since the lock lost is predicated on having TLM working.
 - Also noticed that with the `top` command, the GpsApp does show up as GpsApp under root. Wil `ps -elf` it's the command line w/ opptions,
 - Takes a long time (30+ minutes) for GPS to come up from cold

May 22 2022:

Still getting segfault with the "from scratch" version; going to try starting with working math-tutorial version and:
 - revert to block driver only timing
 - remove math component parts
 - remove Ref parts

Branched math-tutorial to gps_strip_down. 
 - Confirmed that the existing math-tutorial version works.
 - Commented out linuxTimer in files; regenerated, rebuilt, and retested. Looks like it works.
 - Commented out/removed Math Tutorial stuff; regenerated, rebuilt, and retested. Looks like it works.
 - Commented out/removed Signal Gen stuff; regenerated, rebuilt, and retested. Looks like it works.
 - Commented out/removed PingReceiver stuff; regenerated, rebuilt, and retested. Looks like it works.
 - Commented out/removed RecvBuffApp and SendBuffApp stuff; regenerated, rebuilt, and retested. Looks like it works.
 - Cleaned out commented stuff; regenerated, rebuilt, and retested. Looks like it works.
 - Worked through to clean up comments and style; changed case to GPS and GPS_SERIAL
 - Copied all the modified files (basically everything in /Top) over to Test-Gps-Demo files; regenerated, rebuilt, and retested. Seg fault:
```
[ERROR] Failed to send framed data: 1
EVENT: (512) (2:1653272138,641969) DIAGNOSTIC: (rateGroup1Comp) RateGroupStarted : Rate group started.
EVENT: (768) (2:1653272138,642111) DIAGNOSTIC: (rateGroup2Comp) RateGroupStarted : Rate group started.
EVENT: (1024) (2:1653272138,642256) DIAGNOSTIC: (rateGroup3Comp) RateGroupStarted : Rate group started.
Assert file "/home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-raspberrypi/F-Prime/Drv/Udp/ByteStreamPollPo
FATAL 16896 handled.
[ERROR] Failed to send framed data: 1
Connected to 192.168.86.153:50000 as a tcp client
Exiting with segfault and core dump file.
Segmentation fault
```

Tried the opposite way; copied all the /Gps files from Gps-Demo-Test into the gps_strip_down branch, with the /Top files from above, to see if there's something w/ the F' files between the branches. Regenerated, rebuilt, and retested. So that works. Need to figure out what version of fprime/devel I am using on each branch




## Lessons Learned
 - Don't copy over the other components; add them to `/GpsApp/CMakeLists.txt` instead with `add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/MathReceiver")`
 - Scrub though all the /Ref stuff, looking in anything copied over for the /Ref
 - Seems like there should be a "here's how to start a project from scratch using the provided F' components" rather than copying & modding the tutorials


TODO
 - [Define the GPS Component Instances](https://github.com/nasa/fprime/blob/master/docs/Tutorials/MathComponent/Tutorial.md#61-defining-the-component-instances)
 - [Update the Topology](https://github.com/nasa/fprime/blob/master/docs/Tutorials/MathComponent/Tutorial.md#62-updating-the-topology)

Noted that the existing [GPS tutorial points](https://github.com/nasa/fprime/blob/devel/docs/Tutorials/GpsTutorial/Tutorial.md#build-the-topology-sources) at a 404 refernce; it looks like the reference is [here](https://github.com/LeStarch/fprime/tree/gps-application/GpsApp). 
 

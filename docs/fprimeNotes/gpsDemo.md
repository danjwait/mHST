# GPS demo

Runing with documentation [here](https://github.com/nasa/fprime/blob/devel/docs/Tutorials/GpsTutorial/Tutorial.md). 

Created the dir GpsApp/Gps

Rather than GpsComponentAi.xml per tutorial, went back to Math demo and followed that process; starting at [The MathSender Component](https://github.com/nasa/fprime/blob/devel/docs/Tutorials/MathComponent/Tutorial.md#The-MathSender-Component).

Build the fprime/GpsApp/Gps/Gps.fpp file:
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

Copied the Top file over per [Clone the Ref Application](https://github.com/nasa/fprime/blob/devel/docs/Tutorials/GpsTutorial/Tutorial.md#clone-the-ref-application) in GPS tutorial. Didn't have the files to remove.

Copied the CMakeLists.txt file over from Ref `fprime/GpsApp$ cp ../Ref/CMakeLists.txt .` which included other components in Ref & Math demo; copied those over as well:
```
~/02_Projects/fprime/GpsApp$ cp ../Ref/CMakeLists.txt .
~/02_Projects/fprime/GpsApp$ cp -r ../Ref/MathPorts/ .
~/02_Projects/fprime/GpsApp$ cp -r ../Ref/MathReceiver/ .
~/02_Projects/fprime/GpsApp$ cp -r ../Ref/MathSender/ .
~/02_Projects/fprime/GpsApp$ cp -r ../Ref/MathTypes/ .
...
```

### Lesson Learned
 - Don't copy over the other components; comment them out in the `CMakeLists.txt` instead

Added Gps component to /GpsApp/CMakeLists.txt: `add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/Gps")`
  
Removed the previous build dir `rm -r ./build-fprime-automatic-native/` and re-ran `fprime-util generate` but get error:
```
-- Configuring done
CMake Error at /home/djwait/02_Projects/fprime/cmake/target/build.cmake:75 (add_dependencies):
-- Generating done
The dependency target "Ref_Top" of target "Ref" does not exist.
Call Stack (most recent call first):
/home/djwait/02_Projects/fprime/cmake/target/build.cmake:107 (setup_build_module)
/home/djwait/02_Projects/fprime/cmake/target/build.cmake:90 (add_module_target)
/home/djwait/02_Projects/fprime/cmake/target/target.cmake:85 (add_deployment_target)
/home/djwait/02_Projects/fprime/cmake/target/target.cmake:103 (setup_deployment_target)
/home/djwait/02_Projects/fprime/cmake/module.cmake:25 (setup_all_deployment_targets)
/home/djwait/02_Projects/fprime/cmake/API.cmake:341 (generate_deployment)
CMakeLists.txt:57 (register_fprime_deployment)
```
  
Noticed that the copied Ref CMakeLists.txt had a line `project(Ref VERSION 1.0.0 LANGUAGES C CXX)` so changed Ref to GpsApp. Removed the previous build dir again, then re-ran:
  ```
  ~/02_Projects/fprime/GpsApp$ fprime-util generate
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/GpsApp/settings.ini
[INFO] Generating build directory at: /home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-native
[INFO] Using toolchain file None for platform default
  ...
  -- Adding Library: GpsApp_Top
-- Adding Deployment: GpsApp
-- Configuring done
-- Generating done
-- Build files have been written to: /home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-native
...
  Scanning dependencies of target GpsApp_MathReceiver
[ 35%] Building CXX object GpsApp/MathReceiver/CMakeFiles/GpsApp_MathReceiver.dir/MathReceiver.cpp.o
In file included from /home/djwait/02_Projects/fprime/GpsApp/MathReceiver/MathReceiver.cpp:14:
/home/djwait/02_Projects/fprime/Ref/MathReceiver/MathReceiver.hpp:16:10: fatal error: Ref/MathReceiver/MathReceiverComponentAc.hpp: No such file or directory
   16 | #include "Ref/MathReceiver/MathReceiverComponentAc.hpp"
      |          ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
compilation terminated.
make[3]: *** [GpsApp/MathReceiver/CMakeFiles/GpsApp_MathReceiver.dir/build.make:87: GpsApp/MathReceiver/CMakeFiles/GpsApp_MathReceiver.dir/MathReceiver.cpp.o] Error 1
make[2]: *** [CMakeFiles/Makefile2:8637: GpsApp/MathReceiver/CMakeFiles/GpsApp_MathReceiver.dir/all] Error 2
make[1]: *** [CMakeFiles/Makefile2:2289: CMakeFiles/GpsApp.dir/rule] Error 2
make: *** [Makefile:177: GpsApp] Error 2
[ERROR] CMake erred with return code 2
  ```
Looks like the copied files include references back to Ref; will need to clean that up (this is lesson leanred above)
  
Removed Math Ref folders, commented out the components in CMakeLists.txt and retried:
```
~/02_Projects/fprime/GpsApp$ fprime-util generate
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/GpsApp/settings.ini
[INFO] Generating build directory at: /home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-native
[INFO] Using toolchain file None for platform default
...
-- Adding Library: Utils_Types
-- Adding Library: GpsApp_Gps
-- Adding Library: GpsApp_Top
-- Adding Deployment: GpsApp
-- Configuring done
-- Generating done
```
Then:
```
~/02_Projects/fprime/GpsApp$ fprime-util build
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/GpsApp/settings.ini
Scanning dependencies of target Fw_Cfg
[  1%] Building CXX object F-Prime/Fw/Cfg/CMakeFiles/Fw_Cfg.dir/ConfigCheck.cpp.o
[  1%] Linking CXX static library ../../../lib/Linux/libFw_Cfg.a
[  1%] Built target Fw_Cfg
Scanning dependencies of target codegen
[  4%] Built target codegen
...
[ 96%] Linking CXX static library ../../../lib/Linux/libDrv_Udp.a
[ 96%] Built target Drv_Udp
[ 96%] Generating Ports_RateGroupsEnumAi.xml, Ports_StaticMemoryEnumAi.xml, RefTopologyAppAi.xml
fpp-to-xml
/home/djwait/02_Projects/fprime/GpsApp/Top/instances.fpp: 158.26
instance pingRcvr: Ref.PingReceiver base id 0x0A00 \
                       ^
error: undefined symbol PingReceiver
make[3]: *** [GpsApp/Top/CMakeFiles/GpsApp_Top.dir/build.make:188: GpsApp/Top/Ports_RateGroupsEnumAi.xml] Error 1
make[2]: *** [CMakeFiles/Makefile2:8028: GpsApp/Top/CMakeFiles/GpsApp_Top.dir/all] Error 2
make[1]: *** [CMakeFiles/Makefile2:2105: CMakeFiles/GpsApp.dir/rule] Error 2
make: *** [Makefile:177: GpsApp] Error 2
[ERROR] CMake erred with return code 2
```
When up to `fprime/GpsApp` and ran `fprime/GpsApp/Gps$ fprime-util generate` :
```
~/02_Projects/fprime/GpsApp$ fprime-util generate
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/GpsApp/settings.ini
[INFO] Generating build directory at: /home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-native
[INFO] Using toolchain file None for platform default
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
...
-- Adding Library: GpsApp_Gps
-- Adding Library: GpsApp_Top
-- Adding Deployment: GpsApp
-- Configuring done
-- Generating done
-- Build files have been written to: /home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-native
```
Then cd into GpsApp/Gds and run:
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
Noted that the Gps.cpp file I'd generated by `touch Gps.cpp` is still empty; removed it and mv the Imp files 

Started editing Gps.cpp per notes at [GpsApp/Gps/GpsComponentImpl.cpp (Sample)](https://github.com/nasa/fprime/blob/devel/docs/Tutorials/GpsTutorial/Tutorial.md#gpsappgpsgpscomponentimplcpp-sample)

When I got to [GpsApp/Gps/CMakeListst.txt (final)](GpsApp/Gps/CMakeListst.txt (final)) I tried using the CMakeLists.txt there but got an error w/ the ` fprime-util build` command, so re-wrote the CMakeLists.txt to:
```
# Register the standard build
set(SOURCE_FILES
	"${CMAKE_CURRENT_LIST_DIR}/Gps.hpp"
	"${CMAKE_CURRENT_LIST_DIR}/Gps.cpp"
)
register_fprime_module()
```

Then got another error:
```
Scanning dependencies of target GpsApp_Gps
Building CXX object GpsApp/Gps/CMakeFiles/GpsApp_Gps.dir/Gps.cpp.o
In file included from /home/djwait/02_Projects/fprime/GpsApp/Gps/Gps.cpp:14:
/home/djwait/02_Projects/fprime/GpsApp/Gps/Gps.hpp:16:10: fatal error: GpsApp/Gps/GpsComponentAc.hpp: No such file or directory
   16 | #include "GpsApp/Gps/GpsComponentAc.hpp"
      |          ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
compilation terminated.
make[3]: *** [GpsApp/Gps/CMakeFiles/GpsApp_Gps.dir/build.make:63: GpsApp/Gps/CMakeFiles/GpsApp_Gps.dir/Gps.cpp.o] Error 1
make[2]: *** [CMakeFiles/Makefile2:7863: GpsApp/Gps/CMakeFiles/GpsApp_Gps.dir/all] Error 2
make[1]: *** [CMakeFiles/Makefile2:7870: GpsApp/Gps/CMakeFiles/GpsApp_Gps.dir/rule] Error 2
make: *** [Makefile:2218: GpsApp_Gps] Error 2
[ERROR] CMake erred with return code 2
```
so edited Gps.hpp to be:
```
...
#ifndef Gps_HPP
#define Gps_HPP

#include "GpsApp/Gps/Gps.hpp"
...
```

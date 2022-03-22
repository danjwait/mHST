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
Looks like the copied files include references back to Ref; will need to clean that up (this is lesson learned above)
  
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
## lots of debugging

I made a change to the Gps.hpp file:
```
#ifndef Gps_HPP
#define Gps_HPP

//#include "GpsApp/Gps/Gps.hpp"
#include "GpsApp/Gps/GpsComponentAc.hpp"
```
as well as the GpsApp/Gps/CMakeLists.txt file to include the .fpp in place of the .hpp file:
```
# Register the standard build
set(SOURCE_FILES
	"${CMAKE_CURRENT_LIST_DIR}/Gps.cpp"
	"${CMAKE_CURRENT_LIST_DIR}/Gps.fpp"
)
register_fprime_module()
```
I then ran `/02_Projects/fprime/GpsApp/Gps$ fprime-util purge` and `fprime-util generate` and `fprime-util build` and see different errors:

Error in Gps.cpp at ` void Gps :: preamble()` with a type mismatch on the (U64) and the buffer; commented that out to see what the next error would be:
```
//this->m_recvBuffers[buffer].setData((U64)this->m_uartBuffers[buffer]);
```

At `Drv::SerialReadStatus &serial_status` I had missed the &, so fixed that. 

In ` void Gps ::serialRecv_handler` another error with the [renamed symbols](https://nasa.github.io/fprime/UsersGuide/dev/v3-renamed-symbols.html) ; `Drv::SER_OK` becomes `Drv::SerialReadStatus::SER_OK` 
    
Then another type error at `//Fw::Logger::logMsg("[WARNING] Received buffer with bad packet: %d\n", serial_status);` so commented that line out.

In step 4 generate telemetry getting other errors; this code:
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

With all those changes, it appears to work:
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

## Topology work
GPS tutorial says to copy the Ref Topology, make a couple edits, and build that. However the Ref topology includes components in Ref, which leads to errors:
```
~/02_Projects/fprime/GpsApp$ fprime-util build
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/GpsApp/settings.ini
Scanning dependencies of target Fw_Cfg
[  0%] Building CXX object F-Prime/Fw/Cfg/CMakeFiles/Fw_Cfg.dir/ConfigCheck.cpp.o
[  0%] Linking CXX static library ../../../lib/Linux/libFw_Cfg.a
...
[ 96%] Building CXX object F-Prime/Drv/Udp/CMakeFiles/Drv_Udp.dir/RecvStatusEnumAc.cpp.o
[ 96%] Building CXX object F-Prime/Drv/Udp/CMakeFiles/Drv_Udp.dir/SendStatusEnumAc.cpp.o
[ 96%] Linking CXX static library ../../../lib/Linux/libDrv_Udp.a
[ 96%] Built target Drv_Udp
[ 96%] Generating Ports_RateGroupsEnumAi.xml, Ports_StaticMemoryEnumAi.xml, RefTopologyAppAi.xml
fpp-to-xml
/home/djwait/02_Projects/fprime/GpsApp/Top/instances.fpp: 158.26
  instance pingRcvr: Ref.PingReceiver base id 0x0A00 \
                         ^
error: undefined symbol PingReceiver
make[3]: *** [GpsApp/Top/CMakeFiles/GpsApp_Top.dir/build.make:188: GpsApp/Top/Ports_RateGroupsEnumAi.xml] Error 1
make[2]: *** [CMakeFiles/Makefile2:7999: GpsApp/Top/CMakeFiles/GpsApp_Top.dir/all] Error 2
make[1]: *** [CMakeFiles/Makefile2:2105: CMakeFiles/GpsApp.dir/rule] Error 2
make: *** [Makefile:177: GpsApp] Error 2
[ERROR] CMake erred with return code 2
```
Tried adding line to CMakeLists.txt in /GpsApp/ :
```
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref")
```
but still see same error. Tried ` fprime-util purge` and `fprime-util generate` and `fprime-util build` ; didn't see anything like "Adding Library: Ref_" after generate, so wasn't sure it was going to work, and it didn't. Same issue as above.

Ended up copying the component subdirectories back into /GpsApp/ and added to /GpsApp/CMakeLists.txt:
```
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/PingReceiver/")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/RecvBuffApp/")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/SendBuffApp/")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/SignalGen/")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/MathTypes")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/MathPorts")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/MathSender")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/MathReceiver")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/Gps")
```
Re-ran ` fprime-util purge` and `fprime-util generate` and this time see:
```
...
-- Adding Library: Utils_Types
-- Adding Library: GpsApp_PingReceiver
-- Adding Library: GpsApp_RecvBuffApp
-- Adding Library: GpsApp_SendBuffApp
-- Adding Library: GpsApp_SignalGen
-- Adding Library: GpsApp_MathTypes
-- Adding Library: GpsApp_MathPorts
-- Adding Library: GpsApp_MathSender
-- Adding Library: GpsApp_MathReceiver
-- Adding Library: GpsApp_Gps
-- Adding Library: GpsApp_Top
-- Adding Deployment: GpsApp
-- Configuring done
-- Generating done
-- Build files have been written to: /home/djwait/02_Projects/fprime/GpsApp/build-fprime-automatic-native
```
which looks better, but get errors running `fprime-util build` with files that are still in /Ref/

## working version:
Modified /GpsApp/CMakeLists.txt again:
```
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/PingReceiver/")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/RecvBuffApp/")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/SendBuffApp/")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/SignalGen/")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/MathTypes")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/MathPorts")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/MathSender")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/MathReceiver")
add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/Gps")
```

Added Gps in /GpsApp/instances.fpp at the end of the active group:
```
  instance gps: GpsApp.Gps base id 0x0F00 \
    queue size Default.queueSize \
    stack size Default.stackSize \
    priority 100 \
  {

  }
 ```
left the ` { } ` because I wasn't sure if gps needed other configuration.

In same file added  the linux serial driver at the end of  `# Passive component instances` as:
```
  instance gpsSerial: Drv.LinuxSerialDriver base id 0x4C00 \
    at "../../Drv/LinuxSerialDriver/LinuxSerialDriver.hpp" \
  {


  }
```

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

Ended up going into `GpsApp/Top` and deleting everything there unitl all that was left was:
```
djwait@TRON:~/02_Projects/fprime/GpsApp/Top$ lrt
total 84
-rw-r--r-- 1 djwait djwait   427 Mar  8 20:38 CMakeLists.txt
-rw-r--r-- 1 djwait djwait    19 Mar  8 20:38 .gitignore
-rw-r--r-- 1 djwait djwait 45705 Mar 21 19:50 GpsTopologyAppAi.xml
-rw-r--r-- 1 djwait djwait  9952 Mar 21 21:31 instances.fpp
-rw-r--r-- 1 djwait djwait  4948 Mar 21 21:35 topology.fpp
drwxr-xr-x 5 djwait djwait  4096 Mar 21 21:35 ..
drwxr-xr-x 2 djwait djwait  4096 Mar 21 21:41 .
```
Looked at /GpsApp/Top/CMakeLists.txt and say a reference to a `RefTopologyDefs.cpp` file; copied that back into /GpsApp/Top and replace `Ref` with `GpsApp` in the file and in the /GpsApp/Top/CMakeLists.txt file.

Likewise copied RefTopologyAc.hpp and RefTopologyAc.cpp over from /Ref and replaced `Ref` with `GpsApp` within files and changed file names to match. Ended up doing that over and over until /GpsApp/Top had these files, copied from /Ref and edited & renamed to replace `Ref` with `GpsApp`:
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

Then build:
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

### Lessons Learned
 - Don't copy over the other components; add them to `/GpsApp/CMakeLists.txt` instead with `add_fprime_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../Ref/MathReceiver")`
 - Scrub though all the /Ref stuff, aything copied over


TODO
 - [Define the GPS Component Instances](https://github.com/nasa/fprime/blob/master/docs/Tutorials/MathComponent/Tutorial.md#61-defining-the-component-instances)
 - [Update the Topology](https://github.com/nasa/fprime/blob/master/docs/Tutorials/MathComponent/Tutorial.md#62-updating-the-topology)

Noted that the existing [GPS tutorial points](https://github.com/nasa/fprime/blob/devel/docs/Tutorials/GpsTutorial/Tutorial.md#build-the-topology-sources) at a 404 refernce; it looks like the reference is [here](https://github.com/LeStarch/fprime/tree/gps-application/GpsApp). 
 

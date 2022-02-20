# [F’ Math Component Tutorial](https://nasa.github.io/fprime/Tutorials/MathComponent/Tutorial.html)

Get an error at [4.5.1. Set Up the Unit Test Environment](https://nasa.github.io/fprime/Tutorials/MathComponent/Tutorial.html#The-MathSender-Component_Write-and-Run-Unit-Tests_Set-Up-the-Unit-Test-Environment):
```
djwait@TRON:~/02_Projects/fprime/Ref$ fprime-util generate --ut
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/Ref/settings.ini
[ERROR] /home/djwait/02_Projects/fprime/Ref/build-fprime-automatic-native-ut already exists.
djwait@TRON:~/02_Projects/fprime/Ref$ fprime-util impl --ut
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/Ref/settings.ini
make: *** No rule to make target 'Ref_testimpl'.  Stop.
[CMAKE] CMake failed to detect target, attempting CMake cache refresh and retry
make: *** No rule to make target 'Ref_testimpl'.  Stop.
[ERROR] CMake erred with return code 2
```
In discussion [here](https://github.com/nasa/fprime/discussions/1135#discussioncomment-2133443) pointed at [Fix Section 4.5.1 of Math Component Tutorial](https://github.com/nasa/fprime/pull/1243) ; had to look at the commit w/ the updated md, inlcuding finding related issue [fprime-util impl --ut command not generating Tester.cpp and Tester.hpp in Ref/MathSender](https://github.com/nasa/fprime/issues/1238) which included a note about removeing some files [here](https://github.com/nasa/fprime/issues/1238#issuecomment-1025790962). 
Finally ran into [Fatal error with fprime-util build --ut](https://github.com/nasa/fprime/issues/1258); wasn't obvious to me that the write up was to open Tester.hpp and change two instances of MathSenderComponentImpl to MathSender (had to go look at a related GitHub account [here](https://github.com/capsulecorplab/fprime/commit/5a815b4d6766edf7db385354688149462790c87d))


I find myself wanting a checklist / process flow diagram / overview. I am worried that after running the tutorials, I'll have to go back to the tutorials to remember the steps in order. Maybe the current checklist / flow diagram / overview can be expanded in the tutorials, with thinks like "fill in stubs" demoed?


Error at [5.2. Add the Model to the Project](https://nasa.github.io/fprime/Tutorials/MathComponent/Tutorial.html#The-MathReceiver-Component_Add-the-Model-to-the-Project). When trying to re-run [4.3. Build the Stub Implementation](https://nasa.github.io/fprime/Tutorials/MathComponent/Tutorial.html#The-MathSender-Component_Build-the-Stub-Implementation) got an error with `fprime-util impl` had to go up to /Ref/ and run `fprime-util generate` and then `fprime-util build` in /Ref, then cd back into MathReceiver to get `fprime-util impl` to work.

Error at [6.2. Updating the Topology](https://nasa.github.io/fprime/Tutorials/MathComponent/Tutorial.html#Updating-the-Ref-Deployment_Updating-the-Topology); first ran into this error:
```
~/02_Projects/fprime/Ref/Top$ fprime-util fpp-check -u unconnected.txt
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/Ref/settings.ini
[INFO] Updating fpp locations file and build cache. This may take some time.
Traceback (most recent call last):
  File "/home/djwait/.local/bin/fprime-util", line 8, in <module>
    sys.exit(main())
  File "/home/djwait/.local/lib/python3.8/site-packages/fprime/util/__main__.py", line 14, in main
    return fprime.util.build_helper.utility_entry(args=sys.argv[1:])
  File "/home/djwait/.local/lib/python3.8/site-packages/fprime/util/build_helper.py", line 164, in utility_entry
    runners[parsed.command](build, parsed, cmake_args, make_args)
  File "/home/djwait/.local/lib/python3.8/site-packages/fprime/fpp/cli.py", line 71, in run_fpp_check
    run_fpp_util(
  File "/home/djwait/.local/lib/python3.8/site-packages/fprime/fpp/common.py", line 109, in run_fpp_util
    return subprocess.run(app_args, capture_output=False)
  File "/usr/lib/python3.8/subprocess.py", line 493, in run
    with Popen(*popenargs, **kwargs) as process:
  File "/usr/lib/python3.8/subprocess.py", line 858, in __init__
    self._execute_child(args, executable, preexec_fn, close_fds,
  File "/usr/lib/python3.8/subprocess.py", line 1704, in _execute_child
    raise child_exception_type(errno_num, err_msg, err_filename)
FileNotFoundError: [Errno 2] No such file or directory: 'fpp-check'
```
Which looked like part of [fpp-check Resulting in Error Message #1255](https://github.com/nasa/fprime/issues/1255) ; per LeStarch suggestion, added `Ref/build-fprime-automatic-native/fpp-tools-install/` to $PATH. 

# Cross Compile Testing; Porting Math Demo to RPi
Started with the "Cross Compiling for the Raspberry PI" section of [F Fprime GPS demo instructions](https://nasa.github.io/fprime/Tutorials/GpsTutorial/Tutorial.html) but intent is to just cross compile the Math Component to the RPi first.

Tested the Math Component (per above) on WSL2 linux "native" first, then edited to cross compile.

Using toolchains per [raspberrypi / tools](https://github.com/raspberrypi/tools):
```
sudo apt-get install gcc-arm-linux-gnueabihf
sudo apt-get install g++-arm-linux-gnueabihf
sudo apt-get install gcc-aarch64-linux-gnu
sudo apt-get install g++-aarch64-linux-gnu
```
Installed the former pair in case using a 32-bit Pi; installed the latter pair since RPi4 is 64

My installations ended up at:
- /bin/aarch64-linux-gnu-gcc
- /bin/arm-linux-gnueabihf-gcc

## Setup RPi4
Usual sudo apt-get update & upgrade
Followed RPi directions [here](https://www.raspberrypi.com/documentation/computers/remote-access.html) to setup SSH from host to RPi
Confirmed SSH ability from host to RPi4

## Cross-compile testing:
Tried this command: `fprime-util generate raspberrypi` in Ref/ and got error:
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

Tried setting up the complier by editing `/home/djwait/02_Projects/fprime/cmake/toolchain/raspberrypi.cmake` with line: `set(RPI_TOOLCHAIN "/bin/aarch64-linux-gnu-gcc")` and re-ran but ran into error with exisiting directory, so deleted the existing directory:
```
~/02_Projects/fprime/Ref$ fprime-util generate raspberrypi
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/Ref/settings.ini
[ERROR] /home/djwait/02_Projects/fprime/Ref/build-fprime-automatic-raspberrypi already exists.
djwait@TRON:~/02_Projects/fprime/Ref$ rm -r -f ./build-fprime-automatic-raspberrypi/
```

Re-ran and different error:
```
~/02_Projects/fprime/Ref$ fprime-util generate raspberrypi
[WARNING] Failed to find settings file: /home/djwait/02_Projects/fprime/Ref/settings.ini
[INFO] Generating build directory at: /home/djwait/02_Projects/fprime/Ref/build-fprime-automatic-raspberrypi
[INFO] Using toolchain file /home/djwait/02_Projects/fprime/cmake/toolchain/raspberrypi.cmake for platform raspberrypi
-- Using RPI toolchain at: /bin/aarch64-linux-gnu-gcc
-- Using RPI toolchain at: /bin/aarch64-linux-gnu-gcc
-- The C compiler identification is unknown
-- The CXX compiler identification is unknown
CMake Error at CMakeLists.txt:23 (project):
  The CMAKE_C_COMPILER:

    /bin/aarch64-linux-gnu-gcc/bin/arm-linux-gnueabihf-gcc

  is not a full path to an existing compiler tool.

  Tell CMake where to find the compiler by setting either the environment
  variable "CC" or the CMake cache entry CMAKE_C_COMPILER to the full path to
  the compiler, or to the compiler name if it is in the PATH.


CMake Error at CMakeLists.txt:23 (project):
-- Configuring incomplete, errors occurred!
  The CMAKE_CXX_COMPILER:
See also "/home/djwait/02_Projects/fprime/Ref/build-fprime-automatic-raspberrypi/CMakeFiles/CMakeOutput.log".

See also "/home/djwait/02_Projects/fprime/Ref/build-fprime-automatic-raspberrypi/CMakeFiles/CMakeError.log".
    /bin/aarch64-linux-gnu-gcc/bin/arm-linux-gnueabihf-g++

  is not a full path to an existing compiler tool.

  Tell CMake where to find the compiler by setting either the environment
  variable "CXX" or the CMake cache entry CMAKE_CXX_COMPILER to the full path
  to the compiler, or to the compiler name if it is in the PATH.


[ERROR] CMake erred with return code 1. Partial build cache remains. Run purge to clean-up.
```
Deleted directory and edited cmake/toolchain/raspberrypi.cmake to:
```
# Location of pi toolchain
set(RPI_TOOLCHAIN "$ENV{RPI_TOOLCHAIN_DIR}")
# set(RPI_TOOLCHAIN "/bin/aarch64-linux-gnu-gcc")
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
#set(CMAKE_C_COMPILER "${RPI_TOOLCHAIN}/bin/arm-linux-gnueabihf-gcc")
#set(CMAKE_CXX_COMPILER "${RPI_TOOLCHAIN}/bin/arm-linux-gnueabihf-g++")
set(CMAKE_C_COMPILER "${RPI_TOOLCHAIN}/aarch64-linux-gnu-gcc")
set(CMAKE_CXX_COMPILER "${RPI_TOOLCHAIN}/aarch64-linux-gnu-g++")
```
Then ran `/02_Projects/fprime/Ref$ fprime-util generate raspberrypi` and see:
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
-- Installing: /home/djwait/02_Projects/fprime/Ref/build-artifacts/raspberrypi/lib/static/libRef_Top.a
-- Installing: /home/djwait/02_Projects/fprime/Ref/build-artifacts/raspberrypi/dict/RefTopologyAppDictionary.xml
[100%] Built target Ref
```

Find bin file:
```
/02_Projects/fprime/Ref/build-artifacts/raspberrypi/bin$ ls -lrt
total 1388
-rwxr-xr-x 1 djwait djwait 1420648 Feb 19 17:55 Ref
```
scp over to RPi
```
~/02_Projects/fprime/Ref/build-artifacts/raspberrypi/bin$ scp -r Ref pi@<pi IP>:/home/pi
```
see Ref on RPi
```
pi@raspberrypi:~ $ pwd
/home/pi
pi@raspberrypi:~ $ ls -lrt
total 1424
drwxr-xr-x 2 pi pi    4096 May  7  2021 Bookshelf
drwxr-xr-x 2 pi pi    4096 May  7  2021 Desktop
drwxr-xr-x 2 pi pi    4096 May  7  2021 Videos
drwxr-xr-x 2 pi pi    4096 May  7  2021 Templates
drwxr-xr-x 2 pi pi    4096 May  7  2021 Public
drwxr-xr-x 2 pi pi    4096 May  7  2021 Pictures
drwxr-xr-x 2 pi pi    4096 May  7  2021 Music
drwxr-xr-x 2 pi pi    4096 May  7  2021 Downloads
drwxr-xr-x 2 pi pi    4096 May  7  2021 Documents
-rwxr-xr-x 1 pi pi 1420648 Feb 19 18:01 Ref
```




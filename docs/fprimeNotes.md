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

# Porting Math Demo to RPi
Started with [RPi demo instructions](https://nasa.github.io/fprime/Tutorials/GpsTutorial/Tutorial.html) but intent is to just cross compile the Math Component to the RPi first.

Using toolchains per [raspberrypi / tools](https://github.com/raspberrypi/tools):
`sudo apt-get install gcc-arm-linux-gnueabihf` 
`sudo apt-get install gcc-aarch64-linux-gnu`
Using the latter since RPi4 is 64

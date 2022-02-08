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

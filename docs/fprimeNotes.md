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

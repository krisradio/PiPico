# R Pi Pico
## SDK Toolchain Setup
Required tools:  
1. [ARM GCC Compiler](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads); version 10.3-2021.10 selected; added registry information, launched gccvar.bat, added path to env var, and added registry info.
    - Destination folder: `C:\Program Files (x86)\GNU Arm Embedded Toolchain\10 2021.10`.  
    - Readme Location: `C:\Program Files (x86)\GNU Arm Embedded Toolchain\10 2021.10\share\doc\gcc-arm-none-eabi\readme.txt`.

2. [CMake](https://cmake.org/download/); version 3.22.1 selected; added CMake to system path for all users;  
    - Destination folder: `C:\Program Files\CMake\`.  
    - Readme Location: `C:\Program Files\CMake\share\cmake-3.22\Help\index.rst`.

3. Visual Studio 2022 updated to [version 17.0.4](https://docs.microsoft.com/en-us/visualstudio/releases/2022/release-notes#17.0.4) and "C++ Build Tools core features" installed. From the Getting Started guide, the C++ build tools workload (which I could not find a description for on MS' docs) only has the Core Features and Redistributable components included. Not needing the latter update, I only did the C++ core features.

4. Python upgraded to [3.10.1](https://www.python.org/downloads/) and added to the system PATH; unable to "Install For All Users" because the option was greyed out. Disabled path character length limit.

5. Git is already installed on my system. I believe my installation commits UNIX-style, and checks out Windows-style. The Pico documentation recommends leaving both "as-is", which is the first time I as a Windows user have been advised that. Since I have projects that have been committed UNIX-style, I will not be modifying this setting.

6. I then installed C++ CMake tools for Windows *and* for Linux, which included MSVC v143 - VS 2022 C++ x86/x64 build tools (latest), C++ for Linux development, and C++ CMake tools for Linux. 

## Getting the SDK and examples
I had an interesting warning when I launched VS 2022 and opened a new "Hello World!" project - "You're using Python 0.0". I don't think so, MicroSoft, but we'll keep an eye on it. 

As part of getting the SDK, you update the submodules. I have never used [Git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) before, so this is an exciting development.

The Dev Cmd Prompt is opened from "Tools > Command Line > Developer Command Prompt". There are extensions to add a shortcut for this, but I won't install those for the time being. I set the path to the SDK as follows:
```
setx PICO_SDK_PATH "Toolchain\pico-sdk"
SUCCESS: Specified value was saved.
```

In trying to run the hello_world example, I realized the above SDK path is incorrect. I dropped the parent directory double-dots from the location. I re-ran the command with the absolute path to the SDK. I will have to redo this when I reorganize my files after graduation.

The next problem was **nmake** not being found. I re-ran VS Installer and installed C++ CMake tools, as detailed in #6 of SDK Toolchain Setup. Then I restarted my computer (for the first time in the installation process, heheh...).


## Building Hello World 
Running the example build again worked this time! I also realized I left off a double-dot from the cmake command. Not sure what that argument is for, but it made a big (positive) difference in the output. 

However, running nmake still failed. The complete output is shown in "hello-world-build-err.md". nmake is run on line 20.

I asked a question on Stack Overflow. In the end I didn't get any direct answers, but did find some people asking [similar questions](https://stackoverflow.com/questions/14319247/cmake-is-unable-to-configure-project-for-visual-studios-10-amd64/14471934#14471934) with [more helpful answers](https://stackoverflow.com/questions/14319247/cmake-is-unable-to-configure-project-for-visual-studios-10-amd64/14471934#14471934). First I tried setting "devenv.exe" and "cl.exe" to run as administrator, which didn't work. Most of these answers were for targetting traditional platforms, so next I searched "arm m0 compile tools visual studio" which led to a [helpful blog post](https://devblogs.microsoft.com/cppblog/arm-gcc-cross-compilation-in-visual-studio/). I already had the Linux C++ workload installed, but not the Embedded and IoT development tools. 

This STILL didn't work with the Hello World example project, so I tried creating a new Pi project based on a VS template. I was told I didn't have the Linux C++ development tools. I definitely do, so there must have been some sub-component I didn't include. 

> At this point I finally noticed that I had missed a key note, "You must install the full "Windows 10 SDK" package as the SDK will need to build the pioasm and elf2uf2 tools locally.  
> Removing it from the list of installed items will mean that you will be unable to build Raspberry Pi Pico binaries."

Installing this next, hopefully following the instructions does it for me...

Installing that SDK did get me past the last issue, but I ran into a mysterious known bug associated with nmake: raspberrypi/pico-examples#152, raspberrypi/pico-examples#153. This bug is stumping the Pico developers and there is no known solution with nmake, but ninja and mingw don't cause the issue. So at this point I need to do some research and decide whether I want to go with ninja or mingw; additionally, there's the option to [use VSCode instead of Visual Studio](https://shawnhymel.com/2096/how-to-set-up-raspberry-pi-pico-c-c-toolchain-on-windows-with-vs-code/#Build_Blink_Example), which is interesting to me because it is lighter weight.

Researching different build tools, I learned how much I have to learn. Here are some resources on MinGW-w64:
- [Homepage](https://www.mingw-w64.org/)
- [Wikipedia article](https://en.wikipedia.org/wiki/Mingw-w64)

And some on Ninja:
- [Homepage](https://ninja-build.org/)
- [Wikipedia article](https://en.wikipedia.org/wiki/Ninja_%28build_system%29)

What I've learned is that these are very different tools. 

Looking at a project's neighbors can be really helpful, too. Are they tools you already use, or would like to? What I found looking at MinGW's neighbors (on the homepage) is many tools I'm not familiar with, and some I consider to be rather clunky. MinGW also wears many hats, describing itself as "a complete runtime environment for GCC & LLVM". [LLVM](https://en.wikipedia.org/wiki/LLVM) is a really cool technology for cross-referencing any language with any ISA. However, I am working on embedded development for a very specific hardware platform. I imagine LLVM could be used to cross-compile to other platforms, but there could be hardware incompatibilities, and frankly there is no pressing need. 

The list of projects using Ninja ([here](https://github.com/ninja-build/ninja/wiki/List-of-generators-producing-ninja-build-files)) is much shorter, but seems more focused on simply generating builds. This is fitting with the project's mission. Ninja has far more activity on Github, which is a site I'm active on. Ninja is also supposed to be very easy to install - simply download, add to the PATH, and go. For these reasons, I'm selecting Ninja to replace nmake. There's also a [great article on lwn.net](https://lwn.net/Articles/706404/) detailing some benefits of Ninja firsthand that is pretty reassuring. 

It's urgent that I get this working, because not only has the pico-examples project failed to build, the built in MS Visual template failed to build as well!
# Development Environment

> *Build environment heavily informed by Shawn Hymel's [Raspberry Pi Pico C/C++ Toolchain tutorial](https://www.digikey.com/en/maker/projects/raspberry-pi-pico-and-rp2040-cc-part-1-blink-and-vs-code/7102fb8bca95452e9df6150f39ae8422) and [OpenOCD and Picotool for Raspberry Pi Pico tutorial](https://shawnhymel.com/2168/how-to-build-openocd-and-picotool-for-the-raspberry-pi-pico-on-windows/), as well as his [video tutorials](https://youtu.be/B5rQSoOmR5w) made for Digi-Key and [Raspberry Pi's Getting started with Raspberry Pi Pico guide.](https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf)*
> 
> *The main differences are that this guide implements 100% open-source tools (including the IDE and extensions, with the only exceptions being the development OS and the Raspberry Pi silicon), and it is more current (February 2024).*


## Before you begin

- You will need 2 Raspberry Pi Picos *(at least one must be the original Pico, not the Pico W)* and means of connecting them, as shown [here](https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf#picoprobe-wiring-section)  
*(You can get by with just one, but will lose hardware debugging capabilities)*
- This guide was written for development on Windows 10, and may or may not apply to other versions of Windows.



## Overview

The build environment consists of:

- Raspberry Pi Pico SDK
- Arm GNU Toolchain
- GCC (MinGW-w64)
- CMake
- Python

Debugging consists of:

- picoprobe
- OpenOCD (snapshot build / >0.12.0)
- GDB (via Arm GNU Toolchain, arm-none-eabi-gdb)
- picotool
  - libusb

Development environment builds off of:

- VSCodium
  - CMake Tools extension
    - CMake language extension
  - clangd language extension
  - Cortex-Debug extension
    - debug-tracker-vscode extension
    - MemoryView extension
    - Peripheral Viewer extension
    - RTOS Views extension

And this guide will utilize:

- Git for Windows
- A serial monitor (Tabby as an example)
- Zadig
- 7Zip



## Detailed environment setup

> 🐉 **HERE BE DRAGONS: This guide is kind of a brain dump right now, and not particularly well-formatted.**  
> You should certainly be able to get to where you need to be by following it, but the path may be treacherous along the way

### ***We'll begin with build tools***

**1.1. Using [7-Zip](https://7-zip.org/), extract the 32-bit version of [MinGW-w64](https://github.com/niXman/mingw-builds-binaries/releases/tag/13.2.0-rt_v11-rev1)** to a directory without spaces *(such as `C:\mingw32\`)*.  

*This project was built using using the archive:*  
*`i686-13.2.0-release-mcf-dwarf-ucrt-rt_v11-rev1.7z`*  
*SHA256 checksum: `30911A14D3BA143C524C99209B194D47793AE29383A201B9C240F30DE939E5FA`*

&nbsp;



**1.2. If you don't already** have a path containing `make.exe` in your system path *(you can check by entering `make -v` in the command prompt)*, feel free to **create a symbolic link to redirect the command `make` to `mingw32-make.exe`**, as this can simply make calling the command easier during development.  To create this symlink, open a command prompt *(you may need to run it as an administrator)* and enter:  

  ```
  cd C:/mingw32/bin
  mklink make.exe mingw32-make.exe
  ```

&nbsp;



**1.3. Create the following directory structure:**  
`C:\VSARM\armcc`  
`C:\VSARM\sdk\pico`  
*You're welcome to use a different structure, but make sure you replace all paths appropriately through this guide if you do.*  

&nbsp;



**1.4. Install [Arm GNU Toolchain](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)** targeting AArch32 bare-metal to a directory without spaces *(such as `C:\VSARM\armcc\13.2.Rel1\`)*, and ensure boxes are checked when installation is done so that the folder is added to the system path.  

*This project was built using the installer:*  
*`arm-gnu-toolchain-13.2.rel1-mingw-w64-i686-arm-none-eabi.exe`*  
*SHA256 checksum: `CEC1B12F1E4EA21A4F62B52BB2CB97A9781AB49E927EC5E2A48CC699D196ED62`*

&nbsp;



**1.5. Install [CMake](https://github.com/Kitware/CMake/releases/tag/v3.28.1)**, ensuring that you "Add CMake to the system PATH" during installation.  

*This project was built using the installer:*  
*`cmake-3.28.1-windows-x86_64.msi`*  
*SHA256 checksum: `05F46EF9DD9F8C274D92AABDB0006DC834363F393559FFDE1D68D362CB0FC858`*

&nbsp;



**1.6. Install [Python](https://www.python.org/downloads/release/python-3122/)**, ensuring boxes are checked so python.exe is added to PATH, and at the end of installation be sure to select "Disable path length limit".  

*This project was built using the installer:*  
*`python-3.12.1-amd64.exe`*  
*SHA256 checksum: `2437D83DB04FB272AF8DE65EEAD1A2FC416B9FAC3F6AF9CE51A627E32B4FE8F8`*

&nbsp;



**1.7. Using [Git for Windows](https://gitforwindows.org/), download the Raspberry Pi Pico SDK and related repositories.**  
*If you really don't want to use Git, you can download the needed files and their submodules manually from each of their repositories.*

In Git Bash enter:  

  ```
  cd C:/VSARM/sdk/pico
  git clone https://github.com/raspberrypi/pico-sdk.git --branch master
  cd pico-sdk
  git submodule update --init
  git checkout 1.5.1 --recurse-submodules --force
  cd ..
  git clone https://github.com/raspberrypi/pico-examples.git --branch sdk-1.5.1
  git clone https://github.com/raspberrypi/picotool.git --branch 1.1.2
  git clone https://github.com/raspberrypi/picoprobe.git --branch picoprobe-cmsis-v1.1 --depth 1
  cd picoprobe
  git submodule update --init
  ```

*Note that picoprobe's submodules will require your Git is installed with git-lfs (a default setting in Git for Windows).*

*You can certainly use `--branch master` and avoid `git checkout` for any of these repositories if you want the most up-to-date versions; these branches/commits are only listed for reproducibility.*

Picoprobe is pretty big and we don't need all its history (hence why we used `--depth 1`).  
If you want to further save on space at the cost of history, feel free to delete its `.git\` directory.

*The following are the SHA1 hashes of each repo's head as used in this project:*  
*pico-sdk: `6a7db34ff63345a7badec79ebea3aaef1712f374`*  
*pico-examples: `eca13acf57916a0bd5961028314006983894fc84`*  
*picotool: `f6fe6b7c321a2def8950d2a440335dfba19e2eab`*  
*picoprobe: `1267a8c367c73ef659cac3fde22ffc3e3074fd99`*  

&nbsp;



**1.7.1. Patch the Pico SDK:**

At the time of writing (Pico SDK 1.5.1), there exists a bug in the Pico SDK where the PICO_DEOPTIMIZED_DEBUG CMake setting does not update the CMake cache.

To fix it, open the file `C:\VSARM\sdk\pico\pico-sdk\cmake\preload\toolchains\set_flags.cmake`  
and add the line `unset(CMAKE_${LANG}_FLAGS_DEBUG CACHE)` here:

```cmake
  foreach(LANG IN ITEMS C CXX ASM)
      set(CMAKE_${LANG}_FLAGS_INIT "${ARM_TOOLCHAIN_COMMON_FLAGS}")
      unset(CMAKE_${LANG}_FLAGS_DEBUG CACHE)
      if (PICO_DEOPTIMIZED_DEBUG)
          set(CMAKE_${LANG}_FLAGS_DEBUG_INIT "-O0")
```

You can follow the pull request for this patch [here](https://github.com/raspberrypi/pico-sdk/pull/1620).

If you don't want to modify the SDK, you can [alternatively](https://github.com/raspberrypi/pico-sdk/issues/1618#issuecomment-1913635115) pass the `--fresh` option to CMake every time you change PICO_DEOPTIMIZED_DEBUG.

&nbsp;



**1.8. Add `...\mingw32\bin` and `...\pico-sdk` to your system path and as an environment variable:**
  - Press the Windows key, search "env", and select "Edit the system environment variables.  
  - Select "Environment Variables..."  
  - In the "User variables" section, highlight the "Path" variable and select "Edit..."  
    *(You should already see the path you installed the Arm GNU Toolchain to in the list here)*  
  - Select "New" and add your full path for `C:\mingw32\bin` to the list
  - Click "OK" to exit  
  - In the "User variables" section again, select "New..."  
    - Variable name: `PICO_SDK_PATH`  
    - Variable value: `C:\VSARM\sdk\pico\pico-sdk`  
  - Once again, select "New..."
    - Variable name: `CMAKE_GENERATOR`
    - Variable value: `MinGW Makefiles`
  - Click "OK" to exit  
    *(Within the "Path" entry of "System variables", you should be able to verify `...\CMake\bin` and `...\Git\cmd` are in the system path)*  
  - Click "OK" to exit any remaining environment variable windows

*If you don't want to set MinGW Makefiles as your default CMake generator, then be sure to append `-G "MinGW Makefiles"` to any `cmake ..` commands in this guide.*

*As suggested by Shawn, opening a new command prompt at this point and entering commands like `gcc -v` , `make -v` , and `echo %PICO_SDK_PATH%` should now all return expected information.*

&nbsp;



**1.9. Verify the functionality of this development environment so far** by building and uploading a simple "blink" program.

In a new terminal enter:

  ```
  cd C:/VSARM/sdk/pico/pico-examples/
  mkdir build
  cd build
  cmake ..
  cd blink
  make
  ```

With the Raspberry Pi Pico unplugged, hold down the BOOTSEL button and plug in the Pico.  
Copy the file `...\pico-examples\build\blink\blink.uf2` over to the Raspberry Pi Pico's root directory.

If you see a blinking green light on your Pico, then everything appears to have gone right up to this point! 🎉

&nbsp;



### ***Next we'll work on debug tools***

**2.1. Install a serial monitor.**  

There are plenty of options out there, but I will be using [Tabby](https://tabby.sh/) if you'd like to follow along.  [Serial Studio](https://serial-studio.github.io/) looks incredibly promising, but unfortunately [does not initialize the connection with the Pico correctly](https://github.com/Serial-Studio/Serial-Studio/issues/135) on Windows at the time of writing.

&nbsp;



**2.2. Verify serial output.**

Open Windows Device Manager and make a note of all the currently populated COM port indicies.

In a terminal enter:

  ```
  cd C:/VSARM/sdk/pico/pico-examples/build/hello_world/usb/
  make
  ```

With the Raspberry Pi Pico unplugged, hold down the BOOTSEL button and plug in the Pico.  
Copy the file `...\pico-examples\build\hello_world\usb\hello_usb.uf2` over to the Pico.  

Once the Pico restarts, check Device Manager again to see what the new COM port index is.  
In Tabby, select "`Profiles & connections`", and select the "`Serial: ... COMx`" that aligns with the COM port which appeared when the Pico restarted.  
Select a baud rate of 115200.  
You should see the Pico repeatedly printing "`Hello world!`"

> *COM ports can become inaccessible due them to being used by other applications.  Be sure to close other applications that may try attaching to COM ports, such as 3D printing software.*

> *When you disconnect your Pico (or Picoprobe later), be sure to remove it with Windows' "Safely Remove Hardware" interface in the system tray.  
If you don't, you'll likely get countless "zombie" COM ports reserving themselves in the list.*

> *COM ports on Windows can just be annoying in general.  It should "just work", but if it doesn't it may be worth trying to play with manually changing the COM port used by the Pico via Device Manager, as well as manually increasing the Baud rate there.*

&nbsp;



**2.3. Using [7-Zip](https://7-zip.org/), extract [OpenOCD](https://github.com/openocd-org/openocd/actions/runs/7771215993):**

> *The OpenOCD link above takes you to an official automated snapshot build.  To install, download the "artifact" at the bottom of the page.*  
  *OpenOCD 0.12.0 does not provide full debugging capabilities for the Pico, so at this time we need to install a snapshot build.*  
  *In future [release versions](https://github.com/openocd-org/openocd/releases)  of OpenOCD, presumably 0.13.0 and up, you should be able to use the most resent stable release instead.*

Place the extracted files in `C:\Program Files\OpenOCD\`.  The archive will need to be extracted twice to access the folders within.

*This project was built using the archive:*  
*`openocd-4593c75f0-i686-w64-mingw32.tar.gz`*  
*SHA256 checksum: `EDFFCE6293875D8ECED3121D91832DD08BCED813FBFA52FA407182C97F34FE2E`*

&nbsp;



**2.4. Add `C:\Program Files\OpenOCD\bin` to your system path** - *See step 1.8 if you need instructions for editing the system path.*

To confirm OpenOCD installed correctly, in a terminal enter:

  ```
  openocd --version
  ```

&nbsp;



**2.5. Using [7-Zip](https://7-zip.org/), extract [libusb-1.0](https://github.com/libusb/libusb/releases/tag/v1.0.26)** and place the `libusb-MinGW-Win32\` subdirectory into a helpful location *(such as `C:\VSARM\sdk\pico\libusb\libusb-MinGW-Win32\`)*

*This project was built using the archive:*  
*`libusb-1.0.26-binaries.7z`*  
*SHA256 checksum: `9C242696342DBDE9CDC47239391F71833939BF9F7AA2BBB28CDAABE890465EC5`*

&nbsp;



**2.6. Build picotool:**

In a terminal enter:  
  
  ```
  cd C:/VSARM/sdk/pico/picotool/
  mkdir build
  cd build
  cmake -E env LIBUSB_ROOT="C:/VSARM/sdk/pico/libusb/" cmake ..
  make
  ```

You should now have `picotool.exe` in your `...\picotool\build\` directory, but it will not be functional yet.  
Next, navigate to `C:\Program Files\OpenOCD\bin\`  
Copy the file `libusb-1.0.dll` from here to your `...\picotool\build\` directory.  

&nbsp;



**2.7. Add `C:\VSARM\sdk\pico\picotool` to your system path** - *See step 1.8 if you need instructions for editing the system path.*

To confirm picotool is correctly installed, in a terminal enter:

  ```
  picotool version
  ```

&nbsp;



**2.8. Download [Zadig](https://github.com/pbatard/libwdi/releases/tag/v1.5.0)**.  

*This project was built using the installer:*  
*`zadig-2.8.exe`*  
*SHA256 checksum: `20E4CD7B6768718848F603FE928F36E207DC5CA96FC9DB7085D841410D0ABAE4`*

&nbsp;



**2.9. Install drivers for the Pico:**  
  - With the Raspberry Pi Pico unplugged, hold down the BOOTSEL button and plug in the Pico.  
  - Launch Zadig.  
  - Ensure "`RP2 Boot (Interface 1)`" is selected as the device *(if you don't see `RP2 Boot (Interface 1)`, you may need to select `Options > List All Devices`)*.  
  - Ensure the current driver is listed as "`(NONE)`".  
  - Ensure "`WinUSB (...)`" is selected as the driver to install.  
  - Click "`Install Driver`", and wait for it to successfully install.

To confirm the bootloader driver is working, in a terminal enter:

  ```
  picotool info -a
  ```

This should return detailed info about the connected Pico.  *If this doesn't work, maybe try installing other driver types for `RP2 boot (Interface 1)`*  
Once the driver is working, in a terminal enter:

  ```
  picotool reboot
  ```

Ensure the Pico is still running the "`hello_usb`" firmware, such as by checking your serial monitor.  

Return to Zadig to install one more driver:
  - Ensure "`Reset (Interface 2)`" is selected as the device *(if you don't see `Reset (Interface 2)`, you may need to select `Options > List All Devices`)*.
  - Ensure the current driver is listed as "`(NONE)`".
  - Ensure "`WinUSB (...)`" is selected as the driver to install.
  - Click "`Install Driver`", and wait for it to successfully install.

To confirm the application driver is working, in a terminal enter:

  ```
  picotool reboot -f -u
  ```

This should reboot the Pico into bootloader mode  
*(Note that picotool will only work outside of the bootloader like this for firmware with USB interfaces)*.

&nbsp;



**2.10. Build and upload picoprobe** - In a terminal enter:

  ```
  cd C:/VSARM/sdk/pico/picoprobe/
  mkdir build
  cd build
  cmake ..
  make
  ```

*I received some compiler warnings from the FreeRTOS submodule, but none were fatal.*

With the Raspberry Pi Pico you wish to use as your debugger unplugged, hold down the BOOTSEL button and plug in that Pico.  
Copy the file `...\picoprobe\build\picoprobe.uf2` over to the Pico.

The LED on the picoprobe should now be on.

&nbsp;



**2.11. Upload firmware to the target Pico via OpenOCD:**

Wire your 2 Picos together as show in [Raspberry Pi's getting started guide](https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf#picoprobe-wiring-section).

In a terminal enter:

  ```
  cd C:/VSARM/sdk/pico/pico-examples/build/blink/
  openocd -f interface/cmsis-dap.cfg -f target/rp2040.cfg -c "program blink.elf verify reset exit"
  ```

The target Pico should now be blinking.

&nbsp;



**2.12. Launch the OpenOCD server:**

In a terminal enter:

  ```
  openocd -f interface/cmsis-dap.cfg -f target/rp2040.cfg -c "adapter speed 5000"
  ```

**Leave this terminal open!**

&nbsp;



**2.13. Launch GDB:**

In a *separate* terminal enter:

  ```
  cd C:/VSARM/sdk/pico/pico-examples/build/blink/
  arm-none-eabi-gdb blink.elf
  target extended-remote localhost:3333
  monitor reset init
  break gpio_put
  continue
  ```

Every time you enter `continue`, the light should blink and wait for your next `continue`.

Once that works, congratulations!  You now have the debugging tools all set up! 🎉

&nbsp;



### ***And finally, we'll set up the integrated development environment (IDE)***

**3.1. Install [VSCodium](https://github.com/VSCodium/vscodium/releases/tag/1.85.2.24019)** with personal or default settings.

*This project was built using the installer:*  
*`VSCodium-x64-updates-disabled-1.85.2.24019.msi`*  
*SHA256 checksum: `A8BAC8852CE135A64F5DA19553AC0D39B42517D2835DDAC2ABDEAC1C586C40FF`*

&nbsp;



**3.2. Install the [CMake Tools VSCode extension](https://open-vsx.org/extension/ms-vscode/cmake-tools/1.16.32):**

Install the extension's .VSIX file linked above:  

- In the left pane of VSCodium, click on the "Extensions" icon (four assembling blocks).  
- Select the "..." at the top of the list/searchbar, and select "Install from VSIX..."  
- Navigate to the .VSIX you downloaded and select it for installation.  

*(You can alternatively do this by searching within VSCode's extensions marketplace if your application has an internet connection)*  

This extension depends on the **[CMake language support extension](https://open-vsx.org/extension/twxs/cmake/0.0.17)**, so install that the same way.  
*If you install CMake Tools through VSCodium's extensions marketplace, the CMake language support dependency should automatically install as well.*

In the "Extensions" panel, locate the newly installed CMake Tools extension and click the gear icon.  Select "Extension Settings" and update the following settings:

  - Cmake: Generator
    > `MinGW Makefiles`  
  - Cmake > Options: Status Bar Visibility
    > visible

Additionally, if you'd like to avoid being asked to configure your project every time you open it, assert the following setting (ie. don't leave it as an unmodified default):

  - Cmake: Configure On Open
    > unchecked

Restart VSCodium.

*This project was built using the files:*  
*`ms-vscode.cmake-tools-1.16.32.vsix`*  
*SHA256 checksum: `4ED48ACFB71D49D869693898D6C4FB0367C59270AD6E929BF1CCF11A6699B855`*  
*`twxs.cmake-0.0.17.vsix`*  
*SHA256 checksum: `F2AC4A4548AFEB17DB8D0411BEA5DC835F5907A3E755D84DC6F164288293EDBE`*  

&nbsp;



**3.3. Confirm the CMake extensions are working by creating a test project:**

- Create a directory in your personal files called `...\pico-testing\` to contain test/tutorial projects.  
- Create a sub-directory called `...\pico-testing\hello-world\` .  
- In VSCodium go to `File > Open Folder...` and open `...\pico-testing\hello-world\` .  
*If you added VSCodium to your context menu during installation, you can also just navigate to the `...\pico-testing\` directory in File Explorer and right click on the `hello-world\` folder to select "`Open with VSCodium`"*
- When prompted if you trust this workspace, select "`Trust Parent`" to trust all future files within `...\pico-testing\` .  
- Close the "Workspace Trust" tab.
- In the "`Explorer`" pane (two overlapping sheets of paper), click on the "`New File...`" button (a sheet of paper with a "+") next to the project title `HELLO-WORLD`.  
  Call this new file "`main.c`".  We'll come back to this later.
- Create another "`New File...`" and call it "`CMakeLists.txt`"
- Manually input the following code into "`CMakeLists.txt`":



  ```cmake
  # CMakeLists.txt

  # This is the version Raspberry Pi uses, no explicit reasoning provided
  cmake_minimum_required(VERSION 3.13)

  # Tell CMake and the Pico SDK to not optimize anything for debugging
  set(PICO_DEOPTIMIZED_DEBUG ON)

  # Pre-initialize the Pico SDK
  include($ENV{PICO_SDK_PATH}/external/pico_sdk_import.cmake)

  # Set PROJECT_NAME
  project(hello-world)

  # Invoke the Pico SDK's CMakeLists.txt to import build settings and definitions
  pico_sdk_init()

  # Tell CMake what/where our executable source file is
  add_executable(${PROJECT_NAME} main.c)

  # Link the project to pico_stdlib as defined by the SDK
  target_link_libraries(${PROJECT_NAME} pico_stdlib)

  # Enable UART stdio output
  pico_enable_stdio_uart(${PROJECT_NAME} ON)

  # Create map/bin/hex/uf2 files, in addition to ELF
  pico_add_extra_outputs(${PROJECT_NAME})

  ```



As you enter this code, the CMake language support extension should suggest common auto-completions for you.  

- As of writing, the CMake Tools interface only loads if there is a `CMakeLists.txt` file present at start-up, so we need to manually reload the extension.  
*This is due to a bug in the CMake Tools extension, and in the future should hopefully not be a problem - you can track [the issue](https://github.com/microsoft/vscode-cmake-tools/issues/1794) on GitHub.*  
  - To reload the extension, summon the Command Palette in VSCodium by pressing `Ctrl + Shift + P` and type in the prompt "`CMake: Reset`"  
  - Select "`CMake: Reset CMake Tools Extension State (For troubleshooting)`"  
    *(You won't need to do this when re-opening the project in the future)*
- At the bottom of VSCodium, you should now see information like "`CMake: [Debug]: Ready`" and "`No Kit Selected`".  
- Click on "`No Kit Selected`" at the bottom of the screen and select "`[Scan for kits]`" at the top of the window.
- Once it's done scanning, click on "`No Kit Selected`" again, and this time you should see two options (one from MinGW, one from ARM).  
  Select "`GCC 13.2.1 arm-none-eabi`".
- Ensure that "`CMake: [Debug]: Ready`" is what is selected next to that.

CMake's output should appear in VSCodium's integrated terminal, and it should conclude with the successful message:  

  ```
  [cmake] -- Configuring done (...)
  [cmake] -- Generating done (...)
  [cmake] -- Build files have been written to: .../pico-testing/hello-world/build
  ```

> *If certain terminals within VSCodium are not detecting environment variables, like Git Bash not returning `echo $PICO_SDK_PATH`, you may need to fully close VSCodium and restart it without selecting a specific workspace (it should automatically open the most recent workspace).*  
*This may relate to having multiple VSCodium "workspaces" (windows) open at once?  I believe [this issue](https://github.com/microsoft/vscode/issues/47816) is tracking the behaviour.*

&nbsp;



**3.4. Extract the [clangd language server](https://github.com/clangd/clangd/releases/tag/17.0.3)** somewhere helpful.

*(This guide will assume it is extracted to `C:\Program Files\clangd\` with the directory's version number removed)*

*This project was built using the archive:*  
*`clangd-windows-17.0.3.zip`*  
*SHA256 checksum: `BE2E387544DF97A36CDBA9325562A9695C22AF7A8ABCBCB0DEC0E73ED9AD8127`*  

&nbsp;



**3.5. Add `C:\Program Files\clangd\bin` to your system path** - *See step 1.8 if you need instructions for editing the system path.*

&nbsp;



**3.6. Install the [clangd VSCode extension](https://open-vsx.org/extension/llvm-vs-code-extensions/vscode-clangd/0.1.26):**

Install the extension's above .VSIX file as described in 3.2.

Access the clangd extension settings as described in 3.2 and change the following setting:

  - Clangd: Arguments - Add Items:

    > `--query-driver=C:/mingw32/bin/*`  

    > `--query-driver=C:/VSARM/armcc/*/bin/*`

    *This will grant the clangd language server permission to access your compiler directories.*

Restart VSCodium.

*This project was built using the file:*  
*`llvm-vs-code-extensions.vscode-clangd-0.1.26.vsix`*  
*SHA256 checksum: `E77F87E88F064704A7E864B5518486B00817A966469CED12DA2603C7E1F07597`*  

&nbsp;



**3.7. Confirm the clangd extension is working in our test project:**

Because we ran CMake (and in particular, that the CMake Tools extension had CMake generate `...\build\compile_commands.json`), we will be able to fully enjoy clangd's code completion, diagnostics, and other helpful features.

Manually input the following code into "`main.c`":



  ```c
  // main.c

  #include <stdio.h>
  #include "pico/stdlib.h"

  #define PIN_LED 25

  int main() {

    stdio_init_all();

    gpio_init(PIN_LED);
    gpio_set_dir(PIN_LED, GPIO_OUT);

    while (true) {

      printf("Hello, world!\n");
      gpio_put(PIN_LED, true);
      sleep_ms(250);
      gpio_put(PIN_LED, false);
      sleep_ms(750);
      
    }

  }
  ```



You should see plenty of autocompletion suggestions from clangd as you compose this code.

You will know that the clangd language server is *fully* working if you hover over `<stdio.h>` and see that it recognizes it as being from:  
`C:\VSARM\armcc\<VERSION>\arm-none-eabie\include\stdio.h`  
*(Don't worry if the verison number is highlighted red, [that's just an artifact](https://github.com/clangd/vscode-clangd/issues/578) of how VSCode renders text)*  
If this header file is not recognized (ie. it is underlined red), then you may need to double-check your paths (with asterisks) are correct in the `--query-driver` arguements added above.

You also may see that `#include "pico/stdlib.h"` is underlined yellow.  This is because clangd identified we don't need all of the files referenced within `pico/stdlib.h` and it would prefer we include each smaller component individually.  This is known as [include-what-you-use (IWYU)](https://github.com/include-what-you-use/include-what-you-use/blob/master/docs/WhyIWYU.md).  
If you'd like to disable this warning, you can read [this guide](https://clangd.llvm.org/guides/include-cleaner#configuration) from clangd.  It also might be worth tracking [this issue](https://github.com/clangd/clangd/issues/1913) on their GitHub, as this behaviour may change in future versions of clangd.

&nbsp;



**3.8. Configure the Pico SDK using the CMake Tools extension:**

We have thorough diagnostics from clangd within our project directory, but if you `Ctrl + Click` on one of the SDK functions, such as `gpio_put()`, it will take you to the source file in the SDK and there will be errors reported.

To fix this, open the folder `C:\VSARM\sdk\pico\pico-sdk\` in VSCodium.

Trigger CMake Tools to configure by ensuring "`GCC 13.2.1 arm-none-eabi`" is your kit, and then select "`CMake: [Debug]: Ready`" and choose "`Debug`" at the top of the window.  
You can close the VSCodium window containing the SDK.

> *We don't run CMake from a terminal here because the CMake Tools extension passes flags that help clangd, like for generating `compile_commands.json`*

Now if you return to our `hello-world` project, you should be able to browse the SDK source code with full diagnostic accuracy!

*If your SDK diagnostics are not updating, try saving or re-opening the file as that will prompt clangd to re-check for the SDK's `compile_commands.json`.*

> *If you ever need to change the Pico SDK version for testing, be sure to call:*  
> `git checkout <VERSION> --recurse-submodules` in the SDK directory.  
> *You may need to append* `--force` *the first time you change to a commit where submodules were historically added/removed (I think).*

&nbsp;



**3.9. Install the [Cortex-Debug VSCode extension](https://open-vsx.org/extension/marus25/cortex-debug/1.12.1):**

Install the extension's above .VSIX file, as described in 3.2.

This extension depends on the following additional extensions, so install them in the same way:

  - **[debug-tracker-vscode](https://open-vsx.org/extension/mcu-debug/debug-tracker-vscode/0.0.15)**

  - **[MemoryView](https://open-vsx.org/extension/mcu-debug/memory-view/0.0.25)**

  - **[RTOS Views](https://open-vsx.org/extension/mcu-debug/rtos-views/0.0.7)**

  - **[Peripheral Viewer](https://open-vsx.org/extension/mcu-debug/peripheral-viewer/1.4.6)**

*If you install Cortex-Debug through VSCodium's extensions marketplace, the dependencies should automatically install as well.*

Restart VSCodium.

*This project was built using the files:*  
*`marus25.cortex-debug-1.12.1.vsix`*  
*SHA256 checksum: `F315306B5D964C1E40D2802FDF8AA2EC86C62C0A73477D46964666A8381DA71F`*  
*`mcu-debug.debug-tracker-vscode-0.0.15.vsix`*  
*SHA256 checksum: `0BF99FFB60DC82D8D2E5F27A64DDE7B54FC292236902EDAAEFCCDBB1675BFD3E`*  
*`mcu-debug.memory-view-0.0.25.vsix`*  
*SHA256 checksum: `6E928BE25A3F3655596A63084FE537881273F814EA4C808C57F0EDCF530AEC89`*  
*`mcu-debug.rtos-views-0.0.7.vsix`*  
*SHA256 checksum: `9FDE20791C9C3F0E6C12B1C7136AB798806FA55F122F5A84C2218AC076686DB9`*  
*`mcu-debug.peripheral-viewer-1.4.6.vsix`*  
*SHA256 checksum: `273BBD73794F3ED78D99E7388A3A8CFD4A7FC022BE35FC794D8858FCBCFB1AAA`*  

&nbsp;



**3.10. Configure VSCodium debug settings:**

By default, VSCodium will operate based on your user settings.  
If VSCodium finds a `.vscode` directory in the root of your project, then the settings within that directory take priority.

If you'd like your project(s) to use a local `.vscode` folder, refer to [`...\pico-examples\ide\vscode\README.md`](https://github.com/raspberrypi/pico-examples/blob/sdk-1.5.1/ide/vscode/README.md) for more information.  
This guide will set up the global settings instead.

- In VSCodium, click the gear icon in the bottom left and select "Settings"

- Search for `@id:launch`

- Select "`Edit in settings.json`"

- Input the following launch configuration into `settings.json`:



  ```json
  
      "launch": {
          "version": "0.2.0",
          "configurations": [
              {
                  "name": "Pico Debug",
                  "cwd": "${workspaceRoot}",
                  "executable": "${command:cmake.launchTargetPath}",
                  "request": "launch",
                  "type": "cortex-debug",
                  "servertype": "openocd",
                  "gdbPath" : "arm-none-eabi-gdb",
                  "device": "RP2040",
                  "configFiles": [
                      "interface/cmsis-dap.cfg",
                      "target/rp2040.cfg"
                  ],
                  "openOCDLaunchCommands": [
                      // This may be able to go as high as 24000 for certain setups
                      "adapter speed 5000"
                  ],
                  "svdFile": "${env:PICO_SDK_PATH}/src/rp2040/hardware_regs/rp2040.svd",
                  "preLaunchCommands": [
                      "set remote hardware-breakpoint-limit 4",
                      "set remote hardware-watchpoint-limit 2"
                  ]
              }
          ]
      }

  ```

  > The [RP2040 datasheet](https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf#section_Processor-Debug) suggests an adapter speed up to 24000, depending on your setup.  For reference, the highest I was able to get mine was 20000.

  > The [RP2040 datasheet](https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf#section_Processor-Debug) also lists the 4 hardware breakpoints and 2 watchpoints.  As of writing, automatic breakpoints set by GDB do not appear to be counting towards this limit.  I've opened [an issue](https://github.com/Marus/cortex-debug/issues/978) in Cortex-Debug to request explicitly sending hardware breakpoints to GDB, as well as [an issue](https://sourceware.org/bugzilla/show_bug.cgi?id=31403) directly to GDB.  
  **In the meantime, beware that setting more than 4 breakpoints through the UI will cause problems.**
  
  > *Raspberry Pi [recommends](https://github.com/raspberrypi/pico-examples/tree/sdk-1.5.1/ide/vscode) using a launch.json file included in `pico-examples`, however I will be using the above launch configuration as it provides more robust functionality.*



You're good to save and close `settings.json`.

Raspberry Pi also recommends asserting certain CMake Tools settings on the project-level to avoid confusion against the debugger buttons (see [here](https://github.com/raspberrypi/pico-examples/blob/sdk-1.5.1/ide/vscode/settings.json)), but I personally prefer keeping all my CMake Tools UI at the bottom bar, and my debugger UI in the side panel.  
If you'd like to follow my preference, ensure these settings are set:

- Debug > Show In Status Bar
  > never

- Cmake > Options: Status Bar Visibility
  > visible

- Cmake > Build Before Run
  > *checked*

Restart VSCodium.

> *As before, if certain terminals within VSCodium are not detecting environment variables, like Git Bash not returning* `arm-none-eabi-gdb -v`*, you may need to fully close VSCodium and restart it without selecting a specific workspace (it should automatically open the most recent workspace).*  
*This may relate to having multiple VSCodium "workspaces" (windows) open at once.*

And with that, we're all set to begin debugging! 🎉

&nbsp;



### ***Configure, build, and debug with a single click in your IDE!***

**4. Start debugging!**

Click on the "Run and Debug" icon (a play button with a bug next to it).

You should see at the top of the "Run and Debug" panel a green play button and a drop-down menu.  
Ensure this drop-down menu says "Pico Debug".

Press play.

The first time you run the debugger, you'll need to select what file you are targetting.  
Select `hello-world`.

When you first run the debugger, it may appear to hang with nothing changing in its output - do be patient (up to a minute).

And with that...

### ***Congratulations!*** 🎉

***Your entire toolchain, debugger, and IDE are all set up and ready for development!***

Set breakpoints with the red dots next to the line numbers (including conditional breakpoints) and step through code with the controls floating at the top of the window :)  
You can read more about using VSCode's debugging features [here](https://code.visualstudio.com/Docs/editor/debugging).

The Cortex-Debug extension (and its dependencies) also feature the tabs "MEMORY" and "XRTOS" alongside the "TERMINAL" and "DEBUG CONSOLE" in the bottom pane.
I have not yet explored implementing FreeRTOS, so I hide the XRTOS view for simplicity.
If you are interested, you can follow the support for multi-core FreeRTOS viewing at [this pull-request](https://github.com/mcu-debug/rtos-views/pull/44).

If you'd like to view raw memory, then feel free to enter the "MEMORY" tab, and while debugging a program on the Pico click the plus icon next to the drop-down.  When prompted, enter `0x20000000` as the memory address (this is the [starting address](https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf#_address_map) of the RP2040 SRAM)

Happy hacking!

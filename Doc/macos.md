# Troubleshooting STM32 Connectivity on Apple Silicon Macs
# 1. Technical Root Causes
There might be a problem with the CC pin configuration or the USB 2.0 implementation/connection on the board.
### The Hub Solution:
A USB Hub acts as an active intermediary that correctly handles the USB-C handshake protocols which the board might be missing or implementing incorrectly. By connecting through a hub, the Mac acts as the host to the compliant hub, while the hub manages the simpler connection to the development board.
# 2. Recommended Solution
## Hardware: Using a Hub
If you do not have a compatible hub, the following model is known to work: [Recommended USB-C Hub on Amazon](https://amzn.eu/d/gGiMBCe)

# !!!THIS IS JUST A THEORIE!!!
I have no idea if thats the actual problem, however this fixed it for me and many of my classmates and
this only applies to Silicon macs, for Intel there shouldnt be a problem.

# 3. Software Environment Setup
To develop for ARM-based microcontrollers like the STM32 on macOS, you need the cross-compiler toolchain and standard build utilities. This will then work for vscode and eclipse. However vscode
should only be used if you know what you're doing. I recommend using the [cortex Debug](https://marketplace.visualstudio.com/items?itemName=marus25.cortex-debug) extension rather than the one provided by st.
## Install Homebrew
Homebrew is needed to install all kind of things (You should have used it before)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
## Install Build Tools (make, clang, etc.)
Open your terminal and run the following command to install the Apple Command Line Tools:
```bash
xcode-select --install
```
## Install ARM Toolchain
Use Homebrew to install the GNU Arm Embedded Toolchain:
```bash
brew install --cask gcc-arm-embedded
```
## Install other tools that may be needed

```bash
brew install ghidra cmake curl python@3.14 wget microsoft-openjdk@21 git open-ocd@0.12.0
brew pin open-ocd
```
### ImHex
[ImHex](https://imhex.werwolv.net) has to be downloaded manually since brew deprecated it

## Verification
Verify the installation by checking the compiler version:
```bash
    arm-none-eabi-gcc --version
    openocd --version
```

# Eclipse
If eclipse doenst build rightaway you have to add homebrew to the project path. For this you have to type in `which openocd` and copy the path without the opencod in it, for example `/opt/homebrew/bin`.
Copy that path and open eclipse -> right click Project -> properties -> C/C++ Build -> Enviroment -> select path then click on edit and paste the path you just copied.

## External Tool config
Press on the arrow of external tool icon inside the top bar of eclipse. 

![External Tool Icon](external_tool_icon.png)

Then just copy the following into the Arguments text field:
```bash
-f /opt/homebrew/Cellar/open-ocd/0.12.0_1/share/openocd/scripts/interface/stlink.cfg -f /opt/homebrew/Cellar/open-ocd/0.12.0_1/share/openocd/scripts/target/stm32g4x.cfg
```

To get the path for the Location field run `which openocd` in your terminal and copy the path. For example:
```bash
/opt/homebrew/bin/openocd
```

Then Press on Build -> Specific Projects only -> Projects and select your project
## Debug Config
For the Debug configuration select GDB Hardware Debugging. 
- For Project select your Project
- For C/C++ Application go to search Project and then select your desired app (app.elf for now, you have to build it first so eclipse can find it)

Then go to the Debugger Tab
- For the GDB command run `which arm-none-eabi-gdb` in your console and paste it in.
- Change Connection to `localhost:3333``

# Vscode
Download the Recommended [cortex Debug](https://marketplace.visualstudio.com/items?itemName=marus25.cortex-debug) extension (probe rs would also work or other tools).
Create a launch.json inside the .vscode folder and paste in the following:
```json
{
    "version": "0.2.0",
    "configurations": [
          {
            "cwd": "${workspaceFolder}",
            "executable": "./build/app.elf",
            "name": "Debug App",
            "request": "launch",
            "type": "cortex-debug",
            "runToEntryPoint": "main",
            "showDevDebugOutput": "none",
            "servertype": "openocd",
            "configFiles": ["/opt/homebrew/Cellar/open-ocd/0.12.0_1/share/openocd/scripts/interface/stlink.cfg","/opt/homebrew/Cellar/open-ocd/0.12.0_1/share/openocd/scripts/target/stm32g4x.cfg"],
        }
    ]
}
```
> **Warning:** The official development environment for this lecture is Eclipse. Use VS Code only if you are experienced and comfortable troubleshooting your own environment, as no support will be provided for VS Code-related issues.


# Troubleshooting
## Ghidra:
If Ghidra isn't found inside your terminal, add the following line to your .zshrc profile:

```bash
echo "alias ghidra='/opt/homebrew/Cellar/ghidra/*/libexec/ghidraRun'" | sudo tee -a ~/.zshrc
```
Relaunch the terminal and it should work.
## Build error:
if there's a build error trying to build the VPTemplate change line 
164-166 with the following or copy the makefile in this repo. This is an error made by the Lecturer, he will fix this however for now use the fixed version:
```c
$(OBJ_DIR)/libstm32.a: $(OBJS_STM32LIB)
	@echo "  AR      $(notdir $@)"
	@$(AR) rcs $@ $(OBJS_STM32LIB)
```

If any other tool doesn't work, it's most likely a linking error.
If nothing works at all please use the provided Kali Linux VM and contact the creator of this guide for any help so I can update/fix it: [email](mailto:nils_0805@icloud.com). 


# `cube2pio`: Convert STM32CubeIDE 🧊 projects to PlatformIO 🐜 projects!
STM32 MCUs can be programmed with ST's official [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html) or independent tools such as [PlatformIO](https://platformio.org/). To some, the Cube IDE is **bloated, slow and impractical** and they would rather program with a different IDE, such as PlatformIO. However, the Cube IDE still provides solid code generation and graphical settings of pins and the clock tree, which makes it a valuable tool. This repository combines the best of both worlds: 

**Set up low level code with the Cube IDE and write your application with PlatformIO!**

Too many times I tried to write an STM32 application in PlatformIO with the STM32 HAL, only to find out that the amount of configuration one has to make is very overwhelming and unintuitive. Let the code generation handle that and focus on your application!

## 📜 Installing
Install this python repo as module (in a virtual environment) with 
```bash
pip install git+https://github.com/jesdev-io/cube2pio.git
```

## 🧑‍💻 Usage
In your virtual environment, you now have access to the command `cube2pio`:
```bash
cube2pio -p /path/to/pio_project -c /path/to/cube_project
```

You will then observe that a folder `lib/port` will have appeared in your PlatformIO project. It holds the source files and some abstraction logic. You can now run an application in an Arduino-esque style:
```c
// src/main.c

#include "port.h"

void port_setup(){

}

void port_loop(){

}
```
Additionally, you can specify a port folder name with `-l <name>`, e.g. `-l port_adc_app`. This is useful when your PlatformIO project contains more than one firmware that can be selected by the PlatformIO environment. 

You can also add the flag `--use-freertos` if you link FreeRTOS externally with PlatformIO. See [How it works](#-how-it-works) for details on why this is needed.

## 💡 How it works
`cube2pio` copies the necessary source files of the Cube folder `Core/Src` and `Core/Inc` to `lib/<port_name>`. It also puts all contents of `main.c` into `port.c` so you can specify your own `main.c`. There, the function `port_setup()` and `port_loop()` are inserted before and after the `while(1)` of the "true" main function in `port.c`. This means that all HAL initializations happened **before** `port_loop()` is called. This way, your `main.c` can focus on the application without seeing the boilerplate all the time. If something fails during init, the default `Error_Handler()` of the imported project is triggered. If you want to define ISRs with the HAL specific signature, you can do so in your `main.c` or any other custom source file in `src/` or `lib/`.

If you link FreeRTOS in your PlatformIO project, some handler functions of the HAL are already defined there. The RTOS-less Cube project defines these functions by itself, leading to compilation errors when building the PlatformIO project. For this reason, you can specify the flag `--use-freertos`, which formats the imported functions as `__weak` so FreeRTOS can define them properly. **Be careful!** If this flag is set wrongly, your project will either fail at build time (because of double definitions) or the app will never start (because the functions are never defined at all).

## ⚠️ Limitations
Currently, porting usage of **middleware is not supported**. Only source code in the Cube project's `Core/` folder are considered.

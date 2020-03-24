# Development Environment Setting
## Install Extension
1. C/C++ IntelliSense, debugging, and code browsing.
2. Cortex-Debug
3. gitignore
4. Mardown Preview
5. Prettier Code formatter using prettier
6. vscode-icons
7. Python
## Extension Config
1. Coretex-Debug
Create launch.json file, with this config.
``` json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Cortex Debug",
            "cwd": "${workspaceRoot}",
            "executable": "./build/CANOpen-STM32.elf",
            "request": "launch",
            "type": "cortex-debug",
            "servertype": "openocd",
            "device": "STM32F446RE",
            "configFiles": [
                "./board/stm32f4x.cfg"
            ]
        }
    ]
}
```
meanwhile, copy /usr/share/openocd/* files, including stm32f4x.cfg, swj-dp.tcl mem_helper.tcl to local directory.

Fix the directory path problem
```
#
# stm32 devices support both JTAG and SWD transports.
#
source ./board/swj-dp.tcl
source ./board/mem_helper.tcl
```
And put the next code at the beginning of stm32f4x.cfg, for config the st-link config
```
# script for stm32f4x family
interface hla
hla_layout stlink
# lsusb, then can known the st-link usb-id and desc
hla_device_desc "ST-LINK/V2.1"
hla_vid_pid 0x0483 0x374b
adapter_khz 1800
transport select hla_swd

```
2. C/C++ Auto-completion
VSCode will automatically promote add c_cpp_prooperties.json to config project. Add include path to the json files.
``` json
{
    "configurations": [
        {
            "name": "STM32CubeMX",
            "includePath": [
                "${workspaceFolder}/Inc",
                "${workspaceFolder}/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2",
                "${workspaceFolder}/Middlewares/Third_Party/FreeRTOS/Source/include",
                "${workspaceFolder}/Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM4F",
                "${workspaceFolder}/Drivers/STM32F4xx_HAL_Driver/Inc",
                "${workspaceFolder}/Drivers/CMSIS/Device/ST/STM32F4xx/Include"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/clang",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "clang-x64",
            "compileCommands": "${workspaceFolder}/compile_commands.json"
        }
    ],
    "version": 4
}
```
3. compile_commands.json generation

For linux system, it's quite simple, use bear.

For windows/macOS, compiledb from python is another choice.
``` shell
$ bear make
or
$ compiledb make -j8

By default, compiledb make generates the compilation database and runs the actual build command requested (acting as a make wrapper), the build step can be skipped using the -n or --no-build options.

compiledb forwards all the options/arguments passed after make subcommand to GNU Make, so one can, for example, generate compile_commands.json using core/main.mk as main makefile (-f flag), starting the build from build directory (-C flag):

```
---
# STM32F446 Nucleo Perpherial Config
## UART
1. Basic method, poll status
2. Interrupt handler
3. DMA Transfer
## DMA
For the uart, there are two interrupts happened, half transmit and transmit completed.
https://stackoverflow.com/questions/43298708/stm32-implementing-uart-in-dma-mode
From up link, has a clear explaination about how it works.

## Timer
The Basic Timers (BT) TIM6, TIM7, TIM14, etc (1°) are the most simple timers available in the STM32 portfolio.

The BT are 16 bit timer.

The BT are UP timer only.

The BT my be used in DMA and/or under Interrupt.

The BT has the capabilities show below.

Plese refer to the AN4013 for more info on the STM32 Timers.
![20200324140124.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200324140124.png)
![20200324140209.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200324140209.png)

For this application, use basic timer modes.

Hardware timers keep counting up or down depending on the module until the timer period is reached. Then the timer is reset:
![update events.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200324134420.png)

We will use the timer to keep our LED on 80% of the time by setting a period of 500, turning it on when the counter reaches 400 and turning it off when it reaches 500:
![20200324134510.png](https://markdown-picbed.oss-cn-beijing.aliyuncs.com/img/20200324134510.png)

http://www.emcu.eu/stm32-basic-timer/ make a good explanation about the timer.

## CAN
Basic knowledge at here
https://www.notion.so/c2real/CAN-Basic-Knowledge-22b2ce481c124ec38f41f09ae16f0a74

# CANOpen Protocol
https://wenku.baidu.com/view/1bd617d081eb6294dd88d0d233d4b14e85243e96.html

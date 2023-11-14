

# Modbus library for STM32 Microcontrollers

**This is a fork of [MODBUS-STM32-HAL-FreeRTOS](https://github.com/alejoseb/Modbus-STM32-HAL-FreeRTOS). This is a more stripped down version that includes more modbus functions (reading read only coils and registers)**

USART and USB-CDC Modbus RTU Master and Slave library for STM32 microcontrollers 
based on Cube HAL and FreeRTOS.

Includes a project for a Nucleo-P-WB55 project. Includes defines for other STM32 boards. 

This is a port of the Modbus library for Arduino: https://github.com/smarmengol/Modbus-Master-Slave-for-Arduino

`NEW` Script examples to test the library based on Pymodbus

## Characteristics:
- Portable to any STM32 MCU supported by ST Cube HAL.
- Portable to other Microcontrollers, like the [Raspberry PI Pico](https://github.com/alejoseb/Modbus-PI-Pico-FreeRTOS), requiring little engineering effort.
- Multithread-safe implementation based on FreeRTOS. 
- Multiple instances of Modbus (Master and/or Slave) can run concurrently in the same MCU,
  only limited by the number of available UART/USART of the MCU.
- RS232 and RS485 compatible.
- USART DMA support for high baudrates with idle-line detection.
- USB-CDC RTU master and Slave support for F103 Bluepill board. 


## File structure
```
├── LICENSE
├── README.md
├── MODBUS_WB55_SLAVE_RTOS_DMA --> Project folder for the STM32WB55 Nucleo-P board, with freeRTOS, DMA and RS485 enabled.
├── MODBUS-LIB --> Library Folder
    ├── Inc
    │   └── Modbus.h 
    ├── Config
    │   └── ModbusConfigTemplate.h --> Configuration Template
    └── Src
        ├── Modbus.c 
        └── UARTCallback.c
 
```
## How to use the examples
Examples provided for STM32CubeIDE Version: 1.8.0 https://www.st.com/en/development-tools/stm32cubeide.html.

- Import the examples in the STM32Cube IDE from the system folder
- Connect your NUCLEO board
- Compile and start your debugging session!
- If you need to adjust the Baud rate or any other parameter use the Cube-MX assistant (recommended). If you change the USART port you need to enable the interrupts for the selected USART. Check UARTCallback.c for more details.

### Notes and Known issues :
- The standard interrupt mode for Modbus RTU USART is suitable for 115200 bps or lower baud rates. 
For Higher baud rates---tested up to 2 Mbps---it is recommended to use the DMA mode. Check the corresponding examples. It will require 
extra configurations for the DMA channels in the Cube HAL.

- The USB-CDC example supports only the Bluepill development board. It has not been validated with other development boards.
To use this example, you need to activate USB-CDC in your ModbusConfig.h file.

## How to port to your own MCU
- Create a new project in STM32Cube IDE for your MCU
- Enable FreeRTOS CMSIS_V2 in the middleware section of Cube-MX
- Configure a USART and activate the global interrupt
- If you are using the DMA mode for USART, configure the DMA requests for RX and TX
- Configure the `Preemption priority` of USART interrupt to a lower priority (5 or a higher number for a standard configuration) than your FreeRTOS scheduler. This parameter is changed in the NVIC configuration pane.
- Import the Modbus library folder (MODBUS-LIB) using drag-and-drop from your host operating system to your STM32Cube IDE project
- When asked, choose link folders and files
- Update the include paths in the project's properties to include the `Inc` folder of MODBUS-LIB folder
- Create a ModbusConfig.h using the ModbusConfigTemplate.h and add it to your project in your include path
- Instantiate a new global modbusHandler_t and follow the examples provided in the repository 
- `Note:` If your project uses the USART interrupt service for other purposes you have to modify the UARTCallback.c file accordingly


## Recommended Modbus Master and Slave testing tools for Linux and Windows

### Master and slave Python library

Linux/Windows: https://github.com/riptideio/pymodbus

### Master client Qmodbus
Linux:    https://launchpad.net/~js-reynaud/+archive/ubuntu/qmodbus

Windows:  https://sourceforge.net/projects/qmodbus/

### Slave simulator
Linux: https://sourceforge.net/projects/pymodslave/

Windows: https://sourceforge.net/projects/modrssim2/

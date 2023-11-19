# MODBUS STM32 HAL - Master and Slave configuration  documentation

---

>This is code documentation for a MODBUS library running on a STM32 using FreeRTOS. To get better understanding of the MODBUS protocol it is advised to look at [Modbus Protocol](url) first.

## Configuration
The library allows multiple instances of Modbus to run. To allow for this and for more robust/ease of use a structure is used with the **modbushandler_t** type. This structure houses configuration information, register locations, timers, semaphores and the task handle.

Before calling the initialization function, the Modbus instance needs to be configured. Below is an example...
```C
modbushandler_t ModbusH;          //declaring a modbus handler/instance
...
...
int main(void){
...
...
	ModbusH.uModbusType = MB_SLAVE;   //declaring it as a slave device
	ModbusH.port = &huart1;           //which UART port is used
	Modbus.u8id = 1;                  //device ID, 1 to 247. Master is 0
	Modbus.u16timeout = 1000;         //hwo long is the timeout
	ModbusH.EN_PORT = DE_EN_GPIO_PORT;//flow control port (if used, otherwise use NULL)
	ModbusH.EN_Pin = DE_EN_Pin;       //flow control pin (if used, otherwise use NULL)
	ModbusH.u16regs = ModbusDATA;     //register which stores data
	ModbusH.u16regsize = sizeof(ModbusData)/sizeof(ModbusDATA[0]);
	ModbusH.xTypeHW = USART_HW_DMA    //DMA enabled, can also be USART_HW, if DMA is not needed
}
```

During initialization functions the rest of the structure is connected with the declared semaphore, task and timer(s). To see the full structure see the [Modbus.h](url) file


## Block Diagram

Below is a diagram that shows the main points/working of **ModbusInit( )**, **ModbusStart( )** the Modbus thread/task **StartTaskModbus( )** and the UART callbacks.

- **ModbusInit( )** - is the initialization function which defines the necessary thread, semaphore and timer based on the Modbus configuration done in the *main.c* file, mainly on whether the user configured a *slave* or *master* device. 

It should be noted that the diagram doesn't cover the differences for a master device. While most of it is the same, a configured master device has a different thread, a queue and an additional timer for the timeout.

- **ModbusStart( )** - goes over the rest of the configurations done in the *main.c* file and additionally checks for errors, in case a mistake was made in the configuration. Depending on the configuration, if DMA is enabled, the appropriate function is called, **HAL_UARTEx_ReceiveToIdle_DMA**, otherwise **HAL_UART_Receive_IT** is called.

In case DMA is enabled, half-transfer is disabled by calling **__HAL_DMA_DISABLE_IT(modH->port->hdmarx, DMA_IT_HT)**, because the error checking field *(Cyclical Redundancy Check for Modbus RTU)* is at the very end. Which needs to processed in order to determine if the message has any missing information.


![](https://i.imgur.com/j5njxIp.png)

The above diagram showcases the thread for a slave configuration.


## Modbus Thread

### Slave Configuration

**ulTskNotify** is used to block the thread indefinitely. The task is unblocked in the UART callback function by using **xTaskNotifyFromISR** to notify the thread. 

After the thread is awoken it checks the message ID to see if it is for the slave device. It will call a function to validate/calculate the **CRC** value, to see if the message has any missing bits lost in transmission. 

If everything is in order, the thread will take the semaphore and process the requested function and send an answer. After which it will release the semaphore and go back in to a blocked state.

---

### Master Configuration
The workings for a device that is configured as a master is a bit more complicated. A thread will use either **ModbusQuery** or **ModbusQueryInject** function. It will pass it the necessary information like the slave ID, function code, requested register and count. 

The function will then send a queue message with this information to the Modbus Master thread **(StartTaskModbusMaster)**.

After receiving a queue message the master thread will use function **SendQuery** to execute the requested function with the information it got provided and send it to the slave device. 

The master thread will go in a blocked state temporarily till either an answer comes back from a slave device on the Tx buffer or a timeout happens *(a slave device hasn't responded in specified amount of time)*. If a message has been received, the timer is stopped, and the message is then validated. The semaphore is the taken and the message is processed and the task that requested a Modbus query is then notified and the semaphore is released.

It should be noted that the library has quite a bit of error checking for both sklave and master configurations.


app_uart library example with low power mode
==================

This project contains code examples that show nrf51 UART functionality with the app_uart library in low power mode.
There are in fact two projects included, one is the actual low power exampel and the other example is for testing on a nRF51 UART peer device:

ble_app_uart_low_power_with_buffers (UART slave)
------------
This project is configured for low power operation.
 
When the UART on the nRF51 is constantly enabled, it will consume a lot of current, ~1mA constantly. The trick is to only enable the UART when data transmission is needed. There are primarily two methods for achieving this: 

- **Configure two GPIO control pins.** This will require the nRF51 application to manually signal the UART peer device when it is safe to send data. Similarly, the UART peer device will have to signal the nRF51 when it is safe to send data to the UART peer device. 

- **Enable the UART hardware flow control in LOW_POWER mode**, which this example does. This LOW POWER example will have hardware flow control enabled in a specific LOW POWER mode where the nRF51 is woken up and the UART is enabled when CTS goes low. nRF51 will set RTS low when it is ready to receive data. So the peer UART device must set nRF51 CTS low when sending data to nRF51 and set it high again when finished transmitting data.
Sleep current is ~3uA when the softdevice is enabled and the nRF51 is sleeping. With the UART in LOW POWER mode the nRF51 will therefore consume ~3uA + UART current consumption. How much the UART will consume depends on the amount of data the nRF51822 will have to receive and the BAUD rate chosen, the higher the BAUD rate, the less the current consumption. 

ble_app_uart_low_power_with_buffers_peer_device (UART master)
------------
This project is configured for normal operation and is also low power, as it will only enable the UART when there is data to send. It has modified RTS funtionality so that it will work with the UART slave low power example. The modified RTS functionality consists of setting RTS low before transmitting data and setting it high when data transmission is finished. Since the UART master is in control of when UART data is transmitted and when not, it might be desirable to have the UART master periodically enable transmission (by setting the UART slave CTS line low) in order to give the UART slave a chance to send data to the UART master.

Current consumption
------------
- The current consumption of the low power applications (both UART master and UART slave) is about 700uA when advertising
- After connection is established, the current consumption is around 450uA if the Master Control Panel connects with its default 20ms connection interval
- 5 seconds after the connection establishment and enabling services, the low power applications will send a connection parameter update request to the central device (Master Control Panel), which results in a connection parameter update
- The connection parameters are updated to 100ms connection interval and slave latency of 4, so packets will be sent every 500ms, which results in a current consumption of about 25uA
- The user should modify the connection parameters for his own requirements, in order to improve response time, or to lower current consumption even further
- Description of connection parameters and how to modify them in the application are given on https://devzone.nordicsemi.com/question/5186/how-to-minimize-current-consumption-for-ble-application-on-nrf51822/

Test
------------
- Flash softdevice S110 v7.0.0 to two development boards or two evaluation boards
- Flash the ble_app_uart_low_power_with_buffers (UART slave) example to one board and ble_app_uart_low_power_with_buffers_peer_device (UART master) to the other board
- Connect the UART pins on the two boards together. To know what pins to hook together, look at the UART pin configuration in each example
- Connect a PCA10001 BLE dongle with master emulator firmware and open the Master Control Panel. Select Nordic_UART device (this is the device running the UART slave example), perform Service Discovery and enable services
- Connect a second BLE dongle with master emualtor firmware and open a second Master Control Panel. Select UART_master device, perform Service Discovery and enable services
- In the Master Control Panel connected to the UART master, select the UART RX characteristic, write characters in the Value field and press Write button. When you have written 20 characters, you should see those characters appear in the UART TX characteristic on the Master Control Panel connected to the UART slave device.
- Similarly, write to the UART RX characteristic in the Master Control Panel connected to the ble_app_uart_low_power_with_buffersd device to transfer data in the opposite direction. This data will however not come through unless you write data from UART master to UART slave, because the UART master enables the UART slave only when data is transmitted from UART master to UART slave. This could however be modified by letting the UART master periodically enable the UART slave by periodically setting the RTS line low. 
 
Requirements
------------
- nRF51 SDK version 6.1.0
- BLE S110 softdevice version 7.0.0
- nRF51822 Development Kit version 2.1.0 or later

The project may need modifications to work with other versions or other boards. 

To compile it, clone the repository in the \nrf51822\Board\nrf6310\ folder.

About this project
------------------
This application is one of several applications that has been built by the support team at Nordic Semiconductor, as a demo of some particular feature or use case. It has not necessarily been thoroughly tested, so there might be unknown issues. It is hence provided as-is, without any warranty. 

However, in the hope that it still may be useful also for others than the ones we initially wrote it for, we've chosen to distribute it here on GitHub. 

The application is built to be used with the official nRF51 SDK, that can be downloaded from https://www.nordicsemi.no, provided you have a product key for one of our kits.

Please post any questions about this project on https://devzone.nordicsemi.com.

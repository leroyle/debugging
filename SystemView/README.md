### Segger SystemView - Analyzing Embedded Systems
“SystemView is a real-time recording and visualization tool for embedded systems that reveals the true runtime behavior of an application, going far deeper than the system insights provided by debuggers.”  
 
#### Features
* Continuous real-time recording of an embedded system
* Capture tasks, interrupts, timers, resources, API calls, and user events
* Recording via J-Link and SEGGER RTT Technology, IP, or UART
* Live analysis and visualization of captured data
* Minimally system intrusive
* Works on any CPU
* Works with any RTOS and bare-metal systems
* SEGGER embOS, emNet, and emFile API call tracing as standard
* uC/OS-III, Micrium OS Kernel, and FreeRTOS instrumentation included
* Free for non-commercial use without limitation
* Supported on Windows, Mac, and Linux.

#### Systemview Download Page
https://www.segger.com/downloads/systemview

### J-Link Communications Speed
The speed with which J-Link communicates with the device is configurable. For best operation it is strongly suggested that the J-Link communications speed be set to the same values within all products that interface with it, that include the debug device itself, PlatformIO, and System View. This is especially important if you attach more than one process to the J-Link device, for instance have an active PlatformIO session and SystemView GUI, 
The following how to's will set the speed to 30000 for each product:  

#### Set SystemView Speed
It is recommended you do this for best experience especially if you are seeing buffer overflows within yhr SystemView GUI  
at the SystemView Opening Window:
* open SystemView
* select “Target” → “Recorder Configuration”
* if not selected, select J-Link, then click “Ok”
* on the “Recorder Configuration” dialog, set “Interface Speed to : 30000
* click “Ok” to dismiss the dialog

#### Set PlatformIO Speed
##### Enable SystemView:
To enable Segger SystemView support within your project you must add a compile flag to your projects platformio.ini file  
add the CFG_SYSVIEW define to the build_flags as in:  
* build_flags = -DCFG_SYSVIEW=1
##### Set the debug communications parameters
The following should be set in the platformio.ini file for your target device. This set the debug/upload tool and communications speed  
* upload_protocol = jlink
* debug_tool = jlink
* debug_speed = 30000

##### Custom main.cpp support
If you roll your own main() startup function you will need to add some SystemView specific code in order to enable SystemView support within the device application.  
For the current WisBlock runtime the default device application startup file can be found at:
```
.platformio/packages/framework-arduinoadafruitnrf52/cores/nRF5main.cpp 
```
This file has the required conditional compile statements that will initialize and startup the device side system view functionality. If you supply your own custom main.cpp startup code you must incorporate the system view code as found in the default main.cpp.

#### Set J-Link Device Speed
The interface speed for the J-Link device itself is set via the J-Link configuration program which is delivered with the J-Link software package for your development platform. [https://www.segger.com/downloads/jlink/]  
To set the interface speed run the J-Link configuration tool, JlinkConfigExe  

* connect the J-Link device via USB to your developement platform
* open up the configuration tool
* select your J-Link device from the list under "Connected via USB", right click the mouse, select "Configure"
* set the "Max. SWO speed [kHz]" to 30000 
* Click the "OK" button
* Refer to the on screen instructions that request you unplug and reconnect the debug device before clicking the final "OK"
* dimiss the J-Link configuration app   
 NOTE: the interface speed for the non-professional versions of J-Link are speed limited, eg: EDU is limited to 30,000

#### Increase SystemView Buffer Size in Device App
If SystemView is reporting buffer overflow errors you may need to increase SystemView buffer space within the device application. In:
```
.platformio/packages/framework-arduinoadafruitnrf52/cores/nRF5/sysview/Config/SEGGER_SYSVIEW_Conf.h
```
change:
```
#define SEGGER_SYSVIEW_RTT_BUFFER_SIZE        1024
```
to
```
#define SEGGER_SYSVIEW_RTT_BUFFER_SIZE        (8 * 1024)
```

#### SystemView GUI
##### Display Mapping Message Numbers to String Names
To help cut down the amount of trace data transmitted from the device application to SystemView GUI application on the development machine SystemView uses #define variables to associate device applicaiton messages to their origins.  
By default the SystemView GUI has no mapping of the #define number to any string value for human consumption. For example, ISRs (Interrupt service routine) when presented within the GUI will contain the MCU specific ISR number rather than a string name.  
To make the output more user friendly there are mechanisms to allow the SystemView GUI to map the numberic values to string names.
##### Mapping ISRs
To get the names of the ISR’s to show in SystemView
1. Find out which ISR is being handled and what it's number is by setting a debug breakpoint in  
```
2.~/.platformio/packages/framework-arduinoadafruitnrf52/cores/nRF5/sysview/SEGGER/SEGGER_SYSVIEW.c
at SEGGER_SYSVIEW_RecordEnterISR()
```
3. When the break point is hit, check the return from SEGGER_SYSVIEW_GET_INTERRUPT_ID()  
It should be one of the ISR numbers reported by SystemView.
4. check the stack trace to see what Interrupt it is associated with.
5. edit the following file within the device runtime:  
``` 
.platformio/packages/framework-arduinoadafruitnrf52/cores/nRF5/sysview/Config/SEGGER_SYSVIEW_Config_FreeRTOS.c
```
at the line
```
SEGGER_SYSVIEW_SendSysDesc("I#15=SysTick")
```
add your missing IRQ’s, in this case we map IRQ 22, 33 and 55
```
SEGGER_SYSVIEW_SendSysDesc("I#15=SysTick, I#22=GPIOTE,I#33=RTC1,I#55=USBD");
```
6. Rebuild and reflash, re-run your device application
7. Startup SystemView again, the IRQ’s number should now be mapped to string names.

##### Performance Markers
Performance markers let you customize the instrumentation of your code. 
You can tag the start and end of a section of code to determine how long it takes that code to execute.
There are two SystemView API's that must be placed within your device application, 
```
* SEGGER_SYSVIEW_MarkStart(unsigned int MarkerId)
* SEGGER_SYSVIEW_MarkStop(unsigned int MarkerId)
```
* * Note: Some of the Segger documentation may refer to these as XXXMarkerStxx, rather than XXXMarkerStxx, note the extra "er" in the documentation name.

* SystemView will by default ouput the marker messages using the passed in MarkerId number.  
 To map number to text you must call the API:
```
SEGGER_SYSVIEW_NameMarker(Id, “Text_To_Display”);
```


####  My Application Tasks: 
At this point I've only used system view to try to track task execution. These are the typical tasks I see within SystemView for my device application using the Wisblock and Adafruit runtimes  

* ISR 33: -> timer task
* ISR 55: --->  timer task
* SysTick: ---> SysTick timer task
* Scheduler: --->  task scheduler
* usbd:  prio: 3 --> usb handler, called when you using prints
* Callback: prio: 2 --> I do not see were this is used,  
 SystemView does not show it used Adafruit had limited references, looks to be BLE callback related         https://forums.adafruit.com/viewtopic.php?f=53&t=136808&p=678116&hilit=Am
* Idle: --> RTOS required idle task, runs when there is nothing else to do
* loop: prio:  1  --> main user loop,  
 handles all of the normal overhead, the MySensors.org interface, packing the sensor data for the LoRaWan processing, addes the message to the RTOS message queue. ( should separate the MySensors data handling into another task )
* LORA: prio: 2  --> Internal to run Radio.IrqProcess(),  
new as of  version 2 of SX126x-Arduino, in V1 it was called a part of the user loop()
* LoRaTsk: prio1  -->  my LoRa task,  
 waits for an available message in the message queue, handles the users LoRaWan communictionations,  
 not to be confused with the runtimes handling of the LoRaWan protocol.


### A Couple YouTube System View Videos
* Jacob Beningo
https://www.youtube.com/watch?v=1KQ0647NzCo

* Kiran Fastbit Embedded Brain Acadamy  
His SystemView primer:
https://www.youtube.com/watch?v=FklzdUt97gE

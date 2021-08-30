
### Who am I
* Retired Enterprise app programmer
worked with proprietary client/server network interconnect piece of a large CASE tool. (TI IEF) 
* 10 years hardware tech back in the dark ages, FAA Air Traffic control computers
* 15 years software side, 6 as contract programmer in the Unix world
* 8 years retired “playing with IOT”
* Very subject to “Shiney Object Syndrome”   https://en.wikipedia.org/wiki/Shiny_object_syndrome

Shiny object syndrome is the situation where people focus all attention on something that is current and trendy, yet drop this as soon as something new takes its place. 

Hackster.io projects
https://www.hackster.io/dashboard/projects
6 projects either personal or collaboration projects

IOT for Good project  ( year 8 of a 5 year project)
https://www.hackster.io/leroy2le/rv-security-protection-via-the-helium-network-5ca636
Almost got it working in time, but no cigar


1.  Overview of project
- monitor various systems within an RV
- temperature
- water line leak  (faucet, shower, toilet, mains, storage tank overflow)
- roof water leak
- water storage tank levels ( Fresh and waste)
- noise levels
- fire/propane

- Acutators
- Heat/A/C levels
- main water disconnect
- if leak/overflow detected
- when leaving coach

Monitoring requirements
- self contained within the coach, cannot depend on cloud connection for disaster mitigation
- auto shutoff in the case of water detected
- auto adjust of temperature if out of range
- single point of cloud configuration. Do not want to manage multiple LoRaWan connections.


Technology
Linux Ubuntu with VSCode and PlatformIO
- PlatformIO much simpler to get off the ground than say generic Eclipe
- More features than ArduinoIDE
- not as full featured as say “Atollic True Studio”, Keil, or some of the for cost IDE’s


- MySensors.org:
MySensors is an open source hardware and software community focusing on do-it-yourself home automation and Internet of Things. 

This framework is used for sensor montioring/control using NRF51822 SOC’s on a custom prototype PCB. Each sensor type (Temp, water, leak, actuator) is controlled by a single custom programmed NRF51. 
The framework uses the 2.4 Ghz BLE frequency and chips but does not use a BLE stack, rather it’s a proprietary transmission protocol.
These controllers are not LoRaWan aware.


- supports multiple radio technologies (NRF5 2/1, NRF2401, RFM69, RFM95, RS485)
- Does not appear to yet support LoRa/LoRaWan
-multipe cloud contoller platforms (Domoticz, HomeGenie, Home Assistant) 
- multiple cloud transports (Ethernet, MQTT, Serial)
- No support LoRaWan as yet
 multiple MCU boards, (AVR, ESP32, Linux, NRF5, SAMD, STM32F1, teensy3)


LoRaWan
- We needed to merge the MySensors 2.4Ghz frequency/protocol with the sub gigahertz LoRaWan frequency/protocol.
- first iteration used serial interface from NRF51 to  the STMicroelectronics B-L072Z-LRWAN1 Discovery which was the first iteration of the Helium developer kit.
- Serial was somewhat problematic, relatively slow, required a third radio technology, HC-12 serial protocol board.
- Looked from a better solution, then Shiney Object Syndrome kicked in with the WisBlock dev kit.
- Has on board 2.4 Ghz radio in addition to the LoRaWan functionality.
- Much easier merging the two frequency/protocols

RTOS vs Bare Metal super loop
- after a bit of poking around under the covers I discovered that the underlying Adafruit library that Wisblock depends on is implemented using a version of FreeRTOS. Shiney Object Alert


Bare Metal
- all functionality passes through the super loop, interrupts excepted of course
- if one subsection takes too long in execution, other time critical subsections can suffer
- can use rudimentary time based scheduling, but one must return to the super loop to effect the subsection execution change
- if you have diparate timing of supplier/consumers of data one must roll your own circular buffer type of mechanism to account for data burts

RTOS
- much like thread programming in a desktop environment
- subsections of functionality can be separated into distinct functions
- task can be created and allowed to terminate if desired
- each task has it’s own private stack space, data/instructions can be shared among tasks
- time critical subsections will be given the opportunity to run if preemptive scheduling is enabled
- RTOS frameworks supplies inter-task communictionations,
    • RTOS Task Notifications
    • Stream and Message Buffers
    • Queues
    • Binary Semaphores
    • Counting Semaphores
    • Mutexes
    • Recursive Mutexes

- message queues can handle the bursty data, supplier can fill, consumer will wait for data
-

Tasks:
 minimum of 1 task, idle task that runs when no other task is available to run. Started by FreeRTOS framework
 

SystemView: 
These are the typical tasks I see within SystemView for my device application using the Wisblock/ and Adafruit runtimes

 ISR 33:   timer task
 ISR 55:   timer task
 SysTick:  SysTick timer task
 Scheduler:   task scheduler
 usbd:  prio: 3  usb handler, called when you using prints
 Callbac: prio: 2 I do not see were this is used, SystemView does not show it used
         Adafruit had limited references, looks to be BLE callback related         https://forums.adafruit.com/viewtopic.php?f=53&t=136808&p=678116&hilit=Am
 Idle:    RTOS required idle task

loop:  prio:  1  main user loop, handles all of the normal overhead, the MySensors.org interface,
 packing the sensor data for the LoRaWan processing, addes the message to the RTOS message queue. ( should separate the MySensors data handling into another task )

 LORA: prio: 2  Internal to run Radio.IrqProcess(), new as of  version 2 of SX126x-Arduino,
                           in V1 it was called a part of the user loop()
 LoRaTsk: prio1   my LoRa task, waits for an available message in the message queue, handles the users LoRaWan communictionations, not to be confused with the runtimes handling of the protocol.

RTOS Issues
- task stack overflow, stack size is adjustable in the create call
- interrupt response time, limit time spent in interrupts as usual
- some RTOS API’s have/require special forms for executing within interrupts  xxx_ISR
- priority inversion low priority task has a lock a highest priority task needs, thus the highest priority task cannot run until the lowest is completes and releases the lock.
 In the mean time medium priority tasks may preempt the lowest lowest priority task preventing it from getting time to complete and releasing the lock,  the highest priority task is esentially starved.
- deadlocks  task A has a lock on a resource task B needs, task B takes a lock on a resource task A needs. Neither can run until one releases it’s resource lock


RTOS Debugging with J-Link and Segger SystemView
Debugging
1. The following applied to J-Link debug adapter only, it is not known how much of this can be accomplished with a an ST-Link adapter.
NOTE:  Segger has a procedure for updating certain ST-Link adapters found on Discovery and Nucleo dev boards  to be somewhat compatible with J-Link:  https://www.segger.com/products/debug-probes/j-link/models/other-j-links/st-link-on-board/

An education version of Seggers J-Link adapter can be found on various vendor sites including  Newark:    Emulator, J-Link EDU (8.08.90), USB, Inexpensive J-Link for Educational Purpose, it runs about $60 USD.

2. By default the debugger within PlatformIO (Gnu GDB) is not enabled to support RTOS.  Without RTOS support the “GDB” command “info threads” will only show the currently executing task, not all created tasks.
With this RTOS support you will see all tasks displayed 
-  To enable RTOS support an option must be passed to the openocd process when starting debugging.
- this is typically done within the target “.platformio/platforms/???/ platform.py” file.
 for the RAK Wireless WisBlock platform this is done in the file:
.platformio/platforms/nordicnrf52/platform.py
  I
- in this file locate function  	“_add_default_debug_tools(self, board)”
- in jlink section:  “elif link = “jlink”
 to the “arguments list add another argument in the form of
"-rtos", "GDBServer/RTOSPlugin_FreeRTOS",

Segger SystemView
“SystemView is a real-time recording and visualization tool for embedded systems that reveals the true runtime behavior of an application, going far deeper than the system insights provided by debuggers.”
SystemView is supported on Windows, Mac, and Linux. The download can be found at: https://www.segger.com/products/development-tools/systemview/#systemview-media

Device Side:
To enable Segger SystemView support within your project you must add a compile flag to your projects platformio.ini file
- add the CFG_SYSVIEW define to the build_flags as in:
build_flags = -DCFG_SYSVIEW=1

The device application file at .platformio/packages/framework-arduinoadafruitnrf52/cores/nRF5main.cpp 
has conditional compile statements that initialize and startup the device side system view functionality. If you supply your own custom main.cpp startup code you must incorporate the system view code as found in the default main.cpp.
 
J-Link EDU Debugger connections
- the J-Link debugger uses SWD (Single Wire Debug) as the means of communicating with your device. Connections for the Wisblock are located in the LoRa module and are silk screen labeled.
  
J-Tag cable        Rak 4630 LoRa Module
--------------------------------------------------
   Pin 1                    3v3
   Pin 7                    SWDIO
   Pin 9                    SWCLK
   Pin 4                    GND


Webinar/Videos/Courses
These are just a couple of folks who have videos on YouTube, there are many/many more to be found there. I’ve found these two guys typically do an excellent job and are easy to follow.

Jacob Beningo
Embedded programming educator, has a number of multi-video courses on Digi Key’s education site:
https://www.designnews.com/continuing-education-center

His SystemView primer:
https://www.youtube.com/watch?v=1KQ0647NzCo

Kiran Fastbit Embedded Brain Acadamy
He has a several embedded courses on Udemy.com the ones I’ve purchased have been real good.
His SystemView primer:
https://www.youtube.com/watch?v=FklzdUt97gE

The RTOS basics video I posted to discord
https://www.digikey.bg/en/maker/projects/what-is-a-realtime-operating-system-rtos/28d8087f53844decafa5000d89608016


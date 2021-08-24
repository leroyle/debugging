# FreeRTOS - Real Time Operating System

- much like thread programming in a desktop environment
- subsections of functionality can be separated into distinct functions
- task can be created and allowed to terminate if desired
- each task has it’s own private stack space, data/instructions can be shared among tasks
- time critical subsections will be given the opportunity to run if preemptive scheduling is enabled
### RTOS frameworks supplies inter-task communictionations,
* RTOS Task Notifications
* Stream and Message Buffers
* Queues
* Binary Semaphores
* Counting Semaphores
* Mutexes
* Recursive Mutexes

- message queues can handle the bursty data, supplier can fill, consumer will wait for data
-

### Tasks:
 minimum of 1 task, idle task that runs when no other task is available to run. Started by FreeRTOS framework
 

### SystemView: 
These are the typical tasks I see within SystemView for my device application using the Wisblock/ and Adafruit runtimes

* ISR 33:   timer task
* ISR 55:   timer task
* SysTick:  SysTick timer task
* Scheduler:   task scheduler
* usbd:  prio: 3  usb handler, called when you using prints
* Callback: prio: 2 I do not see were this is used, SystemView does not show it used Adafruit had limited references, looks to be BLE callback related         https://forums.adafruit.com/viewtopic.php?f=53&t=136808&p=678116&hilit=Am
* Idle:    RTOS required idle task, runs when there is nothing else to do
* loop:  prio:  1  main user loop, handles all of the normal overhead, the MySensors.org interface, packing the sensor data for the LoRaWan processing, addes the message to the RTOS message queue. ( should separate the MySensors data handling into another task )
* LORA: prio: 2  Internal to run Radio.IrqProcess(), new as of  version 2 of SX126x-Arduino, in V1 it was called a part of the user loop()
* LoRaTsk: prio1   my LoRa task, waits for an available message in the message queue, handles the users LoRaWan communictionations, not to be confused with the runtimes handling of the protocol.

RTOS Issues
* task stack overflow, stack size is adjustable in the create call
* interrupt response time, limit time spent in interrupts as usual
* some RTOS API’s have/require special forms for executing within interrupts  xxx_ISR
* priority inversion low priority task has a lock a highest priority task needs, thus the highest priority task cannot run until the lowest is completes and releases the lock. In the mean time medium priority tasks may preempt the lowest lowest priority task preventing it from getting time to complete and releasing the lock,  the highest priority task is esentially starved.
* deadlocks  task A has a lock on a resource task B needs, task B takes a lock on a resource task A needs. Neither can run until one releases it’s resource lock

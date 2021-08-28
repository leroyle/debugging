# FreeRTOS - Real Time Operating System

- much like thread programming in a desktop environment
- subsections of functionality can be separated into distinct functions rtos calls tasks
- task can be created, self terminated or terminated by other tasks, suspended, blocked, 
- each task has it’s own private stack space, data/instructions can be shared among tasks
- minimum of 1 task, idle task that runs when no other task is available to run, started by FreeRTOS framework
- FreeRTOS kernel was originally developed by Richard Barry around 2003
- supports multiple microcontroller platforms (~35 according to Wiki)
- Since version 10.0.0 in 2017, Amazon has taken stewardship of the FreeRTOS code
- www.freertos.org
 

#### Scheduling
- FreeRTOS kernel supports two types of scheduling policy:
- usual sheduling policy is known as "Prioritized Preemptive Scheduling with Time Slicing".
##### Prioritized Preemptive
- High priority tasks run before lower priority
- High priority tasks can preempt lower priority tasks
- lower priority tasks could, in theory, be starved out by higher priority tasks 
##### Time slicing
Equal priority tasks are given an equal slice of time
- time slicing can be disabled, to allow a given task run to completion if it does not call a blocking function
- equal priroty tasks must wait until currently running task relinquishes control

### RTOS API categories
* Configuration
* Task Creation
* Task Control
* Kernel Control
* Task Notifications
* Task Utilities
* FreeRTOS-MPU Specific
* Queue Management
* Queue Sets
* Semaphores
* Software Timers
* Event Groups
* Stream Buffers
* Message Buffers
* Co-routine specific 

#### RTOS frameworks supports inter-task communictionations,
* Message Queues (can handle the bursty data inter-task data transfer, producer can fill, consumer will wait for data, think circular buffer) 
* Binary Semaphores
* Counting Semaphores
* Mutexes
* Recursive Mutexes
* Stream and Message Buffers
* Suspend/Resume tasks



RTOS Issues
* task stack overflow, stack size is adjustable in the create call
* interrupt response time, limit time spent in interrupts as usual
* some RTOS API’s have/require special forms for executing within interrupts  xxx_ISR
* priority inversion low priority task has a lock a highest priority task needs, thus the highest priority task cannot run until the lowest is completes and releases the lock. In the mean time medium priority tasks may preempt the lowest lowest priority task preventing it from getting time to complete and releasing the lock,  the highest priority task is esentially starved.
* deadlocks  task A has a lock on a resource task B needs, task B takes a lock on a resource task A needs. Neither can run until one releases it’s resource lock

A technique for handling interrupts:
* https://www.youtube.com/watch?v=0lX6OERAwsM
* short story, create another task for the job rather than having the super loop get around to it.
*  create auxillary task
*  suspend immediately within task loop
*  interrupt ISR enables suspended task via portYIELD_FROM_ISR()


## Learn More via Webinars, Videos, Online Courses
These are just a couple of folks who have videos on YouTube, there are many/many more to be found there. I’ve found these two guys typically do an excellent job and are easy to follow.

* Jacob Beningo
Embedded programming educator, has a number of multi-video courses on Digi Key’s education site:  
https://www.designnews.com/continuing-education-center  
His YouTube playlist:  
https://www.youtube.com/channel/UC9k8GahBTE0IVJxOsL4WhOA/videos

* Kiran Fastbit Embedded Brain Acadamy  
He has a several embedded courses on Udemy.com the ones I’ve purchased have been real good.  
YouTube playlist:  
https://www.youtube.com/channel/UCa1REBV9hyrzGp2mjJCagBg/playlists  

* Udemy.com
* Lynda.com
* YouTube

#### The RTOS basics video I posted to discord
https://www.digikey.bg/en/maker/projects/what-is-a-realtime-operating-system-rtos/28d8087f53844decafa5000d89608016

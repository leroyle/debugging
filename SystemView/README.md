# SystemView

### Systemview Download
https://www.segger.com/downloads/systemview

### J-Link Communications Speed
For SystemView
- open SystemView
- select “Target” → “Recorder Configuration”
- if not selected, select J-Link, then click “Ok”
- on the “Recorder Configuration” dialog, set “Interface Speed to : 30000
- click “Ok” to dismiss the dialog
- 
Segger SystemView
“SystemView is a real-time recording and visualization tool for embedded systems that reveals the true runtime behavior of an application, going far deeper than the system insights provided by debuggers.”
SystemView is supported on Windows, Mac, and Linux. The download can be found at: https://www.segger.com/products/development-tools/systemview/#systemview-media

## PlatformIO - Enable SystemView:
To enable Segger SystemView support within your project you must add a compile flag to your projects platformio.ini file
- add the CFG_SYSVIEW define to the build_flags as in:
build_flags = -DCFG_SYSVIEW=1

The device application file at .platformio/packages/framework-arduinoadafruitnrf52/cores/nRF5main.cpp 
has conditional compile statements that initialize and startup the device side system view functionality. If you supply your own custom main.cpp startup code you must incorporate the system view code as found in the default main.cpp.

If you are seeing buffer overflows within SystemView you may need to adjust the Jlink interface speed
at the SystemView GUI
- select “Target” → “Recorder Configuration”
- select “JLink”, click Ok
- at “Recorder Configuration” dialog, increase the Interface speed to match your J-Link interface speed as seen with the JlinkConfigExe tool that is delivered with the J-Link software. eg:  J-Link EDU 30000

JlinkConfigExe:  fire it up, select your J-Link device, note the interface speed, the non-professional versions of J-Link are speed limited, eg: EDU is 30,000

## String Name for ISRs
To get the namesof the ISR’s to show in SystemView
1. set breakpoint in /home/leroy/.platformio/packages/framework-arduinoadafruitnrf52/cores/nRF5/sysview/SEGGER/SEGGER_SYSVIEW.c at SEGGER_SYSVIEW_RecordEnterISR()
- Check the return from SEGGER_SYSVIEW_GET_INTERRUPT_ID()
It should be one of the ISR numbers reported by SystemView.
- check the stack trace to see what Interrupt it is associated with.
- in .platformio/packages/framework-arduinoadafruitnrf52/cores/nRF5/sysview/Config/SEGGER_SYSVIEW_Config_FreeRTOS.c at the line
SEGGER_SYSVIEW_SendSysDesc("I#15=SysTick")
add your missing IRQ’s as in
SEGGER_SYSVIEW_SendSysDesc("I#15=SysTick, I#22=GPIOTE,I#33=RTC1,I#55=USBD");

-re run SystemView, the IRQ’s should now have names.

## Performance Markers
-   Markers lets you custom instrument your code. You can tag the start and end of the instrumentation period via SEGGER_SYSVIEW_MarkerStart(Id) and  SEGGER_SYSVIEW_MarkerStart(Id), SystemView by default is to give these an id number. To map number to text you must call
SEGGER_SYSVIEW_NameMarker(Id, “Text_To_Display”);

### Couple YouTube System View Videos
* Jacob Beningo
https://www.youtube.com/watch?v=1KQ0647NzCo

* Kiran Fastbit Embedded Brain Acadamy  
His SystemView primer:
https://www.youtube.com/watch?v=FklzdUt97gE

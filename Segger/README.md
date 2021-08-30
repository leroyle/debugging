# Segger Debugging with J-Link
Debugging
### 1. With J-Link 

The following assumes you are using PlatformIO as a development IDE and applies to J-Link debug adapter only, it is not known how much of this can be accomplished with a an ST-Link adapter.

NOTE:  Segger has a procedure for updating certain ST-Link adapters found on Discovery and Nucleo dev boards  to be somewhat compatible with J-Link:  https://www.segger.com/products/debug-probes/j-link/models/other-j-links/st-link-on-board/

An education version of Seggers J-Link adapter can be found on various vendor sites including  Newark:    Emulator, J-Link EDU (8.08.90), USB, Inexpensive J-Link for Educational Purpose, it runs about $60 USD.

###  2. Enable RTOS support
By default the debugger within PlatformIO (Gnu GDB) is not enabled to support RTOS.  Without RTOS support the “GDB” command “info threads” will only show the currently executing task, not all created tasks.
With this RTOS support you will see all tasks displayed 
-  To enable RTOS support an option must be passed to the openocd process when starting debugging.
- this is typically done within the target “.platformio/platforms/???/ platform.py” file.
 for the RAK Wireless WisBlock platform this is done in the file:
.platformio/platforms/nordicnrf52/platform.py
  
- in this file locate function  	“_add_default_debug_tools(self, board)”
- in jlink section:  “elif link = “jlink”
 to the “arguments list add another argument in the form of
"-rtos", "GDBServer/RTOSPlugin_FreeRTOS",

### Communication Speed Definition
It is important that Jlink, SystemView and PlatformIO all use the same communication speed definition if one is to get optimal performance and fewer SystemView buffer overflows.  30 Khz seems to be the max for the EDU version of J-Link.
The speed is set differently for each component
#### For PlatformIO
the binary upload to target speed is set in platformio.ini as a debug option:
	debug_speed = 30000
#### For SystemView
- open SystemView
- select “Target” → “Recorder Configuration”
- if not selected, select J-Link, then click “Ok”
- on the “Recorder Configuration” dialog, set “Interface Speed to : 30000
- click “Ok” to dismiss the dialog

#### For the J-Link Debug module
-  plug in the J-Link debug module
- launch the jlink configuration tool that is delivered with the J-Link software installation, on Linux this file is called “JlinkConfigExe”
- when the configuration dialog comes up you should see the J-Link tool enumerated under the “Connected via USB” section
- right click on the J-Link entry, and select “Configure”
- set Max. SWO speed [kHz] to 30000
- select Ok, then follow the prompt which asks you to unplug, re-plug in the J-Link module before pressing the “Ok” button. ( I don’t know the consequence if you fail to do this)


#### ST-Link conversion to J-Link compatible debugger
The above applies to J-Link debug adapter only, it is not known how much of this can be accomplished with a an ST-Link adapter.
NOTE:  Segger has a procedure for updating certain ST-Link adapters found on Discovery and Nucleo dev boards  to be somewhat compatible with J-Link:  https://www.segger.com/products/debug-probes/j-link/models/other-j-links/st-link-on-board/



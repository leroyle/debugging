# Segger Debugging with J-Link
Debugging
## 1. With J-Link 

An education version of Seggers J-Link adapter can be found on various vendor sites including  Newark:    Emulator, J-Link EDU (8.08.90), USB, Inexpensive J-Link for Educational Purpose, it runs about $60 USD.

#  2. Enable RTOS support
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

## ST-Link conversion to J-Link compatible debugger
The above applies to J-Link debug adapter only, it is not known how much of this can be accomplished with a an ST-Link adapter.
NOTE:  Segger has a procedure for updating certain ST-Link adapters found on Discovery and Nucleo dev boards  to be somewhat compatible with J-Link:  https://www.segger.com/products/debug-probes/j-link/models/other-j-links/st-link-on-board/

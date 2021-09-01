#### PlatformIO Demo

##### Projects platformio.ini
````
[platformio]
default_envs = wiscore_rak4631_Gateway

[env:wiscore_rak4631_Gateway]
platform = nordicnrf52
board = wiscore_rak4631
framework = arduino
build_type = debug
; -DCFG_SYSVIEW   ; for adding Segger SysView support to device application
; -DLORA_NOT_AVAILABLE   ; this projects custom define to disable LoRa comm
; -DINCLUDE_xTaskGetHandle
build_flags =  -g -DINCLUDE_xTaskGetHandle -DCFG_SYSVIEW=1 -DLORA_NOT_AVAILABLE -DLIB_DEBUG -DSEND_TEST_MSGS -Wl,-Map=output.map -DCFG_DEBUG=4 
lib_deps =	https://github.com/leroyle/MySensors.git#Mysensors_Lora, beegee-tokyo/SX126x-Arduino
;platform_packages =
;  framework-arduinoadafruitnrf52@https://github.com/leroyle/framework-arduinoadafruitnrf52.git
; platform_packages = toolchain-gccarmnoneeabi@1.70201.0
upload_protocol = jlink
debug_tool = jlink
debug_speed = 30000
````

##### Enable RTOS task support in debugger
We need to enable support in the boards configuration file
```
.platformio/platforms/nordicnrf52/platform.py
```
See the Segger/Readme.md file in this repo for details


##### SystemView Demo
* with run away while loop
* using task suspend

###### SystemView gotchas
* common J-Link communication speed across PlatformIO project, SystemView and the J-Link debug module
* If SystemView is reporting buffer overflow errors you may need to increase SystemView buffer space within the device application.  
See the SystemView/Readme.md in this repo for details  
* I also needed to increase the swap space on my Linux dev platform, (probably unrelated)

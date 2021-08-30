
### Who am I
* Retired Enterprise app programmer
worked with proprietary client/server network interconnect piece of a large CASE tool. (TI IEF) 
* 10 years hardware tech back in the dark ages, FAA Air Traffic control computers
* 15 years software side, 6 as contract programmer in the Unix world
* 8 years retired “playing with IOT”
* Very subject to “Shiney Object Syndrome”   
https://en.wikipedia.org/wiki/Shiny_object_syndrome

#### Hackster.io projects  
* https://www.hackster.io/dashboard/projects  
6 projects either personal or collaboration projects
* IOT for Good project  ( year 8 of a 5 year project)   
https://www.hackster.io/leroy2le/rv-security-protection-via-the-helium-network-5ca636  
Almost got it working in time, but no cigar


###  Overview of pet project
Monitor various systems within an RV
##### Sensors
- temperature in various locations
- water line leak  (faucet, shower, toilet, mains, storage tank overflow)
- roof water leak
- water storage tank levels ( Fresh and waste)
- noise levels
- fire/propane

##### Acutators
- Turn on/off Heat/A/C devices as needed
- main water disconnect if leak/overflow is detected
- main water disconnect when leaving coach
- notification of issues, leaks, over temp, intrusion via email, text, phone call

##### Monitoring requirements
- self contained within the coach, cannot depend on cloud connection for disaster mitigation
- in coach gateway should be able to mitigate a disaster
- auto shutoff in the case of water detected
- auto adjust of temperature if out of range
- single point of cloud configuration.  
Do not want to manage multiple LoRaWan connection per sensor.  
4 temp, 4 tank level, 10 leak, etc


### Development Technology
Linux Ubuntu with VSCode and PlatformIO
- PlatformIO much simpler to get off the ground than say generic Eclipe
- More features than ArduinoIDE
- not as full featured as say “Atollic True Studio”, Keil, or some of the for cost IDE’s

### Frameworks
MySensors.org:  https://www.mysensors.org/  
MySensors is an open source hardware and software community focusing on do-it-yourself home automation and Internet of Things. 
* The framework uses the 2.4 Ghz BLE frequency and chips but does not use a BLE stack, rather it’s a proprietary transmission protocol.  
* supports multiple radio technologies (NRF5 2/1, NRF2401, RFM69, RFM95, RS485)
* Does not appear to yet support LoRa/LoRaWan
* supports multipe cloud contoller platforms (Domoticz, HomeGenie, Home Assistant) 
* multiple cloud transports (Ethernet, MQTT, Serial)
* multiple MCU boards, (AVR, ESP32, Linux, NRF5, SAMD, STM32F1, teensy3)

##### My Usage
* used for sensor montioring/control using NRF51822 SOC’s on a custom prototype PCB.  
* Each sensor type (Temp, water, leak, actuator) is controlled by a single custom programmed NRF51
* uses a star pattern, each of these talks to a central in coach end node gateway. They could/may eventually talk among themselves.
* These sensor processors are not LoRaWan aware.

#### Getting to the cloud, LoRaWan to the rescue
- We needed to merge the MySensors 2.4Ghz frequency/protocol with the sub gigahertz LoRaWan frequency/protocol.
- first iteration used serial interface from NRF51 to the STMicroelectronics B-L072Z-LRWAN1 Discovery which was the first iteration of the Helium developer kit.
- Serial was somewhat problematic, relatively slow, required a third radio technology, HC-12 serial protocol board.
- Looked from a better solution, then Shiney Object Syndrome kicked in with the WisBlock dev kit.
- Has on board 2.4 Ghz radio in addition to the LoRaWan functionality.
- Much easier merging the two frequency/protocols

#### RTOS vs Bare Metal super loop
- after a bit of poking around under the covers I discovered that the underlying Adafruit library that Wisblock depends on is implemented using a version of FreeRTOS. Shiney Object Alert


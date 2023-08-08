# Welcome to Galaxy Cube documentation
Pre-release version

## Features

* 4 high-speed analog outputs
* 8 low-speed analog outputs
* 16 digital input/outputs
* Arbitrary synchronous waveforms
* Ethernet communication


## General description
The Galaxy Cube is a microscopy hardware controller. Thanks to its arbitrary analog output, it can create the needed waveforms for any device controlled by analog signals, like galvo mirrors. The digital outputs create trigger lines for laser, camera or other digitally-controlled devices.

Communication to and from the Galaxy Cube is done through a local network. Simply connect the Cube to the same network as your workstation! With a web browser, you can use the Cube for simple use-cases without need to code anything. For higher modularity needs, an MQTT-based API is accessible, allowing to integrate all the functionalities of the Cube into your existing setup.

## Production Highlights

### High-speed analog output
* Update rate : 1 MHz
* Voltage range : -5V to +5V.
* Settling time : ~100ns
* Able to generate arbitrary functions.
* Can be used to control galvo mirrors.

### Low-speed analog output 
* Update rate : 50 kHz
* Voltage range : 0 to +5V
* Settling time : 2.5 us (typical)
* Used for Laser/LED intensity,etc

### Digital I/Os
* Update rate : 50 MHz
* Logic level : 3.3V
* Used to get and send trigger signals.

### Easy integration
* Internal Web server for WebUI
* Internal MQTT broker
* Easy-to-use API

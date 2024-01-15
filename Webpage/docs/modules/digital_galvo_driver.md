# Digital galvomirror driver

This is a specific output port designed to drive the xxx-xxx (Hansscanner ?) galvomirror.
The port is a HDMI female port, and the signal has to go through another 
custom board before reaching the device.

You can turn this output on or off by configuring the ASG. 

Currently, the FPGA wires the modules like this : 


| ASG | Digital galvomirror output |
|:---:|:---:|
| 0 | 0 |
| 1 | 1 |
| 2 | 2 |
| 3 | 3 |

!!! Warning 
    This is a different behaviour than the default analog output ! 
    Because of the presence of the signal summers, any ASG 16-bit signal can be forwarded to any DAC and used on any AO-port.
    As such, AO-0 *can* be a different signal than DGD-0

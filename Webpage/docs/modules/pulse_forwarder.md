# Pulse forwarder

This module is used to generate triggers synchronous to specified generated signals. 
!!! TODO
    add image / simulation / plot of a pulse forwarder working

## Principle

Every signal generator (ASG or DSG) also generated 2 types of pulses :   
 - at each new section   
 - at each new period

Theses pulses go into the pulse forwarder, where you can select which one (if any) you 
want to propagate to the digital output. 

For more flexibility, you can process the pulses :   
 - only forward every n-th pulse  
 - add a delay to the pulse  
 - strech the pulse (by default it's 4ns long)

``` mermaid
graph LR
In("Pulses from the ASG") --> Selector
Selector --> Divider["Pulse divider"]
Divider --> Delay["Pulse delayer"]
Delay --> Strech["Pulse strecher"]
Strech --> out("Digital output")
```

!!! warning
    There is no FIFO/Buffer in the processing pipeline. If another pulse arrives while 
    one pulse is still being used (delay or streching), that pulse will get skipped.
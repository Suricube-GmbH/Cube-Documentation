# Using the Galaxy Cube

## Connection the Cube
Connect the Cube to the same network as your workstation. By default, the Cube has the following ip address : **192.168.1.111/24**. It has also mDNS enabled, allowing you to connect to it with the following address : **fz3.local**.
If the Cube is successfully connected to the same network as the workstation, you can ping it and remote connect into it through ssh (default username : root ; default password : root ).
Take note that the Cube takes some time to fully boot (~2min). During the booting process, it will not respond to any pings or SSH requests, looking as if the device had some network issue.

## Web interface

### fz3.local
[Work in progress, maybe put a screenshot of it ?]

### Predefined web apps
Some apps with more powerfull capabilities (interactive plots, etc...).

### Autogenerated UI
If you have a JSON command that works for you, but you regurlarly have to change a couple of values inside
(e.g. recalibration of the mirrors), then you can use the autogenerated web UI to create 
a form in the browser where you can type these values, without changing the other settings. 

!!! Todo
    if this works, post an example (json command + screenshot)
!!! Experimental / prospective
    This is still work in progress.

## MQTT interface
See (API documentation)[] to see all the different MQTT commands you can send to the Cube.

You might have to regenerate or update it. Do this by clicking (here)[].
*Because of a bug in the script, you have to click the link twice to make it work.*

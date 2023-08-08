# Using the Galaxy Cube

## Connection the Cube
Connect the Cube to the same network as your workstation. By default, the Cube has the following ip address : 192.168.1.111/24. It has also mDNS enabled, allowing you to connect to it with the following address : fz3.local.
If the Cube is successfully connected to the same network as the workstation, you can ping it and remote connect into it through ssh (default username : root ; default password : root ).
Take note that the Cube takes some time to fully boot (~2min). During the booting process, it will not respond to any pings or SSH requests, looking as if the device had some network issue.

## Web interface
### fz3.local
[Work in progress, maybe put a screenshot of it ?]
## MQTT interface
See here [add link] to see all the different MQTT commands you can send to the Cube.

## MathJax Test Page

When \(a \ne 0\), there are two solutions to \(ax^2 + bx + c = 0\) and they are
$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$$
dd
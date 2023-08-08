# Python communication

## General idea

To start using the Cube with your python code, you'll need an MQTT library for Python. 
We found that [paho-mqtt](https://pypi.org/project/paho-mqtt/) was very nice and easy 
to use. Try to connect to the Cube through MQTT and send dummy messages to check if 
this part is already working. 

Now you'll need to write the correct message to the cube. Look at the API documentation
to determine how it should look like, and what the options are. You can generate the 
JSON string manually (like in this example code).

Probably a better idea (but more work), is to create a python structure which replicates
the JSON object to pass, so that you can then serialize it. If you don't want to 
do this manually, you can try to use the JSON schemas from the API docs and use 
a datamodel code generator, like [this one for pydantic](https://docs.pydantic.dev/latest/datamodel_code_generator/)


## Code

!!! warning
    The code is only inteded to give a rough idead about how to start working.
    It is not robust, nor is it up-to-date to the API of the Cube.

```python
# -*- coding: utf-8 -*-
"""
Created on Mon Mar 13 16:06:50 2023

@author: Sebastian Hambura
"""

# %% Communication with the Galaxy Streamer
import paho.mqtt.publish as publish
import paho.mqtt.client as mqtt
import time

def raw_value_command(value):
    command = \
        """{
    "name": "Name",
    "device": "ASG",
    "payload": {
        "cmd": "load_raw_sections",
        "config": {
            "sections": [
                {
                    "s0_0": """ + str(value) + """,
                    "s1_0": 0,
                    "s2_0": 0,
                    "s3_0": 0,
                    "length": 100,
                    "interpolation": "cubic"
                },
                {
                    "s0_0": """ + str(value) + """,
                    "s1_0": 0,
                    "s2_0": 0,
                    "s3_0": 0,
                    "length": 100,
                    "interpolation": "cubic"
                }
            ],
            "section_order": [
                {
                    "number": 0,
                    "transition": "discontinous"
                },
                {
                    "number": 1,
                    "transition": "discontinous"
                },
                {
                    "number": 0,
                    "transition": "discontinous"
                },
                {
                    "number": 1,
                    "transition": "discontinous"
                }
            ]
        }
    }
}
        """
        
    return command

def voltage_value_command(value):
    command = \
        """{
    "name": "Name",
    "device": "ASG",
    "payload": {
        "cmd": "load_voltage_sections",
        "config": {
            "sections": [
                {
                    "starting_level": """ + str(value) + """,
                    "starting_derivative": 0,
                    "ending_level": 0,
                    "ending_derivative": 0,
                    "duration_in_ms": 1,
                    "interpolation": "constant"
                },
                {
                    "starting_level": """ + str(value) + """,
                    "starting_derivative": 0,
                    "ending_level": 0,
                    "ending_derivative": 0,
                    "duration_in_ms": 1,
                    "interpolation": "constant"
                }
            ],
            "section_order": [
                {
                    "number": 0,
                    "transition": "discontinous"
                },
                {
                    "number": 1,
                    "transition": "discontinous"
                },
                {
                    "number": 0,
                    "transition": "discontinous"
                },
                {
                    "number": 1,
                    "transition": "discontinous"
                }
            ]
        }
    }
}
        """
        
    return command
 
def start_command():
    command = \
        """
{
    "name": "Name",
    "device": "ASG", 
    "payload":{
        "cmd": "start"
    }
}        
        """
    return command

def set_Streamer_raw_value(value):
    MQTT_topic = "suricube/galaxy_cube/11/analog_signal_generator"
    publish.single(MQTT_topic, payload=raw_value_command(value),  hostname="192.168.1.111", port=1884, protocol=mqtt.MQTTv5, client_id="Python test")
    time.sleep(0.5)
    publish.single(MQTT_topic, payload=start_command(),  hostname="192.168.1.111", port=1884, protocol=mqtt.MQTTv5, client_id="Python test")
      
def set_Streamer_voltage_value(value):
    MQTT_topic = "suricube/galaxy_cube/11/analog_signal_generator"
    publish.single(MQTT_topic, payload=voltage_value_command(value),  hostname="192.168.1.111", port=1884, protocol=mqtt.MQTTv5, client_id="Python test")
    time.sleep(0.5)
    publish.single(MQTT_topic, payload=start_command(),  hostname="192.168.1.111", port=1884, protocol=mqtt.MQTTv5, client_id="Python test")
  
    
```


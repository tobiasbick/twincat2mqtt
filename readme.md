# Very simple Twincat2 Mqtt Function Blocks

I use this very simple function blocks to control a floor in our house. 
As broker i use Mosquitto. "Smart home" is done with OpenHab. Alexa is also working a bit thanks to OpenHab.

Uses MQTT Version 3.1.1. See http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html.

It does not send the DICONNECT telegram!  
No last will, not QoS, not HTTPS, no nothing. 

My CX8090 needs around 30ms cycle time with about 210 publish and about 230 subscribe calls (used global search).

## Needed TwinCat libraries

TcpIp.lib
TcUtilities.lib

---

## DataTypes

### ST_MQTTSETTIGNS
Stores the MQTT settings used for the function blocks.

### ST_MQTTTELEGRAM
Holds a single MQTT telegram (raw data).

### ST_MQTTRECEIVEDPUBLISHTELEGRAM
Holds a decoded MQTT telegram and used internal.

---
## POUs

### Intern
Function block for internal usage.

### FB_MQTT_CONTROLLER
Controls the connection to the MQTT broker. When bActive is TRUE it starts a connection. Sends and receives the data. Must be called every PLC cycle.

#### Inputs:
bActive = Use MQTT (connect to the broker)  
sMqttClientId = A client id like "MyMqttClient"  
sMqttUserId = A user id for the broker connection ('' = none), defaults to '', caution: transmitted unencrypted  
sMqttPassword = A password for the broker connection ('' = none), defaults to '', caution: transmitted unencrypted  
sMqttBrokerIP = The IPV4 address from the broker  
nMqttBrokerPort = The port from the broker, normaly 1883

#### Outputs:
bBusy = The function block is doing something

#### InOut:
stMqttSettings = A global variable for the datatype ST_MQTTSETTINGS

If there is a valid connection, bValidConnection in ST_MQTTSETTINGS is TRUE.

### FB_MQTT_PUBLISH
Send a value for a topic. On the first call it will publish the value from sValue. The it checks every call if the value has changed. When this happens, it sends the new value. Must be called every PLC cycle.

#### Inputs:
sTopic = The mqtt topic to publish on  
sValue = The value which will be sent      
bRetain = See http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718037, defaults to TRUE

#### Outputs:
bBusy = The function block is doing something

#### InOut:
stMqttSettings = A global variable for the datatype ST_MQTTSETTINGS, use the same you used with FB_MQTT_CONTROLLER

### FB_MQTT_SUBSCRIBE
Subscribe to a MQTT topic and receive data. Must be called every PLC cycle.

#### Inputs:
sTopic = The mqtt topic for the subscribtion  
bAccept = When FALSE, subscribe but do not set bNewValue output  

#### Outputs:
sValue = The received value  
bNewValue = A new telegram for topic has arrived. The function block does NOT check if the value is different to the last recevied value for this topic.  
bBusy = The function block is doing something

#### InOut:
stMqttSettings = A global variable for the datatype ST_MQTTSETTINGS, use the same you used with FB_MQTT_CONTROLLER

---
## Sample OpenHab item

Below is a sample for a light or switchable plug.
Old Mqtt1 plugin.

### Items file (Filename: something.items in items directory)
Switch Light "A room name" <light>  
(Group1, Group2)  
[ "Lighting" ]  
{  
    mqtt=">[plc:theMqttTopicUsedReplaceWithYourOwn:command:ON:1],>[plc:theMqttTopicUsedReplaceWithYourOwn:command:OFF:1],<[plc:theMqttTopicUsedReplaceWithYourOwn:state:MAP(onoff.map)]",  
    autoupdate="false"  
}

### Mapping file (Filename: onoff.map in transform directory)
0=OFF  
1=ON

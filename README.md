# Configure Pi Pico W as a MQTT Client using Micropython

## Objectives
- Configure an MQTT Broker on a PC.
- Set up MQTT subscribers and publishers on both the PC and IoT devices.

## Equipment
- Computer / Laptop
- M5StickC Plus

## Introduction
MQTT (Message Queue Telemetry Transport) is a lightweight, open, easy-to-implement messaging transport protocol, making it ideal for Internet of Things (IoT) contexts with limited network bandwidth and small code footprint requirements. It operates over TCP/IP or similar network protocols, providing ordered, lossless, bidirectional connections.

### MQTT Components
- **MQTT Broker**: A program that relays messages between clients, requiring each message to be associated with a topic.
- **Topic**: A namespace for categorizing messages on the broker.
- **MQTT Client**: A device that either publishes to or subscribes to a topic on the broker. Clients can publish, subscribe, unsubscribe, and disconnect from the broker.

![image](https://github.com/drfuzzi/CSC2106_MQTT/assets/108112390/1f809798-135f-4cdc-be7e-0085df0b452f)

Figure 1: An Example of a MQTT Implementation

## Setup Instructions

### 1. Setting Up MQTT Broker (on your laptop)
We will use Mosquitto MQTT broker [version 2](http://www.steves-internet-guide.com/download/6-bit-mosquitto-v2/). After unzipping, update the configuration file (`mosquitto.conf`) to allow network address listening.

#### Steps:
- Update `mosquitto.conf` as shown in below.
- Start the broker with `mosquitto -c mosquitto.conf -v` in the command prompt. Default port is 1883.

```
listener 1883
allow_anonymous true
```

### 2. Setting Up MQTT Client - Subscriber (on your laptop)
Use the following command to start the client as a subscriber:
`mosquitto_sub -h 192.168.x.x -t csc2006`
Replace `192.168.x.x` with the broker’s IP and `csc2006` with your chosen topic.

### 3. Setting Up MQTT Client – Publisher (on your laptop)
Start the client as a publisher using:
`mosquitto_pub -h 192.168.x.x -m “This is CSC2006” -t csc2006`
Ensure the broker's IP and topic match the subscriber's settings (`-h` to specify the IP for the broker and `-t` to specify the topic).

### 4. Setting Up MQTT Client (on the M5StickC Plus)
Configure the MQTT client on M5StickC Plus using API libraries and the provided sample code [mqtt.ino](mqtt.ino). Ensure correct SSID, password, and broker IP. Test communication with the PC subscriber.

### Getting MicroPython to work on Pico C
For the Pico C to be able to run MircoPython, it will need to have a UTF firmware loaded in before it is able to run any python code.

Download UTF2 firmware (I’ve downloaded the latest version and it works for me, just in case this is the link to the previous versions https://micropython.org/download/RPI_PICO_W/)

Simple.py is needed for Thonny to allow for it to use MQTT (You can download it from here and this is the GitHub where the file was originally from https://github.com/micropython/micropython-lib/tree/master/micropython/umqtt.simple)

To install simple.py into thorny, you will need to create 2 folders, lib and umqtt (umqtt will need to be inside lib) as seen below.

1)	How to show files inside Thonny
   
<img src="/img/Thonny Where to Find Files.png" width=50% height=50%>

Below is what would be seen, the top would be your PC and below the Pico C

<img src="/img/Thonny Seperate Files.png" width=50% height=50%>

3)	Create the folder by right clicking and selecting new directory (Double click the folder to enter it and create another)

<img src="/img/Thonny Create Directory(Folder).png" width=50% height=50%>
 
4)	To upload, go into the folder by double clicking and that would move you into the folder. Select the file you want to upload and right click and select upload to.

 <img src="/img/Thonny File Upload.png" width=50% height=50%>

You can go back to the main folder by clicking on Raspberry Pi Pico

**Connecting to broker, sending a message then disconnecting**

(To run and rerun the code, you only need to click on the green run button)
 <img src="/img/Thonny How to Run Code.png" width=50% height=50%>


-
## Common Problems and Solutions
- **Mismatched Topics**: Ensure both publisher and subscriber are using the same topic name.
- **Incorrect Broker IP**: Verify the IP address of the MQTT broker.
- **Network Issues**: Check for proper network connectivity and firewall settings.

## Lab Assignment
1. **Node Interaction Using MQTT**
   - Configure two M5StickC devices (Node A and Node B).
   - Node A should toggle Node B's LED (and vice-versa) upon button press.
   - Implement and propose appropriate topic names for this interaction.

## References
1. [Mosquitto Install Guide](http://www.steves-internet-guide.com/install-mosquitto-broker/)
2. [Mosquitto Broker Guide](http://www.steves-internet-guide.com/mosquitto-broker/)
3. [Mosquitto Client Publish & Subscribe Guide](http://www.steves-internet-guide.com/mosquitto_pub-sub-clients/)
4. [Arduino MQTT Client Code](https://github.com/m5stack/M5StickC-Plus/blob/master/examples/Advanced/MQTT/MQTT.ino)
5. [Thingsboard MQTT Integration](https://thingsboard.io/docs/user-guide/integrations/mqtt/)
6. [Thingspeak MQTT Basics](https://www.mathworks.com/help/thingspeak/mqtt-basics.html)

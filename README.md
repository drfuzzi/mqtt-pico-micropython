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

### Getting MicroPython to work on Pico W
For the Pico W to be able to run MircoPython, it will need to have a UTF firmware loaded in before it is able to run any python code.

Download UTF2 firmware (I’ve downloaded the latest version and it works for me, just in case this is the link to the previous versions https://micropython.org/download/RPI_PICO_W/)

Simple.py is needed for Thonny to allow for it to use MQTT (You can download it from here and this is the GitHub where the file was originally from https://github.com/micropython/micropython-lib/tree/master/micropython/umqtt.simple)

To install simple.py into thorny, you will need to create 2 folders, lib and umqtt (umqtt will need to be inside lib) as seen below.

1)	How to show files inside Thonny
   
<img src="/img/Thonny Where to Find Files.png" width=25% height=25%>

Below is what would be seen, the top would be your PC and below the Pico W

<img src="/img/Thonny Seperate Files.png" width=25% height=25%>

2)	Create the folder by right clicking and selecting new directory (Double click the folder to enter it and create another)

<img src="/img/Thonny Create Directory(Folder).png" width=25% height=25%>
 
3)	To upload, go into the folder by double clicking and that would move you into the folder. Select the file you want to upload and right click and select upload to.

 <img src="/img/Thonny File Upload.png" width=25% height=25%>

You can go back to the main folder by clicking on Raspberry Pi Pico

4) To connect Thonny to the Pico W, click on tools, go to options, select MicroPython (Raspberry Pi Pico) and the port your Pico W is connected to.

<img src="/img/Thonny Options.png" width=25% height=25%>

<img src="/img/Thonny Connecting Pico.png" width=25% height=25%>

**Connecting to broker, sending a message then disconnecting**

(To run and rerun the code, you only need to click on the green run button)

 <img src="/img/Thonny How to Run Code.png" width=25% height=25%>
 
```
import network
import time
from umqtt.simple import MQTTClient

# Wi-Fi Credentials 
WIFI_SSID = "SSID "
WIFI_PASS = "PASSWORD"

# MQTT Broker Details
MQTT_BROKER = "xx.xx.xx.xx"  # Change this to your broker's IP
MQTT_PORT = 1883
MQTT_TOPIC = "pico/test"  # Topic to publish
MQTT_CLIENT_ID = "PicoW_Client"

# Connect to Wi-Fi
def wifi_connect():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(WIFI_SSID, WIFI_PASS)

    while not wlan.isconnected():
        print("Connecting to Wi-Fi...")
        time.sleep(1)

    print("Connected to Wi-Fi:", wlan.ifconfig())

# Connect to MQTT Broker
def connect_mqtt():
    client = MQTTClient(MQTT_CLIENT_ID, MQTT_BROKER, port=MQTT_PORT)
    client.connect()
    print(f"Connected to MQTT Broker at {MQTT_BROKER}:{MQTT_PORT}")
    return client

# Publish a test message
def publish_message(client):
    message = "Hello from Raspberry Pi Pico W!"
    client.publish(MQTT_TOPIC, message)
    print(f"Published: {message}")

# Run MQTT Test
wifi_connect()
mqtt_client = connect_mqtt()
publish_message(mqtt_client)
mqtt_client.disconnect()
print("Disconnected from MQTT")
```

What would be shown 

 <img src="/img/Thonny Sending Message.png" width=50% height=50%>
 <img src="/img/CMD Receiving Message.png" width=50% height=50%>

**Pico W sending message on button press**
```
import network
import time
from umqtt.simple import MQTTClient
from machine import Pin

# Wi-Fi Credentials
WIFI_SSID = "SSID "
WIFI_PASS = "PASSWORD"

# MQTT Broker Details
MQTT_BROKER = "xx.xx.xx.xx"  # Change this to your broker's IP
MQTT_TOPIC = "pico/test"  # Topic to publish on button press
MQTT_CLIENT_ID = "PicoW_Client"

# Button and LED Setup
BUTTON_PIN = 20  # GPIO 15 for button input
LED_PIN = "LED"  # Onboard LED

button = Pin(BUTTON_PIN, Pin.IN, Pin.PULL_UP)  # Pull-up to detect press
led = Pin(LED_PIN, Pin.OUT)  # LED for visual feedback

# Connect to Wi-Fi
def wifi_connect():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(WIFI_SSID, WIFI_PASS)

    while not wlan.isconnected():
        print("Connecting to Wi-Fi...")
        time.sleep(1)

    print("Connected to Wi-Fi:", wlan.ifconfig())

# Connect to MQTT Broker
def connect_mqtt():
    client = MQTTClient(MQTT_CLIENT_ID, MQTT_BROKER, port=1883)
    client.connect()
    print(f"Connected to MQTT Broker at {MQTT_BROKER}:1883")
    return client

# Publish message when button is pressed
def publish_message(client):
    message = "Button Pressed!"
    client.publish(MQTT_TOPIC, message)
    print(f"Published: {message}")

# Run Everything
wifi_connect()
mqtt_client = connect_mqtt()

print("Press the button to send an MQTT message!")

# Main Loop: Check for button press
while True:
    if button.value() == 0:  # Button is pressed (active-low)
        print("Button Pressed!")
        led.on()  # Turn on LED
        publish_message(mqtt_client)  # Send MQTT message
        time.sleep(1)  # Debounce delay
        led.off()  # Turn off LED
    time.sleep(0.1)  # Polling delay
```
What would be shown

 <img src="/img/Thonny Sending Message.png" width=50% height=50%>
 <img src="/img/CMD Receiving Message.png" width=50% height=50%>

**Pico W LED to light up when specific message is received**
For this code, the LED light is tied to the message being sent, if the message is “on” then it would turn the LED light on, if the message is “off” it would turn the LED light off.
```
import network
import time
from umqtt.simple import MQTTClient
from machine import Pin

# Wi-Fi Credentials
WIFI_SSID = "SSID "
WIFI_PASS = "PASSWORD"

# MQTT Broker Details
MQTT_BROKER = "xx.xx.xx.xx"  # Change this to your broker's IP
MQTT_TOPIC_SUBSCRIBE = "pico/led"  # Topic to listen for LED control
MQTT_CLIENT_ID = "PicoW_Client"

# LED Setup
LED_PIN = 15  # Change to the GPIO pin you connected your LED to
led = Pin(LED_PIN, Pin.OUT)  # LED for feedback

# Connect to Wi-Fi
def wifi_connect():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(WIFI_SSID, WIFI_PASS)

    while not wlan.isconnected():
        print("Connecting to Wi-Fi...")
        time.sleep(1)

    print("Connected to Wi-Fi:", wlan.ifconfig())

# MQTT Message Callback (Controls LED)
def mqtt_callback(topic, msg):
    message = msg.decode().strip().lower()
    print(f"Received message on {topic.decode()}: {message}")

    if message == "on":
        print("Turning LED ON")
        led.on()
    elif message == "off":
        print("Turning LED OFF")
        led.off()
    else:
        print("Unknown message received!")

# Connect to MQTT Broker
def connect_mqtt():
    client = MQTTClient(MQTT_CLIENT_ID, MQTT_BROKER, port=1883)
    client.set_callback(mqtt_callback)
    client.connect()
    print(f"Connected to MQTT Broker at {MQTT_BROKER}:1883")

    # Subscribe to LED control topic
    client.subscribe(MQTT_TOPIC_SUBSCRIBE)
    print(f"Subscribed to {MQTT_TOPIC_SUBSCRIBE}")

    return client

# Run Everything
wifi_connect()
mqtt_client = connect_mqtt()

print(f"Listening for LED control messages on {MQTT_TOPIC_SUBSCRIBE}")

# Main Loop: Listen for MQTT messages
while True:
    mqtt_client.check_msg()  # Check for incoming MQTT messages
    time.sleep(0.1)  # Polling delay
```
What would be shown

 <img src="/img/CMD LED.png" width=50% height=50%>
 <img src="/img/Thonny LED.png" width=50% height=50%>


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

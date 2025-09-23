# Configure Raspberry Pi Pico W as an MQTT Client using MicroPython

## Objectives
- Set up an MQTT broker on a PC.
- Configure Raspberry Pi Pico W as MQTT publishers/subscribers.
- Explore MQTT features like QoS, LWT, and retained messages.

---

## Equipment
- Computer or laptop
- Raspberry Pi Pico W with MicroPython firmware
- MQTT broker software (e.g., Mosquitto)

---

## Introduction
MQTT (Message Queue Telemetry Transport) is a lightweight messaging protocol ideal for IoT applications where network bandwidth and processing power are limited. It works on a publish/subscribe model using topics, making it easy to send and receive messages between devices.

### Key Components
- **Broker:** Handles all message routing between clients.
- **Client:** A device that can publish, subscribe, or both.
- **Topic:** A hierarchical string that defines where messages are published and from where they are subscribed.

Example MQTT architecture:

![MQTT Example](https://github.com/drfuzzi/CSC2106_MQTT/assets/108112390/1f809798-135f-4cdc-be7e-0085df0b452f)

---

## Setup Instructions

### 1. Install MicroPython on the Pico W
1. Download the latest MicroPython firmware for Pico W:  
   [https://micropython.org/download/RPI_PICO_W/](https://micropython.org/download/RPI_PICO_W/)
2. Flash the firmware onto the Pico W (drag and drop the `.uf2` file when Pico W is in BOOTSEL mode).

---

### 2. Prepare Thonny
- Install [Thonny IDE](https://thonny.org/).
- Connect Pico W to your PC.
- In **Tools → Options → Interpreter**, select `MicroPython (Raspberry Pi Pico)` and choose the correct port.

---

### 3. Install MQTT Library
You need the `umqtt.simple` library for MQTT communication:
1. In Thonny's **Files** view, create a folder named `lib` on the Pico W.
2. Inside `lib`, create another folder named `umqtt`.
3. Download `simple.py` from the [MicroPython Library](https://github.com/micropython/micropython-lib/tree/master/micropython/umqtt.simple) and upload it into the `umqtt` folder.

---

## Example Codes

### A. Simple Publish
```python
import network, time
from umqtt.simple import MQTTClient

# Wi-Fi Credentials
SSID = "YourWiFi"
PASSWORD = "YourPassword"

# MQTT Details
BROKER = "192.168.1.10"
TOPIC = "pico/test"
CLIENT_ID = "PicoW_Client"

def wifi_connect():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(SSID, PASSWORD)
    while not wlan.isconnected():
        time.sleep(1)
    print("Connected:", wlan.ifconfig())

def connect_mqtt():
    client = MQTTClient(CLIENT_ID, BROKER)
    client.connect()
    return client

wifi_connect()
mqtt = connect_mqtt()
mqtt.publish(TOPIC, "Hello from Pico W!")
mqtt.disconnect()
```
### B. Publish on Button Press
```
from machine import Pin
import time

button = Pin(20, Pin.IN, Pin.PULL_UP)
led = Pin("LED", Pin.OUT)

while True:
    if not button.value():
        led.on()
        mqtt.publish(TOPIC, "Button Pressed!")
        time.sleep(1)
        led.off()
    time.sleep(0.1)
```
### C. Control LED via MQTT
```
def callback(topic, msg):
    if msg.decode() == "on":
        led.on()
    elif msg.decode() == "off":
        led.off()

mqtt.set_callback(callback)
mqtt.subscribe("pico/led")

while True:
    mqtt.check_msg()
    time.sleep(0.1)
```
## Advanced MQTT Configuration

### 1. Quality of Service (QoS)
MQTT supports three QoS levels:

QoS 0: At most once (no guarantee of delivery)
QoS 1: At least once (may deliver duplicates)
QoS 2: Exactly once (most reliable, but slowest)

Example with QoS 1:
```
mqtt.subscribe("pico/led", qos=1)
mqtt.publish("pico/led", "on", qos=1)
```

### 2. Last Will & Testament (LWT)

Notifies subscribers if a client disconnects unexpectedly:
```
client = MQTTClient(CLIENT_ID, BROKER, keepalive=60)
client.set_last_will("pico/status", "offline", retain=True)
```

### 3. Retained Messages

Ensures new subscribers immediately receive the last published message:
```
mqtt.publish("pico/led", "on", retain=True)
```

### 4. Persistent Sessions

Keep subscriptions active even when the client disconnects:
```
client = MQTTClient(CLIENT_ID, BROKER, clean_session=False)
```

## Troubleshooting
-------------------------------------------------------------------------
| Issue                        | Solution                               |
| ---------------------------- | -------------------------------------- |
| No messages received         | Check broker IP and topic name         |
| Wi-Fi not connecting         | Verify SSID/password, signal strength  |
| Multiple clients conflicting | Use unique `CLIENT_ID` for each client |
-------------------------------------------------------------------------

## Lab Assignment

Configure two Pico W devices.
Device A controls Device B's LED via MQTT.
Use retained messages and LWT to improve reliability.

## References
Mosquitto Install Guide (http://www.steves-internet-guide.com/install-mosquitto-broker/)
Mosquitto Client Guide(http://www.steves-internet-guide.com/mosquitto_pub-sub-clients/)
Thingsboard MQTT Integration (https://thingsboard.io/docs/user-guide/integrations/mqtt/)
ThingSpeak MQTT Basics (https://www.mathworks.com/help/thingspeak/mqtt-basics.html)

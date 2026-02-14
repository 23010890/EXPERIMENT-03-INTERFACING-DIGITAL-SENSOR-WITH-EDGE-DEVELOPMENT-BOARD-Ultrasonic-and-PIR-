# EXPERIMENT-03-INTERFACTING-DIGITAL-SENSOR-WITH-EDGE-DEVELOPMENT-BOARD-ULTRASONIC-AND-PIR-SENSOR-(RASPBERRYPI-PI4)
### NAME : DHARSHINI S
### DEPARTMENT : CSE(IOT)
### ROLL NO : 212223110010 
### DATE OF EXPERIMENT : 12-02-2026

### AIM
To interface a digital sensor (Ultrasonic and PIR) with the Raspberry Pi 4 and control it using Python.

## APPARATUS REQUIRED
Raspberry Pi 4
LED (Light Emitting Diode)
330Ω Resistor
IR Sensor
Breadboard
Jumper Wires
USB Cable
 ## THEORY

![Raspberry Pi Pin](https://github.com/user-attachments/assets/19e5a1e7-cb46-4909-ba59-e4f4560cae03)





 
 
 
 ### FIGURE-01 RASPI PI 4 PINOUT DIAGRAM 


The Raspberry Pi 4 Model B is built around a Broadcom BCM2711 system-on-chip that integrates a quad-core ARM Cortex-A72 (64-bit) CPU, VideoCore VI GPU, memory controller, and peripheral interfaces, forming a compact yet complete computer architecture where the SoC connects internally to RAM, USB 3.0 controller, Gigabit Ethernet, HDMI display, and wireless modules. Its 40-pin GPIO header provides a flexible pin configuration consisting of power pins (5 V and 3.3 V), multiple ground pins, and general-purpose input/output pins that operate at 3.3 V logic and can be programmed for digital I/O or alternate functions. Key alternate functions include I²C (SDA, SCL) for sensor communication, SPI (MOSI, MISO, SCLK, CS) for high-speed peripheral interfacing, UART (TX, RX) for serial communication, and PWM for control applications.  For communication, I2C (SDA, SCL), SPI (MOSI, MISO, SCK), and UART (TX, RX) interfaces are mapped across different GPIO pins, allowing seamless connectivity with sensors and peripherals. All GPIO pins support PWM (Pulse Width Modulation), making it useful for motor control, LED brightness adjustment, and sound applications. The BOOTSEL button enables USB mass storage mode for firmware flashing, while the DEBUG pins (SWD interface) provide debugging capabilities. With its low power consumption, flexible GPIO options, and rich interface support, the Raspberry Pi Pico is widely used for IoT, embedded systems, robotics, and automation projects.This architecture and pin multiplexing allow the Raspberry Pi 4 to act as both a general-purpose computing platform and an embedded controller, supporting rapid prototyping, hardware interfacing, and IoT applications.
## Ultrasonic Sensor:
An ultrasonic sensor is a distance-measuring device that uses high-frequency sound waves to detect objects and calculate how far away they are. It works by emitting ultrasonic pulses (typically around 40 kHz) and measuring the time taken for the echo to bounce back after hitting an object. Using the speed of sound in air, the sensor converts this time into distance. Ultrasonic sensors are widely used in robotics, obstacle detection, parking systems, and automation because they provide reliable, contactless measurement regardless of lighting conditions.
<img width="600" height="411" alt="image" src="https://github.com/user-attachments/assets/d879d982-347a-45ff-92c1-b78b57e9fdef" />


## PIR Sensor:
A Passive Infrared (PIR) sensor is a motion detection device that senses changes in infrared radiation emitted by warm objects such as humans or animals. Instead of emitting signals, it passively detects heat variations within its field of view. When a warm body moves across the sensor’s detection zones, it triggers an electrical signal indicating motion. PIR sensors are commonly used in security alarms, automatic lighting systems, and energy-saving smart devices due to their low power consumption and ability to detect human presence effectively.
<img width="428" height="494" alt="image" src="https://github.com/user-attachments/assets/bb6b0f22-33d7-4d63-b5c6-05e6d655e71d" />

## Working Principle:
Experiment 1A
The Ultrasonic sensor Trig pin is connected to one of the GPIO pins of the Raspberry Pi 4.
The Ultrasonic sensor Echo pin is connected to one of the GPIO pins of the Raspberry Pi 4.
The Python script sets the take the distance taken echo output and shown in Thingspeak cloud with current status and Console.
CIRCUIT DIAGRAM
Connect the Vcc of the Ultrasonic sensor +5V in Raspberrry Pi4.
Connect the Gnd of the Ultrasonic sensor Gnd in Raspberrry Pi4.
Connect the Trig pin to any one GPIO.
Connect the Echo pin to any one GPIO.


Experiment 1B
The IR sensor is connected one of the GPIO pins in Raspberry Pi 4.
The Python script sets the PIR sensor value based on the motion detected and shown in Thingspeak and console.
## CIRCUIT DIAGRAM
Connect the PIR sensor Vcc to any +5V.
Connect the PIR sensor GND to any GND.
Connect the PIR sensor OUT to any one GPIO. 





## PROGRAM (Python)
### Experiment 3A
```

import RPi.GPIO as GPIO
import time
import requests

# ThingSpeak settings
API_KEY = "22Q3A8TKVUCCLO7S"
THINGSPEAK_URL = "https://api.thingspeak.com/update"

# GPIO pins
TRIG = 18
ECHO = 23

GPIO.setmode(GPIO.BCM)
GPIO.setup(TRIG, GPIO.OUT)
GPIO.setup(ECHO, GPIO.IN)

def get_distance():
    GPIO.output(TRIG, False)
    time.sleep(0.5)

    # Trigger pulse
    GPIO.output(TRIG, True)
    time.sleep(0.00001)
    GPIO.output(TRIG, False)

    while GPIO.input(ECHO) == 0:
        pulse_start = time.time()

    while GPIO.input(ECHO) == 1:
        pulse_end = time.time()

    pulse_duration = pulse_end - pulse_start
    distance = pulse_duration * 17150
    distance = round(distance, 2)

    return distance

try:
    while True:
        distance = get_distance()

        # Console output
        print("distance =", distance, "cm")

        # Text message for ThingSpeak
        status_text = f"distance = {distance} cm"

        # Send data to ThingSpeak
        payload = {
            "api_key": API_KEY,
            "field1": distance,   # numeric for chart
            "status": status_text # text message
        }

        response = requests.get(THINGSPEAK_URL, params=payload)
        print("Sent to ThingSpeak")

        time.sleep(15)

except KeyboardInterrupt:
    GPIO.cleanup()

```

### Experiment 3B

```
import RPi.GPIO as GPIO
import time
import requests

# ---------- ThingSpeak Details ----------
CHANNEL_ID = "3261139"
WRITE_API_KEY = "Z436C9E4G2B8I5Q1"
THINGSPEAK_URL = "https://api.thingspeak.com/update"

# ---------- PIR Setup ----------
PIR_PIN = 23   # GPIO17

GPIO.setmode(GPIO.BCM)
GPIO.setup(PIR_PIN, GPIO.IN)

print("PIR Sensor Initializing...")
time.sleep(2)
print("System Ready. Monitoring Motion...")

def send_to_thingspeak(value):
    payload = {
        "api_key": WRITE_API_KEY,
        "field1": value
    }
    try:
        requests.get(THINGSPEAK_URL, params=payload)
        print("Data sent to ThingSpeak:", value)
    except:
        print("Error sending data")

try:
    while True:
        motion = GPIO.input(PIR_PIN)

        if motion == 1:
            print("Motion Detected!")
            send_to_thingspeak(1)   # 1 = Motion
            time.sleep(15)          # ThingSpeak update delay

        else:
            print("No Motion")
            send_to_thingspeak(0)   # 0 = No motion
            time.sleep(15)

except KeyboardInterrupt:
    print("Program Stopped")
    GPIO.cleanup()
```
### OUPUT

### Experiment 3A
![WhatsApp Image 2026-02-13 at 7 49 31 PM](https://github.com/user-attachments/assets/8cc2a100-9e3b-4509-8246-d0fc597075e6)

![WhatsApp Image 2026-02-14 at 8 41 54 AM](https://github.com/user-attachments/assets/1dd020c1-ce2a-4588-9c6e-11f8261bea3f)

<img width="1810" height="870" alt="Screenshot 2026-02-12 141826" src="https://github.com/user-attachments/assets/6702d4da-b5db-469e-9f0b-2f1d71e7defe" />


### Experiment 3B
![WhatsApp Image 2026-02-13 at 7 49 32 PM](https://github.com/user-attachments/assets/fdc9ce78-13a1-419a-b537-e5d175f56e6f)

![WhatsApp Image 2026-02-14 at 8 41 54 AM (1)](https://github.com/user-attachments/assets/f7b93a4a-8ca4-4124-ba1f-071e24ffc864)

<img width="1893" height="891" alt="Screenshot 2026-02-12 143218" src="https://github.com/user-attachments/assets/e4c7fdcc-32e2-42f5-8920-a6fca9c1d8be" />

<img width="1897" height="892" alt="Screenshot 2026-02-12 143234" src="https://github.com/user-attachments/assets/dd3f659d-7c15-46f5-bb88-bd77e365f77a" />

 
## RESULTS
The Ultrasonic sensor and PIR sensor is connected to the Raspberry Pi 4 successfully and the distance and the motion detection is visualised in thingspeak confirming the proper interfacing of a digital output.

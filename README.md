# 🏠 Smart Room Automation System with ESP32

A low-cost **ESP32-based Smart Room Automation System** that allows users to control multiple appliances using both **IoT (Blynk Cloud)** and **Manual Switches**, ensuring reliable operation even when the internet is unavailable.

---

## 📌 Features

- 🌐 IoT-based remote control using **Blynk**
- 🔘 Manual switch control (Offline Mode)
- ⚡ Control up to 5 appliances
- 💡 LED status indicators
- 🧠 Scene / Group Control
- 🔄 Power Failure Memory (State Restore)
- 📶 Automatic Wi-Fi Reconnection
- 🚀 Professional Stability Layer
- ⏲ Auto-Off Timer
- 🏠 Leaving Home Mode
- 📊 Device Usage Counter
- 📜 Event Logging (Last 10 Actions)
- 🔄 Real-time Blynk synchronization

---

# Project Objective

The objective of this project is to design and develop a low-cost smart room automation system using the **ESP32 microcontroller**. The system enables control of multiple home appliances through:

- Internet-based control using Blynk
- Manual switch control
- Real-time status indication
- Reliable operation during internet failures
- Energy-efficient and affordable smart home automation

---

# System Operation

The system operates in two modes:

## 1. IoT Mode (Online)

```
Mobile App
     ↓
Blynk Cloud
     ↓
ESP32
     ↓
Relay Module
     ↓
Appliance
```

### Process

1. User presses a button in Blynk app.
2. Command reaches Blynk Cloud.
3. ESP32 receives the command via Wi-Fi.
4. GPIO pin activates relay.
5. Relay switches appliance ON/OFF.
6. LED indicator updates status.

### Advantages

- Remote access
- Real-time control
- Mobile automation
- Status monitoring

---

## 2. Manual Mode (Offline)

```
Switch
   ↓
ESP32
   ↓
Relay Module
   ↓
Device
```

### Process

1. User presses a physical switch.
2. ESP32 detects switch state.
3. Corresponding relay changes state.
4. Device turns ON/OFF.
5. LED updates immediately.

### Advantages

- Works without internet
- Fast response
- Reliable operation

---

# Hardware Components

| Component | Purpose |
|------------|---------|
| ESP32 | Main controller + WiFi |
| 5-Channel Relay Module | Appliance switching |
| LEDs | Device status indication |
| 220Ω/330Ω Resistors | LED current limiting |
| Push Buttons | Manual control |
| 5V 2A Adapter | Power supply |
| Wires & Connectors | Electrical connections |
| Integration Board | Assembly platform |

---

# System Architecture

```text
User (App / Switch)
        ↓
Internet (Optional)
        ↓
ESP32
        ↓
Relay Module
        ↓
Appliances
        ↓
LED Indicators
```

---

# Relay Connections

| Device | GPIO Pin | Relay Input |
|----------|---------|------------|
| Device 1 | GPIO 23 | IN1 |
| Device 2 | GPIO 22 | IN2 |
| Device 3 | GPIO 21 | IN3 |
| Device 4 | GPIO 19 | IN4 |
| Device 5 | GPIO 18 | IN5 |

---

# Manual Switch Connections

| Switch | GPIO Pin |
|---------|---------|
| Switch 1 | GPIO 13 |
| Switch 2 | GPIO 12 |
| Switch 3 | GPIO 14 |
| Switch 4 | GPIO 27 |
| Switch 5 | GPIO 26 |

All switch terminals connect to **GND**.

> Use `INPUT_PULLUP` in code.

---

# Blynk Virtual Pins

| Virtual Pin | Device |
|------------|-------|
| V1 | Device 1 |
| V2 | Device 2 |
| V3 | Device 3 |
| V4 | Device 4 |
| V5 | Device 5 |

---

# Module 1 — Basic Device Control

### Online Control

- Blynk Mobile App
- Blynk Cloud
- ESP32 WiFi
- Relay Module

### Offline Control

- Physical Push Buttons
- INPUT_PULLUP mode
- Real-time relay switching

---

# Module 2 — Scene Control

Provides one-touch control for multiple devices.

---

## 🌞 All Lights Scene

**Virtual Pin:** `V20`

Controls:

- Device 1
- Device 4

```cpp
BLYNK_WRITE(V20)
{
    int state = param.asInt();

    setRelay(0, state);
    setRelay(3, state);
}
```

---

## 🌙 Bedtime Mode

**Virtual Pin:** `V21`

Behavior:

- Turns OFF unnecessary devices.
- Keeps essential devices ON.

```cpp
BLYNK_WRITE(V21)
{
    int state = param.asInt();

    if(state==1)
    {
        setRelay(0,0);
        setRelay(1,0);
        setRelay(3,0);

        setRelay(4,1);
    }
}
```

### Advantages

- One-touch control
- User friendly
- Works with manual switches
- Independent from individual controls

---

# Module 3 — Power Failure Memory

Uses ESP32 **Preferences Library**.

### Features

✔ State restoration after power failure

✔ Flash memory storage

✔ Blynk synchronization

✔ Automatic recovery

---

### Library

```cpp
#include <Preferences.h>
```

---

### Save State

```cpp
preferences.putBool(key.c_str(), state);
```

---

### Restore State

```cpp
relayState[i] =
preferences.getBool(key.c_str(),0);
```

---

# Module 4 — Professional Stability Layer

Improves:

- Boot reliability
- WiFi reconnection
- Blynk stability
- Crash prevention
- Faster response

---

## Non-blocking Blynk Connection

Instead of:

```cpp
Blynk.begin(auth, ssid, password);
```

Use:

```cpp
Blynk.config(auth);

Blynk.connectWiFi(ssid,password);
```

---

## Safe Reconnection

```cpp
if(!Blynk.connected())
{
    Blynk.connect();
}
```

---

## Stable Loop

```cpp
void loop()
{
    if(Blynk.connected())
        Blynk.run();
    else
        Blynk.connect();

    handleSwitches();
}
```

---

## Safe Startup

```cpp
delay(1500);
```

Prevents relay glitches during boot.

---

## Debounce Upgrade

```cpp
if(curr==LOW &&
lastSwitch[i]==HIGH &&
millis()-lastTime[i]>200)
{
    setRelay(i,!relayState[i]);

    lastTime[i]=millis();
}
```

---

# Module 5 — Auto-Off Timer

Automatically turns OFF devices after a specified time.

### Library

```cpp
#include <BlynkTimer.h>

BlynkTimer timer;
```

---

### Example

```cpp
autoOffTime[4]=300;
```

Device 5 turns OFF after **5 minutes**.

---

### Timer Scheduling

```cpp
timer.setTimeout(
autoOffTime[i]*1000L,
[i]()
{
    setRelay(i,0);
});
```

---

# Module 6 — Leaving Home Button

**Virtual Pin:** `V30`

Turns OFF all appliances instantly.

```cpp
BLYNK_WRITE(V30)
{
    int state=param.asInt();

    if(state==1)
    {
        for(int i=0;i<N;i++)
        {
            setRelay(i,0);
        }
    }
}
```

### Benefits

- One-click shutdown
- Energy saving
- Safe operation

---

# Module 7 — Usage Counter

Tracks how many times each device has been used.

```cpp
int usageCount[N]={0};
```

Increment:

```cpp
if(state==1)
usageCount[i]++;
```

---

### Print Usage

```cpp
void printUsage()
{
    for(int i=0;i<N;i++)
    {
        Serial.println(usageCount[i]);
    }
}
```

---

# Module 8 — Event Log

Stores last 10 actions.

Example:

```text
Device 1 ON
Device 3 OFF
Device 5 ON
```

---

### Variables

```cpp
String eventLog[10];

int logIndex=0;
```

---

### Logging Function

```cpp
void logEvent(int i,bool state)
{
    String msg="Device ";

    msg+=(i+1);

    msg+=state?" ON":" OFF";

    eventLog[logIndex]=msg;

    logIndex=(logIndex+1)%10;
}
```

---

# Final Loop

```cpp
void loop()
{
    if(Blynk.connected())
    {
        Blynk.run();
    }
    else
    {
        Blynk.connect();
    }

    timer.run();

    handleSwitches();
}
```

---

# Project Advantages

✅ Remote control from anywhere

✅ Manual control without internet

✅ State memory after power failure

✅ Scene automation

✅ Automatic Wi-Fi recovery

✅ Event logging

✅ Device usage analytics

✅ Auto OFF timer

✅ Professional firmware stability

✅ Low-cost implementation

---

# Future Improvements

- Alexa integration
- Google Assistant support
- MQTT communication
- Energy monitoring
- OLED display
- Voice control
- Temperature & humidity sensors
- Scheduling and automation rules

---

# Technologies Used

- ESP32
- Arduino IDE
- Blynk IoT
- Wi-Fi
- Relay Module
- Preferences Library
- BlynkTimer

---

# Author

**Smart Room Automation with ESP32**

IoT-Based Home Automation System using ESP32 and Blynk.

---
⭐ If you found this project useful, give it a star on GitHub!

## 1. Overview

This project implements a hybrid control system for a DC motor and servos using:

- **ROS 2 (Python node)** as a communication interface
- **Arduino Mega 2560** as the hardware controller
- **BTS7960 motor driver** for high-current motor control
- **IBUS RC receiver** for manual override

The system supports:

- **RC Mode** → Manual control via transmitter
- **ROS Mode** → Autonomous control via ROS 2 over serial

### 🔌 Hardware Requirements

- Arduino Mega 2560
- BTS7960 motor driver
- IBUS receiver
- 2x Servo motors
- External motor power supply

---

## 2. System Architecture

```
ROS 2 Node (/motor_command topic)
        ↓
   std_msgs/String
        ↓
Serial (USB, 115200 baud)
        ↓
     Arduino Mega
        ↓
  BTS7960 Motor Driver
        ↓
      DC Motor

        +
   Servo Outputs
        +
   IBUS RC Receiver (mode switching)
```

---

## 3. Installation & Setup

### 3.1 Install ROS 2 Package

#### Step 1: Create Workspace (if not already)

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
```

#### Step 2: Add Package

Clone your package:

```bash
git clone    
```

#### Step 3: Install Dependencies

```bash
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
```

#### Step 4: Build the Package

```bash
colcon build
```

#### Step 5: Source Workspace

```bash
source install/setup.bash
```

#### Step 6: Run Node

```bash
ros2 run arduino_motor motor_node
```

---

### 3.2 Install Python Serial Library

Your ROS node depends on **pyserial**.
Install using:

```bash
pip install pyserial
```

Or (recommended for ROS environments):

```bash
pip3 install pyserial
```

#### Verify Installation

```bash
python3 -c "import serial; print(serial.__version__)"
```


### 3.3 Arduino Mega 2560 Setup

#### Install Arduino IDE

##### Install Required Libraries

In Arduino IDE:

1. Go to **Sketch → Include Library → Manage Libraries**
2. Install:
    - **IBusBM**
    - **Servo** (usually pre-installed)

---

####  Upload (Burn) Code to Arduino Mega

##### Step 1: Connect Arduino
- Plug Arduino via USB
- Identify port:

```bash
ls /dev/ttyUSB*
```

##### Step 2: Select Board
In Arduino IDE:
- **Tools → Board → Arduino Mega or Mega 2560**
- **Tools → Processor → ATmega2560**
- **Tools → Port → /dev/ttyUSB0** (or your port)

##### Step 3: Upload Code
- Paste your Arduino code into IDE
- Click **Upload**

##### Step 4: Verify Serial Output
Open Serial Monitor:
- Baud rate: **115200**

You should see:

```
BTS7960 + Servo + RC/ROS Switch Ready
```

### 3.4 Serial Permissions (Linux)

If you get permission errors:

```bash
sudo usermod -a -G dialout $USER
```

Then logout/login.

---

## 4. ROS Interface

### Topic: `/motor_command`
- **Type:** `std_msgs/msg/String`
- **Direction:** ROS → Arduino

### Behavior

The ROS node:
- Receives string
- Converts to uppercase
- Sends directly via serial

---

## 5. Serial Command Protocol

### 5.1 Motor Commands

|Command|Description|
|---|---|
|`F<speed>`|Forward|
|`B<speed>`|Reverse|
|`STOP`|Stop motor|

Speed range: `0–255`


### 5.2 Servo Commands

|Command|Description|
|---|---|
|`SERVO1<angle>`|Servo 1|
|`SERVO2<angle>`|Servo 2|

Angle range: `0–180`

### 5.3 System Commands

|Command|Description|
|---|---|
|`HEARTBEAT`|Keep-alive|

---
## 6. Example ROS Commands

```bash
ros2 topic pub /motor_command std_msgs/msg/String "{data: 'F150'}"
ros2 topic pub /motor_command std_msgs/msg/String "{data: 'B200'}"
ros2 topic pub /motor_command std_msgs/msg/String "{data: 'STOP'}"
ros2 topic pub /motor_command std_msgs/msg/String "{data: 'SERVO190'}"
ros2 topic pub /motor_command std_msgs/msg/String "{data: 'SERVO245'}"
```

---

## 7. Arduino Behavior

### 7.1 Mode Switching (RC vs ROS)
- Controlled via IBUS Channel 5

|CH5 Value|Mode|
|---|---|
|≤1500|RC Mode|
|>1500|ROS Mode|

### 7.2 ROS Mode
- Reads serial commands
- Executes motor/servo control
- Uses watchdog safety

### 7.3 RC Mode
- Direct IBUS control
- Motor + servo mapping

---

## 8. Watchdog Safety

- Timeout: **1000 ms**
- Stops motor if no ROS command received

---

## 9. Hardware Mapping

### Motor Driver (BTS7960)

|Pin|Function|
|---|---|
|RPWM|Pin 6|
|LPWM|Pin 5|
|R_EN|Pin 4|
|L_EN|Pin 3|

### Servos

|Servo|Pin|
|---|---|
|Servo1|8|
|Servo2|9|

### Serial

|Interface|Usage|
|---|---|
|Serial|ROS|
|Serial1|IBUS|

## 10. Design Notes

### Advantages

- Simple architecture
- Fast communication
- Easy debugging
- Reliable hybrid control


### Limitations

- No structured ROS messages
- No feedback to ROS
- Requires strict command formatting

---

## 11. Summary

This system combines:

- ROS 2 (high-level interface)
- Arduino (low-level control)
- RC fallback (manual safety)

The ROS node acts as a **transparent communication bridge**, while the Arduino performs all control logic.

---

## 12. Usage Workflow

1. Upload Arduino code
2. Connect hardware
3. Run ROS node
4. Switch to ROS mode via RC
5. Publish commands

---

## 13. Notes

- Ensure correct serial port (`/dev/ttyUSB0`)
- Always send valid commands
- System is fully tested and operational

---
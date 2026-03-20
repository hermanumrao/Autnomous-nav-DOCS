## 1. Overview

The `decision_package` implements a **ROS 2 decision-making node** that processes obstacle data and determines robot behavior.

It acts as an **intermediate intelligence layer** between perception and actuation.
### Purpose
- Interpret obstacle detection results
- Classify risk levels
- Generate appropriate motion commands
- Forward commands to the motor control system
---

## 2. System Architecture

```id="x4m2eq"
/obstacle_clusters
        ↓
   decision_node
        ↓
/motor_command
        ↓
   motor_node
        ↓
    Arduino
        ↓
   Motor + Servos
```

---

## 3. Package Description

### Package Name

`decision_package`

### Node Name

`decision_node`

### Language

Python (`rclpy`)

---

## 4. ROS Interfaces

### 4.1 Subscribed Topic

|Topic|Type|Description|
|---|---|---|
|`/obstacle_clusters`|`std_msgs/msg/String`|Obstacle classification data|

### 4.2 Published Topic

|Topic|Type|Description|
|---|---|---|
|`/motor_command`|`std_msgs/msg/String`|Motor control commands|

---

## 5. Input Data Format
The node expects obstacle classification as a **string containing keywords**.
### Supported Keywords

|Keyword|Meaning|
|---|---|
|`SAFE`|No obstacle|
|`MODERATE`|Obstacle nearby|
|`CRITICAL`|Immediate danger|

### Example Inputs

```id="duulhe"
"SAFE"
"MODERATE obstacle detected"
"CRITICAL"
```

---

## 6. Decision Logic

The node evaluates incoming data and maps it to motor commands.

### Logic

```python
if "CRITICAL" in data:
    command = "STOP"
elif "MODERATE" in data:
    command = "F80"
else:
    command = "F150"
```

### Behavior Mapping

|Condition|Output Command|Description|
|---|---|---|
|CRITICAL|`STOP`|Immediate stop|
|MODERATE|`F80`|Slow forward|
|SAFE|`F150`|Normal forward|

## 7. Output Command Protocol
Commands are compatible with the **Arduino motor protocol**.

### Motor Commands

```text
F<speed>   → Forward
B<speed>   → Backward
STOP       → Stop
```

### Examples

```text
F150
F80
STOP
```

## 8. Execution Flow

1. Receive obstacle data
2. Convert to uppercase
3. Match keyword
4. Generate motor command
5. Publish to `/motor_command`
6. Motor node forwards to Arduino

---

## 9. Integration with Motor Package

This package does **NOT directly communicate with hardware**.

Instead:
- It publishes commands
- `arduino_motor` package handles serial communication

### Correct Integration

```text
decision_node → /motor_command → motor_node → Arduino
```

---

## 10. Installation & Setup

### 10.1 Add to Workspace

```bash
cd ~/ros2_ws/src
cp -r decision_package .
```

### 10.2 Install Dependencies

```bash
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
```

### 10.3 Build

```bash
colcon build
```

### 10.4 Source Workspace

```bash
source install/setup.bash
```

### 10.5 Run Node

```bash
ros2 run decision_package decision_node
```

---

## 11. Testing

### Publish Test Data

```bash
ros2 topic pub /obstacle_clusters std_msgs/msg/String "{data: 'SAFE'}"
ros2 topic pub /obstacle_clusters std_msgs/msg/String "{data: 'MODERATE'}"
ros2 topic pub /obstacle_clusters std_msgs/msg/String "{data: 'CRITICAL'}"
```

### Monitor Output

```bash
ros2 topic echo /motor_command
```

---

## 12. Design Considerations

### Advantages

- Clear separation of perception and control
- Modular ROS architecture
- Easy to extend decision logic
- Compatible with existing Arduino protocol

---

### Limitations

- Uses string-based messages (not structured)
- Keyword matching is fragile
- No probabilistic decision-making
- No feedback loop

---

## 13. Future Improvements

- Replace strings with custom ROS messages
- Add velocity-based control (`/cmd_vel`)
- Introduce state machine or behavior tree
- Integrate sensor fusion
- Add feedback from motor system

---

## 14. Summary

The `decision_package` provides a **lightweight decision-making layer** that:

- Interprets obstacle data
- Converts it into motor commands
- Interfaces cleanly with the motor control package

It forms a critical part of a **ROS-based autonomous control pipeline**, enabling reactive behavior based on environmental perception.

---
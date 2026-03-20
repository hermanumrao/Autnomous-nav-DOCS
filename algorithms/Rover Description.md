## Overview of the Package

This is a **ROS 2 package** designed to model, visualize, and launch a **4-wheel-drive (4WD) rover robot**. It focuses on:

- Robot description using **URDF/Xacro**
- Visualization using **RViz**
- Launching the robot model for inspection

It does **not** include control, navigation, or simulation (e.g., Gazebo) — it is primarily a **robot description + visualization package**.

---

## Package Structure

### Core files:

- `package.xml` → package metadata and dependencies
- `CMakeLists.txt` → build configuration
- `urdf/four_wd_rover.urdf.xacro` → robot model
- `launch/display.launch.py` → basic visualization
- `launch/display_all.launch.py` → extended launch
- `rviz/robot_model.rviz` → RViz configuration

---

## 1. Package Configuration

#### `package.xml`

Defines:

- Package name: `four_wd_rover`
- ROS 2 dependencies (likely including):
    - `robot_state_publisher`
    - `xacro`
    - `rviz2`

#### Purpose:

Ensures ROS knows:
- How to build the package
- What dependencies are required

---

#### `CMakeLists.txt`

Handles:

- Installation of:
    - Launch files
    - URDF files
    - RViz configs

#### Key Role:

Even though this is mostly a description package, this file ensures:
- Proper ROS 2 build integration
- Files are discoverable via `ros2 launch`

---

## 2. Robot Description (URDF/Xacro)

### `four_wd_rover.urdf.xacro`

This is the **core of the project**.

#### Key Features:

- Built using **Xacro (XML Macros)** for flexibility
- Defines:
    - Robot links (body, wheels)
    - Joints connecting wheels to chassis
    - Likely uses repeated macros for wheels

### Robot Structure

#### Main Components:

1. **Base Link**
    - Central chassis of the rover
    - Reference frame for the robot

2. **Four Wheels**
    - Front-left
    - Front-right
    - Rear-left
    - Rear-right
    
3. **Joints**
    - Each wheel connected via joints (likely continuous joints for rotation)

## 3. Launch System

### `display.launch.py`

#### Purpose:
- Launches the robot for visualization

#### What it does:
- Loads the URDF via `robot_state_publisher`
- Starts RViz
- Displays the robot model

### `display_all.launch.py`

#### Likely adds:

- Additional nodes or configurations  
- Possibly:
    - Joint state publisher GUI
    - More visualization options

---
git: https://github.com/introlab/rtabmap_ros

![[Attachments/Pasted image 20260309152917.png]]

## Overview of RTAB-Map

**RTAB-Map (Real-Time Appearance-Based Mapping)** is an open-source SLAM framework designed for **real-time localization and mapping using visual and depth data**. 

**RTAB-Map** (Real-Time Appearance-Based Mapping) is a RGB-D, Stereo and Lidar Graph-Based SLAM approach based on an **incremental appearance-based loop closure detector**. The loop closure detector uses a **bag-of-words approach** to determinate how likely a new image comes from a previous location or a new location. When a loop closure hypothesis is accepted, a new constraint is added to the map’s graph, then a graph optimizer minimizes the errors in the map. A memory management approach is used to limit the number of locations used for loop closure detection and graph optimization, so that real-time constraints on large-scale environnements are always respected. RTAB-Map can be used alone with a handheld Kinect, a stereo camera or a 3D lidar for 6DoF mapping, or on a robot equipped with a laser rangefinder for 3DoF mapping.


Key characteristics include:

- **Loop closure detection** using appearance-based recognition
- **Graph optimization** for reducing accumulated drift
- **3D and 2D map generation**
- **Real-time operation on moderate hardware**
- Integration with **ROS / ROS2**

Because it supports RGB-D sensors, stereo cameras, and LiDAR, RTAB-Map has become widely used in **mobile robotics, autonomous navigation, and indoor mapping**.

---

## Core Architecture

RTAB-Map operates using three main components:

### 1. Visual Odometry
Visual odometry estimates the robot’s motion between frames using camera images.

Methods typically used:
- Feature detection(ORB, SURF, SIFT)
- Feature matching between frames
- Pose estimation using PnP or ICP
The result is an incremental estimate of robot motion.

---

### 2. Loop Closure Detection

One of RTAB-Map’s strongest features is its **appearance-based loop closure system**.

When the robot revisits a previously seen area:
1. The system compares the current image with stored visual memories.
2. If a match is found, it detects a **loop closure**
3. The pose graph is optimized to reduce accumulated drift.

This dramatically improves long-term mapping accuracy.

---

### 3. Graph Optimization

The robot’s trajectory is stored as a **pose graph**.

Nodes:
- Camera frames
- Sensor observations

Edges:
- Odometry constraints   
- Loop closure constraint
Graph optimization algorithms (typically **g2o or GTSAM**) refine the robot’s trajectory to maintain map consistency.
#### Illumination-Invariant Visual Re-Localization
#### Lidar and Visual SLAM
#### Simultaneous Planning, Localization and Mapping (SPLAM)
#### Multi-session SLAM
#### Loop closure detection
![Peek 2024-11-30 20-35](https://github.com/user-attachments/assets/f64a44bf-148b-4658-9b25-b46bd0411a69)

## Realsense and its integration with RTAB-map

### ROS2 

I hope you already have either ROS2 humble or Jazzy installed.
If you dont please refer the official documentation or one of my quick install scripts. 

### RTAB-map 

Install and setup RTAB map, you can find instructions on their [official git repo](https://github.com/introlab/rtabmap/tree/master).
```bash
sudo apt install ros-${ROS_DISTRO}-rtabmap-ros  
sudo apt install ros-${ROS_DISTRO}-rtabmap-viz
```

great now source ROS once more to make sure you have it all setup

```bash
source /opt/ros/humble/setup.sh 
```

```bash
source /opt/ros/jazzy/setup.sh 
```

whichever applies to you.

### Realsense SDK

#### [Installing the packages:](https://github.com/realsenseai/librealsense/blob/master/doc/distribution_linux.md#installing-the-packages)

- Register the server's public key:

```
# Ensure the directory exists
sudo mkdir -p /etc/apt/keyrings

# Download and dearmor
curl -sSf https://librealsense.realsenseai.com/Debian/librealsenseai.asc | \
gpg --dearmor | sudo tee /etc/apt/keyrings/librealsenseai.gpg > /dev/null
```

Note: The keyring contains both the new RS public key and the Intel public key for old repos, ensuring compatibility with both new and existing packages.

- Make sure apt HTTPS support is installed: `sudo apt-get install apt-transport-https`
    
- Add the server to the list of repositories:
    

```
echo "deb [signed-by=/etc/apt/keyrings/librealsenseai.gpg] https://librealsense.realsenseai.com/Debian/apt-repo `lsb_release -cs` main" | \
sudo tee /etc/apt/sources.list.d/librealsense.list
sudo apt-get update
```

- Install the libraries (see section below if upgrading packages):  
    `sudo apt-get install librealsense2-dkms`  
    `sudo apt-get install librealsense2-utils`  
    The above two lines will deploy librealsense2 udev rules, build and activate kernel modules, runtime library and executable demos and tools.
    
- Optionally install the developer and debug packages:  
    `sudo apt-get install librealsense2-dev`  
    `sudo apt-get install librealsense2-dbg`  
    With `dev` package installed, you can compile an application with **librealsense** using `g++ -std=c++11 filename.cpp -lrealsense2` or an IDE of your choice.
    

Reconnect the RealSense depth camera and run: `realsense-viewer` to verify the installation.

Verify that the kernel is updated :  
`modinfo uvcvideo | grep "version:"` should include `realsense` string


### relasense ROS 

You can either build from source 

#### Option 1: Install Debian package from ROS servers (Foxy EOL distro is not supported by this option):

- Configure your Ubuntu repositories
```bash
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt install curl # if you haven't already installed curl
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
```
 
- Install all realsense ROS packages by `sudo apt install ros-<ROS_DISTRO>-realsense2-*`
- For example, for Humble distro: `sudo apt install ros-humble-realsense2-*`

#### Option 2: Install from source

- Create a ROS2 workspace
```shell
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src/
```
- Clone the latest ROS Wrapper for RealSense™ cameras from [here](https://github.com/realsenseai/realsense-ros.git) into '~/ros2_ws/src/'
```
git clone https://github.com/realsenseai/realsense-ros.git -b ros2-master
cd ~/ros2_ws
```
- Install dependencies
```shell
sudo apt-get install python3-rosdep -y
sudo rosdep init # "sudo rosdep init --include-eol-distros" for Foxy and earlier
rosdep update # "sudo rosdep update --include-eol-distros" for Foxy and earlier
rosdep install -i --from-path src --rosdistro $ROS_DISTRO --skip-keys=librealsense2 -y
```

- Build
```shell
colcon build
```

- Source environment

```shell
ROS_DISTRO=<YOUR_SYSTEM_ROS_DISTRO>  # set your ROS_DISTRO: kilted, jazzy, iron, humble, foxy
source /opt/ros/$ROS_DISTRO/setup.bash
cd ~/ros2_ws
. install/local_setup.bash
```

### Finally run it

``` bash
ros2 launch rtabmap_examples realsense_d435i_color.launch.py
```

==Note : ==
before you start make sure in a surrounding with as less disturbance as possible. Make sure you are pointing your camera in the direction where you can get the most feature points, (Usually a place with less reflections an glares and a lot of obstacles). Move the camera slowly especially while turning.

If you feel the frame rate is low and compute is not being utilized fully try changing the DDS version being used for communication.
```bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
```
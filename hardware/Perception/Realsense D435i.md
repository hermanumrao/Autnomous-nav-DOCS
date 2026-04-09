![[Attachments/Pastedimage20260308194724.png]]


| Category              | Specification                  | Details                                     |
| --------------------- | ------------------------------ | ------------------------------------------- |
| **Environment**       | Use Environment                | Indoor / Outdoor                            |
| **Sensor Technology** | Depth Sensor Type              | Global Shutter                              |
|                       | Depth Technology               | Stereoscopic                                |
|                       | RGB Sensor Type                | Rolling Shutter                             |
| **Depth Camera**      | Ideal Range                    | 0.3 m – 3 m                                 |
|                       | Minimum Depth Distance (Min-Z) | ~28 cm at max resolution                    |
|                       | Depth Accuracy                 | < 2% at 2 m                                 |
|                       | Depth Field of View            | 87° × 58°                                   |
|                       | Depth Output Resolution        | Up to 1280 × 720                            |
|                       | Depth Frame Rate               | Up to 90 fps                                |
| **RGB Camera**        | RGB Resolution                 | 1920 × 1080                                 |
|                       | RGB Sensor Resolution          | 2 MP                                        |
|                       | RGB Frame Rate                 | 30 fps                                      |
|                       | RGB Field of View              | 69° × 42°                                   |
| **Major Components**  | Camera Module                  | RealSense D430 module + RGB camera          |
|                       | Vision Processor               | RealSense Vision Processor D4               |
| **Physical**          | Dimensions                     | 90 mm × 25 mm × 25 mm                       |
|                       | Connector                      | USB-C (USB 3.1 Gen 1)                       |
| **Mounting**          | Mounting Options               | 1 × 1/4-20 UNC thread, 2 × M3 thread mounts |

## SDK
https://github.com/realsenseai/librealsense

follow the instructions and setup the SDK 
then test with realsense-viewer
only if the viewer works, the setup is right or else something is missing.

### installation
```bash
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```

```bash
sudo apt install curl # if you haven't already installed curl
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
```

## ROS compatibility
https://github.com/realsenseai/realsense-ros

- [Configure](http://wiki.ros.org/Installation/Ubuntu/Sources) your Ubuntu repositories
- Install all realsense ROS packages by `sudo apt install ros-<ROS_DISTRO>-librealsense2*`
    - For example, for Humble distro: `sudo apt install ros-humble-librealsense2*`


## Start the camera node
#### with ros2 run:
```
ros2 run realsense2_camera realsense2_camera_node
# or, with parameters, for example - temporal and spatial filters are enabled:
ros2 run realsense2_camera realsense2_camera_node --ros-args -p enable_color:=false -p spatial_filter.enable:=true -p temporal_filter.enable:=true
```

#### with ros2 launch:
```
ros2 launch realsense2_camera rs_launch.py
ros2 launch realsense2_camera rs_launch.py depth_module.depth_profile:=1280x720x30 pointcloud.enable:=true
```

---

## Livox Mid 360

![[Attachments/MID360_working.mp4]]

### Spec

| Model                          | MID-360                                              |
| ------------------------------ | ---------------------------------------------------- |
| Laser Wavelength               | 905 nm                                               |
| Laser Safety¹\                 | Class 1 (IEC60825-1:2014)(Eye Safety)                |
| Detection Range (@ 100 klx)    | 40 m @ 10% reflectivity  <br>70 m @ 80% reflectivity |
| Close Proximity Blind Zone²    | 0.1 m                                                |
| FOV                            | Horizontal: 360°, Vertical: -7°~52°                  |
| Range Precision³ (1σ)          | ≤ 2 cm ⁴ (@ 10m)  <br>≤ 3 cm ⁵ (@ 0.2m)<br>          |
| Angular Precision（1σ ）         | < 0.15º                                              |
| Point Rate                     | 200,000 points/s (first return)                      |
| Frame Rate                     | 10 Hz (typical)                                      |
| Data Port                      | 100 BASE-TX Ethernet                                 |
| Data synchronization:          | IEEE 1588-2008 (PTPv2), GPS                          |
| Anti-Interference Function     | Available                                            |
| False Alarm Rate (@ 100 klx) ⁶ | < 0.01%                                              |
| IMU                            | Built-in IMU Model: ICM40609                         |
| Operating Temperature          | -4°F to 131°F (-20℃ to 55℃)⁷                         |
| IP Rating                      | IP67                                                 |
| Power ⁸                        | 6.5 W (average)                                      |
| Power Supply Voltage Range     | 9 ~ 27 V DC                                          |
| Dimensions                     | 65×65×60 mm                                          |
| Weight                         | 265 g                                                |

### Livox SDK

refer https://github.com/Livox-SDK/Livox-SDK2
1. Install the **CMake** using apt:

```shell
$ sudo apt install cmake
```

2. Compile and install the Livox-SDK2:

```shell
$ git clone https://github.com/Livox-SDK/Livox-SDK2.git
$ cd ./Livox-SDK2/
$ mkdir build
$ cd build
$ cmake .. && make -j
$ sudo make install
```


### Configuring the SDK

Start by checking out this:
`samples/livox_lidar_quick_start/mid360_config.json`

change the ip of the host_ip and multicast_ip
```json
{
  "MID360": {
    "lidar_net_info" : {
      "cmd_data_port"  : 56100,
      "push_msg_port"  : 56200,
      "point_data_port": 56300,
      "imu_data_port"  : 56400,
      "log_data_port"  : 56500
    },
    "host_net_info" : [
      {
        "host_ip"        : "192.168.1.82",
        "multicast_ip"   : "224.1.1.82",
        "cmd_data_port"  : 56101,
        "push_msg_port"  : 56201,
        "point_data_port": 56301,
        "imu_data_port"  : 56401,
        "log_data_port"  : 56501
      }
    ]
  }
}
```

### ROS for Livox

Livox has ros driver packages for interfacing the SDK with ros so that we can publish the data to ROS1/ROS2.
Fun part this package is good but might play glitchy. 

https://github.com/Livox-SDK/livox_ros2_driver

#### config setting:
```json
{
  "lidar_summary_info" : {
    "lidar_type": 8
  },
  "MID360": {
    "lidar_net_info" : {
      "cmd_data_port": 56100,
      "push_msg_port": 56200,
      "point_data_port": 56300,
      "imu_data_port": 56400,
      "log_data_port": 56500
    },
    "host_net_info" : {
      "cmd_data_ip" : "192.168.1.82",
      "cmd_data_port": 56101,
      "push_msg_ip": "192.168.1.82",
      "push_msg_port": 56201,
      "point_data_ip": "192.168.1.82",
      "point_data_port": 56301,
      "imu_data_ip" : "192.168.1.82",
      "imu_data_port": 56401,
      "log_data_ip" : "",
      "log_data_port": 56501
    }
  },
  "lidar_configs" : [
    {
      "ip" : "192.168.1.182",
      "pcl_data_type" : 1,
      "pattern_mode" : 0,
      "extrinsic_parameter" : {
        "roll": 0.0,
        "pitch": 0.0,
        "yaw": 0.0,
        "x": 0,
        "y": 0,
        "z": 0
      }
    }
  ]
}
```

#### Msgs
There are two formats in which the ROS driver comminicates:
 - the standard **PCL2**  format
 - the custom msgs, this is used in almost all the default packes from livox and even a few other packages.

#### 4.1 Launch file configuration instructions

https://github.com/Livox-SDK/livox_ros2_driver#41-launch-file-configuration-instructions

All launch files of livox_ros2_driver are in the "ws_livox/src/livox_ros2_driver/launch" directory. Different launch files have different configuration parameter values and are used in different scenarios :

|launch file name|Description|
|---|---|
|livox_lidar_rviz_launch.py|Connect to Livox LiDAR device  <br>Publish pointcloud2 format data  <br>Autoload rviz|
|livox_hub_rviz_launch.py|Connect to Livox Hub device  <br>Publish pointcloud2 format data  <br>Autoload rviz|
|livox_lidar_launch.py|Connect to Livox LiDAR device  <br>Publish pointcloud2 format data|
|livox_hub_launch.py|Connect to Livox LiDAR device  <br>Publish pointcloud2 format data|
|livox_lidar_msg_launch.py|Connect to Livox LiDAR device  <br>Publish livox customized pointcloud data|
|livox_hub_msg_launch.py|Connect to Livox Hub device  <br>Publish livox customized pointcloud data|

#### 4.2 Livox_ros2_driver internal main parameter configuration instructions

https://github.com/Livox-SDK/livox_ros2_driver#42-livox_ros2_driver-internal-main-parameter-configuration-instructions

All internal parameters of Livox_ros2_driver are in the launch file. Below are detailed descriptions of the three commonly used parameters :

|Parameter|Detailed description|Default|
|---|---|---|
|publish_freq|Set the frequency of point cloud publish  <br>Floating-point data type, recommended values 5.0, 10.0, 20.0, 50.0, etc.|10.0|
|multi_topic|If the LiDAR device has an independent topic to publish pointcloud data  <br>0 -- All LiDAR devices use the same topic to publish pointcloud data  <br>1 -- Each LiDAR device has its own topic to publish point cloud data|0|
|xfer_format|Set pointcloud format  <br>0 -- Livox pointcloud2(PointXYZRTL) pointcloud format  <br>1 -- Livox customized pointcloud format  <br>2 -- Standard pointcloud2 (pcl :: PointXYZI) pointcloud format in the PCL library|0|

    _**livox_ros2_driver pointcloud data detailed description :**_

1. Livox pointcloud2 (PointXYZRTL) point cloud format, as follows :

```c
float32 x               # X axis, unit:m
float32 y               # Y axis, unit:m
float32 z               # Z axis, unit:m
float32 intensity         # the value is reflectivity, 0.0~255.0
uint8 tag               # livox tag
uint8 line              # laser number in lidar
```

2. Livox customized data package format, as follows :

```c
Header header             # ROS standard message header
uint64 timebase           # The time of first point
uint32 point_num          # Total number of pointclouds
uint8  lidar_id           # Lidar device id number
uint8[3]  rsvd            # Reserved use
CustomPoint[] points      # Pointcloud data
```

    Customized Point Cloud (CustomPoint) format in the above customized data package :

```c
uint32 offset_time      # offset time relative to the base time
float32 x               # X axis, unit:m
float32 y               # Y axis, unit:m
float32 z               # Z axis, unit:m
uint8 reflectivity      # reflectivity, 0~255
uint8 tag               # livox tag
uint8 line              # laser number in lidar
```

3. The standard pointcloud2 (pcl :: PointXYZI) format in the PCL library (Not currently supported) :

    Please refer to the pcl :: PointXYZI data structure in the point_types.hpp file of the PCL library.


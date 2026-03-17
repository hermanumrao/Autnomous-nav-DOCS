![[Attachments/Pasted image 20260308154022.png]] 


| ![](https://d1c6gk3tn6ydje.cloudfront.net/2036899223840006144%2F5b9c14d87437932dbc7350b721519740.webp) | Range Frequency  <br>5000Hz      | ![](https://d1c6gk3tn6ydje.cloudfront.net/2036899223840006144%2Ff5d35334a1afa642a1d6734c7fbabb65.webp) | Scan Frequency  <br>6-12Hz  |
| ------------------------------------------------------------------------------------------------------ | -------------------------------- | ------------------------------------------------------------------------------------------------------ | --------------------------- |
| ![](https://d1c6gk3tn6ydje.cloudfront.net/2036899223840006144%2F067cdc1aa4d470e2e694d2fde3bb522f.webp) | Range Distance  <br>0.12-10m     | ![](https://d1c6gk3tn6ydje.cloudfront.net/2036899223840006144%2F50a54e807045a73d580680b0bb897cba.webp) | Scan Angle  <br>360°        |
| ![](https://d1c6gk3tn6ydje.cloudfront.net/2036899223840006144%2Fdeedbd3dab4b5270a0f1eeccec2ea381.webp) | Angle Resolution  <br>0.43-0.85° | ![](https://d1c6gk3tn6ydje.cloudfront.net/2036899223840006144%2F57ca55c7d412003d9030dfae6a081776.webp) | Size  <br>110.6*71.1*52.3mm |


## SDK
https://github.com/YDLIDAR/YDLidar-SDK/tree/master
You can install these packages using apt:

```shell
sudo apt install cmake pkg-config
```

In the YDLidar SDK directory, run the following commands to compile the project:

```
git clone https://github.com/YDLIDAR/YDLidar-SDK.git
cd YDLidar-SDK
mkdir build
cd build
cmake ..
make
sudo make install
```


## ROS2 Driver
https://github.com/YDLIDAR/ydlidar_ros2_driver

| launch file            | features                                                                                                                       |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| ydlidar.py             | Connect to defualt paramters  <br>Publish LaserScan message on `scan` topic                                                    |
| ydlidar_launch.py      | Connect ydlidar.yaml Lidar specified by configuration parameters  <br>Publish LaserScan message on `scan` topic                |
| ydlidar_launch_view.py | Connect ydlidar.yaml Lidar specified by configuration parameters and setup RVIZ  <br>Publish LaserScan message on `scan` topic |
1. Clone ydlidar_ros2_driver master branch from github for old version:
    
    `git clone https://github.com/YDLIDAR/ydlidar_ros2_driver.git ydlidar_ros2_ws/src/ydlidar_ros2_driver`
    
    Clone ydlidar_ros2_driver humble branch from github for humble,jazzy,etc:
    
    `git clone -b humble https://github.com/YDLIDAR/ydlidar_ros2_driver.git ydlidar_ros2_ws/src/ydlidar_ros2_driver`
    
2. Build ydlidar_ros2_driver package :
    
    ```
    cd ydlidar_ros2_ws
    colcon build --symlink-install
    ```
    
    Note: install colcon [see](https://index.ros.org/doc/ros2/Tutorials/Colcon-Tutorial/#install-colcon)
    
    [![CMAKE Finished](https://github.com/YDLIDAR/ydlidar_ros2_driver/raw/master/images/finished.png "CMAKE Finished")](https://github.com/YDLIDAR/ydlidar_ros2_driver/blob/master/images/finished.png)
    
    >Note: If the following error occurs, Please install [YDLIDAR/YDLidar-SDK](https://github.com/YDLIDAR/YDLidar-SDK) first.
    
    [![CMAKE ERROR](https://github.com/YDLIDAR/ydlidar_ros2_driver/raw/master/images/cmake_error.png "CMAKE ERROR")](https://github.com/YDLIDAR/ydlidar_ros2_driver/blob/master/images/cmake_error.png)
    
3. Package environment setup :
    
    `source ./install/setup.bash`
    
    Note: Add permanent workspace environment variables. It's convenientif the ROS2 environment variables are automatically added to your bash session every time a new shell is launched:
    
    ```
    echo "source ~/ydlidar_ros2_ws/install/setup.bash" >> ~/.bashrc
    source ~/.bashrc
    ```
    
4. Confirmation To confirm that your package path has been set, printenv the `grep -i ROS` variable.
    
    ```
    printenv | grep -i ROS
    ```
    
    You should see something similar to: `OLDPWD=/home/tony/ydlidar_ros2_ws/install`
    
5. Create serial port Alias [optional]
    
    ```
    chmod 0777 src/ydlidar_ros2_driver/startup/*
    sudo sh src/ydlidar_ros2_driver/startup/initenv.sh
    ```
    
    Note: After completing the previous operation, replug the LiDAR again.
    

## Configure LiDAR [Default parameter file](https://github.com/YDLIDAR/ydlidar_ros2_driver/blob/master/params/ydlidar.yaml)

[](https://github.com/YDLIDAR/ydlidar_ros2_driver#configure-lidar-default-parameter-file)

```
ydlidar_ros2_driver_node:
  ros__parameters:
    port: /dev/ttyUSB0
    frame_id: laser_frame
    ignore_array: ""
    baudrate: 230400
    lidar_type: 1
    device_type: 0
    isSingleChannel: false
    intensity: false
    intensity_bit: 0
    sample_rate: 9
    abnormal_check_count: 4
    fixed_resolution: true
    reversion: false
    inverted: false
    auto_reconnect: true
    support_motor_dtr: false
    angle_max: 180.0
    angle_min: -180.0
    range_max: 64.0
    range_min: 0.01
    frequency: 10.0
    invalid_range_is_inf: false
    debug: false
```

Note: It needs to be modified according to LiDAR actual situation.

## Run ydlidar_ros2_driver

[](https://github.com/YDLIDAR/ydlidar_ros2_driver#run-ydlidar_ros2_driver)

##### Run ydlidar_ros2_driver using launch file

[](https://github.com/YDLIDAR/ydlidar_ros2_driver#run-ydlidar_ros2_driver-using-launch-file)

The command format is :

`ros2 launch ydlidar_ros2_driver [launch file].py`

1. Connect LiDAR uint(s).
    
    ```
    ros2 launch ydlidar_ros2_driver ydlidar_launch.py 
    ```
    
    or
    
    ```
    launch $(ros2 pkg prefix ydlidar_ros2_driver)/share/ydlidar_ros2_driver/launch/ydlidar.py 
    ```
    
2. RVIZ
    
    ```
    ros2 launch ydlidar_ros2_driver ydlidar_launch_view.py 
    ```
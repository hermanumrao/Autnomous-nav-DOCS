## 1. Background and Problem Statement

Robotic systems (autonomous vehicles, UAVs, handheld mapping devices, mobile robots) require **accurate real-time localization and mapping** in environments where GPS may not be available.

---
## 2. Goal of FAST-LIO2

The authors propose **FAST-LIO2**, a LiDAR-IMU fusion framework that aims to:

- Achieve **high-speed real-time odometry**
- Work with **different LiDAR types**
- Avoid **feature extraction**
- Provide **accurate mapping**
- Run on **resource-constrained hardware**

The system can achieve **>100 Hz odometry update rates** and works even with high angular velocity motions (≈1000°/s).
![](Attachments/Pasted%20image%2020260316095152.png)

---


## 3. Key Contributions of the Paper

FAST-LIO2 introduces two major innovations.

### 3.1 Direct LiDAR Scan-to-Map Registration

Most LIO systems:
`LiDAR scan → feature extraction → match features → pose update`

FAST-LIO2 instead performs:

`LiDAR raw points → scan-to-map matching → pose update`

This removes the need for manually engineered features.

Advantages:
- Works with **any LiDAR scanning pattern**
- Uses **all geometric information**
- Reduces preprocessing computation

This approach improves **accuracy and robustness** because subtle geometric features can still influence the optimization.

---

### 3.2 Incremental KD-Tree Map (ikd-Tree)

FAST-LIO2 introduces a new dynamic spatial data structure called:

**ikd-Tree (Incremental KD-Tree)**.

Capabilities:
- Efficient nearest-neighbor search
- Incremental point insertion
- Dynamic deletion
- Automatic rebalancing
- Built-in voxel downsampling

Compared with structures like:
- Octrees
- R-trees
- static KD-trees

ikd-Tree provides **better real-time performance for large maps**.

---

## 4. System Architecture

FAST-LIO2 consists of three main components:

IMU Preintegration  
        ↓  
State Estimator (Iterated EKF)  
        ↓  
LiDAR Scan-to-Map Matching  
        ↓  
Map Update (ikd-tree)


Pipeline:
1. IMU propagation
2. Motion compensation
3. Point-to-map registration
4. EKF update
5. Map update

---

## 5. Mathematical Formulation

FAST-LIO2 uses an **Iterated Extended Kalman Filter (IEKF)**.

State vector:

x = [R, p, v, bg, ba]

Where:

- R → rotation
- p → position
- v → velocity
- bg → gyroscope bias
- ba → accelerometer bias

---

### 5.1 IMU Propagation

The IMU predicts motion using:

Rotation:
R_k = R_{k-1} * Exp((ω - bg)Δt)

Velocity:
v_k = v_{k-1} + (R(a - ba) + g)Δt

Position:
p_k = p_{k-1} + vΔt + ½(R(a - ba) + g)Δt²

This provides a **high-frequency motion estimate**.

---

### 5.2 Motion Compensation (Deskewing)

Because LiDAR scans take time (~100 ms), points correspond to different poses.

FAST-LIO2 uses **IMU interpolation** to transform each LiDAR point to a common reference frame.

This process is called:

**scan deskewing**

---

### 5.3 Scan-to-Map Registration

Instead of feature matching, FAST-LIO2 performs:

**direct point-to-plane optimization**

For each point:

1. Find nearest neighbors in map
2. Fit local plane
3. Minimize point-to-plane distance

Error function:

e = nᵀ (R p + t − p_map)

Where:

- n = plane normal
- p = LiDAR point
- p_map = map point

This becomes the measurement update in the Kalman filter.

---

### 5.4 Iterated Kalman Update

FAST-LIO2 performs **iterative EKF updates** to improve convergence.

Process:

prediction → measurement update → iterate → final state

Benefits:

- Higher accuracy
- Robust against nonlinearities

---

## 6. Map Management with ikd-Tree

Traditional SLAM systems store maps using:

- voxel grids
- octrees

FAST-LIO2 instead uses **ikd-Tree**.

Advantages:

### Incremental update
Points inserted directly into tree.

### Efficient nearest neighbor search
Used for scan-to-map matching.

### Dynamic map maintenance
Points can be removed when outside local map region.

### Built-in downsampling
Reduces map size automatically.

This dramatically reduces computation load.

---

## 7. Performance Evaluation

The paper evaluates FAST-LIO2 on:

- **19 benchmark sequences**
- Multiple LiDAR types
- Indoor and outdoor environments

Datasets include:
- Livox datasets
- UAV datasets
- handheld mapping

Results show:
- Lower trajectory error than LIO-SAM and LINS
- Real-time mapping at **100 Hz**
- Stable operation on **ARM processors**

---

## Example Performance

Handheld mapping:

- Speed: 7 m/s
- Drift: < 6 cm

UAV experiment:

- aggressive motion
- accurate dense mapping

---

# 8. Advantages of FAST-LIO2

### 1. Feature-free
Works with any LiDAR.

### 2. High speed
> 100 Hz update rate.

### 3. High robustness
Handles fast rotations (~1000°/s).

### 4. Sensor flexibility
Supports:
- Velodyne
- Ouster
- Livox
- solid-state LiDAR

### 5. Embedded support

Runs on ARM boards like:
- Raspberry Pi
- Jetson

---

## 9. Limitations

Despite its strengths, FAST-LIO2 has some limitations.

### No loop closure
It performs **odometry and local mapping**, not full SLAM.
Long-term drift can occur.

### Sensitive to IMU calibration
Accurate LiDAR-IMU extrinsics are required.

### Degenerate environments
Feature-poor environments (e.g., tunnels, corridors) can reduce accuracy.

---

## 10. Applications

FAST-LIO2 is widely used for:

### Autonomous robots

UGVs and mobile robots.

### UAV navigation

High-speed drone mapping.

### Handheld 3D scanning

### Underground exploration

### Autonomous vehicles

---

## 11. Why FAST-LIO2 Became Popular

FAST-LIO2 is widely adopted because it:

1. Eliminates fragile feature extraction
2. Uses efficient Kalman filtering
3. Maintains a fast map structure
4. Works with modern LiDAR types (especially **Livox**)

This combination made it one of the **most practical LIO systems** in robotics.

---

## 12. Comparison with Other SLAM Systems

|System|Method|Speed|Feature Extraction|
|---|---|---|---|
|LOAM|feature based|medium|yes|
|LIO-SAM|factor graph|medium|yes|
|LINS|EKF|medium|yes|
|FAST-LIO2|direct LIO|very high|no|

---

## 13. Setup

Clone the repository and colcon build:

```shell
cd <ros2_ws>/src # cd into a ros2 workspace folder
git clone https://github.com/Ericsii/FAST_LIO.git --recursive
cd ..
rosdep install --from-paths src --ignore-src -y
colcon build --symlink-install
```

- **Remember to source the livox_ros_driver before build (follow [1.3 livox_ros_driver](https://github.com/hku-mars/FAST_LIO/tree/ROS2#1.3))**
- If you want to use a custom build of PCL, add the following line to ~/.bashrc `export PCL_ROOT={CUSTOM_PCL_PATH}`

---
## 14. Run

Launch livox ros driver. Use MID360 as an example.
```shell
source ~/ws_livox/install/local_setup.bash 
source ~/fastlio/install/local_setup.bash

ros2 launch fast_lio mapping.launch.py config_file:=mid360.yaml & ros2 launch livox_ros_driver2 msg_MID360_launch.py
```


to save maps:
```bash
source ~/fastlio/install/local_setup.bash
ros2 service call /map_save std_srvs/srv/Trigger {}
```


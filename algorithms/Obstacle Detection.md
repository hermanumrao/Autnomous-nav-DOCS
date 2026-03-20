Here’s a clean, technical **README summary** you can directly use (or slightly tweak) for your ROS 2 package:

---

# 🧭 Obstacle Detector (ROS 2)

## Overview

`obstacle_detector` is a lightweight ROS 2 package for real-time obstacle detection and classification using 2D LiDAR data (`sensor_msgs/LaserScan`). It processes incoming scan data, filters noise, groups nearby points into clusters, and categorizes obstacles into safety zones based on distance.

The node publishes both **visualization markers** for RViz and a **high-level obstacle state** for downstream decision-making (e.g., navigation, control).

---

## ✨ Features
- 📡 Subscribes to LiDAR scan data (`/scan`)
- 🧹 Applies moving-average filtering to reduce noise
- 📍 Converts polar scan data → Cartesian coordinates
- 🧠 Classifies obstacles into:
    - **CRITICAL**
    - **MODERATE**
    - **FRIENDLY**
    - **CLEAR**
- 🔗 Performs simple spatial clustering of obstacle points
- 🎨 Publishes clustered obstacles as RViz markers
- 📢 Publishes system state for behavior/control modules

---

## 🏗️ Node Architecture

### Node: `obstacle_detector`

#### Subscriptions

|Topic|Type|Description|
|---|---|---|
|`/scan`|`sensor_msgs/msg/LaserScan`|Input LiDAR data|

#### Publications

|Topic|Type|Description|
|---|---|---|
|`/obstacle_clusters`|`visualization_msgs/msg/Marker`|Clustered obstacle visualization|
|`/obstacle_mode`|`std_msgs/msg/String`|Current obstacle state|

---

## ⚙️ Parameters

|Parameter|Type|Description|Default|
|---|---|---|---|
|`rover_length`|float|Robot length (m)|0.7|
|`rover_width`|float|Robot width (m)|0.5|
|`lidar_offset_x`|float|LiDAR X offset|0.0|
|`lidar_offset_y`|float|LiDAR Y offset|0.0|
|`critical_radius`|float|Immediate danger zone|0.7|
|`moderate_radius`|float|Caution zone|1.5|
|`friendly_radius`|float|Awareness zone|2.0|

---

## Processing Pipeline

### 1. Noise Filtering

A sliding window (moving average) is applied to smooth LiDAR range data:

- Window size: ±2 samples
- Ignores `NaN` and `inf` values

---

### 2. Coordinate Transformation

Each valid scan point is converted:

- Polar → Cartesian
```
x = r * cos(θ)
y = r * sin(θ)
```

---

### 3. Distance-Based Classification

Points are categorized into zones:

- **Critical**: `r ≤ critical_radius`
- **Moderate**: `critical_radius < r ≤ moderate_radius`
- **Friendly**: `moderate_radius < r ≤ friendly_radius`

---

### 4. Clustering

A simple proximity-based clustering algorithm:

- Threshold: **0.4 m**
- Groups nearby points into obstacle clusters
- Computes:
    - Cluster centroid
    - Distance and angle
    - Cluster size

---

### 5. Visualization

- Uses `Marker::SPHERE_LIST`
- Each cluster gets a unique color
- Published to `/obstacle_clusters` for RViz

---

### 6. Mode Estimation

The system publishes a global state:

Priority-based logic:

```
CRITICAL > MODERATE > FRIENDLY > CLEAR
```

Example:

- Any critical point → `CRITICAL`
- Else if moderate → `MODERATE`
- Else if friendly → `FRIENDLY`
- Else → `CLEAR`


---

## Launch

```bash
ros2 launch obstacle_detector detector_launch.py
```

---

## Dependencies

- `rclcpp`
- `sensor_msgs`
- `visualization_msgs`
- `std_msgs`

---

## Example Use Cases

- Autonomous rover safety monitoring
- Reactive obstacle avoidance
- Navigation stack augmentation
- Visualization/debugging of LiDAR perception

---

## Limitations

- Uses a **naive clustering algorithm** (not DBSCAN or Euclidean clustering)
- No temporal tracking of obstacles
- No dynamic obstacle prediction
- Assumes relatively clean LiDAR input

---

## Future Improvements

- Replace clustering with DBSCAN / PCL
- Add obstacle tracking over time
- Integrate velocity estimation
- Fuse with camera/depth sensors
- Adaptive thresholds based on speed
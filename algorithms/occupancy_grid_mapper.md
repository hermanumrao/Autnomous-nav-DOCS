## System Overview

This package implements a **custom 2D occupancy grid mapping pipeline** in ROS2 using:

- **Sensor fusion inputs**:
    
    - `sensor_msgs/LaserScan` → spatial observations
    - `nav_msgs/Odometry` → robot pose
    - `sensor_msgs/Imu` → motion filtering
        
- **Core output**:
    - `nav_msgs/OccupancyGrid` published on `/map`
        
- **Auxiliary capability**:
    
    - Periodic map persistence via:
        - Internal PGM writer (C++)
        - External Nav2 `SaveMap` service (Python node)

---

## Core Architecture

### Node: `OccupancyGridMapper` (C++)

#### Subscriptions

- `/scan` → main mapping input
- `/odom` → pose tracking
- `/livox/imu` → motion quality filtering

#### Publisher

- `/map` → real-time occupancy grid

#### Internal State

```cpp
std::vector<int8_t> grid_;     // occupancy values: {-1, 0, 100}
std::vector<int> last_seen_;   // temporal persistence tracking
geometry_msgs::msg::Pose robot_pose_;
```

#### Grid Configuration

- Resolution: **0.1 m/cell**
- Initial size: **200 × 200**
- Origin: centered around robot using offset

---

## Mapping Pipeline

### 1. Coordinate Transformation

Laser scan points are transformed:

$[  (x_{world}, y_{world}) = T_{robot} \cdot (r\cos\theta, r\sin\theta)  ]$

Where:
- `T_robot` derived from odometry pose + yaw (via `tf2::getYaw`)
- No TF tree used → **direct pose-based transform**

### 2. Ray Casting (Free Space Inference)

#### Algorithm: **Bresenham Line Tracing**

Function:

```cpp
std::vector<int> traceRay(int x0, int y0, int x1, int y1)
```

- Computes all grid cells between robot and endpoint
- Marks them as **free (0)**

#### Key behavior:

```cpp
if (grid_[idx] != 100) grid_[idx] = 0;
```

 Occupied cells are **not overwritten by free space immediately**

### 3. Occupancy Update

For each valid laser return:
- **Ray path** → free cells
- **Endpoint** → occupied

```cpp
grid_[idx] = 100;
last_seen_[idx] = frame_count_;
```

### 4. Temporal Decay Model

#### Purpose:

Handle **dynamic environments / stale obstacles**

#### Mechanism:

```cpp
if ((frame_count_ - last_seen_[idx]) > decay_threshold_) {
    grid_[idx] = 0;
}
```

- `decay_threshold_ = 30 frames`
- Implements **time-based occupancy decay**
- Prevents ghost obstacles

### 5. Frame Skipping via IMU (Motion Filtering)

#### Problem Addressed:

- Motion distortion in LiDAR scans
- Mapping errors during rapid rotation


### IMU Filtering Strategy

#### Step 1: Low-pass filtering

$[ \omega_{filtered} = \alpha \cdot \omega_{raw} + (1-\alpha)\cdot \omega_{prev}  ]$

- `α = 0.01` → strong smoothing

#### Step 2: Angular displacement estimation

$[  \Delta \theta = |\omega| \cdot dt  ]$

- `dt = 1/200` (Livox IMU rate)


#### Step 3: Adaptive spike detection

Conditions:

```cpp
delta_roll > threshold || delta_pitch > threshold || delta_yaw > threshold
```


#### Step 4: Dynamic hysteresis model

Key innovation:

```cpp
dynamic_threshold ∈ [0.1s, 2.0s]
```

- Increases during motion spikes
- Decreases during calm periods

#### Behavior:

| Motion Type      | Action            |
| ---------------- | ----------------- |
| Short spike      | Drop frames       |
| Sustained motion | Resume mapping    |
| Calm period      | Reset sensitivity |

 This avoids **over-filtering during continuous motion**


##  Dynamic Map Resizing

### Function: `ensureMapContains()`

#### Trigger:

- When robot or scan endpoints approach boundary

#### Strategy:

- Expand grid by **+40 cells**
- Re-center around robot

#### Data Migration:

```cpp
new_grid[new_idx] = old_grid[old_idx]
```
 Maintains spatial consistency


#### Optimization Constraint

```cpp
if (dx > 20 || dy > 20) return;
```

- Only expands when **near robot**
- Avoids unbounded growth    

---

##  Map Representation

### Encoding

| Value | Meaning  | PGM Output |
| ----- | -------- | ---------- |
| -1    | Unknown  | 127        |
| 0     | Free     | 255        |
| 100   | Occupied | 0          |



### Publishing

- Frame: `"map"`
- Origin:  
    $(-\frac{width \cdot resolution}{2}, -\frac{height \cdot resolution}{2})$  

 Robot stays approximately centered

---
##  Map Persistence

### Method 1: Internal (C++)

- Writes **PGM image**
- Trigger: every 10 seconds
- Limitation:
    - No YAML metadata (resolution/origin missing)

### Method 2: External (Python Node)

### Node: `AutoMapSaver`

- Calls Nav2 service:
```
    /map_saver/save_map
```
    
- Generates timestamped map:
```
    ~/map_YYYYMMDD_HHMMSS
```

 Produces:
- `.pgm` + `.yaml` (Nav2 compatible)


---

## Design Strengths

### Robustness

- IMU-based **motion-aware filtering**
- Temporal decay for dynamic environments

### Efficiency

- Lightweight grid (no probabilistic log-odds)
- Bresenham ray tracing (fast, integer-based)

###  Adaptability

- Dynamic grid resizing
- Hysteresis-based filtering

### Simplicity

- No TF dependency
- Minimal external packages

---

##  Limitations / Trade-offs

### No probabilistic model

- Uses **binary occupancy**
- Lacks Bayesian update:  

    $p(m|z) \neq \text{modeled}$  


### No sensor noise modeling

- No inverse sensor model
- All hits treated equally

### Pose accuracy dependency

- Relies entirely on `/odom`
- No SLAM loop closure

### No multi-layer costmaps

- Not Nav2 costmap-compatible directly

### Memory growth

- Grid expands but never shrinks


---

##  Notable Innovations

### 1. **Dynamic IMU Hysteresis Filter**

- Rare in basic mappers
- Prevents both:
    - motion blur
    - over-filtering

### 2. **Time-decay occupancy**

- Lightweight alternative to probabilistic aging

### 3. **Selective map expansion**

- Balances:
    - coverage
    - computational cost

---

##  Comparison to Standard Approaches

| Feature            | This Mapper | GMapping / Cartographer |
| ------------------ | ----------- | ----------------------- |
| Probabilistic grid | ❌           | ✅                       |
| Loop closure       | ❌           | ✅                       |
| IMU filtering      | ✅ (custom)  | ✅ (integrated)          |
| Dynamic resizing   | ✅           | ❌ (fixed maps)          |
| Complexity         | Low         | High                    |

---


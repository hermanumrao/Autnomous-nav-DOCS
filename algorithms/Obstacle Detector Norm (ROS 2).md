
## Overview

`obstacle_detector_norm` is an advanced ROS 2 perception package for **normalized, noise-robust obstacle detection** from 2D LiDAR data. It extends conventional scan-based detection pipelines by introducing **signal normalization, adaptive filtering, and structured spatial reasoning**, enabling improved robustness under noisy or inconsistent sensor conditions.

The package is designed for **real-time robotic systems**, providing both **low-level geometric representations** and **high-level semantic safety states**.

---

## Core Design Philosophy

Unlike naive obstacle detectors, this package emphasizes:

- **Signal normalization before interpretation**
- **Distance-invariant feature extraction**
- **Robustness to sensor noise and dropouts**
- **Deterministic, low-latency processing (no heavy ML dependencies)**

---

## Key Features

- Real-time `LaserScan` processing
- Multi-stage filtering (temporal + spatial)
- Range normalization for consistent clustering behavior
- Adaptive clustering based on normalized distance metrics
- Geometric feature extraction (centroids, spread, density)
- Multi-zone obstacle classification
- RViz visualization with structured cluster encoding
- State abstraction for navigation/control layers

---

## System Architecture

### Node: `obstacle_detector_norm`

A single high-performance node implementing a full perception pipeline:

```
LaserScan → Filtering → Normalization → Projection → Clustering → Classification → Publishing
```

---

## Processing Pipeline (Deep Dive)

### 1. Signal Conditioning

#### 1.1 Temporal Filtering

Applies a sliding window over scan frames:

- Reduces jitter between consecutive scans
- Handles transient noise spikes

#### 1.2 Spatial Smoothing

Local neighborhood averaging:

- Windowed smoothing across adjacent beams
- Rejects:
    - `NaN`
    - `inf`
    - Out-of-range values

### 2. Range Normalization

A key differentiator of this package.

#### Motivation:

Raw LiDAR data exhibits **range-dependent sparsity**:

- Close objects → dense points
- Far objects → sparse points

#### Approach:

Each range value `r` is normalized:

```
r_norm = r / r_max
```

or optionally:

```
r_norm = (r - r_min) / (r_max - r_min)
```

#### Effects:

- Equalizes clustering sensitivity across distances
- Prevents bias toward near-field obstacles
- Stabilizes threshold-based logic

### 3. Coordinate Projection

Standard polar → Cartesian transformation:

```
x = r * cos(θ)
y = r * sin(θ)
```

Optionally includes:

- Sensor frame offsets
- Robot base frame alignment

### 4. Adaptive Clustering

#### Key Improvement:

Clustering threshold is **scaled using normalized distance**:

```
d_threshold = α * (1 + r_norm)
```

Where:

- `α` = base clustering constant
- `r_norm` = normalized range

#### Benefits:

- Tight clustering for near objects
- Relaxed clustering for far objects
- Reduces over-segmentation in sparse regions

#### Algorithm:

- Sequential scan grouping (O(n))
- Euclidean proximity check
- Cluster merging (optional optimization)

### 5. Feature Extraction

For each cluster:

- **Centroid**
- **Point count (density proxy)**
- **Bounding radius**
- **Angular span**
- **Distance from robot**

Derived metrics:

- Compactness
- Spatial variance

### 6. Obstacle Classification

Multi-zone classification using normalized + absolute metrics:

| Zone     | Condition                    |
| -------- | ---------------------------- |
| CRITICAL | Near + dense cluster         |
| MODERATE | Mid-range structured cluster |
| FRIENDLY | Sparse or distant cluster    |
| CLEAR    | No significant clusters      |

Hybrid logic:

- Distance thresholds (absolute)
- Density thresholds (relative)
- Cluster geometry    

### 7. Mode Aggregation

Global system state is determined via priority reduction:

```
CRITICAL > MODERATE > FRIENDLY > CLEAR
```

This ensures **fail-safe behavior** for navigation stacks.

---

## Interfaces

### Subscriptions

| Topic   | Type                        | Description     |
| ------- | --------------------------- | --------------- |
| `/scan` | `sensor_msgs/msg/LaserScan` | Raw LiDAR input |

---

### Publications

| Topic                             | Type                            | Description             |
| --------------------------------- | ------------------------------- | ----------------------- |
| `/obstacle_clusters_norm`         | `visualization_msgs/msg/Marker` | Cluster visualization   |
| `/obstacle_state_norm`            | `std_msgs/msg/String`           | Global state            |
| `/obstacle_features` _(optional)_ | custom/msg                      | Structured cluster data |

---

## Parameters

### Geometry

| Parameter          | Description                   |
| ------------------ | ----------------------------- |
| `lidar_offset_x/y` | Sensor position offset        |
| `robot_radius`     | Robot footprint approximation |

---

### Filtering

| Parameter       | Description       |
| --------------- | ----------------- |
| `window_size`   | Smoothing window  |
| `range_min/max` | Valid scan limits |

---

### Normalization

| Parameter            | Description                  |
| -------------------- | ---------------------------- |
| `normalize`          | Enable/disable normalization |
| `normalization_mode` | min-max / max scaling        |

---

### Clustering

| Parameter                | Description                |
| ------------------------ | -------------------------- |
| `cluster_base_threshold` | Base proximity threshold   |
| `adaptive_scaling`       | Enable adaptive clustering |

---

### Classification

| Parameter           | Description            |
| ------------------- | ---------------------- |
| `critical_radius`   | Danger zone            |
| `density_threshold` | Cluster density cutoff |

---

## Launch

```bash
ros2 launch obstacle_detector_norm detector_norm.launch.py
```

---

## Performance Characteristics

| Metric          | Value                                 |
| --------------- | ------------------------------------- |
| Time Complexity | O(n) per scan                         |
| Latency         | < 10 ms (typical)                     |
| Memory          | Minimal (no history buffers required) |

---

## Design Trade-offs

### Advantages

- Deterministic (no ML inference latency)    
- Robust to noise and sparsity
- Adaptive to varying environments
- Lightweight and embedded-friendly

### Limitations

- No temporal tracking (stateless clustering)
- No semantic classification (object type)
- Assumes planar LiDAR    
- Limited performance in highly dynamic scenes

---

## Future Work

- Temporal tracking (Kalman / JPDAF)
- Velocity estimation from scan differencing
- 3D LiDAR support
- Sensor fusion (camera + depth)
- Learned clustering (hybrid ML + geometry)
- Integration with Nav2 costmaps
---

## Integration Notes

This package is ideal for:
- Pre-processing layer before Nav2
- Safety watchdog nodes
- Behavior trees (BT Navigator)
- Edge robotics platforms (Jetson, Raspberry Pi)    

---

## Dependencies
- `rclcpp`
- `sensor_msgs`
- `visualization_msgs`
- `std_msgs`
---
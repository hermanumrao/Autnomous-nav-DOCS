## ORB (Oriented FAST and Rotated BRIEF)
As an OpenCV enthusiast, the most important thing about the ORB is that it came from "OpenCV Labs". This algorithm was brought up by Ethan Rublee, Vincent Rabaud, Kurt Konolige and Gary R. Bradski in their paper **ORB: An efficient alternative to SIFT or SURF** in 2011. As the title says, it is a good alternative to SIFT and SURF in computation cost, matching performance and mainly the patents. Yes, SIFT and SURF are patented and you are supposed to pay them for its use. But ORB is not !!!

ORB is basically a fusion of FAST keypoint detector and BRIEF descriptor with many modifications to enhance the performance. First it use FAST to find keypoints, then apply Harris corner measure to find top N points among them. It also use pyramid to produce multiscale-features. But one problem is that, FAST doesn't compute the orientation. So what about rotation invariance? Authors came up with following modification.

It computes the intensity weighted centroid of the patch with located corner at center. The direction of the vector from this corner point to centroid gives the orientation. To improve the rotation invariance, moments are computed with x and y which should be in a circular region of radius , where  is the size of the patch.

Now for descriptors, ORB use BRIEF descriptors. But we have already seen that BRIEF performs poorly with rotation. So what ORB does is to "steer" BRIEF according to the orientation of keypoints. For any feature set of `n` binary tests at location `(x,y)`, define a `(2Xn)` matrix, `S` which contains the coordinates of these pixels. Then using the orientation of patch, `𝜃`, its rotation matrix is found and rotates the `S` to get steered(rotated) version `S𝜃`.

ORB discretize the angle to increments of `2π/30`(12 degrees), and construct a lookup table of precomputed BRIEF patterns. As long as the keypoint orientation `S𝜃` is consistent across views, the correct set of points  will be used to compute its descriptor.

BRIEF has an important property that each bit feature has a large variance and a mean near 0.5. But once it is oriented along keypoint direction, it loses this property and become more distributed. High variance makes a feature more discriminative, since it responds differentially to inputs. Another desirable property is to have the tests uncorrelated, since then each test will contribute to the result. To resolve all these, ORB runs a greedy search among all possible binary tests to find the ones that have both high variance and means close to 0.5, as well as being uncorrelated. The result is called **rBRIEF**.

For descriptor matching, multi-probe LSH which improves on the traditional LSH, is used. The paper says ORB is much faster than SURF and SIFT and ORB descriptor works better than SURF. ORB is a good choice in low-power devices for panorama stitching etc.
### ORB in OpenCV

As usual, we have to create an ORB object with the function, **[cv.ORB()](https://docs.opencv.org/4.x/db/d95/classcv_1_1ORB.html "Class implementing the ORB (oriented BRIEF) keypoint detector and descriptor extractor.")** or using feature2d common interface. It has a number of optional parameters. Most useful ones are nFeatures which denotes maximum number of features to be retained (by default 500), scoreType which denotes whether Harris score or FAST score to rank the features (by default, Harris score) etc. Another parameter, WTA_K decides number of points that produce each element of the oriented BRIEF descriptor. By default it is two, ie selects two points at a time. In that case, for matching, NORM_HAMMING distance is used. If WTA_K is 3 or 4, which takes 3 or 4 points to produce BRIEF descriptor, then matching distance is defined by NORM_HAMMING2.

Below is a simple code which shows the use of ORB.

```python
import numpy as np
import cv2 as cv
from matplotlib import pyplot as plt

img = cv.imread('simple.jpg', cv.IMREAD_GRAYSCALE)

# Initiate ORB detector
orb = cv.ORB_create()

# find the keypoints with ORB
kp = orb.detect(img,None)

# compute the descriptors with ORB
kp, des = orb.compute(img, kp)

# draw only keypoints location,not size and orientation
img2 = cv.drawKeypoints(img, kp, None, color=(0,255,0), flags=0)
plt.imshow(img2), plt.show()
```

![](Attachments/Pasted%20image%2020260316231410.png)

### Real Time pose estimation of a textured object
Nowadays, augmented reality is one of the top research topic in computer vision and robotics fields. The most elemental problem in augmented reality is the estimation of the camera pose respect of an object in the case of computer vision area to do later some 3D rendering or in the case of robotics obtain an object pose in order to grasp it and do some manipulation. However, this is not a trivial problem to solve due to the fact that the most common issue in image processing is the computational cost of applying a lot of algorithms or mathematical operations for solving a problem which is basic and immediately for humans.

https://docs.opencv.org/3.4/dc/d2c/tutorial_real_time_pose.html  --> Pose estimation example

https://docs.opencv.org/3.4/d6/d55/tutorial_table_of_content_calib3d.html  --> Other related parts

---
## ORB-Slam

Original work: https://github.com/UZ-SLAMLab/ORB_SLAM3
ORB-SLAM3 is the first real-time SLAM library able to perform **Visual, Visual-Inertial and Multi-Map SLAM** with **monocular, stereo and RGB-D** cameras, using **pin-hole and fisheye** lens models. In all sensor configurations, ORB-SLAM3 is as robust as the best systems available in the literature, and significantly more accurate.

---
## **Key Innovations**

ORB-SLAM3 introduces several major advancements over ORB-SLAM2:

### 1 Visual-Inertial Integration

- Uses a **tightly-coupled visual–inertial formulation** based on **Maximum-a-Posteriori (MAP) estimation**
- Enables:
    - **Metric scale recovery** (even for monocular setups)
    - Improved robustness under motion blur or low texture

### **Multi-Map (Atlas) System**

- Introduces an **Atlas framework**: a collection of multiple maps
- Allows:
    - Creation of new maps when tracking fails
    - **Map merging** when revisiting known areas
- Enhances long-term operation and relocalization

### **Improved Place Recognition**

- Uses **bag-of-words (DBoW) place recognition**
- Supports:
    - Loop closure detection
    - Cross-map relocalization
    - Map reuse across sessions

---

## System Architecture**

ORB-SLAM3 follows a **multi-threaded architecture** with three main parallel modules:

### 1 Tracking Thread

- Extracts **ORB (Oriented FAST + Rotated BRIEF)** features
- Estimates current camera pose
- Handles:
    - Frame-to-frame tracking
    - Keyframe insertion
    - Relocalization if tracking is lost

### 2 Local Mapping Thread

- Maintains and optimizes the local map
    
- Tasks include:    
    - Keyframe insertion and culling
    - Map point creation
    - **Local bundle adjustment (BA)** 
- In inertial mode:
    - Estimates IMU biases, velocity, and gravity


### Loop Closing & Map Merging Thread

- Detects loops using place recognition
- Performs:
    - **Loop closure optimization**
    - **Global bundle adjustment (GBA)**
- Enables merging of multiple maps in the Atlas

---

## **Optimization Framework**

ORB-SLAM3 formulates SLAM as a **nonlinear optimization problem**:

- Minimizes a **joint cost function**:
    - Visual reprojection error
    - Inertial measurement residuals
- Uses:
    - **Bundle Adjustment (BA)** for pose and map refinement
    - **Sliding-window optimization** for scalability

---

## **Feature-Based Approach**

- Relies on **ORB features** for:
    - Keypoint detection
    - Descriptor matching
- Advantages:
    - Efficient and rotation-invariant
    - Facilitates loop closure and relocalization
- Limitations:
    - Sensitive to:
        - Motion blur
        - Low-texture environments

---

## **IMU Initialization**

A key contribution is **robust IMU initialization**, which includes:
1. **Visual-only initialization** (structure estimation)
2. **Inertial-only estimation** (scale, gravity, biases)
3. **Joint optimization** for refinement    

This enables accurate **scale estimation and drift reduction**

#NOTE:
This is highly dependent on the Library versions and might not work until and unless you are on the same version of the dependencies as the author. This majorly because the libraries have structured their files differently in each of their release versions. So it better not to manually use the the APT repo to install these dependencies.

## ROS2 Integrations:
https://medium.com/@antonioconsiglio/integrating-orb-slam3-with-ros2-humble-on-raspberry-pi-5-a-step-by-step-guide-78e7b911c361
This is the only tutorial I have found which shows some actual promise but I havn't paid premium and checked yet. Si yep not sure.

The only other decent one I have found is: https://github.com/Mechazo11/ros2_orb_slam3
But the problem with this second one is that ir work and takes input topic od 
`/camera/image`
and gives output of 
`/`
## Key Design Philosophy**

This repo is intentionally **simple and stripped down**:
- ❌ No RViz integration
- ❌ No TF tree publishing
- ❌ No launch files
- ✅ Focus on **core SLAM execution only**

💡 Why this matters:  
Most ROS packages are heavy; this one is meant for **learning + customization**.

## Package Structure

Typical layout:
```
ros2_orb_slam3/  
├── src/                 # ROS2 nodes  
├── include/             # headers  
├── orb_slam3/           # core SLAM library  
├── scripts/             # helper scripts  
├── TEST_DATASET/        # sample EuRoC data  
├── package.xml  
├── CMakeLists.txt
```
👉 Key takeaway:  
It bundles **both SLAM core + ROS2 interface** in one repo.

## **Features Supported**

From what’s implemented:
- Monocular SLAM (primary focus)
- Dataset-based testing (EuRoC sample included)
- Real-time pose estimation

⚠️ Missing / minimal:
- IMU integration (not plug-and-play)
- Visualization tools
- Full ROS2 ecosystem integration

### **Primary Inputs**
- `/camera/image_raw` → `sensor_msgs/Image`
- `/camera/camera_info` → `sensor_msgs/CameraInfo`

### Camera Pose / Odometry
- `/orb_slam3/camera_pose` OR `/odom`  
    → `geometry_msgs/PoseStamped` or `nav_msgs/Odometry`
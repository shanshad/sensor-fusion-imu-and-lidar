This pipeline operates through a staged, sensor-fusion architecture designed to construct a 2D map while tracking the robot’s position without physical wheel encoders.
Upon launch, static transforms establish the fixed physical offsets from the main robot body (base_link) to the LiDAR and IMU sensors (laser_frame and imu_link). 
Once these frame relationships are established, the rf2o laser odometry node analyzes sequential LiDAR range scans from /scan to estimate relative movement, outputting visual odometry on /odom_rf2o. 
The Extended Kalman Filter (ekf_node from robot_localization) then consumes this visual odometry alongside rotational and acceleration data from /imu/data, fusing them into a smooth, filtered estimate published on /odometry/filtered 
while continuously broadcasting the spatial transform between the odom and base_link frames. Finally, slam_toolbox takes this fused odometry transform and raw scan data to build a global grid map on /map, 
continuously correcting for drift by publishing the high-level map $\rightarrow$ odom transform. To prevent frame lookup failures and initialization errors, the launch sequence uses timed delays to ensure
each layer—transforms, odometry, EKF fusion, and map generation—is fully operational before dependent nodes spin up.

Prerequisites & System Requirements

    OS: Ubuntu 24.04 LTS (recommended for ROS 2 Jazzy)

    ROS 2 Distribution: ROS 2 Jazzy Jalisco

    Active Hardware/Topics Required:

        LiDAR publishing on /scan (sensor_msgs/msg/LaserScan)

        IMU publishing on /imu/data (sensor_msgs/msg/Imu)

Required Packages & Dependencies

Install all ROS 2 dependencies required by this pipeline using apt:
Bash

sudo apt update
sudo apt install -y \
  ros-jazzy-tf2-ros \
  ros-jazzy-rf2o-laser-odometry \
  ros-jazzy-robot-localization \
  ros-jazzy-slam-toolbox \
  ros-jazzy-rviz2

Required Package Structure & Configuration Files

This launch script expects to be part of a ROS 2 package named my_robot_slam.

Ensure your workspace directory layout matches the following structure:
Plaintext

my_robot_slam/
├── CMakeLists.txt (or setup.py)
├── package.xml
├── config/
│   ├── ekf.yaml
│   ├── rf2o_params.yaml
│   └── slam_toolbox_params.yaml
└── launch/
    └── slam_pipeline.launch.py

Ensure your hardware or simulation driver nodes are running and streaming data to:

    /scan

    /imu/data

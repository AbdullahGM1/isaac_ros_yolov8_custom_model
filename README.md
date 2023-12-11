# Isaac Ros2 Yolov8 Custom Model 

Introduction
======================

This documentation expalins how to run Yolov8 on Jetson with acceleration for a custom Yolov8 Model. 

Steps:
======================

1- Set up the Developer Environment as shown [here](https://nvidia-isaac-ros.github.io/getting_started/dev_env_setup.html).

2- Clone the following packages inside the `src` folder:

```
git clone https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_common.git && 
git clone https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_object_detection.git
```

3- Run the Docker container using the `run_dev.sh` script:   

```
cd ${ISAAC_ROS_WS}/src/isaac_ros_common && \
  ./scripts/run_dev.sh
```

4- Install this package’s dependencies:

```
sudo apt-get install -y ros-humble-isaac-ros-yolov8
```

5- 

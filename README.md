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

5- Create a folder inside `src` folder, and let's name it `models` to add the custom models inside it. The Yolov8 model has to be in `onnx` formate. You can follow the procedures [here](https://docs.ultralytics.com/modes/export/#key-features-of-export-mode) to export the model from `.pt` to `.onnx`.



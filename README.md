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

5- Create a folder inside `src` folder, and let's name it `models` to add the custom models inside it. The Yolov8 model has to be in `onnx` formate. We can follow the procedures [here](https://docs.ultralytics.com/modes/export/#key-features-of-export-mode) to export the model from `.pt` to `.onnx`.


6- To use `isaac_ros_yolov8` with `Intel RealSense D400 cameras`, we need to install and colone the required packages to run the camera with `Ros2`. We can follow this documentation [here](https://github.com/IntelRealSense/realsense-ros).


7- `isaac_ros_yolov8` system accept only images input of size `640x640` and the input topic name `/image`. So, we need to resize the input image from the `realsence`, and rename the color image input for the camera from `/camera/color/image_raw` to `/image`.


To do so, we can create a lunch file inside `/isaac_ros_yolov8/launch`, to run the the `Realsence` camera and 



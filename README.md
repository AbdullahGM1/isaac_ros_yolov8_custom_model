# Isaac Ros2 Yolov8 Custom Model 

Introduction
======================

This documentation expalins how to run Yolov8 acceleration in Jetson with `Intel Realsense D400` for a custom Yolov8 Model. 

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


7- `isaac_ros_yolov8` system accept only images input of size `640x640` and the input topic name `/image`. So, we need to resize the input image from the `realsence`, and rename the color image input for the camera from `/camera/color/image_raw` to `/image`. To do so, we can create a lunch file inside `/isaac_ros_yolov8/launch`, to run the the `Realsence` camera and rename the input topic and resise the image. We can name the launc file `realsense.launch.py`. We paste the below launch code to the created launch file.

```
import launch
from launch_ros.actions import ComposableNodeContainer, Node
from launch_ros.descriptions import ComposableNode


def generate_launch_description():
    """Launch file which brings up visual slam node configured for RealSense."""
    realsense_camera_node = Node(
        name='camera',
        namespace='camera',
        package='realsense2_camera',
        executable='realsense2_camera_node',
        parameters=[{
                'align_depth.enable': True,
                'enable_infra1': True,
                'enable_infra2': True,
                'enable_color': True,
                'enable_depth': True,
                'depth_module.emitter_enabled': 0,
                'rgb_camera.profile': '640x480x60', 
                'depth_module.profile': '640x480x90',
                'enable_gyro': True,
                'enable_accel': True,
                'gyro_fps': 200,
                'accel_fps': 200,
                'unite_imu_method': 2
        }],
        remappings=[('color/image_raw', '/image')
        ]
    )
    
    image_resize_node_color = ComposableNode(
        package='isaac_ros_image_proc',
        plugin='nvidia::isaac_ros::image_proc::ResizeNode',
        name='image_resize_right',
        parameters=[{
                'output_width': 640,
                'output_height': 640,
                'keep_aspect_ratio': True
        }],
        remappings=[
            ('camera_info', '/camera/color/camera_info'),
            ('image', '	'),
            ('resize/camera_info', '/camera/color/camera_info_resize'),
            ('resize/image', '/image')]
    )
    
    resize_launch_container = ComposableNodeContainer(
        name='resize_launch_container',
        namespace='',
        package='rclcpp_components',
        executable='component_container',
        composable_node_descriptions=[
            image_resize_node_color
        ],
        output='screen'
    )
    
    

    return launch.LaunchDescription([realsense_camera_node, resize_launch_container])
  #  return launch.LaunchDescription([resize_launch_container])
```
We need to make sure that the `enable_depth` is set to `true`, the `isaac_ros_yolov8` does not work smoothly if the camera is running without `Depth`. We can modifiy the profile of the `rgb_camera.profile` and `depth_module.profile` to set with the sutable image size. 

8- Running `Yolov8` acceleration in `Jetson Orin Nx` may causes an issue, and shows `over current` issue, and throttle the hardware pefromance. In this case, we need to to create a custom power mode as shown [here](https://forums.developer.nvidia.com/t/system-throttled-due-to-over-current-on-orin-nx/247300/8?u=aalmusalami).
 
9- Modify `yolov8_decoder_node.cpp` code inside  `/isaac_ros_yolov8/src`. The file contains the information of the model dimensions for the `.onnx` formate. To check the custom model dimensions, we can upload the model to this [website](https://netron.app/). Inside the code, we can see the following part:

```
  int num_classes = 2; 
  int out_dim = 8400;

```

`num_classes` we change the number to the custom models's number of classes, and `int out_dim` and change the number to be the same as shown in custom model dimensions that we get from the website above.

10- `colcon build` the packages.

Running the system:
======================

1- We run first the `yolov8` acceleration"

```
ros2 launch isaac_ros_yolov8 isaac_ros_yolov8_visualize.launch.py model_file_path:=/workspaces/isaac_ros-dev/src/onnx/drone_detection_v22_Yolov8n_int8.onnx engine_file_path:=/workspaces/isaac_ros-dev/src/onnx/drone_detection_v22_Yolov8n_int8.plan input_binding_names:=['images'] output_binding_names:=['output0'] network_image_width:=640 network_image_height:=640 force_engine_update:=False image_mean:=[0.0,0.0,0.0] image_stddev:=[1.0,1.0,1.0] input_image_width:=640 input_image_height:=640 confidence_threshold:=0.60 nms_threshold:=0.45
```
We can change the parameters as shown [here](https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_object_detection/isaac_ros_yolov8/index.html#ros-parameters). A window will show up to show the input image.

* Note 1 - We need to change the name of the custom model.
* Note 2 - Running the code for the first time, it firs will take time to run, wehere it will generate `.plan` formate for the custom model. After the file generates, we can use the same file again without generating `.plan` 

2- Run the Realsense:
```
ros2 launch isaac_ros_yolov8 realsense.launch.py
```
We can see the camera's output with detection shows up in the window. 

Running the system with Rviz2:
======================


1- Modify `isaac_ros_yolov8_visualize.launch.py` launch file in `/isaac_ros_yolov8/launch` by commenting or deleting the following lines"
```
       Node(
            package='rqt_image_view',
            executable='rqt_image_view',
            name='image_view',
            arguments=['/yolov8_processed_image']
        )
```

2- `colcon build` the packages.

3- We run first the `yolov8` acceleration"

```
ros2 launch isaac_ros_yolov8 isaac_ros_yolov8_visualize.launch.py model_file_path:=/workspaces/isaac_ros-dev/src/onnx/drone_detection_v22_Yolov8n_int8.onnx engine_file_path:=/workspaces/isaac_ros-dev/src/onnx/drone_detection_v22_Yolov8n_int8.plan input_binding_names:=['images'] output_binding_names:=['output0'] network_image_width:=640 network_image_height:=640 force_engine_update:=False image_mean:=[0.0,0.0,0.0] image_stddev:=[1.0,1.0,1.0] input_image_width:=640 input_image_height:=640 confidence_threshold:=0.60 nms_threshold:=0.45
```

4- Run the Realsense:
```
ros2 launch isaac_ros_yolov8 realsense.launch.py
```

5- Run `rviz2` and subscribe to `/yolov8_processed_image` to see the detection image.




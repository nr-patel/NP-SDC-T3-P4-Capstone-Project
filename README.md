# Term - 3 - Project - 4 : Capstone Project
Self-Driving Car Engineer Nanodegree Program - Capstone Project

---
[![Udacity Lincoln MKZ driving in simulation using our controllers and light detection](/imgs/output.jpg)](https://youtu.be/rEmeogwxOzw)


This is the Capstone project for the Udacity Self-Driving Car Nanodegree. Developed software to guide a real self-driving car around a test track. Using the Robot Operating System (ROS), created nodes for traffic light detection and classification, trajectory planning, and control.
This Github repo for the final project of the Udacity Self-Driving Car Nanodegree: Programming a Real Self-Driving Car.

## Controllers

PID controller has been used with low pass filtering over current linear velocity and stop accelearation for acceleration/breaking and a Udacity provided YawController for steering. Experimented with the PID controller with low pass filtering for steering as well, but found YawController to perform better.

## Traffic Light Detection and Recognition Models

Developed several traffic light detection and recognition models. Detailed [information about our models](./Model_Info.md).
Developed different models simultaneously to determine the color of the traffic lights. Separate models for the simulation environment and separate models for the real environment - one of our models was trained on both real and simulated images and is uses for detection in both simulated and real environments.

### Components

The following are the main components of the project:

1. Perception: The perception component can be found under tl_detector and is responsible for classifying if the vehicle has a red traffic light or green traffic light ahead of it.

TL Detector components subscribes to the images captured from the camera mounted on the vehicle (/image_color) and whenever the traffic light is within a certain distance, then it starts passing the image to the classifier for classification.

The classifier classifies the image into either traffic light with red lights, green lights or yellow lights.
The check for the traffic light being within a certain distance is done by checking against the YAML file which contains the positions of the traffic lights.
The state of the traffic light is then published to the following topic: /traffic_waypoint to be used by the other components.

2. Planning: The planning component can be found under waypoint_updater and is responsible for creating a list of waypoints with an associated target velocity based on the perception component.

Waypoint Updater component subscribes to the following topics:

1. /traffic_waypoint: The topic that TL detector component publishes on whenever there is a traffic light.
2. /current_velocity: This topic is used to receive the vehicle velocity.
3. /current_pose: The topic used to receive the vehicle position.

Waypoint Updater detects if there is a red traffic light ahead from the /traffic_waypoint in order to trigger deceleration, otherwise it updates the waypoints ahead on the path with velocities equal to the maximum velocities allowed.
This component finally publishes the result to /final_waypoints.

3. Control: The control component can be found under twist_controller and is responsible for the throttle, brake and steering based on the planning component.

DBW Node is the component responsible for the control of the car. DBW node subscribes to the following topics:

1. /current_velocity: This topic is used to receive the vehicle velocity.
2. /twist_cmd: provides proposed linear and angular velocities.
3. /vehicle/dbw_enabled: This topic is used to receive if the manual or autonomous mode is enabled.
4. /current_pose: This topic is used to receive the vehicle position.
5. /final_waypoints: This topic is used to receive the waypoints from the planning component (Waypoint Updater).

DBW Node implements a PID controller that takes into account if manual or autonomous mode is enabled. It takes the input from final_waypoints, current_pose, current_velocity and outputs throttling, steering and brake messages to the following topics /vehicle/throttle_cmd, /vehicle/steering_cmd and /vehicle/brake_cmd.

### Classification

We tried multiple approaches for classification and then we finally settled on one based on the results.

The following are the approaches we tried:

1. VGG16 with ImageNet weights and then finetuning

    We started with VGG16 with ImageNet weights and then we added one extra fully connected layer and finetuned it with both simulator images and site images.
    There were advantages and disadvantages to this approach. This approach gave good predictions with accuracy higher than 90%. However inference speed was very slow.
    This led us to trying out SqueezeNet which has much faster inference speed. Having a fast inference speed is critical here because we want to ensure that this can run in realtime.


2. SqueezeNet that was trained on Nexar and then finetuning

    We started with SqueezeNet that was trained on Nexar dataset and then we further finetuned it with the simulator images.
    SqueezeNet was much more faster than VGG16 and had very high accuracy rate, over 90% and performed much better than the first approach.
    Being originally trained on Nexar dataset as opposed to ImageNet must have contributed to that given that Nexar dataset are mainly traffic lights as opposed to ImageNet.
    However initially this model was not performing well on simulator images, only on site images. After fine tuning the model, the model started to perform much better on simulator images.

3. Tensorflow APIs and then finetuning

    Finally we've used Tensorflow Object Detection API with a fine tuned model (__faster_rcnn_resnet101_coco__) on sim (collected from simulator) and site data (provided in udacity rosbag) separately.

    Video with recognitions on simulator images (new test set that wasn't used during the training):

    Cropping is not used.
	
## Final Project Video

[![Udacity Lincoln MKZ driving in simulation using our controllers and light detection](/imgs/output.jpg)](https://youtu.be/rEmeogwxOzw)


Please use **one** of the two installation options, either native **or** docker installation.

### Native Installation

* Be sure that your workstation is running Ubuntu 16.04 Xenial Xerus or Ubuntu 14.04 Trusty Tahir. [Ubuntu downloads can be found here](https://www.ubuntu.com/download/desktop).
* If using a Virtual Machine to install Ubuntu, use the following configuration as minimum:
  * 2 CPU
  * 2 GB system memory
  * 25 GB of free hard drive space

  The Udacity provided virtual machine has ROS and Dataspeed DBW already installed, so you can skip the next two steps if you are using this.

* Follow these instructions to install ROS
  * [ROS Kinetic](http://wiki.ros.org/kinetic/Installation/Ubuntu) if you have Ubuntu 16.04.
  * [ROS Indigo](http://wiki.ros.org/indigo/Installation/Ubuntu) if you have Ubuntu 14.04.
* [Dataspeed DBW](https://bitbucket.org/DataspeedInc/dbw_mkz_ros)
  * Use this option to install the SDK on a workstation that already has ROS installed: [One Line SDK Install (binary)](https://bitbucket.org/DataspeedInc/dbw_mkz_ros/src/81e63fcc335d7b64139d7482017d6a97b405e250/ROS_SETUP.md?fileviewer=file-view-default)
* Download the [Udacity Simulator](https://github.com/udacity/CarND-Capstone/releases).

### Docker Installation
[Install Docker](https://docs.docker.com/engine/installation/)

Build the docker container
```bash
docker build . -t capstone
```

Run the docker file
```bash
docker run -p 4567:4567 -v $PWD:/capstone -v /tmp/log:/root/.ros/ --rm -it capstone
```

### Port Forwarding
To set up port forwarding, please refer to the [instructions from term 2](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/16cf4a78-4fc7-49e1-8621-3450ca938b77)

### Usage

1. Clone the project repository
```bash
git clone https://github.com/udacity/CarND-Capstone.git
```

2. Install python dependencies
```bash
cd CarND-Capstone
pip install -r requirements.txt
```
3. Make and run styx
```bash
./model_extraction.sh
cd ros
catkin_make
source devel/setup.sh
roslaunch launch/styx.launch
```
4. Run the simulator

### Real world testing
1. Download [training bag](https://s3-us-west-1.amazonaws.com/udacity-selfdrivingcar/traffic_light_bag_file.zip) that was recorded on the Udacity self-driving car.
2. Unzip the file
```bash
unzip traffic_light_bag_file.zip
```
3. Play the bag file
```bash
rosbag play -l traffic_light_bag_file/traffic_light_training.bag
```
4. Launch your project in site mode
```bash
cd CarND-Capstone/ros
source devel/setup.sh
roslaunch launch/site.launch
```
5. Confirm that traffic light detection works on real life images

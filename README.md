# Self Driving Car Nanodegree - Capstone Project: System Integration

## Project objectives

The objective of this project is to implement a ROS-based core for an autonomous vehicle. The vehicle shall be able to complete a closed-circuit test-track, detecting the traffic lights and stopping whenever required. The code will be evaluated in a Unity simulator and a real-world Lincoln MKZ
Specifications
The car should:
Smoothly follow waypoints in the simulator.
Respect the target top speed set for the waypoints.
Stop and restart depending on the state of the traffic lights.
Publish throttle, steering, and brake commands.

We implemented using ROS 3 different modules for the autonomous vehicle:

Perception: Traffic Light Detection Node
Planning: Waypoint Updater Node
Control: DBW Node

### Project video


https://drive.google.com/file/d/1CpIAmb5ED4l56STiP_9a3j5S6NzMbmpz/view?usp=sharing

Team members
* Hugo Trinidad - TL : hugot305@aol.com
* Leandro Borgnino : leo.borgnino@gmail.com
* Mariam Mostafa : mariam.hamdy.mosta@gmail.com
* Stefano Castagnetta : stefanocastagnetta@gmail.com
* Tobias Zwiehoff : T.Zwiehoff@gmail.com





## Overview

This repository contains the code for the final project of the Udacity Self-Driving Car Nanodegree: Programming a real self-driving car which can detect traffic lights and respond to it. 

The code consists of a set of implemented ROS (Robot Operating System) Nodes which interact to maneuver CARLA, Udacitys self driving car platform, autonomously around a test track. In particular the project handles three different subsystems of the vehicle: Perception, Planning and Control:

![](/video/capstone_table.JPG)

### System Architecture
Among the components showed in the picture above, the team focused on the implementation and integration of the following 3 nodes:
Traffic Light Detection: Part of the perception sub-system, the node takes care of detecting traffic lights and classifying their state
Waypoint Updater: Part of the planning sub-system, takes care of generating a trajectory (as a set of waypoints with their respective target velocities) taking into account the detected traffic lights in the environment
Drive by Wire Controller: Part of the control sub-system, takes care of translating the Twist Messages generated by the waypoint follower into throttle, brake and steering values


## Perception
The sensing and perception subsystem’s main objective is to collect data to aid in localization, navigation, spot detection, and obstacle avoidance. In this project the objective is to detect traffic lights, classify its state and take actions according to the result.
The detection and classification was done with a processing model based on SSD architecture (an end to end training and classification for object detection) using transfer learning to customize an existing model for our purpose. Using transfer learning allowed us to save time during the training process.
We used the SSD mobilenet v2 as the base model for transfer learning to our TF Object Detection API. A datasets was extracted from udacity ROSbags and labeled with LabelImg.
The model generation scripts can be found at  https://github.com/leoborgnino/traffic-light-model-generation.

The training process was slow and no more than 4k steps were done during a 12hs training period; in the images below we can see that the loss was still decreasing when the training finished so the model can still be improved.

![](/video/capstone_diagr.png )
![](/video/capstone_diagr1.png )

After training, several tests were made with random images from the rosbag and then in the simulator with site mode. The images from this tests can be seen below:
![](/video/traffic_1.png )
![](/video/traffic_2.png )
![](/video/traffic_3.png )




## Planning

In this subsystem there are 2 nodes: the waypoin_loader and the waypoint_updater. The first one was already given and it loads the static waypoint data and publishes to /base_waypoints. 
The waypoint_updater was developed by us and it subscribes to the /base_waypoint topic, the /obstacle_waypoint topic and the /current_pose topic. With this information, it computes velocities for the waypoints that are in front of the vehicle. The vehicle always tries to keep a speed close to the speed limit unless there is a red light in front of it. In that case it decelerates and stops before the stop line. Once the light turns green the car resumes motion.


## Controls

In this subsystem, there is a DBW(drive-by-wire) node that is responsible for controlling the vehicle. The dbw node subscribes to /current_velocity, /twist_cmd, and /vehicle/dbw_enabled topics. This node will publish throttle, steering, and brake topics. The steering angle is calculated using a YawController while the throttle is calculated using PID. A low pass filter is used to filter out the velocity’s high frequency noise.

![](/video/dbw_node.png  )

DBW was  tested using dbw_test.py, we tested the code against the bag recorded at  https://s3-us-west-1.amazonaws.com/udacity-selfdrivingcar/files/reference.bag.zip
Three csv files are produced which we used to process how DBW node is performing on various commands. The files are found in https://github.com/leoborgnino/CarND-Capstone/tree/master/ros/src/twist_controller

___

For more information about the project, see the project introduction [here](https://classroom.udacity.com/nanodegrees/nd013/parts/6047fe34-d93c-4f50-8336-b70ef10cb4b2/modules/e1a23b06-329a-4684-a717-ad476f0d8dff/lessons/462c933d-9f24-42d3-8bdc-a08a5fc866e4/concepts/5ab4b122-83e6-436d-850f-9f4d26627fd9).

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
roslaunch launch/site.launch
```
5. Confirm that traffic light detection works on real life images

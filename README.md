# ROS Quadrotor Simulator

This project focuses on simulating a quadrotor aerial vehicle for the development of flight control systems, autonomous navigation, obstacle avoidance, and path planning. This software relies on the Robot Operating System (ROS) software. ROS provides hardware abstraction, device drivers, libraries, visualizers, message-passing, package management, and more. Along with ROS, Gazebo is used for simulation. Gazebo offers the ability to accurately and efficiently simulate populations of robots in complex indoor and outdoor environments. 

This project begins with simple manual control of the quadrotor with an Xbox joystick. It then evolves to simulating GPS waypoint navigation. Once the quadrotor can follow a series of waypoints, it is possible to develop 2D and 3D maps of the environment. These maps are used for 2D and 3D path planning and obstacle avoidance.

Detailed instructions and images describing the installation and utilization of this software is available at https://www.wilselby.com/research/ros-integration/

This software package contains several modules, described below:

- action_controller - A simple action controller to translate the trajectory produced from MoveIt! into a velocity command for the quadrotor for 3D navigation
- moveit_simple_controller_manager - A modified version of Moveit_simple_controller_manager to handle a multi degree of freedom trajectory for 3D navigation 
- quad_2dnav - Package to enable 2D map generation and 2D navigation using the Navigation stack	
- quad_3dnav - Package to enable 3D map generation and 2D navigation using MoveIt!
- quad_control - Nodes for the attitude and position controllers as well as a waypoint publisher node 
- quad_description - Files to support the quadrotor model required for Gazebo simulation
- quad_gazebo - Launch files and world files required for Gazebo simulation
- quad_joystick_interface - Package to interface with an Xbox controller to manually control the quadrotor


## Dependencies for ROS Melodic and Gazebo 9.0:

Get

```
$ sudo apt-get -y install python-rosinstall          \
                        ros-melodic-octomap-msgs    \
                        ros-melodic-joy             \
                        ros-melodic-geodesy         \
                        ros-melodic-octomap-ros     \
			            unzip
```


Install mavros

```
$ sudo apt-get -y install ros-melodic-mavros \
			            ros-melodic-mavros-extras
```

### Install the mav comm package

```
$ cd $WORKSPACE/src
$ git clone https://github.com/ElisaTosello/mav_comm.git
```

### Install the glog catkin package

```
$ cd $WORKSPACE/src
$ git clone https://github.com/ethz-asl/glog_catkin.git
```

### Install the catkin simple package

```
$ cd $WORKSPACE/src
$ git clone https://github.com/catkin/catkin_simple.git
```

### Install the ROS Quadrotor Simulator package

```
$ cd $WORKSPACE/src
$ git clone https://github.com/ElisaTosello/ROS_quadrotor_simulator.git
```
 

## Install rotors simulator
RotorS is a UAV gazebo simulator developed by the Autonomous Systems Laboratory at ETH Zurich. It provides some multirotor models such as the AscTec Hummingbird, the AscTec Pelican, or the AscTec Firefly, but the simulator is not limited for the use with these multicopters. There are simulated sensors such as an IMU, a generic odometry sensor, and the VI-Sensor, which can be mounted on the multirotor. This packages also contains some example controllers, basic worlds, a joystick interface, and example launch files. This package will serve as the foundation for the rest of the software development as it is already set-up to interface with the Gazebo simulator.

```
$ cd $WORKSPACE/src
$ git clone https://github.com/ethz-asl/rotors_simulator
```
 
Build the workspace

```
$ cd $WORKSPACE
$ source devel/setup.bash
$ catkin build
```

### Install Xbox 360 Controller 

Install the integrated Ubuntu Xbox driver

```
$ sudo apt-add-repository ppa:rael-gc/ubuntu-xboxdrv
$ sudo apt-get update && sudo apt-get install ubuntu-xboxdrv
```

Install the jstest-gtk package

```
$ sudo apt-get install jstest-gtk
```

The ROS wiki also contains a tutorial (http://wiki.ros.org/joy/Tutorials/ConfiguringALinuxJoystick) for configuring a joystick on Linux. Note that it may be necessary to update the joy node package parameters in the launch file based on the joystick’s assigned input value.

```	
<param name="dev" type="string" value="/dev/input/js0" />
```

### Launch the quadrotor model

The quadrotor model can be visualized in Rviz by running the command

```
$ roslaunch quad_description quad_rviz.launch
```

The quad_world launch file is executed with the following command which displays the quadrotor model in Gazebo.

```
$ roslaunch quad_gazebo quad_world.launch
```

## Manual Quadrotor Control
The quad_joystick_empty_world.launch file creates a simulation with the user providing desired attitude commands via the joystick to the attitude controllers on the quadrotor. A simple simulation of the quadrotor using only attitude controllers can be launched with the following command

```
$ roslaunch quad_gazebo quad_joystick_empty_world.launch
```

The position controller only publishes control commands if the user has entered “GPS mode” by pushing the “B” button by default on the Xbox controller. The “take off” mode, enabled by pushing the “A” button by default, sets the desired waypoint altitude to 1 meters. The “land” mode, enabled by pushing the “X” button by default, sets the desired waypoint altitude to 0 meters without modifying the x, y, or yaw values. A simulation with the quadrotor responding to position commands can be launched with the following command

```
$ roslaunch quad_gazebo quad_joystick_gps_empty_world.launch
```

Lastly, the package has a waypoint mode which allows the quadrotor to follow a pre-defined set of waypoints. The “waypoint mission” mode reads in a waypoint file that contains a series of waypoints and the time for them to be sent. First, the quadrotor must be in “GPS mode.” If enabled by pushing the “Y” button by default, the waypoint node will send each waypoint at the defined time. This allows users to develop and implement autonomous waypoint missions to navigate the quadrotor around the Gazebo world. 

The waypoint files are stored in the quad_control/resource folder and are loaded in the waypoint_publisher_node. Each line in the waypoint file is a waypoint and has the following entries [time (s), x, y, z, yaw (deg)]. The kitchen world series of waypoint is loaded by default and can be tested by running the following command:

```
$ roslaunch quad_gazebo quad_joystick_gps_kinect_kitchen.launch
```

## 2D Mapping and Navigation
Once the quadrotor can reliably and stably navigate the environment based on a series of desired waypoints, the quadrotor system can be used to sense and comprehend it’s surrounding environment. A map is a representation of the environment where the quadrotor is operating. To operate in the map, the quadrotor needs to know its position in the map coordinate frame. Ultimately, the goal is to develop a system that allows the quadrotor to autonomously reach a desired goal state in the map.

A detailed walk through of the mapping process and utilization of Rviz for setting goals is available here: https://www.wilselby.com/research/ros-integration/2d-mapping-navigation/

In order to make an initial map of the environment, a series of waypoints was recorded. These waypoints maneuvered the quadrotor from its initial position outside the office complex, in and around several rooms, and returning back to the initial position. Note that the waypoint_publisher_node needs to be updated to read the wg_waypoints.txt file since this section uses the Willow Garage world by default.

```
$ roslaunch quad_2dnav quad_slam_laser_map.launch
```

Once the environment has been sufficiently mapped, the map can be saved for later use. The map is saved using the map_saver utility using the example code below.

```
$ rosrun map_server map_saver -f mymap
```

After creating the map, it is possible to use it for path planning and navigation. Ensure the correct map file name is being loaded in quad_2dnav.launch. First, the quadrotor must be in “GPS mode” as described above. Once the goal is set in RVIZ and the path created, the quadrotor will follow the path once “Autonomous mode” is enabled by pushing the right bumper by default. This can be tested by running the following command.

```
$ roslaunch quad_2dnav quad_2dnav.launch 
```

## 3D Mapping and Navigation
Ground-based robots are limited to 2D navigation due to their dynamics. However, since the quadrotor can also easily adjust its vertical position, 3D navigation can be implemented. Navigation in 3D enables the quadrotor more maneuverability to explore its environment, the ability to get a much more complete understanding of the environment, and also navigate the environment in more complex paths. This is especially useful when it comes to obstacle avoidance. To demonstrate 3D navigation, the kitchen world was used in Gazebo. 

MoveIt! is state of the art software for mobile manipulation, incorporating the latest advances in motion planning, manipulation, 3D perception, kinematics, control and navigation. The MoveIt Setup Assistant is used to configure the quadrotor with the MoveIt! framework. This GUI generates the SRDF for the quadrotor was well as other necessary configuration files for use with MoveIt!. The MoveIt! Setup Assistant is launched using the following command.

```
$ roslaunch moveit_setup_assistant setup_assistant.launch
```

A detailed walk through of the configuration needed for MoveIt! is available here: https://www.wilselby.com/research/ros-integration/3d-mapping-navigation/

The 3D navigation simulation can be launched with the following command. To enable 3D autonomous navigation, first place the quadrotor into “GPS mode.” After the goal has been set and the path computed, the quadrotor can be set to follow the path by enabling the “3D Navigation mode” using the left bumper button by default.

```
$ roslaunch quad_3dnav quad_3dnav.launch
```

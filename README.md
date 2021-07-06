## Project objectives
The objective of this project is to implement ROS-based core of an autonomous vehicle. The vehicle shall be able to complete a closed-circuit test-track, detecting the traffic lights and stopping whenever required. The code will be evaluated in a Unity simulator and a real-world Lincoln MKZ. More details on the project can be found [here](https://classroom.udacity.com/nanodegrees/nd013/parts/6047fe34-d93c-4f50-8336-b70ef10cb4b2/modules/e1a23b06-329a-4684-a717-ad476f0d8dff/lessons/462c933d-9f24-42d3-8bdc-a08a5fc866e4/concepts/5ab4b122-83e6-436d-850f-9f4d26627fd9).

### Project Implementation

## Waypoint Updater
The waypoint updater is responsible for calculating the next LOOKAHEAD_WPS (200) Waypoints and setting a target velocity for each. The default target velocity at each waypoint is set at system startup and recorded into the base waypoints. This velocity is used as a ceiling for the vehicle velocity and intentionally left unaltered by the waypoint updater. At each call to the waypoint updater, a subset of the base waypoint list is created of length LOOKAHEAD_WPS, starting with the calculated NEXT_WAYPOINT. The velocities of these waypoints are then set to allow smooth and safe deceleration . To aid in this, the current vehicle velocity and maximum acceleration value are used to determine the distance required by the vehicle to safely stop. This value is recorded as the deceleration zone or decel_zone.  

The waypoint updater subscribes to a list of waypoint for the next light. If either of these are populated, they indicate zones the car should not pass through. The traffic light waypoint, if present, is appended to the end of the list of obstacles. Each waypoint in the obstacle list is accompanied by a standoff distance, indicating how far before reaching the obstacle the car should attempt to stop. The standoff value is determined based on the subscription providing the obstacle.  

For every waypoint found to be within the respective deceleration zone plus standoff distance, a stopping gradient is calculated. The velocity at each final waypoint is the lowest value among these stopping gradients and the velocity already set for the waypoint.  

### Specifications
The car should:  

* Smoothly follow waypoints in the simulator.  
* Respect the target top speed set for the waypoints' twist.twist.linear.x in waypoint_loader.py. 
* Stop at traffic lights when needed.
* Stop and restart PID controllers depending on the state of /vehicle/dbw_enabled.
* Publish throttle, steering, and brake commands at 50hz.

## ROS Architecture
The autonomous driving system is composed of perception, planning and control. The modules communicate according to the following ROS structure of nodes and topics : 
![ROS Architecture](ros-architecture.png)

## Control

Per requirements, the control module must publish throttle, steering angle and brake torque at 50 Hz. To accomplish this, an  yaw controller provides the steering angle that matches the target linear and angular speeds, taking into account the current linear speed of the vehicle.

The linear speed of the vehicle is controlled with a classic digital PID controller. To avoid any kind of aliasing, the speed tracking error is filtered with a single pole low-pass filter and then fed to the controller.

The controller signal is limited to the vehicle acceleration and deceleration limits. If the control command signals acceleration,the value is sent to the throttle as is. To avoid braking overuse and excessive jerk, the control is configured to first stop sending throttle signals and start actively braking the car only if the required force exceeds the brake deadband. Due to the asymptotic nature of PID control, we need to force a full stop with the parking torque of 700 N.m whenever the speed of vehicle falls below a threshold.

Since the reference signal is relatively smooth, an automatic tuning process was not needed. The manual tuning started with adjusting the proportional gains and comparing it against the first seconds of the reference implementation. The other two components were adjusted with a manual process of minimizing the root mean squared error between the reference implementation and the output. The final result of this process in shown in [Drive-by-wire testing](#Drive-by-wire-testing)

### Native Installation

* Be sure that your workstation is running Ubuntu 16.04 Xenial Xerus or Ubuntu 14.04 Trusty Tahir. [Ubuntu downloads can be found here](https://www.ubuntu.com/download/desktop).
* If using a Virtual Machine to install Ubuntu, use the following configuration as minimum:
  * 2 CPU
  * 2 GB system memory
  * 25 GB of free hard drive space

  The Udacity team provided a virtual machine that has ROS and Dataspeed DBW already installed, so you can skip the next two steps if you are using this.

* Follow these instructions to install ROS
  * [ROS Kinetic](http://wiki.ros.org/kinetic/Installation/Ubuntu) if you have Ubuntu 16.04.
  * [ROS Indigo](http://wiki.ros.org/indigo/Installation/Ubuntu) if you have Ubuntu 14.04.
* [Dataspeed DBW](https://bitbucket.org/DataspeedInc/dbw_mkz_ros)
  * Use this option to install the SDK on a workstation that already has ROS installed: [One Line SDK Install (binary)](https://bitbucket.org/DataspeedInc/dbw_mkz_ros/src/81e63fcc335d7b64139d7482017d6a97b405e250/ROS_SETUP.md?fileviewer=file-view-default)
* Download the [Udacity Simulator](https://github.com/udacity/CarND-Capstone/releases/tag/v1.2).

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



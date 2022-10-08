# ROSCon 2022 workshop on ros2_control

Thank you for your interest in ros2_control.
This repository was developed with the purpose of supporting the workshop on *ros2_control* @ ROSCon2022 in Kyoto, Japan.
Whether you participated or not, the repository will provide you with detailed instructions on how to use the *ros2_control* framework and explain functionality and purpose of its individual parts.

### Installing this repository

After cloning the repository, go to your source workspace and execute the following commands to import the necessary repositories and to install all dependencies:
```
vcs import --input roscon2022_workshop/roscon2022_workshop.repos .
rosdep update
rosdep install -y -i --from-paths .
```

## What is *ros2_control*

In short, ros2_control is a control framework for ROS2.
But actually, it is much more, it is the kernel of the ROS2 system that controls robots:
 - it abstracts hardware and low-level control for other frameworks such as MoveIt2 and Nav2;
 - it provides resource access management,
 - controls their lifecycle and so on.

For more details check [this presentation](https://control.ros.org/master/doc/resources/resources.html#ros-world-2021)


## Structure of this repository (and workshop)

The structure of the repository follows the flow of integrating robots with ROS2 and these are the following steps:

1. 📑 Setting up the hardware description for *ros2_control*

   1. 🗒 Setting up the robot's URDF using XACRO
   2. 📝 Extending the robot's URDF with the `<ros2_control>` tag

2. 🖥 Using the *Mock Hardware* plugin for easy and generic testing of the setup (and how it can save you ton of time and nerves)

   1. 🛠 How to setup *Mock Hardware* for a robot?
   2. 🔩 How to test it with an off-the-shelf controller?

3. ⚙ Getting to know the roles of the main components of the *ros2_control* framework: *Controller Manager*, *Controllers*, *Resource Manager* and *Hardware Interface*

4. 🔬 Introspection of the *ros2_control* system

5. 💻 Simulating your hardware using Gazebo Classic and Gazebo

6. 🔃 Become familiar with the lifecycle of controllers and hardware. Learn how to use them.

7. 🤖 How to write a hardware interface for a robot

8. 🛂 How to write a controller


## Details about the workshop

### 1. 📑 Setting up the hardware description for *ros2_control*

##### GOAL

  - learn how to setup the URDF of a robot using XACRO macros
  - learn what URDF changes are needed to integrate a robot with `ros2_control`


##### 🗒 Setting up URDF using XACRO for a robot

For any robot that is used with ROS/ROS2 an URDF description is needed.
This description holds information about kinematics, visualization (e.g., for Rviz2) and collision data.
This description is also used by popular ROS2 high-level libraries like, MoveIt2, Nav2 and Simulators.

In this excercise we will focus on setting up the description using the XACRO format which is highly configurable and parameterizable and generally better to use than the static URDF format.

##### Task

Branch: `1-robot-description/task`

Task is to setup the XACRO for RRbot in a package called `controlko_description`.

Kinematics:

  - 2 DoF
  - 1st joint is on a pedestal (box 30x30x30 cm) 30 cm above the ground and rotates around the axis orthogonal to the ground
  - 1st link is 50cm long (cylinder with 20cm diameter)
  - 2nd joint is rotation orthogonal to the first link's height
  - 2nd link is 60cm long  (10x10cm cross-section 5x5cm)

Hardware:

  - Force Torque Sensor at TCP (6D)
  - 2 digital inputs and output (outputs can be measured)

References:

  - https://wiki.ros.org/urdf
  - https://wiki.ros.org/urdf/XML

Files to create or adjust:

  - `rrbot_macro.xacro` - macro with kinematics and geometries for the `rrbot`
  - `rrbot.urdf.xacro` - main xacro file for the robot where macro is instantiated
  - `view_robot.launch.py` - loading and showing robot in `rviz2`


**TIPP**: `RosTeamWS` tool has some scripts that can help you to solve this task faster (on the branch is this already implemented). Resources:

  - [Commonly used robot-package structure](https://stoglrobotics.github.io/ros_team_workspace/master/guidelines/robot_package_structure.html)
  - [Creating a new package](https://stoglrobotics.github.io/ros_team_workspace/master/use-cases/ros_packages/create_package.html)
  - [Setting up robot description](https://stoglrobotics.github.io/ros_team_workspace/master/use-cases/ros_packages/setup_robot_description_package.html)


##### Solution:

Branch: `1-robot-description/solution`

Check the files listed above and execute:
```
ros2 launch controlko_description view_rrbot.launch.py
```
to view the robot and move its joints using the `Joint State Publisher` GUI.


### 2. 🖥 Using *Mock Hardware* plugin for simple and generic testing of the setup

##### GOAL

  - learn what is *Mock Hardware* and how to use it
  - learn how you can fast and easily test you controller's setup and parameters before you deal with simulation or real hardware

*Mock Hardware* is mocking ros2_control `Hardware Interface` based on the robot's description from the `ros2_control` tag.
Its purpose is to simplify and boost the development process when creating a new controller or setting up their configuration.
The advantage of using it, over simulation or real hardware, is a very fast start-up and lean functionality.
It is a well tested module helping you focus on other components in your setup knowing that your "hardware" behaves ideally.
*NOTE:* the functionality of *Mock Hardware* is intentionally limited and it only enables you to reflect commanded values on the state interfaces with the same name. Nevertheless, this is sufficient for most tasks.

**TIPP**:

  - Dr. Denis recommends you to always start to develop things first with the Mock Hardware and then start switching to simulation or real hardware.
    This way you save time dealing with a broken setup with simulation or hardware in the loop.


##### 🛠 How to setup *Mock Hardware* for a robot?

1. Add `hardware` tag under the `ros2_control` tag with plugin `mock_components/GenericSystem` and set `mock_sensor_commands` parameter.
The parameter create fake command interface for sensor values than enables you to simulate the values on the sensor.

2. Create a launch file named `rrbot.launch.py` that starts the ros2_control node with the correct robot description.

**NOTE**: Currently, there is only `GenericSystem` mock component, which can mock also sensor or actuator components (because they just have a reduced feature set compared to a system).

##### 🔩 How to test it with an off-the-shelf controller?

1. Setup the following controllers for the `RRBot`:

- `Joint State Broadcaster` - always needed to get `/joint_states` topic from a hardware.
- `Forward Command Controller` - sending position commands for the joints.
- `Joint Trajectory Controller` - interpolating trajectory between the position commands for the joint.

2. Add to launch file spawning (loading and activating) of controllers.

3. Test forward command controller by sending a reference to it using `ros2 topic pub` command.

4. Create a launch file that starts `ros2_controllers_test_nodes/publisher_joint_trajectory_controller` to publish goals for the JTC.

**TIPP**: `RosTeamWS` tool has some scripts that can help you to solve this task faster. Resources:

  - [Commonly used robot-package structure](https://stoglrobotics.github.io/ros_team_workspace/master/guidelines/robot_package_structure.html)
  - [Creating a new package](https://stoglrobotics.github.io/ros_team_workspace/master/use-cases/ros_packages/create_package.html)
  - [Setting up robot bringup](https://stoglrobotics.github.io/ros_team_workspace/master/use-cases/ros_packages/setup_robot_bringup_package.html)

##### Solution:

Branch: `2-robot-mock-hardware`

Check the files listed above and execute:
```
ros2 launch controlko_bringup rrbot.launch.py
```
then publish a command to the forward command controller:
```
ros2 topic pub /forward_position_controller/commands std_msgs/msg/Float64MultiArray "
layout:
 di.m: []
 data_offset: 0
data:
 - 0.7
 - 0.7"
```

To start `RRBot` with JTC execute:
```
ros2 launch controlko_bringup rrbot.launch.py robot_controller:=joint_trajectory_controller
```
and open a new terminal and execute:
```
ros2 launch controlko_bringup test_joint_trajectory_controller.launch.py
```

**NOTE**: delay between spawning controllers is usually not necessary, but useful when starting a complex setup. Adjust this specifically for the specific use-case.


### 3. ⚙ Getting know the roles of the main components of *ros2_control* framework

Start the previous example one more time and try to answer the following questions:

1. What and where is *Controller Manager*?
2. What are *Controllers*? How they can be seen in ROS2?
3. What is *Resource Manager*? Where can you see it? How to access it?
4. What is *Hardware Interface*? Where is this stored? How to interact with it?

##### Solution:

TODO: Add diagram/figure here.


### 4. 🔬 Introspection of *ros2_control* system

There are two options to interact with the ros2_control, first, using the CLI interface with commands like `ros2 control <command>` (package `ros2controlcli`), and second, using services from the controller manager directly.

Try to figure out how to answer the following questions using those tools:

1. What controllers are loaded in the system?
2. What is the state of the controllers?
3. What hardware interfaces are there and in which state?
4. Which interfaces are available?
5. How can we switch between `forward_position_controller` and `joint_trajectory_controller`?
6. What happens when you try to run all controllers in parallel?
7. What interfaces are controllers using?

Also there are few graphical tools available for `ros2_control`: `rqt_controller_manager` and `rqt_joint_trajectory_controller`. Try to use those tools.

##### Solution:

Answers to the questions:

1. `ros2 control list_controllers`
2. `ros2 control list_controllers`
3. `ros2 service call /controller_manager/list_hardware_components controller_manager_msgs/srv/ListHardwareComponents {}`
4. `ros2 control list_hardware_interfaces`
5. `ros2 run controller_manager spawner forward_position_controller --inactive`
   `ros2 control switch_controllers --deactivate joint_trajectory_controller --activate forward_position_controller`
6. See output in the terminal where `ros2_control_node` is running: `ros2 control switch_controllers --activate joint_trajectory_controller`
7. `ros2 control list_controllers -v`

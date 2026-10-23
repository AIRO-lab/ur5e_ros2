# DH AG95 Gripper ROS2 Driver

**Important:** This is a dependency: https://github.com/ian-chuang/serial-ros2

This repository contains a ROS2 driver for controlling the DH Robotics AG95 Gripper. It builds upon the low-level code from [DH Robotics' ROS1 driver](https://github.com/DH-Robotics/dh_gripper_ros) and integrates it with ros2_control, following the structure of [ros2_robotiq_gripper](https://github.com/PickNikRobotics/ros2_robotiq_gripper).

**Note:** This driver has been tested on Ubuntu 22.04 and ROS2 Humble.

## Installation

1. Source ROS2 Humble:
   ```bash
   source /opt/ros/humble/setup.bash
   ```

2. Clone the repository into your ROS2 workspace:
   ```bash
   cd /path/to/your_ros2_ws/src
   git clone -b humble https://github.com/ian-chuang/dh_ag95_gripper_ros2.git
   ```

3. Install dependencies using rosdep:
   ```bash
   rosdep install -i --from-path . --rosdistro humble -y
   ```

4. Build the package:
   ```bash
   colcon build
   ```

5. Source the setup files:
   ```bash
   source /path/to/your_ros2_ws/install/local_setup.bash
   ```

## Configuration

1. Find the port that the gripper is currently binding to, e.g., `ttyUSB0`.
2. Run the following command to obtain the serial number:
   ```bash
   udevadm info --name=/dev/ttyUSB0 --attribute-walk | grep serial
   ```
   Use the first serial number that shows up; the format should look similar to `FT6S4DSP`.

3. Edit the udev rules file:
   ```bash
   sudo nano /etc/udev/rules.d/99-dh-ag95-gripper.rules
   ```
   Add the following line, replacing `<serial number here>` with your sensor's serial number:
   ```bash
   SUBSYSTEM=="tty", ATTRS{serial}=="<serial number here>", ENV{ID_MM_DEVICE_IGNORE}="1", ATTR{device/latency_timer}="1", SYMLINK+="robot/dh_ag95_gripper"
   ```

4. Apply the changes:
   ```bash
   sudo udevadm control --reload && sudo udevadm trigger
   ```

## Usage

### Local integration change: run alongside the UR arm

This checkout defaults to the `gripper` ROS namespace. The controller manager,
spawners, joint states, robot description, and standalone TF tree are separated
from the UR arm. Stop the old launches before restarting with these changes.
This provides independent control; it does not attach the gripper model to the
UR MoveIt planning scene.

Launch the driver with the following command:
```bash
ros2 launch dh_gripper_driver dh_ag95_control.launch.py namespace:=gripper com_port:=/dev/robot/dh_ag95_gripper gui:=false
```

The launch activates the real gripper and can move its fingers during initialization.
To check each manager separately:
```bash
ros2 control list_controllers -c /controller_manager
ros2 control list_controllers -c /gripper/controller_manager
```

For isolated software testing use `use_mock_hardware:=true`. The namespaced
launch was checked with two simultaneous mock managers; real concurrent arm
and gripper operation still requires validation after restarting. The original
root namespace remains available via `namespace:=/` for standalone use only.

To command the gripper to close, open another terminal and run this command:
```bash
ros2 topic pub --once /gripper/forward_position_controller/commands std_msgs/msg/Float64MultiArray "{data: [0.93]}"
```

Similarly, to open the gripper, use:
```bash
ros2 topic pub --once /gripper/forward_position_controller/commands std_msgs/msg/Float64MultiArray "{data: [0]}"
```

To view the URDF, use:
```bash
ros2 launch dh_ag95_description view_dh_ag95.launch.py
```

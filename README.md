# UR Robotiq Integrated Driver

This repository provides Python 3 packages designed for [ROS 2
Humble](https://index.ros.org/doc/ros2/Releases/Release-Humble-Hawksbill/) to control the [Robotiq 2F
gripper](https://robotiq.com/products/2f85-140-adaptive-robot-gripper) attached to Universal Robot (UR) robot arms using
the toolport available on the robot arms' wrist.

## System Requirements

> :warning: **Only e-Series robots are capable of using the tool port!**
> 
You must complete the following steps before using this repository:

1. Remove any Robotiq related URCap present on the robot controller via the teach pendant.
2. Complete the robot side setup as instructed by [`ur_robot_driver`
   documentation](https://docs.ros.org/en/ros2_packages/humble/api/ur_robot_driver/setup_tool_communication.html).

## Local Installation

### 1. Install System Dependencies

```bash
sudo apt-get update && sudo apt-get install -y \
    git \
    curl \
    wget \
    build-essential \
    cmake \
    python3-dev \
    python3-pip \
    python-is-python3
```

### 2. Install ROS 2 Dependencies

```bash
sudo apt-get install -y \
    ros-humble-ur \
    ros-humble-ros2-control \
    ros-humble-ros2-controllers \
    ros-humble-xacro \
    ros-humble-joint-state-publisher-gui \
    ros-humble-robot-state-publisher \
    ros-humble-rviz2
```

### 3. Install Python Dependencies

The driver uses `pymodbus` for Modbus RTU communication:

```bash
pip install pymodbus numpy
```

Or install via apt:
```bash
sudo apt-get install -y python3-pymodbus python3-numpy
```

> Note: This driver has been updated to work with both pymodbus 2.x and 3.x.

### 4. Build the ROS 2 Workspace

```bash
cd /path/to/2f-Hardware-Driver/colcon_ws

# Source ROS 2
source /opt/ros/humble/setup.bash

# Install any remaining rosdep dependencies
rosdep update
rosdep install --from-paths src --ignore-src -r -y

# Build
colcon build --symlink-install

# Source the workspace
source install/setup.bash
```

## Quick Start

1. Set the robot parameters

```bash
export UR_IP=       # robot IP address e.g., 192.168.1.199
export UR_HOSTNAME= # robot network hostname e.g., ur-20224536182
export UR_TYPE=     # ur robot type, one of [ur3, ur3e, ur5, ur5e, ur10, ur10e, ur16e, ur20, ur30]
export REVERSE_IP=  # host (your PC) IP address e.g., 192.168.1.99
```

2. Start `ur_robot_driver` with tool communication enabled

```bash
ros2 launch ur_robot_driver ur_control.launch.py \
    ur_type:=$UR_TYPE \
    robot_ip:=$UR_IP \
    reverse_ip:=$REVERSE_IP \
    use_tool_communication:=true
```

`UR_HOSTNAME` may be used instead of `UR_IP` if your network resolves it.

3. Start `rvl_robotiq_driver` controller node

In a ***new terminal***:

```bash
source /opt/ros/humble/setup.bash
source /path/to/2f-Hardware-Driver/colcon_ws/install/setup.bash
ros2 run rvl_robotiq_driver robotiq_controller
```

> :spiral_notepad: NOTE: Verify that the gripper is powered and the gripper status LED is on and is solid blue. This means that
> the driver has successfully established communication with the gripper. If the LED is not on or is red, please
> see the [Troubleshooting](#troubleshooting) section.

4. Activate the gripper via provided service

In a ***new terminal***:

```bash
source /opt/ros/humble/setup.bash
source /path/to/2f-Hardware-Driver/colcon_ws/install/setup.bash
ros2 service call /robotiq/activate std_srvs/srv/Trigger
```

At this point, you should see the gripper cycle through its activation sequence by fully closing then fully opening its
fingers. The gripper is now ready to receive commands.

## Provided Services and Topics

### Topics

```
/robotiq/joint_states
/robotiq/status
```

### Services

```
/robotiq/activate
/robotiq/auto_close/fragile
/robotiq/auto_close/medium
/robotiq/auto_close/soft
/robotiq/auto_close/strong
/robotiq/auto_open/fragile
/robotiq/auto_open/medium
/robotiq/auto_open/soft
/robotiq/auto_open/strong
/robotiq/reactivate
/robotiq/request_status
/robotiq/set_opening
/robotiq/set_position
```

You can use `ros2 topic info` and `ros2 service info` to get more information about each topic and service.

## Citation

If you find this code useful, then please consider citing our work.

You can use the "Cite this repository" feature under About section for automatic generation of APA and BibTex
references.

![Cite this repository](images/cite.png)

A raw BibTex entry is also provided below:

```bibtex
@software{tram2023ur,
  author={Tram, Minh},
  title={{UR-Robotiq Integrated Driver}},
  url={https://github.com/robotic-vision-lab/UR-Robotiq-Integrated-Driver},
  version={1.0.0},
  year={2023}
}
```

## Troubleshooting

### 1. Gripper is not powered (No status LED)

- Check physical connection between the gripper and the toolport on the robot arm.
- Check that the robot is not in E-Stop or Power Off state.

### 2. Status LED is RED

- Make sure that the RS-485 URCap is installed and enabled on the robot controller.
- Make sure that the `ur_robot_driver` launch command has `use_tool_communication:=true` argument and correct robot IP
  address.
- Make sure that the gripper controller node is running and is not reporting any errors.

### 3. Status LED is BLUE but gripper is not responding to commands

- Restart the gripper controller node.

### 4. Permission denied on /tmp/ttyUR

The gripper communicates via `/tmp/ttyUR` which is created by the UR driver when `use_tool_communication:=true`. Make sure:
- The UR driver is running with tool communication enabled
- You have proper permissions (try running with sudo if needed, or add user to dialout group)

```bash
sudo usermod -a -G dialout $USER
# Log out and back in for changes to take effect
```

## References

- [ROS Industrial Robotiq](https://github.com/ros-industrial/robotiq)
- [Universal Robot ROS 2 driver](https://github.com/UniversalRobots/Universal_Robots_ROS2_Driver)

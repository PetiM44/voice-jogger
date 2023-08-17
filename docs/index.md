# Voice Jogger
This documentation page contains information about the original Voice Jogger repo and its updates by Péter Telkes during Summer 2023.

# Server Requirements

This project is designed to work with Ubuntu 20.04 LTS with ROS noetic. It may work with earlier different versions with some tweaks. Following are the main requirements:

| Name             | Version   |
|------------------|-----------|
| Ubuntu           | 20.04 LTS |
| ROS              | Noetic    |
| vosk             | 0.3.42    |
| onnxruntime      | 1.11.1    |
| word2number      | 1.1       |
| rospy            | 1.15.11   |
| numpy            | 1.22.3    |
| actionlib        | 1.13.2    |
| moveit-commander | 1.1.9     |
| franka_gripper   |           |

For a complete list of pypi dependencies, see pip_pkgs.txts

Some dependencies may not be available with PYPI and you may need to get them installed via ROS Workspace. Following are the list of main packages installed in the catkin workspace:

- franka_ros_interface
- geometric_shapes
- moveit
- moveit_msgs
- moveit_resources
- moveit_tutorials
- moveit_visual_tools
- panda_moveit_config
- rviz_visual_tools
- srdfdom

For a complete list of catkin workspace packages, see catkin_pkgs.txt file. You may not need to install all of them if the main packages are available system-wide in /opt/ros/.../lib

For working with real franka panda robot, you will also need to install libfranka. (this should already be installed in Robo lab).

In case some packages are missing and are crucial to install and run this project, please reach out to the authors of the original repository.


# Usage and Instruction

The code is divided into several parts.

- Android Application: There exists an android app that was built on AndroidStudio. The source code is in android directory. Android app reads audio from mobile microphone and sends those raw datagram to UDP sockets in the server.

- Server/main: Server main is responsible for receiving audio signal from mobile app, converting it to text commands, and then publishing those text commands into a ros node topic text_command_transmitter/text_commands. The file to run a server is server/main.py. There are other server implementation in server/microphone_input.py and server/main_modified.py. microphone_input contains experimental commands buffer implementation and it is used to get audio signal from microphone connected directly with computer. This is going to require an additional package sounddevice from pypi. There is a third script called ros_message_transmitter_for_testing_robotMover which can be used for debugging and testing purposes. This will send commands directly to robotmover without the audio signal and speech recognition logic via cli inputs.

- Server/robotmover: Robot mover is responsible for reading text commands from main and sending commands to ROS topics that control robot manipulation. The file is located at server/robotMover.py

## Running the code

For running the code in simulation:-

In 1st terminal: 

For simulation:

    roslaunch panda_moveit_config demo.launch

For real robot:

    roslaunch panda_moveit_config franka_control.launch robot_ip:=130.230.36.115 load_gripper:=True

In 2nd terminal:

    python3 main.py 
OR

    python3 microphone_input.py 
    
OR

    python3 ros_message_transmitter_for_testing_robotMover.py

In 3rd terminal:

    python3 robotMover.py


## Support Commands and Modes

The following list contains the supported commands.

- START PANDA: Once ran, all valid commands are going to be published to ROS topic
- STOP PANDA: Once ran, commands may get recognized but they wont get published
- MODE [STEP, DISTANCE]: Select either step or distance mode
- STEP SIZE [LOW, MID, HIGH]: Select step size. Used in step mode with move commands and rotate command.
- SAVE POSITION [1,2,3, etc.]: Save current X,Y,Z, RX, RY, RZ position of the  robot tool in the positions.txt file
- POSITION [1,2,3, etc.]: Go to saved position
- RECORD [TASK1, TASK2, etc.]: Start recording task in tasks.txt file
- FINISH: Stop recording task
- PLAY/DO/TASK [TASK1, TASK2, etc]: Play recorded task
- REPEAT [# of times] TIMES [TASKNAME]: Plays TASKNAME # times.
- JOG [direction] [# of times] TIMES [TASKNAME]: Plays TASKNAME # times, but offsets the starting position in [direction] by the step size each time the task is repeated.
- HOME: Go to the home position of the robot
- REMOVE POSITION [1,2,3, etc.]: Remove position from file
- TOOL OPEN: Open tool
- TOOL CLOSE: Close tool
- TOOL ROTATE: Rotate tool by the step size
- TOOL ROTATE BACK: Rotate tool in the opposite direction

- PICK [position name]: Move robot above the given position, open the gripper, move down to the position, close the gripper and move back up
- PLACE [position name]: Move robot above the given position, move down to the position, open the gripper and move back up
- OFFSET [position name] [direction] [distance]: Similar to PLACE, but the target position for placing the object is offset to the given direction by the given distance. Direction can be left/right/forward/backward. Can be used to place objects near a saved position.
- STACK [position name] DISTANCE [distance]: Similar to PLACE, but the target position for placing the object is above the given position by the distance specified as [distance]
- HOLD [position name] DISTANCE [distance]: Similar to STACK, but it does not actually place the object at the position, but holds it at a distance above. Basically it moves the robot to a position [distance] above [position name].


In step mode:

 - MOVE [UP, DOWN, LEFT, RIGHT, FRONT/FORWARD, BACK/BACKWARD]: Move robot in given direction. In step mode, distance is not provided and the step size is used as distance.

In distance mode:

- MOVE [UP, DOWN, LEFT, RIGHT, FRONT/FORWARD, BACK/BACKWARD] [DISTANCE]: Similar to step except distance is also provided in millimeters.


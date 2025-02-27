# voice-jogger
Voice Jogger for Kone533-2021-2022-1 Robotics Project Work

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

Feel free to provide feedback if some packages are missing and were crucial to install to run this project.


# Usage and Instruction

The code is divided into several parts.

- Android Application: We have a android app that was built on AndroidStudio. The source code is in android directory. Android app reads audio from mobile microphone and sends those raw datagram to UDP sockets in the server.

- Server/main: Server main is responsible for receiving audio signal from mobile app, converting it to text commands, and then publishing those text commands into a ros node topic text_command_transmitter/text_commands. The file to run a server is server/main.py. There are other server implementation in server/microphone_input.py and server/main_modified.py but those can be ignored for now. microphone_input contains experimental commands buffer implementation and it is used to get audio signal from microphone connected directly with computer. This is going to require an additional package sounddevice from pypi. There is a third script called ros_message_transmitter_for_testing_robotMover which can be used for debugging and testing purposes. This will send commands directly to robotmover without the audio signal and speech recognition logic via cli inputs.

- Server/robotomver: Robot mover is responsible for reading text commands from main and sending commands to ROS topics that controls robot manipulation. The file is located at server/robotMover.py

## Running the code

For running the code in simulation:-

**In 1st terminal:** 

For simulation:

    roslaunch panda_moveit_config demo.launch

For real robot (make sure to insert the actual IP address of the robot):

    roslaunch panda_moveit_config franka_control.launch robot_ip:=130.230.36.115 load_gripper:=True

**In second terminal:**

For using mobile app as voice input:

    python3 main.py 
OR

For using the microphone of the computer as voice input:

    python3 microphone_input.py 
    
OR

For manual typing of the commands:

    python3 ros_message_transmitter_for_testing_robotMover.py

**In third terminal:**

    python3 robotMover.py


## Supported Commands and Modes

Following are the most commonly used supported commands. More commands can be found from server/commandCreator.py but the command usage isn't written here.

- START PANDA: Once ran, all valid commands are going to be published to ROS topic
- STOP PANDA: Once ran, commands may get recognized but they wont get published
- MODE [STEP, DISTANCE]: Select either step or distance mode
- STEP SIZE [LOW, MID, HIGH]: Select step size. Used in step mode move commands and rotate command
- SAVE POSITION [1,2,3, etc.]: Save current X,Y,Z, RX, RY, RZ position of the  robot tool in the positions.txt file
- POSITION [1,2,3, etc.]: Go to saved position
- RECORD [TASK1, TASK2, etc.]: Start recording task in tasks.txt file
- FINISH: Stop recording task
- PLAY/DO/TASK [TASK1, TASK2, etc]: Play recorded task
- REPEAT [# of times] TIMES [TASKNAME]: Plays TASKNAME # times.
    - Example: "REPEAT 3 TIMES PAINT" (assuming that a task named "PAINT" was previously recorded)
- JOG [direction] [# of times] TIMES [TASKNAME]: Plays TASKNAME # times, but offsets the starting position in [direction] by the step size each time the task is repeated. [direction] can be LEFT/RIGHT/FORWARD/BACKWARD/UP/DOWN.
    - Example: "JOG LEFT 3 TIMES PAINT" (assuming that a task named "PAINT" was previously recorded)
- AGAIN: repeat the last executed command. Does not work well with other types of repetition (RECORD, REPEAT, JOG, ...)
    - Example:  
    "FORWARD 100"  
    "AGAIN" -> will get executed as "FORWARD 100"
- HOME: Go to home position of the robot
- REMOVE POSITION [1,2,3, etc.]: Remove position from file
- TOOL OPEN: Open tool
- TOOL CLOSE: Close tool
- TOOL ROTATE: Rotate tool by the step size
- TOOL ROTATE BACK/OPPOSITE: Rotate tool in the opposite direction
- TOOL [DISTANCE]: Move tool fingers by given distance (currently not available)

- PICK [position name]: Move robot above the given position, open the gripper, move down to the position, close the gripper and move back up
    - Example: "PICK PART" (assuming that a position named "PART" was previously recorded)
- PLACE [position name]: Move robot above the given position, move down to the position, open the gripper and move back up
    - Example: "PLACE PART" (assuming that a position named "PART" was previously recorded)
- OFFSET [position name] [direction] [distance]: Similar to PLACE, but the target position for placing the object is offset to the given direction by the given distance. Direction can be left/right/forward/backward. Can be used to place objects near a saved position.
    - Example: "OFFSET PART LEFT 100" will place the gripped object at a position 100 mm left to "PART" (assuming that a position named "PART" was previously recorded)
- STACK [position name] DISTANCE [distance]: Similar to PLACE, but the target position for placing the object is above the given position by the distance specified as [distance]
    - Example: "STACK PART DISTANCE 100" will place the gripped object 100 mm above "PART" (assuming that a position named "PART" was previously recorded)
- HOLD [position name] DISTANCE [distance]: Similar to STACK, but it does not actually place the object at the position, but holds it at a distance above. Basically it moves the robot to a position [distance] above [position name].
    - Example: "HOLD PART DISTANCE 100" will move the end effector 100 mm above "PART" (assuming that a position named "PART" was previously recorded)

- CIRCLE [direction] [radius]: Moves the end-effector along a circle in the horizontal plane. The circle starts in the FORWARD direction, and its center point is [radius] distance in [direction] direction from the starting point. This command is currently very basic, and most likely will not be developed further due to limitations of voice commanding. [direction] can be either LEFT or RIGHT.
    - Example: "CIRCLE LEFT 100" will move the end effector on a circular path with a radius of 100 mm, starting forward and with the circle being to the left of the starting position.


In step:

 - MOVE [UP, DOWN, LEFT, RIGHT, FRONT/FORWARD, BACK/BACKWARD]: Move robot in given direction. In step mode, distance is not provided and the step size is used as distance.

In distance:

- MOVE [UP, DOWN, LEFT, RIGHT, FRONT/FORWARD, BACK/BACKWARD] [DISTANCE]: Similar to step except distance is also provided in millimeters


# Autonomous Tic-Tac-Toe

A fully autonomous Tic-Tac-Toe system built on ROS2, combining computer vision and game logic to detect and play the game in real time against a human opponent. The robot perceives the board state through a camera, computes the optimal move, and executes it physically via an Arduino-controlled actuator.

---

## Overview

This project is split into two primary subsystems that communicate through ROS2 topics and services:

- **Vision** — handles all camera input and board state recognition using OpenCV
- **Control** — handles game logic, move computation, and hardware actuation

The system runs on a ROS2 workspace with packages written in a combination of Python and C++. The physical actuation layer runs on an Arduino, which receives move commands over serial from the ROS2 environment.

---

## System Architecture

```
Camera Input
     |
     v
[Vision Node]  -----(board state topic)----->  [Control Node]
  OpenCV                                          Game Logic
  Python                                          Minimax / Move Selector
                                                       |
                                              [Arduino Interface]
                                                  C++ Serial
                                                       |
                                                  Physical Move
```

---

## Repository Structure

```
Autonomous-Tic-Tac-Toe/
├── src/
│   ├── tictactoe_vision/        # ROS2 package — computer vision subsystem
│   │   ├── tictactoe_vision/    # Python nodes for board detection
│   │   ├── CMakeLists.txt
│   │   └── package.xml
│   │
│   └── tictactoe_control/       # ROS2 package — game logic and actuation
│       ├── src/                 # C++ nodes for move planning and serial comm
│       ├── CMakeLists.txt
│       └── package.xml
│
├── Arduino_Tictactoe/           # Arduino sketch for physical move execution
│   └── Arduino_Tictactoe.ino
│
└── README.md
```

---

## Subsystems

### Vision (`tictactoe_vision`)

Written in **Python** using **OpenCV**.

Responsibilities:
- Capture live video feed from a mounted camera
- Detect and localize the Tic-Tac-Toe grid in the frame
- Identify the occupancy state of each of the 9 cells (X, O, or empty)
- Publish the detected board state as a ROS2 message

Techniques used:
- Contour detection and perspective transform for board localization
- Color or shape-based classification to distinguish X and O markers
- CvBridge for converting between ROS2 `sensor_msgs/Image` and OpenCV frames

### Control (`tictactoe_control`)

Written in **C++**.

Responsibilities:
- Subscribe to board state updates from the vision node
- Determine the optimal next move using a game-solving algorithm (Minimax)
- Publish the selected move
- Communicate with the Arduino over serial to trigger physical actuation

### Arduino (`Arduino_Tictactoe`)

Written in **C (Arduino framework)**.

Responsibilities:
- Receive move coordinates from the ROS2 control node via serial
- Drive servo motors or other actuators to place a piece on the board

---

## Dependencies

| Dependency | Version | Purpose |
|---|---|---|
| ROS2 | Humble / Iron | Middleware and communication |
| OpenCV | 4.x | Image processing and board detection |
| cv_bridge | — | ROS2-OpenCV image conversion |
| Python | 3.10+ | Vision node implementation |
| GCC / CMake | — | C++ node build toolchain |
| Arduino IDE | 1.8+ / 2.x | Arduino firmware upload |

---

## Setup and Installation

### 1. Clone the repository

```bash
git clone https://github.com/gustigratia/Autonomous-Tic-Tac-Toe.git
cd Autonomous-Tic-Tac-Toe
```

### 2. Install ROS2 dependencies

```bash
rosdep install --from-paths src --ignore-src -r -y
```

### 3. Build the workspace

```bash
colcon build
source install/setup.bash
```

### 4. Upload Arduino firmware

Open `Arduino_Tictactoe/Arduino_Tictactoe.ino` in the Arduino IDE, select the correct board and port, then upload.

---

## Running the System

Launch the vision node:

```bash
ros2 run tictactoe_vision vision_node
```

Launch the control node:

```bash
ros2 run tictactoe_control control_node
```

Or launch both together if a launch file is provided:

```bash
ros2 launch tictactoe_control tictactoe.launch.py
```

---

## ROS2 Topics

| Topic | Type | Direction | Description |
|---|---|---|---|
| `/camera/image_raw` | `sensor_msgs/Image` | Input | Raw camera feed |
| `/tictactoe/board_state` | Custom msg | Vision → Control | Current state of all 9 cells |
| `/tictactoe/next_move` | Custom msg | Control → Arduino interface | Cell index of the chosen move |

---

## Hardware Requirements

- Camera (USB webcam or CSI camera)
- Arduino Uno / Mega (or compatible board)
- Servo motors or linear actuators for piece placement
- A flat, well-lit playing surface with a printed or drawn Tic-Tac-Toe grid

---

## Author

Gusti Gratia Delpiera  
NRP: 5026231097

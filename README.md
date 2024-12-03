# Sensor Controller System

## Overview

This project is designed to control and manage data from multiple sensors used in a sensor platform, including SBL (Short Baseline Location), Depth, and Oil sensors. It processes sensor data, calculates location coordinates based on sensor inputs, and sends data to a host PC for further use.

The system uses FreeRTOS for task management and inter-process communication, specifically queues, to handle sensor data, process it, and send it to the host PC. This README explains the system structure and key components of the Sensor Controller system.

## Key Components

### 1. **Sensor States**
The system operates in three main states:
- **STARTSTATE**: The initial state where the system waits for commands and sensor acknowledgment.
- **ENABLESTATE**: Once sensors are acknowledged, they are enabled and ready to send data.
- **PARSESTATE**: The final state where data is parsed, and location information is calculated.

### 2. **FreeRTOS Task Management**
- The system uses FreeRTOS tasks to manage sensor data, control flow, and perform calculations asynchronously.
- Each task is responsible for specific parts of the system, such as receiving sensor data, processing it, and sending it to the host PC.

### 3. **Communication Queues**
Queues are used for inter-task communication:
- `Queue_Sensor_Data`: Receives data from the sensor platform.
- `Queue_HostPC_Data`: Sends data to the host PC.
- `Queue_Process_Data`: Carries raw sensor data to the processing task.

### 4. **Sensor Acknowledgments**
The system waits for acknowledgments from all required sensors (SBL1, SBL2, SBL3, Depth, and Oil). Once all acknowledgments are received, the system transitions to the **PARSESTATE**.

### 5. **Location Calculation**
- Based on the distances provided by the SBL sensors and the depth data, the system calculates the location of the sensor platform.
- The `calculateLocation` function uses predefined sensor positions and calculates distances based on speed and depth data.

### 6. **Data Compression and Decompression**
- The system compresses sensor data into a 16-bit value, where the first two bits represent the sensor type, and the remaining 14 bits represent the sensor data.
- Compressed data is transmitted, and for testing purposes, the system also provides decompression for verifying the received data.

### 7. **Processing and Sending Data**
- The `ProcessSendDataTask` task is responsible for processing the raw sensor data, calculating the location (if necessary), and sending the processed data to the host PC.
- Each sensor type is handled differently, and the processed data is printed using `print_str`.

### 8. **Sensor Reset and Re-initialization**
- The system has the ability to reset the sensors and clear their acknowledgment flags. This ensures that new sensor data can be processed without interference from previous sensor interactions.

### 9. **Timer Management**
- A FreeRTOS timer (`xTimer[0]`) is used to manage delays between acknowledgment checks. The system stops waiting once all sensor acknowledgments are received.

## Dependencies

- FreeRTOS: For managing tasks, queues, semaphores, and timers.
- `print_str`: A utility function for outputting data (emulates serial communication to the host PC).
  
  # Remote Sensing Platform

## Overview

The `remoteSensingPlatform.c` file contains the code for managing sensor communication, handling timers for various sensors, and processing sensor data for a remote sensing platform. This platform interacts with multiple sensors, including SBL (Sensor-Based Location) sensors, Depth sensors, and Oil sensors. It leverages FreeRTOS to manage sensor tasks and communication between the sensors and a central controller.

This system is responsible for managing messages from the datalink, starting timers for each sensor, and enabling/disabling sensors based on received commands.

## Key Components

### 1. **Sensor Management**
The platform handles three types of sensors:
- **SBL (Sensor-Based Location) Sensor**
- **Depth Sensor**
- **Oil Sensor**

Each sensor is controlled through timers that are started and stopped based on communication from the central controller. The timers manage the periodic reading of each sensor.

### 3. **Message Parsing and Handling**
The system processes incoming messages through the `parse_sensor_message` function. The messages are parsed to determine the sensor ID and message ID:
- **Controller Messages**: Handle system reset and other control commands.
- **SBL Sensor Messages**: Enable or modify the behavior of the SBL sensors.
- **Depth Sensor Messages**: Enable or modify the behavior of the Depth sensor.
- **Oil Sensor Messages**: Enable or modify the behavior of the Oil sensor.

Each message is checked for validity, including checksum verification, and the appropriate actions are taken based on the message's contents.

### 4. **Sensor Enable and Configuration**
When an enable command (message ID 0) is received for any sensor, the corresponding timer is adjusted to the new period (received in the message) and started. The sensor acknowledgment message is sent back to confirm the action:
- **SBL Sensor Enable**: Sends acknowledgment for enabling the SBL sensor.
- **Depth Sensor Enable**: Sends acknowledgment for enabling the Depth sensor.
- **Oil Sensor Enable**: Sends acknowledgment for enabling the Oil sensor.

If a reset command (message ID 0 from the Controller) is received, the timers for all sensors are stopped, and the system enters a reset state.

## Task Flow

1. **Timer Creation**: The timers for each sensor are created with specified periods and start functions.
2. **Message Reception**: The `SensorPlatformTask` listens for incoming messages and processes them.
3. **Sensor Control**: Upon receiving valid messages, the system controls the timers for the sensors and sends acknowledgment messages as appropriate.
4. **Sensor Reading**: Once timers are started, the sensors are periodically read based on the timer intervals.
5. **Reset**: If a reset message is received, all timers are stopped, and the system is reset.

# Sensor System Overview

This system manages three types of sensors — **Short Baseline Location (SBL) Sensor**, **Oil Sensor**, and **Depth Sensor**. Each sensor tracks specific data, updates periodically, and communicates real-time information about the robot’s position, oil levels, and depth. Below is a breakdown of how each sensor operates and how they work together.

---

## General Operation of Sensors

- **SBL Sensor**: Tracks the robot's location relative to three reference stations by simulating signal travel times. The robot's position is estimated based on changes in signal time as it moves.
- **Oil Sensor**: Simulates oil concentration levels, with states that include oil increase, decrease, and steady levels. It periodically updates oil levels and detects anomalies like high oil levels.
- **Depth Sensor**: Monitors the robot’s depth by simulating movement up and down. It adjusts the depth value periodically based on the robot’s movement direction and speed.

---

## SBLSensor

### Overview

The **SBLSensor.c** file simulates the robot's position based on signal travel times between the robot and three reference stations. These values are updated periodically to track the robot's movement and provide real-time location updates.

### Key Functionality

- **Signal Time Simulation**: Simulates fluctuating signal times between the robot and three stations, adjusting based on movement direction.
- **Movement Simulation**: Adjusts signal time to simulate movement towards or away from stations.
- **Data Transmission**: Transmits updated signal times to track the robot's position.

### Key Parameters

- **Variance**: Controls random fluctuation in signal time to simulate environmental effects.
- **Mean Signal Time**: The baseline signal time when the robot is at a known position.
- **Start/End Time**: Initial and maximum signal times that help reverse direction once limits are reached.

---

## OilSensor

### Overview

The **OilSensor.c** file simulates oil level changes over time, fluctuating between states of oil increase, decrease, or steadiness. It mimics natural oil behavior with random fluctuations and periodic updates.

### Key Functionality

- **Oil Level Simulation**: Simulates oil level increase, decrease, or steady states, with transitions between these states based on predefined conditions.
- **State Transitions**: The sensor moves through states (increase, decrease, steady), influenced by random variance.
- **High Oil Detection**: Detects unusually high oil levels in the steady state, simulating potential leaks.
- **Data Transmission**: Sends periodic updates on the oil level.

### Key Parameters

- **precision**: The accuracy of oil level readings (in units).
- **Variance**: Random fluctuation during oil increase, decrease, or steady states.
- **maxCount**: Controls state transitions based on oil level changes.

---

## DepthSensorController

### Overview

The **DepthSensorController.c** file simulates the robot’s depth as it moves up and down. Depth is updated periodically based on the robot’s movement, and it is constrained within a defined range, from the start depth to the maximum depth.

### Key Functionality

- **Depth Calculation**: Updates depth at regular intervals based on the robot’s movement direction.
- **Movement Direction Control**: The robot alternates between moving up and down, constrained within depth boundaries.
- **Data Transmission**: Periodically sends updated depth values to monitor changes in real-time.

### Key Parameters

- **MaxDepth**: Maximum depth the robot can reach (100 meters).
- **StartDepth**: The initial depth of the robot (20 cm).
- **RobotSpeed**: The fixed movement speed (20 cm/s).

---

## Dependencies

- **FreeRTOS**: Used for task management, timers, and inter-process synchronization.
- **Comm_Datalink**: Handles communication with the datalink for receiving and sending messages.
- **SBLSensor**, **DepthSensor**, **OilSensor**: Sensor drivers responsible for controlling the specific sensors.
- **SensorPlatform**: Manages overall sensor platform operations and communication.

## Acknowledgments

- **Andre Hendricks & Kadh1**: Provided initial development and system design.
- **Dr. JF Bousquet**: Contributed to the overall system architecture and implementation.
- **Evan Lowe**: Assisted with the development and sensor management.

- This system was built as part of a collaborative effort by **Connor McLeod**, **Evan Lowe**

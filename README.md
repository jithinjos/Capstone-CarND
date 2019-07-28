This is the final project of the Udacity Self-Driving Car Nanodegree: Programming a Real Self-Driving Car. Starter repo for project can be found [here](https://github.com/udacity/CarND-Capstone). And the simulator [here](https://github.com/udacity/CarND-Capstone/releases). I'm submitting this project individually.

For this project the Robot Operating System is used and several nodes are written that implemets the functionality of the self driving car. However, this is a partial solution only. I haven't implemented the **tl-detector** node. The car drives around in the simulator and stop at traffic lights, but it uses the traffic light info from simulator.

[//]: # (Image References)
[image1]: ./imgs/carla_architecture.png
[image2]: ./imgs/system_architecture.png

### Setup
The project [repo](https://github.com/udacity/CarND-Capstone) discusses 2 methods: either native **or** docker installation. I ran my code using docker and the simulator in native ubuntu.

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
To set up port forwarding, please refer to the "uWebSocketIO Starter Guide" found in the classroom (see Extended Kalman Filter Project lesson).

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
4. Run the simulator. Untick the checkbox **Manual** on simulator.

I'll skip the real world testing, as I haven't got there.

### Other library/driver information
Outside of `requirements.txt`, here is information on other driver/library versions used in the simulator and Carla:

Specific to these libraries, the simulator grader and Carla use the following:

|        | Simulator | Carla  |
| :-----------: |:-------------:| :-----:|
| Nvidia driver | 384.130 | 384.130 |
| CUDA | 8.0.61 | 8.0.61 |
| cuDNN | 6.0.21 | 6.0.21 |
| TensorRT | N/A | N/A |
| OpenCV | 3.2.0-dev | 2.4.8 |
| OpenMP | N/A | N/A |


## Project Overview
Udacity has a Lincoln MKZ converted into a self-driving car. The fundamentals of any self-drivig car is similar. It has 4 major sub-systems: **Sensors**, **Perception**, **Planning** and **Control** 

![][image1]

#### Sensors
This system consist of devices needed to understand the surroundings of the self-driving unit. It includes **cameras**, **lidar**, **GPS**, **radar**, and **IMU**

#### Perception 
This can be classified into **detection** and **localization**
- Detection is the part where the software identifies things around it-using information from the sensors. Lane lines, road signs, traffic signals, other cars or pedestrians
- Localization is placing the self driving car on a map. It uses GPS and a myriad other information to improve the accuracy. Sensor Fusion is big in here.

### Planning
Path planning can be broken down as: **route planning**, **prediction**, **behavioral planning**, and **trajectory planning**
- Route planning is the high-level idea of how to get from point A to B. It is more like the output from google maps or any similar navigation app.
- Prediction estimates the future state of objects around the car. A big part is the information of other cars on road.
- Behavioral planning determines the action needed to define the future state of the self-driving car. The car needs change in behavior - accelerate, decelerate, brake, change lane etc... based on a road sign, traffic signal or other vehicles on road.
- Trajectory planning is the final behavior that the car needs to follow. unlike the initial route planned, this is one that gets constantly updated.

### Control
The control component takes trajectory outputs and processes them with algorithms like **PID** or **MPC** to adjust the control inputs for smooth operation of the vehicle.

## Architecture and ROS Node Design
The ROS Architecture consists of nodes (written in Python or C++) that communicate with each other via ROS messages. For this project we have a central styx-server that links the simulator and ROS by providing information about the car's state and surroundings (car's current position, velocity and images of the front camera) and receiving control input (steering, braking, throttle). For this project I updated the waypoint updater node, Drive-By_Wire and the Traffic light detector.

![][image2]

### Waypoint Updater
The waypoint Updator publishes a set number of waypoints at a desired rate for the car to follow. It calls base waypoint, the car's pose and traffic waypoint frequently to calculate the desired waypoint list. The function to stop the car by decelerating smoothly at red light is also built in.

### Traffic Light Detector
In my solution, I'm using the information passed on from simulator to infer traffic light status and plan path accordingly. For the self-driving car, the camera images will have to run through a trained neural network to identify them as red/yello/green and pass the information on to plan the path.

### Drive-By-Wire (DBW) Node
The DBW node is responsible for steering the car. It subscribes to a twist controller which outputs throttle, brake and steering values with the help of a PID-controller and Lowpass filter. The dbw node directly publishes throttle, brake and steering commands for the car/simulator, in case dbw_enabled is set to true.
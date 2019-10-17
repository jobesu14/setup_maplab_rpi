## Install OS + ROS

Follow the instructions to install Ubiquity Robotics Raspberry Pi images based on Ubuntu 16.04.
https://downloads.ubiquityrobotics.com/pi.html

Don't forget to disable Ubiquity Robotics startup scripts

```sudo systemctl disable magni-base```

## Install Raspicam node

Instruction from [here](https://github.com/SamSpaulding/ros_raspberry_pi_zero)
([Maybe those instructions make more sense?](https://github.com/UbiquityRobotics/raspicam_node))

Enable camera and I2C

```sudo raspi-config```

The raspicam node software below requires the following GPU interface code be checked out at the location /home/pi/userland, so we download this repository here:

```
sudo mkdir -p /home/pi/userland
sudo git clone https://github.com/raspberrypi/userland.git /home/pi/userland/
```

Create a vio workspace where we will build Raspicam node

```
mkdir -p ~/vio-ws/src
cd ~/vio-ws/src
sudo git clone https://github.com/raspberrypi/userland.git /home/pi/userland/
cd ~/vio-ws/
catkin_make
source devel/setup.bash
```

Running the camera node (roscore should be already running)

```
rosrun raspicam raspicam_node &
rosservice call /camera/start_capture
```

Viewing the camera node

```
rqt_image_view image:=/raspicam_node/camera/image/compressed
```

## Install Maplab

TODO

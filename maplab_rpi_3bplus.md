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
export CAM_WS=~/raspicam_ws
mkdir -p $CAM_WS/src
cd $CAM_WS/src
sudo git clone https://github.com/raspberrypi/userland.git /home/pi/userland/
cd $CAM_WS/
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

## Install Rovio

Increase the swap size

```
sudo apt-get install dphys-swapfile
free -h
```

```free -h``` should tell you the the swap size is 1.7G. If it is not the case, you can increase the swap file like that (not tested):

```
sudo nano /etc/dphys-swapfile
CONF_SWAPSIZE=100 // Change to your preferred amount (Ie. 2048)
sudo dphys-swapfile setup
sudo /etc/init.d/dphys-swapfile stop
sudo /etc/init.d/dphys-swapfile start
```

Create Rovio workspace and build it with one job to avoid overheating (that should take around 1h10).

```
mkdir -p rovio_ws/src
cd rovio_ws/src
git clone https://github.com/ANYbotics/kindr.git
git clone https://github.com/ethz-asl/rovio.git
cd rovio
git submodule update --init --recursive
cd ~/rovio_ws
catkin build rovio -j1 --cmake-args -DCMAKE_BUILD_TYPE=Release
```

## Install Maplab (To test with catkin -j1 and swap file)

```
sudo apt install autotools-dev ccache doxygen dh-autoreconf git liblapack-dev libblas-dev libgtest-dev libreadline-dev libssh2-1-dev pylint clang-format-3.8 python-autopep8 python-catkin-tools python-pip python-git python-setuptools python-termcolor python-wstool libatlas3-base --yes
sudo -H python -m pip install --upgrade pip
# sudo -H python -m pip install requests
sudo apt install -y ccache &&\
echo 'export PATH="/usr/lib/ccache:$PATH"' | tee -a ~/.bashrc &&\
source ~/.bashrc && echo $PATH
ccache --max-size=2G
```

Go back to the vio-ws

```
export MAPLAB_WS=~/maplab_ws
mkdir -p $CATKIN_WS/src
cd $MAPLAB_WS
catkin config --merge-devel # Necessary for catkin_tools >= 0.4.
catkin config --extend /opt/ros/kinetic/
catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release
cd $MAPLAB_WS/src
git clone https://github.com/ethz-asl/maplab.git --recursive
git clone https://github.com/ethz-asl/maplab_dependencies --recursive
cd $MAPLAB_WS/src/maplab
./tools/linter/init-git-hooks.py
cd $MAPLAB_WS
catkin build maplab
```

## Install OS + ROS

Follow the instructions to install Ubiquity Robotics Raspberry Pi images based on Ubuntu 16.04.
https://downloads.ubiquityrobotics.com/pi.html

Don't forget to disable Ubiquity Robotics startup scripts

```sudo systemctl disable magni-base```

## Few rpi setup (optional)

Automatic NTPD to update system time (needed to use the Firefox).
[More on NTP time sync on Linux](https://www.tecmint.com/synchronize-time-with-ntp-in-linux/)

```
sudo timedatectl set-ntp true
sudo timedatectl set-timezone "Europe/Zurich"
```

Change the keyboard layout to swiss french, modify the two lines below and save the file.

```
sudo nano /etc/default/keyboard
```

```
XKBLAYOUT="ch"
XKBVARIANT="fr"
```

## Install Raspicam Ubiquity

[Instruction and more information here](https://github.com/UbiquityRobotics/raspicam_node)

```
sudo sh -c 'echo "deb https://packages.ubiquityrobotics.com/ubuntu/ubiquity xenial main" > /etc/apt/sources.list.d/ubiquity-latest.list'
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key C3032ED8
sudo apt-get update
sudo apt install ros-kinetic-raspicam-node
```

To launch the raspicam node: ```roslaunch raspicam_node camerav2_410x308_30fps.launch```

The raspicam launch files are under: ```/opt/ros/kinetic/share/raspicam_node/launch```

To view the video feed on a connected computer: ```rqt_image_view```

Run the dynamic reconfigure node on a connected computer: ```rosrun rqt_reconfigure rqt_reconfigure```

## Install Rovio

Increase the swap size

```
sudo apt-get install dphys-swapfile
free -h
source devel/setup.bash
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

## Install IMU libraries

Tested with the Adafruit [NXP Precision 9DoF IMU Breakout](https://learn.adafruit.com/nxp-precision-9dof-breakout/overview).
The [CPython code](https://learn.adafruit.com/nxp-precision-9dof-breakout/python-circuitpython) to read the IMU data is also from Adafruit.
To install it:

```
pip3 install --upgrade pip
sudo python3 -m pip install adafruit-circuitpython-lis3dh
sudo python3 -m pip install adafruit-circuitpython-fxos870
sudo python3 -m pip install adafruit-circuitpython-fxas21002c
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




## Install Raspicam node (old)

Instruction from [here](https://github.com/SamSpaulding/ros_raspberry_pi_zero)
([Maybe those instructions make more sense?](https://github.com/UbiquityRobotics/raspicam_node))

Enable camera and I2C

```sudo raspi-config```

The raspicam node software below requires the following GPU interface code be checked out at the location /home/pi/userland, so we download this repository here:

```
sudo mkdir -p /home/pi/userland
sudo git clone https://github.com/raspberrypi/userland.git /home/pi/userland/
```

Create a raspicam workspace where we will build Raspicam node

```
mkdir -p raspicam_ws/src
cd raspicam_ws/srcsudo timedatectl set-ntp true

sudo git clone https://github.com/raspberrypi/userland.git /home/pi/userland/
cd raspicam_ws
catkin_make
```

Running the camera node (roscore should be already running)

```
cd 
source devel/setup.bash
rosrun raspicam raspicam_node &
rosservice call /camera/start_capture
```

Viewing the camera node

```
rqt_image_view image:=/raspicam_node/camera/image/compressed
```

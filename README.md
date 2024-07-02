# Autoware Install
Ubuntu 22.04

ROS2 humble

nvidia-smi: Driver Version: 555.42.02 CUDA Version: 12.5

## Prepare
make sure there is no old 'autoware', 'autoware_data', 'autoware_map' folder
```
cd ~
```
update and git install
```
sudo apt-get -y update
sudo apt-get -y install git
```
git clone
```
git clone https://github.com/autowarefoundation/autoware.git
gdown -O ~/autoware_map/ 'https://docs.google.com/uc?export=download&id=1499_nsbUbIeturZaDj7jhUownh5fvXHd'
unzip -d ~/autoware_map ~/autoware_map/sample-map-planning.zip
```
#### Memory check (In another terminal)
```
free -h
```
if you do not have 32G memory, do the following:

Remove the current swapfile
```
sudo swapoff /swapfile
sudo rm /swapfile
```
Create a new swapfile
```
sudo fallocate -l 32G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```
Check if the change is reflected
```
free -h
```
## Option-1(Source-Build): Workspace Setup
In autoware folder
#### Environment Setup (first time)
```
cd autoware
./setup-dev-env.sh
```
#### Build workspace
```
mkdir src
vcs import src < autoware.repos
sudo apt update
rosdep update
rosdep install -y --from-paths src --ignore-src --rosdistro $ROS_DISTRO
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
```
colon build will take up to 1 hour, could quit and terminal all other processes to avoid stuck.
## Option-2 (Docker-Build): Development setup
**Docker Image: ghcr.io/autowarefoundation/autoware:latest-devel-cuda**
In autoware folder
#### Environment Setup (first time)
```
cd autoware
./setup-dev-env.sh -y --download-artifacts docker
```
#### Pull docker
```
./docker/run.sh --devel --map-path ~/autoware_map/sample-map-planning --workspace ~/autoware
```
After these, you should launch Autoware and rviz successfully, and you could see the same screen as shown in [Planning simulation](https://autowarefoundation.github.io/autoware-documentation/main/tutorials/ad-hoc-simulation/planning-simulation/)

Local folder 'autoware' will be mounted on '/workspace' by default.
## Option-3 (Docker-Build) OpenADKit Install
**Docker Images: ghcr.io/autowarefoundation/autoware-universe:latest-cuda**
```
git clone https://github.com/autowarefoundation/autoware.git
cd autoware
```
#### Environment Setup (first time)
```
cd autoware
./setup-dev-env.sh -y --download-artifacts docker
```
### Option-3.1 (Build-Own)
#### Login Docker
```
rocker --nvidia --x11 --user --volume $HOME/autoware_data -- ghcr.io/autowarefoundation/autoware-universe:latest-cuda
```
This will mount your local folder $~/autoware_data$ to your container and connect your local x11 rviz visualization to this container.
#### Docker prepare
```
cd ~
sudo apt-get -y update
sudo apt-get -y install git
sudo apt-get install unzip
git clone https://github.com/autowarefoundation/autoware.git
gdown -O ~/autoware_map/ 'https://docs.google.com/uc?export=download&id=1499_nsbUbIeturZaDj7jhUownh5fvXHd'
unzip -d ~/autoware_map ~/autoware_map/sample-map-planning.zip
```
#### Workspace setup
```
cd autoware
mkdir src
vcs import src < autoware.repos
sudo apt update
rosdep update
rosdep install -y --from-paths src --ignore-src --rosdistro $ROS_DISTRO
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
```
### Option-3.2 (Use-Pre-Build)
On your local machine
```
docker pull ariiees/openadk:build
```
You can check this imageID using the following command:
```
docker images
```
Put this imageID into the following command:
```
docker run -it  -v $HOME/autoware_data:$HOME/autoware_data  -e DISPLAY -e TERM   -e QT_X11_NO_MITSHM=1   -e XAUTHORITY=/tmp/.docker5guv3arw.xauth -v /tmp/.docker5guv3arw.xauth:/tmp/.docker5guv3arw.xauth   -v /tmp/.X11-unix:/tmp/.X11-unix   -v /etc/localtime:/etc/localtime:ro [imageID] /bin/bash
```
Inside the docker, test tutorial
```
source ~/autoware/install/setup.bash
ros2 launch autoware_launch planning_simulator.launch.xml map_path:=$HOME/autoware_map/sample-map-planning vehicle_model:=sample_vehicle sensor_model:=sample_sensor_kit
```

# Autoware Build
Ubuntu 22.04

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
### Memory check (In another terminal)
'''
free -h
'''
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
## Environment Setup (first time)
In autoware folder
```
cd autoware
./setup-dev-env.sh -y --download-artifacts docker
```
## Option-1(Source-Build): Workspace Setup (NO SUDO!!, in autoware folder)
```
mkdir src
vcs import src < autoware.repos
sudo apt update
rosdep update
rosdep install -y --from-paths src --ignore-src --rosdistro $ROS_DISTRO
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
```
colon build will take up to 1 hour, could quiet and terminal all other processes to avoid stuck.
## Option-2 (Docker-Build): Development setup
```
./docker/run.sh --devel --map-path [absolute path]
```
The [absolute path] could be checked using the following command:
```
cd ~/autoware_map/sample-map-planning
pwd
```

After these, you should launch autoware and rviz successfully, and you could see the same screen as shown in [https://autowarefoundation.github.io/autoware-documentation/main/tutorials/ad-hoc-simulation/planning-simulation/](Planning simulation)
## Build docker image for devp
```
cd autoware/
./docker/build.sh --devel-only
```

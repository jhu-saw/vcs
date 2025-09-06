# Introduction

This repository contains `vcs` files which can be used with the ROS tool `vcs` to clone and maintain multiple repositories in a single ROS workspace.

Most cisst/SAW components use multiple git repositories so we provide some `vcs` files to automatically find the repositories (and versions) needed for each SAW component.

In general, the `vcs` command will look like:
```bash
 vcs import --recursive --input https://raw.githubusercontent.com/jhu-saw/vcs/main/ros<1|2>-<component>-<branch|tag>.vcs
```

You will have to edit the URL based on the ROS version, component and branch or version.  For example, for ROS2 sawAtracsysFusionTrack main branch, the command line would be:
```bash
vcs import --recursive --input https://raw.githubusercontent.com/jhu-saw/vcs/main/ros2-atracsys-main.vcs
```

Note that if you need multiple SAW components in the same workspace, you need to first make sure the versions of all the dependencies for each component match.  In general, the `main` and `devel` branches should be compatible.  You can then use multiple calls to `vcs`:
```bash
vcs import --recursive --input https://raw.githubusercontent.com/jhu-saw/vcs/main/ros2-atracsys-main.vcs
vcs import --recursive --input https://raw.githubusercontent.com/jhu-saw/vcs/main/ros2-universal-robot-main.vcs
```

By convention, the `main` branch should be stable and all new features should be included in the next release. It is updated between releases (tagged) so users can use a recent version of the code. The `devel` branches might contain code that has not been tested or might not be included in the next release.

> **:warning: connection reset by peer**
>
> `vcs` will sometime fail with some ssh related errors.  If that happen, you might want to add the command line options `--workers 1 --retry 10`.  For example: `vcs import --workers 1 --retry 10 --recursive --input https://raw.githubusercontent.com/jhu-saw/vcs/main/ros2-dvrk-devel.vcs`

# Compilation

> **:warning: Python and environment variables**
> 
> ROS uses Python extensively for the build process.  Make sure you use the default Python and deactivate any local Python install (`which python` should return `/usr/bin/python`).  This is a common issue with virtualenv and anaconda.<br>
> You should also make sure your ROS environment variables match the workspace and ROS version you're using (check with `env | grep -i ros`).  If your environment variables are incorrect, you might want to edit either `~/.bashrc` or `~/.profile` and logout for the changes to take effect.

## ROS1

We recommend to use `catkin build`.  Some of the cisst/SAW packages might not compile properly if you're using `catkin_make`.

### Requirements

* Ubuntu 18.04:
  ```bash
  sudo apt install libxml2-dev libraw1394-dev libncurses5-dev qtcreator swig sox espeak cmake-curses-gui cmake-qt-gui git subversion gfortran libcppunit-dev libqt5xmlpatterns5-dev libbluetooth-dev libhidapi-dev python-vcstool python-catkin-tools clang
  ```
* Ubuntu 20.04:
  ```bash
  sudo apt install libxml2-dev libraw1394-dev libncurses5-dev qtcreator swig sox espeak cmake-curses-gui cmake-qt-gui git subversion gfortran libcppunit-dev libqt5xmlpatterns5-dev libbluetooth-dev libhidapi-dev python3-pyudev python3-vcstool python3-catkin-tools python3-osrf-pycommon python-is-python3
  ```

### Download and compile

:warning: Ubuntu 18.04 support requires clang instead of gcc.  You will need to configure your workspace using: `catkin config --cmake-args -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DCMAKE_BUILD_TYPE=Release`

```sh
source /opt/ros/noetic/setup.bash # or melodic
mkdir ~/catkin_ws
cd ~/catkin_ws
catkin init
catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release
mkdir src
cd src
vcs import --recursive --input https://raw.githubusercontent.com/jhu-saw/vcs/main/ros1-<component>-<branch|tag>.vcs
catkin build --summary
source ~/catkin_ws/devel/setup.bash
```

## ROS 2

### Requirements

This has to be done once per computer.  Install ROS 2 following instructions from www.ros.org.  The following packages might not be installed by default:
```sh
sudo apt install python3-vcstool python3-colcon-common-extensions python3-pykdl
```

For cisst/SAW, you will also need the following Ubuntu packages:
* Ubuntu 20.04:
  ```sh
  sudo apt install libxml2-dev libraw1394-dev libncurses5-dev qtcreator swig sox espeak cmake-curses-gui cmake-qt-gui git subversion gfortran libcppunit-dev libqt5xmlpatterns5-dev libbluetooth-dev libhidapi-dev python3-pyudev ros-galactic-joint-state-publisher* ros-galactic-xacro
  ```
* Ubuntu 22.04:
  ```sh
  sudo apt install libxml2-dev libraw1394-dev libncurses5-dev qtcreator swig sox espeak cmake-curses-gui cmake-qt-gui git subversion gfortran libcppunit-dev libqt5xmlpatterns5-dev libbluetooth-dev libhidapi-dev python3-pyudev gfortran-9 ros-humble-joint-state-publisher* ros-humble-xacro
  ```

### Download and compile

Create your ROS 2 workspace and clone all repositories using `vcs`:
```bash
source /opt/ros/galactic/setup.bash # or humble, iron...
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
vcs import --recursive --input https://raw.githubusercontent.com/jhu-saw/vcs/main/ros2-<component>-<branch|tag>.vcs
cd ~/ros2_ws
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
source ~/ros2_ws/install/setup.bash
```

# VCS commands

All these commands should be used in your source (`src/`) directory!

* To update all the repositories in your source directory:
  ```sh
  vcs pull
  ```
* To see all the local changes across repositories:
  ```sh
  vcs status -s
  ```
* After you commit all your changes, you can make sure all are pushed to their remote:
  ```sh
  vcs push
  ```
* A bit more advanced, assuming all your repositories have a main and a devel branch, you can toggle back and forth:
  ```sh
  # move all to devel and update
  vcs custom --git --args checkout devel
  vcs pull
  # back to main
  vcs custom --git --args checkout main
  vcs pull
  ```

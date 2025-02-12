[![Build Status](https://github.com/gazebosim/ros_gz/actions/workflows/ros2-ci.yml/badge.svg?branch=ros2)](https://github.com/gazebosim/ros_gz/actions/workflows/ros2-ci.yml)

# Notes for Apple Silicon Installation

First, complete the installation of ROS2 Jazzy and Gazebo using [ROS2_Jazzy_MacOS_Native_AppleSilicon](https://github.com/IOES-Lab/ROS2_Jazzy_MacOS_Native_AppleSilicon).
```
# ROS2 Jazzy and Gazebo installation script (not needed if already executed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/IOES-Lab/ROS2_Jazzy_MacOS_Native_AppleSilicon/main/install.sh)"
```
Upon completion, the paths of the installed ROS2 Jazzy and Gazebo are saved in `$HOME/.ros2_jazzy_install_config`. (Default paths are `ros2_jazzy`, `gz_harmonic`)
Check the contents of the config file with `cat $HOME/.ros2_jazzy_install_config`

Now, to install `ros_gz`, proceed as follows:

- Diable SIP
  - Restart the Mac with long press power button
  - Select `Options` to boot into a recovery mode 
  - Use `Utilities` -> `Terminal` and type below command to disable SIP
    ```
    csrutil disable
    ```

- Make workspace directory
  ```
  mkdir -p $HOME/ros_gz_ws/src
  git clone https://github.com/IOES-Lab/ROS_GZ_MacOS_Native_AppleSilicon.git
  git clone https://github.com/swri-robotics/gps_umd.git
  git clone https://github.com/rudislabs/actuator_msgs.git
  git clone https://github.com/ros-perception/vision_msgs.git
  cd $HOME/ros_gz_ws
  ```

- Set environment variables
  ```
  export CMAKE_PREFIX_PATH=$(brew --prefix qt@5)/lib:$(brew --prefix qt@5)/lib/cmake:/opt/homebrew/opt:${CMAKE_PREFIX_PATH}
  export PATH=$(brew --prefix qt@5)/bin:$PATH
  # This is rough.. but works
  ln -s $(brew --prefix qt@5)/mkspecs /opt/homebrew/mkspecs
  ln -s $(brew --prefix qt@5)/plugins /opt/homebrew/plugins
  ```

- Modify `TINYXML2:TINYXML2` at `gz-msgs10` cmake file
  ```
  sudo nano /opt/homebrew/opt/gz-msgs10/lib/cmake/gz-msgs10/gz-msgs10-targets.cmake
  # Find TINYXML2:TINYXML2 and replace with tinyxml2::tinyxml2
  ```

- Compile
  ```
  source $HOME/ros2_jazzy/activate_ros
  python3.11 -m colcon build --symlink-install \
      --packages-skip-by-dep python_qt_binding \
      --cmake-args \
      -DBUILD_TESTING=OFF \
      -DCMAKE_OSX_SYSROOT=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk \
      -Wno-dev --event-handlers console_cohesion+
  ```

- Add script to activate_ros
  ```
  # If Bash
  echo "source $HOME/ros_gz_ws/install/setup.bash" >> $HOME/ros2_jazzy/activate_ros
  # If Zsh
  echo "source $HOME/ros_gz_ws/install/setup.zsh" >> $HOME/ros2_jazzy/activate_ros
  ```

- Revert back `TINYXML2:TINYXML2` at `gz-msgs10` cmake file
  ```
  sudo nano /opt/homebrew/opt/gz-msgs10/lib/cmake/gz-msgs10/gz-msgs10-targets.cmake
  # Find tinyxml2::tinyxml2 and replace with  TINYXML2:TINYXML2
  ```

- Re-enable SIP
  - Restart the Mac with long press power button
  - Select `Options` to boot into a recovery mode 
  - Use `Utilities` -> `Terminal` and type below command to disable SIP
    ```
    csrutil enable
    ```

## Notes

- Finding the USB connection port
  ```
  system_profiler SPUSBDataType | awk '/ArduPilot/{found=1} found && /Location ID/{print "/dev/cu.usbmodem" int(substr($3,3,3) "01"); found=0}'
  ```



# Integration between ROS and Gazebo

## Packages

This repository holds packages that provide integration between
[ROS](http://www.ros.org/) and [Gazebo](https://gazebosim.org):

* [ros_gz](https://github.com/gazebosim/ros_gz/tree/ros2/ros_gz):
  Metapackage which provides all the other packages.
* [ros_gz_image](https://github.com/gazebosim/ros_gz/tree/ros2/ros_gz_image):
  Unidirectional transport bridge for images from
  [Gazebo Transport](https://gazebosim.org/libs/transport)
  to ROS using
  [image_transport](http://wiki.ros.org/image_transport).
* [ros_gz_bridge](https://github.com/gazebosim/ros_gz/tree/ros2/ros_gz_bridge):
  Bidirectional transport bridge between
  [Gazebo Transport](https://gazebosim.org/libs/transport)
  and ROS.
* [ros_gz_sim](https://github.com/gazebosim/ros_gz/tree/ros2/ros_gz_sim):
  Convenient launch files and executables for using
  [Gazebo Sim](https://gazebosim.org/libs/gazebo)
  with ROS.
* [ros_gz_sim_demos](https://github.com/gazebosim/ros_gz/tree/ros2/ros_gz_sim_demos):
  Demos using the ROS-Gazebo integration.
* [ros_gz_point_cloud](https://github.com/gazebosim/ros_gz/tree/ros2/ros_gz_point_cloud):
  Plugins for publishing point clouds to ROS from
  [Gazebo Sim](https://gazebosim.org/libs/gazebo) simulations.


---
title: "ROS2 Humble with zsh"
date: 2022-02-17 16:31:00 -0400
category: ros2
draft: false
---

## Add the ROS2 apt repository

```bash
sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install curl
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu jammy main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

## Install ROS2

```bash
sudo apt update
sudo apt install ros-humble-desktop
sudo apt install ros-dev-tools
```

## Install other packages

```bash
sudo apt update && sudo apt install -y build-essential cmake git libbullet-dev python3-colcon-common-extensions python3-flake8 python3-pip python3-pytest-cov python3-rosdep python3-setuptools python3-vcstool wget python3-argcomplete
python3 -m pip install -U flake8-blind-except flake8-builtins flake8-class-newline flake8-comprehensions flake8-deprecated flake8-docstrings flake8-import-order flake8-quotes pytest-repeat pytest-rerunfailures pytest
sudo apt install --no-install-recommends -y libasio-dev libtinyxml2-dev libcunit1-dev
```

## Humble environment setup shell (for zsh, add to ~/.zshrc)

```bash
source /opt/ros/humble/setup.zsh
source ~/robot_ws/install/local_setup.zsh
source ~/uros_ws/install/local_setup.zsh

source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.zsh
source /usr/share/vcstool-completion/vcs.zsh
source /usr/share/colcon_cd/function/colcon_cd.sh
export _colcon_cd_root=~/robot_ws

# argcomplete for ros2 & colcon
eval "$(register-python-argcomplete3 ros2)"
eval "$(register-python-argcomplete3 colcon)"

# export ROS_NAMESPACE=robot1

export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
# export RMW_IMPLEMENTATION=rmw_connext_cpp
# export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
# export RMW_IMPLEMENTATION=rmw_gurumdds_cpp

# export RCUTILS_CONSOLE_OUTPUT_FORMAT='[{severity} {time}] [{name}]: {message} ({function_name}() at {file_name}:{line_number})'
export RCUTILS_CONSOLE_OUTPUT_FORMAT='[{severity}]: {message}'
export RCUTILS_COLORIZED_OUTPUT=1
export RCUTILS_LOGGING_USE_STDOUT=0
export RCUTILS_LOGGING_BUFFERED_STREAM=1

alias cw='cd ~/robot_ws'
alias cs='cd ~/robot_ws/src'
alias ccd='colcon_cd'

alias cb='cd ~/robot_ws && colcon build --symlink-install'
alias cbs='colcon build --symlink-install'
alias cbp='colcon build --symlink-install --packages-select'
alias cbu='colcon build --symlink-install --packages-up-to'
alias ct='colcon test'
alias ctp='colcon test --packages-select'
alias ctr='colcon test-result'

alias tl='ros2 topic list'
alias te='ros2 topic echo'
alias nl='ros2 node list'

alias killgazebo='killall -9 gazebo & killall -9 gzserver & killall -9 gzclient'

alias af='ament_flake8'
alias ac='ament_cpplint'

alias testpub='ros2 run demo_nodes_cpp talker'
alias testsub='ros2 run demo_nodes_cpp listener'
alias testpubimg='ros2 run image_tools cam2image'
alias testsubimg='ros2 run image_tools showimage'

alias di='rosdep install --from-paths src -y --ignore-src --os=ubuntu:jammy'

# export ROS_DOMAIN_ID=7
```

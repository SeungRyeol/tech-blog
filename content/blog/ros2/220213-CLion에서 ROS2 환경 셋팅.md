---
title: "CLion에서 ROS2 환경 셋팅"
date: 2022-02-13 16:31:00 -0400
category: ros2
draft: false
---

## ROS2 setup tutorial in CLion

1. Prepare the ROS2 environment

2. Create a directory for a new workspace and navigate into it (e.g. ~/robot_ws)

3. Create an example package

    ```bash
    cd ~/robot_ws && mkdir src && cd src
    ros2 pkg create --build-type ament_cmake my_project
    ```

4. Build a workspace in terminal and generate a compilation database

    - Let's assume that only **selected packages** are built.

        - `--packages-select` option

    ```bash
    cd ..
    colcon build --cmake-args -DCMAKE_EXPORT_COMPILE_COMMANDS=ON -G Ninja --symlink-install --packages-select my_project
    ```

5. When build is finished, make sure that **robot_ws\build** contains a **compile_commands.json** file.

6. In CLion, call **File - Open** from the main menu and select the **compile_commands.json** file in the top-level **build** directory

7. Call **Tools - Compilation Database - Change Project Root** from the main menu and select the workspace directory (**robot_ws** in our case).

## How to build a package

### 1. Create a script for the `colcon build` commands

1. Navigate to the build logs directory. In our case, it's **~/robot_ws/log/latest_build my_project**. Open the **command.log** file from the latest build.

2. Copy the commands into another file and modify them to the following:

    ```bash
    #!/bin/zsh

    source /opt/ros/humble/setup.zsh
    source ~/robot_ws/install/local_setup.zsh
    CMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}:/opt/ros/humble /usr/bin/cmake ~/robot_ws/build/my_project -DCMAKE_EXPORT_COMPILE_COMMANDS=ON -DCMAKE_INSTALL_PREFIX=~/robot_ws/install/my_project
    CMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}:/opt/ros/humble /usr/bin/cmake --build ~/robot_ws/build/my_project -- -j23 -l23
    CMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}:/opt/ros/humble /usr/bin/cmake --install ~/robot_ws/build/my_project
    ```

3. Save the file as a `.zsh` script. In our example, it is called `cmake_commands.zsh` and placed into `~/robot_ws/build/my_project`.

4. Make the script executable.

    ```bash
    chmod +x cmake_commands.zsh
    ```

### 2. Create a custom build target

1. In CLion, go to **Settings / Preferences - Build, Execution, Deployment - Custom Build Targets** and click '+' to add a new target.

2. Set each item.

    - **Toolchain** field: Set the value to default.

    - **Build** field: Click '...' icon to add an external.

        - **Program** field: Select the newly created script(cmake_commands.zsh).

        - Set the Working directory to the package build directory.

        <img src="https://user-images.githubusercontent.com/33774319/154174313-ca466623-3fb9-4d3a-8b99-5770c68f6ed4.png"/>

3. Save the custom target.

### 3. Create a run/debug configuration for the custom build target

1. Go to **Run - Edit Configurations**, click '+' and select **Custom Build Application**.

2. In the configuration settings, select the **Target** and make sure to remove **Build** from the Before launch area.

### 4. Build a package

1. Select the build configuration.

2. Build through the icon.

### 5. Run/debug a package

1. Open the **Edit Configurations** dialog again.

2. Modify the configuration for build to the following:

    - **Executable** field: select binary file or script file.

        - example: ~/robot_ws/build/my_project/src/run_debug_commands.zsh

            ```bash
            #!/bin/zsh

            source /opt/ros/humble/setup.zsh
            source ~/robot_ws/install/local_setup.zsh
            ~/robot_ws/build/manibot_bringup/joy_commander
            ```

    - option: Select the Run with Administrator privileges checkbox.

3. After saving the configuration, it is ready to be **Run** or **Debugged**.

## 참고자료

[ROS2 setup tutorial in CLion](https://www.jetbrains.com/help/clion/ros2-tutorial.html)

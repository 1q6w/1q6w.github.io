+++
date = '2025-05-07T15:58:32+08:00'
draft = false
title = 'Ros2_install'
categories = ["机器人操作系统"]
tags = ["Ros2-humble"]
+++

- **docker + Ros2 hunble 环境构建**

     ~~~bash
     # 拉取Ros2 humble docker镜像 
     docker pull osrf/ros:humble-desktop-full 
     # 运行docker
     docker run -it --privilieged \
       --gpus all \
       --name ros2_humble \
       --network host \
       -v /tmp/.X11-unix:/tmp/.X11-unix \
       -v ~/.Xauthority:/root/.Xauthority \
       -e DISPLAY=$DISPLAY \
       -v /home/x/ROS_WS:/home/x/ws \
       osrf/ros:humble-desktop-full
     ~~~

-   **添加环境变量**

~~~bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
~~~

-   **验证环境是否搭载成功**

~~~bash
# 检查ros2 turtlesim 是否正确安装
ros2 pkg executables turtlesim
# 启动 turtlesim 此时会弹出一个窗口，窗口内有一只小乌龟
ros2 run turtlesim turtlesim_node
# 新建终端，启动键盘控制小乌龟
ros2 run turtlesim turtle_teleop_key
~~~


- 不采用docker也可以通过鱼香ROS的一键安装命令
~~~bash
wget http://fishros.com/install -O fishros && . fishros
~~~

+++
date = '2025-05-07T15:58:32+08:00'
draft = true
title = 'Ros2_1'
+++

~~~
docker run -it --rm \
  --gpus all \
  --name ros2_humble \
  --network host \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -e DISPLAY=$DISPLAY \
  --device /dev/ttyUSB0 \
  -v /home/x/ROS_WS:/home/x/ws \
  -v ~/.ros:/root/.ros \
  -v ~/.gazebo:/root/.gazebo \
  osrf/ros:humble-desktop-full
~~~


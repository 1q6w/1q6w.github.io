+++
date = '2025-05-16T14:56:46+08:00'
draft = false
title = 'Docker Stm32 Toolchains'
ategories = ["工具链搭建"]
tags = ["stm32 docker arm-gcc工具链"]

+++

1. 创建Dockerfile文件

   ~~~dockerfile
   # 使用官方Ubuntu LTS镜像作为基础
   FROM ubuntu:22.04
   
   # 设置非交互式前端以避免安装过程中的提示
   ENV DEBIAN_FRONTEND=noninteractive
   
   # 安装基本工具和依赖
   RUN apt-get update && apt-get install -y \
       build-essential \
       cmake \
       git \
       wget \
       curl \
       unzip \
       libncurses-dev \
       libusb-1.0-0-dev \
       python3 \
       python3-pip \
       libx11-6 \
       libxext6 \
       libxrender1 \
       libxtst6 \
       libxi6 \
       libgtk-3-0 \
       openjdk-17-jre \
       libswt-gtk-4-jni \
       libswt-gtk-4-java \
       && rm -rf /var/lib/apt/lists/*
   
   # 安装ARM GCC工具链
   RUN wget https://developer.arm.com/-/media/Files/downloads/gnu-rm/10.3-2021.10/gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2 \
       && tar -xjf gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2 -C /opt \
       && rm gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2
   ENV PATH="/opt/gcc-arm-none-eabi-10.3-2021.10/bin:${PATH}"
   
   # 安装STM32CubeMX（使用本地文件）
   COPY en.stm32cubemx-lin-v6-14-1.zip /tmp/
   RUN unzip /tmp/en.stm32cubemx-lin-v6-14-1.zip -d /opt \
       && chmod +x /opt/STM32CubeMX/STM32CubeMX \
       && ln -s /opt/STM32CubeMX/STM32CubeMX /usr/local/bin/STM32CubeMX \
       && rm /tmp/en.stm32cubemx-lin-v6-14-1.zip
   
   # 安装J-Link软件包（包含Ozone）
   RUN wget https://www.segger.com/downloads/jlink/JLink_Linux_x86_64.deb \
       && dpkg -i JLink_Linux_x86_64.deb || apt-get install -f -y \
       && rm JLink_Linux_x86_64.deb
   
   # 创建工作目录
   RUN mkdir /workspace
   WORKDIR /workspace
   
   # 设置容器启动命令
   CMD ["/bin/bash"]
   ~~~

2. 下载STM32CubeMX zip安装包，并将复制到与`Dockerfile`文件相同目录下

3. 进行构建Docker镜像

   ~~~bash
   docker build -t stm32-dev-env .
   ~~~

4. 运行docker

   ~~~bash
   docker run -it --rm \
       -v /path/to/your/project:/workspace \
       -v /tmp/.X11-unix:/tmp/.X11-unix \
       -e DISPLAY=$DISPLAY \
       --device /dev/bus/usb \
       --privileged \
       stm32-dev-env
   ~~~

   


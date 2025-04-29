+++
date = '2025-04-29T16:34:29+08:00'
draft = false
title = 'Deploy_yolo'
+++
# 使用 Docker 容器训练YOLO部署到MaixCam

- **安装Docker**

  - 卸载旧版本：在安装 Docker Engine 之前，您需要卸载任何冲突的软件包。

    - 卸载 Docker Engine、CLI、containerd 和 Docker Compose 软件包

      ~~~bash
      sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
      ~~~

    - 主机上的镜像、容器、卷或自定义配置文件不会自动删除。要删除所有镜像、容器和卷

      ~~~bash
      sudo rm -rf /var/lib/docker
      sudo rm -rf /var/lib/containerd
      ~~~

    - 删除源列表和密钥环

      ~~~bash
      sudo rm /etc/apt/sources.list.d/docker.list
      sudo rm /etc/apt/keyrings/docker.asc
      ~~~


  - 使用 `apt` 仓库安装

      - 设置 Docker 的 `apt` 仓库

          ~~~bash
          # Add Docker's official GPG key:
          sudo apt-get update
          sudo apt-get install ca-certificates curl
          sudo install -m 0755 -d /etc/apt/keyrings
          sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
          sudo chmod a+r /etc/apt/keyrings/docker.asc
          
          # Add the repository to Apt sources:
          echo \
            "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
            $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
            sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
          sudo apt-get update
          ~~~

      - 安装 Docker 软件包

          ~~~bash
          sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
          ~~~

      - 通过运行 `hello-world` 镜像来验证安装是否成功

          ~~~bash
          sudo docker run hello-world
          ~~~

  - docker 换镜像源: 修改 /etc/docker/daemon.json，设置 registry mirror

    ~~~bash
    sudo mkdir -p /etc/docker
    sudo tee /etc/docker/daemon.json <<EOF
    {
        "registry-mirrors": [
            "https://docker.1ms.run",
            "https://docker.xuanyuan.me"
        ]
    }
    EOF
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    ~~~

- **docker pull images**

  ~~~bash
  sudo docker pull ultralytics/ultralytics:latest
  ~~~

  ~~~bash
  docker pull heartexlabs/label-studio:latest
  ~~~

- **准备本地数据集**

  - 用label stduio标注工具标注，导出带图像的YOLO格式数据集

  - ~~~bash
    docker run -it -p 8080:8080 -v $(pwd)/mydata:/label-studio/data heartexlabs/label-studio:latest
    ~~~

  - 标记好的数据集存储在主机的路径 `/home/user/datasets/yolo_dataset`，其中包含：
    - `images/` 文件夹：存储训练和验证的图片。
    - `labels/` 文件夹：存储对应的标注文件。
    - `labels/`：存储与图像对应的标签文件。
    - `labels/`：存储图像标签系统标签(0-999)

  - 运行分离数据集和验证集脚本

  - 在划分好的数据集下创建data.yaml

    ~~~yaml
    train: /workspace/dataset/dataset1/train/images
    val: /workspace/dataset/dataset1/val/images
    
    nc: 6
    names:
      - blackBall
      - blueBall
      - blueTarget
      - redBall
      - redTarget
      - yellowBall
    ~~~

- **运行 YOLO Docker 容器**

  - 在容器中显示图形应用程序(通过 X11)

    - 配置 X11
      - 确保主机安装了 X11（Linux 通常默认安装）。
      - 允许 X11 接收连接

    ~~~bash
    # 在主机终端运行
    xhost +local:docker
    ~~~

  - 使用 `-v` 参数将本地数据集挂载到容器中

    ~~~bash
    docker run --name YOLO \
      -it --gpus all \
      -e DISPLAY=$DISPLAY \
      -v /tmp/.X11-unix:/tmp/.X11-unix \
      -v /home/user/datasets/yolo_dataset:/workspace/dataset \
      -v /home/user/yolo_output:/workspace/output \
      ultralytics/ultralytics:latest
    ~~~

    - 为容器指定名称为 YOLO，方便后续管理（如停止、启动、删除等)
    - `-it`：交互模式。
    - `--gpus all`：启用 GPU 支持。
    - `-e DISPLAY=$DISPLAY`：将主机的 `DISPLAY` 环境变量传递给容器，用于图形显示。
    - `-v /tmp/.X11-unix:/tmp/.X11-unix`：挂载主机的 X11 套接字到容器中，允许图形应用程序显示。
    - **`/home/user/datasets/yolo_dataset`** 是主机上的数据集路径（源路径）。
    - **`/workspace/dataset`** 是容器内的数据集路径（目标路径）。
    - **`/home/user/yolo_output`** 是主机上的输出路径，用于保存训练结果。
    - **`ultralytics/ultralytics:latest`** 是 YOLO 的 Docker 镜像。

  - 权限路径问题

    ~~~bash
    sudo chmod -R 777 /home/user/datasets/yolo_dataset
    ~~~

  - GPU支持(cuda)

    - 安装nvidia驱动程序
    - 安装cuda container toolkit
    - 在docker运行时，使用 `--gpus all` 参数。

  - 验证图片显示

    - 进入容器后，可以运行一个简单的 GUI 应用程序（如 `xeyes` ）来验证 X11 是否正常工作。

    ~~~bash
    apt update && apt install -y x11-apps
    xeyes
    ~~~

- **容器训练**

  - 进入容器后，运行 YOLO 的训练命令，指定挂载的路径作为数据集路径。

    - ~~~bash
      cd /path/in/container/dataset
      ~~~

    - 运行yolo CLI命令

      ~~~bash
      yolo task=detect mode=train model=yolo11n.yaml data=/workspace/dataset/dataset1/data.yaml epochs=75 imgsz=640 workers=0
      ~~~

    - 对于小模型避免过拟合，epochs应在50-100之间
    
    - 避免内存不足 ，设置数据加载的工作线程数workers= 0

- **查看训练结果**

  - 训练完成后，结果会保存在挂载的输出路径 `/home/user/yolo_output` 中，可以直接在主机上查看。

  - 可以在doker环境中，进入结果输出目录看(借助feh)。

    ~~~bash
    apt update && apt install -y feh
    ~~~

    ~~~bash
    feh /path/to/images/image.jpg
    ~~~

- **导出ONNX模型**

  ~~~bash
  yolo export model=yolov11n.pt format=onnx imgsz=320,224 dynamic=False opset=17 simplify=True name=yolov11n
  ~~~

- ****

  **安装模型转换环境**

  借助算能的https://github.com/sophgo/tpu-mlir，在 docker 环境中安装

  ~~~bash
  docker pull sophgo/tpuc_dev:latest
  ~~~

  本机的`~/data`目录挂载到了容器的`~/data`，实现文件共享

  ~~~bash
  docker run --privileged --name tpu-env -v /home/$USER/data:/home/$USER/data -it sophgo/tpuc_dev
  ~~~

  - 安装 tpu-mlir

    先到[github](https://github.com/sophgo/tpu-mlir/releases)下载 `whl` 文件，放到`~/data`目录下。在容器中执行命令安装

    ~~~bash
    pip install tpu_mlir*.whl 
    ~~~

  - 容器内直接输入`model_transform.py`回车执行会有打印帮助信息就算是安装成功

- **编写转换脚本**

  ~~~bash
  #!/bin/bash
  
  set -e
  
  net_name=best
  input_w=640
  input_h=640
  
  # mean: 0, 0, 0
  # std: 255, 255, 255
  
  # mean
  # 1/std
  
  # mean: 0, 0, 0
  # scale: 0.00392156862745098,0.00392156862745098,0.00392156862745098
  
  mkdir -p workspace
  cd workspace
  
  # convert to mlir
  model_transform.py \
  --model_name ${net_name} \
  --model_def ../${net_name}.onnx \
  --input_shapes [[1,3,${input_h},${input_w}]] \
  --mean "0,0,0" \
  --scale "0.00392156862745098,0.00392156862745098,0.00392156862745098" \
  --keep_aspect_ratio \
  --pixel_format rgb \
  --channel_format nchw \
  --output_names "/model.23/dfl/conv/Conv_output_0,/model.23/Sigmoid_output_0" \
  --test_input ../1ae1aed2-678_1669.jpg \
  --test_result ${net_name}_top_outputs.npz \
  --tolerance 0.99,0.99 \
  --mlir ${net_name}.mlir
  
  # export bf16 model
  #   not use --quant_input, use float32 for easy coding
  model_deploy.py \
  --mlir ${net_name}.mlir \
  --quantize BF16 \
  --processor cv181x \
  --test_input ${net_name}_in_f32.npz \
  --test_reference ${net_name}_top_outputs.npz \
  --model ${net_name}_bf16.cvimodel
  
  echo "Preparing calibration images..."
  # 创建校准图片文件夹
  mkdir -p ../calibration_images
  # 从 train/images 文件夹中随机抽取 200 张图片
  find ../images/train/images -type f -name "*.jpg" | shuf -n 200 | xargs -I {} cp {} ../calibration_images/
  
  echo "calibrate for int8 model"
  # export int8 model
  run_calibration.py ${net_name}.mlir \
  --dataset ../calibration_images \
  --input_num 200 \
  -o ${net_name}_cali_table
  
  echo "convert to int8 model"
  # export int8 model
  #    add --quant_input, use int8 for faster processing in maix.nn.NN.forward_image
  model_deploy.py \
  --mlir ${net_name}.mlir \
  --quantize INT8 \
  --quant_input \
  --calibration_table ${net_name}_cali_table \
  --processor cv181x \
  --test_input ${net_name}_in_f32.npz \
  --test_reference ${net_name}_top_outputs.npz \
  --tolerance 0.9,0.6 \
  --model ${net_name}_int8.cvimodel
  ~~~

  - `output_names` 就是我们前面说到的输出节点的输出名。
  - `mean, scale` 就是训练时使用的预处理方法

  - `test_input` 就是转换时用来测试的图像，根据你的实际情况替换图像。
  - `tolerance` 就是量化前后允许的误差，如果转换模型时报错提示值小于设置的这个值，说明转出来的模型可能相比 onnx  模型误差较大，如果你能够容忍，可以适当调小这个阈值让模型转换通过，不过大多数时候都是因为模型结构导致的，需要优化模型，以及仔细看后处理，把能去除的后处理去除了。
  - `quantize` 即量化的数据类型，在 MaixCAM 上我们一般用 INT8  模型，这里我们虽然也顺便转换了一个 BF16 模型，BF16 模型的好处时精度高，不过运行速率比较慢，能转成 INT8 就推荐先用  INT8,实在不能转换的或者精度要求高速度要求不高的再考虑 BF16。
  - `dataset` 表示用来量化的数据集，也是放在转换脚本同目录下，比如这里是`images`文件夹，里面放图片即可
  -  用`--input_num` 可以指定实际使用图片的数量（小于等于 images 目录下实际的图片)

- **执行转换脚本**

  ~~~bash
  chmod +x convert_yolov11_to_cvimodel.sh && ./convert_yolov11_to_cvimodel.sh
  ~~~

  执行完后可以在`workspace`文件夹下看到有`**_int8.cvimodel` 文件

- **编写`mud`文件**

  修改`model_type`,`labels`

  ~~~bash
  [basic]
  type = cvimodel
  model = best_int8.cvimodel
  
  [extra]
  model_type = yolo11
  input_type = rgb
  mean = 0, 0, 0
  scale = 0.00392156862745098, 0.00392156862745098, 0.00392156862745098
  anchors = 10,13, 16,30, 33,23, 30,61, 62,45, 59,119, 116,90, 156,198, 373,326
  labels = blackBall, blueBall, blueTarget, redBall, redTarget, yellowBall
  ~~~

- 部署模型到maixcam

  对于YOLO11直接调用`MaixPy`或者`MaixCDK`对应的代码加载

  

  

  

  
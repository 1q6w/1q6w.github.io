+++
date = '2025-05-04T23:07:46+08:00'
draft = false
title = 'X Anylabeling start'
categories = ["计算机视觉"]
tags = ["X-Anylabeling 自动标注"]
+++

**1. 安装**

**1.1 从源码进行安装(推荐)**

**1.1.1 先决条件：** 从官网安装[anaconda|miniconda](https://www.anaconda.com/download) 

~~~bash
#  使用 Python 版本 3.10 或更高版本创建新的 conda 环境，然后将其激活。
conda create --name x-anylabeling python=3.10 -y
conda activate x-anylabeling
~~~

**1.1.2 安装  [ONNX 运行时](https://onnxruntime.ai/)**

~~~python
# Install ONNX Runtime CPU
pip install onnxruntime

# Install ONNX Runtime GPU (CUDA 11.x)
pip install onnxruntime-gpu==x.x.x

# Install ONNX Runtime GPU (CUDA 12.x)
pip install onnxruntime-gpu --extra-index-url https://aiinfra.pkgs.visualstudio.com/PublicPackages/_packaging/onnxruntime-cuda-12/pypi/simple/
~~~

**1.2 Git 克隆存储库 **

~~~bash
git clone https://github.com/CVHub520/X-AnyLabeling.git
cd X-AnyLabeling
~~~

****

**1.3  安装 `requirements.txt` 文件**.

| 依赖项文件   |操作系统 |	运行时环境 	|	可编译 	|
| ----------- | ----------- |----------- | ----------- |
| requirements.txt  | Windows/Linux|  CPU	| No |
| requirements-dev.txt   |Windows/Linux       |		CPU	| Yes|
| requirements-gpu.txt | Windows/Linux       |	GPU		|	No |
|requirements-gpu-dev.txt|	 Windows/Linux 	|GPU|Yes|

~~~python
pip install -r requirements-[xxx].txt
~~~

**1.4   启动  **

**1.4.1 完成必要的步骤后，使用以下命令生成资源**

~~~bash
pyrcc5 -o anylabeling/resources/resources.py anylabeling/resources/resources.qrc
~~~

**1.4.2  为避免潜在冲突，请使用以下命令卸载 AnyLabeling 的任何现有安装：**

~~~python
pip uninstall anylabeling -y
~~~

**1.4.3 设置环境变量：**

~~~bash
# linux or macos
export PYTHONPATH=/path/to/X-AnyLabeling
# windows
set PYTHONPATH=C:\path\to\X-AnyLabeling
~~~

**1.4.4 要运行该应用程序，请执行以下命令**

~~~python
python anylabeling/app.py
~~~

**1.4.5 再次启动**

~~~bash
# 激活conda环境
conda activate x-anylabeling
# 进入软件安装目录
cd /path/to/X-AnyLabeling
# 运行应用程序
python anylabeling/app.py
~~~

**2. [用户手册](https://github.com/CVHub520/X-AnyLabeling/blob/main/docs/zh_cn/user_guide.md)**

**3. [自动化标注](https://github.com/CVHub520/X-AnyLabeling/blob/main/docs/en/custom_model.md)**

**3.1 导入导出的  `*.onnx` 文件 [使用 Netron ](https://netron.app/) 在线工具检查输入和输出节点信息，确保尺寸和其他详细信息符合预期**

**3.2 可以浏览  [Model Zoo ](https://github.com/CVHub520/X-AnyLabeling/blob/main/docs/en/model_zoo.md) 文件以查找并复制相应模型的配置文件**

以YOLO11为例

~~~yaml
type: yolo11
name: yolo11s-r20240930
provider: Ultralytics
display_name: YOLO11s
model_path: best.onnx
iou_threshold: 0.45
conf_threshold: 0.25
classes:
  - blackball
  - blueball
  - bluetarget
  - redball
  - redtarget
  - yellowball
~~~

**3.3 将yaml文件和`*.onnx`文件置于`/path/to/xanylabeling_data/models/your_model/`**

**3.4 加载模型进行标注**


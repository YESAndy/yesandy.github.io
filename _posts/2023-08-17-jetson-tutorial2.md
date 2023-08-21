---
title: Jetson Nano project template-camera
author: andy
date: 2023-08-18 11:59:00 +/-0080
categories: [tutorial]
tags: [jetson]     # TAG names should always be lowercase
math: true
toc: true
layout: post
---

## Content
1. Introduction
2. Methodology
3. Data collection
4. Data processing

## Introduction

![Desktop View](/assets/img/post/2023-08-18-trt-pose-demo.gif){: width="480" height="480" }
_Object Detection using YOLOV8 demo_

This tutorial aims to provide a starter project for image data collection for object detection using Jetson Nano and a Web camera. 

There are many other interesting projects from [Jetson Community](https://developer.nvidia.com/embedded/community/jetson-projects?page=1). 

In this tutorial, the goal is to perform real-time object detection using Jetson Nano and a monocular camera. We will utilize the training pipeline from [Ultralytics](https://github.com/ultralytics/ultralytics).

## Code preparation
### Python virtual environment
virtualenv is a virtual environment tool for organizing Python packages for projects. It is very helpful when you have multiple projects with different package dependencies requirements. To install virtualenv, open a terminal and run:

```bash
sudo apt-get install python3-pip
sudo pip3 install virtualenv
sudo apt install python3.8
sudo apt install libpython3.8-dev
```

To create a virtual environment, run:
```bash
cd $YOUR_PROJECT_DIRECTORY
virtualenv your_env_name python=3.8
```

To activate the created virtual environment, run:
```bash
source ./your_env_name/bin/activate
```

and you can see your_env_name is activated:

## Data collection and preprocessing
We will first make sure we can receive images from the web camera using opencv-python package. 
### Camera Installation
Connect the web camera to Jetson Nano through USB port

### Data acquisition
Open a terminal, activate the created virtual environment,
```bash
source ./your_env_name/bin/activate
```

Install dependencies:
```bash
sudo apt update
sudo apt install v4t-utils
pip install opencv-python
```

Open the ".bashrc" file under home directory, and type `export OPENBLAS_CORETYPE=ARMV8` at the end of the file. 

If you can not find ".bashrc", it is because it is a hidden file. To make it appear, follow this [guide](https://www.makeuseof.com/view-hidden-files-and-folders-linux/#:~:text=By%20default%2C%20your%20file%20manager,files%20on%20Linux%20as%20well.)

Then source .bashrc in terminal:
```bash
source ~/.bashrc
```

Now we can test image capture using OpenCV package by the following script:
```Python
import cv2

cam = cv2.VideoCapture(0)
while True:
  check, frame = cam.read()
  cv2.imshow("video", frame)

  key = cv2.waitKey(1)
  if key == 27:
    break
cam.release()
cv2.destroyAllWindows()
```


### Install Ultralytics package and other dependencies

#### 1. Install Ultralytics
Open a terminal, and activate the created virtual environment:
```bash
source ./your_env_name/bin/activate
pip install ultralytics
```

#### 2. Install PyTorch

Download PyTorch wheel file from this [link](https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048).

![Desktop View](/assets/img/post/2023-08-18-download-pytorch.png){: width="480" height="480" }
_Download Pytorch wheel file_

Click on "torch-1.10.0-cp36-cp36m-linux_aarch64.whl" and download it to your project directory.

Set the install path by:
```bash
export TORCH_INSTALL=path/to/torch-1.10.0-cp36-cp36m-linux_aarch64.whl
```

Then install PyTorch by:
```bash
python3 -m pip install --upgrade pip; python3 -m pip install aiohttp numpy=='1.19.4' scipy=='1.5.3' export "LD_LIBRARY_PATH=/usr/lib/llvm-8/:$LD_LIBRARY_PATH"; python3 -m pip install --upgrade protobuf; python3 -m pip install --no-cache $TORCH_INSTALL
```


#### 3. Install other dependencies:
```bash
pip install tqdm cython pycocotools
sudo apt-get install python3-matplotlib
```
#### 4. Model inference





## Reference
- TRT-POSE <https://github.com/NVIDIA-AI-IOT/trt_pose/tree/master>
- Install PyTorch in Jetson <https://docs.nvidia.com/deeplearning/frameworks/install-pytorch-jetson-platform/index.html>
- YOLO-v8 architecture <https://github.com/ultralytics/ultralytics/issues/189>

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
This tutorial aims to provide a starter for data collection using Jetson Nano and a Raspberry Pi camera. The source project is from [Jetson Community](https://developer.nvidia.com/embedded/community/jetson-projects?page=1). 

In this tutorial, the goal is to perform real-time pose estimation using Jetson Nano and a monocular camera. We will use [trt-pose](https://github.com/NVIDIA-AI-IOT/trt_pose/tree/master) as our template.

## Code preparation
### Python virtual environment
virtualenv is a virtual environment tool for organizing Python packages for projects. It is very helpful when you have multiple projects with different package dependencies requirements. To install virtualenv, open a terminal and run:

```bash
sudo apt-get install python3-pip
sudo pip3 install virtualenv
```

To create a virtual environment, run:
```bash
cd $YOUR_PROJECT_DIRECTORY
virtualenv your_env_name python=3.6
```

To activate the created virtual environment, run:
```bash
source ./your_env_name/bin/activate
```

and you can see your_env_name is activated:

### Install trt-pose 

#### 1. Install PyTorch
Open a terminal, and activate the created virtual environment:
```bash
source ./your_env_name/bin/activate
```

Download PyTorch wheel file from this [link](https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048).

![Desktop View](/assets/img/post/2023-08-18-download-pytorch.png){: width="480" height="480" }
_Download Pytorch wheel file_

Click on "torch-1.10.0-cp36-cp36m-linux_aarch64.whl" and download it to your project directory.

Set the install path by:
```bash
export TORCH_INSTALL=path/to/torch-1.10.0-cp36-cp36m-linux_aarch64.whl
```




Install repo dependencies:
```bash
pip install tqdm cython pycocotools
sudo apt-get install python3-matplotlib
```

Install trt-pose:
Download and install trt-pose from GitHub by,

```bash
git clone https://github.com/NVIDIA-AI-IOT/trt_pose.git
cd torch2trt
python setup.py install --plugins
```


## Data collection and preprocessing

reference video tutorial
{% include embed/youtube.html id='dHvb225Pw1s' %}


## Model Training



## Reference
- TRT-POSE <https://github.com/NVIDIA-AI-IOT/trt_pose/tree/master>

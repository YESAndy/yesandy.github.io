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
2. Code preparation
3. Data collection and preprocessing
4. Data Labeling
5. Model training and inference

## Introduction

![Desktop View](/assets/img/post/2023-08-21-yolo-demo.gif){: width="480" height="480" }
_Object Detection using YOLOV8 demo_

This tutorial aims to provide a starter project for image data collection for Hard Hat detection using Jetson Nano and a Web camera. 

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
Connect the web camera to Jetson Nano through USB port.

Open the ".bashrc" file under home directory, and type `export OPENBLAS_CORETYPE=ARMV8` at the end of the file. 

If you can not find ".bashrc", it is because it is a hidden file. To make it appear, follow this [guide](https://www.makeuseof.com/view-hidden-files-and-folders-linux/#:~:text=By%20default%2C%20your%20file%20manager,files%20on%20Linux%20as%20well.)

Then source .bashrc in the terminal:
```bash
source ~/.bashrc
```

### Data acquisition

#### 1. Install dependencies:
Open a terminal, activate the created virtual environment,
```bash
source ./your_env_name/bin/activate
```

```bash
sudo apt update
sudo apt install v4t-utils
pip install opencv-python
```

#### 2. Test video streaming

Now we can test image capture using opencv-python package by the following script:
```python
import cv2

# 0 means /dev/video0, you may adjust this value if you have another camera.
cam = cv2.VideoCapture(0)
while True:
  check, frame = cam.read()
  cv2.imshow("video", frame)

  key = cv2.waitKey(1)
  if key == 27:  # 27 mean ESC in keyboard
    break
cam.release()
cv2.destroyAllWindows()
```

#### 3. Save images using the keyboard

Here we show a simple demo about how to collect and save images.
```python
import cv2

file_root = "/path/to/project/"  # change accordingly
count = 0

# 0 means /dev/video0, you may adjust this value if you have another camera.
cam = cv2.VideoCapture(0)
while True:
  check, frame = cam.read()
  cv2.imshow("video", frame)

  key = cv2.waitKey(1)
  if key == 27:  # 27 mean ESC in keyboard
    break
  elif key == 32:  # 32 means space
    cv2.imwrite(f"{file_root}/{count}.png", frame) # save images

cam.release()
cv2.destroyAllWindows()
```



## Dataset Creation
We will use [Roboflow](https://roboflow.com/), an online image annotation platform, to generate our dataset. 

Sign up for an account and create a project. Then we can start uploading the collected from Jetson Nano. 

![Desktop View](/assets/img/post/2023-08-21-roboflow-upload.png){: width="640" height="480" }
_Upload the collected images_

After uploading the images, assign annotation tasks to your team members and we can now start annotating. 

![Desktop View](/assets/img/post/2023-08-21-roboflow-annotate.png){: width="640" height="540" }
_Annotate the uploaded images_

We can use the "box" to crop an object such as the helmet and create a label for it. 

After the annotation, we can now generate the dataset!

First, split the dataset into training, validation, and testing. Commonly, the ratios are around 7:2:1.

Then, add data preprocessing (e.g., cropping) and augmentation (e.g., blur and rotation). 

Finally, just click on "generate" to generate a version of the dataset. 

![Desktop View](/assets/img/post/2023-08-21-roboflow-generate.png){: width="640" height="540" }
_Generate dataset_


In the "version" section, select an annotation format (e.g., YOLOv8) and click on "Get Snippet".

![Desktop View](/assets/img/post/2023-08-21-roboflow-download.png){: width="640" height="540" }
_Download dataset_

There are two ways to download the dataset. First, we can just download a zip file and extract it to a local folder. Second, we can download it by running a script copied from the snippet like the following:

```python
# This is just a template
!pip install roboflow  # remove this line if you have installed it

from roboflow import Roboflow
rf = Roboflow(api_key="SUWxwWJf88eAaWZdhWnx")
project = rf.workspace("xmagical").project("hard-hat-sample-jebrb")
dataset = project.version(2).download("yolov8")
```

For example, we can download the dataset under "./data" folder. 

![Desktop View](/assets/img/post/2023-08-21-dataset.png){: width="640" height="540" }
_Content of dataset folder_

The "data.yaml" file is a configuration file that sets up the label names and dataset paths. An example of it is the following:

```yaml
train: ../train/images
val: ../valid/images
test: ../test/images

nc: 3
names: ['head', 'helmet', 'person']

roboflow:
  workspace: xmagical
  project: hard-hat-sample-jebrb
  version: 1
  license: Public Domain
  url: https://app.roboflow.com/xmagical/hard-hat-sample-jebrb/1
```

We may have to change the train, val, and test path to an absolute form in the data.yaml, for example:
```yaml
train: ~/Downloads/data//Hard Hat Sample.v1-raw.yolov8/train/images
val: ~/Downloads/data//Hard Hat Sample.v1-raw.yolov8//images
test: ~/Downloads/data//Hard Hat Sample.v1-raw.yolov8//images
```

## Model Training and Inference
We will utilize the training pipeline provided by Ultralytics.

### Install Ultralytics package and other dependencies

#### 1. Install Ultralytics
Open a terminal, and activate the created virtual environment:
```bash
source ./your_env_name/bin/activate
pip install ultralytics
```

You can test if the packages are installed successfully by running the following command:
```bash
yolo predict model=yolov8n.pt source='https://ultralytics.com/images/bus.jpg'
```

And check the results under "./runs/detect/predict".


#### 2. Alternative way to install PyTorch 
> In case PyTorch installation fails.
{: .prompt-info }

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

### Train your model

#### 1. Initiate the training
Thanks to Ultralytics, we can train the object detection model by simply running the following command line:

```bash
yolo train data=~/Downloads/data/data.yaml model=yolov8n.pt epochs=100 lr0=0.01
```

Change the value of "data" argument accordingly. 

Ultralytics provides a series of models with different sizes. "epochs" (i.e., number of epochs to train) and "lr0" (i.e., initial learning rate) are hyperparameters. A list of hyperparameters can be found at [link](https://docs.ultralytics.com/usage/cfg/#train).

#### 2. Track the training progress
After starting the training, the program may ask you to log into [wandb](https://wandb.ai/site) used to track training progress. 

Sign up for an account on wandb website, and go to "User Settings". Find "API key" section and copy the API key:

![Desktop View](/assets/img/post/2023-08-21-wandb-keyapi.png){: width="640" height="540" }
_Get API key from wandb_

Enter the API key in the terminal if required. 

After the training is completed, the model is automatically saved to a local folder. 

### Model inference





## Reference
- TRT-POSE <https://github.com/NVIDIA-AI-IOT/trt_pose/tree/master>
- Install PyTorch in Jetson <https://docs.nvidia.com/deeplearning/frameworks/install-pytorch-jetson-platform/index.html>
- YOLO-v8 architecture <https://github.com/ultralytics/ultralytics/issues/189>
- YOLOV8 demo with Jetson Nano <https://wiki.seeedstudio.com/YOLOv8-DeepStream-TRT-Jetson/>
- 

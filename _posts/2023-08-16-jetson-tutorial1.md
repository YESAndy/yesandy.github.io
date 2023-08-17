---
title: Jetson Nano project tutorial-hardware
author: andy
date: 2023-08-16 15:40:00 +/-0080
categories: [tutorial]
tags: [jetson]     # TAG names should always be lowercase
math: true
toc: true
layout: post
---

## Content
1. Jetson Nano setup
2. A Brief Intro to Linux 
3. Camera setup
4. 

## Jetson Nano Setup
### Overview of Jetson Nano
![Desktop View](/assets/img/post/2023-08-17-jetson-nano.jpg){: width="480" height="480" }
_Jetson Nano 4Gb including 4 USB ports, a wired network port, an HDMI port, an HP port, a CSI camera connector, and a 40-pin port_

"NVIDIA® Jetson Nano™ Developer Kit is a small, powerful computer that lets you run multiple neural 
networks in parallel for applications like image classification, object detection, segmentation, and speech processing. 
All in an easy-to-use platform that runs in as little as 5 watts."

### Install Ubuntu in Jetson Nano
Jetson Nano uses Ubuntu 18.04 system.

There are several ways to boost Jetson Nano with Ubuntu system: 1) __Nvidia SDK manager__, 2) __SD card mirroring__. Both of them require you to have a Ubuntu system as the host computer. So if you do not have a Ubuntu system installed on your computer, follow
1. For Windows <https://www.thomasmaurer.ch/2019/06/how-to-create-an-ubuntu-vm-on-windows-10/>
2. For MacOS <https://medium.com/analytics-vidhya/step-by-step-guide-to-download-and-install-virtual-box-in-macos-7341b6f99827>

Before the boosting, 
1. Insert an empty SD card (at least 32 Gb) in the SD card slot.
2. Make sure your power supply is at least 5V/4A
> Follow this [link](https://itsfoss.com/format-usb-drive-sd-card-ubuntu/) to make sure your SD card is correctly formatted to __Ext4__ format
{: .prompt-info }


### Nvidia SDK manager
Download the SDK manager by the following link:
<https://developer.nvidia.com/sdk-manager>,

![Desktop View](/assets/img/post/2023-08-17-sdk-manager-download.png){: width="480" height="480" }
_Download SDK manager_
click on the ".deb Ubuntu" to download the SDK manager.

> You need to register an account. If you don't know how to register, you can refer to [link](https://www.waveshare.com/wiki/NVIDIA-acess)
{: .prompt-info }





### SD card mirroring

### Usage of the ports 

## A brief intro to Ubuntu

## Reference
Jetson Nano Specs <https://developer.nvidia.com/embedded/jetson-nano-developer-kit>
Jetson Nano Dev Kit Manual <https://www.waveshare.com/wiki/JETSON-NANO-DEV-KIT-MANUAL>
Nvidia SDK manager user guide <https://docs.nvidia.com/sdk-manager/index.html>



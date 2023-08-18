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
2. Camera setup


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
- For Windows <https://www.thomasmaurer.ch/2019/06/how-to-create-an-ubuntu-vm-on-windows-10/>
- For MacOS <https://medium.com/analytics-vidhya/step-by-step-guide-to-download-and-install-virtual-box-in-macos-7341b6f99827>

Before the boosting, 
1. Insert an empty SD card (at least 32 Gb) in the SD card slot.
2. Make sure your power supply is at least 5V/4A

![Desktop View](/assets/img/post/2023-08-17-insert-sd-card.jpg){: width="480" height="480" }
_insert the sd card_

> Follow this [link](https://itsfoss.com/format-usb-drive-sd-card-ubuntu/) to make sure your SD card is correctly formatted to __Ext4__ format
{: .prompt-info }


### Nvidia SDK manager
#### 1. Download the SDK manager by the following link:
<https://developer.nvidia.com/sdk-manager>,

![Desktop View](/assets/img/post/2023-08-17-sdk-manager-download.png){: width="480" height="480" }
_Download SDK manager_
click on the ".deb Ubuntu" to download the SDK manager.

> You need to register an account. If you don't know how to register, you can refer to [link](https://www.waveshare.com/wiki/NVIDIA-acess)
{: .prompt-info }

#### 2. After the download is complete, we enter the download path Downloads to install and input the following content in the [terminal](https://ubuntucommunity.s3.dualstack.us-east-2.amazonaws.com/original/2X/8/85e591c2bdc94b4159329bf19cc1d6740f233b84.png):


```bash
cd Downloads
sudo dpkg -i sdkmanager_1.6.1-8175_amd64.deb
```
> Change the package name "sdkmanager_1.6.1-8175_amd64.deb" according to your downloaded version.
{: .prompt-info }

#### 3. After the installation is complete, the system may report an error that the dependency files cannot be found. Enter the following command to solve this problem.

```bash
sudo apt --fix-broken install
```

#### 4. Now you can find the sdk manager in the application pages in Ubuntu
   
#### 5. Enter Recovery mode
1. Use jumper caps or Dupont wires to short-circuit pin 3 (FC REC) and pin 4 (GND pins), as shown in the figure below, at the bottom of the core board. 
2. Connect the DC power supply to the circular power port and wait a moment.


![Desktop View](/assets/img/post/2023-08-17-enter-recovery-mode.jpg){: width="480" height="480" }
_Connect pin 3 (FC REC) and pin 4 (GND pins) to enter recovery mode_

#### 6. Flash the Ubuntu image into Jetson



### SD card mirroring
#### 1. Install dependencies in host computer
Open a terminal, run the following commands:

```bash
sudo apt-get install qemu-user-static
sudo apt-get install python
```

#### 2. Download Jetpack image
Go to <https://developer.nvidia.com/embedded/linux-tegra-r3251>,
![Desktop View](/assets/img/post/2023-08-17-download-jetpack.png){: width="480" height="480" }
_Download Jetpack image_

Click on "L4T Driver Package (BSP)" and "	Sample Root Filesystem" to download the L4T driver and the sample root system for linux.

#### 3. Compile the image


Open a terminal, create a folder named "32.5.1":
```bash
mkdir 32.5.1&cd 32.5.1
```

Move the downloaded driver package "Tegra186_Linux_R32.5.1_aarch64.tbz2" and the sample root system "Tegra_Linux_Sample-Root-Filesystem_R32.5.1_aarch64.tbz2" into the "32.5.1" folder.

In the same terminal, uncompress the driver package:
```bash
tar -xf Tegra186_Linux_R32.5.1_aarch64.tbz2
```

Then go to the L4T package directory and uncompress the sample root file system:
```bash
cd Linux_for_Tegra/rootfs
sudo tar -xpf ../../ Tegra_Linux_Sample-Root-Filesystem_R32.5.1_aarch64.tbz2
```

Wait until the compile is completed, then go back to the root directory "./Linux_for_Tegra":
```bash
cd ..
```

Finally, install the image by:
```bash
sudo ./apply_binaries.sh
```

#### 4. Flash the image into SD card
Before connecting to the host computer, Jetson Nano should be in recovery mode.

Connect the Micro USB port of the Jetson Nano to the Ubuntu host with a USB cable.

In the same terminal in step 3, flash the image into SD card by:
```bash
sudo ./flash.sh jetson-nano-emmc mmcblk0p1
```

After the compilation is completed, insert the SD card back into Jetson Nano. Now we are ready to boost Jetson Nano!

### Boost Jetson Nano
> Make sure you unplug the short-circuit for the recovery mode.
{: .prompt-warning }

Connect the DC power supply to the circular power port and wait a moment.

Connect a keyboard, a mouse, and a monitor to Jetson Nano.

Follow the guide to complete Ubuntu installation.


## Reference
- Jetson Nano Specs <https://developer.nvidia.com/embedded/jetson-nano-developer-kit>
- Jetson Nano Dev Kit Manual <https://www.waveshare.com/wiki/JETSON-NANO-DEV-KIT-MANUAL>
- Nvidia SDK manager user guide <https://docs.nvidia.com/sdk-manager/index.html>
- Ubuntu tutorials <https://ubuntu.com/tutorials>
- Ubuntu installation guide <https://ubuntu.com/tutorials/install-ubuntu-desktop#9-create-your-login-details>
- The 40-pin port <https://jetsonhacks.com/nvidia-jetson-nano-j41-header-pinout/>



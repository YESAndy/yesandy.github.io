---
title: GIRL demo data collection
author: andy
date: 2023-06-13 16:35:00 +/-0080
categories: [doc]
tags: [girl]     # TAG names should always be lowercase
math: true
toc: true
layout: post
---

## Content
1. record videos 
2. convert bag file to mp4


### Record videos

```bash
# in a new terminal
realsense-viewer

```

turn on the **Stereo module** and **RGB Camera** on the left of the interface

click on the **Record** button to start recording demos.

click on the **stop** button to end recordings.

The saved bag files should be under ``~/Document/Andy``.

> Every object (e.g., bottle) should have its individual folder to store the corresponding demo bag files.
{: .prompt-info }


### Convert bag file to mp4



```bash
# in a new terminal
cd ./Document/Andy/catkin_jsk_ws/
source ./devel/setup.bash
sh ./run_bag2video.sh
```

> Every object (e.g., bottle) should have its individual folder to store the corresponding demo mp4 files.
{: .prompt-info }

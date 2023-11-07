---
title: Jetson Nano project template-environmental sensor
author: andy
date: 2023-10-25 11:59:00 +/-0080
categories: [tutorial]
tags: [jetson]     # TAG names should always be lowercase
math: true
toc: true
mermaid: true
layout: post
---

## Content
1. Introduction
2. Installation of the sensor
3. Code preparation
4. Data collection and preprocessing
5. Model training and inference

## Introduction
In this tutorial, we will install and use an environmental sensor in Jetson Nano. The environmental sensor collects information
including temperature, humidity, air pressure, ambient VOC, IR light intensity, and so on. These data are generated in time scale, 
for which we can put them in a table. For example, 

| Time / Attribute   | Temperature (Celsius degree) | Humidity (RH) |
|:-------------------|:-----------------------------|:--------------|
| 0                  | 10                           | 80.2          |
| 1                  | 9.8                          | 80.4          |
| 2                  | 9.9                          | 80.2          |
| 3                  | 10                           | 80            |

Since they are time-series data, we can use logistic regression to predict the values we want. For example, use temperature to predict humidity. 

## Sensor Installation

Align the 40-pin connectors between Jetson Nano and the sensor:

![Desktop View](/assets/img/post/2023-11-07-install-env-sensor.jpg){: width="480" height="480" }
_Environmental sensor installation_

![Desktop View](/assets/img/post/2023-11-07-env-sensor.png){: width="480" height="480" }
_Environmental sensor installed on Jetson Nano_

## Code Preparation
### Install dependencies
```bash
sudo apt-get install python-smbus
sudo -H apt-get install python-pil
sudo apt-get install i2c-tools
```

### Download the data collection package:
```bash
sudo apt-get install p7zip-full
wget https://files.waveshare.com/upload/f/f5/Environment_sensor_fot_jetson_nano_rev3.zip
7z x Environment_sensor_fot_jetson_nano.7z  -r -o./Environment_sensor_fot_jetson_nano
```

### Test the package
```bash
cd Environment_sensor_fot_jetson_nano
sudo python test.py
```
If everything is done, the screen will visualize the sensor outputs:
![Desktop View](/assets/img/post/2023-11-07-sensor-viz.png){: width="480" height="480" }
_Sensor outputs on the screen_

`test.py` visualizes all sensor outputs. There are other scripts for a single reading of the sensor signal:

| Sensor                          | Script      | Note                              |
|:--------------------------------|:------------|:----------------------------------|
| Ambient light Sensor            | TSL2591.py  |                                   |
| Temperature and Humidity Sensor | BME280.py   |                                   |
| 9-AXIS Sensor                   | ICM20948.py |                                   |
| IR/UV Sensor                    | LTR390.py   |                                   |
| VOC Sensor                      | SGP40.py    | 0 to 1,000 ppm ethanol equivalent |


## Reference
sensor wiki <https://www.waveshare.com/wiki/Environment_Sensor_for_Jetson_Nano#How_to_use>


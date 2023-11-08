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

Take a closer look at the `test.py`:

```Python
#!/usr/bin/python
# -*- coding:utf-8 -*-
import time
import SH1106 #OLED
import MPU9255 #Gyroscope/Acceleration/Magnetometer
import BME280   #Atmospheric Pressure/Temperature and humidity
import LTR390   #UV
import TSL2591  #LIGHT
import SGP40
from PIL import Image,ImageDraw,ImageFont
import math
import os

os.system('i2cdetect -y -r 1')
# time.sleep(1)


bme280 = BME280.BME280()
bme280.get_calib_param()
print("bme280 T&H I2C address:0X76")
light = TSL2591.TSL2591()
print("TSL2591 Light I2C address:0X29")
uv = LTR390.LTR390()
print("UV I2C address:0x60")
sgp = SGP40.SGP40()
print("SGP40 VOC I2C address:0X59")
MPU9255 = MPU9255.MPU9255()
print("MPU9255 9-DOF I2C address:0X68")
oled = SH1106.SH1106()
print("OLED I2C address:0x3c")

try:
	
	print("Comprehensive test program...")
	print("please Enter ctrl+c to end program")
	image = Image.new('1', (oled.width, oled.height), "BLACK")
	draw = ImageDraw.Draw(image)
	
	x = 0
	font = ImageFont.truetype('Font.ttc', 10)
	while True:
		x = x + 1
		time.sleep(0.2)
		if(x < 20):
			bme = []
			bme = bme280.readData()
			pressure = round(bme[0], 2) 
			temp = round(bme[1], 2) 
			hum = round(bme[2], 2)
			
			lux = round(light.Lux(), 2)
			
			# uv = round(uv.UVS(), 2) 
			uvdata = uv.UVS()
			# ir = round(uv.readdata()[1], 2)
            
			gas = round(sgp.raw(), 2)
			
			draw.rectangle((0, 0, 128, 64), fill = 0)
			
			draw.text((0, 0), str(pressure), font = font, fill = 1)
			draw.text((40, 0), 'hPa', font = font, fill = 1)
			draw.text((0, 15), str(temp), font = font, fill = 1)
			draw.text((40, 15), 'C', font = font, fill = 1)
			draw.text((0, 30), str(hum), font = font, fill = 1)
			draw.text((40, 30), '%RH', font = font, fill = 1)
			
			draw.text((0, 45), str(lux), font = font, fill = 1)
			draw.text((40, 45), 'Lux', font = font, fill = 1)
			
			draw.text((65, 0), str(uvdata), font = font, fill = 1)
			draw.text((105, 0), 'UV', font = font, fill = 1)
			# draw.text((65, 15), str(ir), font = font, fill = 1)
			# draw.text((105, 15), 'IR', font = font, fill = 1)
			
			draw.text((65, 30), str(gas), font = font, fill = 1)
			draw.text((105, 30), 'GAS', font = font, fill = 1)
            
			oled.display(image)
		elif(x<40):
			icm = []
			icm = MPU9255.getdata()
			# roll = round(icm[0], 2)
			# pitch = round(icm[1], 2)
			# yaw = round(icm[2], 2)
			
			draw.rectangle((0, 0, 128, 64), fill = 0)
			font8 = ImageFont.truetype('Font.ttc', 9)
			
			draw.text((0, 0), 'RPY', font = font8, fill = 1)
			# draw.text((20, 0), str(roll), font = font8, fill = 1)
			# draw.text((50, 0), str(pitch), font = font8, fill = 1)
			# draw.text((90, 0), str(yaw), font = font8, fill = 1)
			
			draw.text((0, 15), 'Acc', font = font8, fill = 1)
			draw.text((20, 15), str(icm[0]), font = font8, fill = 1)
			draw.text((50, 15), str(icm[1]), font = font8, fill = 1)
			draw.text((90, 15), str(icm[2]), font = font8, fill = 1)

			draw.text((0, 30), 'Gyr', font = font8, fill = 1)
			draw.text((20, 30), str(icm[3]), font = font8, fill = 1)
			draw.text((50, 30), str(icm[4]), font = font8, fill = 1)
			draw.text((90, 30), str(icm[5]), font = font8, fill = 1)

			draw.text((0, 45), 'Mag', font = font8, fill = 1)
			draw.text((20, 45), str(icm[6]), font = font8, fill = 1)
			draw.text((50, 45), str(icm[7]), font = font8, fill = 1)
			draw.text((90, 45), str(icm[8]), font = font8, fill = 1)
			
			oled.display(image)
		elif(x >= 40):
			x = 0

except KeyboardInterrupt:
	image2 = Image.new('1', (oled.width, oled.height), "BLACK")
	oled.display(image2)
	exit()	

```


## Reference
sensor wiki <https://www.waveshare.com/wiki/Environment_Sensor_for_Jetson_Nano#How_to_use>


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


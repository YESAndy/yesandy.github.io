---
title: Columbia Ginsburg HPC Doc
author: andy
date: 2024-10-17 14:20:00 +/-0080
categories: [doc]
tags: [hpc]     # TAG names should always be lowercase
math: true
toc: true
layout: post
---

## Content
1. login
2. use conda to install packages

## login and initiate modules

Refer to the official [documentation](https://columbiauniversity.atlassian.net/wiki/spaces/rcs/pages/62141877/Ginsburg+HPC+Cluster+User+Documentation)

significant modules that needs to be loaded

- module load anaconda/3-2023.09
- module load cuda12.0/toolkit
- module load gcc/10.2.0
- module load cudnn8.0-cuda11.1

> Check the version of the loaded modules
{: .prompt-info }


## common issues

### CUDA
'''console
/usr/lib/ld or /compiler_compat/ld: cannot find -lcuda: No such file or directory
'''

This is because the system cannnot find libcuda.so which is normally located under '/cm/shared/apps/cuda11.1/toolkit/11.7.1/lib64/stubs/'. 

It can be fixed by adding this to LIBRARY_PATH:

'''bash
export LIBRARY_PATH=/cm/shared/apps/cuda11.1/toolkit/11.7.1/lib64/stubs/:$LIBRARY_PATH
'''

---
title: UBC ARC Sockeye Documentation
author: andy
date: 2024-01-24 17:23:00 +/-0080
categories: [doc]
tags: [hpc]     # TAG names should always be lowercase
math: true
toc: true
layout: post
---

## Introduction 

## Install and load software

See the available software:
```bash
module avail
```

Load software:
```bash
module load gcc
module load cuda
module load 

```

## Run a GPU job
```bash
#!/bin/bash
 
#SBATCH --job-name=jobname            
#SBATCH --account=alloc-code-gpu    
#SBATCH --nodes=1                  
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=12                           
#SBATCH --mem=64G                  
#SBATCH --time=72:00:00             
#SBATCH --gpus-per-node=2
#SBATCH --output=output.txt         
#SBATCH --error=error.txt          
#SBATCH --mail-user=your.name@ubc.ca
#SBATCH --mail-type=ALL                               
 
module load gcc
module load cuda
module load <software_package_1>
module load <software_package_2>
 
cd $SLURM_SUBMIT_DIR
 
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
<gpu_executable>

```

## Reference
- Official Documentation <https://confluence.it.ubc.ca/display/UARC/GPU+Jobs+in+Detail>


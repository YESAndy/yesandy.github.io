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


## Use vscode to login

- Open a vscode interface, press `Ctrl+Shift+P` (`Command+Shift+P` for Mac) to open the search bar.
- In the search bar, search `remote ssh`, then click on `new SSH connection`
- Type `<cwl-account>@sockeye.ubc.ca` and save the ssh config
- Connect to sockeye with your CWL password and DUO verification.


## Install and load software

See the available software:

```bash
module avail
```

Load software:

```bash
module load gcc
module load cuda
module load miniconda3
```

## Create your conda environment
In a terminal,

```bash
module load miniconda3
module load git
source ~/.bashrc
```

Note that some packages may require git, so also load git

Create a conda environment from yml file:

```bash
conda env create -f environment.yml
```

It is the first time to activate conda env under your account,

```bash
conda init bash
```

Then open a new terminal,
```bash
conda activate <your conda env>
```

## Run a GPU job

Create a bash file called run_job.sh:

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

<gpu_executable>

```

Submit run_job.sh by:

```bash
sbatch run_job.sh
```

You can check the running or pending jobs by running this command:
```bash
squeue -u <your_CWL> -t RUNNING
squeue -u <your_CWL> -t PENDING
```

## Data Transfer
To download the data from sockeye, use the following command:

```bash
scp -r cwl@sockeye.arc.ubc.ca:path local_pat
```



## Reference
- Official Documentation <https://confluence.it.ubc.ca/display/UARC/GPU+Jobs+in+Detail>


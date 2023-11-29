<h1>CYENS GPU Cluster Documentation</h1>

<h2>Overview</h2>

This document serves as a comprehensive guide to understanding and utilizing the GPU cluster available at CYENS. 

<details>
    <summary>
<h2>Cluster Specification</h2>        
    </summary>

<h3>Hardware Configuration</h3>

* **Head Node**
  * **Chassis:** GIGABYTE R182-Z90-00
  * **Motherboard:** GIGABYTE MZ92-FS0-00
  * **CPU:** 2x AMD EPYC 7313, 16C/32T
  * **RAM:** 16x 32GB Samsung M393A4K40EB3-CWE - total 512GB
  * **Storage:** 2x 1.92TB Intel SSDSC2KB019T8 (/trinity/home - 400G)
* **Compute Nodes**
  * **Number of Compute Nodes:** 8 
  * **Nodelist:** ```gpu[01-08]```
  * **Chassis:** Supermicro AS -4124GS-TNR
  * **Motherboard:** Supermicro H12DSG-O-CPU
  * **CPU:** 2x AMD EPYC 7313, 16C/32T 
  * **GPU:** 8x NVIDIA A5000, 24GB, 8192 CUDA cores, 256 Tensor Cores, 27.8 TFLOPS FP32 
  * **RAM:** 16x 32GB SK Hynix HMAA4GR7AJR8N-XN - total 512GB
  * **Storage:** 1x 1TB Samsung SSD 980
* **Storage Nodes:**
  * **Number of Storage Nodes:** 2 
  * **Chassis:** Supermicro Super Server
  * **Motherboard:** Supermicro H12SSL-i 
  * **CPU:** 1x  AMD EPYC 7302P, 16C/32T 
  * **RAM:** 8x 16GB Samsung M393A2K40DB3-CWE - total 256GB 
  * **Storage:**
    * 2x 240GB Intel SSDSC2KB240G7 
    * 24x 7.68TB Samsung MZILT7T6HALA/007 (/lustreFS - 305TB)

<h3>Operating System and Software Environment</h3>

* **Operating System:** <a href=https://rockylinux.org/>Rocky Linux 8.5 (Green Obsidian)</a>
* **Linux kernel version:** 4.18.0-348.23.1.el8_5.x86_64
* **Resource Management System:** <a href=https://slurm.schedmd.com/documentation.html>Slurm</a>
* **Environment Module System:** <a href=https://lmod.readthedocs.io/en/latest/>Lmod</a>

</details>

<details>
  <summary>
<h2>Submitting Jobs</h2>
  </summary>

<h3>Introduction to Slurm: The Job Scheduler</h3>

Slurm is the job scheduler we use. Here we will go into depth about some elements of the scheduler. There are many more 
features of Slurm that go beyond the scope of this guide, but all that you as a user need to know should be available.

The compute nodes are under a single slurm partition, called ```defq```. By using ```sinfo```
you can get the following info:
```bash
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
defq*        up   infinite      1  idle~ gpu05
defq*        up   infinite      2    mix gpu[01,08]
defq*        up   infinite      5  alloc gpu[02-04,06-07]
```

where you will see the current state of each compute node. If you want to check the current queue of jobs
you can use the ```squeue``` command. If you add the  ```-u $USER``` argument you get a list of your current jobs.
If you want to sumbit a job in the cluster used the following two methods. **NEVER EVER RUNS JOBS DIRECTLY ON THE
LOGIN/HEAD NODE.**

<h3>Batch Jobs</h3>

In order to submit a batch job you can use the ```sbatch``` command. ```sbatch``` is a non-blocking command, meaning 
there is not a circumstance where running the command will cause it to hold. Even if the resources requested are not 
available, the job will be thrown into the queue and will start to run once resources become available.

```sbatch``` is based around running a single file. That being said, you shouldn’t need to specify any parameters in 
the command other than ```sbatch <batch file>```, because you can specify all parameters in the command inside the file itself.

The following is an example of a batch script. Please note that the top of the script must start with ```#!/bin/bash```, 
and then immediately follow with ```#SBATCH <param> parameters```. An example of common SBATCH parameters and a 
simple script is below. 

```bash
#SBATCH -o res_%j.txt      # output file
#SBATCH -e res_%j.err      # File to which STDERR will be written
#SBATCH -J <job-name>      #
#SBATCH --partition=defq   # Partition to submit to
#SBATCH --ntasks=1         # Number of tasks
#SBATCH --cpus-per-task=2  # Number of cores per task
#SBATCH --gres=gpu:1       # Number of GPUs
#SBATCH --mem=50000        # Memory in MB
#SBATCH --time=3-00:00     # Maximum runtime in D-HH:MM

python ...
```

This script will allocate 2 CPUs, 1 GPU and 50,000MB of RAM in the defq partition for up to 3 days.

<h3>Interactive Jobs</h3>

You can use the ```srun``` command in order to run interactive jobs. ```srun``` is a blocking command and it will not let
you execute other commands until this command (job) is finished. You can create an interactive job by using the same arguments
as in a batch script (see the following example):

```bash
srun -c 1 -n 1 -p defq --mem=100 --gres=gpu:0 -t 01:00 --pty /bin/bash
```

</details>


<details>
  <summary>
<h2>Storage</h2>
  </summary>

Below is a table of all available storage.

|            Mountpoint             |       Name       |Type| User Quota |        Group Quota         |                                                                                                                                                                                   Description                                                                                                                                                                                    |
|:---------------------------------:|:----------------:|---:|:----------:|:--------------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
|  ```/trinity/home/<user-name>```  | Home directories |SSD|    20GB    |             -              |                                                                                                                                   Home directories should be used only for user init files. You can check your quota by using ```quota -us```                                                                                                                                    |
| ```/lustreFS/data/<group-name>``` | Work directories |SSD|     -      | 30TB (or 20,971,520 files) | Should be used as the primary location for running cluster jobs. Moreover, you can setup your conda installation under this directory. It's a good practise to create a new subfolder where you will store all of your data, code, etc. This is a shared folder for all users in the group. You can check the group's quota by using ```lfs quota -gh <group-name> /lustreFS/``` |

</details>

<details>
  <summary>
<h2>Module Usage</h2>
  </summary>

The process for using environment modules is convenient and simple. You can ```load``` and ```unload``` them as you 
please, enabling and disabling different software. You can list currently active modules with ```module list```, search 
for modules with ```module avail```, and unload all active modules with ```module purge```. The following guide outlines 
each of these processes.

<h3>List All Available Modules</h3>

To list all available modules, use any of the four commands listed below:

```bash
module available
module avail
module av
ml av
```

<h3>Search for modules</h3>

To filter the output of ```module avail``` for just the ```gcc``` modules, use the following command:

```bash
module avail gcc
```

<h3>Load modules</h3>

To load modules, use the following command:

```bash
module load GCC/10.3.0
```

<h3>Unload modules</h3>

To unload modules, use the following command:

```bash
module unload GCC/10.3.0
```

<h3>Unload all modules</h3>

To unload _all_ modules, use the following command:

```bash
module purge
```

<h3>List currently loaded modules</h3>
To list the modules that are currently loaded, use the following command:

```bash
module list
```

</details>
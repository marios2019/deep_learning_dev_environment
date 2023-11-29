<h1>CYENS GPU Cluster Documentation</h1>

<h2>Overview</h2>

This document serves as a comprehensive guide to understanding and utilizing the GPU cluster available at CYENS. 

<details>
    <summary>
<h3>Cluster Specification</h3>        
    </summary>
<details>
  <summary>
<h4>Hardware Configuration</h4>
  </summary>

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
</details>

<details>
<summary>
<h4>Operating System and Software Environment</h4>
</summary>

* **Operating System:** <a href=https://rockylinux.org/>Rocky Linux 8.5 (Green Obsidian)</a>
* **Linux kernel version:** 4.18.0-348.23.1.el8_5.x86_64
* **Resource Management System:** <a href=https://slurm.schedmd.com/documentation.html>Slurm</a>
* **Environment Module System:** <a href=https://lmod.readthedocs.io/en/latest/>Lmod</a>
</details>

</details>


<h1>CYENS GPU Cluster Documentation</h1>

<h2>Overview</h2>

This document serves as a comprehensive guide to understanding and utilizing the GPU cluster available at CYENS. 

<details open>
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

<details open>
  <summary>
<h2>Accessing the Cluster</h2>
  </summary>

In order to connect to the CYENS cluster you will need to use an **SSH connection**. SSH stands for “_secure shell_”. 
A shell is the terminal, or Command Line Interface (CLI), that you type commands into. The most common shell in Linux is **bash**, 
which is most likely what you will be using in the CYENS cluster.

<h3>Generate SSH Keys</h3>

The authentication method we use for SSH connections is with public/private RSA keys. There is a public SSH key that is 
stored on the cluster, and a private SSH key that you keep on your local computer. In order to login to the CYENS cluster, 
you need to authenticate the public key with your private key.

Use the following instructions to generate and save your public key on the cluster (the following instru:

1. Open your terminal and run the following command to create an ssh-key pair:

```bash
ssh-keygen
```

2. Follow the on-screen instructions and if successful, the private (```id_rsa```) and public key (```id_rsa.pub```) will be created under the ```$HOME/.ssh```
directory. 
3. Forward the public key to your MRG leader so he/she can request a cluster account for you.
4. For Linux or Mac users use the following command to change the permissions of the private key due to its importance to security:

```bash
chmod 600 ~/.ssh/id_rsa
```

5. **Optional:** It's important that you use a dedicated ssh-key pair for accessing the CYENS cluster. It is also recommended
to add a passphrase to your key:

```bash
ssh-keygen -p -f ~/.ssh/id_rsa
```

<h3>Connect to the cluster using SSH connection</h3>
The following instructions will show you how to connect to the cluster using the SSH keys that you generated and stored, through your terminal:

1. If the file  ```~/.ssh/config``` doesn't exist, creat it.
2. Using a text editor, copy the following contents to that file:

```bash
Host cyens_cluster
  Hostname 82.116.197.12
  User <user-name>
  IdentityFile <path-to-private-key>
```

3. Save the file **without an extension**
4. Type the command ```ssh cyens_cluster``` and you should be able to connect to the cluster.

<h3>Connecting Desktop VS Code</h3>

If you prefer to use <a href="https://code.visualstudio.com/">**Visual Studio Code**</a> (VS Code) as your editor, you can 
connect VS Code to the cluster. The following instructions will guide you through how to connect the VS Code Desktop App to the cluster.

<h4>Configure SSH</h4>

To connect your local VS Code to the cluster using the <a href="https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh">**Remote-SSH**</a>
feature, you must configure your ssh client to be able to hop through the login node to a compute node. To configure your ssh client, 
add the following lines to your local ```~/.ssh/config``` file.

```bash
Host cyens_cluster
  Hostname 82.116.197.12
  User <user-name>
  IdentityFile <path-to-private-key>

Host *.cluster
  User <user-name>
  IdentityFile <path-to-private-key>
  ProxyJump cyens_cluster
```

<h4>Start a user SSHD process</h4>

Connect to the CYENS cluster and create a new pair of ssh keys (on the head node):

```bash
ssh-keygen -t rsa -f .ssh/cluster_user_sshd
```

Then create the following ```sshd.sh``` bash script:

```bash
#!/bin/bash
#SBATCH -o res_%j.txt      # output file
#SBATCH -e res_%j.err      # File to which STDERR will be written
#SBATCH -J sshd            # Job name
#SBATCH --partition=defq   # Partition to submit to
#SBATCH --ntasks=1         # Number of tasks
#SBATCH --cpus-per-task=2  # Number of cores per task
#SBATCH --gres=gpu:1       # Number of GPUs
#SBATCH --mem=1000         # Memory in MB
#SBATCH --time=0-04:00     # Maximum runtime in D-HH:MM

PORT=$(python -c 'import socket; s=socket.socket(); s.bind(("", 0)); print(s.getsockname()[1]); s.close()')

echo "********************************************************************"
echo "Starting sshd in Slurm as user"
echo "Environment information:"
echo "Date:" $(date)
echo "Allocated node:" $(hostname)
echo "Path:" $(pwd)
echo "Listening on:" $PORT
echo "********************************************************************"

/usr/sbin/sshd -D -p ${PORT} -f /dev/null -h ${HOME}/.ssh/cluster_user_sshd
```

and submit a batch job based on this, e.g., ```sbatch sshd.sh```. This will create a batch job
with 2 CPUs, 1 GPU, 1GB of RAM and will run for up to 4 hours. The user-defined ```sshd``` process will
accept ```ssh``` connections to the port ```$PORT``` of the allocated compute node ```$(hostname)```. This information can
be found in the corresponding ```res_<job-id>.txt``` log file. 

At this point, you will be able to connect via ssh to the batch job:

```bash
ssh -p <port-where-the-sshd-process-started> <allocated-node> # gpu<01-08>.cluster
```

Notice that the ```ssh``` session can only see the resources allocated to the job. It is important to only allocate resources
that you will actually use and for a reasonable amount of time, usually up to 12 hours.

<h4>Connect VS Code</h4>

Once the ```sshd``` process is set up through a batch job, you can connect VS Code to the cluster. To do so, select 
"_Remote-SSH: Connect to host_" from the command pallette and type in the allocated hostname and port, e.g., ```ssh -p 6000 gpu01.cluster```.
VS Code will update your ```config``` file automatically as follows:

```bash
Host gpu01.cluster
    HostName gpu01.cluster
    Port 6000
```

<h4>Remember to end the SSHD process</h4>

It is important to cancel the Slurm job when we don’t need the ```sshd``` process listening anymore. Moreover, make sure
to close the connection to the remote host from VS Code and remove the additional entries in the ```config``` file, added
by VS Code.
</details>

<details open>
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
If you want to sumbit a job in the cluster used the following two methods. **NEVER EVER RUN JOBS DIRECTLY ON THE
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


<details open>
  <summary>
<h2>Storage</h2>
  </summary>

Below is a table of all available storage.

|            Mountpoint             |       Name       |Type| User Quota |        Group Quota         |                                                                                                                                                                                   Description                                                                                                                                                                                    |
|:---------------------------------:|:----------------:|---:|:----------:|:--------------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
|  ```/trinity/home/<user-name>```  | Home directories |SSD|    20GB    |             -              |                                                                                                                                   Home directories should be used only for user init files. You can check your quota by using ```quota -us```                                                                                                                                    |
| ```/lustreFS/data/<group-name>``` | Work directories |SSD|     -      | 30TB (or 20,971,520 files) | Should be used as the primary location for running cluster jobs. Moreover, you can setup your conda installation under this directory. It's a good practise to create a new subfolder where you will store all of your data, code, etc. This is a shared folder for all users in the group. You can check the group's quota by using ```lfs quota -gh <group-name> /lustreFS/``` |

</details>

<details open>
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


<details open>
  <summary>
<h2>Installing 3rd party libraries</h2>
  </summary>

<details open>
  <summary>
    <h3><a href="https://github.com/NVIDIA/MinkowskiEngine">MinkowskiEngine</a></h3>
 </summary>

The Minkowski Engine is an auto-differentiation library for sparse tensors. It supports all standard neural network 
layers such as convolution, pooling, unpooling, and broadcasting operations for sparse tensors. For more information, 
please visit the <a href="https://nvidia.github.io/MinkowskiEngine/overview.html">documentation page</a>.

<h4>Installation on cluster using Conda and CUDA 11.3</h4>

1. First create the following conda environment and install the necessary python libraries:

```bash
conda create -n py3-mink python=3.8
conda activate py3-mink

conda install openblas-devel -c anaconda
conda install pytorch==1.10.1 torchvision==0.11.2 torchaudio==0.10.1 cudatoolkit=11.3 -c pytorch -c conda-forge
```

2. Load the ```CUDA/11.3.1``` and ```gnu9``` module:

```bash
module load CUDA/11.3.1 gnu9
```

3. Create the following interactive job:

```bash
srun -n 1 -c 4 --gres=gpu:1 --mem=20000 --pty /bin/bash
```

4. Activate again the ```py3-mink``` conda environment and install the latest MinkowskiEngine as follows:

```bash
conda activate py3-mink
pip install -U git+https://github.com/NVIDIA/MinkowskiEngine -v --no-deps --install-option="--blas_include_dirs=${CONDA_PREFIX}/include" --install-option="--blas=openblas"
```

</details>

<details open>
  <summary>
    <h3><a href="https://github.com/CGuangyan-BIT/PointGPT">PointGPT</a></h3>
 </summary>

PointGPT is a novel approach that extends the concept of GPT to point clouds, utilizing a point cloud auto-regressive 
generation task for pre-training transformer models. For more information, please refer to the [arXiv preprint](https://arxiv.org/pdf/2305.11487.pdf).

<h4>Installation on cluster using Conda and CUDA 11.3</h4>

1. First create the following conda environment and install the necessary python libraries:

```bash
conda create -n pointgpt python=3.8
conda activate pointgpt

conda install pytorch==1.10.1 torchvision==0.11.2 torchaudio==0.10.1 cudatoolkit=11.3 tensorboard -c pytorch -c conda-forge
pip install easydict h5py matplotlib open3d opencv-python pyyaml timm tqdm transforms3d termcolor scipy ninja plyfile numpy==1.23.4
pip install setuptools==59.5.0
```

2. Load the ```CUDA/11.3.1``` module:

```bash
module load CUDA/11.3.1
```

3. Clone the PointGPT GitHub repository:

```bash
git clone https://github.com/CGuangyan-BIT/PointGPT.git
cd PointGPT
```

4. Create the following interactive job:

```bash
srun -n 1 -c 4 --gres=gpu:1 --mem=20000 --pty /bin/bash
```

5. Activate again the ```pointgpt``` conda environment and install the following extensions:

```bash
conda activate pointgpt
# Chamfer Distance & emd
cd ./extensions/chamfer_dist
python setup.py install --user
cd ../emd
python setup.py install --user
cd ../
# PointNet++
pip install "git+https://github.com/erikwijmans/Pointnet2_PyTorch.git#egg=pointnet2_ops&subdirectory=pointnet2_ops_lib"
# GPU kNN
pip install --upgrade https://github.com/unlimblue/KNN_CUDA/releases/download/0.2/KNN_CUDA-0.2-py3-none-any.whl
cd ../
```
</details>

</details>

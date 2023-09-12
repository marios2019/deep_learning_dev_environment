<h2>Objective</h2>

Provide a comprehensive guide for installing a Linux distro alongside Windows (dual boot) and setting up all necessary Nvidia drivers/libaries. It also follows a virtual enviroment paradigm for handling Python packages and creating environments with different independecies like different Python, CUDA, PyTorch/TensorFlow/NumPy/SciPy/matplotlib etc. versions. 

<details>
    <summary>
<h2>Environment Info</h2>        
    </summary>
This guide was tested on the following environment:

* PC
    * Laptop: HP Omen 16-b1007nv
    * CPU: Intel i7-12700H
    * GPU: NVIDIA GeForce RTX 3070 Ti Laptop GPU
    * RAM: 32 GB
    * Disk: 1 TB
    * OS: Windows 11 Business (21H2), Ubuntu Desktop 22.04.3 LTS

</details>

<details>
    <summary>
<h2>Dual-boot Windows 11 and Ubuntu 22.04.3 LTS on your PC</h2>
    </summary>

<details>
<summary>
  <h3>Preliminary optional step: Disable Secure Boot</h3>
</summary>
    
While UEFI Secure Boot is supported by Ubuntu, some issues still may rise like not be able to install some 3rd party kernel modules that are not signed. To disable Secure Boot, <b>first make sure to find your BitLocker recovery key</b>. Then follow these steps:
<ol>
  <li>Open the Start menu and open the power menu in the bottom right corner. Then, hold <b>Shift</b> on your keyboard and click <b>Restart</b>. After the PC restarts you will be taken to a blue screen with the following options:</li>
  <ul>
    <li>Continue</li>
    <li>Use a device</li>
    <li>Troubleshoot</li>
    <li>Turn off your PC</li>
  </ul>
  <li>Here, choose <b>Troubleshoot</b>, followed by <b>Advanced options</b>.</li>
  <li>Select <b>UEFI Firmware Settings</b> and then <b>Restart</b>.</li>
  <li>This will take you to your PC's BIOS settings, where you can turn off Secure Boot. Every BIOS is a little different, so you may have to look around. In the HP Omen laptop that we are using, navigate to the <b>BIOS Setup</b> and then from the upper left main menu select <b>Boot Options</b>. There, you can disable <b>Secure Boot</b> by moving the slider to the left. Lastly, click <b>Exit</b> and make sure that the <b>Save Changes and Exit</b> option is selected before choosing <b>Yes</b>.</li>
</ol>
</details>

<details>
<summary>
    <h3>Step 1: Create bootable USB drive</h3>
</summary>
For this step you'll need to have a USB drive that you can install Ubuntu from. To turn your flash drive into installation media, everything on it will be erased, so make sure you've backed up anything you might need.

<ol>
    <li>First download the latest version of Ubuntu Desktop LTS (Long-term Support) from the official <a href=https://ubuntu.com/download/desktop>website</a>.</li>
    <li>Next, you'll need a tool that creates bootable USB drives from ISO files. I am using <a href=https://rufus.ie/en/>Rufus</a> for this purpose.</li>
    <li>Run Rufus and insert the flash drive you want to use as installation media. Then, click <b>Select</b> and choose the ISO file you downloaded. All the options will be filled in automatically.</li>
    <li>Click <b>Start</b>, then click <b>OK</b> in the prompt that shows up. </li>
</ol>
</details>

<details>
<summary>
    <h3>Step 2: Creating a partition to dual-boot Linux</h3>
</summary>

Next, you'll need to create a second partition on your drive for Linux.
<ol>
    <li>Right-click your Start menu icon (or press <b>Windows key + X</b> on your keyboard) and choose <b>Disk Management</b>. You'll see a list of your drives and partitions.</li>
    <li>Right-click your primary partition (the one labeled as Windows C:) and choose <b>Shrink Volume...</b></li>
    <li>Specify the amount of space you want to remove from the partition. This will be limited by the files you already have stored on it, and the amount you enter will be the space you have for your Linux installation. In my case, I decided to allocate 500 GB for the Linux partition so I set the amount of space to shring in MB to 512000.</li>
    <li>The space you chose will be deducted from your partition, and it will be listed as unallocated space in the Disk Management window. You can leave it as is and close the window.</li>
</ol>
</details>

<details>
<summary>
    <h3>Step 3: Installing Ubuntu to dual-boot with Windows 11</h3>
</summary>

Now you're ready to install Linux on your empty partition. If you removed the USB installation media from your PC, insert it again (remove other flash drives), then follow these steps:
<ol>
    <li>Open the Start menu and then click the power button and — while holding <b>Shift</b> on your keyboard — click <b>Restart</b>.</li>
    <li>Click <b>Use a device</b>, then choose the USB flash drive you have inserted and your PC will boot from it.</li>
    <li>You'll now be in the Ubuntu boot menu. Press <b>Enter</b> to boot into Ubuntu.</li>
    <li>Ubuntu and other Linux operating systems let you try it out by booting from the USB drive without installing it. To install Linux on your empty partition, click <b>Install Ubuntu</b>.</li>
    <li>Follow the setup experience by first choosing your keyboard layout.</li>
    <li>At the <b>Updates and other software section</b> choose to <b>Download updates while isntalling Ubuntu</b> and to <b>Install third-party software for graphics and Wi-Fi hardware and additional media formats</b>, where you will need to set a password for Secure Boot.</li>
    <li>Next, at the <b>Installation type</b> select <b>Something else</b> and click <b>Continue</b>. At the next window select the unallocated space that you have created for install Linux (it should be named as <b>free space</b>). Hit <b>Right Click</b> and select <b>add</b> to create the primary partition for Ubuntu. Set the partition's size (I set it to 526871 MB) and the mount point (in my case I set to `/`) and then click <b>OK</b>. The newly created partition should be listed now. Repeat the process by using the rest of the free space partition (10000 MB) in order to create a <b>swap</b> partition. Finally, click <b>Install Now</b> to continue with installation process.</li>
    <li>Next you will select your timezone and enter your name and computer's name along with a login password.</li>
    <li>Proceed with Ubuntu installation.</li>
</ol>
</details>

</details>

<details>
    <summary>
        <h2>Install NVIDIA GPU drivers</h2>
    </summary>

1. Open the command terminal and search for NVIDIA drivers using the `apt` command:

```bash
apt-cache search 'nvidia-driver-' | grep '^nvidia-driver-[[:digit:]]*' | sort -k 3 -t '-'
```

2. Next, use the following two commands as you must apply all pending security updates:

```bash
sudo apt update
sudo apt upgrade
```

3. Now, lets install the latest NVIDIA driver (535 as of September 12th, 2023):

```bash
sudo apt install nvidia-driver-535 nvidia-dkms-535
```

4. Then, reboot your machine:

```bash
sudo reboot
```

5. To verify the NVIDIA drivers installation you can use the [NVIDIA System Management Interface (SMI)](https://developer.nvidia.com/nvidia-system-management-interface)

```bash
nvidia-smi
```

You can also check the version of the installed drivers:
```bash
nvidia-smi --query-gpu=driver_version --format=csv
```

</details>

<details>
    <summary>
        <h2>Setup Conda</h2>
    </summary>

<a href=https://docs.conda.io/en/latest/>Conda</a> is an open source package management system and environment management system that runs on Windows, macOS, and Linux. For this guide will be using <a href=https://docs.conda.io/projects/miniconda/en/latest/#>Miniconda</a> which is a free minimal installer for conda. For installing Miniconda follow the next steps:

1. Open a command terminal and execute these four commands to quickly and quietly install the latest 64-bit version of the installer and then clean up afterwards. To install a different version or architecture of Miniconda for Linux, change the name of the `.sh` installer in the `wget` command.
        
```bash
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm -rf ~/miniconda3/miniconda.sh
```
2. After installing, initialize your newly-installed Miniconda. The following commands initialize for bash and zsh shells:

```bash
~/miniconda3/bin/conda init bash
~/miniconda3/bin/conda init zsh
```
</details>

<details>
    <summary>
        <h2>Setup PyTorch/TensorFlow</h2>
    </summary>

<details>
    <summary>
        <h3>Install PyTorch</h3>    
    </summary>

For the purpose of this guideline I will be using the latest version of [PyTorch](https://pytorch.org/get-started/locally/) (stable 2.0.1 as of September 12th, 2023)

1. First, create and activate a separate conda environmet for PyTorch:

```bash
conda create -n torch2 python=3.10
conda activate torch2
```

2. Then, run the following command to install PyTorch with the appropriate version of CUDA:

```bash
conda install pytorch torchvision torchaudio pytorch-cuda=11.7 -c pytorch -c nvidia
```

3. To ensure that PyTorch was installed correctly,verify the installation by running the following sample PyTorch code:

```bash
python -c "import torch; print(f'{torch.__version__=}'); x = torch.rand(5, 3); print(f'{x=}')"
```

4. Additionally, to check if your GPU driver and CUDA/ROCm is enabled and accessible by PyTorch, run the following commands to return whether or not the GPU driver is enabled:

```bash
python -c "import torch; print(f'{torch.cuda.is_available()=}'); x = torch.rand(5, 3); x = x.cuda(); print(f'{x.device}')"
```
</details>

<details>
    <summary>
        <h3>Install TensorFlow</h3>    
    </summary>

This guide is for the latest stable version of [TensorFlow](https://github.com/tensorflow/tensorflow/releases/tag/v2.13.0) (v2.13.0 as of September 12th, 2023).

1. Create a new conda environment named `tf` with the following command:

```bash
conda create --name tf python=3.9
```
You can deactivate and activate it with the following commands.

```bash
conda deactivate
conda activate tf
```
Make sure it is activated for the rest of the installation.

2. Then, install CUDA and cuDNN with `conda` and `pip`:

```bash
conda install -c conda-forge cudatoolkit=11.8.0
pip install nvidia-cudnn-cu11==8.6.0.163
```

3. Next configure the system paths:

```bash
mkdir -p $CONDA_PREFIX/etc/conda/activate.d
echo 'CUDNN_PATH=$(dirname $(python -c "import nvidia.cudnn;print(nvidia.cudnn.__file__)"))' >> $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh
echo 'export LD_LIBRARY_PATH=$CUDNN_PATH/lib:$CONDA_PREFIX/lib/:$LD_LIBRARY_PATH' >> $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh
```

The system paths will be automatically configured when you reactivate this conda environment.

4. Install TensorFlow with `pip`:

```bash
pip install tensorflow==2.13.*
```

5. Verify the TensorFlow setup:

```bash
python3 -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
```

If a tensor is returned, you've installed TensorFlow successfully.

6. Verify the GPU setup:

```bash
python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
```
If a list of GPU devices is returned, you've installed TensorFlow successfully.
</details>

</details>


<details>
    <summary>
        <h2>Optional steps</h2>
    </summary>

<details>
    <summary>
        <h3>Install Gnome-tweaks</h3>
    </summary>

You may notice, especially in monitors with high resolution, e.g., 3840 x 2160 (4K) or 2560 x 1440 (WQHD), Ubuntu tends to not scale its UI optimally. Thus, fonts and icons may appear very small and not easy to read. Although, Ubunut 22.04.3 allows to scale its UI through the display settings, I prefer to use [gnome-tweaks](https://gitlab.gnome.org/GNOME/gnome-tweaks) instead. In order to install this tool you will need to go through the following steps:

1. Make sure that you have the `universe` repository enabled on your Ubuntu system:
```bash
sudo add-apt-repository universe
```

2. Next, execute the following command to install Tweak Tool on your Ubuntu 22.04 system:
```bash
sudo apt install gnome-tweaks
````

3. Finally, run `gnome-tweaks` through your command terminal.

4. From the menu on the left select **Fonts** and then adjust the **Scaling Factor** accordingly.

</details>

</details>



<h2>Resources</h2>

  - [How to dual-boot Windows 11 and Linux on your PC](https://www.xda-developers.com/dual-boot-windows-11-linux/)
  - [Disable secure boot y/n?](https://askubuntu.com/questions/785120/disable-secure-boot-y-n)
  - [Ubuntu](https://ubuntu.com/)
  - [NVIDIA GPU drivers](https://www.nvidia.com/Download/index.aspx)
  - [NVIDIA SMI](https://developer.nvidia.com/nvidia-system-management-interface)
  - [Ubuntu Linux Install Nvidia Driver (Latest Proprietary Driver)](https://www.cyberciti.biz/faq/ubuntu-linux-install-nvidia-driver-latest-proprietary-driver/)
  - [Conda](https://docs.conda.io/en/latest/)
  - [Miniconda](https://docs.conda.io/projects/miniconda/en/latest/#)
  - [PyTorch](https://pytorch.org/)
  - [TensorFlow](https://www.tensorflow.org/)
  - [Gnome-tweaks](https://linuxconfig.org/how-to-install-tweak-tool-on-ubuntu-20-04-lts-focal-fossa-linux)

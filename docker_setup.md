## Objective

Use mutliple [Deep Learning Frameworks](https://developer.nvidia.com/deep-learning-frameworks) on the same machine, with the use of **docker images-containers** for easy setup of different frameworks and their dependencies (CUDA stack - CUDA, cuDNN CuBLAS, NCCL) + **Python IDE** for faster development. To enable these capabilities we are going to use the following tools: (feel free to add extra tools-services that you think they might me useful for the aforementioned cause)

* [nvidia-docker](https://github.com/NVIDIA/nvidia-docker)
* [PyCharm](https://www.jetbrains.com/pycharm/)
* [TensorFlow](https://www.tensorflow.org/)

***
(The following procedure was tested on **Ubuntu 18.04.1 LTS 64-bit**) 
## nvidia-docker setup

1. Install [NVidia driver](https://github.com/NVIDIA/nvidia-docker/wiki/Frequently-Asked-Questions#how-do-i-install-the-nvidia-driver)
2. Install [Docker](https://docs.docker.com/install/)
	1. Uninstall old versions
	``` bash
	sudo apt-get remove docker docker-engine docker.io
	```

	2. Set up the repository
	``` bash
	# 1. Update the apt package index
	sudo apt-get update
	# 2. Install packages to allow apt to use a repository over HTTPS
	sudo apt-get install \
	    apt-transport-https \
	    ca-certificates \
	    curl \
	    software-properties-common
	# 3. Add Docker’s official GPG key:
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
	# 4. Verify that you now have the key with the fingerprint 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 
	# 0EBF CD88, by searching for the last 8 characters of the fingerprint.
	sudo apt-key fingerprint 0EBFCD88
	pub   4096R/0EBFCD88 2017-02-22
	      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
	uid                  Docker Release (CE deb) <docker@docker.com>
	sub   4096R/F273FCD8 2017-02-22
	# 5. Use the following command to set up the stable repository
	sudo add-apt-repository \
	   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
	   $(lsb_release -cs) \
	   stable"
	```

	3. Install Docker CE
	``` bash
	# 1. Update the apt package index
	sudo apt-get update
	# 2. Install the latest version of Docker CE
	sudo apt-get install docker-ce
	# 3. Verify that Docker CE is installed correctly by running the hello-world image.
	sudo docker run hello-world
	```

	4. Post-installation steps for Linux
	This link contains optional procedures for configuring Linux hosts to work better with Docker
	[Post-installation steps for Linux](https://docs.docker.com/install/linux/linux-postinstall/)

3. Install nvidia-docker
``` bash
# 1. If you have nvidia-docker 1.0 installed: we need to remove it and all existing GPU containers
sudo docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
sudo apt-get purge -y nvidia-docker
# 2. Add the package repositories
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
# 3. Install nvidia-docker2 and reload the Docker daemon configuration
sudo apt-get install -y nvidia-docker2
sudo pkill -SIGHUP dockerd
# 4. Test nvidia-smi with the latest official CUDA image
docker run --runtime=nvidia --rm nvidia/cuda:9.0-base nvidia-smi
```

## PyCharm setup
1. Download [PyCharm Professional Edition](https://www.jetbrains.com/pycharm/download/#section=linux)
2. Follow installation instructions found in *pycharm-\*/Install-Linux-tar.txt* 
	* **Linux Installation Instructions**
		1. Unpack the PyCharm distribution archive that you downloaded to
		where you wish to install the program. We will refer to this destination
		location as your {installation home} below.
		2. Open a console and cd into "{installation home}/bin" and type:
		``` bash
		./pycharm.sh
		```
		to start the application. As a side effect, this will initialize various
		configuration files in the ~/.PyCharm2018.3 directory.

## TensorFlow setup
Download [TensorFlow Docker Images](https://hub.docker.com/r/tensorflow/tensorflow/) and run TensorFlow containers. Docker uses containers to create virtual environments that isolate a TensorFlow installation from the rest of the system. TensorFlow programs are run within this virtual environment that can share resources with its host machine (access directories, use the GPU, connect to the Internet, etc.)

Docker is the easiest way to enable TensorFlow GPU support on Linux since only the NVIDIA® GPU driver is required on the host machine (the NVIDIA® CUDA® Toolkit does not need to be installed).

Follow [Docker install guide](https://www.tensorflow.org/install/docker)

#### TensorFlow Docker requirements
1. Install Docker on your local host machine
2. For GPU support on Linux, install nvidia-docker

#### Download a TensorFlow Docker image
Image releases are tagged using the following format

|**Tag**|**Description**|
|-------|-----------------------------------------------------------------------------------
|tag-gpu|The specified tag release with GPU support|
|tag-py3|The specified tag release with Python 3 support|
|tag-jupyter|The specified tag release with Jupyter (includes TensorFlow tutorial notebooks)|

#### Download TensorFlow release images
Docker pull command
``` bash
# Image containing TensorFlow 1.5.0 with GPU support
sudo docker pull tensorflow/tensorflow:1.5.0-gpu
```

#### Run a GPU-enabled TensorFlow image
Command Example
``` bash
sudo nvidia-docker run -it -rm --mount source=<volume_name>,target=<directory_to_save_in_container> <image_name> <command>
```

Use the already downloaded TensorFlow 1.5.0-gpu image to start a bash shell session in the container
``` bash
sudo nvidia-docker run -it -rm tensorflow/tensorflow:1.5.0-gpu bash
```

You can rename the image as follows
``` bash
# Create a new tag
sudo docker tag <old_name> <new_name>
# Remove old tag (WARNING: if image has only one tag rmi will remove the image - 
# run sudo docker image ls to list image information)
sudo docker rmi <old_name>
```

***

## PyCharm - enable Docker support

You can enable Docker support in PyCharm as explained at the first section of the following link [PyCharm - Enable Docker Support](https://www.jetbrains.com/help/pycharm/docker.html).
If you are in a Linux environment, make sure to connect to Docker daemon through Unix socket.

***

## PyCharm - configure remote interpreter via Docker

To run applications in development environments deployed in Docker containers follow the next set of instructions [PyCharm - Docker Remote Interpreter](https://www.jetbrains.com/help/pycharm/using-docker-as-a-remote-interpreter.html). If you wish to use GPU-enabled libaries you have to configure the Docker daemon, since PyCharm doesn't support nvidia-docker and does not provide the ability to use command line arguments to `docker run` command, before deploying the container. Basically, you need to edit the `/etc/docker/daemon.json`, by adding the first level key `"default-runtime": "nvidia"`, restart docker daemon (ex. `sudo service docker restart`) and then all containers on that host will run with nvidia runtime.
 
***

## Run Docker image

#### Run tensorboard

Create a .sh script and copy-paste the following code 

``` bash
#!/bin/bash

POSITIONAL=()
while [[ $# -gt 0 ]]
do
key="$1"

case $key in
    --host_port)
    HOST_PORT="$2"
    shift # past argument
    shift # past value
    ;;
    --container_port)
    CONTAINER_TCP_PORT="$2"
    shift # past argument
    shift # past value
    ;;
    --local_dir)
    LOCAL_DIR="$PWD/"$2""
    shift # past argument
    shift # past value
    ;;
    --container_dir)
    CONTAINER_DIR="$2"
    shift # past argument
    shift # past value
    ;;
    --image)
    IMAGE_NAME="$2"
    shift # past argument
    shift # past value
    ;;
    *)    # unknown option
    POSITIONAL+=("$1") # save it in an array for later
    shift # past argument
    ;;
esac
done
set -- "${POSITIONAL[@]}" # restore positional parameters

docker run -p "${HOST_PORT}":"${CONTAINER_TCP_PORT}" -v "${LOCAL_DIR}":"${CONTAINER_DIR}" "${IMAGE_NAME}" tensorboard --port "${CONTAINER_TCP_PORT}" --logdir "${CONTAINER_DIR}"
```
Run the script as follows
``` bash
chmod +x <script_name>.sh
./<script_name>.sh --host_port <port-id> --container-port <port-id> --local_dir <relative-path-to-model-directory> --container_dir <container-directory> --image <image-name>
```

Check if port forwarding is working
``` bash
curl http://localhost:<host_port>
```

Visit the previous url from your favorite browser, to see some nice visualizations e.g. model's  computational graph, metric plots etc.

#### Xserver support

``` bash
xhost +
docker run -it --rm --privileged -e DISPLAY=$DISPLAY --net=host --ipc=host -v /tmp/.X11-unix:/tmp/.X11-unix <image-name> <command>
```
docker run -it --rm --privileged -e DISPLAY=$DISPLAY --net=host --ipc=host -v /tmp/.X11-unix:/tmp/.X11-unix -v /home/melinos/Repos:/home/melinos/Repos -v /media/:/media tf_1_5_custom bash

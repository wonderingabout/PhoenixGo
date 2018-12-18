In the example below, ubuntu 16.04 LTS is used with cuda 9.0 (deb install) ,
cudnn 7.1.4 (deb install), tensorrt 3.0.4 (deb install), as well as bazel 0.17.2 (.sh file install) (0.11.1 also works)

Other linux distributions with nvidia tar install are possible too, but it is easier to do it like that, and remember that this is just an example. The settings below have been tested to be working and to fix most common path issues, and
are shown as an interactive help :

Before starting, after cuda 9.0 and cudnn 7.1.4 deb installs, run a test to see if your install works by compiling and runing a cudnn code sample :

```
cp -r /usr/src/cudnn_samples_v7/ ~ && cd ~/cudnn_samples_v7/mnistCUDNN && make clean && make && ./mnistCUDNN
```

should display this : `Test passed!`
Then, after cuda 9.0 deb install and cudnn 7.1.4 deb are installed successfully, one post install step is needed : add the path to cuda-9.0.

### 1) post-install : do the path exports

The `export` command alone adds the path during current boot, but the changes will be lost after reboot.
This is why we will add paths permanently using `bashrc` (and `/etc/environment` for ubuntu here)

##### bashrc

open bashrc file :

`sudo nano ~/.bashrc`

you need to add the lines below at the end of the bashrc file (using nano, or the text editor of your choice) :

```
# add paths for cuda-9.0
export PATH=${PATH}:/usr/local/cuda-9.0/bin
export CUDA_HOME=${CUDA_HOME}:/usr/local/cuda:/usr/local/cuda-9.0
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/local/cuda-9.0/lib64
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/extras/CUPTI/lib64
```

same as this screenshot
![bashrc](https://raw.githubusercontent.com/wonderingabout/nvidia-archives/master/nano-bashrc.png)

With nano, after editing is finished, save and exit with `Ctrl+X` + `y` + press ENTER key.

Now update bashrc file :

`source ~/.bashrc`

as you can now see, now the command nvcc --version will work successfully : 

`nvcc --version`

should display something like this : 

```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2017 NVIDIA Corporation
Built on Fri_Sep__1_21:08:03_CDT_2017
Cuda compilation tools, release 9.0, V9.0.176
```

##### etc/environment (ubuntu)

for ubuntu users, it is also recommended to add path in /etc/environment :

`sudo nano /etc/environment`

add this part in the paths (with the `:`) at the end of the file

`:/usr/local/cuda-9.0/bin`

same as the screenshot below :

![etc-environment](https://github.com/wonderingabout/nvidia-archives/blob/master/nano-etc-environment.png)

then save and exit nano.

Reboot to finalize the installation of cuda and cudnn, as well as the add in /etc/environment

`sudo reboot`


After reboot, in addition to `nvcc --version`, you can also check if cuda installation is a success with `cat /proc/driver/nvidia/version`

should display something like this :

```
NVRM version: NVIDIA UNIX x86_64 Kernel Module  384.130  Wed Mar 21 03:37:26 PDT 2018
GCC version:  gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.10)
```

If you need help of how to install the deb files of cuda 9.0, cudnn 7.1.4, and tensorrt 3.0.4 for ubuntu 16.04, you can go here [@wonderingabout](https://github.com/wonderingabout/nvidia-archives)

### 2) locate cuda and cudnn paths, and update database if not here

Run this command : `locate libcudart.so && locate libcudnn.so.7`

you need to see something like this : 

```
/usr/local/cuda-9.0/doc/man/man7/libcudart.so.7
/usr/local/cuda-9.0/targets/x86_64-linux/lib/libcudart.so
/usr/local/cuda-9.0/targets/x86_64-linux/lib/libcudart.so.9.0
/usr/local/cuda-9.0/targets/x86_64-linux/lib/libcudart.so.9.0.176
/usr/lib/x86_64-linux-gnu/libcudnn.so.7
/usr/lib/x86_64-linux-gnu/libcudnn.so.7.1.4
```
If you don't see this, run this command : 

`sudo updatedb && locate libcudart.so && locate libcudnn.so.7`

It should now display all the cuda and cudnn paths same as above.
Reboot your computer to finalize.

### 3) during bazel compile, this is the paths you need to put

Press ENTER for every prompt to choose default settings (or `n` if you dont want a setting), except for these : 

- CUDA : choose `y` , version `9.0`, and custom cuda path `/usr/local/cuda-9.0/`
- cudnn : choose version `7.1` and custom cudnn path `/usr/lib/x86_64-linux-gnu/`
- if you use tensorrt do `y` and press enter to keep default path

same as below :

```
Do you wish to build TensorFlow with CUDA support? [y/N]: y 
CUDA support will be enabled for TensorFlow.

Please specify the CUDA SDK version you want to use, e.g. 7.0. [Leave empty to default to CUDA 9.0]: 9.0

Please specify the location where CUDA 9.0 toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]: /usr/local/cuda-9.0/

Please specify the cuDNN version you want to use. [Leave empty to default to cuDNN 7.0]: 7.1

Please specify the location where cuDNN 7 library is installed. Refer to README.md for more details. [Default is /usr/local/cuda-9.0/]: /usr/lib/x86_64-linux-gnu/

Do you wish to build TensorFlow with TensorRT support? [y/N]: y
TensorRT support will be enabled for TensorFlow.

Please specify the location where TensorRT is installed. [Default is /usr/lib/x86_64-linux-gnu]:
```

Final words :

Remember that these settings are just an example, other settings or package versions or linux distributions are possible too, but this example has been tested to successfully work on ubuntu 16.04 LTS with deb install of cuda 9.0, deb install of cudnn 7.1.4, deb install of tensorrt 3.0.4, as well as .sh file install of bazel 0.17.2

They are provided as a general help for linux compile and run, they are not an obligatory method to use, but will hopefully make using PhoenixGo on linux systems easier

credits : 
- [nvidia pdf install guide for cuda 9.0](http://developer.download.nvidia.com/compute/cuda/9.0/Prod/docs/sidebar/CUDA_Installation_Guide_Linux.pdf)
- [medium.com/@zhanwenchen/](https://medium.com/@zhanwenchen/install-cuda-and-cudnn-for-tensorflow-gpu-on-ubuntu-79306e4ac04e)
- [medium.com/@mishra.thedeepak](https://medium.com/@mishra.thedeepak/cuda-and-cudnn-installation-for-tensorflow-gpu-79beebb356d2)
- [nvidia cudnn](https://developer.nvidia.com/rdp/cudnn-archive)
- [nvidia tensorrt](https://developer.nvidia.com/nvidia-tensorrt3-download)
- [wonderingabout](https://github.com/wonderingabout/nvidia-archives)

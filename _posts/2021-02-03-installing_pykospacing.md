---
title: "Installing pykospacing"
date: 2021-02-03 00:00:00 -0400
categories: NLP
---




# pykospacing.git

아래와 같이 type하여 install

pip install git+https://github.com/haven-jeon/PyKoSpacing.git

이때, tensorflow version error 발생 시, tensorflow를 upgrade

pip install pip --upgrade
pip install --user --upgrade tensorflow

&nbsp;

# code installation

git clone https://github.com/haven-jeon/PyKoSpacing.git
cd PyKoSpacing
pip install -r requirements.txt

sudo aptitude search cudart

libcudart9.1/bionic 9.1.85-3ubuntu1 amd64
  NVIDIA CUDA Runtime Library

sudo apt-get install libcudart9.1

&nbsp;

# installing libcudart

https://developer.nvidia.com/cuda-downloads

&nbsp;

## installing on Ubuntu 18.04

wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/11.2.0/local_installers/cuda-repo-ubuntu1804-11-2-local_11.2.0-460.27.04-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1804-11-2-local_11.2.0-460.27.04-1_amd64.deb
sudo apt-key add /var/cuda-repo-ubuntu1804-11-2-local/7fa2af80.pub
sudo apt-get update
sudo apt-get -y install cuda


&nbsp;

## set PATH & LD_LIBRARY_PATH

vim ~/.bashrc
export PATH=$PATH:/usr/local/cuda-11/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-11/lib64

source ~/.profile

&nbsp;

이후, 

>>> from pykospacing import spacing

2021-02-03 17:55:06.558850: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library libcudart.so.11.0
2021-02-03 17:55:17.036538: I tensorflow/compiler/jit/xla_cpu_device.cc:41] Not creating XLA devices, tf_xla_enable_xla_devices not set
2021-02-03 17:55:17.058282: I tensorflow/stream_executor/platform/default/dso_loader.cc:49] Successfully opened dynamic library libcuda.so.1
2021-02-03 17:55:17.728322: E tensorflow/stream_executor/cuda/cuda_driver.cc:328] failed call to cuInit: CUDA_ERROR_NO_DEVICE: no CUDA-capable device is detected
2021-02-03 17:55:17.730261: I tensorflow/stream_executor/cuda/cuda_diagnostics.cc:156] kernel driver does not appear to be running on this host (DESKTOP-EDP2VIM): /proc/driver/nvidia/version does not exist
2021-02-03 17:55:17.733295: I tensorflow/compiler/jit/xla_gpu_device.cc:99] Not creating XLA devices, tf_xla_enable_xla_devices not set

&nbsp;

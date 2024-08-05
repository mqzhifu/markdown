# PCL

## filter

过滤掉 点云数据中的 噪音

## feature

特征

## keyword point

关键点

## registration

配准

## kd\-tree

## oc\-tree

## segmentation

分割

## sample consensus

拟合

## surface

## recognition

识别

## io

处理：PCD 格式点云数据文件

处理：IO设备的数据

## viszualization

可视化

## 安装\-ubuntu18.0

前置回顾：

1. mac 安装失败
2. cantos7 centos8 centos\-strem 均安装失败

最后只能用ubuntu了

包安装：

```
apt-get install llvm
apt-get install clang clang-format-10
apt-get install git build-essential linux-libc-dev
apt-get install cmake cmake-gui
apt-get install libusb-1.0-0-dev libusb-dev libudev-dev
apt-get install mpi-default-dev openmpi-bin openmpi-common 
apt-get install libeigen3-dev
apt-get install libboost-all-dev
apt-get install libflann1.9 libflann-dev
apt-get install libqhull* libgtest-dev 
apt-get install libx11-doc mesa-utils   
apt-get install mono-complete
apt-get install libpcap-dev
apt-get install libopencv-dev
apt-get install libopenni-dev libopenni2-dev 

#下面几个包循环依赖这个包，但这个包在不同os版本下数字不太一样，注意下：
apt-get install libx11-6=2:1.6.9-2ubuntu1.2
apt-get install libx11-6=2:1.6.4-3ubuntu0.4

apt-get install libxmu-dev libxi-dev
apt-get install freeglut3-dev pkg-config
apt-get install libvtk7.1p libvtk7.1p-qt libvtk7-dev libvtk7-qt-dev vtk7 
apt-get install qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools

```

```
wget https://www.coin-or.org/download/source/metslib/metslib-0.5.2.zip
unzip metslib-0.5.2.zip 
cd metslib-0.5.2
./configure
make
make install
```

PCL安装：

```
#67MB，而且是github，速度很慢，可以从本机传上去一上
wget wget https://github.com/PointCloudLibrary/pcl/releases/download/pcl-1.12.0/source.zip
unzip source.zip
cd pcl
mkdir build
cd build

cmake  -D CMAKE_BUILD_TYPE=None  -D BUILD_GPU=ON  -D BUILD_apps=ON  -D BUILD_examples=ON ..

#看一眼，build后的信息，是不是都正常，有 not found 的，继续安装

make & make install
```

## 安装\-ubuntu\-UI

sudo apt\-get update && sudo apt\-get upgrade

sudo apt\-get install lightdm

sudo systemctl start lightdm

sudo apt\-get install tasksel

sudo tasksel install xubuntu\-desktop

sudo apt\-get install ubuntu\-desktop

# How to use VEYE and CS series cameras on Firefly's RK35XX board (Ubuntu)
This is a mirror of [our wiki article](http://wiki.veye.cc/index.php/VEYE_CS_Camera_on_Firfly_Boards).
## Overview
VEYE series and CS series cameras are the video streaming mode MIPI cameras we designed. This article takes Firefly's ROC-RK3566-PC board as an example to introduce how to connect VEYE and CS series cameras to RK3566/RK3568/RK3588 system.
## Camera Module List

| Series  | Model  | Status  |
| ------------ | ------------ | ------------ |
| VEYE Series  | VEYE-MIPI-IMX327S  | Done  |
| VEYE Series  | VEYE-MIPI-IMX462  | Done  |
| VEYE Series  | VEYE-MIPI-IMX385  | Done  |
## Hardware Setup
VEYE series and CS series cameras are provided with Raspberry Pi compatible 15Pin FFC connector. An [ADP-Tfirefly adapter board](resources/ADP-Tfirefly-V1.0.pdf) is required to adapt to the ROC-RK3566-PC board.
### Connection of camera and ADP-Tfirefly
The two are connected using 1.0 mm pitch*15P FFC cable with opposite direction. The cable must be inserted with the silver contacts facing outside.

![Firefly Board and VEYE camera overall](resources/Firefly%20Board%20and%20VEYE%20camera%20overall.jpg)

### Connection of ADP-Tfirefly and Firefly Board
The two are connected using 0.5 mm pitch*30P FFC cable with same direction. The cable must be inserted with the silver contacts facing inside.

![ADP-Tfirefly connect to Firefly board](resources/ADP-Tfirefly%20connect%20to%20Firefly%20board.jpg)

### Overall connection
![Firefly Board and VEYE camera overall](resources/Firefly%20Board%20and%20VEYE%20camera%20overall.jpg)
## Introduction to github repositories
includes：
- driver source code
- i2c toolkits
- application demo

In addition, a compiled linux kernel installation package is provided in the releases.

## Upgrade Firefly Ubuntu system
### Overview
This section describes how to update the RK35xx system to support our camera modules.

For some versions, we provide a deb installer that can be installed directly. For versions where no installer is provided, you will need to refer to later chapters to compile from the driver source code.

Although we are now using Ubuntu system as an example to introduce, other Linux distributions can also refer to this article.

### Burn Firefly standard system
Refer to the [Firefly documentation](https://wiki.t-firefly.com/en/ROC-RK3566-PC/01-bootmode.html) to burn in a standard system.

### Using prebuilt Image and dtb file
Using the compiled debain installation package

On the RK35xx board, execute :
```
wget https://github.com/veyeimaging/rk356x_firefly/releases/latest/download/rk356x_firefly.tar.gz

tar -xavf rk356x_firefly.tar.gz

cd cd rk356x_firefly/released_images/ROC-RK3566-PC/ubuntu/

sudo dpkg -i linux-image-4.19.232_4.19.232-21_arm64.deb
```
If the version does not match, it needs to be compiled from the source code.

## Check system status
Run the following command to confirm whether the camera is probed.
- VEYE-MIPI-XXX
`dmesg | grep veye`
The output message appears as shown below：
```
veyecam2m 4-003b:  camera id is veyecam2m

veyecam2m 4-003b: sensor is IMX327
```
- Run the following command to check the presence of video node.

`ls /dev/video0`

The output message appears as shown below.

`video0`

The camera can be seen connected to the i2c-4.

## Samples
### v4l2-ctl

#### Install v4l2-utils

`sudo apt-get install v4l-utils`

####  List the data formats supported by the camera

`v4l2-ctl --list-formats-ext`

#### Snap YUV picture
`v4l2-ctl --set-fmt-video=width=1920,height=1080,pixelformat=UYVY --stream-mmap --stream-count=1 --stream-to=uyvy-1920x1080.yuv`

`v4l2-ctl --set-fmt-video=width=1920,height=1080,pixelformat='NV12' --stream-mmap --stream-count=100 --stream-to=nv12-1920x1080.yuv`

Play YUV picture
`ffplay -f rawvideo -video_size 1920x1080 -pix_fmt nv12 nv12-1920x1080.yuv`

#### Check frame rate
`v4l2-ctl --set-fmt-video=width=1920,height=1080,pixelformat=UYVY --stream-mmap --stream-count=-1 --stream-to=/dev/null`

### yavta
```
git clone https://github.com/veyeimaging/yavta.git

cd yavta;make

./yavta -c1 -Fuyvy-1920x1080.yuv --skip 0 -f UYVY -s 1920x1080 /dev/video0
```

### gstreamer
We provide several gstreamer routines that implement the preview, capture, and video recording functions. See the samples directory on github for details.

### Import to OpenCV

First install OpenCV:
`sudo apt install python3-opencv`

We provide several routines to import camera data into opencv. See the samples directory on github for details.

In addition, [this page from Firefly](https://wiki.t-firefly.com/en/Firefly-Linux-Guide/demo_OpenCV_support.html) has some reference value.

## i2c script for parameter configuration

Because of the high degree of freedom of our camera parameters, we do not use V4L2 parameters to control, but use scripts to configure parameters.

[https://github.com/veyeimaging/rk356x_firefly/tree/main/i2c_cmd](https://github.com/veyeimaging/rk356x_firefly/tree/main/i2c_cmd)

using -b option to identify which bus you want to use.

- VEYE series
Video Control Toolkits Manual ：[VEYE-MIPI-327 I2C](http://wiki.veye.cc/index.php/VEYE-MIPI-290/327_i2c/)

- CS series
Video Control Toolkits Manual ：[CS-MIPI-X I2C](http://wiki.veye.cc/index.php/CS-MIPI-X_i2c)

##  Compile drivers and dtb from source code
[https://github.com/veyeimaging/rk356x_firefly/tree/main/linux/drivers](https://github.com/veyeimaging/rk356x_firefly/tree/main/linux/drivers)

## References
- ROC-RK3566-PC Manual
[https://wiki.t-firefly.com/en/ROC-RK3566-PC/](https://wiki.t-firefly.com/en/ROC-RK3566-PC/)
- Firefly Linux User Guide
[https://wiki.t-firefly.com/en/Firefly-Linux-Guide/index.html](https://wiki.t-firefly.com/en/Firefly-Linux-Guide/index.html)

## Document History
- 2022-10-22
Release 1st version.

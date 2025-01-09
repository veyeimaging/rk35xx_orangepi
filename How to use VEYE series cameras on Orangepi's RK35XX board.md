# How to use VEYE series cameras on Orange Pi's RK35XX board

This is a mirror of [our wiki article](https://wiki.veye.cc/index.php/VEYE_Camera_on_Orange_Pi%27s_RK35XX_Boards).

[toc]

## Overview
VEYE series cameras are the video streaming mode MIPI cameras we designed. This article takes OrangePi CM4 and CM5 board as an example to introduce how to connect VEYE series cameras to RK3566/RK3568/RK3588 system.

We provide drivers for Linux.

## Camera Module List

| Series  | Model  | Status  |
| ------------ | ------------ | ------------ |
| VEYE Series  | VEYE-MIPI-IMX327S  | Done  |
| VEYE Series  | VEYE-MIPI-IMX462  | Done  |
| VEYE Series  | VEYE-MIPI-IMX385  | Done  |

## Hardware Setup
We use the official baseboards of the Orange Pi CM4 and CM5, which both provide a 15-pin connector compatible with Raspberry Pi. This allows us to install our cameras directly onto their mainboards without the need for an adapter board.

### Connection of Camera and OrangePi CM4 Board
The two are connected using 15-pin FFC cables with opposite-facing contacts, please pay attention to the orientation of the contact surfaces.

Note that only the CAM1 shown in the image below supports VEYE cameras.

![OrangePi CM4 to VEYE cam](resources/OrangePi_CM4_to_VEYE_cam.jpg)

### Connection of Camera and OrangePi CM5 Board
The two are connected using 15-pin FFC cables with opposite-facing contacts; please pay attention to the orientation of the contact surfaces.

The OrangePi CM5 supports up to four VEYE cameras. The following diagram shows the hardware connection method for simultaneously connecting multiple cameras.

![OrangePi CM5 to all cam overview](resources/OrangePi_CM5_to_all_cam_overview.jpg)

![OrangePi CM5 to all cam backview](resources/OrangePi_CM5_to_all_cam_backview.jpg)

## Introduction to github repositories

https://github.com/veyeimaging/rk35xx_veye_bsp
https://github.com/veyeimaging/rk35xx_orangepi

includes：
- driver source code
- i2c toolkits
- application demo

In addition, a compiled linux kernel installation package and Android image is provided in the [releases](https://github.com/veyeimaging/rk35xx_orangepi/releases).

## Upgrade the Ubuntu system

We provide a flashing image for the release system, as well as a deb package for the Linux kernel.

Refer to the [OrangePi CM4 user manual](http://www.orangepi.org/orangepiwiki/index.php/Orange_Pi_CM4) or the [OrangePi CM5](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-CM5.html) user manual for instructions on flashing the system. Alternatively, you can use the general `dpkg` command to install the deb package.

## Check system status
Run the following command to confirm whether the camera is probed.
`dmesg | grep veye`
### CM4
The output message appears as shown below：
```bash
veyecam2m 1-003b: camera id is veyecam2m
veyecam2m 1-003b: sensor is IMX462
```
- Run the following command to check the presence of video node.

`ls /dev/video0`

The output message appears as shown below.
`video0`

The correspondence of the various information is as follows:

| CAM num | I2C | media node         | video node    |
|---------|-----|--------------------|---------------|
| 1       | 1   | veyecam2m 1-003b   | /dev/video0   |

### CM5
The CM5 supports the connection of up to 4 cameras. The dmesg information includes the following content:
```bash
veyecam2m 3-003b:  camera id is veyecam2m
veyecam2m 3-003b: sensor is IMX462
veyecam2m 4-003b:  camera id is veyecam2m
veyecam2m 4-003b: sensor is IMX462
veyecam2m 5-003b:  camera id is veyecam2m
veyecam2m 5-003b: sensor is IMX462
veyecam2m 6-003b:  camera id is veyecam2m
veyecam2m 6-003b: sensor is IMX462
```
The correspondence of the various information is as follows:

| CAM num | I2C | media node         | video node    |
|---------|-----|--------------------|---------------|
| 1       | 4   | veyecam2m 4-003b   | /dev/video22  |
| 2       | 3   | veyecam2m 3-003b   | /dev/video33  |
| 3       | 5   | veyecam2m 5-003b   | /dev/video11  |
| 4       | 6   | veyecam2m 6-003b   | /dev/video0   |


## Application examples
### v4l2-ctl
In the following example, `/dev/video0` can be modified as needed to correspond to the video node of the camera you wish to operate.

#### Install v4l2-utils
`sudo apt-get install v4l-utils`

####  List the data formats supported by the camera

`v4l2-ctl --list-formats-ext`

#### Snap picture

`v4l2-ctl --set-fmt-video=width=1920,height=1080,pixelformat='NV12' --stream-mmap --stream-count=100 --stream-to=nv12-1920x1080.yuv -d /dev/video0`

For RK3566, also:
`v4l2-ctl --set-fmt-video=width=1920,height=1080,pixelformat=UYVY --stream-mmap --stream-count=1 --stream-to=uyvy-1920x1080.yuv -d /dev/video0`

Play YUV picture
`ffplay -f rawvideo -video_size 1920x1080 -pix_fmt nv12 nv12-1920x1080.yuv`
You can use software like YUV Player or Vooya to play the images.

#### Check frame rate
`v4l2-ctl --set-fmt-video=width=1920,height=1080,pixelformat=NV12 --stream-mmap --stream-count=-1 --stream-to=/dev/null -d /dev/video0`

#### yavta
```bash
git clone https://github.com/veyeimaging/yavta.git

cd yavta;make

./yavta -c1 -Fnv12-1920x1080.yuv --skip 0 -f NV12 -s 1920x1080 /dev/video0
```

### gstreamer
We provide several gstreamer routines that implement the preview, capture, and video recording functions. See the [samples](https://github.com/veyeimaging/rk35xx_veye_bsp/tree/main/samples) directory on github for details.

### Import to OpenCV

First install OpenCV:
`sudo apt install python3-opencv`

We provide several routines to import camera data into opencv. See the samples directory on github for details.

##  Compile drivers and dtb from source code

https://github.com/veyeimaging/rk35xx_orangepi/tree/main/linux

## i2c script for parameter configuration

Because of the high degree of freedom of our camera parameters, we do not use V4L2 parameters to control, but use scripts to configure parameters.

https://github.com/veyeimaging/rk35xx_veye_bsp/tree/main/i2c_cmd

using -b option to identify which bus you want to use.

- VEYE series
Video Control Toolkits Manual ：[VEYE-MIPI-327 I2C](http://wiki.veye.cc/index.php/VEYE-MIPI-290/327_i2c/)

## References
- OrangePi CM4
http://www.orangepi.org/orangepiwiki/index.php/Orange_Pi_CM4

- OrangePi CM5
http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-CM5.html

## Document History
- 2025-01-09
Release 1st version.

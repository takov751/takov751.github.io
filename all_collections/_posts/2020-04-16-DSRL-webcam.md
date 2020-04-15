---
layout: post
title:  "DSLR camera as webcam on any Linux distro"
date:   2020-04-16 10:00:00 +0000
categories: dsrl repurpose
author: takov751
published: true
---

## The basic idea

In these times there are some cases,where you need a webcam even , if you does not own one. You could purchase a cheap webcam from any webshop,however the quality and compatiblity allways questionable. Of course there are the top of the line Logitech webcams, however if you read this small article you neither has the will the buy one, nor you have time to wait for the delivery. This brings us here. There are other solution like using your phone over the USB/Wifi, however I choose this solution ,because the DSLR quality is greater,but beware neither of the solution mentioned before gives you high framerate video feed.I am using Ubuntu 20.04 ,in case you are using a different distribution you only need to make sure that the packages installed with the package manager of your choosen distro.

## Video feed over USB

For this we will need the software called `gphoto2`.First we have to make sure that the choosen DSLR supported. On the following website the gphoto2 project updates this list after all mayor build . 

[SUPPORTED DSLR](http://www.gphoto.org/proj/libgphoto2/support.php)

From my experience we are looking for the `Liveview` ability. If our devices supports this function, then we are lucky. We have to capture_movie and then we have to direct this feed to the virtual webcam device which we will create in the next section.
To make sure that our DSLR working. Connect the Camera to the PC via USB and turn on the device.
With the following command we'll be able to check if the device working as intended :
first install dependencies:
```bash
sudo apt install gphoto2 vlc
```

```bash
gphoto2 --capture-movie --stdout | vlc - 
```
With this you will be able to see your camera feed the resolution and the framerate might vary on each device. The video feed quality can be adjusted on the DSLR itself.

I use a `Canon EOS 1200D` the video feed is `1056x704  5-8fps`

## Virtual Webcam device

This section we will install a kernel module called `v4l2loopback`, which is not part of the kernel by default. We just have to install it :

```bash
sudo apt install v4l2loopback-dkms v4l2loopback-utils
```

then we have to create a configuration file for this kernel module,which will tell the module to create one device `/dev/video10` with the name `VirtCam`. The `exclusive_caps` option makes the device viewable for only one application at once,this is necessary for Electron/Chrome based softwares.

```bash
sudo echo "options v4l2loopback devices=1 video_nr=10 card_label=\"VirtCam\" exclusive_caps=1 max_buffers=2" >> /etc/modprobe.d/v4l2loopback.conf
```

## Bundle everything together

And then the make it work we have to create a script to pipe the video feed from the camera to the virtual device.For this we will using a small bash script. Every time we want to use the camera as a webcam.

```bash
#/bin/bash

gphoto2 --stdout --capture-movie | ffmpeg -i - -f rawvideo -pix_fmt yuv420p -threads 0  -s:v 1056x704 -r 25 -f v4l2 /dev/video10
```
of course at the end of the script tailor the resolution and the framerate as well for your own devices.


1. Make sure DSLR connected via USB
2. Turn on DSLR
3. start script to webcam-script.

A extra note here, with this VirtCam device you could even play video,but beware it has to be a nice mp4 file to not cause any problem.
If you have to `v4l2loopback` device you'll be able to get even feed from ipcams as well.


## Conclusion

This is not a perfect solution. I would say this is a temporary solution. In case you are looking for a long term solution ,you probably have to buy a proper webcam or if your DSLR capable of the HDMI output you should check on `Elgato Camlink`. That device might be bit expensive around $100 at the moment, however the quality of the video feed is as you would expect from your DSLR and your PC will see the camera as a Generic USB webcam.

Good luck everyone
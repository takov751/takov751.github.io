---
layout: post
title:  "Raspberrypi used as wireless UART"
date:   2020-04-15 10:00:00 +0000
categories: raspberrypi
author: takov751
published: true
---

I started a project to see ,if i could reflash my old ISP modem/router with Openwrt , however for that i have to solder a few cables to the PCB and I need something to connect to the router UART port. Well i have several small IoT device which i could use,however feature richness and stability wise my old `Raspberry pi 2B v1.2` more than enough. __This small trick can be used with the other versions as well !__

First of all we will go full headless in this setup. This way i'll be able to use this setup even from a small powerbank , in case i need this on the go.

## 1. Burn the latest Raspbian Lite version to a microSD card


## 2. Before you remove the SD card after burning it. Create a file called SSH on the Boot partition

## 3. Create a configuration file for the wireless connection on the same boot partition [source](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md    )
filename `wpa_supplicant.conf`:

```bash
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=<Insert country code here>

network={
 ssid="<Name of your WiFi>"
 psk="<Password for your WiFi>"
}
```
## 4. We have to gain access to the serial port

By default Raspbian uses serial port as output,to disable it. At first start when the device connected to the wireless network SSH into the Rpi box. 

start `sudo raspi-config` Select option 5, Interfacing options, then option P6, Serial, and select No. Exit raspi-config.
Open `sudo nano /boot/cmdline.txt` remove `console=serial0,115200`

After reboot you will have access to the serial port on the path `/dev/ttyAMA0`.
you can add yourself to the dialout group with `sudo usermod -aG dialout $USER`

## 5. Install application to communicate

screen , picocom , stty its your choice.
From this point you have a static UART jumpbox wirelessly wherever you would like.
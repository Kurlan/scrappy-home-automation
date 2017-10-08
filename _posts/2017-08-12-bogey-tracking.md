---
layout: post
title: "Bogey Tracking"
date:   '2017-08-12'
comments: true
---

## Goal
This is my dog Bogey
{% include images.html name="bogey.png" caption="Bogey" %}

Bogey is a great dog, but he's hard to find in my house.  My next project is to track Bogey by putting a  `Bluetooth Low Energy (BLE)` device on his collar.  Using a `Bluetooth` receiver, like the one on a Raspberry Pi we can measure the relative signal strength of the BLE device to the receiver.  An array of these receivers can resolve the floor and room of the house the device is.  Because the relative signal strength of these devices has high variance, we will use machine learning techniques to improve the prediction of which room the device resides.  I will then use the new Amazon Echo binding with `openHAB 2.1` to allow me to ask my Echo where my dog is.

The goal of this post is to make sure your host's Bluetooth receiver is working correctly and to install `bluepy`, a Python library that allows us to interface with Bluetooth programmatically.

## What you need
* A `Raspberry Pi` with a `Bluetooth` adapter (A `Raspberry Pi 3 Series B` has one built in)
* A `Bluetooth LE` tracking device

## Ensure the Bluetooth receiver is working
`hcitool` is used to configure Bluetooth connections and send some special command to Bluetooth devices. The first step is to verify the `hcitool` installation. You should see an entry like the following after running `hcitool dev`

```
pi@raspberrypi:~ $ hcitool dev
Devices:
    hci0    B8:27:EB:44:3A:DC
```

Then we will do a scan!  The output is below.  The first entry is the MAC address of the receiver.  MAC addresses are for every device.  The second column is the advertised name.  Many devices do not identify themselves without pairing.  Our `BTLE` devices will usually identify themselves with this scan.  Here are I have three different devices.  This scan will pick up not only `BTLE` devices.  It will also pick up your phone if Bluetooth is on, as well as things like wireless keyboards and headphones.  

```
 sudo hcitool lescan

 LE Scan ...
D0:B5:C2:CF:39:B1 E-Beacon_CF39B1
F1:47:8D:FF:EC:17 VSF
C8:A9:20:FE:BB:DC MiniBeacon_54301
```
Next we will install `bluepy` so we can get programmatic access to our Bluetooth receiver.

```
$ sudo apt-get install python-pip libglib2.0-dev
$ sudo pip install bluepy
```

We will now create a simple `python` program that will print out the *Received signal strength indication* (`rssi`), for all `Bluetooth` device addresses passed into it on the command line.  We will name it `btle_rssi.py`.  It spends 4.5 seconds scanning for devices.  

```
import sys
import time

from bluepy.btle import Scanner, DefaultDelegate

class ScanDelegate(DefaultDelegate):
    def __init__(self):
        DefaultDelegate.__init__(self)

scanner = Scanner().withDelegate(ScanDelegate())
devices = scanner.scan(4.5)

deviceMap = {}

inputList = sys.argv[1:]
devNumber = 1

for deviceId in inputList:
    deviceMap[deviceId] = devNumber
    devNumber += 1

for dev in devices:
    for deviceId in inputList:
       if deviceId.lower() == dev.addr:
            print("rssi.device%d %d %d" % (deviceMap[deviceId], dev.rssi, time.time()))

```

To run it issue the following command.  You can specify any number of mac_addresses with this program.

```
  sudo python btle_rssi.py [mac_address]

    >> rssi.device1 -42 1504584026
```

Congratulations!  You can now do a `Bluetooth` scan and programmatically measure the `rssi` of any device within range.

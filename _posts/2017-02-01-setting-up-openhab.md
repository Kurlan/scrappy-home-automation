---
layout: default
title: "Installing OpenHAB"
date:   2017-02-01 18:01:00 -0800
---
# Installing openHAB

## Goal
The goal for this post is to install openHAB 2.0 beta

## What you need
* A Raspberry Pi with Raspbian installed and ssh access

## What is openHAB
There is a fundamental problem with the current state of home automation--each *smart* device connects to their own cloud and their own smartphone app.  Enter [openHAB](http://www.openhab.org/), an open-source home automation hub with dozens of plug-ins. openHAB is written in java so it can run on any computer that has a Java Virtual Machine (JVM). openHAB, at the time of this post, is at version 2.0.

## Install the software
Shell into your box then

```shell
sudo apt-get update  #This will take about 30 seconds
sudo apt-get upgrade  #This will take several minutes.  Go get a beer
``` 

### Install Oracle Java 8
```shell
echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | sudo tee /etc/apt/sources.list.d/webupd8team-java.list
echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | sudo tee -a /etc/apt/sources.list.d/webupd8team-java.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886
sudo apt-get update
sudo apt-get install oracle-java8-installer
sudo apt-get install oracle-java8-set-default
```

You should see the default java set to 1.8

```shell
pi@raspberrypi:~ $ java -version
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) Client VM (build 25.121-b13, mixed mode)
```

### Install vi
```shell
sudo apt-get install vim
```

### Install the openHAB repo

```shell
wget -qO - 'https://bintray.com/user/downloadSubjectPublicKey?username=openhab' | sudo apt-key add -
echo 'deb http://dl.bintray.com/openhab/apt-repo2 stable main' | sudo tee /etc/apt/sources.list.d/openhab2.list
```

Synchronize once more

```shell
sudo apt-get update
```

### Install openHAB
Now you can finally install.  We will also install all add-ons to avoid having to install them later by hand

```shell
sudo apt-get install openhab2
sudo apt-get install openhab2-addons
```

After the installation there is the following message

```shell
### NOT starting on installation, please execute the following statements to configure openHAB to start automatically using systemd
 sudo /bin/systemctl daemon-reload
 sudo /bin/systemctl enable openhab2.service
### You can start openhab2 by executing
 sudo /bin/systemctl start openhab2.service
```

Running the three commands will ensure that if power is lost openHAB will restart, and will also start openHAB.  Going to `http://raspberrypi:8080` (or the hostname/IP of your Pi) in a browser should load the following page:

Go ahead and click Standard.  Congratulations, you've installed openHAB 2.0!

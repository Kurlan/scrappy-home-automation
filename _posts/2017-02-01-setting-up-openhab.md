---
layout: post
title: "Installing OpenHAB"
date:   '2017-02-01T18:01:00+0800'
comments: true
---
## Goal
The goal for this post is to install openHAB 2.0

## What you need
* A Raspberry Pi with Raspbian installed and `ssh` access

## What is openHAB
There is a fundamental problem with the current state of home automation--each *smart* device connects to their own cloud and their own smartphone app.  Enter [openHAB](http://www.openhab.org/), an open-source home automation hub with dozens of plug-ins. openHAB is written in java so it can run on any computer that has a Java Virtual Machine (JVM). openHAB, at the time of this post, is at version 2.0.

## Install the software
`apt-get` is the package management system for Debian based linux distributions like Raspbian.  The first thing we shuold do is update and upgrade the package management system.

```shell
sudo apt-get update  #This will take about 30 seconds
sudo apt-get upgrade  #This will take several minutes after a fresh install.  Go get a beer
``` 

### Install Oracle Java 8
openHAB 2 requires java 8 to be installed.  We will install Oracle Java 8 and ensure it is the default java for the system.  First though, we have to add the appropriate keys to authenticate the java packages.

```shell
# Update keystore
echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | sudo tee /etc/apt/sources.list.d/webupd8team-java.list
echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | sudo tee -a /etc/apt/sources.list.d/webupd8team-java.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886

# Update
sudo apt-get update

# Install
sudo apt-get install oracle-java8-installer
sudo apt-get install oracle-java8-set-default
```

You should see the default java set to 1.8 when you run `java -version`

```shell
pi@raspberrypi:~ $ java -version
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) Client VM (build 25.121-b13, mixed mode)
```

### Install the openHAB repository
We need to add openHAB to our `apt-get` repository.  

```shell
wget -qO - 'https://bintray.com/user/downloadSubjectPublicKey?username=openhab' | sudo apt-key add -
echo 'deb http://dl.bintray.com/openhab/apt-repo2 stable main' | sudo tee /etc/apt/sources.list.d/openhab2.list
```

Synchronize once more.

```shell
sudo apt-get update
```

### Install openHAB
Now we can finally install openHAB with `apt-get`.

```shell
sudo apt-get install openhab2
```

After the installation there is the following message

```shell
### NOT starting on installation, please execute the following statements to configure openHAB to start automatically using systemd
 sudo /bin/systemctl daemon-reload
 sudo /bin/systemctl enable openhab2.service
### You can start openhab2 by executing
 sudo /bin/systemctl start openhab2.service
```

Running the three commands will ensure that if power is lost the openHAB service will restart, and will also start openHAB.  Going to `http://raspberrypi:8080` (or the hostname/IP of your Pi) in a browser should load the following page:

{% include images.html name="openhab_initial_screen.png" caption="We have lift off!" %}

Go ahead and click `Standard`.  You should now see:

{% include images.html name="openhab_start_screen.png" caption="You should see three different choices for UI" %}

### Install add-ons
To avoid having to install all of the add-ons manually you can install them all at once now.

```shell
sudo apt-get install openhab2-addons
```
Congratulations, you've installed openHAB 2.0!

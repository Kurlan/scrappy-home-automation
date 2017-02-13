---
layout: post
title: "Installing the Garage system"
date:   '2017-02-10T17:16:00-0800'
comments: true
---
## Goal
The goal for this post is to install our relay and sensor in the garage

## What you need
* The relay and sensor from the previous post
* A project board and some solder

## Creating a project board
It's time to transfer the relay and sensor wiring to a project board. Here's what ours looks like

{% include images.html name="project_board.png" %} 

After creating the project board our components look like this.

{% include images.html name="garage_components.png" caption="We used a double relay because the single relay we ordered was defective and that's all we had.  A bit a of a waste though." %} 

## Install

### Install the relay
The next step was to connect to relay to our garage door system.  Here's what mine looks like after I popped the panel.  

{% include images.html name="garage_panel.png" %} 

Create a couple of wire splices.  Connect one end to your existing switch and one to your relay.

{% include images.html name="garage_splice.png" %} 

You should now be able to control the garage through your sitemap.

### Install the sensor
You will need to find a place on your existing garage door where you can install a sensor.  I was able to find a spot near the top of my garage door.  I needed to drill a hole through some sheet metal.

{% include images.html name="garage_sensor.png" %} 

You should now be able to open and close the door and notice that the sensor has switched from `OPEN` to `CLOSED`.

Congratulations!  You have now installed a web-enabled Garage door opener!

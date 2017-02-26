---
layout: post
title: "Binding a Relay"
date:   '2017-02-05'
comments: true
---
## Goal
The goal for this post is to do a bench test for an LED, a relay, and then bind it to openHAB

## What you need
* Breadboard
* Resistor, transistor, diode and LED
* <a href="http://amzn.to/2m9ylvX"> Relay</a>

## Bench testing an LED
First we will wire an LED into our GPIO ports and test it via the command line.  Wire up an LED to your breadboard as such (use GPIO17)
{% include images.html name="led_breadboard.png" caption="Use GPIO 17" %} 

Next, fire up your terminal and let's get the light switching on and off:

```shell
# Set up GPIO 17 and set to output
echo "17" > /sys/class/gpio/export
echo "out" > /sys/class/gpio/gpio17/direction
```

We've now setup our pin for output.  Let's set the value

```shell
echo "1" > /sys/class/gpio/gpio17/value
```

The light should turn on.

```shell
echo "0" > /sys/class/gpio/gpio17/value
```

The light should turn off.

Afterwards, we should unexport our pin so other processes can use it:

```shell
echo "17" > /sys/class/gpio/unexport
```

## Binding the LED to openHAB
We simply need to add an item to our `garage.items` file (`sudo -u openhab vim /etc/openhab2/items/garage.items`)

```
Switch GarageDoorSwitch "Garage Door" <switch> (GarageDoor) {gpio="pin:17 force:true activelow:yes")
```

If we reload our sitemap page (`http://raspberrypi:8080/basicui/app?sitemap=myhouse`) we should be able to control our LED there.

## Creating a relay
We followed the instructions of [this blog post](http://www.susa.net/wordpress/2012/06/raspberry-pi-relay-using-gpio/) to drive our garage door.  I created the following wiring diagram based on that post:

{% include images.html name="garage_relay.png" %}

The basic idea is that the GPIO pin will control the transistor.  When the transisitor will control the current to the relay.  After wiring this up you should be able to hear a nice click on your relay when you press the button on your sitemap.

## Creating a momentary switch
The last step is to make our switch momentary.  We will do this via openHAB rules.  Create a rules file `sudo -u vim openhab /etc/openahb2/rules/garagedoor.rules` with the following content:

```
var SLEEP_TIME = 1000

rule "Garage Door is a momentary switch"
        when
                Item GarageDoorSwitch received command
        then
		if (receivedCommand == OFF) {
                	Thread::sleep(SLEEP_TIME)
                	sendCommand(GarageDoorSwitch,ON)
		}
end
```

Here we basically create a rule that when it the `GarageDoorSwitch` received a command we sleep for `SLEEP_TIME` (`1000` milliseconds or 1 second in this case) and then turn the switch back ON.  Going back to the sitemap, when we hit our switch you should now hear two clicks three seconds apart.

### Modifying the sitemap control
Since there's essentially one state for our switch we will change it to a button control instead of a control with two states.  Open your sitemap file (`sudo -u openhab vim /etc/openhab2/sitemaps/myhouse.sitemap` and alter the garage section:

```
sitemap myhouse label="My home automation" {
        Frame label="Garage" {
                Text item=GarageDoorClosedSensor
                Switch item=GarageDoorSwitch mappings=[OFF="Press the button"]

        }
}
```

After reloading the sitemap you should now see a button instead of a slider switch.

{% include images.html name="momentary_switch.png" %}

Congratulations!  You've just created a momenty switch that controls a relay and bound it to openHAB.

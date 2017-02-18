---
layout: post
title: "Automating the Door and Pushover Notifications"
date:   '2017-02-16'
comments: true
---
## Goal
So it's all well and nice that we can control the garage door with openHAB but I wouldn't call our setup "smart" or even "automated".  All we've done is basically create a remote control for our garage.  We can make it cooler.  

We have a large problem in my household though.  We are really bad at closing the garage door.  We once left on vacation with the door wide open and we had to have our neighbors call and close it for us.  So in this post we will create some rules that will notify us when the garage door is open for more than a couple of hours.  On top of this we will code in a rule that will automatically close the door when it is open for too long.

## What you need
* An automated garage door as outlined in the previous blog posts

## Pushover
[Pushover](https://pushover.net/) is a simple notificatins framework with a binding in openHAB. After a free seven day trial it will cost $5 to send notifications for the rest of your life, so its worth it. Integration will be relatively straight-forward.  First, we will download the mobile app on our phones.  Next, we will register our openHAB application with Pushover and install the Pushover binding.

### Install the mobile client and register your application
First things first, you will need to download the mobile application to receive notifications.  You can find links [here](https://pushover.net/clients). You can sign up for an account after the app has downloaded. You will need the User token for the next section.

After you have verified your email you can register a new application at <a href="https://pushover.net">pushover.net</a>.  You will need the API token key in the next section.

### Installing and configuring the Pushover add-on
Go to your `PaperUI` (`http://raspberrypi:8080/start/index`) and go to the `Add-ons` section.  Click on the `Actions` tab and install the `Pushover`.

The Pushover configuration file will be located at `/etc/openhab2/services/pushover.cfg`.  Edit this file (`sudo -u openhab vim /etc/openhab2/services/pushover.cfg`).  We will need to add values to the following keys:

```
defaultUser=<YOUR USER KEY FROM https://pushover.net/>
defaultToken=<YOUR API KEY FROM https://pushover.net/>
defaultTitle=openHAB
```

### Sending notifications
Now we can test the notifications we've set up with our existing rule from a previous post.  We can add a new rule our rule set to send a notification so that we can test.  Open up our `garagedoor.rules` configuration file (`sudo -u openhab vim /etc/openhab2/rules/garagedoor.rules`) and add the following rule:

```
rule "Garage Door sensor changed state"
        when
                Item GarageDoorClosedSensor changed
        then
                pushover("The Garage door state has been changed")
        end
```

When you got to your openHAB page (http://raspberrypi:8080/basicui/app?sitemap=myhouse) and open and close the garage door you should receive two notifications on your phone, one for the opening and one when it closed.

## Creating useful rules
Now that we have tested pushover it's time to create some useful rules.  The first rule I would like to create is to send me a notification if the garage door has been opened for some extended period of time.  This will allow me to log in and close the garage door if I didn't intend to keep it open for that long.  The rule below meet this requirement:

```
import org.openhab.model.script.actions.Timer
var Timer extended_timer

var EXTENDED_GARAGE_DOOR_TIME_SECONDS = 10

rule "Garage Door open for extended period of time sends notification"
        when
                Item GarageDoorClosedSensor changed
        then
                if (GarageDoorClosedSensor.state == OPEN) {
                        extended_timer = createTimer(now.plusSeconds(EXTENDED_GARAGE_DOOR_TIME_SECONDS)) [|
                                pushover("Garage door has been open for an extended period of time");
                        ])
                } else {
                        if(extended_timer!=null) {
                                extended_timer.cancel
                                extended_timer = null
                        }
                }
        end
```

After saving the file you should receive an alert 10 seconds after the door opens.  Once this has been verfied you can change the `EXTENDED_GARAGE_DOOR_TIME_SECONDS` variable to a reasonable number (I have set mine to `3600` seconds or one hour)

The second rule I'd like to create is to automatically close the door when it's been open an unusual amount of time.  This way, when we go on vacation I'll never have to worry that I've left the garage door open (which actually happened to us).  The rule is very similar to the previous rule:

```
import org.openhab.model.script.actions.Timer

var Timer unusual_timer

var UNUSUAL_GARAGE_DOOR_OPENED_SECONDS = 10

rule "Garage Door open for unusual period of time closes door and sends notification"
        when
                Item GarageDoorClosedSensor changed
        then
                if (GarageDoorClosedSensor.state == OPEN) {
                        unusual_timer = createTimer(now.plusSeconds(UNUSUAL_GARAGE_DOOR_OPENED_SECONDS)) [|
                                sendCommand(GarageDoorSwitch,ON)
                                pushover("Garage door has been open for an unusual period of time.  Door has been automaticaly closed");
                        ])
                } else {
                        if(unusual_timer!=null) {
                                unusual_timer.cancel
                                unusual_timer = null
                        }
                }
        end
```

We can test this again and this time, the garage door will shut on its own after ten seconds.  Again, change this value to something reasonable (I have this set to `7200` seconds or two hours).

Combining all rules together here is what my `garagedoor.rules` file looks like:

```
import org.openhab.model.script.actions.Timer

var Timer extended_timer
var Timer unusual_timer

var EXTENDED_GARAGE_DOOR_TIME_SECONDS = 3600
var UNUSUAL_GARAGE_DOOR_OPENED_SECONDS = 7200

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

rule "Garage Door open for extended period of time sends notification"
        when
                Item GarageDoorClosedSensor changed
        then
                if (GarageDoorClosedSensor.state == OPEN) {
                        extended_timer = createTimer(now.plusSeconds(EXTENDED_GARAGE_DOOR_TIME_SECONDS)) [|
                                pushover("Garage door has been open for an extended period of time");
                        ])
                } else {
                        if(extended_timer!=null) {
                                extended_timer.cancel
                                extended_timer = null
                        }
                }
        end

rule "Garage Door open for unusual period of time closes door and sends notification"
        when
                Item GarageDoorClosedSensor changed
        then
                if (GarageDoorClosedSensor.state == OPEN) {
                        unusual_timer = createTimer(now.plusSeconds(UNUSUAL_GARAGE_DOOR_OPENED_SECONDS)) [|
                                sendCommand(GarageDoorSwitch,OFF)
                                pushover("Garage door has been open for an unusual period of time.  Door has been automaticaly closed");
                        ])
                } else {
                        if(unusual_timer!=null) {
                                unusual_timer.cancel
                                unusual_timer = null
                        }
                }
        end
```

Congratulations!  You've created a smart garage and will never unintentionally leave your door open ever again!

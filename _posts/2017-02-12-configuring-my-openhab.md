---
layout: post
title: "Configuring my.openHAB"
date:   '2017-02-12T17:16:00-0800'
comments: true
---
## Goal
The goal for this post is to properly configure my.openHAB to access your sitemap without being on the same wi-fi network. my.openHAB is a hosted service that allows us to access our home-automation from anywhere.

## What you need
* The sitemap for the garage system from previous posts

## my.openHAB registration
You will need to go to [myopenhab.org](myopenhab.org) and register for an account.  After you have done this check your email to confirm it.  Without confirmation you will not be able to use the features of my.openHAB.

## Install the openHAB Cloud Connector
Go to the `PaperUI` (`raspberrypi:8080/start/index`) and click on `Add-ons`.  Select the `MISC` column and click `INSTALL` on the `openHAB Clound Connector`.

## Setup persistence
Create a file called `myopenhab.persist` in the `persistence` file of your openHAB installation (`sudo -u openhab vim /etc/openhab2/persistence/myopenahb.persist`) with the following contents:

```
Strategies {
    default = everyChange
}
Items {
    * : strategy = everyChange
}
```

What we have done here is to say that we will update all of our persistences for every change.  This will allow my.openHAB to pick up our Item states every time they change.

## Provide your secrets to my.openHAB
my.openHAB requires two pieces of information, a UUID and a secret.  These values can be found by running the following two commands:

```shell
cat /var/lib/openhab2/uuid
> YOUR UUID PRINTED HERE
cat /var/lib/openhab2/openhabcloud/secret
> YOUR SECRET PRINTED HERE
```

Keep these values secret!  Go to the `Account` section of [myopenhab.org](myopenhab.org) (visible when you hover over your email) and click update.  Then, you should restart your openHAB service

```shell
sudo systemctl restart openhab2.service
```

After some time the tab on the header of [myopenhab.org](myopenhab.org) will go from `Offline` (shaded red) to `Online` (shaded green).  When this happens you can navigate to your sitemap remotely by going to `https://home.myopenhab.org/basicui/app?sitemap=myhouse`.  You can see the status of your garage door and control your garage door remotely.

{% include images.html name="my_openahb.png" %}

Congratulatons! You have configured my.openHAB to control your home automation from anywhere!


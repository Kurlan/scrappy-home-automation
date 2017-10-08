---
layout: post
title: "Carbon and Graphite"
date:   '2017-10-01'
comments: true
---
## Goal
We will install <a href="https://markinbristol.wordpress.com/2015/09/20/setting-up-graphite-api-grafana-on-a-raspberry-pi/">carbon</a> and graphite to log and access metrics about our devices.  This will allow us to emit the RSSI values in an easily accessible manner.

## What you need
* A `Raspberry Pi` with a `Bluetooth` adapter 
* A `Bluetooth LE` tracking device
* The scripts from the previous blog post

## Installing Carbon

```
 sudo apt-get install graphite-carbon
```

Next we will edit `/etc/default/graphite-carbon` to set the following value.

```
CARBON_CACHE_ENABLED=true
```

And start the carbon service.

```
sudo service carbon-cache start
```

No we will install required dependencies then the `graphite-api`

```
 sudo apt-get install python python-pip build-essential python-dev libcairo2-dev libffi-dev
 sudo pip install graphite-api
```

We will create a file `/etc/graphite-api.yaml` with the following content:

```
search_index: /var/lib/graphite/index
finders:
  - graphite_api.finders.whisper.WhisperFinder
functions:
  - graphite_api.functions.SeriesFunctions
  - graphite_api.functions.PieFunctions
whisper:
  directories:
    - /var/lib/graphite/whisper
carbon:
  hosts:
    - 127.0.0.1:7002
  timeout: 1
  retry_delay: 15
  carbon_prefix: carbon
  replication_factor: 1
```

Almost there!

```
 sudo apt-get install apache2
 sudo apt-get install apache2-dev
 sudo apt-get install libapache2-mod-wsgi
 sudo pip install mod_wsgi

```

Next create `/var/www/wsgi-scripts/graphite-api.wsgi`

```
   sudo mkdir -p /var/www/wsgi-scripts/
	 sudo vim /var/www/wsgi-scripts/graphite-api.wsgi
```

```
  # /var/www/wsgi-scripts/graphite-api.wsgi
	from graphite_api.app import app as application
```

```
  sudo mkdir -p /etc/apache2/sites-available
	sudo vim /etc/apache2/sites-available/graphite.conf
```

```
# /etc/apache2/sites-available/graphite.conf
	LoadModule wsgi_module /usr/lib/apache2/modules/mod_wsgi.so
	WSGISocketPrefix /var/run/wsgi
	Listen 8013
	<VirtualHost *:8013>

 WSGIDaemonProcess graphite-api processes=5 threads=5 display-name='%{GROUP}' inactivity-timeout=120
 WSGIProcessGroup graphite-api
 WSGIApplicationGroup %{GLOBAL}
 WSGIImportScript /var/www/wsgi-scripts/graphite-api.wsgi process-group=graphite-api application-group=%{GLOBAL}

 WSGIScriptAlias / /var/www/wsgi-scripts/graphite-api.wsgi

 <Directory /var/www/wsgi-scripts/>
 Order deny,allow
 Allow from all
 </Directory>
 </VirtualHost>
```

Create a symlink:

```
  sudo mkdir /etc/apache2/sites-enabled/
	cd /etc/apache2/sites-enabled/
	sudo ln -s ../sites-available/graphite.conf .
```

We just need to start our service.

```
service apache2 restart
```

## Checking our work
Navigate to your local host http://<hostname>:8013/render

You should see a nice
```
	{"errors": {"target": "This parameter is required."}}
```

Let's populate some metrics!  Create a file with the contents below.  We will call it `metricsLogger.py`:

```
import sys
import time
import socket

sock = socket.socket()
metric = sys.argv[1]
value = int (sys.argv[2])

sock.connect( ("localhost", 2003) )
sock.send("%s %d %d \n" % (metric, value, time.time()))
sock.close()
print("%s %d %d \n" % (metric, value, time.time()))
```

Then run the command

```
python metricsLogger.py testMetric 1
```

Now you can navigate to `http://<yourHost>:8013/render?target=test.metric` and see a graph with a single data point.

Congratulations!  You are now logging metrics and making them available with graphite and carbon!



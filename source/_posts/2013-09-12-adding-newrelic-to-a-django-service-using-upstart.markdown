---
layout: post
title: "Adding newrelic to a django service using upstart"
date: 2013-09-12 16:16
comments: true
categories: 
- technical 
- edx
---
Some context: I am running a production instance of edx’s platform. This means I am dealing here with django on a nginx + gunicorn + virtualenv stack. This is managed using upstart on ubuntu 12.04.2 LTS. 

Back to the main topic, in case you are trying to use gunicorn and newrelic on a somewhat similar configuration this experience I had today might be of some use.

<!-- more -->

As I said, the django service uses gunicorn as it’s WSGI Http server. To start the service I normally call: $ sudo start edxapp. This runs an upstart script that works using the next line:

```
exec /opt/edx/bin/gunicorn --preload -b 127.0.0.1:$PORT -w $WORKERS --timeout=300 --pythonpath=/opt/wwc/edx-platform lms.wsg
```

The script is located at `/etc/init/edxapp.conf`

I decided to start using new-relic to monitor this service, so I needed it to start using the new relic agent. The upstart scripts had some env variables for newrelic commented out, but setting them right did not go all the way to make me happy.

After some [search][1], some gessing and some trial and error I got it.

In short here it is:

5. Make sure you installed the new [relic agent][2]. Change your newrelic.ini as you need to and [test it][3] to be sure you got it right.
4. Define the environment variable inside the upstart script: `env NEW_RELIC_CONFIG_FILE=/opt/wwc/newrelic.ini`
3. Use the new relic agent admin to call the gunicorn process: `exec /opt/edx/bin/newrelic-admin run-program /opt/edx/bin/gunicorn --preload -b 127.0.0.1:$PORT -w $WORKERS --timeout=300 --pythonpath=/opt/wwc/edx-platform lms.wsgi`
2. `$ sudo start edxapp`

[1]: https://newrelic.com/docs/python/python-agent-and-gunicorn
[2]: https://newrelic.com/docs/python/python-agent-installation
[3]: https://newrelic.com/docs/python/testing-the-python-agent
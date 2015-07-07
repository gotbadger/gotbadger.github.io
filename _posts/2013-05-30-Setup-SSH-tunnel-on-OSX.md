---
layout: post
Title: Setup SSH tunnel on OSX
Date: 2013-05-30 8:00
tags: [Devops, Web, Programming]
image: /assets/images/osx.jpg
---

More often I'm working with 3rd party API's and suchlike. One problem that developers often encounter is how to deal with this in dev when they cant open up ports on their local machine. One solution is to use a service like [localtunnel](http://progrium.com/localtunnel/) this is great for quickly testing somthing but every time localtunnel looses connection you will get a different endpoint this means endlessly changing webhook links on API's.

So how can we do this better? Well suprisingly easily if you have access to a public facing linux server.
{% highlight bash %}
#!/bin/bash

# setup local
BIND_ADAPTER="en0"
LOCAL_PORT=4000

# setup remote
REMOTE_USER="user"
REMOTE_HOST="server.example.com"
REMOTE_PORT=9999

echo "Make sure $REMOTE_PORT is open on $REMOTE_HOST and 'GatewayPorts yes' is set in sshd.conf"

LOCAL=$(ifconfig $BIND_ADAPTER | grep 'inet ' | cut -d " " -f2 | awk '{print $1}')
echo "$REMOTE_HOST:$REMOTE_PORT -> $LOCAL:$LOCAL_PORT"
ssh $REMOTE_USER@$REMOTE_HOST -N -R $REMOTE_PORT:$LOCAL:$LOCAL_PORT
{% endhighlight %}
As indicated by the script you have to open the port on your servers firewall and add the following to sshd.conf
{% highlight bash %}
GatewayPorts yes
{% endhighlight %}
**BIND_ADAPTER** might vary depending on your setup, use ipconfig to find the adapter your lan IP is bound to. Everything else should be fairly self explanatory.

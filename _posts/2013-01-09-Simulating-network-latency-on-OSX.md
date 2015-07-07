---
layout: post
Title: Simulating network latency on OSX
Date: 2013-01-09 18:00
tags: [Devops, Web, Programing]
dark: false
image: /assets/images/osx.jpg
---
When developing websites and such its good to know how loading will perform for users with slow/laggy connections. On FreeBSD or OSX you can model this using ipfw.

{% highlight bash %}
sudo ipfw pipe 1 config bw 50KBytes/s delay 300ms
sudo ipfw add 1 pipe 1 src-port 4000 && sudo ipfw add 2 pipe 1 dst-port 4000
{% endhighlight %}

This will add a 300ms latency and a 50kBytes/s bandwidth restriction to our app running on localhost:4000. Finaly we can delete the restrictions using the following command to delete the pipes we setup.

{% highlight bash %}
sudo ipfw delete 1 && sudo ipfw delete 2
{% endhighlight %}

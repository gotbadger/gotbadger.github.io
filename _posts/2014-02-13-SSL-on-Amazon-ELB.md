---
layout: post
title: "Comodo Instant SSL Certs on Amazon ELB"
modified: 2013-08-03
tags: [Devops, AWS]
image: /assets/images/aws.jpg
---

Getting SSL certs working always ends up being a case of knowing what *generaly* is required but the specific quirks of each platfrom end up throwing a spanner in the works. Hopefully this will help somone in getting things working.

##Requirements

I'm going to assume you have sucessfully got a SSL cert from [http://www.instantssl.com/](http://www.instantssl.com/) download the zip they provide in it you should find the following:

   * Site.ca-bundle
   * Site.crt

In additon to this you will need the private key you used to generate the signing request

   * Site.key

## Configuring ELB

Load ballancers are in the EC2 section in the AWS managment console, locate your ballancer and select it. Next click on the Listeners tab. We will need to add a new listener set the protocol to HTTPS and since we will be doing SSL Termination at the ballancer make sure the instance protocol is straight HTTP. Next click on SSL Certificate and click 'Select'.

![ELB SSL setup screen][1]

Set the certificate name to some reference you will use to identify this certificate setup

Run the following command to generate the data for private key:

**NB:** I'm using pbcopy to send the output to the clipboard but you can just cut and paste from a terminal if need be
{% highlight bash %}
	openssl rsa -in Site.key | pbcopy
  {% endhighlight %}
Run the following to generate the data for Public Key Certificate, amazon requires a certain format so run though openssl:
{% highlight bash %}
	openssl x509 -inform PEM -in Site.crt | pbcopy
  {% endhighlight %}
Finally add certificate chain data from site.ca-bundle:
{% highlight bash %}
	cat Site.ca-bundle | pbcopy
  {% endhighlight %}
Click Save and if everything was valid you should now have a working ssl setup. you can use one of the many online tools such as [http://www.digicert.com/help/](http://www.digicert.com/help/) to make sure the SSL setup is completely valid.

[1]: http://dl.dropbox.com/u/78443198/apps/scriptogram/aws-ssl.png

---
layout: post
title: Automatic forwarder configuration
tags: ["freebsd", "named", "bind"]
---
Since FreeBSD 7.3-RELEASE, it has been possible to
automatically setup a forwarding nameserver simply by adding

{% highlight sh %}
named_enable="YES"
named_auto_forward="YES"
{% endhighlight %}

to /etc/rc.conf.  This allows you to utilize a local resolver for
better performance, and decreased network traffic while still
relying on the benefits of your local network resolver.

When named is started, a forwarder configuration file is generated
by using the the contents of /etc/resolv.conf

Modify /etc/resolv.conf so that the first nameserver queried is 127.0.0.1
which will be the caching nameserver running on the local system.

{% highlight sh %}
search example.com
nameserver 127.0.0.1
nameserver 8.8.8.8
nameserver 208.67.222.222
nameserver 208.67.220.220
{% endhighlight %}

The file will be saved as /etc/namedb/auto_forward.conf. Make sure this
file is included by uncommenting the following in in /etc/named.conf

{% highlight sh %}
include "/etc/namedb/auto_forward.conf";
{% endhighlight %}


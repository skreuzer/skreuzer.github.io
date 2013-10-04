---
layout: post
title:  Stepping the system clock during boot
---
FreeBSD allows you to perform an instantaneous change to your system clock while
the host is booting up no matter how great the difference between a machine's
current clock setting and the correct time.

To enable, add the following to /etc/rc.conf

	ntpdate_enable="YES"
	ntpdate_hosts="north-america.pool.ntp.org"



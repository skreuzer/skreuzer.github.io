---
title:  Stepping the system clock during boot
date: 2011-12-19
tags: [freebsd]
---
FreeBSD allows you to perform an instantaneous change to your system clock while
the host is booting up no matter how great the difference between a machine's
current clock setting and the correct time.

Use the `sysrc` command as root to append ntpdate_enable and ntpdate_hosts to
/etc/rc.conf

    $ sysrc ntpdate_enable=YES
    ntpdate_enable: NO -> YES
    $ sysrc ntpdate_hosts=north-america.pool.ntp.org
    ntpdate_hosts:  -> north-america.pool.ntp.org

Finally, to verify that everything worked correctly

    $ grep ntpdate /etc/rc.conf
    ntpdate_enable="YES"
    ntpdate_hosts="north-america.pool.ntp.org"

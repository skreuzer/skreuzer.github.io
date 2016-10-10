---
title: Running csup from periodic
---
I keep a copy of the FreeBSD source and ports repository locally on disk so its
possible for me to work offline and still be able to review the revision history
of a file or views diffs. I have been running csup from the root crontab at
night to keep my local copy fairly up to date.

However, I rewrote the script to be a job that is executed by periodic and then
have the output of csup included with the daily run output emails I
get each morning so I can quickly see the changes to the ports tree.

Place a copy of 600.csup into /usr/local/etc/periodic/daily (you may have to
create this directory if it does not exist)

{% highlight sh %}
#!/bin/sh
#
# $FreeBSD$
#

# If there is a global system configuration file, suck it in.
#

if [ -r /etc/defaults/periodic.conf ]
then
    . /etc/defaults/periodic.conf
    source_periodic_confs
fi

case "$daily_csup_enable" in
    [Yy][Ee][Ss])
    if [ -z "$daily_csup_supfile" ]
    then
        echo '$daily_csup_enable is set but' \
            '$daily_csup_supfile is not'
        rc=2
    else
        if [ -z "$daily_csup_binary" ]
        then
            daily_csup_binary=/usr/bin/csup
        fi

        if [ ! -x "$daily_csup_binary" ]
        then
            echo '$daily_csup_binary is set but ' \
            $daily_csup_binary 'is not executable'
            rc=2
        else
            out=`$daily_csup_binary $daily_csup_supfile`
            rc=$?
            echo "$out"
        fi

    fi;;

    *)  rc=0;;
esac

exit $rc
{% endhighlight %}

To enable the script, add the following to /etc/periodic.conf.local (which may
need to be created if it does not exist)

{% highlight sh %}
daily_csup_enable="YES"
daily_csup_supfile="/home/skreuzer/cvsup/freebsd-cvs-supfile"
{% endhighlight %}

One thing to keep in mind is that all scripts the get executed by periodic daily
are run at 3:01am localtime so it will cause a huge spike in traffic if you have
lots of machines connecting to the same csup server all at the same time.

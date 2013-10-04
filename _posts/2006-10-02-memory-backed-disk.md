---
layout: post
title: Creating a File-Backed Disk with mdconfig
tags: ["freebsd"]
---
{% highlight bash %}
$ dd if=/dev/zero of=newimage bs=1k count=5k
5120+0 records in
5120+0 records out
$ mdconfig -a -t vnode -f newimage -u 0
$ bsdlabel -w md0 auto
$ newfs md0a
/dev/md0a: 5.0MB (10224 sectors) block size 16384, fragment size 2048
        using 4 cylinder groups of 1.25MB, 80 blks, 192 inodes.
super-block backups (for fsck -b #) at:
 160, 2720, 5280, 7840
$ mount /dev/md0a /mnt
$ df /mnt
Filesystem 1K-blocks Used Avail Capacity  Mounted on
/dev/md0a       4710    4  4330     0%    /mnt
{% endhighlight %}

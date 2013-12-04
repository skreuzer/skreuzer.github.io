---
layout: post
title: Zeroing Spare Disks on a NetApp
tags: ["netapp"]
---
Having spare disks already zeroed will reduce the amount of time creating a
new volume or rebuilding a raid set

{% highlight sh %}
toaster> vol status -s

Pool1 spare disks (empty)

Pool0 spare disks

RAID Disk       Device  HA  SHELF BAY CHAN Pool Type  RPM  Used (MB/blks)    Phys (MB/blks)
---------       ------  ------------- ---- ---- ---- ----- --------------    --------------
Spare disks for block checksum
spare           0c.19   0c    1   3   FC:A   0  FCAL 15000 272000/557056000  274845/562884296 (not zeroed)
spare           16b.17  16b   1   1   FC:B   0  FCAL 15000 272000/557056000  274845/562884296 (not zeroed)
spare           0b.44   0b    2   12  FC:A   0  FCAL 15000 272000/557056000  280104/573653840
spare           16c.40  16c   2   8   FC:B   0  FCAL 15000 272000/557056000  280104/573653840 (not zeroed)
spare           16d.91  16d   5   11  FC:B   0  FCAL 15000 272000/557056000  280104/573653840
toaster> disk zero spares
toaster> aggr status -s

Pool1 spare disks (empty)

Pool0 spare disks

RAID Disk       Device  HA  SHELF BAY CHAN Pool Type  RPM  Used (MB/blks)    Phys (MB/blks)
---------       ------  ------------- ---- ---- ---- ----- --------------    --------------
Spare disks for block checksum
spare           0c.19   0c    1   3   FC:A   0  FCAL 15000 272000/557056000  274845/562884296 (zeroing, 2% done)
spare           16b.17  16b   1   1   FC:B   0  FCAL 15000 272000/557056000  274845/562884296 (zeroing, 2% done)
spare           0b.44   0b    2   12  FC:A   0  FCAL 15000 272000/557056000  280104/573653840
spare           16c.40  16c   2   8   FC:B   0  FCAL 15000 272000/557056000  280104/573653840 (zeroing, 2% done)
spare           16d.91  16d   5   11  FC:B   0  FCAL 15000 272000/557056000  280104/573653840
{% endhighlight %}

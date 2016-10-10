---
title: Determine what drive is referenced using the stats show disk command
---
Here is what a portion of the output from the `stats show disk` command
looks like:

disk:20000020:37F352F1:00000000:00000000:00000000:00000000:00000000:00000000:00000000:00000000:disk_busy:0%

To determine what drive is being referenced above, copy the last 6 digits of
the second set of numbers (i.e. F352F1) Run the `storage show disk -a` command
and look for those digits in the WWN:

    Disk: 7a.21
    Shelf: 1
    Bay: 5
    Serial: <serial number>
    Vendor: NETAPP
    Model: X234_ST336704FC
    Rev: NA42
    RPM: 10000
    WWN: 2:000:002037:f352f1
    Downrev: no
    Pri Port: B
    Sec Name: 8a.21
    Sec Port: A
    Power-on Hours: 19873
    Blocks read: 113349936
    Blocks written: 11408
    Time interval: 80:19:53
    Glist count: 0
    Scrub last done: 00:01:27
    Scrub count: 163
    Dynamically qualified: No

Reference the drive number above to determine what drive is being referenced
(in this case our drive is 7a.21)

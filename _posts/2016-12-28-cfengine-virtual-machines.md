---
title: Detecting Virtual Machines in CFEngine
---
Out of the box CFEngine has some support for detecting if it is running on top
of a hypervisor. Unfortunately, it appears that a
[regression](https://tracker.mender.io/browse/CFE-1563) was introduced in 3.6.0
which prevented cf-agent from detecting Xen and this functunality appears to
have has never worked on [FreeBSD](https://tracker.mender.io/browse/CFE-2203)
as well.

I submitted a [pull request](https://github.com/cfengine/core/pull/2746) which
addresses the issue documented in CFE-1563 but it will not be included in
CFEngine 3.10 and right now only fixes the issue on Linux. If you are a FreeBSD
user and install cfengine via the
[sysutils/cfengine39](https://www.freshports.org/sysutils/cfengine39/) port I
have added a local patch in
[r428081](https://svnweb.freebsd.org/ports?view=revision&revision=428081) to
work around the issue documented in CFE-2203. My goal is to eventually get this
patch pushed back upstream as well.

While it is frustrating that this functionality has been broken for so long,
assuming that the host you are running on is not going out of its way to hide
the fact that its virtualized it seems like much of the code in
libenv/sysinfo.c to detect various hypervisors can rewritten as policies which
make it much easier to extend. In addition, these examples provided below are a
bit more portable, detect additional hypervisors and should work across a wider
range of platforms as well.

### Detecting virtualized hosts through Vendor OUIs

The MAC address is a unique identifier for network interfaces. It is a 48-bit
number with the first 24 bits being the Organizationally Unique Identifier. The
OUI is assigned via the IEEE and uniquely identifies the vendor or
manufacturer. CFEngine will collect the mac addresses of all network interfaces
it detects and populate them in an array called `$(sys.hardware_mac)`. Cfengine
will also set the variable `$(sys.interface)` to what it believes is the main
system interface on the host. Using these two variables we have an operating
system agnostic method to look at the first 3 octets of a mac address and see
if they match any known OUIs used in different hypervisor products.

The `string_head()` function returns the first 8 characters of the primary mac
address which we are referencing as `$(sys.hardware_mac[$(sys.interface)])`.
The `string_downcase()` function is used to convert it to lower case and after
that `strcmp()` is used to iterate through all the keys in the `$(vm_ouis)`
data container looking for a match. If one is found a class based on the value
of the matching key will be defined.

```
bundle agent detect_guest_oui
{
    vars:

        "mac_address" string => string_downcase(string_head("$(sys.hardware_mac[$(sys.interface)])", "8"));

        "vm_ouis" data => '{
            "00:05:69": "vmware",
            "00:0c:29": "vmware",
            "00:1c:14": "vmware",
            "00:50:56": "vmware",
            "00:15:5d": "hyper_v",
            "00:16:3e": "xen_domu",
            "58:9c:fc": "bhyve",
        }';

        "vm_oui" slist => getindices("vm_ouis");

    classes:

        "found_vm_mac" expression => strcmp("$(mac_address)", "$(vm_oui)");

        "$(vm_ouis[$(mac_address)])" expression => "found_vm_mac",
            scope => "namespace";
}
```

### Detecting virtualized hosts through Linux's /proc filesystem

The procfs filesystem on Linux is a special filesystem which presents various
information about running processes. While procfs has been implemented in
various different Unix-like operating systems it is slowly being depreciated in
FreeBSD and has been fully removed from OpenBSD. On Linux procfs has been
extended to also include non-process related data and certain hypervisors will
create entries under /proc which allows can be used to check if the host is
being virtualized. This method is generally not as reliable since it may
require special drivers to be installed on the guest host. We can also use this
method to deteremine if the host is acting as the initial privileged domain
(dom0) started by the Xen Hypervisor on boot.

```
bundle agent detect_guest_files
{
    vars:

      "openvz_files" slist => { "/proc/vz/vzaquota", "/proc/user_beancounters" };
      "xen_dom0_files" slist => { "/proc/xen/capabilities" };

    classes:

      linux::

        "openvz" expression => fileexists("@(openvz_files)"),
          scope => "namespace";

        "xen_dom0" expression => fileexists("@(xen_dom0_files)"),
          scope => "namespace";
}
```

### Detecting a FreeBSD virtualized host

When FreeBSD detects that it is being virtualized it will expose the hypervisor
as a sysctl under kern.

    $ sysctl -n kern.vm_guest
    xen

The possible values in FreeBSD include "none", "generic", "xen", "hv",
"vmware", and "bhyve" which makes it trivial to detect the most popular
hypervisors. Using a method similiar to detection via the OUI, a data container
called `$(fbsd_known_guest_vms)` is created that contains key value pairs of
possible values that can be returned and the class we want to define if a
`strcmp()` of that of key is true.

The value of the variable `$(fbsd_vm_guest)` is set via the `execresult()`
function which will run the command `sysctl -n kern.vm_guest`. The value of
`$(fbsd_vm_guest)` is compared to every value in `$(fbsd_known_guest)` which
are the keys of the data container `$(fbsd_known_guest_vms)`

```
bundle agent detect_guest_freebsd
{
    vars:

      freebsd::

        "fbsd_vm_guest" string => execresult("$(paths.path[sysctl]) -n kern.vm_guest", "noshell");

        "fbsd_known_guest_vms" data => '{
          "xen":     "xen_domu",
          "hv":      "hyper_v",
          "vmware":  "vmware",
          "generic": "generic_hypervisor"
        }';

        "fbsd_known_guest" slist => getindices("known_guest_vms");

    classes:

      freebsd::

        "found_vm_guest" expression => strcmp("$(fbsd_known_guest)", "$(fbsd_vm_guest)");

          "$(known_guest_vms[$(vm_guest)])" expression => "found_vm_guest",
            scope => "namespace";

    reports:

      found_vm_guest::

        "Host is Virtualized";

      xen_domu::

        "Running on Xen";
}
```

Doing a test of this agent on a DomU FreeBSD host will set a 'xen_domu' class
and report that the host is virtualized.

    $ uname -sr
    FreeBSD 12.0-CURRENT

    $ cf-agent -f ./vm.cf -b detect_guest_freebsd
    R: Host is Virtualized
    R: Running on Xen

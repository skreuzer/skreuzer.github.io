---
layout: post
title: Use realpath to work around the CFEngine 3.5.3 symlink change
tags: ["cfengine"]
---
In CFEngine 3.5.3, a symlink created by a user pointing to resources owned by a
different user is no longer be followed.

There is currently no way to override this behavior and it is causing several
to fail because on each machine I expect /opt to be a symlink with the
permissions root:wheel that is pointing to a 'current' symlink on an NFS volume
that is owned by the build user which is pointing to our most recent release.

When a new release is ready, the build user has the ability to flip the
'current' symlink to point at the new release automatically. In the event that
we need to roll back to a known good state, it is as simple as switching the
'current' symlink to the last good build.

This issue can be worked around using `realpath` which will return the resolved
physical path of the path passed to it. I opened a [pull request](https://github.com/cfengine/masterfiles/pull/100)
which has been merged into back into [cfengine/masterfiles](https://github.com/cfengine/masterfiles)

It sets the path to the `realpath` binary on several different flavors of Unix
and Linux and to allow you to write an agent like this

```
bundle agent copy_release
{
    vars:

        "file"     slist  => { "foo.py", "bar.py", "baz.py" };
        "src_path" string => execresult("$(paths.path[realpath]) /opt", "noshell")

    files:

        "/usr/local/bin/$(file)"
            copy_from => local_dcp("$(src_path)/$(file)");
}
```

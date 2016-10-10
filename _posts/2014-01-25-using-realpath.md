---
title: Use realpath to work around the CFEngine 3.5.3 symlink change
---
In CFEngine 3.5.3, a symlink created by a user pointing to resources owned by a
different user is no longer followed.

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

<script src="https://gist.github.com/skreuzer/379299c2cc37567b5321.js"></script>

---
title: Use realpath to work around the CFEngine 3.5.3 symlink change
---
In CFEngine 3.5.3, a symlink created by a user pointing to resources owned by a
different user is no longer followed. This is an intential change and while it makes
sense from a security standpoint, people who came to depend on the old behavior and
built tooling and infrastructre around it found themselves in a bit of a bind.

As best as I can tell there is no way to override this behavior and the end result
is that it is causing several promises which are part of my release process to fail.
On each machine I expect /opt to be a symlink with the permissions `root:wheel` that
is pointing to a 'current' symlink on an NFS volume that is owned by the build user
which is pointing to our most recent release.

When a new release is ready, the build user has the ability to flip the
'current' symlink to point at the new release automatically. In the event that
we need to roll back to a known good state, it is as simple as switching the
'current' symlink to point at previous build.

This issue can be worked around using `realpath` which will return the resolved
physical path of the path passed to it. I opened a [pull request](https://github.com/cfengine/masterfiles/pull/100)
which adds the path to the realpath binary into the stdlib so you can simply reference
`$(paths.path[realpath])` in your agent.

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

The pull request has been accepted and it has been merged into back into [cfengine/masterfiles](https://github.com/cfengine/masterfiles).
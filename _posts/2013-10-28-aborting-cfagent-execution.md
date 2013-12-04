---
layout: post
title: Aborting execution of cf-agent
tags: ["cfengine"]
---
From time to time it might be necessary to temporary halt the
execution of cf-agent for any number of reasons. If during a run
the class halt_cfagent gets defined, cf-agent will exit its current
run,

{% highlight sh %}
body common control
{
    bundlesequence => { "blackout" };
}

body agent control
{
    abortclasses => { "abort_cfagent" };
}

bundle agent blackout
{
    vars:

        any::

            "blackout_file" string => "/tmp/cf_blackout";

    classes:

        "abort_cfagent" expression => fileexists("$(blackout_file)"),
            comment => "Abort execution of cf-agent";
}
{% endhighlight %}

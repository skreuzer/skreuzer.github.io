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
bundle agent blackout
{
    vars:

        any::

            "blackout_file" string => "/tmp/cf_blackout";

    classes:

        "halt_cfagent" expression => fileexists("$(blackout_file)"),
            handle => "define_classes_halt_cfagent",
            comment => "Abort execution of cf-agent";

    reports:

        halt_cfagent::

            "Cfengine is in blackout mode and will not execute. Remove $(blackout_file) to continue.";
}
{% endhighlight %}

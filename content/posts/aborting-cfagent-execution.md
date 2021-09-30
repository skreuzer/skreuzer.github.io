---
title: Aborting execution of cf-agent
date: 2013-10-28
tags: [cfengine]
---
From time to time it might be necessary to temporary halt the
execution of cf-agent for any number of reasons. If during a run
the class halt_cfagent gets defined, cf-agent will exit its current
run

{{< gist skreuzer 00629d18c444db52df46 >}}

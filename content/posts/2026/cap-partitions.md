+++ 
draft = false
date = 2026-01-19T14:05:56-07:00
title = "CAP Theorem - Partion is a verb, not a noun"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

People who’ve spent some time around databases usually have an intuitive sense of ACID, even if the acronym slips their mind. And if you work with distributed systems, you usually have a rough idea of the CAP Theorem, even if the details are fuzzy. The first thing you remember about CAP is probably, "Pick two out of three." And maybe you can even remember the letters without a search engine ...

* Consistency

* Availability

* Perf ... oops! It's not Performance. Better check that search engine.
{{< center >}}
![Venn diagram - wrong CAP](/images/CAP_wrong_venn.png)
{{< /center >}}

Since distributed systems require network hops, it feels right for P to mean "Performance" - but that's not what it means. The P in CAP stands for **Partition Tolerance**.

For the longest time, "Partition Tolerance" just wouldn’t stick in my brain. Then something finally clicked. In the CAP theorem, partition isn’t a noun - it's a verb.

I had thought of the word partition in "Partition Tolerance" as referencing a subset of data. After all, when you partition data stores, you shard them by some attribute - an ID or a geographic location, for example. If you think of "partition" as a noun, you might imagine a system that keeps working for everyone except the people in Maryland, whose data is now offline. Of course that’s hard to remember - that doesn't sound like a good thing. Certainly not for the people in Maryland.

Turns out, "Partition Tolerance" isn't about a data partition at all; it's about the behavior of the network. In this context, the word partition is a verb. It means that when one of the nodes goes AWOL, the system continues operating — though what "operating" looks like depends on the system’s design.

This is a crucial distinction because it shifts the focus from a static state ("partition" as a noun) to a dynamic event ("partition" as a verb). It means the system is designed to handle communication failures gracefully, continuing to operate despite network splits or node isolation.

But what about those poor folks in Maryland? Could they still lose access? Yes, but not because their data is sharded. It's because when the network broke, the system had to pick a side. Depending on the design, Maryland might even be the only side that stays up. Take that, Virginia!

Partition tolerance isn’t about where your data lives. It’s about what your system does.
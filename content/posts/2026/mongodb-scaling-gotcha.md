+++ 
draft = true
date = 2026-05-20T14:01:04-06:00
title = "Do You Know Where Your Mongo Requests Are Going?"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = ["MongoDB Config"]
+++

The promise of MongoDB: by using MongoDB, you can put control of the database into the developers' hands, accelerating product development. And it's a distributed database, so when your CTO asks you if you can support horizontal scaling, you can answer in the affirmative. So far, so good.

But databases are complicated. And distributed databases are even more complicated. While you're out there trying to write stateless services, distributed database systems don't have that luxury - they must maintain state while scaling. It's a big ask. Especially when you're also trying to enable the average developer, who doesn't have deep knowledge of database internals or distributed data stores. So MongoDB gives you advanced configuration options, and it also sets defaults.

If you only ever plan to need one node, you can skip the rest of this post. But if you want to use a distributed database, you need someone who understands databases; someone who can set the right configurations for your particular situation. 

If you're not careful, you could end up in a scenario I've seen, where the team thought that their multi-node cluster* was providing horizontal scaling. In actuality, every single read and write went to the same node. That's fine if your goal is to have a warm backup for offline indexing and failover. Not so great if you think you're ready for a Black Friday event.

Put your hands up - how many of you have worked with MongoDB? Okay, how many have heard of "read preferences", "read concerns", and "write concerns"? Okay, now, how many of you can define them, or know how to use them, without checking your phones?

MongoDB's [Read Preference](https://www.mongodb.com/docs/manual/core/read-preference/) specifies which node to query. The default value is "Primary". This means that by default, no matter how many nodes you have in your cluster, every query hits the same node.

MongoDB's [Read Concern](https://www.mongodb.com/docs/manual/reference/read-concern/) "controls the consistency and isolation of data read from replica sets and sharded clusters." If that sentence makes you nervous, you should probably consult a DBA.

MongoDB's [Write Concern](https://www.mongodb.com/docs/manual/reference/write-concern/) "describes the level of acknowledgment requested from MongoDB for write operations to a standalone mongod, replica sets, or sharded clusters." See what I mean?

Here's what happens if you use the defaults in a multi-node cluster:

| Setting (where default lives)| What it does |Implications|
|--------------------------------------|----------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| **Read Preference** (`primary`, driver) | “All reads go to the primary node.”                | You're paying for nodes you don't use. When traffic spikes, you discover you can't actually scale horizontally. No master switch; you must update applications individually. |
| **Read Concern** (`local`, server/cluster) | “Different nodes may return different values (briefly).” | This is normal for a distributed database, but it still surprises people.                               |
| **Write Concern** (`w: majority`, server/cluster) | “Writes are durable once majority‑replicated and journaled.” |                                       |


So, what happens when you put the defaults together?

Let's look at a single node system first. In a single node system, Read Preference and Read Concern are irrelevant. 


If your developers don't understand these details and don't have a DBA to help them, then at best, you're wasting your company's money on hosting fees. At worst, you have misplaced confidence in your ability to scale. You won't find out until you need it - and when you do, you're going to have a real bad day.


\* I'm using the term "cluster," but the correct term in Mongo-land is "Replica Set."
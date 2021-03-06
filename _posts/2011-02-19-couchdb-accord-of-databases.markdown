---
layout: post
title: "CouchDb: The Honda Accord of databases"
published: true
author: Sreekanth
categories:
- couchdb
- databases
- mapreduce
---

We have been using with CouchDB at Activesphere, for some of our customer projects.

We looked at a few options before we eventually plunged into Couchdb 'Relax'. I'm trying to list out in hindsight why CouchDb has worked out a good choice for us.


**Powerful Replication**

CouchDb has amazing replication support. Period. It supports all kinds of scenarios, you can think of and best of all it is trivial to set up continuos replication between two couchdb's.


On my current project we needed elastic scaling. We let Amazon ELB
distribute the load for us and depending on the load, add or remove
machines from our "cluster". So all nodes having exactly the same
software stack and data is extremely important. We run couchdb on
every machine of the cluster, and each machine replicates with the
master, so all the data spreads over the cluster in short time. This
means no big beefy dedicated central database.

An accidental (and big) advantage of this architecure is that the
database and the application server are always on the same machine. It
takes away the network latency accessing the central database


**Fast View Queries**


We have a large amount of data that is mostly readonly, and that is
primarily used to search information. The couch views provide us with
an incredibly simple and performant solution. Even with a few million
documents in couchDb, we get search results in a few hundredths of a
second.

Among other things that is fundamentally different in couchdb and
source of perpetual confusion is the lack of SQL finders, So there is
no ability to run dynamic queries on your database that we are so used
to.

What we get instead is this concept called Map Reduce. CouchDb map
reduce is only conceptually similar to the Google/Hadoop Map reduce,
but it is an easy switch for people coming from the big data world,
where Map reduce is de rigueur.

By giving up the ability to query arbitrary SQL, CouchDb can always
run queries that always uses the indices, which is the reason for it's
query performance. Views are stored on the disk in a B-Tree based disk
index. A View is built once as it is created, and after that it is
always updated incrementally. Which explains the Consistent Query
times even with few million documents, and slightly slower write
times. In our system read-write ratios are about 90% - 10%. This works
brilliantly for us.

**Http Access to the views**

CouchDb providers a simple http api to access the views. We use this
extensively for our internal applications. We just document the ways
of using these views and let users hit the database from ruby console
or via their Apps. For internal Apps, this has become the standard,
and avoids the layers of code that build up to support exposing the
SQL reporting structure. I still dread at the thought of writing the
multiple pages of SQL for reporting and turning them into objects that
API could use. All our Billing information and metrics are exposed via
Views. We use the myriad different ways of using the startkey, endkey,
grouplevel to allow the different usages of the Information. That is a
topic i plan to address in a later blog post.

And no it does not increase the load our database server, because we
run analytics off a replicated analytics database.

There are of course some cons of using Couchdb some of it are easily
addressed. I plan to write more in the next few posts.

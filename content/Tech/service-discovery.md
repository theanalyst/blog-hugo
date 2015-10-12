+++
title = "The very basics of service discovery"
date = "2015-06-11"
tags = ["consul", "distributed-systems","service-discovery","chubby"]
image = "images/signs.jpg"
+++

This is meant to be an introductory post to a series of posts about
Service Discovery and Distributed Consensus. Hopefully this will be
series of posts on more aspects of service discovery & [Consul][1] etc.

### Service Discovery

We begin with the simple problem of identifying which host/port
services are running. For eg. in a small webapp, a simple static conf
file could point to the DB node. As functionality grows you would have
various (preferably stateless) services talking on different ports
(and even different servers). Continuing with a static configuration
will mean that every time a new service is introduced, a lot of
configuration change is expected. Also typically nodes go down, will
reappear with a new address etc. new services will appear
etc. Basically it is a problem of every service being in agreement on
the environment which it is in.

One way of solving this problem includes having a sort of service
registry, which could be something like a simple key value store where
all services can query & use for coordination. Since this store will
be the basis of other distributed systems, it will need to be
consistent in face of network partitions etc., thus typically
requiring a quorum of writes for commiting a value in the registry,
typically using something like Paxos at its core. If that sounded
greek to you, let us assume that you host something that is just a
simple key/value store service. Since you can't trust hosting this
service on a single node, as it may go down, become unreachable
etc. you need to host it in multiple nodes, say 3. Now writing a key
in this store has to be consistent across all the 3 nodes, or else bad
things may happen as the client will try to read value and start
making decisions based on that. So every write to the store sort of
goes to one of the nodes, which will be designated as the leader
[^paxos], and the leader ensures that every entry is passed on to its
follower nodes. Any query reaching any of the other two nodes will be
forwarded to the leader. The leader has the responsibility of ensuring
that all entries are atleast written by a majority of nodes so that if
something bad happens, a network partition for example, writes cannot
be made until there is a majority & once the partition heals there can
only be one view of the world. (Also the emphasis on odd number of
nodes for quorum as the system can take the loss of n-1/2 nodes)

Currently the tools for doing this kind of service discovery &
coordination include Apache Zookeeper, CoreOS' etcd, and now
Consul. Here I'll try to explore a little bit into the paper that
started it all, Google's Chubby paper.

### Chubby

Google's [Chubby][3], is described as a coarse grained *Lock Service*
& a low volume datastore for aiding loosely coupled distributed
systems. It was sort of like a Paxos as a Service for other systems to
coordinate and reach a consenus about its environment & in electing
leaders among a set of similar nodes etc.  Chubby also provided a
filesystem like interface, which applications could use to share
details about its configuration etc. Chubby was deployed in sets of
Chubby Cells, which contained a set of 5 nodes with a master and 4
replica nodes.

#### Locks

Chubby provided advisory locks, ie. locks only conflict with others
trying to acquire the same lock. The locks could be used as a leader
election primitive, for eg. by giving leadership to the lock
holder.(Consul's [session][4] & leader election primitives are
heavily based on this)

#### Sessions & Keepalives

In order to check for membership of clients, (so as to know what
services are up, nodes are up etc.) each client maintained a sesion
with a Chubby Cell, with periodic handshakes called KeepAlives. As a
sessions lease expires the client is expected to respond, lest its
locks, cached data etc. could be invalidated. 

### Uses
- Allowed services to use distributed consensus primitives (like
Paxos) without redesigning the application for it
- FileSystem interface was used for managing configuration files,
metadata etc. by services
- Used as a nameserver to discover other services etc. 

In further posts I'll try to cover how tools like consul implement
many of these features and how they can aid service discovery &
coordination.

### Other links to read
- [Open Source Service Discovery][2]
- [Consul Service Discovery with Docker][3]
- [Camille Fournier's Chubby Presentation @ Papers we Love][5]

[^paxos]: Paxos doesn't technically require a leader for commits, but explaining things is kind of more easier with a leader. 

[1]: https://consul.io
[2]: http://jasonwilder.com/blog/2014/02/04/service-discovery-in-the-cloud/
[3]: http://research.google.com/archive/chubby.html
[4]: http://www.consul.io/docs/internals/sessions.html
[5]: https://github.com/papers-we-love/papers-we-love/issues/169

+++
date = "2015-05-25T23:45:44+05:30"
draft = true
title = "Running Go ci tests in a docker"
tags = ["docker","Go","ci","ceph"]
image = "https://flic.kr/p/nzE9tm"
+++

I've been toying around with the idea of using containers for running
ci tests, primarily to have a quicker feedback loop; instead of setups
that involve VMs etc. A project which I'm interested in [go-ceph][1]
(Go bindings for ceph/rados) kind of ideally fit the bill for using
this, since testing this project locally, usually needed something
like a VM running a ceph cluster, or a locally running ceph. Though
both of the above aren't that hard, and there are projects around
which kind of ease the process (Vagrant ceph for eg.), it still
requires a somewhat longer setup time if you're looking for a bringup
teardown sort of environment. Writing dockerfiles & scripts for this
helped me appreciate the [best practices for writing dockerfiles][2] a
bit better.

## Workflow
Ideally the workflow expected would be that you would have
a docker container with the necessary setup & dependencies already
installed, and the docker container could ultimately be run with
something as simple as a `go test` to test the latest code. Also since
building a docker container for every run may not be what want; the
idea would be to volume mount the current code tree as a volume, so
that a simple docker run would do the job of a ci tester/builder etc
(something like a local travis)


[1]:https://github.com/noahdesu/go-ceph
[2]:https://docs.docker.com/articles/dockerfile_best-practices/

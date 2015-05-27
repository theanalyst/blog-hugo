+++
date = "2015-05-25T23:45:44+05:30"
title = "Running Go ci tests in a docker"
tags = ["docker","Go","ci","ceph"]
image = "images/docker.jpg"
+++

I've been toying around with the idea of using containers for running
ci tests, primarily to have a quicker feedback loop; instead of setups
that involve VMs etc. They are ideal for getting an environment up and
running quickly and cheap to throw away too.

A project which I'm spending some time lately [go-ceph][1], which provides
Go bindings for ceph/rados, kind of ideally fit the bill for using
this, since testing this project locally, usually needed something
like a VM running a ceph cluster, or a locally running ceph. Though
both of the above aren't that hard, and there are projects around
which kind of ease the process (Vagrant ceph for eg.), it still
requires a somewhat longer setup time if you're looking for a bringup
teardown sort of environment. Writing dockerfiles & scripts for this
helped me appreciate the [best practices for writing dockerfiles][2] a
bit better.

### Workflow
Ideally the workflow expected would be that you would have
a docker container with the necessary setup & dependencies already
installed, and the docker container could ultimately be run with
something as simple as a `go test` to test the latest code. Also since
building a docker container for every run may not be what want; the
idea would be to volume mount the current code tree as a volume, so
that a simple docker run would do the job of a ci tester/builder etc
(something like a local travis)

### Dockerfiles & ENTRYPOINT hack

Docker containers are well suited for single processes, which can be
set as default by the `CMD` or the `ENTRYPOINT` directives. (There are
differences between the two, which I'll not be getting into for
now). However this presented a problem since even the most basic ceph cluster required atleast 4 daemon processes to be run. Ways of solving this include:

- Creating multiple docker containers for each process and linking them
Cleanest method. This is what [ceph-docker][3] actually uses and the recommended way if you want to run a ceph cluster in containers.
- Manage multiple processes in a docker using a process manager such as [supervisor][4], [official docker docs][5]

A third alternative is to run a shell script as the entrypoint; finishing off the script with an `exec`, note that this method will fail when the entrypoint is overridden. However since all I wanted was a minimally working ceph cluster; this hack was used to run a shell script that basically started all the necessary processes and finished off with an `exec` call running `go test -v` for the package. This is how the dockerfile looked ultimately


{{<highlight docker>}}
FROM golang:1.3-wheezy
MAINTAINER Abhishek Lekshmanan "abhishek.lekshmanan@gmail.com"

ENV CEPH_VERSION giant

RUN echo deb http://ceph.com/debian-$CEPH_VERSION/ wheezy main | tee /etc/apt/sources.list.d/ceph-$CEPH_VERSION.list

# Running wget with no certificate checks, alternatively ssl-cert package should be installed
RUN wget --no-check-certificate -q -O- 'https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc' | apt-key add - &&\
    apt-get update && \
    apt-get install -y ceph \
    librados-dev librbd-dev libcephfs-dev

VOLUME /go/src/github.com/noahdesu/go-ceph

COPY ./ci/entrypoint.sh /tmp/entrypoint.sh

ENTRYPOINT ["/tmp/entrypoint.sh", "/tmp/micro-ceph"]
{{</highlight>}}

For those interested in ceph, the entrypoint script was a modification
of [Loic's micro-osd script][6], with the only addition being getting the go dependencies and finishing of with an `exec` call of `go test`. For the gory details refer to the actuall [pull request][7] submitted to the upstream project.

Though this particular case needed a little bit of a tweak to run
tests in containers, in a general case it is far easier to run local
ci like tests even covering multiple Go versions with other
dependencies etc easily in a docker.

[1]:https://github.com/noahdesu/go-ceph
[2]:https://docs.docker.com/articles/dockerfile_best-practices/
[3]:https://github.com/ceph/ceph-docker
[4]:http://supervisord.org/
[5]:https://docs.docker.com/articles/using_supervisord/
[6]:http://dachary.org/?p=2374
[7]:https://github.com/noahdesu/go-ceph/pull/21

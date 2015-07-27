+++
comments = true
date = "2015-07-28T00:03:28+05:30"
draft = false
image = ""
slug = "post-title"
tags = ["python", "docker", "pip", "devpi"]
title = "Caching pip packages using devpi & docker"
+++

If you're using python packages & virtualenvs a lot, you can
drastically speed up the time to setup your environment, (which mostly
involves download of pip packages, which tend to be network intensive
& time consuming) by caching pip packages. Similar to other caching
proxies like apt, python has its own caching proxy in the form of
[devpi](http://doc.devpi.net/latest/)[^1], which allows you to run a
pypi mirror in your laptop. (devpi is much more than just a pip
mirror, for more on its capabilities read the link)

Though running devpi by downloading the pip package is
[easy enough](http://doc.devpi.net/latest/quickstart-pypimirror.html),
running it permanently requires steps like configuring nginx etc. If
you're lazy, it is simple enough, to run it as a docker container, and
configure your init system to start the container on system startup.
Scrapinghub's
[docker-devpi](https://github.com/scrapinghub/docker-devpi) image
makes it easy enough to get started. Running a devpi server is as
simple as:

{{<highlight sh>}}
 $ docker pull scrapinghub/devpi
 $ docker run -d --name devpi -p 3141:3141 scrapinghub/devpi
{{</highlight>}}
	
Next configure your pip to pull from here. This is as simple as
sticking the following line into your pip.conf (which should reside in
~/.pip/pip.conf, if there is no file, create it)


{{<highlight cfg>}}
[global]
index-url = http://localhost:3141/root/pypi/+simple/
{{</highlight>}}

Next downloading a pip package will be mirrored, trying to install it
again (even in other virtualenvs) should be almost instantaneous.

Since the docker container was already started with a name parameter,
this container can be restarted next time simply by doing a `docker
start devpi`. Of course this can be easily handed off to your init
system. If you're using Ubuntu <= 14.04, the relevant upstart script
would be something like `/etc/init/devpi-docker.conf` with contents
similiar to below. After adding the script, you would have to do an
`initctl reload-configuration` for upstart to see the script.


{{<highlight sh>}}
description "Devpi Docker"
author "You"
start on filesystem and started docker
stop on runlevel [!2345]
respawn
script
  /usr/bin/docker start devpi
end script
{{< /highlight >}}

After this doing a `sudo start devpi-docker` would start the devpi
docker container. Also this should be picked up by default when your
system starts.

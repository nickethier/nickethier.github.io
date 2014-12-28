---
layout: post
title:  "Dockerized Rocket"
date:   2014-12-28 14:29:00
categories: docker rocket containers
---

[Rocket][rocket-gh] is an implementation of the [Application Container Spec][appc-gh], built by CoreOS.
Naturally I built a docker image to more easily kick the tires on rocket.

## Rocket on Docker

The image `nickethier/rkt` can be pulled from the docker hub.
It has to be ran with the `--privileged` flag in order to launch containers, and there must be a volume mounted to persist storage.
The image is built with the `rkt` binary as the entrypoint.
Following the rocket README the etcd Application Container Image or aci can be fetched and ran by doing the following.

{% highlight bash %}
>$ docker pull nickethier/rkt
>$ docker run --privileged --rm -v /mnt:/var/lib/rkt nickethier/rkt fetch https://github.com/coreos/etcd/releases/download/v0.5.0-alpha.4/etcd-v0.5.0-alpha.4-linux-amd64.aci
sha256-6635e9cbe18c6f51e8c70c143948df111b5626db39198182fbeb9277beb606db
>$ docker run --privileged -d -v /mnt:/var/lib/rkt nickethier/rkt run sha256-6635e9cbe18c6f51e8c70c143948df111b5626db39198182fbeb9277beb606db
{% endhighlight %}

You can also override the entrypoint to a shell to inspect the rocket datadir.

{% highlight bash %}
>$ docker run -it --privileged --rm -v /mnt:/var/lib/rkt --entrypoint="sh" nickethier/rkt
># find /var/lib/rkt/cas/blob/
/var/lib/rkt/cas/blob/
/var/lib/rkt/cas/blob/sha256
/var/lib/rkt/cas/blob/sha256/66
/var/lib/rkt/cas/blob/sha256/66/sha256-6635e9cbe18c6f51e8c70c143948df111b5626db39198182fbeb9277beb606db
{% endhighlight %}

The Dockerfile for this image is available [on github][docker-rkt], feel free to fork, open an issue, ignore etc.
The Application Container Spec seems like the natrual evolution in the container movement.
It seems to borrow ideas from the kubernetes project.
Rocket is still a very young project and only currently implements a subset of the Application Container Spec.
If you get anything out of rocket, I hope its that you read through the Application Container Spec pretty thoroughly.
If you're on the Rocket vs Docker bandwagon, I invite you to read [this awesome post][docker-v-rocket] over at the containerops blog.


[rocket-gh]:       https://github.com/coreos/rocket
[appc-gh]:         https://github.com/appc/spec
[docker-v-rocket]: http://containerops.org/2014/12/19/docker-vs-rocket-gimme-a-break/
[docker-rkt]:      https://github.com/nickethier/docker-rkt

---
title: "How to stop Docker exposing your containers to the world"
date: 2020-07-12T17:51:30-03:00
categories:
  - Blog
tags:
  - Docker
---

When using Docker, a common thing to do is map a port from the container to the host machine using the `docker run -p` in the CLI or the _ports_ argument in a docker-compose file. However, if you are not careful you can end up exposing services to the world when they should only be accessible inside the host. Let's see how this can happen and how to avoid it.

## Map a port to the host

A port can be mapped from the container to the host using the `-p host:container` argument in `docker run`. An example would be to create an Nginx container and make it accessible from the host on port 1234:

```console
$ docker run -p 1234:80 nginx
```

Accessing `localhost:1234` from a browser gives us the Nginx welcome page:

![Nginx running in a container accessible from host](/assets/images/blog/docker-ports-nginx-browser.png)

That is great! We now have a working Nginx container accessible on port 1234 in the host machine.

## The problem

However, let's take a look at the open ports in our host:

```console
$ netstat -tuln | grep LISTEN
...   
tcp6       0      0 :::1234                   :::*                    LISTEN     
...
```

Notice that it is listening for connections from any address on port 1234 (this is what `:::1234` means). Now anyone in the same network as the host can connect to the container and see the Nginx welcome page. 

For example, if you have a MongoDB container running on your laptop while using the Wi-Fi of the hotel you are staying at, a hacker in the same network can connect to your container and steal all the information inside it. This is especially worrisome if the host is a production server with a public IP. It could end up exposing functionalities that should only be accessible inside the host, like internal APIs with sensitive information or destructive actions.

## Why is Docker doing this?

By default, Docker edits the `iptables` entries in the host machine, bypassing firewall configurations. It means that even if the sysadmin correctly configured the machine to deny all incoming traffic, Docker will ignore it and create a hole in the firewall. This behavior sure provides some ease o use, but create a lot of vulnerabilities, especially with a user that just started using Docker. Check [this thread](https://github.com/moby/moby/issues/22054) for more insight on the issue.

## How to avoid this situation

The first thing to notice when trying to fix this is that if all applications that communicate with the container are also containerized, it's not even necessary to expose a port on the host. Just [let Docker manage the connections](https://docs.docker.com/compose/compose-file/) and reference the container inside the source code using service defined in `docker-compose.yml`. While looking for Docker examples on Google I noticed a lot of people still don't use this feature even when both applications are containerized.

However, if you need an application in the host to connect to the container, the way to not expose your container to the world is to explicitly declare that the port should only be accessible from the localhost. This can be achieved prepending `127.0.0.1` to the host port number in `docker run`:

```console
$ docker run -p "127.0.0.1:27017:27017" mongo
```

Or in `docker-compose.yml`:

```yaml
...
    mongodb:
        image: mongo
        ports:
        - "127.0.0.1:27017:27017"
...
```

Now our application is only accessible inside the host machine:

```console
$ netstat -tuln | grep LISTEN
...   
tcp        0      0 127.0.0.1:1234          0.0.0.0:*               LISTEN 
...
```

However, [some people reported](https://github.com/moby/moby/issues/22054#issuecomment-214496744) that even explicitly referencing `127.0.0.1`, the container could still be accessed from outside. For an extra layer of security, you could also modify the default Docker behavior and stop it from tampering with the `iptables` altogether. For more info on how to do this, take a look at the links I provided at the end of the post.

## Conclusion

We saw how Docker can expose your containers to the world if not used correctly. You should always check if you are exposing containers that should not be exposed to the world, specifying `127.0.0.1` as the address to be listening for connections. Also, always follow the best practices to secure your containers on production, setting appropriate authentication for them.

Some interesting reading to follow on if you are interested:

https://github.com/moby/moby/issues/22054

https://www.jeffgeerling.com/blog/2020/be-careful-docker-might-be-exposing-ports-world

https://fralef.me/docker-and-iptables.html

https://blog.viktorpetersson.com/2014/11/03/the-dangers-of-ufw-docker.html
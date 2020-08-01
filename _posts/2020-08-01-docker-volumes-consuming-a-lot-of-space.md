---
title: "Docker volumes consuming a lot of space"
excerpt: "Use the --rm flag to stop Docker from keeping useless volumes from your containers."
date: 2020-08-01T13:51:30-03:00
categories:
  - Blog
tags:
  - Docker
---

This week I was cleaning my home directory from all the things that were not useful anymore. When I went to see the disk usage, I had a surprise: There was only 20% of free space in my SSD. I don't have a lot of programs installed, neither did I remember downloading any huge files. So something was taking a lot of space without my knowledge.

The search for the root of the problem quickly led me to the Docker folder. The _volumes_ folder was taking more than 30GB of disk space, 50% of the total partition size. I wasn't making any huge project that would have big volumes. So what was the problem?

### Docker doesn't remove volumes by default

When you create a container that requires a volume, like MongoDB, the volume is not deleted after the container is removed. Even though when you run `docker run` again a new volume is created, the previous one is not deleted from disk.

So what was happening is that I was running `docker run mongo` every time I needed to create a fresh database while developing an application. When I needed to wipe out everything, I would just remove the container and create a new one. 

What I was unaware of is that deleting the container does not delete the created volume. So every time a MongoDB container was created, a 300MB volume was created alongside it, and never removed. To fix it, the first thing I did was to prune the unused volumes:

```terminal
$ docker volume prune
```

Which cleaned more than 30GB of unused volumes. 

To stop `docker run` to keep the volumes while I am developing something, I now run it with the `--rm` flag, which deletes the volume when the container is removed:

```terminal
$ docker run --rm mongo
```

Now I don't have to be worried about my SSD being consumed by unused volumes.

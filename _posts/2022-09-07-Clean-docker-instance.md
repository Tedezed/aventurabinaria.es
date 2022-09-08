---
layout: post
title: "Clean Docker Daemon"
date: 2022-09-06 19:17:06 -0700
comments: true
---

**Clean Logs**
```
for l in $(docker inspect --format='{{.LogPath}}' $(docker ps -q)); do
  echo "" > $l
done
```

**Clean images**
```
docker rmi $(docker images -a --filter=dangling=true -q)
```

**Clean containers with filtered by status**
```
docker rm $(docker ps --filter=status=exited --filter=status=created -q)
```

**Rotate Docker daemon logs**

Edit: `/etc/docker/daemon.json`
```
{
  "log-driver": "json-file",
  "log-opts": {"max-size": "10m", "max-file": "3"}
}
```

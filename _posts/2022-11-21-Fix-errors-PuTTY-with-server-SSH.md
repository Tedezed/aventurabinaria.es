---
layout: post
title: "Fix errors PuTTY with server SSH"
date: 2022-11-21 16:30:06 -0700
comments: true
---

### Error: No supported authentication methods available

- Error in client: `no supported authentication methods available (server sent publickey)`
- Error in server: `key type ssh-rsa not in PubkeyAcceptedAlgorithms`

##### Solution

Add ssh-rsa to PubkeyAcceptedAlgorithms
```
# nano /etc/ssh/sshd_config

HostKeyAlgorithms +ssh-rsa
PubkeyAcceptedAlgorithms +ssh-rsa
PubkeyAuthentication yes
```

Verify your configuration
```
# sshd -t
```

Restart sshd
```
# systemctl restart sshd
```


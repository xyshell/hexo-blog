---
title: Linux Note
date: 2020-06-08 01:26:27
tags:
---

# SSH
``` bash
ssh -i <path-to-pem> admin@<ip/dns>
``` 

Local Port Forwarding:
``` bash
ssh -i <path-to-pem> -l admin <ip/dns> -L 9999:localhost:9999
```

# SCP
``` bash
scp -i <path-to-pem> <path-to-source-file> admin@<ip/dns>:/home/admin/<path-to-target-file>
``` 

# Auto-start after reboot
```bash
$ systemctl enable docker
Synchronizing state of docker.service...
Executing /lib/systemd/systemd-sysv-install enable docker
$ systemctl is-enabled docker
enabled
```
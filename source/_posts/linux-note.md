---
title: Linux Note
date: 2020-06-08 01:26:27
tags:
---

# SSH
``` bash
ssh -i <path-to-pem> admin@<ip/dns>
``` 

# SCP
``` bash
scp -i <path-to-pem> <path-to-source-file> admin@<ip/dns>:/home/admin/<path-to-target-file>
``` 
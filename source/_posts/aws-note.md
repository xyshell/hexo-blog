---
title: AWS Note
date: 2020-06-08 01:30:49
tags: aws
---

# AWS CLI

## Upload folder to S3

``` bash
aws s3 cp <path-to-source-folder> s3://<path-to-target-folder> --recursive --exclude ".DS_Store"
```
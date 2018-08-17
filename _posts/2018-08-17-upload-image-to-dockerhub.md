---
layout: post
title:  "How to push Docker images to DockerHub"
date:   2018-08-17 21:05:00
categories: Container
tags: Docker
---

* content
{:toc}

### Step 1
Create an account @[DockerHub](https://hub.docker.com)
### Step 2
Create a repository. You can name it "test" for example.
### Step 3
Create a Dockerfile. e.g.
```dockerfile
FROM alpine:3.7
ENTRYPOINT ['sh']
```
### Step 4
Build and push with following commands
```bash
docker login docker.io
# login with your user id and password
# docker build -t <dockerhub-user-id>/<repository-name>[:<tag>]
docker build -t precompiler/test:v1.0.0 .
docker push precompiler/test:v1.0.0
```
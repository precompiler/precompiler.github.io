---
layout: post
title:  "How to setup Concourse using Docker Compose on Win 10"
date:   2018-08-11 17:05:00
categories: CI/CD
tags: Pivotal Concourse CI/CD
---

* content
{:toc}

Environment details
  - Windows 10 Home Edition 64 bit
  - Docker version 17.10.0-ce, build f4ffd25 (Docker Toolbox)
### **NOTE**
You'll probably run into issues following [Concourse doc](http://concourse.ci/docker-repository.html). Need to modify the docker-compose.yml. Issues I had
  - By default, docker doesn't have privilege to mount D, E, F... drive. If you put your key files in these drives and mount them, docker won't be able to see the key files. Put key files to c:/Users/<USER_NAME> to solve this issue.
  - RunC container permission issue, if you have following issue, you need to add CONCOURSE_BAGGAGECLAIM_DRIVER: naive environment variable to Concourse worker configuration.
  ```
  runc create: exit status 1: container_linux.go:264: starting container process caused "process_linux.go:339: container init caused \"rootfs_linux.go:56: mounting \\\"/worker-state/3.8.0/assets/bin/init\\\" to rootfs \\\"/worker-state/volumes/live/37f9973b-5a72-4f45-493b-9c6baaa6fb37/volume/rootfs\\\" at \\\"/worker-state/volumes/live/37f9973b-5a72-4f45-493b-9c6baaa6fb37/volume/rootfs/tmp/garden-init\\\" caused \\\"open /worker-state/volumes/live/37f9973b-5a72-4f45-493b-9c6baaa6fb37/volume/rootfs/tmp/garden-init: permission denied\\\"\""
  ```

### docker-compose.yml
  - Use following docker-compose.yml to setup concourse service
  ```yaml
 version: '3'
 services:
      concourse-db:
        image: postgres:9.6
        environment:
          POSTGRES_DB: concourse
          POSTGRES_USER: concourse
          POSTGRES_PASSWORD: changeme
          PGDATA: /database
      concourse-web:
        image: concourse/concourse
        links: [concourse-db]
        command: web
        depends_on: [concourse-db]
        ports: ["8080:8080"]
        volumes: ["/c/Users/<USER_NAME>/keys/web:/concourse-keys"]
        restart: unless-stopped # required so that it retries until concourse-db comes up
        environment:
          CONCOURSE_BASIC_AUTH_USERNAME: concourse
          CONCOURSE_BASIC_AUTH_PASSWORD: changeme
          CONCOURSE_EXTERNAL_URL: "${CONCOURSE_EXTERNAL_URL}"
          CONCOURSE_POSTGRES_HOST: concourse-db
          CONCOURSE_POSTGRES_USER: concourse
          CONCOURSE_POSTGRES_PASSWORD: changeme
          CONCOURSE_POSTGRES_DATABASE: concourse
    
      concourse-worker:
        image: concourse/concourse
        privileged: true
        links: [concourse-web]
        depends_on: [concourse-web]
        command: worker
        volumes: ["/c/Users/<USER_NAME>/keys/worker:/concourse-keys"]
        environment:
          CONCOURSE_TSA_HOST: concourse-web
          CONCOURSE_BAGGAGECLAIM_DRIVER: naive
  ```
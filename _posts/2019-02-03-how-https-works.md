---
layout: post
title:  "How Does HTTPS Work"
date:   2019-02-03 20:00:00
categories: Security
tags: HTTPS SSL TLS
---

* content
{:toc}

> I've been learning security stuff recently, will write several posts to cover HTTPS, Certs etc. This post gives you an overview of how HTTPS works.


![How HTTPS Works](https://github.com/precompiler/precompiler.github.io/raw/master/_posts/HowHttpsWorks.png)

* Client first starts the conversation, telling the server https connection needed
* The server sends the public key and certificates to the client
* The client generates a session key, which will be used to encrypt/decrypt messages. The session key is encrypted using the public key, then sends to the server.
* The server decrypts the session key with the private key
* Both sides get the session key now, symmetric encryption will be used to protect consecutive messages






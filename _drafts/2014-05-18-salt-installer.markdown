---
layout: bootstrap_post
title: Salt Master Creator
date: 2014-05-18 23:10:00
author: Oz Akan
abstract: Creating a salt-master with a single shell command
categories:
    - SaltStack
---

I have found myself in need of creating a salt master several times. Even though it is really easy to install salt-master on a server, I had always found it a bit harder than I would like it to be. My challange hasn't been just installing required packages but also configuring salt-cloud as well as environments.

We have seen the need of an easy way of creating salt-master server which would create a sandbox on a local virtualbox instance or on a cloud instance (Rackspace Cloud is supported currently). It should be as simple as running a single command. This gave birth to salt-installer which is nothing more than a shell script that uses vagrant to create a salt-master server on virtualbox on cloud. If cloud is choosen it will install vagrant on the cloud instance and then create a salt-master using the vagrant instance. So we won't have to install a single package at all.

At the moment, salt-installer creates a vanilla salt-master. Pretty soon, it will be pulling the github repository given so salt-master will be ready to create minions based on the formulas pulled.

## How It Works?

There are two concepts

* Vagrant Server Location
* Provider



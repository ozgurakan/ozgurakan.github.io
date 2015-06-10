---
layout: bootstrap_post
title: OpenStack Cinder Concepts
date: 2015-06-10 18:10:00
author: Oz Akan
abstract: Notes on OpenStack Cinder
categories:
    - Cloud
tags:
    - OpenStack
    - Cinder
    - Concepts
---

## Cinder Terms

- A **Cinder volume** is the fundamental resource unit allocated by the Block Storage service.
    - It is accessed via FC, iSCSI or NFS depending on the storage solution and cinder driver.
    - Cinder volumes are identified by UUID.
    - A Cinder volume (all blocks that makes up the volume) reside on a single Cinder backend.
- A **Cinder Backend** represent a single block storage provider. Provider, provisions the cinder volume and presents to compute.
- **Cinder Snapshot** is a read-only copy of a cinder volume taken at a specific point-in-time. A cinder volume can be created from a cinder snapshot. A snapshot can be created from a cinder volume that is in attached or detached state.
- A **Cinder Driver** implements abstract Cinder APIs for a particular storage solution that provides the Cinder backend.
- **Cinder Volume Type** is used to define different tiers of storage. Different volume types can be provisioned from different backends.
- **QoS Specs** brings QoS support that can be enforced on hypervisor or on storage backend. Not every storage vendor supports QoS on backend.
- **Storage Groups**, allows multiple storage backends to be used for the same OpenStack Compute.

## Cinder Services

All services presented below communicate via AMQP so there is a message broker like RabbitMQ or Q-Pid involved. There is also a database like MySQL to store persistant data.

- **cinder-api** providers a restful API and routes requests for this API to other processes using AMQP.
- **cinder-scheduler** determines where a volume is going to be created based on the specs for these backends
- **cinder-volume** wraps the backend. There is one cinder-volume process per backend. *This is not "Cinder Volume" that is described in Cinder Terms section.*
- **cinder-backup** handles backup requests for volumes.
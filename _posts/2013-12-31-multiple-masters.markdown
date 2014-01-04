---
layout: bootstrap_post
title: Multiple Salt Masters
date: 2013-10-02 22:56:00
author: Oz Akan
abstract: The first time I heard it, I was pretty excited. The first time I used it, I was impressed.
categories:
    - SaltStack
---

The first time I heard it, I was pretty excited. The first time I used it, I was impressed.

"High Availability" is the number one requirement for any service that will go to production. Salt is no different. Thanks to Murphy, we all know that, salt-master server will work without any issues like for ever until the raid controller dies on the server which not surprisingly happens right in the middle of a release. 

But then, we will know that we have one more master, probably in a different data center, likely in a different country or continent. Oh yes, we are on the Cloud. Our service runs on many data centers all around the globe. We need a tool that will be on par with the standards we have. It won't feel like second salt-master is across the ocean. It will just work, thanks to zeromq.

"Like everything else in the world there is cons and pros" would be a wrong statement in this case. So far, I couldn't find one downside of having multiple masters.  It might be early to have a strong statement, I will write an update if my experience changes in time.

Setup is really easy as well. Just put both masters in minion configuration and copy a key from one master to another before doing anything else. docs.saltstack.com has the details.
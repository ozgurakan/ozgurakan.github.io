---
layout: bootstrap_post
title: Execution Order and Overstate
date: 2013-06-05 23:00:00
author: Oz Akan
abstract: Execution order is important while provisioning new servers, while creating environments for the first time or while setting up an database service.
categories:
    - SaltStack
---

A configuration management tool wouldn't care about in which order you need to apply your configuration. It assumes that everything has already been installed and configured so it just has to maintain things as they are or change a little here and there. That is correct. Totally correct. Likely, in an another universe.

Execution order is important while provisioning new servers, while creating environments for the first time or while setting up an database service.

It is not one but two problems.

1. On a server, I have to modify configuration file of an application after application is installed
2. I have to configure a server after another one is configured.

Lets give examples for each:

1. I can edit ```my.cnf``` only after ```mysql``` is installed on a server
2. I can configure web servers only after I have database servers configured as I will have to know IP addresses of database servers to place in my web server configuration.

First problem is solved by ```require``` statment in SLS files. The example below says, I can only install ```mongo-10gen``` after I configure ```10gen.repo``` and I can only install ```mongo-10gen-server``` after I have ```mongo-10gen``` installed.

    /etc/yum.repos.d/10gen.repo:
      file:
        - managed
        - source: salt://mongodb/files/10gen.repo
    
    mongo-10gen:
      pkg:
        - installed
        - require:
          - file: /etc/yum.repos.d/10gen.repo
          
    mongo-10gen-server:
      pkg:
        - installed
        - require:
          - pkg: mongo-10gen

I can create a chain of ```require``` to install / configure everything in a perfect order. Life is beautiful.

Second problem is solved by a file called ```overstate.sls``` which looks similar to ```top.sls``` for state files. Overstate understands ```require``` statement. The example below says, first install all mongodb servers and then configure replica set. Install memcached when ever you like but before web servers. Configure web servers when everything else in the environment is ready.

    mongodb_server:
        match: 'G@roles:mongodb_server'
        sls:
            - mongodb_server
    mongodb_replica:
        match: 'G@roles:mongodb_server and G@mongodb_role:primary'
        sls:
            - mongodb_server.replica
        require:
            - mongodb_server
    memcached_server:
        match: 'G@roles:memcached_server'
        sls:
            - memcached_server
    web_server:
        match: 'G@roles:web_server'
        sls:
            - web_server
        require:
            - memcached_server
            - mongodb_replica

To run this ```overstate.sls``` file, place it under your environment and call it with ```salt-run```.

    salt-run state.over your_environment

Now, watch salt-minons reporting execution results. Watch the flowing text as if it is the potion seeping out of a spring promising the greatness. What a beautiful day it is.

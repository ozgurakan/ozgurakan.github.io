---
layout: bootstrap_post
title: Assigning YAML and JSON output to a bash variable
date: 2014-05-19 11:04:00
author: Oz Akan
abstract: I like how easy it is to use python to parse output of a bash command which produces JSON or YAML output.
categories:
    - Tips
---

I like how easy it is to use python to parse output of a bash command which produces JSON or YAML output. Below is a simple example to read IP address from salt-call output which is a simple YAML file.

By default salt-call, as other command line salt tools, returns a YAML output.

    # salt-call network.ipaddrs eth0
    local:
        - 162.242.227.82

If you want to use this value programatically within a bash script, you can use the sample below;

    # echo $(salt-call network.ipaddrs eth0 | python -c "import sys,yaml;print yaml.load(sys.stdin.readlines()[1])[0]")
    162.242.227.82

Below you can find an example to read JSON output that is returned by Rackspace Cloud Servers API. Pretty much all REST APIs return JSON so it is useful in many cases. The code below checks creation of a server and loops until server state changes to "ACTIVE".

    STATUS="INITIATED"

    while [ "$STATUS" != "ACTIVE" ]
    do
      echo "Waiting for instance to boot up, status : $STATUS"
      sleep 5
      STATUS=`curl -s https://iad.servers.api.rackspacecloud.com/v2/$RACKSPACE_ACCOUNT/servers/$INSTANCE_ID \
           -H "X-Auth-Token: $TOKEN" | python -c "import sys,json;print json.loads(sys.stdin.readlines()[0])['server']['status']"`
    done
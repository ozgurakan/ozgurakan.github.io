---
layout: bootstrap_post
title: Salt Publish vs Mine
date: 2013-12-30 22:56:00
author: Oz Akan
abstract: Salt has two ways to access minion information. One of them is called publish other one is called mine.
categories:
    - SaltStack
---

Salt publish goes a step further and collects the information in real time while mine would collect data every 60 seconds. Mine would keep the traffic at minimum as each minion reports once a minute (which can be changed) and query hits only the master. Publish happens in real time so if 10 web servers want to get information about all database servers in the environment they will create 10 requests instantly.

Sample for Publish

{% raw %}
```
{% for host in salt['publish.publish']('*', 'network.ip_addrs', 'eth0') %}
[{{ host.fqdn }}]
    address {{ host.ip }}
    use_node_name yes
{% endfor %}
```
{% endraw %}

Sample for Mine

{% raw %}
```
{% for host, hostinfo in salt['mine.get']('*', 'network.interfaces').items() %}
[{{ host }}]
    address {{ hostinfo['eth0']['inet'][0]['address'] if hostinfo['eth0'].has_key('inet') else hostinfo['br0']['inet'][0]['address'] }}
    use_node_name yes
{% endfor %}
```
{% endraw %}

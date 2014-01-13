---
layout: bootstrap_post
title: SaltStack Environment Proposal
date: 2013-12-31 10:58:00
author: Oz Akan
abstract: Managing environments is a challange with any configuration management tool. Let's see how we can solve it with SaltStack.
categories:
    - SaltStack
---
Folder Layout

```
1 /srv/salt/
2 /srv/salt-rocks
3 /srv/salt/project_name
4 /srv/salt/project_name/base
5 /srv/salt/project_name/base/rocks (symbolic link)
6 /srv/salt/project_name/base/web_server
7 /srv/salt/project_name/prod/
8 /srv/salt/project_name/test/
```


* Line 1: default path for salt
* Line 2: salt-rocks library which has shared formulas, pulled from git
* Line 3: project's home, pulled from git
* Line 4: project's base folder for base environment
* Line 5: symbolic link in the base to line 2
* Line 6: example for where web_server formulas would be stored
* Line 7: production environment which would be used in overlay  configuration to hold production specific changes
* Line 8: example for test environment, similar to Line 7

This is how master configuration will look like;

```
file_roots:
  base:
    - /srv/salt
  my-project-prod:
    - /srv/salt/my_project/prod
    - /srv/salt/my_project/base
  my-project-test:
    - /srv/salt/my_project/test
    - /srv/salt/my_project/base
```
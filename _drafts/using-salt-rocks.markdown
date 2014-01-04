---
layout: plain
title: Using Salt Rocks
date: 2014-01-03 10:20:00
author: Oz Akan
abstract: How to use a rocks in your environment
categories:
    - SaltStack
project_name: rcbu
---

## What are we doing?

We will import salt-racks into our salt environment and create a project in which we will hold all environments specific to that project. Eventually we will be able to use salt-rocks formulas like ntp, emcas in our formulas without being have to re-invent the wheel. In our project folder we will have environments like production, test, development and also location based ones.

## Let's start with Basics

{% include saltdoc/top-file.markdown %}

{% include saltdoc/fileserver-path-inheritance.markdown %}

## Create Folders

Our project name is `{{ page.project_name }}`, yours can be anything. We will create these folders:

```
/srv/salt
/srv/salt/rocks
/srv/salt/{{ page.project_name }}
```

`/srv/salt/{{ page.project_name }}` will hold the formulas for our project.

First, I am pulling my project under my project folder.

```
# git clone git@github.com:rackerlabs/{{ page.project_name }}.git /srv/salt/{{ page.project_name }}
Cloning into '{{ page.project_name }}'...
Warning: Permanently added the RSA host key for IP address '392.330.352.330' to the list of known hosts.
remote: Reusing existing pack: 4, done.
remote: Total 4 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (4/4), 14.17 KiB, done.
```

If you don't alrady have a repository for your project, just skip the step above.

Project folder, in our case `/srv/salt/{{ page.project_name }}` will have `base` folder and one folder for each environment as seen below:

```
/srv/salt/{{ page.project_name }}/base
/srv/salt/{{ page.project_name }}/prod
/srv/salt/{{ page.project_name }}/test
```

`/srv/salt/{{ page.project_name }}/base` will have formulas to create for example a `web_server` while others will have mostly pillar data and a few formulas definind the servers that have to be different in different environments. Because of the overlay set up  we will have for environments, both `prod` and `test` will be able to use the formulas under `base` environment.

Now pull salt-rocks under `/srv/salt/rocks`

```
# git clone git@github.com:rackerlabs/salt-rocks.git /srv/salt/rocks
Cloning into 'rocks'...
Warning: Permanently added the RSA host key for IP address '392.330.352.330' to the list of known hosts.
remote: Reusing existing pack: 4, done.
remote: Total 4 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (4/4), 4.17 KiB, done.
```

Then edit `.gitignore` file to ignore `rocks` folder

## Configure Salt Master

### Configure Environments

We will set the environments 


```
# /etc/salt/master

file_roots:
  base:
    - /srv/salt/base
  {{ page.project_name }}-prod:
    - /srv/salt/{{ page.project_name }}/prod
    - /srv/salt/{{ page.project_name }}/base
  {{ page.project_name }}-test
    - /srv/salt/{{ page.project_name }}/test
    - /srv/salt/{{ page.project_name }}/base

...

pillar_roots:
  base:
    - /srv/salt/base/pillar
  {{ page.project_name }}-prod:
    - /srv/salt/{{ page.project_name }}/prod/pillar
    - /srv/salt/{{ page.project_name }}/base/pillar
  {{ page.project_name }}-test:
    - /srv/salt/{{ page.project_name }}/test/pillar
    - /srv/salt/{{ page.project_name }}/base/pillar

```

Restart salt-master to make changes effective.

```
# service restart salt-master
```



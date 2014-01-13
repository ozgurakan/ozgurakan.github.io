---
layout: plain
title: Using Salt Rocks
date: 2014-01-03 10:20:00
author: Oz Akan
abstract: How to use a rocks in your environment
categories:
    - SaltStack
project_name: my_project
---

## Rocks, What is it?

Rocks or salt-rocks, is an experimental git repo with group of salt formulas which could be used as a starting point for new formulas. For example, salt-rocks has a formula for mongodb so you can just import salt-rocks and with little effort define your own mongodb servers. Still, purpose of this article is to articulate how an external library (like salt-rocks) can be used in a configuration with several environments.

## What are we doing?

We will import salt-rocks into our salt environment and create a project in which we will hold all environments specific to that project. Eventually we will be able to use salt-rocks formulas like ntp, emacs in our formulas without being have to re-invent the wheel. In our project folder we will have environments like production, test, development and also location based ones.

<!--
## Let's Start With Basics

{% include saltdoc/top-file.markdown %}

{% include saltdoc/fileserver-path-inheritance.markdown %}
-->
## Create Folders

To have proper folders we will have to pull code from two github repositories. First one is our projects repo, second one is salt-rocks repo.

Our project name is `{{ page.project_name }}`, yours can be anything. Our formulas are under a git repo and contains these folders:

```
./base
./{{ page.project_name }}
./{{ page.project_name }}/base
./{{ page.project_name }}/prod
./{{ page.project_name }}/test
```

`./{{ page.project_name }}/prod` and `./{{ page.project_name }}/test` folders have other sub folders with state formulas and pillar data. `./{{ page.project_name }}/base` has the formulas common for prod and test environments. `./base` will hold the base salt environment.

After creating `/srv/salt`, I pull my project under my project (`/srv/salt/{{ page.project_name }}`) folder.

```
# mkdir /srv/salt
# git clone git@github.com:rackerlabs/{{ page.project_name }}.git /srv/salt/{{ page.project_name }}
Cloning into '{{ page.project_name }}'...
Warning: Permanently added the RSA host key for IP address '392.330.352.330' to the list of known hosts.
remote: Reusing existing pack: 4, done.
remote: Total 4 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (4/4), 14.17 KiB, done.
```

New folder structure will look like this:

```
/srv/salt
├── base
└── {{ page.project_name }}
    ├── base
    ├── prod
    └── test
```

`/srv/salt/{{ page.project_name }}` will hold the formulas for our project.


`/srv/salt/{{ page.project_name }}/base` will have formulas to create for example a `web_server` while others will have mostly pillar data and a few formulas defining the servers that have to be different in different environments. Because of the overlay set up  we will have for environments, both `prod` and `test` will be able to use the formulas under `base` environment.

Now pull salt-rocks under `/srv/salt-rocks`

```
# git clone git@github.com:rackerlabs/salt-rocks.git /srv/salt-rocks
Cloning into 'salt-rocks'...
Warning: Permanently added the RSA host key for IP address '392.130.132.130' to the list of known hosts.
remote: Reusing existing pack: 4, done.
remote: Total 4 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (4/4), 4.17 KiB, done.
```

Folder stricture will look like this:

```
/srv
├── salt
│   ├── base
│   └── {{ page.project_name }}
│       ├── base
│       ├── prod
│       └── test
└── salt-rocks

```

Last step is to create a symbolic link in `{{ page.project_name }}/base` folder to `/srv/salt-rocks`

```
# ln -s /srv/salt-rocks /srv/salt/{{ page.project_name }}/rocks
```

Now, unless overwritten by overlay files, all environments will have salt-rocks formulas.

## Configure Salt Master

### Configure Environments

We will set the environments:

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

Restart salt-master to make configuration effective.

```
# service restart salt-master
```

### What About The Secrets?

Very often we had configuration data that we didn't want to put on any kind of shared repository. Directory overlay is again the solution. We basically created a local folder on salt-master and defined this in salt configuration.

```
pillar_roots:
  base:
    - /srv/salt/base/pillar
  {{ page.project_name }}-prod:
    - /srv/salt/{{ page.project_name }}/prod/pillar
    - /srv/salt/{{ page.project_name }}/base/pillar
    - /srv/secret/{{ page.project_name }}/prod/pillar
    - /srv/secret/{{ page.project_name }}/base/pillar
  {{ page.project_name }}-test:
    - /srv/salt/{{ page.project_name }}/test/pillar
    - /srv/salt/{{ page.project_name }}/base/pillar
    - /srv/secret/{{ page.project_name }}/test/pillar
    - /srv/secret/{{ page.project_name }}/base/pillar
```

Now we have a  environment.



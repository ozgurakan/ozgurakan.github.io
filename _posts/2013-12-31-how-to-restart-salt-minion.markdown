---
layout: bootstrap_post
title: How To Restart Salt Minion
date: 2013-12-31 10:58:00
author: Oz Akan
abstract: It is not so easy to restart salt minion but sure there is a way.
categories:
    - SaltStack
---
We found that, we wanted to change minion configuration. So easy it was. Just write a jinja template (```minion.jinja```) for the minon and a formula (```change_minion.sls```) to create the file on the minion.

I won't paste the contents for ```minion.jinja``` but simply we are assigning some grains there and enable salt mine. Below is the code for ```change_minion.sls```.

```
salt-minion:
  pkg:
    - installed
    - name: salt-minion
  file:
    - managed
    - name: /etc/salt/minion
    - template: jinja
    - source: salt://minion/files/minion.jinja
    - require:
      - pkg: salt-minion
  service:
    - running
    - enable: True
    - require:
      - pkg: salt-minion
    - watch:
      - file: salt-minion
```

We would run this as the first formula, so that salt mine would be enabled, roles are assigned and minion would be ready for the formulas that would use mine and grains to make clever decisions.

Problem was with restarting the minion.

```
salt-minion:
  ...
  file:
    ...
    - name: /etc/salt/minion
    ...
  service:
    ...
    - watch:
      - file: salt-minion
```

Watch statement ensures that service will be restarted when there is a change with ```/etc/salt/minion```. The problem is it never works properly which makes sense but also there could be a work around.

Anyway, there is always a few ways to achieve same results with salt. We decided to run a bash script that would restart the minion. But it a regular cmd.run didn't work, for the same reason minion can't restart itself.

A quick solution to this is, placing that bash script in a cronjob that will run next minute. To this day, it still works.

First we changed ```change_minion``` to ```install.sls```.

```
  file:
    - managed
    - name: /etc/salt/minion
    - template: jinja
    - source: salt://minion/files/minion.jinja
    - require:
      - pkg: salt-minion
  service:
    - running
    - enable: True
    - require:
      - pkg: salt-minion

restart-minion:
  file:
    - managed
    - name: /tmp/restart-minion.sh
    - template: jinja
    - source: salt://minion/files/restart-minion.sh.jinja
    - mode: 755
```

The bash script that we put into crontab ```restart-minion.sh.jinja```:

```
SHELL=/bin/bash
SSH_TTY=/dev/pts/0
USER=root
MAIL=/var/spool/mail/root
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
LANG=en_US.UTF-8
SHLVL=1
HOME=/root
LOGNAME=root
LESSOPEN=|/usr/bin/lesspipe.sh %s
G_BROKEN_FILENAMES=1
_=/bin/env


service salt-minion restart
sleep 5
/usr/bin/salt-call state.sls minion.disable_restart {{ pillar['environment'] }}
```

Cronjob formula which places the bash script we created above into cronjob:

```
/tmp/restart-minion.sh:
  cron:
    - present
    - user: root
```

Then the formula to clean cron that we will call with salt-call in the bash script above on the minion:

```
/tmp/restart-minion.sh:
  cron:
    - absent
    - user: root
```

Finally ``init.sls`` to include required formulass:

```
include:
  - minion.install
  - minion.schedule_restart
```

All together looks like this

```
.
├── disable_restart.sls
├── files
│   ├── minion.jinja
│   └── restart-minion.sh.jinja
├── init.sls
├── install.sls
└── schedule_restart.sls

1 directory, 6 files
```

Now, when we create instaces we run minion formula as seen below:
```
salt 'my-instance' state.sls minion test-environment
```

Bug mentioned : https://github.com/saltstack/salt/issues/5721
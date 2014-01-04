### The Top File(s) and Environments

The top file defines which modules are accessible by which minions by utulizing a concept called environment.

Environments uses a `matcher` which defines which minons are under specific environment and then provides the modules to these minions via salt state system. All you need to think is salt has a file server and everthing is organized uder folders. Environment definition makes some of these folders ( and salt formulas in those folders) accesible by some minions. 

The top files under different environments (different folders) are compiled into single set of data following a few rules. 

First we need to define which environments are under which folders. Base is a special environment as if no environment if specified in it is going to be used by default.


```
# /etc/salt/master
# partial file

file_roots:
  base:
    - /srv/salt/base
  production:
  	- /srv/salt/production
  test:
  	- /srv/salt/test

...
```

Then lets create our base (default) `top.sls` file.

```
# /srv/salt/base/top.sls
base:
  '*':
  	- common_packages
production:
  'prod*':
  	- prod_packages
test:
  'test*':
  	- test_packages

```

Lets assume that `common_packages`, `prod_packages` and `test_packages` installs packages sepecific for the environment. `base_packages` intstalls `ntp` on all servers while `prod_packages` installs a specfic version of mysql or apache etc.

If we create other `top.sls` files under other environments defined in `/etc/salt/master`, all off these fill be merged into one top collection and top file in base environment will have priority over other files for the matchig environments.

Check this example. Assume we have two `top.sls` files, one for `base` one for `production`.

```
# /srv/salt/base/top.sls
base:
  '*':
  	- common_packages
test:
  'test*':
  	- test_packages
  	- add_users

```

```
# /srv/salt/production/top.sls
production:
  'prod*':
  	- prod_packages
test:
  'test*':
  	- test_packages
  	- install_web_server

```

The logical representation for `top.sls` file would be something like this:


```
# logical top data
base:
  '*':
  	- common_packages
production:
  'prod*':
  	- prod_packages
test:
  'test*':
  	- test_packages
  	- add_users

```

`test` environment would be represented as it is in `base` top file.

### Pillar of Salt

Pillar provides global values that can be distributed to all minions. It is made of files storing data in YAML format, requires `top.sls` file and is managed like states.

```
# /etc/salt/master
# partial file

pillar_roots:
  base:
    - /srv/salt/pillar
```

```
# /srv/salt/pillar/top.sls
# partial file

/srv/pillar/top.sls

base:
  '*':
    - packages
```

<p><small>reference: <a href="http://http://docs.saltstack.com/ref/states/top.html">docs.saltstack.com</a></small></p>
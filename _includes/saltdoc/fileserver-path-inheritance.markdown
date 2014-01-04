### Salt Fileserver Path Inheritance
Salt's fileserver allows for more than one root directory per environment, like in the below example, which uses two directories:

```
# In the master config file (/etc/salt/master)
file_roots:
  base:
    - /srv/salt
    - /srv/salt/base
```

Salt's fileserver collapses the list of root directories into a single virtual environment containing all files from each root. If the same file exists at the same relative path in more than one root, then the top-most match "wins". For example, if `/srv/salt/foo.txt` and `/srv/salt/base/foo.txt` both exist, then `salt://foo.txt` will point to `/srv/salt/foo.txt`.
<p><small>reference: <a href="http://docs.saltstack.com/topics/tutorials/states_pt4.html#environment-configuration">docs.saltstack.com</a></small></p>


---
layout: bootstrap_post
title: VirtualBox won't start on Ubuntu 14.04 (Trusty)
date: 2014-05-08 11:04:00
author: Oz Akan
abstract: When VirtualBox complains about /dev/vboxdrv...
categories:
    - Tips
---

If you are getting this error message

    WARNING: The character device /dev/vboxdrv does not exist.
         Please install the virtualbox-dkms package and the appropriate
         headers, most likely linux-headers-generic.

Do this

    # sudo apt-get install dkms build-essential linux-headers-$(uname -r)
    # sudo dpkg-reconfigure virtualbox-dkms
    # sudo dpkg-reconfigure virtualbox

Then you could start VirtualBox or run any command like the one below to create a local network.

    # sudo VBoxManage hostonlyif create ipconfig vboxnet0 --ip 192.168.56.1 --netmask 255.255.255.0
    0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
    Interface 'vboxnet0' was successfully created

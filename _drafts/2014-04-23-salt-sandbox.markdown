---
layout: bootstrap_post
title: SaltStack Sandbox
date: 2014-04-24 12:44:00
author: Oz Akan
abstract: You have created salt-master twice, which is more than once, automating it would be vise!
categories:
    - SaltStack
---

## Automating salt-master Installation

We had to create new salt-master every once in a while. It is relatively easy to install salt-master but the boring part is putting all required files in one place.

I will use vagrant and virtualbox on this article but for most part it can be replicated on the cloud using salt-cloud.

## Installing Vagrant and Virtualbox

Download VirtualBox
[https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads "Download VirtualBox")

Download Vagrant
[http://www.vagrantup.com/downloads.html](http://www.vagrantup.com/downloads.html "Download vagrantup")

As both VirtualBox and Vagrant have their installers, I think I don't need to write one more word under this section.

## VirtualBox Host-Only Network

We will create a host-only network for our VMs so they can communicate internally.

    $ VBoxManage hostonlyif create ipconfig vboxnet0 --ip 192.168.56.1 --netmask 255.255.255.0
    0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
    Interface 'vboxnet1' was successfully created

## Vagrant Section

#### Create a Vagrant Project.

    $ mkdir sandbox
    $ cd sandbox
    $ vagrant init

This will create a file named ```Vagrant``` which is the configuration file for our VMs.

#### Create Box

Boxes are ready to use base images. Download Ubuntu 13.4 (saucy)

    $ vagrant box add saucy https://cloud-images.ubuntu.com/vagrant/saucy/current/saucy-server-cloudimg-amd64-vagrant-disk1.box
    ==> box: Adding box 'saucy' (v0) for provider:
    box: Downloading: https://cloud-images.ubuntu.com/vagrant/saucy/current/saucy-server-cloudimg-amd64-vagrant-disk1.box
    ==> box: Successfully added box 'saucy' (v0) for 'virtualbox'!
    $ vagrant box add trusty https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box
    ==> box: Adding box 'trusty' (v0) for provider:
    box: Downloading: https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box
    ==> box: Successfully added box 'trusty' (v0) for 'virtualbox'!

#### Modify Vagrant File

Vagrant file will have this:

    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    # Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
    VAGRANTFILE_API_VERSION = "2"

    Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

      config.vm.define "master-init" do |master|
        master.vm.box = "saucy"
        master.vm.host_name = "master-init"
        master.vm.network :private_network, ip: "192.168.56.101"
        master.vm.network "public_network", :bridge => 'en0: Ethernet (AirPort)'
        master.vm.provision "shell",
          inline: "curl -L http://bootstrap.saltstack.org | sudo sh -s -- -M git v2014.1.3"
      end
      
      config.vm.define "master" do |minion|    
        minion.vm.box = "saucy"
        minion.vm.host_name = "master"
        minion.vm.network :private_network, ip: "192.168.56.102"
        minion.vm.network "public_network", :bridge => 'en0: Ethernet (AirPort)'
        minion.vm.provision :salt do |salt|
          salt.run_highstate = true
          salt.minion_config = "./salt/minion_on_master.conf"
          salt.minion_key = "./salt/master.pem"
          salt.minion_pub = "./salt/master.pub"
        end 
      end  

    end

You may want to spend some time reading the Vagrant configuration file above as we will take actions as seen below that will match the lines in the Vagrant file. ```"curl -L http://bootstrap.saltstack.org | sudo sh -s -- -M git v2014.1.3"``` indicates that we will use version ```2014.1.3```. You can remove ```git v2014.1.3``` and it will install the latest production version (details [here](http://docs.saltstack.com/en/latest/topics/tutorials/salt_bootstrap.html "salt-stack bootstrap")). Network interfaces are for OS X, so Linux and Windows hosts will have differences there.

#### Check Status

Check if you are able to see VM's.

    $ vagrant status
    Current machine states:

    master-init               not created (virtualbox)
    master                    not created (virtualbox)

#### Start master-init VM

We will start master-init first.

    $ vagrant up master-init

A few lines after, check if ```master-init``` is up

    $ vagrant status
    Current machine states:

    master-init               running (virtualbox)
    master                    not created (virtualbox)

Login to master-init

    $ vagrant ssh master-init

Verify salt-master is running

    $ ps -fe | awk {'print $9'} | grep salt | uniq
    /usr/bin/salt-minion
    /usr/bin/salt-master

#### Create Keys for Minion (Master VM)

We will create minion keys on master so we won't have to accept them manually each time we build minion.

    $ sudo salt-key --gen-keys=master

Copy public key into the folder of accepted minions:

    $ sudo cp master.pub /etc/salt/pki/master/minions/master

Verify if master is accepted (we don't yet have master vm running):

    $ salt-key -L
    Accepted Keys:
    master
    Unaccepted Keys:
    Rejected Keys:

Copy both public and private keys into your host OS file system.

    $ sudo scp master.p* me@192.168.56.1:~/sandbox/salt

#### Create Minion Configuration on Host

Create ```sandbox/salt/minion_on_master.conf``` file with the content below:

    master: 192.168.56.101
    file_client: remote

#### Create Master VM

    $ vagrant up master

After a few minutes, let's check if master can talk to minion

On master run this:

    $ salt master test.ping
    master:
        True




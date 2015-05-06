---
layout: bootstrap_post
title: SaltStack + VirtualBox + Vagrant
date: 2014-04-23 22:11:00
author: Oz Akan
abstract: To create the next great SaltStack formula, you need less than I thought you did.
categories:
    - SaltStack
---

## Need for MyCloud

It took a while for me to figure out that it might be a good idea to have a local environment where I can play with the SaltStack formulas we have been working on. Since I had easy access to cloud environment, like anyone on earth with a cloud account on one of the IaaS providers, my perception was the best way to develop SaltStack formulas was to have a salt-master on cloud and create as many minions as I need with salt-cloud. I would then destroy them, create them again.

Cycle would go on.

Then, I recognized that, I have a very underutilized laptop which can easily host a good amount of VMs. To be honest, before that, I just wanted to have an environment, which would be just mine. Where I could develop without hesitation to affect others' work in the team. I wanted to checkout my [salt-rocks](https://github.com/rackerlabs/salt-rocks "salt-rocks repo") fork quitely, make changes and create a pull request without relying on nothing but my laptop.

Meanwhile, we have several people in the team working on salt formulas. Even developers that are interested now have their own salt-masters. Thanks to the simplicity of SaltStack, I have never seen this much adoption in any team before. We have people using Linux, Windows and OS X. I wanted to have a solution that I could replicate for anyone independent of their OS.

Solution was, SaltStack + VirtualBox + Vagrant

We will attack the problem by creating a salt-master server and then minions on our own computer using virtualbox and vagrant. Vagrant will help us to manage virtual machines under virtualbox without using virtualbox gui so we can automate virtual machine creations. We will even be able to install salt on the VMs by bootstrapping. Life is good.

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

    $ mkdir saltvms
    $ cd saltvms
    $ vagrant init

This will create a file named ```Vagrant``` which is the configuration file for our VMs.

#### Create Box

Boxes are ready to use base images. Download Ubuntu 13.4 (saucy)

    $ vagrant box add saucy https://cloud-images.ubuntu.com/vagrant/saucy/current/saucy-server-cloudimg-amd64-vagrant-disk1.box
    ==> box: Adding box 'saucy' (v0) for provider:
    box: Downloading: https://cloud-images.ubuntu.com/vagrant/saucy/current/saucy-server-cloudimg-amd64-vagrant-disk1.box
    ==> box: Successfully added box 'saucy' (v0) for 'virtualbox'!

#### Modify Vagrant File

Vagrant file will have this:

    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    # Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
    VAGRANTFILE_API_VERSION = "2"

    Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

      config.vm.define "master" do |master|
        master.vm.box = "saucy"
        master.vm.host_name = "master"
        master.vm.network :private_network, ip: "192.168.56.102"
        master.vm.network "public_network", :bridge => 'en0: Ethernet (AirPort)'
      end

      config.vm.define "minion" do |minion|
        minion.vm.box = "saucy"
        minion.vm.host_name = "minion"
        minion.vm.network :private_network, ip: "192.168.56.103"
        minion.vm.network "public_network", :bridge => 'en0: Ethernet (AirPort)'
        minion.vm.provision :salt do |salt|
          salt.run_highstate = true
          salt.minion_config = "./minion.conf"
          salt.minion_key = "./minion.pem"
          salt.minion_pub = "./minion.pub"
        end
      end  

    end

You may want to spend some time reading the Vagrant configuration file above as we will take actions below that will match the lines in the Vagrant file. Network interfaces are for OS X, so Linux and Windows hosts will have differences there.

#### Check Status

Check if you are able to see VM's.

    $ vagrant status
    Current machine states:

    master                    not created (virtualbox)
    minion                    not created (virtualbox)

#### Start Master

We will start master first and install salt-master there. Then we will create keys to use with the minion which will make bootsrapping easier.

    $ vagrant up master

A few lines after, check if ```master``` is up

    $ vagrant status
    Current machine states:

    master                    running (virtualbox)
    minion                    not created (virtualbox)

## SaltStack Section

SaltStack has a bootstrap script which can be found at [https://github.com/saltstack/salt-bootstrap](https://github.com/saltstack/salt-bootstrap "SaltStack Bootstrap").
([Documentation here](http://docs.saltstack.com/en/latest/topics/tutorials/salt_bootstrap.html "SaltStack Bootstrap Documentation"))

We will use bootstrap scrips to install salt.

#### Login to Master and Install Salt

    $ vagrant ssh master

Now we are on master VM. Let's install salt-master and salt-minion

    $ curl -L http://bootstrap.saltstack.org | sudo sh -s -- -M

After several lines, we see both salt-master and salt-minion are installed and running on our master.

    $ ps -fe | awk {'print $9'} | grep salt | uniq
    /usr/bin/salt-minion
    /usr/bin/salt-master

#### Create Keys for Minion

We will create minion keys on master so we won't have to accept them manually each time we build minion.

    $ sudo salt-key --gen-keys=minion

Copy public key into the folder of accepted minions:

    $ sudo cp minion.pub /etc/salt/pki/master/minions/minion

Verify if minion is accepted (we don't yet have minion vm running):

    $ salt-key -L
    Accepted Keys:
    minion
    Unaccepted Keys:
    Rejected Keys:

Copy both public and private keys into your host OS file system.

    $ sudo scp minion.p* me@192.168.56.1:~/saltvms

#### Create Minion Configuration on Host

Create ```saltvms/minion.conf``` file with the content below:

    master: 192.168.56.102
    id: "minion"
    file_client: remote

#### Create Minion VM

    $ vagrant up minion

After a few minutes, let's check if master can talk to minion

On master run this:

    $ salt minion test.ping
    minion:
        True

All right. That was easy. We can create more minions by modifying Vagrant file. We may destroy and create minions to try our formulas from scratch without the need for any external resources. I believe this environment serves well for development purposes and can be replicated easily.

Enough? Not really. Stay tuned.

---
layout: bootstrap_post
title: Network Namespaces, OpenvSwitch and VLAN Tags
date: 2015-06-08 18:10:00
author: Oz Akan
abstract: Finally you found that network namespaces are being used for so long in so many places.
categories:
    - Cloud
---

You can google and find out what a linux network namespace is. Ok, I will save you from a search.

> A network namespace is logically another copy of the network stack, with its own routes, firewall rules, and network devices.

Ok so, how this is going to help me to do anything?

> By default a process inherits its network namespace from its parent. Initially all the processes share the same default network namespace from the init process.

That means, processes are associated with network namespaces thus can be isolated from each other by being run in different network namespaces.

If you run a web server in namespace named `ns-http` and curl it from `default` network namespace, you won't be able to access to port 80 unless it is configured to be accessible. `curl` and `httpd`, from networking point of view, act similar to two seperate processes running in two different virtual machines. Yes, virtual machines and containers use the same structure to be isolated.

## Show Me A Network Namespace

If you don't run linux on your computer or want to be on the safe side, as of tradition, let's boot up a linux server under virtualbox using vagrant. 

> If you are not going to use vagrant but just use virtualbox gui to boot up a linux server, please stop reading right now. You are not the type of person I would like to be in communicaton through this web site which happens to be against human keyboards, such as yourself. 

> If you use vagrant, please go ahead ;)


Need help with creating a VM under vagant? 
Check this short [vagrant tutorial]({% post_url 2015-01-01-vagrant-basics %}).


<a name="start_of_OvS"></a> 
### List Network Namespaces

ip tools, so `ip` command is used to alter network namespaces. To list available namespaces;

    $ ip netns list
    
We just booted up our linux VM and have no namespaces yet except the default one. The command we ran above is executed in the default network namespace. 

## Two Namespaces, 4 Virtual and One Physical Interface and 2 VLANs

I want to create two network namespaces (`ns1` and `ns2`), create two virtual interfaces in each (`tap1a`, `tab1b`, `tap2a` and `tap2b`), use VLANs (tagged as `100` and `200`) to isolate traffic between these two interfaces and also want to connect physical interface (`eth0`) to the OvS bridge (`br1`) so default network namespace can also communicate with these network namespaces and could be used as gateway.

<img src="/images/ovs_vlans.png" class="img-responsive center-block" alt="openvswitch diagram">

### Create Network Namespace

    $ ip netns add ns1
    mount --make-shared /var/run/netns failed: No such file or directory
    
Ups, let's be root. You can sudo, though I will just switch to root for convenience. 

    # ip netns add ns1
    # ip netns list
    ns1

Let's add one more

    # ip netns add ns2
    # ip netns list
    ns2
    ns1

At this point we have two network namespaces. I want to create a process under ns1 and ns2 and then I want these two processes communicate over TCP/IP on a specific VLAN. We need NICs under these network namespaces which will establish the connection. These NICs have to be virtual as we can not assign physical interfaces.

There a few ways to create virtual interfaces and connect them. Connecting two namespaces with veth pairs, connecting veth pairs to a linux bridge or openvswitch and finally by using OvS (Open vSwitch) ports which is the method we will focus on.

So, let's connect these two network namespaces using OvS ports.

### Install Open vSwitch

My Ubuntu didn't have OvS, so I had to install.

    # apt-get update -y
    # apt-get install openvswitch-switch -y
    # ovs-vsctl show
    7b7c0dd0-efaf-4296-b485-030e863ff260
        ovs_version: "2.0.2

We seem to have a functional OvS.

### Create New Switch (Bridge)

Let's create a bridge named `br1`

    # ovs-vsctl add-br br1
    # ovs-vsctl show
    7b7c0dd0-efaf-4296-b485-030e863ff260
        Bridge "br1"
            Port "br1"
                Interface "br1"
                    type: internal
        ovs_version: "2.0.2"

Once a bridge is created, it gets an internal port with the same name of bridge `br1` which can have an IP address of it's own and as seen in the output of `ovs-vsctl show` it's type is `internal`.

### Create Ports on OvS Bridge




<p class="highlight">...stay tuned, rest coming soon...</p>


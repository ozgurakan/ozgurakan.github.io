## Installing Vagrant and Virtualbox

Download VirtualBox
[https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads "Download VirtualBox")

Download Vagrant
[http://www.vagrantup.com/downloads.html](http://www.vagrantup.com/downloads.html "Download vagrantup")

As both VirtualBox and Vagrant have their installers, I think I don't need to write one more word under this section. Just please install these both before going to next section.

## VirtualBox Host-Only Network

We will create a host-only network for our VMs so they can communicate internally.

    $ VBoxManage hostonlyif create ipconfig vboxnet0 --ip 192.168.56.1 --netmask 255.255.255.0
    0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
    Interface 'vboxnet1' was successfully created

## Vagrant Section

#### Create a Vagrant Project.

    $ mkdir vagrant
    $ cd vagrant
    $ touch Vagrantfile

This will create an empty file named ```Vagrantfile``` which is the configuration file for our VMs. You can alternatively run command below to get a pre-populated configuration file.
    
    $ Vagrant init


#### Create Box

Boxes are ready to use base images. Download Ubuntu 13.4 (saucy)

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

      config.hostmanager.enabled = true
      config.hostmanager.manage_host = false
      config.hostmanager.ignore_private_ip = false
      config.hostmanager.include_offline = false


      config.vm.define "testvm" do |node|
        node.vm.box = "trusty"
        node.vm.hostname = "testvm"
        node.vm.network :private_network, ip:"192.168.56.2", :netmask => "255.255.255.0"

        node.vm.provider "virtualbox" do |v|
          v.memory = 512
          v.cpus = 1
        end
        
      end
      
    end

#### Check Status

Check if configuration is ok.

    $ vagrant status
    Current machine states:

    testvm                    not created (virtualbox)

    The environment has not yet been created. Run `vagrant up` to
    create the environment. If a machine is not created, only the
    default provider will be shown. So if a provider is not listed,
    then the machine is not created for that environment.

#### Start testvm

Start `testvm` by running the command below.

    $ vagrant up testvm

Once command is executed, check your new vm;

    $ vagrant status
    Current machine states:

    testvm                    running (virtualbox)

    The VM is running. To stop this VM, you can run `vagrant halt` to
    shut it down forcefully, or you can run `vagrant suspend` to simply
    suspend the virtual machine. In either case, to restart it again,
    simply run `vagrant up`.

#### Check out Vagrant commands

You may run `--help` or `list-commands` to see available commands.

    $ vagrant list-commands
    Below is a listing of all available Vagrant commands and a brief
    description of what they do.

    box             manages boxes: installation, removal, etc.
    connect         connect to a remotely shared Vagrant environment
    destroy         stops and deletes all traces of the vagrant machine
    docker-logs     outputs the logs from the Docker container
    docker-run      run a one-off command in the context of a container
    global-status   outputs status Vagrant environments for this user
    halt            stops the vagrant machine
    help            shows the help for a subcommand
    hostmanager     
    init            initializes a new Vagrant environment by creating a Vagrantfile
    list-commands   outputs all available Vagrant subcommands, even non-primary ones
    login           log in to Vagrant Cloud
    package         packages a running vagrant environment into a box
    plugin          manages plugins: install, uninstall, update, etc.
    provision       provisions the vagrant machine
    rdp             connects to machine via RDP
    reload          restarts vagrant machine, loads new Vagrantfile configuration
    resume          resume a suspended vagrant machine
    rsync           syncs rsync synced folders to remote machine
    rsync-auto      syncs rsync synced folders automatically when files change
    share           share your Vagrant environment with anyone in the world
    ssh             connects to machine via SSH
    ssh-config      outputs OpenSSH valid configuration to connect to the machine
    status          outputs status of the vagrant machine
    suspend         suspends the machine
    up              starts and provisions the vagrant environment
    version         prints current and latest Vagrant version

Saw docker there? A good topic for next time, right?

#### SSH to Your VM

Vagrant can ssh into your VM.

    # vagrant ssh testvm
    
Now you can con use this VM for what ever purpose you like.


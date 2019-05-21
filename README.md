# ansible-generic-help

Why ansible?
============

Deploy the roles in a way that is faster than doing all manually, keep centralized the configuration. 
You don't need to install any client on servers that will be on the inventory, only python is required and it is installed by default on most of the systems.

ansible roles makes our work much more shareable, and you can start to use all what we have solved immediately. 

You can even use it when you have only one host, doesn't matter if you have only one or hundred of hosts, it will always make your sysadmin life better :). 


Getting Started
================

You need to know some basics from [ansible](http://docs.ansible.com/ansible/quickstart.html), and ofcourse [install ansible](http://docs.ansible.com/ansible/intro_installation.html)

Don't be scared about it, ansible is the most simple IT automation tool and you will get benefits learing this to manage
any other thing on your servers. Check also [how ansible works](https://www.ansible.com/how-ansible-works), and [get-started](https://www.ansible.com/get-started)


[install ansible](http://docs.ansible.com/ansible/intro_installation.html)

Note for fedora (when using pip3 (python3)):  
You need `python3-devel openssl-devel redhat-rpm-config` and use pip3 install  
Install also: `redhat-rpm-config` if not installed pip got error: `gcc: error: /usr/lib/rpm/redhat/redhat-hardened-cc1:` No such file or directory


Note for Ubuntu16.04+ (when using pip):
Have installed: `python3-dev libssl-dev` 

For all distros (when using pip):

    pip3 install ansible
    
Or specific version:

    pip3 install ansible==2.7.10

See: https://pypi.org/project/ansible/#history

Also use `pip3 install --upgrade pip` before install  

A **simple example with burp2_server role**, but you can change the name of the role to any other on [ansible-galaxy](https://galaxy.ansible.com/):


Installing roles
----------------

Before starting to define the usage of our roles, we need them installed on our 
**[ansible-control-machine](http://docs.ansible.com/ansible/intro_installation.html#control-machine-requirements)**.
 As we will use this machine to actively connect and push configurations to our **server/s**. 


We will use `requirements.yml` file to make the installation of roles on multiple machines, and install multiple roles
with one command.

Add/create your `requirements.yml` file: 

```yaml

# burp2_server

- src: CoffeeITWorks.burp2_server
  version: master
```

It is also possible to use github link  see: http://docs.ansible.com/ansible/latest/reference_appendices/galaxy.html#installing-multiple-roles-from-a-file 

Then use: 

    $ sudo ansible-galaxy install -r requirements.yml

Or in a traditional way:

Install the role on the system: 

    $ ansible-galaxy install CoffeeITWorks.burp2_server

Start using [Examples on this repo](#Examples on this repo)

Then deploy: 
    
    $ cd to dir where you have your site.yml and inventory
    
    $ ansible-playbook -i inventory site.yml --limit burp2_servers --tag burp2_server -u user -k -K 

    # -k is to ask password for user
    # -K is to ask sudo password for the user

Done! you have deployed the role choosen  
Now you can use same steps to add any other role to your servers, separate them in different groups, etc. 

Check the manpage for more information 

http://linux.die.net/man/1/ansible-playbook

Good structure of files
=======================

```shell
.
├── ansible.cfg
├── inventory
│   ├── group_vars
│   ├── host_vars
│   └── test
├── requirements.yml
├── roles
├── roles.burp2_servers.yml
└── site.yml
```

Having this structure and running `ansible-playbook` command from that directory will provide you a lot of potential.

You don't need to modify the roles itself when you install one.
You can have your own roles inside `roles/` dir, also clone or add as git submodule on that dir.
When you need to use your own values on some variables, add them inside `inventory/group_vars` or `inventory/host_vars` files, so you don't modify the roles file inside `defaults`.


Additional information about our roles
======================================

In most of our roles: 

Ubuntu support is automatically tested with molecule docker Ubuntu 14.04 and latest image.  
Debian support is automatically tested with molecule docker Debian Jessie (8)  
Centos/7 support is tested locally, check molecule.yml file for more information  


Environments
============

These roles can be used in any environment, like: production, testing, development.  

Also ansible allows you to use almost any platform, like: bare metals servers, VMs, containers, lxc, openstack, e2, etc. 


Roles Variables
===============

### Add to your host/group_vars:
 
Create host_vars or group_vars dirs to specify you own variables to override default settings.  

Check the [Examples on this repo](#Examples on this repo)  

Inside it you can add a file with the name of the group or the host where you want to add specific options of this role.  

Every role has variables in `defaults/main.yml`, check those variables and then you can override any default using your host/group_vars.  



Examples on this repo
=====================

check [localhost_only](localhost_only) or [example1](example1) examples

Then modify the inventory according to you real servers  
Also modify the group_vars if required.  

If required you can also modify ansible.cfg to have more options, check the ansible.cfg doc. 

Ensure you have communication to your servers before start.  

Check connection before run:

    ssh user@address

Extra tips
==========

Use `ssh-copy-id user@address` to avoid usage of -k (ask password)

Increase your knowledge about ansible: 

http://docs.ansible.com/ansible/playbooks_best_practices.html

More info for developers:
=========================

[Developers.md](Developers.md)

Other resources:
================

https://networklore.com/ansible-getting-started/

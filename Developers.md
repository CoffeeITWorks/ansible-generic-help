Testing for developers
======================

https://molecule.readthedocs.io/en/latest/

The following examples are to test with molecule, but you can also use your own servers and setup an [Ansible inventory](http://docs.ansible.com/ansible/intro_inventory.html)

Molecule resolves some good things for a dev environment, like: automatic provision new hosts, automatic inventory for ansible, automatic deploy, automatic destroy, automatic test idempotence, etc. 

* Install molecule

    sudo pip install molecule
    

Testing with molecule+docker: 
-----------------------------

I have stopped using docker due to some strange issues with centos7 and systemd with docker. 

* clone this repo and move to that dir.
* Install docker-engine


Testing with molecule+vagrant: 
------------------------------

It's very useful for local test and ansible development, also to test burp with ansible in multiple environments/distribution. 

* have installed the role (see getting started)
* [Install vagrant](https://www.vagrantup.com/docs/installation/)


Choose your provider, example for libvirt: 

[vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt)

modify ´molecule.yml´ file and change driver, example: 

    driver:
      name: vagrant

Also ensure you are using same provider as choosen:


    providers:
      - name: libvirt
        type: libvirt

Run molecule
------------

    sudo molecule test 
